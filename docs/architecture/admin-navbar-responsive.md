# Admin 顶部导航响应式设计

## Scope

- scope: 定义 nursing-admin-v2 顶部导航在不同视口宽度下的适配策略、状态管理和回滚路径。
- boundaries: 仅覆盖 `TopNavbar`、`AppWrapper` 和全局样式中的导航壳层，不改业务路由内容。
- dependencies: Next.js App Router、现有 `NAV_ITEMS` 单一导航源、全局 CSS 断点。
- failure modes: 顶栏拥挤但未切换为抽屉、下拉层被导航容器裁切、抽屉打开后无法关闭、路由切换后菜单状态残留、窄屏内容间距异常。
- verification: nursing-documents `npm run docs:build`，admin `npm run lint` 与 `npm run build`。
- rollback: 回退 `TopNavbar.tsx`、`globals.css` 与本文件。

## Design Strategy

- 使用单一导航源 `NAV_ITEMS` 同时驱动桌面横向菜单、桌面“更多”溢出菜单和移动抽屉菜单。
- 顶栏先基于导航区可用宽度计算当前可显示的一级导航数量；放不下的一级导航按顺序并入“更多”菜单。
- 只有在手机级宽度下才进入 full compact 模式，隐藏桌面横向导航并切换为汉堡菜单。
- 抽屉菜单沿用当前分组语义，并根据当前路由自动展开对应分组。

## State Model

- `openDropdown`: 桌面态下控制一级分组和“更多”菜单的下拉状态。
- `mobileOpen`: 手机宽度下控制抽屉开关。
- `visibleCount`: 当前宽度下还能留在横向导航区的一级导航数量。
- `overflowItems`: 当前宽度下被收纳进“更多”菜单的一级导航集合。
- `isMobileCompact`: 是否已经进入手机级抽屉模式。
- `openGroup`: 抽屉内当前展开的一级分组；路由切换或重新打开抽屉时应与当前路径对齐。

## Responsive Rules

- 宽度充足时：显示全部一级导航。
- 宽度开始不足但仍高于手机阈值时：从横向导航末尾开始，把一级导航逐个收纳进“更多”菜单。
- 若当前激活页面对应的一级导航已被收纳，“更多”菜单本身需要暴露激活态。
- 到达手机阈值后：隐藏横向导航，只保留汉堡按钮和抽屉菜单。
- 小屏下继续收紧 logo 文案、页面 padding 和导航右侧间距，优先保证点击目标可达。

## Healthy Signals

- 顶部导航在中屏和窄屏下不出现横向溢出、元素重叠或不可点击区域。
- 随着宽度缩小，“更多”菜单中的项目数量逐步增加，而不是一次性全部消失。
- 处于“更多”菜单中的当前激活导航仍能通过顶栏状态被识别。
- 一级导航和“更多”菜单的下拉层在 hover 时都能完整显示，不会被 `navbar` 或 `navbar-nav` 裁切。
- 抽屉菜单打开后，当前激活分组和子项能正确高亮。
- 切换路由、按下 Escape 或点击遮罩后，抽屉和桌面下拉状态都能正确收起。

## Verification Strategy

- 自动化回归至少验证 1440px 桌面宽度下的一级 hover 下拉。
- 自动化回归至少验证中屏宽度下“更多”菜单存在且 hover 后能展示溢出导航项。
- 自动化回归至少验证手机宽度下只保留汉堡入口，并能通过抽屉进入目标页面。

## Future Extension

- 未来若接权限接口或组织级菜单配置，应继续以 `NAV_ITEMS` 的单一来源模式为目标，把动态数据映射到当前导航结构，而不是并行维护第二套抽屉配置。
- 若后续需要更细的断点治理，可把 compact 计算提升为全局 layout capability，而不是散落在页面组件中。

## Residual Risks

- 当前仍是纯前端静态菜单，不能代表真实权限裁剪和多组织差异化导航。
- 若未来继续增加一级导航分组，compact 切换阈值可能需要重新校准。
