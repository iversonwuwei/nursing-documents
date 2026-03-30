# 核心模块 API 草案

## 目的

用于先沉淀模块边界，再逐步补齐具体接口协议。

## 长者管理

- 老人列表查询
- 老人详情查询
- 入住登记与更新
- 探视记录查询

## 健康监测

- 健康指标列表查询
- 单个老人健康趋势查询
- 异常提醒查询
- 健康解释结果查询

## 报警与事件

- 报警列表查询
- 报警详情查询
- 事件记录查询
- 事件复盘结果查询

## 员工协同

- 员工列表查询
- 任务列表查询
- 排班查询与更新

## 资源管理

- 房间与床位查询
- 机构状态查询
- 设备列表与告警查询
- 物资库存与补货建议查询

## AI 运营中心

- AI 推理结果查询
- AI 规则状态查询
- AI 日志与审计查询

## 后续补充建议

1. 每个模块拆成独立文档。
2. 为每条接口补请求示例和响应示例。
3. 标注调用方、兼容性和回滚影响。

## 已拆分文档

- [长者管理 API](/api/elderly-management)
- [健康监测 API](/api/health-monitoring)
- [报警与事件 API](/api/alerts-incidents)
- [员工协同 API](/api/staff-collaboration)
- [机构管理 API](/api/organization-management)
- [房间与床位 API](/api/room-management)
- [设备管理 API](/api/equipment-management)
- [物资管理 API](/api/supply-management)
- [AI 运营中心 API](/api/ai-operations-center)

## 已补充接口级文档

- [长者列表查询](/api/endpoint-elderly-list)
- [长者详情查询](/api/endpoint-elderly-detail)
- [健康监测列表](/api/endpoint-health-monitoring-list)
- [事故详情查询](/api/endpoint-incident-detail)
- [排班更新](/api/endpoint-staff-schedule-update)
- [机构详情查询](/api/endpoint-organization-detail)
- [床位分配摘要](/api/endpoint-room-allocation-summary)
- [设备监控状态](/api/endpoint-equipment-monitoring)
- [低库存摘要](/api/endpoint-supply-low-stock-summary)
- [AI 日志查询](/api/endpoint-ai-logs-query)

## 已补充字段级文档

- [健康监测列表字段说明](/api/field-health-monitoring-list)
- [AI 日志查询字段说明](/api/field-ai-logs-query)
- [事故详情字段说明](/api/field-incident-detail)
- [排班更新字段说明](/api/field-staff-schedule-update)
- [设备监控字段说明](/api/field-equipment-monitoring)

## 已补充契约级文档

- [健康监测列表完整契约](/api/contract-health-monitoring-list)
- [AI 日志查询完整契约](/api/contract-ai-logs-query)
- [排班更新完整契约](/api/contract-staff-schedule-update)
- [事故详情完整契约](/api/contract-incident-detail)
- [机构详情完整契约](/api/contract-organization-detail)
- [设备监控状态完整契约](/api/contract-equipment-monitoring)
- [低库存摘要完整契约](/api/contract-supply-low-stock-summary)

## 已补充治理级文档

- [长者详情查询治理说明](/api/governance-elderly-detail)
- [健康监测列表治理说明](/api/governance-health-monitoring-list)
- [排班更新治理说明](/api/governance-staff-schedule-update)
- [机构详情查询治理说明](/api/governance-organization-detail)
- [床位分配摘要治理说明](/api/governance-room-allocation-summary)
- [设备监控状态治理说明](/api/governance-equipment-monitoring)
- [低库存摘要治理说明](/api/governance-supply-low-stock-summary)
- [AI 日志查询治理说明](/api/governance-ai-logs-query)
