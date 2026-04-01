# 文档索引

当前站点可从顶部导航进入首页，模板说明见 [模板说明](/templates)。

## Harness 交付模板

- 目录: `templates/`
- 说明: 存放按 Harness Engineering 范式编写的需求、前端、API、发布与回滚模板
- 模板入口: [模板说明](/templates)

## 需求

- 目录: `docs/requirements/`
- 说明: 存放业务需求、范围说明、验收标准
- 初始文档: [护理项目需求概览](/requirements/project-overview)

### 核心模块

- [长者管理](/requirements/elderly-management)
- [健康监测](/requirements/health-monitoring)
- [报警与事件](/requirements/alerts-incidents)
- [员工协同](/requirements/staff-collaboration)
- [机构管理](/requirements/organization-management)
- [房间与床位](/requirements/room-management)
- [设备管理](/requirements/equipment-management)
- [物资管理](/requirements/supply-management)
- [AI 运营中心](/requirements/ai-operations-center)

## 架构

- 目录: `docs/architecture/`
- 说明: 存放系统架构、模块设计、数据流、边界说明
- 初始文档: [系统概览](/architecture/system-overview)

## 平台专题

- 目录: `docs/platform/`
- 说明: 存放从 admin 工程迁移而来的平台级架构、设计、数据库与实施蓝图文档
- 初始文档: [平台专题总览](/platform/overview)

### 迁移文档

- [平台总体架构](/platform/PLATFORM_ARCHITECTURE)
- [实施蓝图](/platform/IMPLEMENTATION_BLUEPRINT)
- [模块与页面映射](/platform/MODULE_PAGE_MAPPING)
- [数据库设计](/platform/DATABASE_DESIGN)
- [AI Agent 架构](/platform/AI_AGENT_ARCHITECTURE)
- [产品设计](/platform/PRODUCT_DESIGN)
- [设计系统](/platform/DESIGN_SYSTEM)
- [UI 设计规范](/platform/UI-DESIGN-SPEC)
- [Admin 初版设计规范归档](/platform/admin-design-system-legacy)

### SQL 资产

- `docs/platform/POSTGRESQL_DDL_CORE.sql`
- `docs/platform/TIMESCALEDB_TIMESERIES.sql`

## API

- 目录: `docs/api/`
- 说明: 存放接口协议、字段定义、示例请求响应
- 初始文档: [核心模块 API 草案](/api/core-modules)

### 模块 API

- [长者管理 API](/api/elderly-management)
- [健康监测 API](/api/health-monitoring)
- [报警与事件 API](/api/alerts-incidents)
- [员工协同 API](/api/staff-collaboration)
- [机构管理 API](/api/organization-management)
- [房间与床位 API](/api/room-management)
- [设备管理 API](/api/equipment-management)
- [物资管理 API](/api/supply-management)
- [AI 运营中心 API](/api/ai-operations-center)

### 接口级文档

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

### 字段级文档

- [健康监测列表字段说明](/api/field-health-monitoring-list)
- [AI 日志查询字段说明](/api/field-ai-logs-query)
- [事故详情字段说明](/api/field-incident-detail)
- [排班更新字段说明](/api/field-staff-schedule-update)
- [设备监控字段说明](/api/field-equipment-monitoring)

### 契约级文档

- [健康监测列表完整契约](/api/contract-health-monitoring-list)
- [AI 日志查询完整契约](/api/contract-ai-logs-query)
- [排班更新完整契约](/api/contract-staff-schedule-update)
- [事故详情完整契约](/api/contract-incident-detail)
- [机构详情完整契约](/api/contract-organization-detail)
- [设备监控状态完整契约](/api/contract-equipment-monitoring)
- [低库存摘要完整契约](/api/contract-supply-low-stock-summary)

### 治理级文档

- [长者详情查询治理说明](/api/governance-elderly-detail)
- [健康监测列表治理说明](/api/governance-health-monitoring-list)
- [排班更新治理说明](/api/governance-staff-schedule-update)
- [机构详情查询治理说明](/api/governance-organization-detail)
- [床位分配摘要治理说明](/api/governance-room-allocation-summary)
- [设备监控状态治理说明](/api/governance-equipment-monitoring)
- [低库存摘要治理说明](/api/governance-supply-low-stock-summary)
- [AI 日志查询治理说明](/api/governance-ai-logs-query)

