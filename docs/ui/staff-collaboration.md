# 员工协同页面说明

## 页面范围

- 员工列表页
- 员工详情页
- 任务中心
- 排班管理页

## 页面目标

统一查看人员结构、任务优先级和班次覆盖，帮助主管做人工协调。

## 数据来源

- 员工列表接口与员工详情接口
- 员工新建与确认入职接口
- 任务列表接口
- 排班接口
- 覆盖风险摘要接口

## UI 状态

- 加载态: 列表首屏、新建提交、确认入职和详情查询都要显式反馈
- 空态: 无员工、无任务或无排班数据
- 错误态: 员工接口不可用、排班更新失败、覆盖摘要加载失败

## 关键交互

- 列表筛选部门、状态和岗位
- 从新建页提交员工并进入待入职闭环
- 在列表页人工确认待入职员工
- 从员工页跳任务和排班页
- 主管查看 AI 覆盖建议与交接提醒

## AI 边界

- 可提示覆盖风险、优先级和交接建议
- 不直接改写责任人、排班或绩效结果

## 当前交付边界

- scope: 本轮只真实化员工列表、新建、确认入职和员工详情。
- dependent systems: Admin BFF、Staffing Service、admin `/staff`、`/staff/new`、`/staff/[id]`。
- verification: `npm run docs:build`、`dotnet build nursing-backend-services.slnx`、admin `npm run lint && npm run build`。
- rollback: 回退 staff API、Admin BFF 代理和 admin staff 页面 live 化实现，恢复 resource workflow 版本。
