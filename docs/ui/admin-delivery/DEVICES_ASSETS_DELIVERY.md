# Devices Assets Delivery Unit

## Scope

- Entry route: src/app/devices/assets/page.tsx
- Affected users: 设备台账管理、资产运维、后勤协同用户
- Rollout stage: 第七批设备子路由治理说明

## User Impact

- 设备资产页当前以标准模块页承接台账视图、筛选和资产状态浏览。
- 当前交付单元先固定说明与验证门禁，不修改标准模块配置或展示行为。
- 保持当前 StandardModulePage 渲染路径和资产模块配置不变。

## Data Source

- Route type: standard module wrapper
- Primary source: devicesAssetsPage standard config
- Downstream dependency: shared StandardModulePage component behavior

## UI States

- Loading state: 标准模块页后续若接真实台账数据，需统一遵循标准页加载反馈。
- Empty state: 台账无结果时应保持标准页级空态一致性。
- Error state: 配置、筛选或数据异常时应保留标准页框架并显式失败。
- Mobile impact: 由标准模块页统一承担窄屏布局责任。

## Health Signals

- Healthy signal: devices assets route 稳定复用标准模块配置，和设备台账场景保持一致。
- Failure signal: 标准页配置漂移，导致标题、筛选或列表定义与资产场景不匹配。
- Verification proxy: lint 通过；行为改动时加 build 与标准模块页人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 校验标题、筛选、列表与标准页配置仍匹配设备资产场景

## Rollback

- Revert this delivery note and any future devices assets route changes together.
- If regressions appear, fallback is the previous StandardModulePage config wiring.
