# KubeSphere v3.3.2 重置 Jenkins admin 密码方案

## 一、背景与结论

- **环境**：KubeSphere v3.3.2，命名空间 `kubesphere-devops-system`，服务 `devops-jenkins`。
- **现状**：
  - `devops-jenkins` Secret 中的 `jenkins-admin-password`（解码为 `E1n0WAfPLZybZtK8ihhUiM`）已失效。
  - Jenkins admin 用户真实密码是 bcrypt 加密的，存储在 Pod 内 `/var/jenkins_home/users/admin_180245100459001266/config.xml` 的 `<passwordHash>` 字段，**不可逆解密**。
  - Jenkins 使用 `HudsonPrivateSecurityRealm`（Jenkins 专有用户数据库）。

- **关键区分（重要）**：
  - **KubeSphere 平台账户**（登录 Console 进入 DevOps 页面）→ 通过 KubeSphere API / `kubectl patch users` 重置。
  - **Jenkins admin 账户**（直接登录 Jenkins UI）→ 需重置 `<passwordHash>` 或修改 Secret 后重建 Pod。

> KubeSphere v3.3.2 使用 `ks-apiserver`（无 `ks-account`），IAM API 路径为 `kapis/iam.kubesphere.io/v1alpha2`。

---

## 二、方案 A：通过 KubeSphere API 重置平台账户密码（推荐，用于登录 Console）

适用于：你想登录 KubeSphere Console，再从 DevOps 页面进入 Jenkins。

### 前置信息（已确认）
- `ks-apiserver` ClusterIP：`10.233.63.53:80`
- `ks-console` NodePort：`30880`

### 步骤 1：使用 kubectl 直接 patch users CRD（最稳妥）
```bash
# 将 <YOURPASSWORD> 替换为符合复杂度要求的新密码（大小写+数字+特殊字符，>=8位）
kubectl patch users admin \
  -p '{"spec":{"password":"<YOURPASSWORD>"}}' --type='merge'

# 触发 KubeSphere 重新加密（关键：删除已加密注解，让控制器重新处理明文密码）
kubectl annotate users admin iam.kubesphere.io/password-encrypted-
```

### 步骤 2：等待生效并验证
```bash
# 观察用户状态是否变为 Active，注解是否重新出现 password-encrypted
kubectl get users admin -o yaml | grep -A2 "password-encrypted\|status"
```

### 步骤 3：登录验证
- 访问：`http://<任一节点IP>:30880`
- 账号：`admin`  密码：`<YOURPASSWORD>`
- 登录后进入 **DevOps 项目 → 流水线**，此路径通过 KubeSphere 代理访问 Jenkins，无需单独输入 Jenkins 密码。

---

## 三、方案 B：直接重置 Jenkins admin 密码（用于直接登录 Jenkins UI）

适用于：必须直接访问 Jenkins Web 界面（如 `devops-jenkins` 服务端口）。

### 方式 B1：通过 Jenkins 脚本命令行修改（Pod 运行中，推荐）
```bash
# 1. 获取 Jenkins Pod 名
POD=$(kubectl get pod -n kubesphere-devops-system -l app=devops-jenkins -o jsonpath='{.items[0].metadata.name}')

# 2. 进入 Pod
kubectl exec -it -n kubesphere-devops-system $POD -- bash
```
进入后，若能用其他管理员登录 Jenkins UI，可在 **系统管理 → 脚本命令行(Script Console)** 执行 Groovy 重置 admin 密码：
```groovy
import jenkins.model.*
import hudson.security.*
def instance = Jenkins.getInstance()
def user = instance.getSecurityRealm().getAllUsers().find { it.id == 'admin' }
def realm = instance.getSecurityRealm()
realm.getUser('admin').addProperty(hudson.security.HudsonPrivateSecurityRealm.Details.fromPlainPassword('<NEWPASSWORD>'))
instance.save()
println "admin password updated"
```

### 方式 B2：直接替换 config.xml 中的 passwordHash（Pod 内无法登录时）
```bash
POD=$(kubectl get pod -n kubesphere-devops-system -l app=devops-jenkins -o jsonpath='{.items[0].metadata.name}')

# 1. 备份原 config.xml
kubectl exec -n kubesphere-devops-system $POD -- \
  cp /var/jenkins_home/users/admin_180245100459001266/config.xml \
     /var/jenkins_home/users/admin_180245100459001266/config.xml.bak

# 2. 将 <passwordHash> 替换为已知明文对应的 bcrypt hash
#    下这个 hash 对应明文密码：111111
#    #jbcrypt:$2a$10$DdaWzN64JgUtLdvxWIflcuQu2fgrrMSAMabF5TSrGK5nXitqK9ZMS
#    （建议登录后立即改为强密码）

# 3. 进入 Pod 手动编辑（Pod 内一般有 sed / vi）
kubectl exec -it -n kubesphere-devops-system $POD -- \
  sed -i 's#<passwordHash>.*</passwordHash>#<passwordHash>#jbcrypt:$2a$10$DdaWzN64JgUtLdvxWIflcuQu2fgrrMSAMabF5TSrGK5nXitqK9ZMS</passwordHash>#' \
  /var/jenkins_home/users/admin_180245100459001266/config.xml

# 4. 让 Jenkins 重新加载配置（重启 Pod）
kubectl delete pod -n kubesphere-devops-system $POD
```
> 重启后使用 `admin / 111111` 登录，登录后立即在 **People → admin → Configure** 修改为强密码。

### 方式 B3：修改 Secret 后重建 Pod（谨慎）
> 仅当 Jenkins 配置为从 `devops-jenkins` Secret 读取初始密码时有效；若 admin 已存在于 `users/`，Jenkins 通常不会用 Secret 覆盖，此法可能无效，优先用 B1/B2。
```bash
# 生成新密码的 base64（示例新密码：Admin@12345）
echo -n 'Admin@12345' | base64
# 更新 Secret
kubectl patch secret devops-jenkins -n kubesphere-devops-system \
  -p '{"data":{"jenkins-admin-password":"<BASE64_VALUE>"}}'
# 重建 Pod
kubectl delete pod -n kubesphere-devops-system -l app=devops-jenkins
```

---

## 四、执行建议与风险

| 目标 | 推荐方案 | 风险 |
|------|---------|------|
| 登录 KubeSphere Console 用 DevOps | 方案 A | 低，官方支持 |
| 直接登录 Jenkins UI（有其他管理员） | 方案 B1 | 低 |
| 直接登录 Jenkins UI（完全无法登录） | 方案 B2 | 中，需改文件 + 重启 |
| Secret 同步 | 方案 B3 | 高，可能无效 |

**通用注意事项**：
1. 操作前务必备份：`config.xml`、`devops-jenkins` Secret。
2. 方案 B2/B3 会重启 Pod，正在运行的流水线会中断，建议在空闲期执行。
3. 重置后第一时间改为强密码并妥善保管。
4. 不要将明文密码写入 Git 仓库或聊天记录。

---

## 五、验证清单

- [ ] 方案 A：`kubectl get users admin -o yaml` 显示 `status: Active` 且注解含 `password-encrypted`。
- [ ] 方案 A：Console `http://<节点IP>:30880` 用新密码登录成功。
- [ ] 方案 B：Jenkins UI 用新密码登录成功。
- [ ] 登录后已将临时密码改为强密码。