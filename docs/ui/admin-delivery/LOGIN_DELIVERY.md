# Login Delivery Unit

## Scope

- Entry route: src/app/login/page.tsx
- Affected users: admin 登录用户、认证维护用户
- Rollout stage: 登录页视觉契约修复

## User Impact

- 登录页当前承接账号密码登录、租户选择、认证错误提示和登录成功后的首页跳转。
- 本次修复只对齐登录页结构类名与全局样式契约，恢复卡片、输入框、按钮和辅助提示的视觉样式。
- 保持成功登录后跳转 `/`，失败时展示用户名或密码错误提示；不修改凭证校验、租户列表或认证模式。

## Data Source

- Route type: client auth form
- Primary dependency: next-auth credentials signIn flow
- Downstream dependency: next/navigation router push and refresh

## UI States

- Loading state: 提交登录时以 loading 按钮反馈请求进行中。
- Empty state: 当前账号密码输入为空时依赖原生 required 校验；后续若扩展租户或验证码，应显式补充表单空态说明。
- Error state: 凭证错误时通过 error 文案暴露失败，不应静默停留在原页面。
- Layout state: 登录表单卡片应在可视区域水平与垂直居中；输入区、租户选择、提交按钮和测试账号提示应在同一视觉卡片内完整显示。
- Mobile impact: 登录卡片在手机宽度下应保留安全边距并保持可读、可点击，不能贴边或被顶部偏移挤出首屏。

## Health Signals

- Healthy signal: 登录页能正确触发 credentials 登录，成功后进入首页，失败时稳定反馈错误。
- Failure signal: 登录提交无响应、成功后不跳转、失败时错误提示丢失，或登录卡片未居中/输入控件退化为浏览器默认样式。
- Verification proxy: lint 通过；视觉修复需加浏览器访问 `/login` 的布局检查；行为改动时加 build 与认证人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 打开 `/login`，确认登录卡片在桌面与移动视口居中，账号、租户、密码、提交按钮和测试账号提示均使用设计系统样式；再验证测试账号登录成功后进入 `/`，错误凭证会显示失败提示

## Rollback

- Revert this delivery note and the matching login route/global CSS changes together.
- If regressions appear, fallback is the current credentials login form and redirect behavior, with the visual fix reverted to the previous login layout.
