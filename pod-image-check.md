# Deployment列表
wallet-core
wallet-gateway
wallet-txn
wallet-identity
management-portal
management-gateway
payment-core
payment-notify
payment-gateway
wallet-demo
app-customer
app-gateway
app-h5
app-partner
h5-gateway
oss-gateway
foundation-audit
foundation-kit
foundation-oss
foundation-workflow
biz-notification
biz-task
biz-broker
promotion-core
promotion-gateway
alert-agent

# uat环境镜像
kubectl get deploy -n etb wallet-core wallet-gateway wallet-txn wallet-identity management-portal management-gateway payment-core payment-notify payment-gateway wallet-demo app-customer app-gateway app-h5 app-partner h5-gateway oss-gateway foundation-audit foundation-kit foundation-oss foundation-workflow biz-notification biz-task biz-broker promotion-core promotion-gateway alert-agent -o custom-columns=NAME:.metadata.name,IMAGE:.spec.template.spec.containers[0].imagent-gateway

NAME                  IMAGE
wallet-core           nl-registry.joypaydev.com/appleseed/wallet-core:release-appleseed_20250831_dev_1.221.0-96
wallet-gateway        nl-registry.joypaydev.com/appleseed/wallet-gateway:release-appleseed_20250831_dev_1.191.0-31
wallet-txn            nl-registry.joypaydev.com/appleseed/wallet-txn:release-appleseed_20250831_dev_1.223.0-56
wallet-identity       nl-registry.joypaydev.com/appleseed/wallet-identity:release-appleseed_20250831_dev_1.200.0-52
management-gateway    nl-registry.joypaydev.com/appleseed/management-gateway:release-appleseed_20250831_dev_1.190.0-28
payment-core          nl-registry.joypaydev.com/appleseed/payment-core:release-appleseed_20250831_dev_1.200.0-62
payment-notify        nl-registry.joypaydev.com/appleseed/payment-notify:release-appleseed_20250831_dev_1.190.0-34
payment-gateway       nl-registry.joypaydev.com/appleseed/payment-gateway:release-appleseed_20250831_dev_1.190.0-40
wallet-demo           nl-registry.joypaydev.com/appleseed/wallet-demo:release-appleseed_20250831_dev_1.190.0-24
app-customer          nl-registry.joypaydev.com/appleseed/app-customer:release-appleseed_20250831_dev_1.222.0-75
app-gateway           nl-registry.joypaydev.com/appleseed/app-gateway:release-appleseed_20250831_dev_1.190.0-39
app-h5                nl-registry.joypaydev.com/appleseed/app-h5:release-appleseed_20250831_dev_1.200.0-35
app-partner           nl-registry.joypaydev.com/appleseed/app-partner:release-appleseed_20250831_dev_1.221.0-81
h5-gateway            nl-registry.joypaydev.com/appleseed/h5-gateway:release-appleseed_20250831_dev_1.190.0-23
oss-gateway           nl-registry.joypaydev.com/appleseed/oss-gateway:release-appleseed_20250831_dev_1.200.0-24
foundation-audit      nl-registry.joypaydev.com/appleseed/foundation-audit:release-appleseed_20250831_dev_1.200.0-35
foundation-kit        nl-registry.joypaydev.com/appleseed/foundation-kit:release-appleseed_20250831_dev_1.190.0-29
foundation-oss        nl-registry.joypaydev.com/appleseed/foundation-oss:release-appleseed_20250831_dev_1.200.0-36
foundation-workflow   nl-registry.joypaydev.com/appleseed/foundation-workflow:release-appleseed_20250831_dev_1.190.0-32
biz-notification      nl-registry.joypaydev.com/appleseed/biz-notification:release-appleseed_20250831_dev_1.221.0-30
biz-task              nl-registry.joypaydev.com/appleseed/biz-task:release-appleseed_20250831_dev_1.200.0-41
biz-broker            nl-registry.joypaydev.com/appleseed/biz-broker:release-appleseed_20260131_hotfix_1.0.0-66
promotion-core        nl-registry.joypaydev.com/appleseed/promotion-core:release-appleseed_20250831_dev_1.190.0-37
promotion-gateway     nl-registry.joypaydev.com/appleseed/promotion-gateway:release-appleseed_20250831_dev_1.190.0-28
alert-agent           nl-registry.joypaydev.com/appleseed/alert-agent:release-appleseed_20250831_dev_1.200.0-7
wallet-management-portal   nl-registry.joypaydev.com/appleseed/wallet-management-portal:release-appleseed_20260131_hotfix_1.0.0-88

# test环境镜像

