# Backend Provider Callback 与 Observability 配置手册

## Scope

- 适用仓库: nursing-backend-services
- 适用对象: backend 平台团队、DevOps、SRE、供应商接入同学
- 变更范围: Notification provider callback 签名 profile、Kubernetes ConfigMap/Secret、OpenTelemetry Collector 上游出口配置
- 发布阶段: 生产可运维性收尾阶段，当前目标是让真实供应商接入和监控接线通过配置完成，而不是继续改代码
- 回滚方式: 回退本页引用的 ConfigMap/Secret 与 Notification 配置改动，恢复上一版 profile 定义

## 适用场景

本页用于两类操作：

- 给 Notification 接入真实短信、邮件或站外消息供应商 webhook 回调
- 给集群内置 OpenTelemetry Collector 配置真实上游观测平台出口

如果当前环境仍只做本地联调，可以仅修改 appsettings。只要进入 Kubernetes 环境，就应优先以 ConfigMap 和 Secret 为准。

## 配置入口

### Notification 本地与镜像默认值

- backend 源配置: `src/Services/NursingBackend.Services.Notification/appsettings.json`
- development 样例: `src/Services/NursingBackend.Services.Notification/appsettings.Development.json`

### Kubernetes 运行时覆盖

- 非敏感配置: `deploy/k8s/base/configmap.yaml`
- 敏感配置: `deploy/k8s/base/secret.yaml`

### 观测链路

- Collector pipeline: `deploy/k8s/base/otel-collector-config.yaml`
- Collector workload: `deploy/k8s/base/otel-collector.yaml`
- 告警规则: `deploy/k8s/base/alerts.yaml`

## Provider Callback Profile 模型

每个 provider profile 由以下字段组成：

- `Provider`: 回调体内的 provider 名称匹配键
- `SignatureMode`: 当前支持 `HmacSha256`
- `SignatureEncoding`: 支持 `HexLower`、`HexUpper`、`Base64`
- `SignaturePayloadMode`: 支持 `TimestampDotBody`、`TimestampBody`、`Body`
- `SignaturePrefix`: 可选，用于剥离 `sha256=` 之类的前缀
- `SignatureHeaderName`: 签名请求头
- `TimestampHeaderName`: 时间戳请求头
- `TimestampToleranceSeconds`: 签名时间窗；`Body` 模式可设为 `0`
- `SignatureSecret`: HMAC secret
- `ChannelOverride`: 把回调统一映射到内部 channel
- `StatusMappings`: 将供应商原始状态映射为内部 `Queued`、`Delivered`、`Failed`

## 当前仓库内置模板

### 模板 1: timestamp + hex

适用于很多“时间戳 + body 拼接后做 HMAC，再以 hex 传输”的 webhook。

```json
{
  "Provider": "twilio",
  "SignatureMode": "HmacSha256",
  "SignatureEncoding": "HexLower",
  "SignaturePayloadMode": "TimestampDotBody",
  "SignatureHeaderName": "X-Twilio-Signature",
  "TimestampHeaderName": "X-Twilio-Timestamp",
  "TimestampToleranceSeconds": 300,
  "ChannelOverride": "sms"
}
```

### 模板 2: body-only + base64 + prefix

适用于很多“直接对原始 body 做 HMAC，以 base64 传输，并带 `sha256=` 前缀”的 webhook。

```json
{
  "Provider": "email-hmac-base64",
  "SignatureMode": "HmacSha256",
  "SignatureEncoding": "Base64",
  "SignaturePayloadMode": "Body",
  "SignaturePrefix": "sha256=",
  "SignatureHeaderName": "X-Provider-Signature",
  "TimestampToleranceSeconds": 0,
  "ChannelOverride": "email"
}
```

说明：

- 上述模板是接入样式模板，不代表仓库已经内建任意第三方厂商的完整官方协议适配
- 如果供应商使用非 HMAC-SHA256 算法，例如非对称签名，本轮代码仍不支持，需要单独扩展

## Kubernetes 变量对照表

### Shared key 与默认 profile

放在 ConfigMap:

