# Backend + Admin 内容管理设计补全

> 版本：v1.0 | 日期：2026-04-07 | 状态：设计补全文档
> 适用范围：nursing-backend-services + nursing-admin-v2

## 交付 framing

- scope: 为 Admin 后台管理 App 中所有可配置内容（静态文本、静态下拉选项、动态内容）补全 Backend 数据模型、API 端点、权限控制和 Admin 管理界面设计。
- affected audience: 后端研发、Admin 前端研发、Family/Nani App 研发、产品、测试。
- validation: 需与现有 DATABASE_DESIGN.md、backend-service-domain-design.md、MODULE_PAGE_MAPPING.md 保持一致，并通过 `npm run docs:build`。
- rollback: 回退本文及相关索引修改。

---

## 总览

本文档为现有产品/技术设计中**缺失的三类 App 内容管控能力**提供补全设计：

| 内容类型 | 说明 | 管控粒度 |
|---------|------|---------|
| 静态文本 | App 中的固定文案、提示语、错误信息、帮助文本 | key-value，支持多语言 |
| 静态下拉选项 | App 中的枚举值：类别、状态、排序条件、筛选字段等 | 分组 → 选项列表，支持排序/启停 |
| 动态内容 | 由业务流程产生的实体（老人、任务、活动、账单等） | CRUD + 审核/发布/上下架 |

在现有平台架构中，这三类能力归属于以下服务边界：

- **静态文本 + 静态下拉选项** → 新建 `Config Service`（或挂在 Tenant Service 下作为子模块）
- **动态内容** → 已有的 Elder / Care / Health / Operations / Billing 等领域服务

---

## 1. Backend 设计补全

### 1.1 数据模型（数据库表结构）

#### 1.1.1 静态文本管理表

> 设计理念：key-value 表，按 namespace 隔离不同使用场景，原生支持多语言。

```
表名: static_text

字段              类型              约束                        说明
─────────────────────────────────────────────────────────────────────
id                uuid              PK, default gen_random_uuid()
tenant_id         uuid              NOT NULL, FK → tenant(id)    租户隔离
namespace         varchar(64)       NOT NULL                     命名空间（如 app_family, app_nani, admin, common）
text_key          varchar(128)      NOT NULL                     文本标识，如 error.network_timeout
locale            varchar(16)       NOT NULL, default 'zh-CN'    语言代码（BCP 47）
text_value        text              NOT NULL                     实际文案内容
description       varchar(512)                                   用途说明（仅 Admin 可见）
version           integer           NOT NULL, default 1          乐观锁版本号
updated_by        uuid              FK → app_user(id)            最后修改人
created_at        timestamptz       NOT NULL, default now()
updated_at        timestamptz       NOT NULL, default now()

唯一约束: UNIQUE (tenant_id, namespace, text_key, locale)
索引:
  - idx_static_text_tenant_ns: (tenant_id, namespace)
  - idx_static_text_key: (tenant_id, text_key)
触发器: set_updated_at()
```

#### 1.1.2 静态下拉选项管理表

> 设计理念：option_group + option_item 两级结构。分组定义业务含义，选项承载可选值。

```
表名: option_group

字段              类型              约束                        说明
─────────────────────────────────────────────────────────────────────
id                uuid              PK
tenant_id         uuid              NOT NULL, FK → tenant(id)
group_code        varchar(64)       NOT NULL                     分组编码（如 care_level, alert_type, room_type）
group_name        varchar(128)      NOT NULL                     分组显示名称
description       varchar(512)                                   分组说明
is_system         boolean           NOT NULL, default false      是否系统内置（内置分组不可删除）
status            varchar(32)       NOT NULL, default 'active'   active / archived
created_at        timestamptz       NOT NULL, default now()
updated_at        timestamptz       NOT NULL, default now()

唯一约束: UNIQUE (tenant_id, group_code)
索引:
  - idx_option_group_tenant: (tenant_id, status)
触发器: set_updated_at()
```

```
表名: option_item

字段              类型              约束                        说明
─────────────────────────────────────────────────────────────────────
id                uuid              PK
group_id          uuid              NOT NULL, FK → option_group(id) ON DELETE CASCADE
option_code       varchar(64)       NOT NULL                     选项编码（存入业务表的值）
label_zh          varchar(128)      NOT NULL                     中文显示文本
label_en          varchar(128)                                   英文显示文本（可选）
sort_order        integer           NOT NULL, default 0          排序权重，越小越靠前
is_active         boolean           NOT NULL, default true       是否启用
is_default        boolean           NOT NULL, default false      是否默认选中
extra_data        jsonb                                          扩展属性（如颜色、图标）
created_at        timestamptz       NOT NULL, default now()
updated_at        timestamptz       NOT NULL, default now()

唯一约束: UNIQUE (group_id, option_code)
索引:
  - idx_option_item_group_active: (group_id, is_active, sort_order)
触发器: set_updated_at()
```

#### 1.1.3 操作日志表

> 审计要求：所有静态文本和下拉选项的变更必须可追溯。

