# 员工协同 API 契约

## Scope

- scope: 为 admin 员工协同补齐真实员工列表、员工详情、新建员工、机构归属和确认入职接口。
- compatibility: 本轮不替换任务中心与排班页现有接口，只补 staff 主档与机构归属链路。
- caller impact: Admin 前端 `/staff`、`/staff/new`、`/staff/{id}`，Admin BFF，Staffing Service。
- verification: `npm run docs:build`、`dotnet build nursing-backend-services.slnx`、admin `npm run lint && npm run build`。
- rollback: 回退本文、Admin BFF staff 代理、Staffing Service 持久化字段和 admin staff live 化实现，恢复当前 resource workflow staff 版本。

## Delivery Boundary

- 当前交付真实化 staff 主档与机构归属链路。
- staff tasks、schedule、coverage summary 仍沿用现有 care/AI 路径，不在本次接口范围内。
- 机构养老 staff create 需显式写入机构归属；第三方合作人员同时保留护理服务机构名称。home 协同人员在 partners 切片落地前允许暂不绑定机构主档。

## Endpoints

### Query staff list

- method: `GET`
- path: `/api/admin/staff`
- purpose: 返回员工列表页所需的 staff 主档。
- filters:
 	- `keyword`: 匹配姓名、工号、电话、邮箱
 	- `department`: 按部门筛选
 	- `employmentSource`: `自营` / `第三方合作`
 	- `status`: `在职` / `休假` / `离职` / `待入职`
 	- `lifecycleStatus`: `待入职` / `已入职`
 	- `partnerAgency`: 匹配第三方合作机构名称
 	- `organizationId`: 按机构归属筛选
 	- `page`, `pageSize`: 当前先返回分页结构，admin 前端可继续本地二次筛选

### Query staff detail

- method: `GET`
- path: `/api/admin/staff/{staffId}`
- purpose: 返回员工详情页需要的完整 staff 对象事实。

### Create staff draft

- method: `POST`
- path: `/api/admin/staff`
- purpose: 新增员工主档，初始写入 `待入职` / `待入职` 状态。

### Activate staff onboarding

- method: `POST`
- path: `/api/admin/staff/{staffId}/activate`
- purpose: 人工确认员工入职，允许纳入排班与任务口径。

## Field Model

### StaffRecord

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| staffId | string | 稳定员工编号 |
| tenantId | string | 租户 id |
| name | string | 姓名 |
| role | string | 角色 |
| department | string | 部门 |
| organizationId | string? | 所属机构 id |
| organizationName | string? | 所属机构名称 |
| employmentSource | string | 用工来源 |
| partnerAgencyId | string? | 第三方合作机构 id，机构域未接通时可为空 |
| partnerAgencyName | string? | 第三方合作机构名称 |
| partnerAffiliationRole | string? | 第三方合作角色 |
| phone | string | 联系电话 |
| status | string | 人员状态 |
| gender | string | 性别 |
| email | string | 邮箱 |
| age | number | 年龄 |
| performance | number | 综合绩效分 |
| attendance | number | 出勤率 |
| satisfaction | number | 满意度 |
| hireDate | string | 入职日期 |
| schedule | object[] | 当前周排班摘要 |
| schedule[].day | string | 星期标签 |
| schedule[].shift | string | 班次标签 |
| certificates | string[] | 资质证书 |
| bonus | string | 奖金展示文案 |
| lifecycleStatus | string | 生命周期状态 |
| createdAt | string | 创建时间 |
| activatedAt | string? | 确认入职时间 |
| onboardingNote | string? | 入职确认备注 |

## Example Requests

### POST /api/admin/staff

```json
{
 "name": "陈静",
 "role": "护士",
 "department": "护理部",
 "organizationId": "ORG-1744876800000",
 "organizationName": "北城护理院",
 "employmentSource": "第三方合作",
 "partnerAgencyId": null,
 "partnerAgencyName": "安心照护服务中心",
 "partnerAffiliationRole": "驻场护工",
 "phone": "13800005555",
 "gender": "女",
 "email": "chenjing@example.com",
 "age": 34,
 "hireDate": "2026-04-17"
}
```

### POST /api/admin/staff/STF-1744876800000/activate

```json
{
 "onboardingNote": "资料已复核，可纳入排班与任务台账。"
}
```

## Example Response

```json
{
 "staffId": "STF-1744876800000",
 "tenantId": "tenant-demo",
 "name": "陈静",
 "role": "护士",
 "department": "护理部",
 "organizationId": "ORG-1744876800000",
 "organizationName": "北城护理院",
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
  { "day": "周一", "shift": "待排班" },
  { "day": "周二", "shift": "待排班" }
 ],
 "certificates": [],
 "bonus": "待核定",
 "lifecycleStatus": "待入职",
 "createdAt": "2026-04-17T08:00:00Z",
 "activatedAt": null,
 "onboardingNote": null
}
```

## Error Codes

- `400`: 必填字段缺失或第三方合作缺少机构名称
- `401`: 未登录或缺少租户上下文
- `404`: staff 不存在
- `409`: staff 主档重复或状态流转非法
- `502`: Admin BFF 下游 Staffing Service 调用失败

## Frontend Rules

- `/staff`、`/staff/new`、`/staff/{id}` 在 live 模式下只读取该契约返回的 staff 主档。
- organization 域已作为机构养老 staff 主档的真实归属来源；机构详情员工 tab 只展示匹配当前 organizationId 的员工。home 协同人员在 partners 切片落地前可保持空归属，不回退本地 mock。
- 任务中心与排班页尚未接通时，必须显式维持当前边界，不能暗中混回本地 staff mock。
