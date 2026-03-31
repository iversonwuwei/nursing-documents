# Workspace 交付治理总览

本页汇总整个 workspace 中四个工程当前的 Harness Engineering 落地入口、文件清单和验证状态，作为统一审阅面。

## 工程入口

### nursing-family-app

- 工程规范入口：family app 根目录 AGENTS.md
- 交付模板：nursing-documents/docs/ui/family-delivery/frontend-delivery-template.md
- 交付索引：nursing-documents/docs/ui/family-delivery/index.md
- 文件清单：nursing-documents/docs/ui/family-delivery/file-inventory.md
- 当前状态：本地 docs Markdown 已迁移到 nursing-documents，页面级说明、链路测试和主 Tab 或次级页治理已完成

### nursing-nani-app

- 工程规范入口：nani app 根目录 AGENTS.md
- 交付模板：nursing-documents/docs/ui/nani-delivery/frontend-delivery-template.md
- 交付索引：nursing-documents/docs/ui/nani-delivery/index.md
- 文件清单：nursing-documents/docs/ui/nani-delivery/module-inventory.md
- 当前状态：本地 docs Markdown 已迁移到 nursing-documents，当前已注册路由的页面级说明已归档完成

### nursing-admin-v2

- 工程规范入口：admin 根目录 AGENTS.md
- 交付模板：仓库根目录 FRONTEND_DELIVERY_TEMPLATE.md
- 交付索引：仓库根目录 DELIVERY_INDEX.md
- 路由清单：仓库根目录 DELIVERY_ROUTE_INVENTORY.md
- 当前状态：工程级模板、索引和全路由清单已完成，路由级说明待逐路由下沉

### nursing-documents

- 工程规范入口：documents 根目录 AGENTS.md
- 模板入口：[模板说明](/templates)
- 文档索引：[文档索引](/doc-index)
- 当前状态：模板体系、文档站和 workspace 治理总览已完成

## 当前验证状态

- family app：已验证 `flutter analyze` 和 `flutter test`
- nani app：当前工程默认门禁为 `flutter analyze` 和 `flutter test`，本轮未新增代码改动
- admin：当前工程默认门禁为 `npm run lint` 与 `npm run build`，本轮未新增业务代码改动
- documents：已验证 `npm run docs:build`

## 统一执行规则

- 所有工程默认按 Harness Engineering 范式执行
- 每次行为变更必须显式描述范围、用户影响、数据来源、健康信号、验证门禁和回滚路径
- 新增依赖、脚本或服务接入必须说明来源、用途、风险和最小必要性
- 真实服务接入后，需要在现有页面级治理基础上补服务级健康信号和失败恢复验证
