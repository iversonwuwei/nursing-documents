# Admin SaaS 业务模块架构设计

## Scope

- scope: 为 nursing-admin-v2 设计可按业务模块订阅的 entitlement 解释层、导航分组规则和首页订阅视图。
- boundaries: 覆盖 `saas-config`、session runtime、顶部导航、Dashboard 首页；不修改 backend tenant service 的真实 API 实现。
- dependencies: NextAuth JWT/session、tenant descriptor、顶部导航单一配置源、Dashboard 首页。
- failure modes: entitlement 仍然是粗粒度 admin/family/nani 导致无法按模块收费；导航与合同模块名不一致；场景入口与收费模块交叉重复。
- verification: nursing-documents `npm run docs:build`；nursing-admin-v2 `npm run lint`、`npm run build`。
- rollback: 回退业务模块目录、entitlement 映射函数、首页订阅卡片与导航模块过滤。

## 设计原则

- 顶层只表达可收费业务模块，不再把机构养老与居家养老当作唯一一级收费单元。
- 场景仍然保留，但通过 `scene=institutional|home` 继续在模块内部承接，不复制路由。
- entitlement 解释层先兼容旧值，再支持新值，避免 backend tenant service 一次性同步升级成为前置阻塞。

## 运行时模型

### 1. 粗粒度兼容层

- 兼容旧 entitlement：`admin`、`family`、`nani`、`billing`、`ai`。
- 兼容策略：在前端 runtime 中把旧值映射成新的业务模块集合。
- 这样 tenant service 即使还返回旧值，admin 前端仍能按照新模块信息架构运行。

### 2. 业务模块目录

- Dashboard
- 长者照护
- 健康设备
- 报警服务
- 评定与长护险
- 财务服务
- 通知服务
- 机构协同
- 运营分析
- AI 运营

每个目录同时携带：

- 模块标签
- 模块说明
- 默认入口
- 收费口径提示

### 3. UI 可见性

- 顶部导航根据归一化后的业务模块列表决定是否显示一级菜单。
- 首页展示当前租户已订阅模块，用于把套餐信息显式暴露给管理员。
- 右上角快捷通知入口等附属入口，也需要遵守对应模块是否开通。
- 业务模块主入口页需要复用统一的 entitlement gate，在租户未开通时返回只读禁用态，而不是继续渲染完整工作台。

### 4. Navbar Overflow Accessibility

- `更多` 菜单属于中宽屏主导航的关键降级路径，不能只支持鼠标悬停。
- 需要支持：
  - `Enter` / `Space` / `ArrowDown` 打开并进入菜单
  - `ArrowUp` / `ArrowDown` 在条目间移动
  - `Home` / `End` 跳到首尾
  - `PageUp` / `PageDown` 按批次移动
  - `Escape` 关闭并把焦点返回触发按钮
- 菜单打开后应尽量让当前激活项进入可视区。

### 5. Visual Regression Gate

- `更多` 菜单的滚动渐隐提示属于视觉型健康信号，不能只靠 DOM 断言。
- Playwright smoke 需要对 overflow dropdown 做 locator 级截图断言，确保滚动提示和版式稳定。
- 导航视觉门禁应继续扩展到：
  - 桌面一级悬浮菜单，用于保护分区标题、业务入口排序和 hover 下拉容器。
  - 移动抽屉主面板，用于保护窄屏降级路径下的导航可读性和层级结构。
  - 顶部右侧动作区，用于保护通知快捷入口、系统设置入口、分隔线与用户信息区的组合布局。
  - 顶部身份信息区，用于保护用户名、租户名、模块数和角色标签在不同 session 组合下的呈现稳定性。
  - 顶部头像退出入口，用于保护用户从当前 session 快速回到登录页的可用性和可访问名称。
- 对容易受滚动条、焦点描边或原生阴影影响的区域，测试应优先采用页面裁剪截图或稳定化样式，而不是依赖浏览器连续稳定元素截图。

## 路由归属规则

- 场景优先的页面继续保留 `scene` 查询参数。
- 一级收费模块负责“入口归属”，场景负责“进入后看哪个版本”。
- 例如：
  - 长者照护模块中同时放机构养老与居家养老入口。
  - 财务服务只负责财务入口本身，不再混在长护险业务下面作为纯后置节点。
  - 通知服务成为独立模块，而不是运营分析下的一个列表入口。

## 健康信号

- 顶部导航一级菜单能直接对齐报价单上的模块名称。
- tenant-private 这类未开通模块的租户，在导航中不应再看到对应模块。
- Dashboard 首页能直观看到当前租户开通模块数量和套餐名。
- 未开通模块直接访问主入口页时，看到统一的 `Entitlement Off` 禁用态，而不是进入真实工作流页面。
- 中宽屏 `更多` 菜单既能鼠标滚动，也能键盘操作，并通过截图回归保持视觉稳定。
- 桌面一级悬浮菜单与移动抽屉都应进入截图回归面，确保桌面 hover 路径与手机抽屉路径在视觉上保持稳定。
- 右侧动作区应在“通知入口显示”和“通知入口隐藏”两种 entitlement 组合下都保持稳定，避免图标按钮间距、分隔线和用户信息错位。
- 顶部身份信息区应在 super admin 和 org admin 等角色切换下保持稳定，避免角色文案、模块数变化或租户名称长度导致用户摘要错位。
- 顶部头像退出入口应保持稳定的点击行为和可访问名称，避免 session 摘要区域重构后退出能力丢失。

## 残余风险

- 本轮先完成 entitlement 解释层与 admin UI 结构，页面级 100% 禁用态不会一次性全部补齐。
- 后续 backend tenant service 若返回新的业务模块列表，应以当前前端目录作为标准输入，逐步移除 legacy 映射。
