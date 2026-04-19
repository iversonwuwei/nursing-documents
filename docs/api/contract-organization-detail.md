# 完整契约文档：机构详情查询

## 契约目标

支撑机构详情页与 AI 承接分析读取一致的机构主体、rooms 聚合摘要与员工接入状态。

## 接口定义

- 方法: `GET`
- 路径: `/api/admin/organizations/:organizationId`
- 调用方: 机构列表页、机构详情页、AI 运营中心

## 路径契约

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| organizationId | string | 是 | 机构 ID |

## 响应契约

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| organization.organizationId | string | 是 | 机构 ID |
| organization.name | string | 是 | 机构名称 |
| organization.manager | string | 是 | 负责人 |
| organization.totalBeds | number | 是 | 聚合后的总床位数 |
| organization.occupiedBeds | number | 是 | 聚合后的已入住床位 |
| organization.staffCount | number | 是 | 当前真实已归属的员工数 |
| organization.staffIntegrationStatus | string | 是 | `pending | live` |
| organization.address | string | 是 | 地址 |
| organization.phone | string | 否 | 联系电话 |
| rooms | array | 是 | 当前机构房间摘要列表 |

## 状态流

1. 页面按机构 ID 拉取详情。
2. Admin BFF 聚合 rooms 真实摘要，并显式标记员工接入状态。
3. 页面展示机构卡片、房间台账和承接压力信息。
4. 用户可进入 AI 运营中心查看调配建议。

## 兼容性策略

- `rooms` 明细允许追加清洁、生命周期等字段，但不能删除 `roomId`、`capacity`、`occupied`。
- `manager`、`address` 为高频展示字段，不应随意变更命名。

## 失败模式

- `404`: 机构不存在
- `404`: 机构不存在
- `502`: organization service 或 rooms 聚合失败

## 可观测性

- 建议日志字段: `organizationId`、`occupiedBeds`、`roomCount`、`staffIntegrationStatus`
- 建议指标: 查询成功率、机构摘要加载耗时

## 回滚说明

如聚合字段异常，可回退到基础机构信息与真实空态，但不得恢复本地 organizations mock 数据。
