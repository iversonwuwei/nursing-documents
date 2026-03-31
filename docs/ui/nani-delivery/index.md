# 护工端交付索引

本目录现作为 nursing-nani-app 的统一归档入口，集中管理工程设计、模板、模块清单和页面级交付说明。

## 通用模板

- [护工 APP 工程设计](./app-design.md)
- [护工端前端交付模板](./frontend-delivery-template.md)
- [护工端模块清单](./module-inventory.md)

## 当前状态

- 工程级 Harness Engineering 规范入口仍位于 nursing-nani-app 根目录 AGENTS.md
- 已具备本地前端交付模板，可用于任务流、报警流、登录鉴权和交接班页面改动
- 已落地首批页面级交付单元：
  - [登录页交付说明](./login_page_delivery.md)
  - [首页交付说明](./home_page_delivery.md)
  - [任务中心交付说明](./tasks_page_delivery.md)
  - [报警处理交付说明](./alerts_page_delivery.md)
- 已落地第二批页面级交付单元：
  - [交接班交付说明](./handover_page_delivery.md)
  - [排班页交付说明](./schedule_page_delivery.md)
  - [AI 护理助手交付说明](./ai_assistant_page_delivery.md)
- 已落地第三批页面级交付单元：
  - [重点长者交付说明](./residents_page_delivery.md)
  - [长者详情交付说明](./resident_detail_page_delivery.md)
  - [健康录入交付说明](./health_entry_page_delivery.md)
  - [护理执行交付说明](./care_execution_page_delivery.md)
- 已落地第四批页面级交付单元：
  - [消息中心交付说明](./notifications_page_delivery.md)
  - [健康趋势交付说明](./health_page_delivery.md)
  - [个人页交付说明](./profile_page_delivery.md)
- 已落地第五批页面级交付单元：
  - [报警详情交付说明](./alert_detail_page_delivery.md)
  - [交接详情交付说明](./handoff_detail_page_delivery.md)
- 已具备页面回归用例：登录阻断、首页快捷入口、底部导航、任务筛选、报警筛选、交接详情链路、排班入口、AI 上下文与空态、重点长者链路、长者详情动作、健康录入、护理执行、消息中心、健康趋势、个人页、报警详情、交接详情
- 当前 GetX 已注册路由已全部具备页面级交付说明、稳定锚点与回归用例。
- 当前仅剩 `lib/app/modules/shift_schedule/` 空占位目录未接入路由，不纳入本轮页面治理范围。
- 后续若新增真实页面或接入设备告警等新流程，应继续按同一模板扩展。

## 预期验证门禁

- `flutter analyze`
- `flutter test`
- 涉及关键流程时补充模拟器路径验证
