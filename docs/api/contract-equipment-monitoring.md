# 完整契约文档：设备监控状态查询

## 契约目标

确保设备监控页、设备状态页和 AI 巡检摘要使用同一套设备实时状态数据。

## 接口定义

- 方法: `GET`
- 路径: `/api/equipment/monitoring`
- 调用方: 设备监控页、设备状态页、AI 推理页

## 请求契约

| 字段 | 类型 | 必填 | 约束 | 说明 |
| --- | --- | --- | --- | --- |
| status | string | 否 | 枚举 | `online`、`warning`、`offline` |
| category | string | 否 | 文本 | 设备分类 |
| keyword | string | 否 | 最长 50 字符 | 设备名或位置关键词 |

## 响应契约

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| items | array | 是 | 设备监控列表 |
| items[].id | string | 是 | 设备 ID |
| items[].name | string | 是 | 设备名 |
| items[].room | string | 是 | 位置 |
| items[].status | string | 是 | 设备状态 |
| items[].signal | number | 是 | 信号百分比 |
| items[].battery | number | 是 | 电量百分比 |
| items[].lastAlert | string \| null | 否 | 最近告警时间 |

## 状态流

1. 页面发起监控查询。
2. 服务按状态/分类/关键词过滤。
3. 结果返回后页面高亮异常设备。
4. AI 摘要按状态、信号和电量组合做优先级解释。

## 兼容性策略

- `signal`、`battery` 范围固定为 0-100。
- `status` 枚举新增前需同步前端状态映射。
- `lastAlert` 可为空，页面应支持无告警历史。

## 失败模式

- `400`: 查询参数非法
- `500`: 监控查询失败

## 可观测性

- 建议日志字段: `status`、`category`、`keyword`
- 建议指标: 异常设备比例、查询耗时、离线设备数

## 回滚说明

如实时监控数据不稳定，可回退为只读静态设备状态列表，并关闭 AI 巡检优先级解释。