```
表名: content_audit_log

字段              类型              约束                        说明
─────────────────────────────────────────────────────────────────────
id                uuid              PK
tenant_id         uuid              NOT NULL
operator_id       uuid              NOT NULL, FK → app_user(id)   操作人
resource_type     varchar(64)       NOT NULL                     资源类型（static_text / option_group / option_item / 业务实体名）
resource_id       uuid              NOT NULL                     资源 ID
action            varchar(32)       NOT NULL                     动作（create / update / delete / enable / disable / publish / unpublish）
before_snapshot   jsonb                                          变更前快照
after_snapshot    jsonb                                          变更后快照
ip_address        varchar(64)
user_agent        varchar(512)
created_at        timestamptz       NOT NULL, default now()

索引:
  - idx_audit_resource: (tenant_id, resource_type, resource_id, created_at DESC)
  - idx_audit_operator: (tenant_id, operator_id, created_at DESC)
分区建议: 按月分区（created_at）
```

#### 1.1.4 App 配置快照表（可选，用于客户端批量拉取）

```
表名: app_config_snapshot

字段              类型              约束                        说明
─────────────────────────────────────────────────────────────────────
id                uuid              PK
tenant_id         uuid              NOT NULL, FK → tenant(id)
namespace         varchar(64)       NOT NULL                     配置命名空间（对应 App 端）
locale            varchar(16)       NOT NULL
snapshot_version  bigint            NOT NULL                     单调递增版本号
content           jsonb             NOT NULL                     合并后的文本 + 选项快照
generated_at      timestamptz       NOT NULL, default now()

唯一约束: UNIQUE (tenant_id, namespace, locale, snapshot_version)
索引:
  - idx_snapshot_latest: (tenant_id, namespace, locale, snapshot_version DESC)
```

#### 1.1.5 动态内容管理 — 沿用现有领域表

动态内容（老人、护理任务、活动、事件、账单等）的数据模型已在 `DATABASE_DESIGN.md` 和 `POSTGRESQL_DDL_CORE.sql` 中定义。以下补充管理状态所需的通用字段建议：

| 字段 | 类型 | 说明 | 适用表 |
|------|------|------|--------|
| `publish_status` | varchar(32) | draft / published / unpublished / archived | 活动、通知模板、护理服务项 |
| `reviewed_by` | uuid | 审核人 | 活动、入住评估 |
| `reviewed_at` | timestamptz | 审核时间 | 活动、入住评估 |
| `review_notes` | text | 审核备注 | 活动、入住评估 |
| `pinned` | boolean | 是否置顶 | 活动、通知 |
| `visible_to` | varchar(32)[] | 可见范围（admin / nani / family） | 活动、通知模板 |

---

### 1.2 API 端点设计

> 说明：所有 Admin API 走 Admin BFF，App 读取配置走各自 BFF。路径前缀遵循现有网关规范。

#### 1.2.1 静态文本管理 API（Admin BFF）

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/admin/static-texts` | 列表查询（支持 namespace、key、locale 筛选，分页） | content_editor, admin |
| GET | `/api/admin/static-texts/:id` | 单条详情 | content_editor, admin |
| POST | `/api/admin/static-texts` | 新建文本条目 | content_editor |
| PUT | `/api/admin/static-texts/:id` | 更新文本内容（需带 version 做乐观锁校验） | content_editor |
| DELETE | `/api/admin/static-texts/:id` | 删除（仅非系统内置可删） | admin |
| POST | `/api/admin/static-texts/batch-import` | 批量导入（JSON / Excel） | content_editor |
| GET | `/api/admin/static-texts/export` | 批量导出（Excel / JSON） | content_editor, viewer |
| GET | `/api/admin/static-texts/namespaces` | 获取所有命名空间列表 | content_editor, viewer |

**请求示例 — 列表查询：**

```
GET /api/admin/static-texts?namespace=app_family&locale=zh-CN&keyword=探视&page=1&pageSize=20

Response 200:
{
  "items": [
    {
      "id": "uuid",
      "namespace": "app_family",
      "text_key": "visit.booking_success",
      "locale": "zh-CN",
      "text_value": "探视预约已提交，请等待审批。",
      "description": "家属 APP 探视预约成功提示",
      "version": 3,
      "updated_by": { "id": "uuid", "display_name": "张管理员" },
      "updated_at": "2026-04-01T10:00:00Z"
    }
  ],
  "total": 42,
  "page": 1,
  "pageSize": 20
}
```

**请求示例 — 更新文本：**

```
PUT /api/admin/static-texts/:id

Body:
{
  "text_value": "探视预约已成功提交，工作人员将在24小时内审核。",
  "version": 3
}

Response 200:
{
  "id": "uuid",
  "version": 4,
  "updated_at": "2026-04-07T14:00:00Z"
}

Response 409 (版本冲突):
{
  "type": "ConcurrencyConflict",
  "detail": "该文本已被其他管理员修改，请刷新后重试。"
}
```

#### 1.2.2 下拉选项管理 API（Admin BFF）

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/admin/option-groups` | 选项分组列表 | content_editor, viewer |
| POST | `/api/admin/option-groups` | 新建分组 | content_editor |
| PUT | `/api/admin/option-groups/:id` | 更新分组信息 | content_editor |
| DELETE | `/api/admin/option-groups/:id` | 删除分组（仅空组且非系统内置可删） | admin |
| GET | `/api/admin/option-groups/:groupId/items` | 获取某分组下的选项列表 | content_editor, viewer |
| POST | `/api/admin/option-groups/:groupId/items` | 新增选项 | content_editor |
| PUT | `/api/admin/option-groups/:groupId/items/:itemId` | 更新选项 | content_editor |
| DELETE | `/api/admin/option-groups/:groupId/items/:itemId` | 删除选项（仅无业务引用时可删） | admin |
| PATCH | `/api/admin/option-groups/:groupId/items/:itemId/toggle` | 启用/禁用选项 | content_editor |
| PUT | `/api/admin/option-groups/:groupId/items/reorder` | 批量更新排序 | content_editor |

