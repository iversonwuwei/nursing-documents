# Admin SaaS 业务模块契约

## Scope

- scope: 定义 admin 前端消费的租户模块 entitlement 契约，说明如何从旧的粗粒度模块兼容到新的业务模块目录。
- compatibility: 前端必须同时兼容旧值和新值，避免 tenant service 升级成为前置阻塞。
- caller impact: tenant service、identity 登录会话、NextAuth JWT/session、admin 前端导航与首页。
- verification: nursing-documents `npm run docs:build`；nursing-admin-v2 `npm run lint`、`npm run build`。
- rollback: entitlement 仍可退回 legacy 值，前端通过兼容映射维持可用。

## 模块字段

### legacy 值

- `admin`
- `family`
- `nani`
- `billing`
- `ai`

### billable 值

- `dashboard`
- `elderly-care`
- `health-device`
- `alert-service`
- `ltci-service`
- `finance-service`
- `notification-service`
- `organization`
- `analytics`
- `ai-assistant`

## 兼容映射

| legacy 值 | billable 模块映射 |
| --- | --- |
| `admin` | `dashboard`、`elderly-care`、`health-device`、`alert-service`、`ltci-service`、`notification-service`、`organization`、`analytics` |
| `billing` | `finance-service` |
| `ai` | `ai-assistant` |
| `family` | 无 admin 一级模块自动扩展 |
| `nani` | 无 admin 一级模块自动扩展 |

## 示例

### tenant descriptor 返回新值

```json
{
  "tenantId": "tenant-demo",
  "tenantName": "演示养老集团",
  "plan": "saas-professional",
  "enabledModules": [
    "dashboard",
    "elderly-care",
    "health-device",
    "alert-service",
    "ltci-service",
    "finance-service",
    "notification-service",
    "organization",
    "analytics",
    "ai-assistant"
  ],
  "enabledFeatures": ["tenant-context", "care-workflow", "module-billing"]
}
```

### tenant descriptor 仍返回旧值

```json
{
  "tenantId": "tenant-legacy",
  "tenantName": "联调租户",
  "plan": "legacy-enterprise",
  "enabledModules": ["admin", "billing", "ai"],
  "enabledFeatures": ["tenant-context"]
}
```

前端应把它解释为：

```json
[
  "dashboard",
  "elderly-care",
  "health-device",
  "alert-service",
  "ltci-service",
  "notification-service",
  "organization",
  "analytics",
  "finance-service",
  "ai-assistant"
]
```

## 演进策略

- 新后端优先直接返回 billable 模块值。
- 前端在迁移期继续保留 legacy -> billable 的解释逻辑。
- 等 tenant service 与合同配置平台都切到新值后，再考虑移除 legacy 映射。
