# 页面级文档：排班管理页

## 交付约束

- scope: 收敛 `/staff/schedule` 在 live 模式下的数据展示，只保留真实派案、模板复核和覆盖统计，不再直接渲染前端本地 AI 排班摘要。
- affected users: 机构管理员、排班运营、值班主管。
- changed behavior: `NEXT_PUBLIC_NURSING_WORKFLOW_MODE=bff` 且派案板同步成功时，页面只展示 Care Service -> Admin BFF -> Admin 前端返回的真实派案和确定性摘要；本地 AI 排班摘要仅保留在 demo 模式或 fallback 路径。
- dependent systems: Nursing workflow BFF、Care Service、排班页前端路由、AI 运营中心跳转入口。
- verification: 文档构建通过；前端文件级 lint 通过；浏览器运行态确认 live 模式下不再出现“AI 排班摘要 / AI 调整建议”的本地摘要区块，只保留真实派案摘要、联动总览和排班表格。
- rollback: 恢复 `/staff/schedule` 直接调用前端本地 `getScheduleAiInsights()` / `getScheduleAiNarratives()` 的渲染逻辑，不涉及后端接口或数据库结构回滚。

## 页面目标

支持主管查看班次分布、覆盖风险和排班调整；在 live 模式下，页面优先表达真实派案看板，而不是本地推演出来的 AI 建议。

## 主要区块

- 班次统计卡片
- 派案总览卡片
- 认定联动总览
- 排班表格
- 调整入口与 AI 运营中心跳转

## 数据来源

- 排班列表接口：护理工作流派案板
- 模板复核与重点计划：护理工作流派生摘要
- 排班更新接口：护理工作流动作接口
- AI 运营入口：独立 AI 页面，不在 live 模式下直接把本地 AI 摘要嵌回排班页

## 状态设计

- 加载态: 派案板同步中，展示 Loading 卡片
- 空态: 当前无排班记录或无重点计划，保留真实空态提示
- 错误态: 派案同步失败，展示 Workflow Error 卡片
- 移动端: 通过横向滚动表格保留班次矩阵，不把统计卡片和总览卡片折成不可读摘要

## 关键动作

- 查看班次覆盖与待复核模板
- 查看休假和补位情况
- 跳转 AI 运营中心继续看 AI 解释或治理信息