**请求示例 — 选项分组列表：**

```
GET /api/admin/option-groups?status=active&keyword=护理

Response 200:
{
  "items": [
    {
      "id": "uuid",
      "group_code": "care_level",
      "group_name": "护理等级",
      "description": "老人护理等级选项",
      "is_system": true,
      "status": "active",
      "item_count": 5,
      "updated_at": "2026-04-01T10:00:00Z"
    }
  ]
}
```

**请求示例 — 批量排序：**

```
PUT /api/admin/option-groups/:groupId/items/reorder

Body:
{
  "ordering": [
    { "item_id": "uuid-1", "sort_order": 0 },
    { "item_id": "uuid-2", "sort_order": 1 },
    { "item_id": "uuid-3", "sort_order": 2 }
  ]
}

Response 200: { "success": true }
```

#### 1.2.3 App 配置读取 API（Family BFF / Nani BFF）

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/app/config` | 获取当前版本的完整配置快照（文本 + 选项） | authenticated app user |
| GET | `/api/app/config/delta?since_version={v}` | 增量配置（仅返回 since_version 之后的变更） | authenticated app user |
| GET | `/api/app/options/:groupCode` | 获取某分组的活跃选项 | authenticated app user |

**请求示例 — 完整配置快照：**

```
GET /api/app/config?locale=zh-CN

Response 200:
{
  "snapshot_version": 142,
  "texts": {
    "error.network_timeout": "网络连接超时，请检查网络设置后重试。",
    "visit.booking_success": "探视预约已成功提交，工作人员将在24小时内审核。",
    ...
  },
  "options": {
    "care_level": [
      { "code": "self_care", "label": "自理", "sort_order": 0 },
      { "code": "partial_care", "label": "半自理", "sort_order": 1 },
      { "code": "full_care", "label": "全护理", "sort_order": 2 },
      { "code": "special_care", "label": "特护", "sort_order": 3 }
    ],
    "alert_type": [ ... ],
    "room_type": [ ... ]
  }
}
```

**请求示例 — 增量配置拉取：**

```
GET /api/app/config/delta?since_version=140&locale=zh-CN

Response 200:
{
  "current_version": 142,
  "changed_texts": {
    "visit.booking_success": "探视预约已成功提交，工作人员将在24小时内审核。"
  },
  "changed_options": {
    "care_level": [ ... ]
  }
}

Response 304 (无变更):
（空响应体）
```

#### 1.2.4 动态内容管理 API（Admin BFF）

动态内容的 CRUD 端点已在现有 API 文档中按领域定义（elderly-management、health-monitoring、alerts-incidents 等）。以下补充通用管理操作端点模式：

**通用列表查询模式：**

| 方法 | 路径模式 | 说明 |
|------|----------|------|
| GET | `/api/admin/{entity}` | 分页列表，支持 status / keyword / date_range / sort 筛选 |
| GET | `/api/admin/{entity}/:id` | 详情查询 |
| POST | `/api/admin/{entity}` | 新建 |
| PUT | `/api/admin/{entity}/:id` | 编辑 |
| DELETE | `/api/admin/{entity}/:id` | 软删除 |
| PATCH | `/api/admin/{entity}/:id/status` | 状态变更（上架/下架/发布/撤回/审核通过/拒绝） |
| POST | `/api/admin/{entity}/batch` | 批量操作（批量删除、批量上架、批量审核） |

**各动态实体具体归属：**

| Admin 路由 | API 路径 | 领域服务 | 管理操作 |
|-----------|----------|---------|---------|
| `/elderly` | `/api/admin/elders` | Elder Service | 列表/详情/新建/编辑/入住/退住 |
| `/elderly/checkin` | `/api/admin/admissions` | Elder Service + Care Service | 入住审核/评估确认/计划生成 |
| `/elderly/visits` | `/api/admin/visits` | Visit Service | 列表/审批/登记 |
| `/staff` | `/api/admin/staff` | Staffing Service | 列表/详情/新建/编辑/启停 |
| `/staff/tasks` | `/api/admin/care-tasks` | Care Service | 列表/筛选/重分配 |
| `/staff/schedule` | `/api/admin/schedules` | Staffing Service | 排班查看/编辑/发布 |
| `/nursing/packages` | `/api/admin/care-packages` | Care Service | CRUD/上架/下架 |
| `/nursing/plans` | `/api/admin/care-plans` | Care Service | 列表/查看/调整/归档 |
| `/nursing/services` | `/api/admin/care-services` | Care Service | 服务项 CRUD/启停 |
| `/activities` | `/api/admin/activities` | Operations Service | CRUD/发布/取消/归档 |
| `/incidents` | `/api/admin/incidents` | Operations Service | 列表/详情/分派/关闭/复盘 |
| `/alerts` | `/api/admin/alerts` | Operations Service | 列表/详情/分派/升级/关闭 |
| `/devices` | `/api/admin/devices` | Operations Service | 列表/绑定/解绑/维保 |
| `/supplies` | `/api/admin/supplies` | Operations Service | 库存/入库/出库/补货 |
| `/rooms` | `/api/admin/rooms` | Operations Service | 列表/详情/状态变更 |
| `/organizations` | `/api/admin/organizations` | Tenant Service | 列表/详情/启停 |
| `/financial` | `/api/admin/bills` | Billing Service | 账单列表/详情/调整/对账 |
| `/notifications` | `/api/admin/notification-templates` | Notification Service | 模板 CRUD/发布 |

#### 1.2.5 操作日志查询 API

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/admin/audit-logs` | 查询操作日志（支持 resource_type / operator / date_range 筛选） | admin, auditor |
| GET | `/api/admin/audit-logs/:resourceType/:resourceId` | 查询某资源的变更历史 | admin, auditor |

