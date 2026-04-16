# 接口级文档：长者详情查询

## 基本信息

- 方法: `GET`
- 路径: `/api/admin/elders/:elderId`
- 说明: 查询单个长者的主档摘要，用于 admin 详情页首屏、编辑页回填和机构委托信息回显。

## 路径参数

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| elderId | string | 是 | 长者 ID |

## 响应示例

```json
{
  "elderId": "ELD-1743490800000",
  "tenantId": "tenant-demo",
  "elderName": "张桂英",
  "gender": "女",
  "age": 81,
  "identityCard": "310101194401010022",
  "birthDate": "1944-01-01",
  "elderPhone": "13800001111",
  "careLevel": "一级护理",
  "roomNumber": "201-1",
  "admissionStatus": "AdmissionReviewed",
  "familyContactName": "张敏",
  "familyContactPhone": "13900002222",
  "adlScore": 45,
  "cognitiveLevel": "轻度认知障碍",
  "medicalAlerts": ["高血压", "夜间低氧"],
  "entrustmentType": "政府委托",
  "entrustmentOrganization": "静安区民政局长护险中心",
  "monthlySubsidy": 3200,
  "serviceItems": ["生活照料", "基础护理", "健康监测"],
  "serviceNotes": "按政府托底套餐执行，每周复盘一次。"
}
```

## 错误码

- `404`: 长者不存在
- `500`: 长者详情查询失败

## 字段兼容性

- `gender`、`age`、`identityCard`、`birthDate`、`elderPhone`、`familyContactPhone`、`adlScore`、`cognitiveLevel` 为新增可选字段；旧调用方可忽略，不构成破坏性变更。
- `entrustmentType`、`entrustmentOrganization`、`monthlySubsidy`、`serviceItems`、`serviceNotes` 为增量字段；旧调用方可忽略，不构成破坏性变更。
- `serviceItems` 始终返回数组；旧数据默认返回空数组而不是 `null`。

## 调用页面

- 长者详情页
- 健康监测页详情侧栏
- 长者编辑页
