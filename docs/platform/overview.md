# 平台专题总览

## 说明

这里集中维护原 admin 工程 `docs/` 目录下的平台级文档。

自 2026-03-30 起，跨工程 Markdown 文档统一维护在当前 `nursing-documents` 工程，后续新增文档请直接放在本仓库 `docs/` 下对应栏目，不再回写到 admin 工程。

## 推荐阅读顺序

1. [平台总体架构](./PLATFORM_ARCHITECTURE.md)
2. [实施蓝图](./IMPLEMENTATION_BLUEPRINT.md)
3. [模块与页面映射](./MODULE_PAGE_MAPPING.md)
4. [数据库设计](./DATABASE_DESIGN.md)
5. [AI Agent 架构](./AI_AGENT_ARCHITECTURE.md)
6. [产品设计](./PRODUCT_DESIGN.md)
7. [设计系统](./DESIGN_SYSTEM.md)
8. [UI 设计规范](./UI-DESIGN-SPEC.md)

## SQL 初稿

- `docs/platform/POSTGRESQL_DDL_CORE.sql`
- `docs/platform/TIMESCALEDB_TIMESERIES.sql`

## 补充归档

- [Admin 初版设计规范归档](./admin-design-system-legacy.md)

## 维护边界

- 平台级架构、设计、数据库、实施蓝图类文档放在 `docs/platform/`
- 需求、API、UI、运维、纪要继续放在各自栏目
- 若文档直接服务于某个产品模块，优先落到现有 `requirements/`、`api/`、`ui/` 目录，而不是新增平行目录