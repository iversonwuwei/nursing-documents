# 字段级文档：排班更新

## 请求体字段

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| staffId | string | 是 | 员工 ID |
| shift | string | 是 | 班次，建议值为 `早班`、`中班`、`夜班`、`休息` |
| date | string | 是 | 排班日期 |
| remark | string | 否 | 调整原因或补位说明 |

## 响应字段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| success | boolean | 是否更新成功 |
| scheduleId | string | 排班记录 ID |
| updatedAt | string | 更新时间 |
| conflictHint | string | 冲突说明，成功时可为空 |

## 校验规则

- 同一员工同一天只能存在一条有效排班记录。
- `shift` 不能填写未定义班次。
- 若为补位调整，`remark` 建议必填。

## 失败场景

- 班次冲突
- 员工不存在
- 已超过允许修改时间窗
- 后端保存失败