## UI

- 目录: `docs/ui/`
- 说明: 存放页面说明、交互规则、组件规范、视觉约束
- 初始文档: [管理端页面说明](/ui/admin-overview)

### 模块页面说明

- [长者管理页面](/ui/elderly-management)
- [健康监测页面](/ui/health-monitoring)
- [报警与事件页面](/ui/alerts-incidents)
- [员工协同页面](/ui/staff-collaboration)
- [机构管理页面](/ui/organization-management)
- [房间与床位页面](/ui/room-management)
- [设备管理页面](/ui/equipment-management)
- [物资管理页面](/ui/supply-management)
- [AI 运营中心页面](/ui/ai-operations-center)

### 页面级文档

- [Dashboard 首页](/ui/page-dashboard)
- [长者详情页](/ui/page-elderly-detail)
- [健康监测页](/ui/page-health-monitoring)
- [报警中心页](/ui/page-alerts-center)
- [事故详情页](/ui/page-incident-detail)
- [排班管理页](/ui/page-staff-schedule)
- [机构列表页](/ui/page-organizations-overview)
- [房间列表页](/ui/page-rooms-overview)
- [设备详情页](/ui/page-equipment-detail)
- [物资列表页](/ui/page-supplies-overview)
- [AI 日志审计页](/ui/page-ai-logs)

### Admin 路由交付归档

- [Admin 路由交付总览](/ui/admin-delivery/)
- 说明: 统一归档 nursing-admin-v2 的页面级与路由级交付文档，作为跨工程检索和历史记录入口

### Family 交付归档

- [Family 家属端交付总览](/ui/family-delivery/)
- 说明: 统一归档 nursing-family-app 的工程设计、模板、文件清单与页面级交付文档

### Nani 交付归档

- [Nani 护工端交付总览](/ui/nani-delivery/)
- 说明: 统一归档 nursing-nani-app 的工程设计、模板、模块清单与页面级交付文档

### 流程级文档

- [管理端新建数据闭环流程](/ui/flow-admin-create-data)
- [健康监测页交互流程](/ui/flow-health-monitoring)
- [报警中心页交互流程](/ui/flow-alerts-center)
- [排班管理页交互流程](/ui/flow-staff-schedule)
- [设备详情页交互流程](/ui/flow-equipment-detail)
- [AI 日志审计页交互流程](/ui/flow-ai-logs)

### 测试与验收文档

- [健康监测页测试与验收](/ui/test-health-monitoring)
- [报警中心页测试与验收](/ui/test-alerts-center)
- [排班管理页测试与验收](/ui/test-staff-schedule)
- [设备详情页测试与验收](/ui/test-equipment-detail)
- [AI 日志审计页测试与验收](/ui/test-ai-logs)
- [Dashboard 首页测试与验收](/ui/test-dashboard)
- [长者详情页测试与验收](/ui/test-elderly-detail)
- [机构列表页测试与验收](/ui/test-organizations-overview)
- [房间列表页测试与验收](/ui/test-rooms-overview)
- [物资列表页测试与验收](/ui/test-supplies-overview)

### 实施级文档

- [管理端新建数据闭环实施说明](/ui/implementation-admin-create-data)
- [Dashboard 首页实施说明](/ui/implementation-dashboard)
- [长者详情页实施说明](/ui/implementation-elderly-detail)
- [健康监测页实施说明](/ui/implementation-health-monitoring)
- [报警中心页实施说明](/ui/implementation-alerts-center)
- [设备详情页实施说明](/ui/implementation-equipment-detail)
- [房间列表页实施说明](/ui/implementation-rooms-overview)
- [物资列表页实施说明](/ui/implementation-supplies-overview)
- [AI 日志审计页实施说明](/ui/implementation-ai-logs)

## 运维

- 目录: `docs/operations/`
- 说明: 存放部署流程、发布门禁、回滚方案、巡检手册
- 初始文档: [发布运行手册](/operations/release-runbook)

### 交付治理

- [Workspace 交付治理总览](/operations/workspace-delivery-governance)

### 历史记录

- [Admin 历史任务日志](/operations/admin-task-log)

## 会议纪要

- 目录: `docs/meeting-notes/`
- 说明: 存放评审、同步会、需求澄清会的会议记录
- 初始文档: [项目启动记录](/meeting-notes/2026-03-30-kickoff)