kubectl get deploy -n etb wallet-core wallet-gateway wallet-txn wallet-identity management-portal management-gateway payment-core payment-notify payment-gateway wallet-demo app-customer app-gateway app-h5 app-partner h5-gateway oss-gateway foundation-audit foundation-kit foundation-oss foundation-workflow biz-notification biz-task biz-broker promotion-core promotion-gateway alert-agent -o custom-columns=NAME:.metadata.name,IMAGE:.spec.template.spec.containers[0].image

NAME                  IMAGE
wallet-core           global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/wallet-core:SNAPSHOT-appleseed_20260131_dev_1.5.3-103
wallet-gateway        global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/wallet-gateway:SNAPSHOT-appleseed_20260131_dev_1.1.0-33
wallet-txn            global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/wallet-txn:SNAPSHOT-appleseed_20260131_dev_1.6.1-61
wallet-identity       global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/wallet-identity:SNAPSHOT-appleseed_20260131_dev_1.3.0-55
management-gateway    global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/management-gateway:SNAPSHOT-appleseed_20260131_dev_1.1.0-31
payment-core          global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/payment-core:SNAPSHOT-appleseed_20260131_dev_1.1.0-64
payment-notify        global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/payment-notify:SNAPSHOT-appleseed_20260131_dev_1.1.0-36
payment-gateway       global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/payment-gateway:SNAPSHOT-appleseed_20260131_dev_1.1.0-42
wallet-demo           global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/wallet-demo:SNAPSHOT-appleseed_20260131_dev_1.1.0-26
app-customer          global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/app-customer:SNAPSHOT-appleseed_20260131_dev_1.5.0-79
app-gateway           global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/app-gateway:SNAPSHOT-appleseed_20260131_dev_1.1.0-42
app-h5                global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/app-h5:SNAPSHOT-appleseed_20260131_dev_1.1.0-37
app-partner           global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/app-partner:SNAPSHOT-appleseed_20260131_dev_1.4.0-84
h5-gateway            global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/h5-gateway:SNAPSHOT-appleseed_20260131_dev_1.1.0-25
oss-gateway           global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/oss-gateway:SNAPSHOT-appleseed_20260131_dev_1.1.0-27
foundation-audit      global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/foundation-audit:SNAPSHOT-appleseed_20260131_dev_1.1.0-37
foundation-kit        global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/foundation-kit:SNAPSHOT-appleseed_20260131_dev_1.1.0-31
foundation-oss        global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/foundation-oss:SNAPSHOT-appleseed_20260131_dev_1.1.0-38
foundation-workflow   global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/foundation-workflow:SNAPSHOT-appleseed_20260131_dev_1.3.0-35
biz-notification      global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/biz-notification:SNAPSHOT-appleseed_20260131_dev_1.1.0-32
biz-task              global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/biz-task:SNAPSHOT-appleseed_20260131_dev_1.3.0-44
biz-broker            global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/biz-broker:SNAPSHOT-appleseed_20260131_dev_1.7.0-67
promotion-core        global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/promotion-core:SNAPSHOT-appleseed_20260131_dev_1.1.0-39
promotion-gateway     global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/promotion-gateway:SNAPSHOT-appleseed_20260131_dev_1.1.0-30
alert-agent           global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/alert-agent:SNAPSHOT-appleseed_20260131_dev_1.1.0-9
wallet-management-portal   global-payment-harbor-cn-south-1.jcr.service.jdcloud.com/wallet-management-portal:SNAPSHOT-appleseed_20260131_dev_1.7.0-89

---

# UAT vs Test 环境镜像对比结果

## 对比结论

**所有 26 个服务的 UAT 和 Test 环境镜像均存在差异，无一一致。**

## 逐服务对比表

