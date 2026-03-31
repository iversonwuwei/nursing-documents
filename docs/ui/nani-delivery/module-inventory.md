# 护工端模块清单

本页用于汇总 nursing-nani-app 中当前主要模块与建议下沉顺序，作为后续页面级交付治理入口。

## 认证与壳层

- lib/app/modules/login/login_page.dart
- lib/app/modules/root/root_page.dart

## 主业务模块

- lib/app/modules/home/home_page.dart
- lib/app/modules/tasks/tasks_page.dart
- lib/app/modules/alerts/alerts_page.dart
- lib/app/modules/ai_assistant/ai_assistant_page.dart
- lib/app/modules/profile/profile_page.dart

## 次级模块

- lib/app/modules/alert_detail/alert_detail_page.dart
- lib/app/modules/notifications/notifications_page.dart
- lib/app/modules/health/health_page.dart
- lib/app/modules/health_entry/health_entry_page.dart
- lib/app/modules/care_execution/care_execution_page.dart
- lib/app/modules/residents/residents_page.dart
- lib/app/modules/elder_detail/elder_detail_page.dart
- lib/app/modules/handoff/handoff_page.dart
- lib/app/modules/handover/handover_page.dart
- lib/app/modules/schedule/schedule_page.dart

## 建议优先级

- 第一批：login、home、tasks、alerts
- 第二批：alert_detail、care_execution、handoff、schedule
- 第三批：ai_assistant、profile、notifications、health 系列

## 当前状态

- 已实现：工程级规范入口、前端交付模板、本地交付索引、当前全部已注册路由的页面级交付说明
- 已完成首批页面：login、home、tasks、alerts
- 已完成第二批页面：handover、schedule、ai_assistant
- 已完成第三批页面：residents、elder_detail、health_entry、care_execution
- 已完成第四批页面：notifications、health、profile
- 已完成第五批页面：alert_detail、handoff
- 当前 AppPages 已注册路由已全部覆盖；`lib/app/modules/shift_schedule/` 为空占位目录，尚未接入路由
- 默认门禁：`flutter analyze`、`flutter test`