---

### 1.3 权限与安全

#### 1.3.1 管理员角色定义

| 角色 | 编码 | 权限范围 |
|------|------|---------|
| 超级管理员 | `super_admin` | 全平台所有功能，包括租户管理、角色配置、系统设置 |
| 机构管理员 | `org_admin` | 本机构范围内的全部管理功能 |
| 内容编辑员 | `content_editor` | 静态文本、下拉选项、活动、通知模板的 CRUD |
| 护理主管 | `care_supervisor` | 护理服务项配置、护理计划审核、任务分配 |
| 运营主管 | `ops_manager` | 老人管理、入住审核、财务查看、报警处置、设备管理 |
| 只读观察员 | `viewer` | 所有模块只读查看，不可修改 |
| 审计员 | `auditor` | 操作日志查看、合规审计 |

#### 1.3.2 权限粒度矩阵

| 资源 | super_admin | org_admin | content_editor | care_supervisor | ops_manager | viewer | auditor |
|------|:-----------:|:---------:|:--------------:|:---------------:|:-----------:|:------:|:-------:|
| 静态文本 CRUD | ✓ | ✓ | ✓ | — | — | R | — |
| 下拉选项 CRUD | ✓ | ✓ | ✓ | — | — | R | — |
| 下拉选项删除 | ✓ | ✓ | — | — | — | — | — |
| 老人管理 | ✓ | ✓ | — | R | ✓ | R | — |
| 入住审核 | ✓ | ✓ | — | ✓ | ✓ | R | — |
| 护理服务配置 | ✓ | ✓ | — | ✓ | — | R | — |
| 护理计划审核 | ✓ | ✓ | — | ✓ | — | R | — |
| 员工管理 | ✓ | ✓ | — | — | ✓ | R | — |
| 排班管理 | ✓ | ✓ | — | ✓ | ✓ | R | — |
| 活动管理 | ✓ | ✓ | ✓ | — | ✓ | R | — |
| 报警处置 | ✓ | ✓ | — | ✓ | ✓ | R | — |
| 设备管理 | ✓ | ✓ | — | — | ✓ | R | — |
| 财务管理 | ✓ | ✓ | — | — | ✓ | R | — |
| 系统设置 | ✓ | ✓ | — | — | — | — | — |
| 角色权限管理 | ✓ | — | — | — | — | — | — |
| 操作日志查看 | ✓ | ✓ | — | — | — | — | ✓ |

#### 1.3.3 安全约束

- 所有 Admin API 需 JWT Bearer token 认证，由 Gateway 统一校验。
- 权限检查在 BFF 层执行，基于 token 中的 `roles` claim。
- 静态文本和下拉选项的删除操作需二次确认 + 审计日志。
- App 端的 `/api/app/config` 为只读接口，不可通过此接口修改配置。
- 批量导入需校验文件格式与内容安全性（防止 XSS 注入、超限字段）。

---

### 1.4 其他建议

#### 1.4.1 缓存策略

| 层级 | 策略 | TTL | 失效方式 |
|------|------|-----|---------|
| App 配置快照 | Redis 缓存 `app_config:{tenant}:{namespace}:{locale}` | 10 分钟 | Admin 修改文本/选项时写入新 snapshot 并删除缓存 |
| 选项分组列表 | Redis 缓存 `option_groups:{tenant}` | 30 分钟 | 分组变更时 invalidate |
| 静态文本运行时 | App 本地 SQLite / SharedPreferences | 启动时 delta 同步 | `since_version` 增量拉取 |
| 动态内容列表 | 不建议全局缓存 | — | 通过分页 + 索引保证查询性能 |

#### 1.4.2 操作日志记录方案

- **记录时机**：所有 CUD 操作在 BFF 层统一拦截记录，通过 ASP.NET Core Action Filter 自动采集。
- **记录内容**：操作人、操作时间、资源类型、资源 ID、动作类型、变更前后快照、IP 地址。
- **存储方式**：写入 `content_audit_log` 表（同步写入，不经 outbox 异步），按月分区。
- **查询方式**：Admin 提供"操作日志"页面，支持按资源类型、操作人、时间范围筛选。
- **保留策略**：默认保留 2 年，超过保留期的日志可归档至冷存储。

