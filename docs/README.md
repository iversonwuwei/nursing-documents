---
layout: home

hero:
  name: Nursing Documents
  text: 养老护理项目文档中台
  tagline: 以模块化需求沉淀、可预览文档站和可回滚发布说明为核心的内部知识库
  actions:
    - theme: brand
      text: 打开文档索引
      link: /doc-index
    - theme: alt
      text: 查看需求模块
      link: /requirements/project-overview
    - theme: alt
      text: 查看模板说明
      link: /templates

features:
  - title: 模块化需求
    details: 已按长者、健康、报警、员工、房间、设备、物资、AI 中心拆成独立模块文档，并补充 SaaS 业务模块订阅、报警/财务/通知三服务设计专题。
  - title: 平台专题归档
    details: 原 admin 工程 docs 已整体迁入当前站点，平台架构、设计和数据库资料统一在这里维护。
  - title: 可预览站点
    details: 基于 VitePress，支持本地预览、静态构建和统一导航检索。
  - title: 发布可治理
    details: 运维、回滚、会议纪要和模板都在同一套知识库中闭环维护。
---

# Nursing Documents

## 当前站点覆盖

- 需求：项目概览与 9 个核心模块文档
- 架构：系统概览
- 平台专题：8 份平台级架构/设计文档与 2 份 SQL 初稿
- API：核心模块边界草案、9 份模块 API 草案、10 份接口级文档、5 份字段级文档、7 份完整契约文档与 8 份治理级文档
- UI：管理端总览、9 份模块页面说明、11 份页面级文档、5 份流程级文档、10 份测试验收文档与 8 份实施级文档
- 运维：发布运行手册
- 纪要：项目启动记录

## 推荐使用顺序

1. 先从 [文档索引](/doc-index) 看全局结构。
2. 再进入 [平台专题](/platform/overview) 查看平台级架构、设计和数据库基线。
3. 然后进入 [需求模块](/requirements/project-overview) 补齐各业务域说明。
4. 最后用 [模板说明](/templates) 复制模板，继续扩展到更细粒度文档。

## 维护约定

- 自 2026-03-30 起，跨工程 Markdown 文档统一维护在当前 nursing-documents 工程。
- admin 工程不再保留独立 docs 目录，后续新增文档请放入本仓库 docs 下对应栏目。

## 核心模块快捷入口

- [长者管理](/requirements/elderly-management)
- [健康监测](/requirements/health-monitoring)
- [报警与事件](/requirements/alerts-incidents)
- [Admin SaaS 业务模块订阅](/requirements/admin-saas-billable-modules)
- [Admin 报警财务通知补齐](/requirements/admin-alert-finance-notification-gap-closure)
- [员工协同](/requirements/staff-collaboration)
- [房间与床位](/requirements/room-management)
- [设备管理](/requirements/equipment-management)
- [物资管理](/requirements/supply-management)
- [AI 运营中心](/requirements/ai-operations-center)
