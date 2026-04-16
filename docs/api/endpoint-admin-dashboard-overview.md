# 接口级文档：Admin Dashboard 聚合总览

## 基本信息

- 方法: `GET`
- 路径: `/api/admin/dashboard/overview`
- 说明: 由 Admin BFF 组合长者、租户、账单、通知、告警和护理工作流摘要，返回 Dashboard 首页首屏所需的最小实时快照。

## 请求参数

无。

## 响应字段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| generatedAtUtc | string | 聚合生成时间 |
| kpis | object | 首页 KPI 区块 |
| kpis.elderCount | number | 当前长者总数 |
| kpis.tenantCount | number | 当前租户总数 |
| kpis.pendingAlerts | number | 待处理告警总数 |
| kpis.workflowPendingCount | number | 护理工作流待处理总数 |
| alertModules | array | 告警模块摘要列表 |
| notificationBreakdown | array | 通知投递状态摘要 |
| financeBreakdown | array | 财务状态摘要 |
| workflowBreakdown | array | 护理工作流状态摘要 |
| staffLeaderboard | array | 基于护理任务的员工执行排行 |

## 响应示例

```json
{
  "generatedAtUtc": "2026-04-14T09:30:00Z",
  "kpis": {
    "elderCount": 3,
    "tenantCount": 2,
    "pendingAlerts": 4,
    "workflowPendingCount": 5
  },
  "alertModules": [
    {
      "label": "bed_exit",
      "pending": 1,
      "processing": 1,
      "resolved": 3,
      "critical": 1,
      "totalOpen": 2
    }
  ],
  "notificationBreakdown": [
    { "label": "待发送", "value": 2 },
    { "label": "已送达", "value": 8 },
    { "label": "失败", "value": 1 }
  ],
  "financeBreakdown": [
    { "label": "待复核", "value": 2 },
    { "label": "已开票", "value": 5 },
    { "label": "已逾期", "value": 1 }
  ],
  "workflowBreakdown": [
    { "label": "待复核计划", "value": 2 },
    { "label": "未分配计划", "value": 1 },
    { "label": "已完成任务", "value": 12 }
  ],
  "staffLeaderboard": [
    {
      "name": "李护士",
      "role": "护士",
      "tasks": 6,
      "completed": 6,
      "completionRate": 100,
      "trend": "up"
    }
  ]
}
```

## 错误码

- `401`: 未登录或租户上下文不可用
- `502`: 下游服务不可达或聚合失败

## 调用页面

- Dashboard 首页
- `/analytics` 兼容总览页

## 兼容性说明

- 该接口为新的只读聚合路由，不替换现有分项摘要接口。
- 前端应把缺失区块视为 unavailable，而不是退回静态数组。