#### 1.4.3 多语言支持方案

- 静态文本表通过 `(text_key, locale)` 组合实现多语言。
- 下拉选项通过 `label_zh` / `label_en` 列支持双语；如需更多语言，扩展为 `label_{locale}` 或使用 JSONB `labels` 字段。
- App 配置快照按 locale 生成独立版本，App 启动时按设备语言拉取对应 locale。
- Admin 编辑界面支持同一 key 切换 locale tab 编辑。

#### 1.4.4 版本与回滚

- 静态文本使用 `version` 字段做乐观锁，防止并发覆盖。
- 操作日志的 `before_snapshot` 字段可作为人工回滚的参考依据。
- App 配置快照通过 `snapshot_version` 单调递增，App 端可 fallback 到上一个已缓存版本。
- 首期不建议实现自动版本回滚功能；若有需求，可基于 `content_audit_log` 的 `before_snapshot` 做一键还原。

#### 1.4.5 开发环境初始化数据策略

- scope: 为 `tenant-demo` 提供一组开发环境可重复执行的 Config Service 初始化数据，覆盖 `/settings/static-texts`、`/settings/option-groups`、`/settings/audit-logs` 三个已落地页面的非空态联调。
- affected audience: 本地联调的后端研发、Admin 前端研发、测试。
- changed behavior: 仅在 Development 环境启动 Config Service 时执行一次幂等 seed；若目标租户已有同 key 数据，则保留现有数据，不做覆盖。
- dependent systems: Admin BFF 内容管理代理、Admin 设置中心三张页面、后续 Family/Nani 端只读配置拉取。
- verification: `npm run docs:build`；Config Service 启动后通过 `/api/admin/static-texts`、`/api/admin/option-groups`、`/api/admin/audit-logs` 或对应 admin 前端代理接口确认返回非空结果。
- rollback: 回退启动 seed 逻辑；如需清理运行态，可删除 `tenant-demo` 下由固定 seed id 写入的静态文本、选项组、选项项和审计日志记录。

建议首批样本范围：

- 静态文本：覆盖 `app_family`、`app_nani`、`admin`、`common` 四类 namespace，至少包含探视、任务提醒、审核提示和通用错误文案。
- 下拉选项：至少覆盖 `care_level`、`alert_type`、`visit_type`、`payment_status` 四个高频分组，并带 2 到 6 个条目用于列表、排序和启停展示。
- 操作日志：包含 `static_text`、`option_group`、`option_item` 三类资源的 create、update、disable 示例，保证页面筛选和展开详情时有真实样本。

---

## 2. Admin 管理后台设计补全

### 2.1 页面结构（菜单与布局）

> 在现有 `TopNavbar.tsx` 的已有导航基础上，新增"内容管理"导航组。

#### 2.1.1 新增导航分组

```
内容管理
├── 静态文本管理        /settings/static-texts
├── 下拉选项管理        /settings/option-groups
└── 操作日志            /settings/audit-logs
```

#### 2.1.2 为什么放在 /settings 下

- "内容管理"本质是系统配置能力，与角色权限、系统参数属于同一治理域。
- 放在 `/settings` 下不增加顶层导航复杂度，符合现有导航收敛原则。
- 超级管理员和内容编辑员通过权限可见；普通运营人员在 `/settings` 下不可见此入口。

#### 2.1.3 动态内容管理的入口

动态内容不新建独立菜单，而是通过各业务模块已有的列表页承载管理操作：

| 动态实体 | 管理入口 | 页面路由 |
|---------|---------|---------|
| 老人档案 | 老人管理 → 老人列表 | `/elderly` |
| 入住审核 | 老人管理 → 入住办理 | `/elderly/checkin` |
| 探视记录 | 老人管理 → 探视管理 | `/elderly/visits` |
| 护理套餐 | 护理服务 → 套餐管理 | `/nursing/packages` |
| 护理计划 | 护理服务 → 计划管理 | `/nursing/plans` |
| 护理服务项 | 护理服务 → 服务项目库 | `/nursing/services` |
| 活动 | 运营 → 活动列表 | `/activities` |
| 事件/事故 | 运营 → 事件列表 | `/incidents` |
| 报警 | 报警中心 | `/alerts` |
| 设备 | 设备管理 | `/devices` |
| 物资 | 物资管理 | `/supplies` |
| 账单 | 财务管理 | `/financial` |
| 通知模板 | 系统设置 → 通知模板 | `/settings/notifications` |

---

### 2.2 静态文本管理模块

**路由**：`/settings/static-texts`

**权限**：content_editor, org_admin, super_admin（viewer 只读）

#### 2.2.1 页面布局

