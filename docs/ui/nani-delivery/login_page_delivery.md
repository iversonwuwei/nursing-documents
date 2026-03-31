# 登录页交付说明

## Scope

- Entry page: lib/app/modules/login/login_page.dart
- Affected users: 护工执行端登录用户
- Rollout stage: 第一批页面级治理补齐

## User Impact

- 登录入口补充稳定交互锚点，便于验证账号、密码和提交动作。
- 保持现有 mock 登录规则不变，仍然是非空账号密码即可进入班次。
- 失败时继续停留当前页，并给出阻断提示。

## Data Source

- Controller: LoginController
- Auth source: app/data/services/auth_service.dart
- Route dependency: AppRoutes.login -> AppRoutes.root

## UI States

- Loading state: 提交中按钮文案切换为“登录中...”。
- Empty state: 账号或密码为空时阻断提交并弹出失败提示。
- Error state: 当前仅覆盖空输入失败，不引入新接口错误处理。
- Mobile impact: 表单键值稳定，可在小屏滚动后继续定位提交按钮。

## Health Signals

- Healthy signal: 非空账号密码提交后进入首页。
- Failure signal: 空输入提交后出现“登录失败”提示且不跳转。
- Stable selectors: login-username-input, login-password-input, login-submit-button

## Verification

- flutter analyze
- flutter test
- Widget test covers empty submit blocking and successful sign-in path.

## Rollback

- Revert the stable keys and associated widget tests.
- Authentication behavior returns to the previous unanchored UI without changing route logic.