# Admin Gateway Admin API 代理契约

## Scope

- scope: 定义 Gateway 对 admin API 的统一代理契约，仅覆盖 `/api/admin/**` 在 Gateway 层的入口行为。
- compatibility: 追加 Gateway 入口，不破坏现有 Admin BFF `/api/admin/**` 路径与响应结构。
- caller impact: `nursing-admin-v2` 的 Next route handlers、后续平台入口联调脚本、部署侧 ingress/edge 配置。
- examples: 给出 bootstrap 与 admin 业务查询通过 Gateway 的调用方式。
- verification: 文档构建通过；`/api/gateway/bootstrap` 与至少 10 条 `/api/admin/**` 读接口通过 `5200` 返回 200。
- rollback: 删除 Gateway `/api/admin/{**path}` 代理，调用方恢复直连 Admin BFF。

## 路径规则

- Gateway bootstrap 保持不变：`GET /api/gateway/bootstrap`
- 新增 Gateway admin 代理入口：`/api/admin/{**path}`
- Gateway 必须保持完整 path 和 query，不得改写成其他前缀。

## Header 规则

- 必须透传 `Authorization`
- 必须透传 `X-Tenant-Id`
- 必须透传 `X-Correlation-Id`
- 若请求有 `Content-Type`，应继续透传

## 状态码规则

- 下游 Admin BFF 返回什么状态码，Gateway 应尽量原样返回。
- 常见状态：
  - `200`: 查询成功
  - `401`: 未登录或 token 无效
  - `404`: Admin BFF 不存在该业务路径
  - `502`: Gateway 到 Admin BFF 转发失败

## 示例

### 1. Bootstrap

- method: `GET`
- path: `/api/gateway/bootstrap`

### 2. Admin 评定列表通过 Gateway

- method: `GET`
- path: `/api/admin/assessments?page=1&pageSize=2`

### 3. Admin 运营活动列表通过 Gateway

- method: `GET`
- path: `/api/admin/activities?page=1&pageSize=2`

### 4. Admin 财务摘要通过 Gateway

- method: `GET`
- path: `/api/admin/finance/summary`

## 响应兼容性

- Gateway 不应包装为新的 envelope。
- Gateway 不应把 Admin BFF 的 JSON 数组/对象改成字符串或其他媒体类型。
- 前端可继续沿用当前 DTO 解析逻辑，无需区分自己命中的是 `5146` 还是 `5200`。

## 本地联调基线

- Gateway: `http://localhost:5200`
- Admin BFF: `http://localhost:5146`
- Identity: `http://localhost:5265`
- Tenant: `http://localhost:5186`

## 观测建议

- 本地至少记录 `X-Correlation-Id`，便于串联 Gateway 与 Admin BFF 日志。
- 若某条 admin 接口通过 `5146` 成功但通过 `5200` 失败，应优先检查 Gateway 是否遗漏 query、Authorization 或 path catch-all 绑定。