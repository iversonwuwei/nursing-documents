# 测试与验收文档：AI 日志审计页

## 验收目标

确认结构化过滤、兜底过滤、上下文展示和跨页跳转稳定可用。

## 用例矩阵

| 场景 | 前置条件 | 操作 | 预期结果 |
| --- | --- | --- | --- |
| 精确过滤 | 提供 source/focus/entity | 打开日志页 | 命中精确日志 |
| 兜底过滤 | 无结构化命中 | 打开日志页 | 回退到 channel/keyword |
| 空结果 | 无日志数据 | 打开日志页 | 展示空态 |
| 查看 outcome | 日志存在 | 展开/查看日志项 | summary 与 outcome 完整展示 |
| 切换规则/推理页 | 上下文存在 | 点击跳转按钮 | 上下文保留 |

## 异常路径

- 日志接口失败
- source/focus 参数非法
- 返回字段缺失

## 埋点校验

- `ai_logs_context_loaded`
- `ai_logs_filter_changed`
- `ai_logs_jump_to_rules`
- `ai_logs_jump_to_inference`

## 回归清单

- 精确过滤优先级正确
- 兜底匹配可工作
- 空态/错误态清晰
- 三页间上下文不丢失