```
┌──────────────────────────────────────────────────────────────┐
│  静态文本管理                                    [导入] [导出] │
├──────────────────────────────────────────────────────────────┤
│  命名空间: [全部 ▼]   语言: [zh-CN ▼]   搜索: [________🔍]  │
├──────────────────────────────────────────────────────────────┤
│  文本 Key          │ 文本内容               │ 说明    │ 修改时间  │ 操作      │
│  ──────────────────┼────────────────────────┼─────────┼──────────┼──────────│
│  error.network     │ 网络连接超时，请检查…    │ 通用错误 │ 04-01    │ [编辑]   │
│  visit.booking_ok  │ 探视预约已成功提交…     │ 家属APP │ 04-03    │ [编辑]   │
│  care.task_remind  │ 您有一项护理任务即将…    │ 员工APP │ 03-28    │ [编辑]   │
│  ...              │                        │         │          │          │
├──────────────────────────────────────────────────────────────┤
│  共 142 条    < 1 2 3 4 5 >                                   │
└──────────────────────────────────────────────────────────────┘
```

#### 2.2.2 功能说明

| 功能 | 说明 |
|------|------|
| 列表展示 | 显示 text_key、text_value（截断）、description、命名空间标签、最后修改时间 |
| 统计卡片 | 命名空间卡片数量必须来自 Config Service 列表接口返回的真实 total；不可再复用前端 demo helper 或硬编码样例 |
| 筛选 | 按命名空间（app_family / app_nani / admin / common）、语言（zh-CN / en-US） |
| 搜索 | 按 text_key 或 text_value 模糊搜索 |
| 编辑 | 弹窗编辑，显示完整 key、当前值、说明、多语言 tab 切换 |
| 新建 | 弹窗新建，需填 namespace、key、locale、value、description |
| 批量导入 | 支持 Excel / JSON 格式上传，预览变更行后确认导入 |
| 批量导出 | 导出当前筛选范围的数据为 Excel / JSON |
| 删除 | 仅 admin 角色可删，删除前二次确认，系统 key（is_system 标记）不可删 |

静态文本页的健康信号定义如下：

- scope: `/settings/static-texts` 的列表、分页总数与命名空间统计卡片统一收敛到 Config Service -> Admin BFF -> Admin 前端的只读链路。
- affected audience: 内容编辑、机构管理员、超级管理员；viewer 保持只读。
- verification: `npm run docs:build`；前端运行时确认筛选列表总数与命名空间卡片都能在真实后端返回下稳定展示，不再因本地 demo helper 导致统计与列表不一致。
- rollback: 恢复前端统计卡片到旧的本地 helper 计算逻辑，服务端接口与数据库结构无需回退。

#### 2.2.3 编辑弹窗

```
┌─────────────────────────────────────┐
│  编辑静态文本                   [×]  │
├─────────────────────────────────────┤
│  命名空间: app_family               │
│  Key: visit.booking_success         │
│                                     │
│  [zh-CN] [en-US]                    │
│  ┌─────────────────────────────┐    │
│  │ 探视预约已成功提交，工作人     │    │
│  │ 员将在24小时内审核。          │    │
│  └─────────────────────────────┘    │
│  说明: 家属APP探视预约成功提示       │
│                                     │
│  [取消]              [保存]         │
└─────────────────────────────────────┘
```

---

### 2.3 静态下拉选项管理模块

**路由**：`/settings/option-groups`

**权限**：content_editor, org_admin, super_admin（viewer 只读）

#### 2.3.1 页面布局 — 分组列表

```
┌──────────────────────────────────────────────────────────────┐
│  下拉选项管理                                   [新建分组]    │
├──────────────────────────────────────────────────────────────┤
│  搜索: [____________🔍]    状态: [全部 ▼]                     │
├──────────────────────────────────────────────────────────────┤
│  分组编码        │ 分组名称      │ 选项数 │ 类型   │ 操作          │
│  ────────────────┼──────────────┼────────┼────────┼──────────────│
│  care_level      │ 护理等级     │ 4      │ 🔒系统  │ [管理选项]     │
│  alert_type      │ 报警类型     │ 6      │ 🔒系统  │ [管理选项]     │
│  room_type       │ 房间类型     │ 3      │ 🔒系统  │ [管理选项]     │
│  activity_type   │ 活动类型     │ 5      │ 自定义  │ [管理选项] [删除] │
│  supply_category │ 物资分类     │ 8      │ 自定义  │ [管理选项] [删除] │
│  ...            │              │        │        │              │
└──────────────────────────────────────────────────────────────┘
```

#### 2.3.2 页面布局 — 选项管理

点击"管理选项"后进入选项管理页或展开侧面板：

```
┌──────────────────────────────────────────────────────────────┐
│  护理等级（care_level）选项管理          🔒系统分组  [新增选项]  │
├──────────────────────────────────────────────────────────────┤
│  ☰ │ 选项编码     │ 中文名称  │ 英文名称     │ 排序 │ 状态  │ 操作         │
│  ──┼─────────────┼──────────┼─────────────┼──────┼───────┼─────────────│
│  ⠿ │ self_care   │ 自理     │ Self Care   │ 0    │ ✅启用 │ [编辑] [禁用] │
│  ⠿ │ partial     │ 半自理   │ Partial     │ 1    │ ✅启用 │ [编辑] [禁用] │
│  ⠿ │ full_care   │ 全护理   │ Full Care   │ 2    │ ✅启用 │ [编辑] [禁用] │
│  ⠿ │ special     │ 特护     │ Special     │ 3    │ ✅启用 │ [编辑] [禁用] │
│  ⠿ │ icu_care    │ ICU护理  │ ICU Care    │ 4    │ ⛔禁用 │ [编辑] [启用] │
├──────────────────────────────────────────────────────────────┤
│  提示：拖拽左侧 ⠿ 图标可调整选项排序                            │
└──────────────────────────────────────────────────────────────┘
```