| 服务名称 | UAT 环境镜像版本 | Test 环境镜像版本 | 是否一致 |
|---|---|---|---|
| wallet-core | `release-appleseed_20250831_dev_1.221.0-96` | `SNAPSHOT-appleseed_20260131_dev_1.5.3-103` | ❌ 不一致 |
| wallet-gateway | `release-appleseed_20250831_dev_1.191.0-31` | `SNAPSHOT-appleseed_20260131_dev_1.1.0-33` | ❌ 不一致 |
| wallet-txn | `release-appleseed_20250831_dev_1.223.0-56` | `SNAPSHOT-appleseed_20260131_dev_1.6.1-61` | ❌ 不一致 |
| wallet-identity | `release-appleseed_20250831_dev_1.200.0-52` | `SNAPSHOT-appleseed_20260131_dev_1.3.0-55` | ❌ 不一致 |
| management-gateway | `release-appleseed_20250831_dev_1.190.0-28` | `SNAPSHOT-appleseed_20260131_dev_1.1.0-31` | ❌ 不一致 |
| payment-core | `release-appleseed_20250831_dev_1.200.0-62` | `SNAPSHOT-appleseed_20260131_dev_1.1.0-64` | ❌ 不一致 |
| payment-notify | `release-appleseed_20250831_dev_1.190.0-34` | `SNAPSHOT-appleseed_20260131_dev_1.1.0-36` | ❌ 不一致 |
| payment-gateway | `release-appleseed_20250831_dev_1.190.0-40` | `SNAPSHOT-appleseed_20260131_dev_1.1.0-42` | ❌ 不一致 |
| wallet-demo | `release-appleseed_20250831_dev_1.190.0-24` | `SNAPSHOT-appleseed_20260131_dev_1.1.0-26` | ❌ 不一致 |
| app-customer | `release-appleseed_20250831_dev_1.222.0-75` | `SNAPSHOT-appleseed_20260131_dev_1.5.0-79` | ❌ 不一致 |
| app-gateway | `release-appleseed_20250831_dev_1.190.0-39` | `SNAPSHOT-appleseed_20260131_dev_1.1.0-42` | ❌ 不一致 |
| app-h5 | `release-appleseed_20250831_dev_1.200.0-35` | `SNAPSHOT-appleseed_20260131_dev_1.1.0-37` | ❌ 不一致 |
| app-partner | `release-appleseed_20250831_dev_1.221.0-81` | `SNAPSHOT-appleseed_20260131_dev_1.4.0-84` | ❌ 不一致 |
| h5-gateway | `release-appleseed_20250831_dev_1.190.0-23` | `SNAPSHOT-appleseed_20260131_dev_1.1.0-25` | ❌ 不一致 |
| oss-gateway | `release-appleseed_20250831_dev_1.200.0-24` | `SNAPSHOT-appleseed_20260131_dev_1.1.0-27` | ❌ 不一致 |
| foundation-audit | `release-appleseed_20250831_dev_1.200.0-35` | `SNAPSHOT-appleseed_20260131_dev_1.1.0-37` | ❌ 不一致 |
| foundation-kit | `release-appleseed_20250831_dev_1.190.0-29` | `SNAPSHOT-appleseed_20260131_dev_1.1.0-31` | ❌ 不一致 |
| foundation-oss | `release-appleseed_20250831_dev_1.200.0-36` | `SNAPSHOT-appleseed_20260131_dev_1.1.0-38` | ❌ 不一致 |
| foundation-workflow | `release-appleseed_20250831_dev_1.190.0-32` | `SNAPSHOT-appleseed_20260131_dev_1.3.0-35` | ❌ 不一致 |
| biz-notification | `release-appleseed_20250831_dev_1.221.0-30` | `SNAPSHOT-appleseed_20260131_dev_1.1.0-32` | ❌ 不一致 |
| biz-task | `release-appleseed_20250831_dev_1.200.0-41` | `SNAPSHOT-appleseed_20260131_dev_1.3.0-44` | ❌ 不一致 |
| biz-broker | `release-appleseed_20260131_hotfix_1.0.0-66` | `SNAPSHOT-appleseed_20260131_dev_1.7.0-67` | ❌ 不一致 |
| promotion-core | `release-appleseed_20250831_dev_1.190.0-37` | `SNAPSHOT-appleseed_20260131_dev_1.1.0-39` | ❌ 不一致 |
| promotion-gateway | `release-appleseed_20250831_dev_1.190.0-28` | `SNAPSHOT-appleseed_20260131_dev_1.1.0-30` | ❌ 不一致 |
| alert-agent | `release-appleseed_20250831_dev_1.200.0-7` | `SNAPSHOT-appleseed_20260131_dev_1.1.0-9` | ❌ 不一致 |
| wallet-management-portal | `release-appleseed_20260131_hotfix_1.0.0-88` | `SNAPSHOT-appleseed_20260131_dev_1.7.0-89` | ❌ 不一致 |

## 核心差异维度

| 差异维度 | UAT 环境 | Test 环境 |
|---|---|---|
| **镜像仓库** | `nl-registry.joypaydev.com` | `global-payment-harbor-cn-south-1.jcr.service.jdcloud.com` |
| **构建日期** | `20250831`（8月31日） | `20260131`（1月31日） |
| **标签前缀** | `release-` | `SNAPSHOT-` |
| **版本号体系** | `1.xxx.0-xx`（如 1.221.0-96） | `1.x.x-xx`（如 1.5.3-103） |

## 总结

两个环境使用了完全不同的镜像仓库、不同的构建日期和不同的版本号体系。UAT 基于 `20250831` 的 `release` 版本构建，Test 基于 `20260131` 的 `SNAPSHOT` 版本构建，属于不同时间点的不同构建产物。