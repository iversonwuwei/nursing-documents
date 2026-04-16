# 实施级文档：Dashboard 首页

## 范围

覆盖 Dashboard 首页的前端实施拆分、Admin BFF 聚合顺序、空态与降级策略。

## 用户影响

- 管理员通过首页快速查看真实运营态势，而不是演示数字
- 机构负责人通过告警、通知、财务和护理执行卡片判断优先事项

## 页面入口

- 路由: `/`
- 入口来源: 顶部导航默认首页
- 发布阶段: 全量用户默认入口

## 组件拆分

- 顶部概览区: 核心 KPI 卡片
- 风险与告警区: 告警模块分布、待处理清单
- 运营摘要区: 通知、财务、护理工作流摘要
- 执行表现区: 护理任务驱动的员工排行

## 数据来源

| 区块 | 数据源 | 加载策略 | 降级策略 |
| --- | --- | --- | --- |
| KPI 卡片 | `GET /api/admin/dashboard/overview` | 首屏单次请求 | 显示加载/错误文案，不显示静态快照 |
| 告警与运营摘要 | `GET /api/admin/dashboard/overview` | 与 KPI 同一快照 | 分块显示 `0` 或 unavailable |
| 员工排行 | `GET /api/admin/dashboard/overview` | 与 KPI 同一快照 | 列表为空时显示“暂无任务排行” |

## 状态设计

- 加载态: 首屏显示“正在同步后端聚合数据...”提示
- 空态: 新机构或无任务时显示 `0` 和明确空文案
- 错误态: 聚合失败时展示错误说明，不阻断页面 header 与骨架布局
- 移动端: 卡片改为单列堆叠，排行表格保持横向滚动

## 事件与埋点

- `dashboard_loaded`
- `dashboard_alert_card_clicked`
- `dashboard_ai_entry_clicked`
- `dashboard_resource_card_clicked`

## 发布门禁

- 首屏核心卡片 2 秒内可见
- KPI、告警、通知、财务和护理排行均来自同一个聚合快照
- 聚合失败时页面不显示旧静态数组
- `/analytics` 与 `/data-dashboard` 展示同一份快照数据

## 回滚方案

1. 移除 `GET /api/admin/dashboard/overview` 聚合路由。
2. Dashboard 页面恢复当前静态数组实现。
3. 保持 Dashboard 作为导航首页不变。
