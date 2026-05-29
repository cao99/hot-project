# cc-switch 完整使用指引

> **版本**: 1.0.0  
> **最后更新**: 2026-05-29  
> **作者**: Cloud Team

---

## 目录

1. [项目概述](#1-项目概述)
2. [系统架构](#2-系统架构)
3. [安装指南](#3-安装指南)
4. [快速开始](#4-快速开始)
5. [基础功能](#5-基础功能)
6. [配置管理](#6-配置管理)
7. [云服务商配置](#7-云服务商配置)
8. [高级功能](#8-高级功能)
9. [安全管理](#9-安全管理)
10. [故障排查](#10-故障排查)
11. [API 参考](#11-api-参考)
12. [开发与扩展](#12-开发与扩展)
13. [最佳实践](#13-最佳实践)
14. [常见问题](#14-常见问题)
15. [附录](#15-附录)

---

## 1. 项目概述

### 1.1 什么是 cc-switch

**cc-switch** (Cloud Configuration Switcher) 是一个强大的命令行工具，专为管理多云环境配置而设计。它允许开发者和运维人员在不同的云服务提供商配置之间快速切换，极大地简化了多云架构的管理复杂度。

#### 核心价值

在现代云计算环境中，企业和开发团队往往需要在多个云服务商（AWS、Azure、GCP 等）之间进行操作。每个云服务商都有自己的配置文件、凭证管理和环境设置。这种多样性带来了以下挑战：

- **配置管理复杂**: 不同云服务商的配置文件格式、存储位置各不相同
- **环境切换繁琐**: 手动切换开发、测试、生产环境容易出错
- **凭证安全风险**: 多账号管理增加了凭证泄露的风险
- **团队协作困难**: 团队成员可能使用不同的配置，导致环境不一致

cc-switch 通过统一的命令行界面和智能配置管理机制，完美解决了这些问题。

### 1.2 主要功能

#### 1.2.1 多云配置管理

支持主流云服务提供商：

| 云服务商 | 支持版本 | 配置类型 | 特殊功能 |
|---------|---------|---------|---------|
| AWS | 全版本 | Credentials, Config, CLI | IAM Role 切换, MFA 支持 |
| Azure | 全版本 | Service Principal, CLI, PowerShell | Tenant 管理, Subscription 切换 |
| GCP | 全版本 | Service Account, gcloud config | Project 切换, 区域管理 |
| Alibaba Cloud | 全版本 | AccessKey, RAM Role | 资源组管理 |
| Tencent Cloud | 全版本 | SecretId/Key, CAM | 账号体系集成 |
| Custom | 可扩展 | 自定义配置 | 插件系统支持 |

#### 1.2.2 环境管理

- **多环境支持**: 轻松管理 dev、staging、prod 等环境
- **快速切换**: 一键切换不同环境的配置
- **环境隔离**: 配置文件完全隔离，避免交叉污染
- **环境继承**: 支持配置继承，减少重复配置

#### 1.2.3 凭证管理

- **安全存储**: 使用系统密钥链（Keychain/Keyring）加密存储凭证
- **自动刷新**: 支持 OAuth token 和临时凭证自动刷新
- **权限控制**: 细粒度的访问权限管理
- **审计日志**: 完整的凭证使用记录

#### 1.2.4 CI/CD 集成

- **Pipeline 友好**: 专为 CI/CD 流水线设计
- **环境变量注入**: 自动注入必要的配置环境变量
- **无交互模式**: 支持完全非交互式操作
- **Docker 支持**: 提供 Docker 镜像，方便容器化部署

### 1.3 适用场景

#### 场景一：开发团队的多环境管理

**背景**: 一个开发团队需要在 AWS 的开发、测试、生产环境中频繁切换，每个环境有不同的账号和权限配置。

**解决方案**:
```bash
# 添加三个环境的配置
cc-switch add aws --profile dev --account 123456789012
cc-switch add aws --profile staging --account 234567890123
cc-switch add aws --profile prod --account 345678901234

# 快速切换
cc-switch use aws-dev        # 切换到开发环境
cc-switch use aws-staging    # 切换到测试环境
cc-switch use aws-prod       # 切换到生产环境
```

#### 场景二：多云架构管理

**背景**: 企业采用多云策略，使用 AWS 托管计算服务，Azure 托管数据库，GCP 托管机器学习服务。

**解决方案**:
```bash
# 添加多云配置
cc-switch add aws --profile compute
cc-switch add azure --profile database
cc-switch add gcp --profile ml

# 使用组合配置
cc-switch use-group full-stack  # 同时激活多个云配置
```

#### 场景三：CI/CD 自动化

**背景**: 在 CI/CD 流水线中，需要根据部署阶段自动切换云配置。

**解决方案**:
```bash
# GitLab CI 配置示例
deploy:
  stage: deploy
  script:
    - cc-switch use aws-${CI_ENVIRONMENT_NAME} --non-interactive
    - ./deploy.sh
```

### 1.4 技术特点

#### 1.4.1 架构设计

cc-switch 采用模块化架构设计：

```
┌─────────────────────────────────────────┐
│          CLI Interface Layer            │
│  (commander.js + inquirer.js)           │
└─────────────┬───────────────────────────┘
              │
┌─────────────▼───────────────────────────┐
│       Configuration Manager             │
│  (YAML Parser + File System)            │
└─────────────┬───────────────────────────┘
              │
┌─────────────▼───────────────────────────┐
│       Provider Abstraction Layer        │
│  (AWS/Azure/GCP/Custom Providers)       │
└─────────────┬───────────────────────────┘
              │
┌─────────────▼───────────────────────────┐
│       Security Layer                    │
│  (keytar + encryption)                  │
└─────────────────────────────────────────┘
```

#### 1.4.2 核心依赖

| 依赖包 | 版本 | 用途 |
|-------|------|------|
| commander | ^9.0.0 | CLI 框架 |
| inquirer | ^8.2.0 | 交互式命令行 |
| chalk | ^4.1.2 | 终端颜色输出 |
| ora | ^5.4.1 | 进度指示器 |
| yaml | ^2.1.0 | YAML 配置解析 |
| keytar | ^7.9.0 | 系统密钥链集成 |
| node-fetch | ^2.6.7 | HTTP 请求 |

#### 1.4.3 性能优化

- **配置缓存**: 智能缓存常用配置，减少磁盘 I/O
- **懒加载**: 按需加载云服务商模块
- **并行操作**: 支持配置并行验证和同步
- **增量更新**: 只更新变更的配置项

### 1.5 版本历史

| 版本 | 发布日期 | 主要特性 |
|-----|---------|---------|
| 1.0.0 | 2026-05-01 | 初始版本，支持 AWS/Azure/GCP |
| 0.9.0 | 2026-04-15 | Beta 版本，添加凭证管理 |
| 0.8.0 | 2026-03-01 | Alpha 版本，基础配置切换 |

### 1.6 许可证

本项目采用 MIT 许可证，详见 LICENSE 文件。

---

## 2. 系统架构

### 2.1 整体架构

cc-switch 的系统架构遵循分层设计原则，每一层都有明确的职责边界：

#### 2.1.1 表示层 (Presentation Layer)

负责用户交互和命令解析：

```javascript
// 命令解析示例
program
  .command('add <provider>')
  .option('--profile <name>', 'Configuration profile name')
  .option('--account <id>', 'Account identifier')
  .action(async (provider, options) => {
    await configManager.addProvider(provider, options);
  });
```

主要组件：
- **Command Parser**: 解析命令行参数和选项
- **Interactive Prompt**: 提供交互式输入界面
- **Output Formatter**: 格式化输出结果
- **Progress Indicator**: 显示长时间操作的进度

#### 2.1.2 业务逻辑层 (Business Logic Layer)

核心业务逻辑处理：

```javascript
// 配置管理器
class ConfigManager {
  constructor() {
    this.providers = new Map();
    this.cache = new ConfigCache();
  }
  
  async addProvider(name, config) {
    // 验证配置
    await this.validateConfig(config);
    // 存储配置
    await this.storeConfig(name, config);
    // 更新缓存
    this.cache.update(name, config);
  }
}
```

主要模块：
- **Config Manager**: 配置的增删改查
- **Switch Manager**: 配置切换逻辑
- **Validation Engine**: 配置验证
- **Sync Manager**: 多节点配置同步

#### 2.1.3 数据访问层 (Data Access Layer)

处理数据持久化：

```javascript
// 数据存储抽象
class DataStore {
  async save(key, value) {
    const encrypted = await this.encrypt(value);
    await this.writeFile(key, encrypted);
  }
  
  async load(key) {
    const encrypted = await this.readFile(key);
    return await this.decrypt(encrypted);
  }
}
```

主要组件：
- **File Storage**: 文件系统存储
- **Keychain Integration**: 系统密钥链集成
- **Cache Layer**: 内存缓存
- **Backup Manager**: 备份和恢复

#### 2.1.4 提供商适配层 (Provider Adapter Layer)

云服务商特定逻辑：

```javascript
// AWS 提供商适配器
class AWSProvider extends BaseProvider {
  async configure(options) {
    // 生成 AWS 配置文件
    await this.generateCredentials(options);
    await this.generateConfig(options);
    // 设置环境变量
    this.setEnvironment({
      AWS_PROFILE: options.profile,
      AWS_DEFAULT_REGION: options.region
    });
  }
}
```

支持的提供商：
- **AWS Provider**: AWS 配置管理
- **Azure Provider**: Azure 配置管理
- **GCP Provider**: GCP 配置管理
- **Custom Provider**: 自定义提供商

### 2.2 数据流架构

#### 2.2.1 配置添加流程

```
User Input (CLI)
    │
    ▼
Command Parser ───▶ Validate Input
    │                     │
    │                     ▼
    │              Is Valid? ──No──▶ Show Error
    │                     │
    │                    Yes
    │                     │
    ▼                     ▼
Provider Adapter ───▶ Test Connection
    │                     │
    │                     ▼
    │              Success? ──No──▶ Show Error
    │                     │
    │                    Yes
    │                     │
    ▼                     ▼
Security Layer ───▶ Encrypt Credentials
    │
    ▼
File Storage ───▶ Save Configuration
    │
    ▼
Update Cache
    │
    ▼
Notify User
```

#### 2.2.2 配置切换流程

```
User Request (cc-switch use <name>)
    │
    ▼
Load Configuration
    │
    ▼
Decrypt Credentials
    │
    ▼
Validate Current State
    │
    ▼
Backup Current Config
    │
    ▼
Apply New Config
    │
    ├─▶ Update ~/.aws/credentials
    ├─▶ Update ~/.azure/config
    ├─▶ Set Environment Variables
    └─▶ Update Shell Profile
    │
    ▼
Verify Application
    │
    ▼
Update Active State
    │
    ▼
Notify User
```

### 2.3 配置文件结构

#### 2.3.1 主配置文件

位置: `~/.cc-switch/config.yaml`

```yaml
# 主配置文件结构
version: 1.0
active_profile: aws-dev

settings:
  auto_backup: true
  cache_enabled: true
  log_level: info
  default_region: us-east-1

providers:
  aws:
    - name: aws-dev
      account: "123456789012"
      region: us-east-1
      profile: dev
      credentials:
        type: access_key
        key_id: ${ENCRYPTED:AES256:...}
        secret: ${ENCRYPTED:AES256:...}
      
  azure:
    - name: azure-prod
      tenant: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
      subscription: "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy"
      credentials:
        type: service_principal
        client_id: ${ENCRYPTED:AES256:...}
        client_secret: ${ENCRYPTED:AES256:...}
```

#### 2.3.2 凭证存储

凭证采用双层加密存储：

1. **系统密钥链加密**: 使用操作系统提供的密钥链服务
2. **应用层加密**: AES-256-GCM 算法二次加密

```javascript
// 凭证加密流程
async function encryptCredential(plaintext) {
  // 第一层：应用层加密
  const appKey = await deriveKey(masterPassword);
  const encrypted = await aesGcmEncrypt(plaintext, appKey);
  
  // 第二层：系统密钥链
  const keychainKey = await keytar.setPassword(
    'cc-switch',
    'credential-key',
    encrypted
  );
  
  return encrypted;
}
```

#### 2.3.3 环境配置文件

每个环境维护独立的配置文件：

```
~/.cc-switch/
├── config.yaml           # 主配置
├── profiles/
│   ├── aws-dev.yaml
│   ├── aws-staging.yaml
│   ├── aws-prod.yaml
│   ├── azure-dev.yaml
│   └── gcp-prod.yaml
├── cache/
│   ├── sessions.cache
│   └── metadata.cache
└── logs/
    ├── cc-switch.log
    └── audit.log
```

### 2.4 安全架构

#### 2.4.1 认证机制

cc-switch 支持多种认证方式：

**1. 交互式认证**
```bash
cc-switch login
# 提示输入主密码
```

**2. 环境变量认证**
```bash
export CC_SWITCH_MASTER_PASSWORD="your-password"
cc-switch use aws-prod
```

**3. 文件认证**
```bash
echo "your-password" | cc-switch login --stdin
```

#### 2.4.2 权限模型

```
┌─────────────────────────────────────┐
│          Permission Model           │
└─────────────┬───────────────────────┘
              │
    ┌─────────┴─────────┐
    │                   │
    ▼                   ▼
Admin Role          User Role
    │                   │
    ├─ Create Config    ├─ Use Config
    ├─ Delete Config    ├─ List Configs
    ├─ Modify Config    ├─ View Own Configs
    └─ Manage Users     └─ Change Password
```

#### 2.4.3 审计日志

所有操作都会记录到审计日志：

```json
{
  "timestamp": "2026-05-29T10:30:45.123Z",
  "user": "john.doe@example.com",
  "action": "config.switch",
  "resource": "aws-prod",
  "source_ip": "192.168.1.100",
  "result": "success",
  "details": {
    "from_profile": "aws-dev",
    "to_profile": "aws-prod"
  }
}
```

### 2.5 扩展机制

#### 2.5.1 插件系统

cc-switch 支持插件扩展：

```javascript
// 插件接口定义
interface CCPlugin {
  name: string;
  version: string;
  
  // 生命周期钩子
  onLoad(): Promise<void>;
  onUnload(): Promise<void>;
  
  // 扩展点
  providers?: Provider[];
  commands?: Command[];
  formatters?: Formatter[];
}

// 插件示例
class MyCustomProvider implements CCPlugin {
  name = 'my-custom-provider';
  version = '1.0.0';
  
  providers = [new CustomCloudProvider()];
  
  async onLoad() {
    console.log('Custom provider loaded');
  }
}
```

#### 2.5.2 钩子系统

关键操作的钩子：

```yaml
# 钩子配置
hooks:
  before_switch:
    - name: notify-team
      script: /scripts/notify.sh
      async: true
      
  after_switch:
    - name: verify-connection
      script: /scripts/verify.py
      timeout: 30
      
  on_error:
    - name: rollback
      automatic: true
      
  on_success:
    - name: log-success
      script: /scripts/log.sh
```

---

## 3. 安装指南

### 3.1 系统要求

#### 3.1.1 操作系统支持

| 操作系统 | 最低版本 | 推荐版本 | 备注 |
|---------|---------|---------|------|
| macOS | 10.15 (Catalina) | 13.0+ (Ventura) | 需要 Xcode Command Line Tools |
| Windows | Windows 10 | Windows 11 | 需要 PowerShell 5.1+ |
| Linux | Ubuntu 18.04 | Ubuntu 22.04 | 需要 build-essential |
| Fedora | 30+ | 36+ | 需要 development tools |
| CentOS | 7+ | Stream 9 | 需要 EPEL repository |

#### 3.1.2 软件依赖

**必需依赖**:
- Node.js: v16.0.0 或更高版本
- npm: v7.0.0 或更高版本
- Git: v2.0.0 或更高版本

**可选依赖**:
- Python 3.8+: 用于部分脚本和插件
- Docker: 用于容器化部署

#### 3.1.3 硬件要求

| 资源 | 最低配置 | 推荐配置 |
|-----|---------|---------|
| CPU | 1 core | 2+ cores |
| 内存 | 512 MB | 1 GB+ |
| 磁盘空间 | 100 MB | 500 MB+ |

### 3.2 安装方式

#### 3.2.1 通过 npm 安装（推荐）

**全局安装**:
```bash
npm install -g cc-switch
```

**验证安装**:
```bash
cc-switch --version
# 输出: cc-switch v1.0.0
```

**安装特定版本**:
```bash
npm install -g cc-switch@1.0.0
```

#### 3.2.2 从源码安装

**克隆仓库**:
```bash
git clone https://github.com/cloudteam/cc-switch.git
cd cc-switch
```

**安装依赖**:
```bash
npm install
```

**构建项目**:
```bash
npm run build
```

**全局链接**:
```bash
npm link
```

#### 3.2.3 使用 Docker 安装

**拉取镜像**:
```bash
docker pull cloudteam/cc-switch:latest
```

**运行容器**:
```bash
docker run -it --rm \
  -v ~/.cc-switch:/root/.cc-switch \
  -v ~/.aws:/root/.aws \
  cloudteam/cc-switch:latest \
  cc-switch list
```

**创建别名**:
```bash
alias cc-switch='docker run -it --rm \
  -v ~/.cc-switch:/root/.cc-switch \
  -v ~/.aws:/root/.aws \
  cloudteam/cc-switch:latest cc-switch'
```

#### 3.2.4 使用二进制文件安装

从 GitHub Releases 下载预编译的二进制文件：

```bash
# macOS/Linux
curl -L https://github.com/cloudteam/cc-switch/releases/latest/download/cc-switch-$(uname -s)-$(uname -m) -o /usr/local/bin/cc-switch
chmod +x /usr/local/bin/cc-switch

# Windows (PowerShell)
Invoke-WebRequest -Uri "https://github.com/cloudteam/cc-switch/releases/latest/download/cc-switch-win-amd64.exe" -OutFile "cc-switch.exe"
Move-Item cc-switch.exe -Destination "C:\Program Files\cc-switch\"
```

### 3.3 初始化配置

#### 3.3.1 首次运行

安装完成后，需要进行初始化配置：

```bash
cc-switch init
```

初始化向导会引导您完成以下步骤：

```
? Choose your default cloud provider:
  ❯ AWS
    Azure
    GCP
    Alibaba Cloud
    Tencent Cloud

? Enter your master password for encryption: ********

? Confirm master password: ********

? Choose cache strategy:
  ❯ In-memory (faster, lost on restart)
    File-based (persistent, slower)

? Enable auto-backup? (Y/n): Y

? Configure shell integration?
  ❯ Yes (recommended)
    No
```

#### 3.3.2 配置文件位置

初始化后，配置文件将存储在：

| 平台 | 配置目录 |
|-----|---------|
| macOS/Linux | `~/.cc-switch/` |
| Windows | `%USERPROFILE%\.cc-switch\` |

目录结构：
```
~/.cc-switch/
├── config.yaml           # 主配置文件
├── credentials.enc       # 加密的凭证库
├── cache/                # 缓存目录
│   ├── metadata.json
│   └── sessions.json
├── logs/                 # 日志目录
│   ├── cc-switch.log
│   └── audit.log
├── backups/              # 备份目录
└── plugins/              # 插件目录
```

#### 3.3.3 环境变量配置

cc-switch 支持通过环境变量进行配置：

```bash
# 主密码（不推荐在生产环境使用）
export CC_SWITCH_MASTER_PASSWORD="your-password"

# 配置目录
export CC_SWITCH_CONFIG_DIR="/custom/path/.cc-switch"

# 日志级别
export CC_SWITCH_LOG_LEVEL="debug"

# 缓存策略
export CC_SWITCH_CACHE_STRATEGY="memory"

# 代理配置
export CC_SWITCH_PROXY="http://proxy.example.com:8080"

# 超时设置
export CC_SWITCH_TIMEOUT="30000"
```

### 3.4 卸载

#### 3.4.1 卸载 npm 包

```bash
npm uninstall -g cc-switch
```

#### 3.4.2 清理配置文件

```bash
# macOS/Linux
rm -rf ~/.cc-switch

# Windows (PowerShell)
Remove-Item -Recurse -Force "$env:USERPROFILE\.cc-switch"
```

#### 3.4.3 清理凭证

从系统密钥链中移除凭证：

```bash
# macOS
security delete-generic-password -s "cc-switch"

# Linux (需要 keychain 工具)
keychain --clear cc-switch

# Windows
cmdkey /delete:cc-switch
```

### 3.5 升级指南

#### 3.5.1 检查更新

```bash
cc-switch update --check
```

输出示例：
```
Current version: 1.0.0
Latest version: 1.1.0
Update available: https://github.com/cloudteam/cc-switch/releases/v1.1.0
```

#### 3.5.2 自动升级

```bash
cc-switch update
```

#### 3.5.3 手动升级

```bash
npm update -g cc-switch
```

#### 3.5.4 迁移配置

升级后可能需要迁移配置：

```bash
cc-switch migrate
```

迁移向导会检测配置文件版本并自动迁移到新格式。

---

## 4. 快速开始

### 4.1 五分钟入门教程

本教程将帮助您在五分钟内掌握 cc-switch 的基本使用。

#### 步骤 1: 初始化配置

```bash
cc-switch init
```

按照提示设置主密码和基本配置。

#### 步骤 2: 添加云配置

以 AWS 为例：

```bash
cc-switch add aws
```

交互式添加向导：
```
? Configuration name: aws-dev
? AWS Access Key ID: AKIAIOSFODNN7EXAMPLE
? AWS Secret Access Key: ********************************
? Default region: us-east-1
? Output format: json

✓ Configuration "aws-dev" added successfully!
```

#### 步骤 3: 查看配置列表

```bash
cc-switch list
```

输出：
```
┌─────────────┬──────────┬─────────────┬────────────┐
│ Name        │ Provider │ Account     │ Region     │
├─────────────┼──────────┼─────────────┼────────────┤
│ aws-dev *   │ AWS      │ 12345678901 │ us-east-1  │
└─────────────┴──────────┴─────────────┴────────────┘

* = active configuration
```

#### 步骤 4: 切换配置

```bash
cc-switch use aws-dev
```

输出：
```
✓ Switched to "aws-dev"
  AWS credentials updated
  Environment variables set
  Ready to use!
```

#### 步骤 5: 验证配置

```bash
aws sts get-caller-identity
```

输出：
```json
{
  "UserId": "AIDAI23456789012345678",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/dev-user"
}
```

恭喜！您已成功完成基础配置。

### 4.2 命令概览

#### 4.2.1 基础命令

| 命令 | 描述 | 示例 |
|-----|------|------|
| `init` | 初始化配置 | `cc-switch init` |
| `add` | 添加配置 | `cc-switch add aws` |
| `list` | 列出配置 | `cc-switch list` |
| `use` | 切换配置 | `cc-switch use aws-dev` |
| `remove` | 删除配置 | `cc-switch remove aws-dev` |
| `show` | 查看详情 | `cc-switch show aws-dev` |
| `edit` | 编辑配置 | `cc-switch edit aws-dev` |

#### 4.2.2 高级命令

| 命令 | 描述 | 示例 |
|-----|------|------|
| `backup` | 备份配置 | `cc-switch backup` |
| `restore` | 恢复配置 | `cc-switch restore backup.tar.gz` |
| `sync` | 同步配置 | `cc-switch sync` |
| `export` | 导出配置 | `cc-switch export --format json` |
| `import` | 导入配置 | `cc-switch import config.json` |
| `validate` | 验证配置 | `cc-switch validate aws-dev` |

#### 4.2.3 管理命令

| 命令 | 描述 | 示例 |
|-----|------|------|
| `config` | 管理设置 | `cc-switch config set cache true` |
| `update` | 更新工具 | `cc-switch update` |
| `doctor` | 诊断问题 | `cc-switch doctor` |
| `completion` | 生成补全 | `cc-switch completion bash` |

### 4.3 配置模板

#### 4.3.1 AWS 配置模板

```yaml
# aws-template.yaml
name: aws-template
provider: aws
credentials:
  type: access_key
  access_key_id: ${AWS_ACCESS_KEY_ID}
  secret_access_key: ${AWS_SECRET_ACCESS_KEY}
  session_token: ${AWS_SESSION_TOKEN}  # 可选
settings:
  region: us-east-1
  output: json
  max_attempts: 3
  retry_mode: standard
```

#### 4.3.2 Azure 配置模板

```yaml
# azure-template.yaml
name: azure-template
provider: azure
credentials:
  type: service_principal
  tenant_id: ${AZURE_TENANT_ID}
  client_id: ${AZURE_CLIENT_ID}
  client_secret: ${AZURE_CLIENT_SECRET}
settings:
  subscription: ${AZURE_SUBSCRIPTION_ID}
  cloud: AzureCloud
  management_endpoint: https://management.azure.com/
```

#### 4.3.3 GCP 配置模板

```yaml
# gcp-template.yaml
name: gcp-template
provider: gcp
credentials:
  type: service_account
  project_id: ${GCP_PROJECT_ID}
  private_key_id: ${GCP_PRIVATE_KEY_ID}
  private_key: ${GCP_PRIVATE_KEY}
  client_email: ${GCP_CLIENT_EMAIL}
  client_id: ${GCP_CLIENT_ID}
settings:
  region: us-central1
  zone: us-central1-a
```

### 4.4 环境变量

#### 4.4.1 AWS 环境变量

切换到 AWS 配置后，以下环境变量将被设置：

```bash
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
AWS_SESSION_TOKEN=optional-token  # 如果使用临时凭证
AWS_DEFAULT_REGION=us-east-1
AWS_REGION=us-east-1
AWS_PROFILE=dev
AWS_DEFAULT_OUTPUT=json
AWS_CA_BUNDLE=/path/to/certificate.pem
AWS_SHARED_CREDENTIALS_FILE=/custom/path/credentials
AWS_CONFIG_FILE=/custom/path/config
```

#### 4.4.2 Azure 环境变量

```bash
AZURE_TENANT_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
AZURE_CLIENT_ID=yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy
AZURE_CLIENT_SECRET=zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz
AZURE_SUBSCRIPTION_ID=aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa
AZURE_CLOUD_NAME=AzureCloud
ARM_SUBSCRIPTION_ID=aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa
ARM_TENANT_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
ARM_CLIENT_ID=yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy
ARM_CLIENT_SECRET=zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz
```

#### 4.4.3 GCP 环境变量

```bash
GOOGLE_CLOUD_PROJECT=my-project-id
GCLOUD_PROJECT=my-project-id
GCP_PROJECT=my-project-id
GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
CLOUDSDK_CORE_PROJECT=my-project-id
CLOUDSDK_COMPUTE_REGION=us-central1
CLOUDSDK_COMPUTE_ZONE=us-central1-a
```

---

## 5. 基础功能

### 5.1 配置管理

#### 5.1.1 添加配置

**交互式添加**

最简单的方式是使用交互式向导：

```bash
cc-switch add aws
```

系统会提示您输入必要的配置信息：

```
? Configuration name: aws-production
? AWS Access Key ID: AKIAIOSFODNN7EXAMPLE
? AWS Secret Access Key: ********************************
? Default region: us-west-2
? Output format: (Use arrow keys)
  ❯ json
    text
    table
? Enable MFA? (y/N): n
? Add tags? (y/N): y
? Tags (comma-separated): production,critical
```

**命令行参数添加**

对于脚本和自动化，可以使用命令行参数：

```bash
cc-switch add aws \
  --name aws-production \
  --access-key AKIAIOSFODNN7EXAMPLE \
  --secret-key wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY \
  --region us-west-2 \
  --output json \
  --tags production,critical
```

**从文件导入**

从 JSON 或 YAML 文件导入配置：

```bash
# 从 JSON 文件导入
cc-switch add aws --from-file aws-config.json

# 从 YAML 文件导入
cc-switch add azure --from-file azure-config.yaml
```

配置文件示例：
```json
{
  "name": "aws-production",
  "provider": "aws",
  "credentials": {
    "type": "access_key",
    "access_key_id": "AKIAIOSFODNN7EXAMPLE",
    "secret_access_key": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
  },
  "settings": {
    "region": "us-west-2",
    "output": "json"
  },
  "tags": ["production", "critical"]
}
```

**从现有配置复制**

从已有的云配置复制：

```bash
# 从 AWS CLI 配置复制
cc-switch add aws --from-cli-profile default

# 从 Azure CLI 配置复制
cc-switch add azure --from-cli-profile production

# 从环境变量导入
cc-switch add aws --from-env
```

#### 5.1.2 列出配置

**基本列表**

```bash
cc-switch list
```

输出：
```
┌──────────────────┬──────────┬────────────────┬────────────┬──────────┐
│ Name             │ Provider │ Account        │ Region     │ Tags     │
├──────────────────┼──────────┼────────────────┼────────────┼──────────┤
│ aws-dev *        │ AWS      │ 123456789012   │ us-east-1  │ dev      │
│ aws-staging      │ AWS      │ 234567890123   │ us-east-1  │ staging  │
│ aws-production   │ AWS      │ 345678901234   │ us-west-2  │ prod     │
│ azure-dev        │ Azure    │ prod-tenant    │ eastus     │ dev      │
│ gcp-production   │ GCP      │ my-project-id  │ us-central1│ prod     │
└──────────────────┴──────────┴────────────────┴────────────┴──────────┘

* = active configuration
```

**过滤列表**

按提供商过滤：
```bash
cc-switch list --provider aws
```

按标签过滤：
```bash
cc-switch list --tag production
```

按区域过滤：
```bash
cc-switch list --region us-east-1
```

**输出格式**

JSON 格式：
```bash
cc-switch list --format json
```

输出：
```json
[
  {
    "name": "aws-dev",
    "provider": "aws",
    "account": "123456789012",
    "region": "us-east-1",
    "tags": ["dev"],
    "active": true
  },
  {
    "name": "aws-staging",
    "provider": "aws",
    "account": "234567890123",
    "region": "us-east-1",
    "tags": ["staging"],
    "active": false
  }
]
```

YAML 格式：
```bash
cc-switch list --format yaml
```

表格格式（可自定义）：
```bash
cc-switch list --format table --columns name,provider,region
```

#### 5.1.3 查看配置详情

**基本查看**

```bash
cc-switch show aws-production
```

输出：
```
Configuration: aws-production
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Provider:         AWS
Account ID:       345678901234
Region:           us-west-2
Output Format:    json
Active:           No
Created:          2026-05-15 10:30:00
Last Used:        2026-05-28 14:22:31
Tags:             production, critical

Credentials:
  Type:           Access Key
  Access Key ID:  AKIA***EXAMPLE
  Secret Key:     *******************KEY
  Last Rotated:   2026-05-01

Settings:
  Max Attempts:   3
  Retry Mode:     standard
  Timeout:        30s

Metadata:
  Version:        2
  Checksum:       abc123def456
  Size:           1.2 KB
```

**查看敏感信息**

需要验证主密码才能查看完整凭证：

```bash
cc-switch show aws-production --reveal
```

```
? Enter master password: ********

Configuration: aws-production
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Credentials:
  Access Key ID:  AKIAIOSFODNN7EXAMPLE
  Secret Key:     wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

**导出配置**

导出为文件：
```bash
# 导出单个配置
cc-switch show aws-production --export aws-prod.yaml

# 导出所有配置
cc-switch export --all --output configs/
```

#### 5.1.4 编辑配置

**交互式编辑**

```bash
cc-switch edit aws-production
```

编辑界面：
```
? What would you like to edit?
  ❯ Credentials
    Region
    Output format
    Tags
    Advanced settings
    Cancel

# 选择 Credentials
? Update Access Key ID? (current: AKIA***EXAMPLE) (y/N): y
? New Access Key ID: AKIANEWKEY123456
? Update Secret Access Key? (y/N): y
? New Secret Access Key: ********************************
? Test new credentials? (Y/n): y

✓ Credentials updated and validated successfully!
```

**命令行编辑**

```bash
# 更新区域
cc-switch edit aws-production --region us-east-1

# 更新标签
cc-switch edit aws-production --tags production,critical,pci

# 更新凭证
cc-switch edit aws-production \
  --access-key AKIANEWKEY123456 \
  --secret-key newSecretKey123456
```

**批量编辑**

```bash
# 批量更新所有生产环境的区域
cc-switch edit --tag production --region us-west-2

# 批量添加标签
cc-switch edit --provider aws --add-tag migrated
```

#### 5.1.5 删除配置

**单个删除**

```bash
cc-switch remove aws-old-config
```

确认提示：
```
? Are you sure you want to remove "aws-old-config"? (y/N): y
? This configuration is used by 3 CI pipelines. Continue? (y/N): y

✓ Configuration "aws-old-config" removed successfully
  Backup saved to: ~/.cc-switch/backups/aws-old-config-20260529.tar.gz
```

**批量删除**

```bash
# 删除所有带 "old" 标签的配置
cc-switch remove --tag old

# 删除所有开发环境配置
cc-switch remove --tag dev --force
```

**清理未使用配置**

```bash
# 删除超过 90 天未使用的配置
cc-switch cleanup --unused-days 90
```

### 5.2 配置切换

#### 5.2.1 基本切换

**切换到指定配置**

```bash
cc-switch use aws-production
```

输出：
```
✓ Switched to "aws-production"
  ✓ AWS credentials file updated
  ✓ Environment variables set
  ✓ Shell profile updated
  ✓ Connection verified

Current context:
  Account: 345678901234
  Region: us-west-2
  User: production-user
```

**快速切换**

使用配置索引快速切换：
```bash
cc-switch list --short
# 1. aws-dev
# 2. aws-staging
# 3. aws-production

cc-switch use 3  # 切换到 aws-production
```

**切换到上一个配置**

```bash
cc-switch use -
```

#### 5.2.2 临时切换

**临时会话**

临时切换配置，不影响全局状态：

```bash
# 在子 shell 中使用临时配置
cc-switch run aws-production -- aws s3 ls

# 或者使用环境变量
cc-switch use aws-production --temp
# 当前 shell 会话使用该配置
# 关闭终端后恢复原配置
```

**一次性执行**

```bash
cc-switch exec aws-production -- terraform plan
```

#### 5.2.3 切换验证

**自动验证**

切换时自动验证凭证有效性：

```bash
cc-switch use aws-production --verify
```

**跳过验证**

加快切换速度：

```bash
cc-switch use aws-production --no-verify
```

**验证选项**

```bash
# 验证特定权限
cc-switch use aws-production --verify-permissions s3:ListBucket,ec2:DescribeInstances

# 验证区域可用性
cc-switch use aws-production --verify-region
```

#### 5.2.4 切换钩子

**前置钩子**

切换前执行脚本：

```bash
cc-switch use aws-production --before "/scripts/notify.sh switching to production"
```

**后置钩子**

切换后执行脚本：

```bash
cc-switch use aws-production --after "/scripts/verify.sh"
```

**配置钩子**

在配置文件中定义钩子：

```yaml
# ~/.cc-switch/profiles/aws-production.yaml
hooks:
  before_switch:
    - script: /scripts/check-mfa.sh
      required: true
  after_switch:
    - script: /scripts/update-ssh-config.sh
      async: true
  on_error:
    - script: /scripts/notify-error.sh
      async: true
```

### 5.3 配置组

#### 5.3.1 创建配置组

配置组允许同时管理多个配置：

```bash
cc-switch group create full-stack \
  --configs aws-production,azure-production,gcp-production
```

输出：
```
✓ Group "full-stack" created
  Members:
    - aws-production (AWS)
    - azure-production (Azure)
    - gcp-production (GCP)
```

#### 5.3.2 管理配置组

**列出配置组**

```bash
cc-switch group list
```

输出：
```
┌──────────────┬──────────────────────────────────────┬───────┐
│ Group Name   │ Configurations                       │ Count │
├──────────────┼──────────────────────────────────────┼───────┤
│ full-stack   │ aws-prod, azure-prod, gcp-prod       │ 3     │
│ dev-all      │ aws-dev, azure-dev, gcp-dev          │ 3     │
│ aws-only     │ aws-dev, aws-staging, aws-prod       │ 3     │
└──────────────┴──────────────────────────────────────┴───────┘
```

**添加成员**

```bash
cc-switch group add full-stack --config alibaba-production
```

**移除成员**

```bash
cc-switch group remove full-stack --config alibaba-production
```

**删除配置组**

```bash
cc-switch group delete full-stack
```

#### 5.3.3 使用配置组

**激活配置组**

```bash
cc-switch use-group full-stack
```

输出：
```
✓ Activated group "full-stack"
  ✓ aws-production - AWS credentials set
  ✓ azure-production - Azure credentials set
  ✓ gcp-production - GCP credentials set

Environment variables ready for multi-cloud operations
```

**对配置组执行命令**

```bash
# 验证所有配置
cc-switch group exec full-stack -- verify

# 列出所有配置的状态
cc-switch group exec full-stack -- status

# 更新所有配置的区域
cc-switch group exec full-stack -- edit --region us-west-2
```

### 5.4 配置搜索

#### 5.4.1 基本搜索

**按名称搜索**

```bash
cc-switch search production
```

输出：
```
Found 3 configurations matching "production":

1. aws-production
   Provider: AWS
   Account: 345678901234
   Region: us-west-2
   Tags: production, critical

2. azure-production
   Provider: Azure
   Tenant: prod-tenant-id
   Region: eastus
   Tags: production

3. gcp-production
   Provider: GCP
   Project: prod-project-id
   Region: us-central1
   Tags: production
```

**高级搜索**

```bash
# 组合条件搜索
cc-switch search \
  --provider aws \
  --region us-west-2 \
  --tag production

# 正则表达式搜索
cc-switch search --regex "^aws-.*-prod$"

# 模糊搜索
cc-switch search --fuzzy "producshun" --threshold 0.7
```

#### 5.4.2 搜索历史

**查看使用历史**

```bash
cc-switch history
```

输出：
```
Recent Configuration Usage:

1. 2026-05-29 14:30:22 - aws-production (switched from aws-staging)
2. 2026-05-29 10:15:33 - aws-staging (switched from aws-dev)
3. 2026-05-28 16:45:12 - azure-production (switched from azure-dev)
4. 2026-05-28 09:22:41 - aws-dev (initial)
```

**搜索历史**

```bash
# 查找过去一周的使用记录
cc-switch history --since "1 week ago"

# 查找特定配置的使用记录
cc-switch history aws-production --last 30
```

### 5.5 配置备份与恢复

#### 5.5.1 备份配置

**手动备份**

```bash
# 备份所有配置
cc-switch backup

# 备份单个配置
cc-switch backup aws-production

# 备份到指定位置
cc-switch backup --output /backups/cc-switch-$(date +%Y%m%d).tar.gz
```

输出：
```
✓ Backup created successfully
  Location: ~/.cc-switch/backups/backup-20260529-143022.tar.gz
  Size: 15.3 KB
  Configurations: 12
  Includes:
    - Configuration files
    - Encrypted credentials
    - Metadata
    - Audit logs (last 7 days)
```

**自动备份**

配置自动备份策略：

```bash
# 启用每日自动备份
cc-switch config set backup.enabled true
cc-switch config set backup.schedule daily
cc-switch config set backup.retention 30  # 保留 30 天
```

**增量备份**

```bash
cc-switch backup --incremental
```

#### 5.5.2 恢复配置

**从备份恢复**

```bash
# 列出可用备份
cc-switch backup list

# 从备份恢复
cc-switch restore ~/.cc-switch/backups/backup-20260529-143022.tar.gz
```

输出：
```
? Backup contains 12 configurations. Restore options:
  ❯ Restore all
    Restore selected
    Preview only
    Cancel

? Restore mode:
  ❯ Merge (keep existing, add new)
    Replace (overwrite existing)
    Cancel

✓ Restored 12 configurations
  - 10 configurations merged
  - 2 configurations added
  - 0 configurations skipped
```

**选择性恢复**

```bash
# 只恢复特定配置
cc-switch restore backup.tar.gz --configs aws-production,azure-production

# 恢复前预览
cc-switch restore backup.tar.gz --dry-run
```

#### 5.5.3 灾难恢复

**完整恢复**

在系统崩溃后恢复所有配置：

```bash
# 从远程备份恢复
cc-switch restore --from-remote s3://my-backups/cc-switch/

# 从加密备份恢复
cc-switch restore backup.tar.gz.enc --password
```

**配置验证**

恢复后验证配置完整性：

```bash
cc-switch verify --all
```

---

## 6. 配置管理

### 6.1 配置文件详解

#### 6.1.1 主配置文件结构

主配置文件 `~/.cc-switch/config.yaml` 包含全局设置和元数据：

```yaml
# cc-switch 主配置文件
# 版本信息
version: "1.0"
config_version: 3

# 当前激活的配置
active_profile: aws-production
previous_profile: aws-staging

# 全局设置
settings:
  # 缓存配置
  cache:
    enabled: true
    ttl: 3600  # 秒
    max_size: 100  # MB
    strategy: lru  # lru | lfu | fifo
  
  # 日志配置
  logging:
    level: info  # debug | info | warn | error
    file: ~/.cc-switch/logs/cc-switch.log
    max_size: 10  # MB
    max_files: 5
    format: json  # json | text
  
  # 备份配置
  backup:
    enabled: true
    schedule: "0 2 * * *"  # Cron 表达式
    retention: 30  # 天
    location: ~/.cc-switch/backups
    include_logs: true
  
  # 安全配置
  security:
    encryption: aes-256-gcm
    key_derivation: pbkdf2
    iterations: 100000
    session_timeout: 3600  # 秒
    max_login_attempts: 5
    lockout_duration: 900  # 秒
  
  # UI 配置
  ui:
    color: true
    progress: true
    table_style: rounded
    date_format: "YYYY-MM-DD HH:mm:ss"
    timezone: local
  
  # 网络配置
  network:
    proxy: null
    timeout: 30000  # 毫秒
    retries: 3
    retry_delay: 1000  # 毫秒

# 云服务提供商配置
providers:
  aws:
    # AWS 特定设置
    default_region: us-east-1
    default_output: json
    max_attempts: 3
    retry_mode: adaptive
    
  azure:
    # Azure 特定设置
    default_cloud: AzureCloud
    management_endpoint: https://management.azure.com/
    
  gcp:
    # GCP 特定设置
    default_region: us-central1
    default_zone: us-central1-a

# 插件配置
plugins:
  enabled: true
  directory: ~/.cc-switch/plugins
  auto_update: false
  allowed_plugins:
    - cc-switch-plugin-aws-extended
    - cc-switch-plugin-azure-extended

# 钩子配置
hooks:
  before_switch:
    - name: check-mfa
      script: ~/.cc-switch/hooks/check-mfa.sh
      required: true
      timeout: 10
      
  after_switch:
    - name: notify
      script: ~/.cc-switch/hooks/notify.sh
      async: true
      
  on_error:
    - name: error-handler
      script: ~/.cc-switch/hooks/error.sh
      notify: admin@example.com

# 别名配置
aliases:
  prod: aws-production
  staging: aws-staging
  dev: aws-dev
  
# 元数据
metadata:
  created_at: "2026-01-15T10:00:00Z"
  updated_at: "2026-05-29T14:30:22Z"
  created_by: john.doe@example.com
  machine_id: abc123
```

#### 6.1.2 配置文件模板

每个云服务配置都有独立的模板文件：

**AWS 配置模板**

```yaml
# ~/.cc-switch/profiles/aws-production.yaml
apiVersion: cc-switch/v1
kind: Profile
metadata:
  name: aws-production
  provider: aws
  labels:
    environment: production
    critical: "true"
    compliance: pci-dss
  annotations:
    owner: cloud-team@example.com
    cost-center: CC-12345

spec:
  # 凭证配置
  credentials:
    type: access_key  # access_key | assume_role | sso | instance_profile
    access_key_id: ${ENCRYPTED:AES256:U2FsdGVkX1...}
    secret_access_key: ${ENCRYPTED:AES256:U2FsdGVkX2...}
    
    # 可选：MFA 配置
    mfa:
      enabled: true
      serial: arn:aws:iam::345678901234:mfa/production-user
      duration: 3600
      
    # 可选：Assume Role 配置
    assume_role:
      role_arn: arn:aws:iam::345678901234:role/ProductionRole
      external_id: unique-external-id
      session_name: cc-switch-session
      duration: 3600

  # AWS 区域配置
  region: us-west-2
  output: json
  
  # 高级设置
  settings:
    s3:
      max_concurrent_requests: 10
      max_queue_size: 1000
      multipart_threshold: 8MB
      multipart_chunksize: 8MB
      
    ec2:
      max_retries: 5
      
    cloudwatch:
      endpoint: https://monitoring.us-west-2.amazonaws.com
      
  # 环境变量
  env:
    AWS_MAX_ATTEMPTS: "5"
    AWS_RETRY_MODE: "adaptive"
    AWS_STS_REGIONAL_ENDPOINTS: "regional"

  # 钩子
  hooks:
    before_switch:
      - name: validate-mfa
        script: ~/.cc-switch/hooks/validate-mfa.sh
        required: true
    after_switch:
      - name: update-ssh
        script: ~/.cc-switch/hooks/update-ssh-config.sh
        async: true

status:
  last_used: "2026-05-29T14:30:22Z"
  validation:
    last_check: "2026-05-29T14:30:22Z"
    status: valid
    permissions:
      - s3:ListBucket
      - ec2:DescribeInstances
  usage_stats:
    switch_count: 156
    last_week_active: true
```

**Azure 配置模板**

```yaml
# ~/.cc-switch/profiles/azure-production.yaml
apiVersion: cc-switch/v1
kind: Profile
metadata:
  name: azure-production
  provider: azure
  labels:
    environment: production
    tenant: prod-tenant

spec:
  # 凭证配置
  credentials:
    type: service_principal  # service_principal | user_principal | managed_identity
    tenant_id: ${ENCRYPTED:AES256:U2FsdGVkX3...}
    client_id: ${ENCRYPTED:AES256:U2FsdGVkX4...}
    client_secret: ${ENCRYPTED:AES256:U2FsdGVkX5...}
    
    # 可选：证书认证
    certificate:
      path: ~/.cc-switch/certs/azure-prod.pem
      password: ${ENCRYPTED:AES256:U2FsdGVkX6...}

  # Azure 配置
  subscription: aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa
  cloud: AzureCloud  # AzureCloud | AzureChinaCloud | AzureUSGovernment
  
  # 区域和资源组
  default_location: eastus
  default_resource_group: prod-resources
  
  # 服务端点
  endpoints:
    management: https://management.azure.com/
    active_directory: https://login.microsoftonline.com/
    
  # 环境变量
  env:
    ARM_SUBSCRIPTION_ID: aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa
    ARM_TENANT_ID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    ARM_CLIENT_ID: yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy

status:
  last_used: "2026-05-29T14:30:22Z"
  validation:
    status: valid
```

#### 6.1.3 配置继承

支持配置继承，减少重复配置：

```yaml
# 基础配置 base-aws.yaml
apiVersion: cc-switch/v1
kind: BaseProfile
metadata:
  name: base-aws

spec:
  settings:
    max_attempts: 3
    retry_mode: standard
    output: json
    
  env:
    AWS_MAX_ATTEMPTS: "3"
    AWS_RETRY_MODE: "standard"

---
# 继承基础配置
apiVersion: cc-switch/v1
kind: Profile
metadata:
  name: aws-production
  inherits: base-aws

spec:
  credentials:
    type: access_key
    access_key_id: AKIAIOSFODNN7EXAMPLE
    secret_access_key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    
  region: us-west-2
  
  # 覆盖基础配置
  settings:
    max_attempts: 5  # 覆盖
```

### 6.2 配置验证

#### 6.2.1 自动验证

配置保存时自动验证：

```bash
cc-switch add aws --validate
```

验证项目：
- ✅ 凭证格式正确性
- ✅ 区域可用性
- ✅ API 连接性
- ✅ 基本权限
- ✅ 必要字段完整性

#### 6.2.2 手动验证

**验证单个配置**

```bash
cc-switch validate aws-production
```

输出：
```
Validating "aws-production"...

✅ Configuration format: Valid
✅ Credentials: Valid
✅ Region: Valid (us-west-2)
✅ API Connectivity: Connected
✅ Basic Permissions: Granted
  - sts:GetCallerIdentity ✓
  - s3:ListAllMyBuckets ✓
  - ec2:DescribeRegions ✓

⚠️ Warnings:
  - MFA not enabled (recommended for production)
  
ℹ️ Info:
  - Account type: Standard
  - Account ID: 345678901234
  - IAM User: production-user
  - Last credential rotation: 28 days ago

Validation completed with 0 errors, 1 warning
```

**验证所有配置**

```bash
cc-switch validate --all
```

输出：
```
Validating all configurations...

aws-dev: ✅ Valid
aws-staging: ✅ Valid
aws-production: ⚠️ Valid (with warnings)
  - Warning: MFA not enabled
azure-dev: ✅ Valid
azure-production: ❌ Invalid
  - Error: Credentials expired
gcp-production: ✅ Valid

Summary: 4 valid, 1 with warnings, 1 invalid
```

**深度验证**

验证特定权限和资源访问：

```bash
cc-switch validate aws-production --deep
```

输出：
```
Deep validation for "aws-production"...

✅ Identity & Access Management (IAM)
  - iam:GetUser ✓
  - iam:ListAttachedUserPolicies ✓
  - iam:ListUserTags ✓
  
✅ Simple Storage Service (S3)
  - s3:ListBucket ✓ (12 buckets accessible)
  - s3:GetObject ✓
  - s3:PutObject ✓
  
⚠️ Elastic Compute Cloud (EC2)
  - ec2:DescribeInstances ✓
  - ec2:RunInstances ❌ (Action not allowed)
  - ec2:TerminateInstances ❌ (Action not allowed)
  
❌ AWS Key Management Service (KMS)
  - kms:ListKeys ❌ (Access denied)
  - kms:DescribeKey ❌ (Access denied)

Deep validation completed
  Granted: 8 permissions
  Denied: 4 permissions
  Access Level: Read-Only
```

#### 6.2.3 验证策略

定义验证策略：

```yaml
# ~/.cc-switch/validation-policy.yaml
policies:
  production:
    required_checks:
      - credentials_valid
      - mfa_enabled
      - permissions_sufficient
    required_permissions:
      aws:
        - sts:GetCallerIdentity
        - s3:ListBucket
    forbidden_permissions:
      aws:
        - iam:DeleteUser
        - organizations:DeleteAccount
    settings:
      max_credential_age_days: 90
      require_mfa: true
      
  development:
    required_checks:
      - credentials_valid
    settings:
      max_credential_age_days: 365
      require_mfa: false
```

应用验证策略：

```bash
# 验证配置是否符合生产策略
cc-switch validate aws-production --policy production
```

### 6.3 配置同步

#### 6.3.1 本地同步

在多个终端会话间同步配置：

```bash
# 启用同步
cc-switch sync enable

# 手动同步
cc-switch sync now

# 查看同步状态
cc-switch sync status
```

输出：
```
Synchronization Status:
  Enabled: Yes
  Last Sync: 2 minutes ago
  Status: Synchronized
  
Active Sessions:
  - Session 1 (tty001): aws-production (current)
  - Session 2 (tty002): aws-staging
  - Session 3 (pts/0): aws-dev
  
Lock Status: No locks held
```

#### 6.3.2 团队同步

通过共享存储同步团队配置：

```bash
# 配置共享存储
cc-switch sync configure \
  --backend s3 \
  --bucket my-team-cc-switch \
  --region us-east-1 \
  --prefix configs/

# 推送本地配置到共享存储
cc-switch sync push

# 从共享存储拉取配置
cc-switch sync pull

# 查看远程配置
cc-switch sync list-remote
```

**冲突解决**

```bash
# 拉取时遇到冲突
cc-switch sync pull
```

输出：
```
Conflict detected for "aws-production":
  Local version: Modified 2 hours ago (version 5)
  Remote version: Modified 1 hour ago (version 6)

? How would you like to resolve:
  ❯ Keep remote (overwrite local)
    Keep local (overwrite remote)
    Merge (manual)
    Skip this configuration
```

#### 6.3.3 Git 同步

使用 Git 仓库同步配置：

```bash
# 初始化 Git 同步
cc-switch sync git init \
  --repo git@github.com:myteam/cc-switch-configs.git \
  --branch main \
  --gpg-sign

# 提交本地更改
cc-switch sync git commit -m "Add production configs"

# 推送到远程
cc-switch sync git push

# 拉取远程更改
cc-switch sync git pull
```

**自动提交**

```bash
# 配置自动提交
cc-switch config set sync.git.auto_commit true
cc-switch config set sync.git.commit_message "Auto-sync: {action} {profile}"
```

### 6.4 配置导入导出

#### 6.4.1 导出配置

**导出所有配置**

```bash
# 导出为 YAML
cc-switch export --all --format yaml --output ./configs/

# 导出为 JSON
cc-switch export --all --format json --output configs.json

# 导出为 ZIP
cc-switch export --all --format zip --output cc-switch-configs.zip
```

**选择性导出**

```bash
# 导出特定配置
cc-switch export aws-production azure-production --output ./prod-configs/

# 按标签导出
cc-switch export --tag production --output ./prod-configs/

# 按提供商导出
cc-switch export --provider aws --output ./aws-configs/
```

**导出选项**

```bash
# 包含敏感信息（需要密码）
cc-switch export --all --include-secrets --output secrets-backup.enc

# 包含历史记录
cc-switch export --all --include-history --output full-backup.tar.gz

# 包含审计日志
cc-switch export --all --include-audit-logs --output compliance-export.tar.gz
```

#### 6.4.2 导入配置

**从文件导入**

```bash
# 导入单个配置文件
cc-switch import aws-new.yaml

# 导入多个配置
cc-switch import configs/*.yaml

# 从 ZIP 导入
cc-switch import cc-switch-configs.zip
```

**导入选项**

```bash
# 跳过已存在的配置
cc-switch import configs/*.yaml --skip-existing

# 覆盖已存在的配置
cc-switch import configs/*.yaml --overwrite

# 合并配置
cc-switch import configs/*.yaml --merge

# 预览导入
cc-switch import configs/*.yaml --dry-run
```

**从其他工具导入**

```bash
# 从 AWS CLI 配置导入
cc-switch import --from-aws-cli

# 从 Azure CLI 配置导入
cc-switch import --from-azure-cli

# 从环境变量导入
cc-switch import --from-env --name aws-from-env
```

#### 6.4.3 迁移助手

从其他配置管理工具迁移：

```bash
# 从 aws-vault 迁移
cc-switch migrate --from aws-vault

# 从 aws-profiles 迁移
cc-switch migrate --from aws-profiles

# 从自定义格式迁移
cc-switch migrate --from custom --template migration-template.yaml
```

迁移模板示例：
```yaml
# migration-template.yaml
source:
  type: custom
  path: /path/to/old/configs
  format: json
  
mapping:
  - source_field: aws_access_key
    target_field: credentials.access_key_id
    
  - source_field: aws_secret_key
    target_field: credentials.secret_access_key
    
  - source_field: region_name
    target_field: region
    
transformations:
  - field: name
    pattern: "aws-{env}"
    variables:
      env: source.environment
```