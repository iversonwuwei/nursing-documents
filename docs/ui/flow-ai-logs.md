# 流程级文档：AI 日志审计页交互流程

## 场景目标

围绕 source、focus、entity 做精确日志过滤，并在推理、规则、日志之间保持同一上下文。

## 主流程

1. 用户从业务页进入 AI 日志页。
2. 页面读取 query 参数并展示当前上下文卡片。
3. 系统按结构化字段优先查询日志。
4. 用户查看 summary 和 outcome。
5. 用户切换到推理页或规则页继续追查。

## 分支流程

### 无结构化命中

1. 系统回退到 channel 或 keyword 启发式过滤。
2. 页面提示当前为兜底匹配结果。

### 无日志结果

1. 页面展示空态。
2. 引导用户返回业务页或切换筛选条件。

## 埋点建议

- `ai_logs_context_loaded`
- `ai_logs_filter_changed`
- `ai_logs_jump_to_rules`
- `ai_logs_jump_to_inference`

## 测试清单

- source、focus、entity 精确过滤正确
- 兜底筛选正常工作
- 在三页间跳转时上下文不丢失
