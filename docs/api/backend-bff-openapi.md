# Backend 统一 OpenAPI 规范（Phase 1）

## 交付 framing

- scope: 提供基于当前真实 API 清单整理的 phase 1 BFF OpenAPI 规范文件，覆盖内容管理、护理工作流和 AI 三条真实链路。
- affected audience: 后端研发、前端研发、测试、集成与联调人员。
- validation: 说明页与原始 YAML 一并通过 `npm run docs:build`。
- rollback: 回退本文、原始 YAML 文件和索引修改。

## 规范文件

- 原始 YAML: [backend-bff-phase1.yaml](/openapi/backend-bff-phase1.yaml)

## 覆盖范围

本规范当前覆盖：

1. Admin 内容管理 API
2. Admin 护理工作流 API
3. Admin AI 治理与推理 API
4. Nani AI 预览 API
5. Family AI 预览 API

## 兼容性说明

- 这是面向当前已接入后端的真实 API 面的阶段性 OpenAPI，不是对所有未来业务模块的一次性承诺。
- 规范优先描述 BFF 对前端暴露的稳定契约，而不是内部服务互调路径。
- 后续若新增 elders、billing、notifications 等真实 API 接入，应继续在同一规范中追加路径和 schema，而不是另建平行规范。