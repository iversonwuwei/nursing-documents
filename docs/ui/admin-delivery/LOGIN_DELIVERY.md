# Login Delivery Unit

## Scope

- Entry route: src/app/login/page.tsx
- Affected users: admin 登录用户、认证维护用户
- Rollout stage: 第十批高频入口治理说明

## User Impact

- 登录页当前承接账号密码登录、认证错误提示和登录成功后的首页跳转。
- 当前交付单元先固定认证入口职责和验证门禁，不修改凭证校验或 UI 行为。
- 保持成功登录后跳转 `/`，失败时展示用户名或密码错误提示。

## Data Source

- Route type: client auth form
- Primary dependency: next-auth credentials signIn flow
- Downstream dependency: next/navigation router push and refresh

## UI States

- Loading state: 提交登录时以 loading 按钮反馈请求进行中。
- Empty state: 当前账号密码输入为空时依赖原生 required 校验；后续若扩展租户或验证码，应显式补充表单空态说明。
- Error state: 凭证错误时通过 error 文案暴露失败，不应静默停留在原页面。
- Mobile impact: 登录卡片、表单间距和错误提示需要在手机宽度下保持可读与可点击。

## Health Signals

- Healthy signal: 登录页能正确触发 credentials 登录，成功后进入首页，失败时稳定反馈错误。
- Failure signal: 登录提交无响应、成功后不跳转，或失败时错误提示丢失。
- Verification proxy: lint 通过；行为改动时加 build 与认证人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证测试账号登录成功后进入 `/`，错误凭证会显示失败提示

## Rollback

- Revert this delivery note and any future login route changes together.
- If regressions appear, fallback is the current credentials login form and redirect behavior.