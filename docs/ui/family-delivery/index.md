# 家属端交付索引

本目录现作为 nursing-family-app 的统一归档入口，集中管理工程设计、模板、文件清单和页面级交付说明，作为后续实现、评审和回滚参考入口。

## 通用模板

- [家属 APP 工程设计](./app-design.md)
- [家属端前端交付模板](./frontend-delivery-template.md)
- [家属端文件清单](./file-inventory.md)

## 已完成页面交付说明

- [首页交付说明](./home_page_delivery.md)
- [健康页交付说明](./health_page_delivery.md)
- [护理页交付说明](./nursing_page_delivery.md)
- [我的页交付说明](./profile_page_delivery.md)
- [消息页交付说明](./messages_page_delivery.md)
- [账单页交付说明](./bills_page_delivery.md)
- [报警历史页交付说明](./alert_history_delivery.md)
- [探视页交付说明](./visit_page_delivery.md)
- [视频入口与导航链路交付说明](./video_call_delivery.md)
- [AI 今日摘要页交付说明](./ai_today_summary_delivery.md)
- [AI 探视助手页交付说明](./ai_visit_assistant_delivery.md)
- [护理反馈页交付说明](./feedback_page_delivery.md)

## 当前覆盖范围

- 页面级：首页、健康、护理、我的、消息、账单、报警历史、探视、视频入口、AI 今日摘要、AI 探视助手、护理反馈
- 链路级：首页快捷入口、探视页次级入口、我的服务入口
- 验证级：静态检查、组件测试、空态验证、筛选/切换/导航行为验证

## 尚未覆盖

- 真实 API、BFF、支付、视频房间、推送和告警回执等服务级健康信号
- 需要服务接入后再补的运行态指标、失败恢复和审计链路
