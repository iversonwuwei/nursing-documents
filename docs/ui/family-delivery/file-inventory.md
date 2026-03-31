# 家属端文件清单

本页用于汇总 nursing-family-app 中当前已纳入交付治理的主页面、次级页面和验证资产。

## 主 Tab 页面

- lib/app/modules/home/home_page.dart
- lib/app/modules/health/health_page.dart
- lib/app/modules/nursing/nursing_page.dart
- lib/app/modules/visit/visit_page.dart
- lib/app/modules/profile/profile_page.dart

## 次级页面

- lib/app/modules/messages/messages_page.dart
- lib/app/modules/bills/bills_page.dart
- lib/app/modules/alert_history/alert_history_page.dart
- lib/app/modules/video_call/video_call_page.dart
- lib/app/modules/ai_today_summary/ai_today_summary_page.dart
- lib/app/modules/ai_visit_assistant/ai_visit_assistant_page.dart
- lib/app/modules/feedback/feedback_page.dart

## 导航与壳层

- lib/app/modules/root/root_page.dart
- lib/app/routes/app_pages.dart
- lib/app/routes/app_routes.dart
- lib/app/family_binding.dart

## 交付说明文档

- ./home_page_delivery.md
- ./health_page_delivery.md
- ./nursing_page_delivery.md
- ./profile_page_delivery.md
- ./messages_page_delivery.md
- ./bills_page_delivery.md
- ./alert_history_delivery.md
- ./visit_page_delivery.md
- ./video_call_delivery.md
- ./ai_today_summary_delivery.md
- ./ai_visit_assistant_delivery.md
- ./feedback_page_delivery.md

## 验证资产

- test/widget_test.dart
- test/messages_page_test.dart
- test/bills_page_test.dart
- test/alert_history_page_test.dart
- test/visit_and_navigation_test.dart
- test/ai_and_feedback_pages_test.dart
- test/main_tab_pages_test.dart

## 当前状态

- 已实现：页面级交付说明、主链路锚点、组件测试与导航测试
- 已验证：`flutter analyze`、`flutter test`
- 残余风险：仍以 Mock 数据为主，服务级健康信号待真实接口接入后补齐
