# 完整契约文档：机构详情查询

## 契约目标

支撑机构详情页与 AI 承接分析读取一致的机构主体、床位结构与人员配置摘要。

## 接口定义

- 方法: `GET`
- 路径: `/api/organizations/:organizationId`
- 调用方: 机构列表页、机构详情页、AI 运营中心

## 路径契约

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| organizationId | string | 是 | 机构 ID |

## 响应契约

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| id | string | 是 | 机构 ID |
| name | string | 是 | 机构名称 |
| manager | string | 是 | 负责人 |
| totalBeds | number | 是 | 总床位数 |
| occupiedBeds | number | 是 | 已入住床位 |
| staffCount | number | 是 | 员工数 |
| address | string | 是 | 地址 |
| phone | string | 否 | 联系电话 |
| occupancyRate | number | 否 | 入住率，可后端直接返回 |

## 状态流

1. 页面按机构 ID 拉取详情。
2. 服务聚合床位和人员摘要。
3. 页面展示机构卡片和承接压力信息。
4. 用户可进入 AI 运营中心查看调配建议。

## 兼容性策略

- `occupancyRate` 可新增，但前端需兼容缺失时自行计算。
- `manager`、`address` 为高频展示字段，不应随意变更命名。

## 失败模式

- `404`: 机构不存在
- `500`: 机构详情查询失败

## 可观测性

- 建议日志字段: `organizationId`、`occupiedBeds`、`staffCount`
- 建议指标: 查询成功率、机构摘要加载耗时

## 回滚说明

如聚合字段异常，可回退到基础机构信息，床位和员工摘要改为分接口读取。
