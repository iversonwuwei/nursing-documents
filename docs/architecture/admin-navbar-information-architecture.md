# Admin 顶部导航信息架构设计

## Scope

- scope: 定义 nursing-admin-v2 顶部导航的领域划分、唯一归属规则、角色排序机制与回滚方式。
- boundaries: 仅覆盖 `TopNavbar.tsx` 中的导航数据结构、角色排序和激活态归属，不改业务页面内部逻辑。
- dependencies: `BASE_NAV_ITEMS`、`getOrderedNavItems(role)`、Playwright navbar smoke、`/nursing/services` 场景化总览。
- failure modes: 同一路由出现在多个一级菜单、角色排序与路由归属耦合、响应式折叠后出现第二套信息架构、机构管理员丢失高频入口。
- verification: nursing-documents `npm run docs:build`，admin `npm run lint`、`npm run build`、聚焦 navbar smoke。
- rollback: 回退 `TopNavbar.tsx`、`e2e/admin-smoke.spec.ts` 与本文件。

## Design Strategy

- 以 `BASE_NAV_ITEMS` 作为唯一导航源，所有桌面、`更多`、移动抽屉都从同一份归属数据渲染。
- 先定义路由归属，再定义角色顺序；不允许通过复制子路由来满足多个角色诉求。
- `长护险业务` 作为业务域菜单，承接认定受理、执行协同、结算监管和标准治理。
- `机构养老` 与 `居家养老` 作为场景域菜单，只承接场景独有页面和主操作入口。
- `设备与健康`、`机构管理`、`运营分析` 作为通用域菜单，仅保留未被前述业务域接管的通用能力。

## Ownership Model

| 路由域 | 一级菜单 | 设计理由 |
| --- | --- | --- |
| 长护险主链路 | 长护险业务 | 用户需要从单一菜单看到完整认定与监管闭环，避免跨菜单拼路径 |
| 院内照护与入住执行 | 机构养老 | 这些入口与院内运营和床位承载强绑定，不应被长护险菜单吞并 |
| 居家协同与档案补充 | 居家养老 | 保留居家侧独立工作台，但不复刻长护险主链路 |
| 设备与指标子域 | 设备与健康 | 只保留设备页、指标页和报警历史，避免与机构养老冲突 |
| 组织主数据 | 机构管理 | 只保留组织主数据和分院治理，不重复挂合作机构与房间 |
| 经营辅助 | 运营分析 | 保持分析、活动、事件、物资、AI 等通用运营入口 |

## State Model

- `BASE_NAV_ITEMS`: 顶级导航与二级导航的唯一配置源。
- `getOrderedNavItems(role)`: 根据 `super_admin` / `org_admin` 输出展示顺序，但不修改归属内容。
- `openDropdown`: 桌面 hover 菜单和 `更多` 菜单的展开状态。
- `visibleCount` 与 `overflowItems`: 响应式计算后仍可直接显示和需要收纳进 `更多` 的一级菜单集合。
- `openGroup`: 手机抽屉中当前展开的一级分组。

## Healthy Signals

- 顶级导航在任意宽度下都遵守相同的一级菜单归属，不会因为进入 `更多` 或抽屉而出现重复项。
- 当前页面的一级激活态可以通过唯一归属直接识别，不需要额外兼容映射。
- 机构管理员在桌面宽度下仍能直接访问机构养老、居家养老和长护险业务，不需要先进入 `更多` 才看到核心域。

## Verification Strategy

- 文档侧通过 `npm run docs:build` 验证索引、相对链接和站点构建正常。
- 前端侧通过 `npm run lint` 和 `npm run build` 验证导航配置与类型闭环。
- 浏览器侧至少覆盖四条 smoke：长护险业务下拉完整性、中宽度 `更多` 菜单、手机抽屉导航、机构管理员角色排序。

## Evolution Rules

- 若未来新增长护险链路页面，默认优先归入 `长护险业务`，除非该页面只服务于机构或居家独有场景。
- 若未来新增场景菜单入口，必须先证明它不与已有一级菜单重复；否则优先通过页面内筛选或总览卡片解决。
- 若未来需要多视角复用同一页面，应先增加显式场景参数或新路由契约，再更新导航归属表。共享页 scene 设计见 [Admin 共享页面场景上下文设计](/architecture/admin-shared-page-scene-context)。

## Residual Risks

- 当前 `居家养老` 仍是基于现有页面能力抽取出的独立模块，不代表已经具备完整的居家专属路由体系。
- 若后续恢复把同一路由挂到多个一级菜单，激活态和 smoke 规则都会重新复杂化。