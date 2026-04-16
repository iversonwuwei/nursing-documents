# Admin 顶部导航信息架构重组

## Scope

- scope: 为 nursing-admin-v2 的顶部导航定义稳定的信息架构归属，明确长护险业务、机构养老、居家养老与通用运营模块的边界。
- affected users: 超级管理员、机构管理员、日班运营人员、评定管理人员。
- changed behavior: 顶级导航不再允许同一路由在多个一级菜单下重复出现；长护险相关入口统一收口到单一业务域，场景菜单只保留独立模块。
- dependent systems: nursing-admin-v2 `src/components/layout/TopNavbar.tsx`、`/nursing/services` 总览页、Playwright navbar smoke。
- verification: nursing-documents `npm run docs:build`，nursing-admin-v2 `npm run lint`、`npm run build`、聚焦 navbar smoke。
- rollback: 回退 `TopNavbar.tsx`、相关 smoke 用例和本文件。

## 目标

- 把长护险认定、协同、结算、规则治理相关入口完整收口到 `长护险业务`。
- 让 `机构养老` 与 `居家养老` 保留独立一级入口，但只承载各自场景独有模块。
- 对每个路由建立唯一一级归属，避免用户在多个菜单里反复看到同一入口。
- 将“路由归属”和“角色排序”拆开治理，避免为了排序再次复制菜单数据。

## 用户影响

- 超级管理员仍然可以从一个完整的长护险业务菜单进入认定主链路，同时保留机构与居家两个独立业务视角。
- 机构管理员优先看到 `机构养老`、`居家养老` 和 `长护险业务`，但不会再遇到同一入口在多个一级菜单反复出现。
- 中宽度和手机宽度下，`更多` 菜单和抽屉菜单继续复用相同归属，不会因为响应式折叠引入新的重复项。

## 路由归属表

| 一级导航 | 负责的模块入口 | 不再重复挂载的能力 |
| --- | --- | --- |
| 长护险业务 | `/nursing/services`、`/elderly/import`、`/elderly/checkin`、`/staff/tasks`、`/staff/schedule`、`/nursing/checkin`、`/organizations/partners`、`/financial`、`/analytics/report`、`/nursing/packages`、`/nursing/plans` | 不再让这些长护险链路重复出现在机构养老、居家养老、机构管理 |
| 机构养老 | `/elderly/new`、`/elderly`、`/rooms`、`/health`、`/alerts`、`/elderly/visits`、`/elderly/vitals` | 不再额外挂长护险评定、派案、结算、定点机构协同 |
| 居家养老 | `/elderly/health`、`/elderly/face`、`/staff`、`/staff/new` | 不再额外挂资料导入、派案、结算、稽核、评定中心 |
| 设备与健康 | `/devices`、`/devices/realtime`、`/devices/assets`、`/devices/status`、`/devices/stats`、`/health/bp`、`/health/hr`、`/health/sleep`、`/alerts/history` | 不再重复挂机构养老已接管的健康总览和实时报警 |
| 机构管理 | `/organizations`、`/organizations/new`、`/branch` | 不再重复挂定点机构和房间 |
| 运营分析 | `/analytics`、`/activities`、`/incidents`、`/supplies`、`/ai-assistant` | 保持通用运营入口，不承接长护险主链路 |

## 角色排序原则

- `super_admin`: 首页概览优先，再进入日班工作台、长护险业务与两个场景域。
- `org_admin`: 日班工作台、机构养老、居家养老前置，随后才是长护险业务和通用管理域。
- 排序只影响一级展示顺序，不改变任何路由归属。

## 验收方向

- 在桌面、`更多` 菜单和移动抽屉中，同一路由只能出现在一个一级分组下。
- 长护险业务菜单在桌面 hover 下可完整展示认定受理、执行协同、结算监管和标准治理四段。
- 机构管理员登录后仍能优先看到机构养老、居家养老与长护险业务，而不会因去重损失主要入口。

## 边界说明

- 本次只重构 admin 顶部导航和关联说明文档，不新增业务页面或新路由。
- `/nursing/services` 仍保留场景化总览职责，但不作为重复导航兜底层。
- 若未来确需让一个页面支持多个业务视角，应先新增显式场景参数或独立路由，再考虑导航层扩展。共享页 scene 方案见 [Admin 共享页面场景上下文扩展](/requirements/admin-shared-page-scene-context)。