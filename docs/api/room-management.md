# 房间与床位 API 契约

## Scope

- Bounded context: rooms-and-capacity
- Delivery boundary: 本轮只真实化 admin 房间列表、房间新建、房间详情和房间启用。
- Affected callers: `nursing-admin-v2` 的 `/rooms`、`/rooms/new`、`/rooms/[id]`。

## Compatibility

- 房间状态、清洁状态和生命周期状态保持拆分字段，不复用单字段表达多重语义。
- 机构归属通过真实 organizations 主档选择器写入，`organizationId` 与 `organizationName` 同时保留以支撑正式引用与冗余展示。
- 床位入住对象由 Admin BFF 聚合真实老人主档，rooms service 自身不负责老人主档事实源。

## Service Endpoints

### `GET /api/rooms/rooms`

- 用途: 返回房间列表，支持关键字与状态筛选。
- Query:
 	- `keyword?`: 匹配房间编号、名称、机构名称
 	- `status?`: `可入住 | 已满 | 维护中 | 待启用`
 	- `lifecycleStatus?`: `待启用 | 已启用`
 	- `organizationName?`: 按机构名称筛选
 	- `page?`, `pageSize?`

### `GET /api/rooms/rooms/{roomId}`

- 用途: 返回单个房间主档详情与床位信息。

### `POST /api/rooms/rooms`

- 用途: 创建房间主档，默认进入 `待启用`。

### `POST /api/rooms/rooms/{roomId}/activate`

- 用途: 人工确认房间启用，启用后进入可分配资源池。

## Admin BFF Endpoints

### `GET /api/admin/rooms`

- 转发 rooms service 列表，并聚合老人主档生成 `bedsInfo` 中的入住对象摘要。

### `GET /api/admin/rooms/{roomId}`

- 转发 rooms service 详情，并聚合该房间下的老人入住对象。

### `POST /api/admin/rooms`

- 创建房间主档。

### `POST /api/admin/rooms/{roomId}/activate`

- 确认房间启用。

## Field Model

### Room Record

```json
{
 "roomId": "R501",
 "tenantId": "tenant-default",
 "name": "康复双人间",
 "floor": 5,
 "floorName": "5楼",
 "type": "双人间",
 "capacity": 2,
 "occupied": 1,
 "status": "可入住",
 "organizationId": "ORG-PD-01",
 "organizationName": "浦东康养中心",
 "facilities": ["空调", "独立卫浴", "紧急呼叫"],
 "cleanStatus": "已清洁",
 "lastClean": "2026-04-17 07:00",
 "nextClean": "2026-04-18 07:00",
 "lifecycleStatus": "已启用",
 "createdAt": "2026-04-17",
 "activatedAt": "2026-04-17 10:00",
 "activationNote": "房间资料已复核，允许进入排房资源池。",
 "bedsInfo": [
  {
   "bedId": 1,
   "status": "occupied",
   "elder": {
    "elderId": "E001",
    "name": "张秀英",
    "careLevel": "全护理",
    "checkIn": "2026-04-10"
   }
  },
  {
   "bedId": 2,
   "status": "available"
  }
 ]
}
```

### Create Request

```json
{
 "roomId": "R501",
 "name": "康复双人间",
 "floor": 5,
 "type": "双人间",
 "capacity": 2,
 "organizationId": "ORG-PD-01",
 "organizationName": "浦东康养中心",
 "facilities": ["空调", "独立卫浴", "紧急呼叫"]
}
```

### Activate Request

```json
{
 "activationNote": "房间资料已复核，允许进入排房资源池。"
}
```

## Error Codes

- `400`: 缺少必要字段、楼层或床位数非法
- `404`: 房间不存在
- `409`: 房间编号重复或房间已启用
- `502`: Admin BFF 下游 rooms service 或 elder service 聚合失败

## Verification

- `npm run docs:build`
- `dotnet build nursing-backend-services.slnx`
- admin `lint && build`

## Rollback

- 回退本契约和 rooms service/Admin BFF/Next proxy/page 代码改动。
- 若聚合链路异常，可暂时把 admin 页面回退为只读本地 room workflow，但不得保留双写状态。