#### 2.3.3 功能说明

| 功能 | 说明 |
|------|------|
| 分组列表 | 展示分组编码、名称、选项数量、系统/自定义标记 |
| 新建分组 | 弹窗输入 group_code、group_name、description |
| 选项列表 | 展示选项编码、多语言标签、排序、启用状态 |
| 拖拽排序 | 通过拖拽手柄调整 sort_order，松手后自动保存 |
| 启用/禁用 | 切换选项的 is_active 状态。禁用后 App 端不可见，但已有数据不受影响 |
| 新增选项 | 弹窗输入 option_code、label_zh、label_en、extra_data |
| 编辑选项 | 弹窗编辑显示文本和扩展属性，option_code 创建后不可修改 |
| 删除选项 | 仅在选项无业务引用时可删，否则提示"该选项已被 N 条记录引用，请先禁用" |
| 删除分组 | 仅自定义分组且分组下无选项时可删 |

---

### 2.4 动态内容管理模块

> 动态内容的管理界面已融入各业务模块的现有页面中。以下补全通用管理交互模式。

#### 2.4.1 通用列表页模式

```
┌──────────────────────────────────────────────────────────────┐
│  {实体名称}列表                               [新建] [批量操作 ▼] │
├──────────────────────────────────────────────────────────────┤
│  状态: [全部 ▼]  时间: [近7天 ▼]  关键词: [________🔍]  [重置] │
├──────────────────────────────────────────────────────────────┤
│  □ │ 名称       │ 状态    │ 负责人  │ 创建时间  │ 操作              │
│  ──┼────────────┼─────────┼─────────┼──────────┼──────────────────│
│  □ │ 春日郊游    │ 🟢已发布 │ 张主管  │ 04-01    │ [编辑][下架][删除] │
│  □ │ 健康讲座    │ 🟡待审核 │ 李编辑  │ 04-03    │ [编辑][审核][删除] │
│  □ │ 手工课程    │ ⚪草稿   │ 王编辑  │ 04-05    │ [编辑][提交审核]   │
│  ...            │         │         │          │                  │
├──────────────────────────────────────────────────────────────┤
│  共 38 条    < 1 2 >              已选 2 项 [批量发布] [批量删除] │
└──────────────────────────────────────────────────────────────┘
```

#### 2.4.2 通用状态流转

根据实体类型，状态流转分两类：

**简单发布型**（活动、通知模板、护理服务项）：
```
草稿 → 待审核 → 已发布 → 已下架 → 已归档
                ↗                ↙
           拒绝（回到草稿）
```

**工作流型**（入住审核、报警、事件）：
```
已在各 workflow 文档中定义（admission-workflow、operations-workflow），此处不重复。
```

#### 2.4.3 行级操作

| 操作 | 触发条件 | 说明 |
|------|---------|------|
| 编辑 | 草稿/待审核/已发布 | 打开编辑弹窗或跳转编辑页 |
| 提交审核 | 仅草稿状态 | 变更为"待审核"，通知审核人 |
| 审核通过 | 仅待审核状态 | 变更为"已发布"，需审核权限 |
| 审核拒绝 | 仅待审核状态 | 回到"草稿"，需填写拒绝原因 |
| 上架 | 仅已下架状态 | 重新发布 |
| 下架 | 仅已发布状态 | 设为"已下架"，App 端不可见 |
| 删除 | 草稿/已下架 | 软删除，需二次确认 |
| 归档 | 已下架 | 设为"已归档"，不可恢复上架 |

#### 2.4.4 详情页/编辑弹窗

根据实体复杂度采用两种模式：

- **简单实体**（通知模板、选项等）：使用弹窗编辑，所有字段在一个 Modal 内展示。
- **复杂实体**（老人档案、护理计划、活动等）：跳转独立编辑页，使用多 tab 或分步表单。

#### 2.4.5 批量操作

| 操作 | 说明 | 确认方式 |
|------|------|---------|
| 批量发布 | 将选中的草稿/待审核项直接设为已发布 | 确认对话框，显示影响数量 |
| 批量下架 | 将选中的已发布项设为已下架 | 确认对话框 |
| 批量删除 | 软删除选中项（仅草稿/已下架可删） | 危险确认对话框（红色按钮 + 输入确认文字） |
| 批量分派 | 报警/事件场景，批量指定负责人 | 选择负责人弹窗 |

---

### 2.5 交互与体验建议

#### 2.5.1 表单与验证

- 所有表单使用即时校验（失焦触发 + 提交前全量校验）。
- 必填项标记 `*`，校验失败时在字段下方显示红色错误提示。
- 文本编辑框使用 `textarea` 并显示当前字符数 / 最大字符数。
- 选项编码（`option_code`、`text_key`）仅允许字母、数字、下划线、点号。

#### 2.5.2 危险操作保护

