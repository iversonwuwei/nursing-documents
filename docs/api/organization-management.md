# 机构管理 API 契约

## Scope

- Bounded context: organization-topology
- Delivery boundary: 本轮只真实化 admin 机构列表、机构新建、机构详情和机构启用。
- Affected callers: `nursing-admin-v2` 的 `/organizations`、`/organizations/new`、`/organizations/[id]`，以及 `/rooms/new` 的机构选择器。

## Compatibility

- 机构主档是独立服务事实源，不复用 tenant entitlement DTO。
- 床位/入住摘要由 Admin BFF 聚合 rooms 真实数据得出，organization service 自身不负责床位占用事实。
- 员工摘要由 Admin BFF 聚合 staffing 的真实机构归属字段得出；未绑定机构的员工不计入机构员工口径。

## Service Endpoints

### `GET /api/organizations/organizations`

- 用途: 返回机构主档列表。
- Query:
 	- `keyword?`: 匹配机构名称、地址、负责人、电话
 	- `status?`: `运营中 | 筹备中 | 暂停营业`
 	- `lifecycleStatus?`: `待启用 | 已启用`
 	- `page?`, `pageSize?`

### `GET /api/organizations/organizations/{organizationId}`

- 用途: 返回单个机构主档详情。

### `POST /api/organizations/organizations`

- 用途: 创建机构主档，默认进入 `待启用`。

### `POST /api/organizations/organizations/{organizationId}/activate`

- 用途: 人工确认机构启用，启用后进入机构运营台账。

## Admin BFF Endpoints

### `GET /api/admin/organizations`

- 转发 organization service 列表，并聚合 rooms 实时床位/入住摘要。

### `GET /api/admin/organizations/{organizationId}`

- 转发 organization service 详情，并附加 rooms 明细、床位汇总和当前机构 staff 名册。

### `POST /api/admin/organizations`

- 创建机构主档。

### `POST /api/admin/organizations/{organizationId}/activate`

- 确认机构启用。

## Field Model

### Organization Summary

```json
{
 "organizationId": "ORG-1713339999123",
 "tenantId": "tenant-demo",
 "name": "浦东康养中心",
 "address": "上海市浦东新区张江镇科苑路888号",
 "phone": "021-58881234",
 "status": "运营中",
 "establishedDate": "2026-04-17",
 "manager": "王建国",
 "managerPhone": "13812340001",
 "description": "提供失能老人长期照护与康复支持。",
 "lifecycleStatus": "已启用",
 "createdAt": "2026-04-17",
 "activatedAt": "2026-04-17 11:30",
 "activationNote": "机构资料已复核，允许进入运营台账。",
 "totalBeds": 120,
 "occupiedBeds": 98,
 "availableBeds": 22,
 "elderlyCount": 98,
 "staffCount": 0,
 "roomCount": 56,
 "staffIntegrationStatus": "pending"
}
```

### Organization Detail

```json
{
 "organization": {
  "organizationId": "ORG-1713339999123",
  "tenantId": "tenant-demo",
  "name": "浦东康养中心",
  "address": "上海市浦东新区张江镇科苑路888号",
  "phone": "021-58881234",
  "status": "运营中",
  "establishedDate": "2026-04-17",
  "manager": "王建国",
  "managerPhone": "13812340001",
  "description": "提供失能老人长期照护与康复支持。",
  "lifecycleStatus": "已启用",
  "createdAt": "2026-04-17",
  "activatedAt": "2026-04-17 11:30",
  "activationNote": "机构资料已复核，允许进入运营台账。",
  "totalBeds": 120,
  "occupiedBeds": 98,
  "availableBeds": 22,
  "elderlyCount": 98,
  "staffCount": 18,
  "roomCount": 56,
  "staffIntegrationStatus": "live"
 },
 "rooms": [
  {
   "roomId": "R501",
   "name": "康复双人间",
   "floorName": "5楼",
   "type": "双人间",
   "capacity": 2,
   "occupied": 1,
   "status": "可入住",
   "cleanStatus": "已清洁"
  }
 ],
 "staff": [
  {
   "staffId": "STF-1744876800000",
   "tenantId": "tenant-demo",
   "name": "陈静",
   "role": "护士",
   "department": "护理部",
   "organizationId": "ORG-1713339999123",
   "organizationName": "浦东康养中心",
   "employmentSource": "第三方合作",
   "partnerAgencyId": null,
   "partnerAgencyName": "安心照护服务中心",
   "partnerAffiliationRole": "驻场护工",
   "phone": "13800005555",
   "status": "待入职",
   "gender": "女",
   "email": "chenjing@example.com",
   "age": 34,
   "performance": 0,
   "attendance": 0,
   "satisfaction": 0,
   "hireDate": "2026-04-17",
   "schedule": [
    { "day": "周一", "shift": "待排班" }
   ],
   "certificates": [],
   "bonus": "待核定",
   "lifecycleStatus": "待入职",
   "createdAt": "2026-04-17T08:00:00Z",
   "activatedAt": null,
   "onboardingNote": null
  }
 ]
}
```

### Create Request

```json
{
 "name": "浦东康养中心",
 "address": "上海市浦东新区张江镇科苑路888号",
 "phone": "021-58881234",
 "manager": "王建国",
 "managerPhone": "13812340001",
 "description": "提供失能老人长期照护与康复支持。"
}
```

### Activate Request

```json
{
 "activationNote": "机构资料已复核，允许进入运营台账。"
}
```

## Error Codes

- `400`: 缺少必要字段、电话格式或描述不合法
- `404`: 机构不存在
- `409`: 机构名称重复或机构已启用
- `502`: Admin BFF 下游 organization service 或 rooms aggregation 失败

## Verification

- `npm run docs:build`
- `dotnet build nursing-backend-services.slnx`
- admin `lint && build`

## Rollback

- 回退本契约和 organization service/Admin BFF/Next proxy/page 代码改动。
- 若聚合链路异常，可暂时让 admin 页面只展示机构主档事实与明确错误提示，但不得恢复前端 organizations mock 双读。
