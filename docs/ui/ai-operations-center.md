# AI 运营中心页面说明

## 交付约束

- scope: 收敛 `/ai-assistant` 根页在 live 模式下的数据源，并把即时问答从根页拆到 `/ai-assistant/qa`；前端不再在总览页自动回退本地 mock AI 总览或本地 mock 问答。
- affected users: 平台管理员、机构管理员、运营人员、审计人员。
- changed behavior: `NEXT_PUBLIC_ADMIN_AI_MODE=bff` 时，`/ai-assistant` 只展示真实 AI 总览与分流入口；即时问答移到 `/ai-assistant/qa` 调真实接口。若接口失败，前端展示 unavailable 提示和错误信息，而不是在根页伪造本地问答或本地总览。demo 模式下问答页仍保留前端 mock 以支持离线演示。
- dependent systems: `/api/ai/*` Next 代理、Admin BFF AI 路由、AiOrchestration 服务、AI 日志页、AI 问答页。
- verification: 浏览器运行态确认根页不再承载问答面板，`/ai-assistant/qa` 初始回答为 live 占位文案而非本地 mock，点击预设问题后返回服务端回答；文档构建通过；前端文件级 lint 通过。
- rollback: 恢复 `/ai-assistant` 根页承载问答面板的旧结构，并回退 `/ai-assistant/qa` 新路由，不涉及后端接口或数据库结构回滚。

## 页面范围

- AI 总览页
- AI 问答页
- AI 推理页
- AI 规则治理页
- AI 日志审计页

## 页面目标

围绕来源上下文、对象和关注点，集中展示 AI 解释、治理与留痕。

## 数据来源

- AI 总览接口
- 推理结果接口
- 规则治理接口
- 日志审计接口
- 各业务页上下文字段

## UI 状态

- 加载态: 模型状态、日志列表和规则卡片骨架
- 空态: 当前对象无 AI 留痕或无可用规则
- 错误态: 日志过滤失败、规则切换失败

## 关键交互

- 从业务页携带 source、focus、entity 进入 AI 中心
- 在问答、推理、规则、日志页间保留同一上下文
- 查看证据卡、规则卡和回滚说明
- 即时问答只在问答页内完成，根页只负责入口分流

## AI 边界

- 强调 AI 先建议、人再确认
- 高风险动作必须保留人工复核和回退路径
- live 模式下即使后端当前返回联调期确定性结果，也应视为服务端链路结果，而不是前端本地 mock