- 删除操作：弹出确认对话框，显示"此操作不可恢复，确定要删除 {资源名} 吗？"
- 批量删除：确认文案升级为"您即将删除 N 条记录，此操作不可恢复。"，按钮为红色。
- 禁用选项：提示"禁用后该选项将不再出现在 App 下拉列表中，已有数据不受影响。"
- 下架/撤回：提示"下架后 App 端将不再展示此内容。"

#### 2.5.3 操作反馈

- 保存成功：页面顶部 Toast "保存成功"（绿色，3 秒自动消失）。
- 保存失败：页面顶部 Toast "保存失败：{错误信息}"（红色，需手动关闭）。
- 版本冲突：弹窗提示"该内容已被其他管理员修改，请刷新页面后重试。"
- 列表操作后：保持当前分页位置和筛选条件，仅刷新操作行的状态。

#### 2.5.4 加载状态

- 列表加载：显示骨架屏（Skeleton），不使用全局 loading spinner。
- 保存中：提交按钮显示 loading 状态，禁止重复点击。
- 导入中：显示进度条和当前处理行数。

#### 2.5.5 空状态

- 列表无数据：显示空状态图标 +"暂无{实体名}数据"+ 新建引导按钮。
- 筛选无结果：显示"未找到匹配结果"+ 清除筛选按钮。

---

## 3. 需要向用户澄清的问题

基于现有文档，以下事项仍有不确定性，建议在实施前确认：

### 3.1 多语言范围

- 当前设计支持 zh-CN / en-US 双语。是否需要支持更多语言（如繁体中文 zh-TW、日语 ja）？
- 多语言是面向 App 端用户，还是 Admin 后台本身也需要多语言？

### 3.2 静态文本版本回滚

- 首期是否需要支持一键回滚到某个历史版本？当前设计基于审计日志的 before_snapshot 支持人工对照恢复，但未实现自动化回滚。
- 是否需要"文本变更审批流"（编辑后先进入待审核，审核通过后才生效）？

### 3.3 Admin 登录系统

- 现有设计中 Admin 使用 Identity Service 的统一登录（JWT Bearer）。是否需要 Admin 独立的登录系统或独立的用户管理？
- 当前架构推荐 Admin 用户与 App 用户共用 `app_user` 表、通过 `user_type` 区分。是否需要独立的 `admin_user` 表？

### 3.4 下拉选项与业务数据的关联校验

- 禁用某个选项后，已有业务数据中引用该选项的记录如何展示？建议保持显示原选项文本但标记为"已停用"。
- 是否允许彻底删除已被引用的选项？建议不允许，仅允许禁用。

### 3.5 动态内容的审核流程

- 所有动态内容都需要审核流程，还是仅特定实体（如活动、通知模板）需要？
- 审核人是否需要配置为特定角色/人员？还是任何 admin/supervisor 均可审核？

### 3.6 配置快照的生成策略

- App 配置快照是在每次文本/选项变更时实时生成（写放大），还是定时批量生成（读延迟）？
- 建议：少量高频变更场景用实时生成；大批量初始化导入后用批量生成。

### 3.7 通知模板管理

- 通知模板是否包含推送通道配置（站内信/短信/APP push）？
- 通知模板变量是否需要在 Admin 中提供可视化占位符编辑？

### 3.8 已有系统内置选项清单

- 以下分组建议作为系统内置（is_system = true），需确认是否完整：
  - care_level（护理等级）
  - alert_type（报警类型）
  - alert_level（报警级别）
  - room_type（房间类型）
  - device_type（设备类型）
  - elder_status（老人状态）
  - admission_status（入住状态）
  - task_status（任务状态）
  - task_type（任务类型）
  - supply_category（物资分类）
  - activity_type（活动类型）
  - gender（性别）
  - risk_level（风险等级）

---

## 附录 A：与现有文档的映射关系

| 现有文档 | 本文档对应章节 | 关系 |
|---------|-------------|------|
| DATABASE_DESIGN.md | 1.1 数据模型 | 新增 static_text / option_group / option_item / content_audit_log / app_config_snapshot 表 |
| POSTGRESQL_DDL_CORE.sql | 1.1 数据模型 | 需追加新表的 DDL |
| backend-service-domain-design.md | 1.2 API 端点 | Config Service 归属确认 |
| MODULE_PAGE_MAPPING.md | 2.1 页面结构 | 新增 /settings/static-texts、/settings/option-groups、/settings/audit-logs 路由 |
| backend-dataflow-workflows.md | 2.4 动态内容 | 沿用已有 workflow 定义 |
| core-modules.md | 1.2 API 端点 | 新增"内容配置管理"模块 |

## 附录 B：服务边界归属

```
Config Service（新建）
├── 静态文本 CRUD + 快照生成
├── 下拉选项分组/项 CRUD
├── App 配置快照查询
└── 操作日志记录（写入 content_audit_log）

Admin BFF
├── /api/admin/static-texts → 调用 Config Service
├── /api/admin/option-groups → 调用 Config Service
├── /api/admin/audit-logs → 调用 Config Service
└── /api/admin/{entity} → 调用各领域服务

Family BFF
├── /api/app/config → 调用 Config Service（缓存优先）
└── /api/app/options/:groupCode → 调用 Config Service

Nani BFF
├── /api/app/config → 调用 Config Service（缓存优先）
└── /api/app/options/:groupCode → 调用 Config Service
```
