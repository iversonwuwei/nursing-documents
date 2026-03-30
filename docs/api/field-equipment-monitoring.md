# 字段级文档：设备监控状态查询

## 请求参数字段

| 字段 | 类型 | 必填 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| status | string | 否 | 全部 | 设备状态，建议值为 `online`、`warning`、`offline` |
| category | string | 否 | 全部 | 设备分类 |
| keyword | string | 否 | 空字符串 | 设备名或位置关键词 |

## 响应字段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| items | array | 设备监控对象列表 |
| items[].id | string | 设备 ID |
| items[].name | string | 设备名称 |
| items[].room | string | 所在位置 |
| items[].status | string | 运行状态 |
| items[].signal | number | 信号强度百分比 |
| items[].battery | number | 电量百分比 |
| items[].lastAlert | string \| null | 最近一次告警时间 |

## 字段约束

- `signal` 与 `battery` 建议返回 0 到 100 的整数。
- `lastAlert` 为空表示当前无历史告警。
- `status` 必须与前端颜色映射保持一致。

## 使用说明

- 列表页优先用 `status` 做一级分组。
- AI 巡检摘要会对 `signal`、`battery` 和 `status` 做综合排序。