- `ProviderCallbacks__SharedKeyHeaderName`
- `ProviderCallbacks__AllowSharedKeyFallback`
- `ProviderCallbacks__DefaultProfile__Provider`
- `ProviderCallbacks__DefaultProfile__SignatureMode`
- `ProviderCallbacks__DefaultProfile__SignatureEncoding`
- `ProviderCallbacks__DefaultProfile__SignaturePayloadMode`
- `ProviderCallbacks__DefaultProfile__SignatureHeaderName`
- `ProviderCallbacks__DefaultProfile__TimestampHeaderName`
- `ProviderCallbacks__DefaultProfile__TimestampToleranceSeconds`

放在 Secret:

- `ProviderCallbacks__SharedKey`
- `ProviderCallbacks__DefaultProfile__SignatureSecret`

### Profile 数组模板

当前仓库已预留两个 profile 下标：

- `ProviderCallbacks__Profiles__0__*`: timestamp + hex 模板
- `ProviderCallbacks__Profiles__1__*`: body-only + base64 + prefix 模板

对应 secret：

- `ProviderCallbacks__Profiles__0__SignatureSecret`
- `ProviderCallbacks__Profiles__1__SignatureSecret`

注意：

- 数组下标必须稳定。如果你调整 appsettings 里的 profile 顺序，就必须同步改 ConfigMap 和 Secret 的 `Profiles__{index}`
- 如果新增更多 provider，继续追加 `Profiles__2__*`、`Profiles__3__*`
- 旧的 `ProviderCallbacks__SignatureSecret` 不再生效，不能继续使用

## 推荐接入步骤

1. 在 appsettings.Development 中先按真实供应商头名、编码、原文模式、prefix 和状态值调通。
2. 把调通后的非敏感字段写入 ConfigMap，把 secret 写入 Secret。
3. 用模拟回调请求验证签名、去重、状态映射和 Billing 补偿链路。
4. 观察 `nursing_notification_provider_signature_failures_total` 与 `nursing_notification_provider_callback_duplicates_total` 是否稳定。

## 回调验收命令示例

### timestamp + hex

```bash
body='{"provider":"twilio","channel":"sms","status":"Delivered","notificationId":"<notificationId>"}'
timestamp=$(date +%s)
signature=$(printf '%s.%s' "$timestamp" "$body" | openssl dgst -sha256 -hmac 'replace-twilio-signature-secret' -binary | xxd -p -c 256)
curl -X POST http://localhost:5144/api/provider-callbacks/notifications \
  -H 'Content-Type: application/json' \
  -H "X-Twilio-Timestamp: $timestamp" \
  -H "X-Twilio-Signature: $signature" \
  -d "$body"
```

### body-only + base64 + prefix

```bash
body='{"provider":"email-hmac-base64","channel":"email","status":"processed","notificationId":"<notificationId>"}'
signature=$(printf '%s' "$body" | openssl dgst -sha256 -hmac 'replace-email-provider-signature-secret' -binary | openssl base64 -A)
curl -X POST http://localhost:5144/api/provider-callbacks/notifications \
  -H 'Content-Type: application/json' \
  -H "X-Provider-Signature: sha256=$signature" \
  -d "$body"
```

## Observability 配置位

ConfigMap:

- `OTEL_UPSTREAM_OTLP_ENDPOINT`
- `OTEL_COLLECTOR_DEBUG_VERBOSITY`
- `Monitoring__DashboardUrl`
- `Monitoring__AlertRouteName`

Secret:

- `OTEL_UPSTREAM_OTLP_AUTHORIZATION`

Collector config 中的 exporter 会读取这些变量，因此切换到真实 Tempo、Grafana Cloud 或其他 OTLP 端点时，不需要再改 collector 模板结构。

## 健康信号

- provider callback 返回 `200`，且重复投递返回 `duplicate=true`
- `nursing_notification_provider_signature_failures_total` 不持续上涨
- `nursing_notification_provider_callback_duplicates_total` 只在重放或供应商重复投递时上涨
- Notification observability summary 中失败数与实际事件匹配
- Billing 补偿仅在被映射到 `Failed` 的状态上触发
- collector Pod 正常运行，且上游 exporter 没有持续报错

## 验证

- backend: `dotnet build nursing-backend-services.slnx`
- tests: `dotnet test nursing-backend-services.slnx`
- docs: `npm run docs:build`
- manifests: `kubectl kustomize deploy/k8s/overlays/dev`

## Rollback

- 回退 `NotificationProviderCallbackOptions` 与 webhook policy 相关代码
- 回退 appsettings、ConfigMap、Secret 中新增的 profile 字段
- 回退本页和索引/侧边栏入口