# AI 运营中心 API 草案

## 模块用途

服务 AI 总览、推理详情、规则治理和日志审计的统一数据查询。

## 建议接口

### 查询 AI 总览

- 方法: `GET`
- 路径: `/api/ai/overview`
- 用途: 返回模型状态、建议数量、场景分布

### 查询推理结果

- 方法: `GET`
- 路径: `/api/ai/inference`
- 用途: 按 source、focus、entity 查询上下文推理结果

### 查询规则治理项

- 方法: `GET`
- 路径: `/api/ai/rules`
- 用途: 返回规则启停、适用范围、回滚说明

### 更新规则状态

- 方法: `PUT`
- 路径: `/api/ai/rules/:ruleId`
- 用途: 启用或停用规则

### 查询日志审计

- 方法: `GET`
- 路径: `/api/ai/logs`
- 用途: 按 source、focus、entity、channel 精确过滤 AI 日志

## 调用方

- AI 总览页
- AI 推理页
- AI 规则治理页
- AI 日志审计页
- 各业务页 AI 入口跳转链路

## 兼容性

- 上下文字段 source、focus、entityId、entityName 应作为稳定契约。
- 日志查询必须兼容无 AI 数据时的降级显示。

## 回滚说明

如规则更新或日志过滤逻辑异常，可回退到只读 mock 展示和人工治理。
