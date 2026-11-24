# VRR 配了却一直不生效：Proton 游戏的 app_id 不是 `steam_app_*`

!!! info "一句话"
    `niri msg outputs` 里 VRR 永远 `disabled`，但规则明明写了。

## 现象

配置里 `variable-refresh-rate on-demand=true`，窗口规则写了 `match app-id="^steam_app_"`，
游戏跑起来后查输出，`vrr_enabled` 仍然是 False。

## 环境

niri 26.04；外接 DP-9 支持 VRR；游戏通过 Proton + XWayland（xwayland-satellite）运行。

## 排查

先查游戏窗口的真实 app_id，别假设：

```bash
niri msg --json windows | python3 -c "import json,sys;[print(w['app_id']) for w in json.load(sys.stdin)]"
# 实测输出：pbz.exe      ← 不是 steam_app_4115450
niri msg --json outputs | python3 -c "import json,sys;[print(n,o['vrr_enabled']) for n,o in json.load(sys.stdin).items()]"
```

`steam_app_<appid>` 只会出现在 Steam 客户端/冒烟层的窗口上；游戏本体（XWayland）用的是
`<可执行文件名>`，本机是 `pbz.exe`。所以那条规则从建起来那天起**一次都没命中过**。

## 原因

规则没命中 → `VrrRule` 不成立 → 输出的 on-demand VRR 永远不开。

## 解决办法

一条规则覆盖所有 Proton 游戏：

```kdl
window-rule {
    match app-id=r#"^.*\.exe$"#
    variable-refresh-rate true
}
```

热重载即可生效，**游戏不用退**：

```bash
niri validate -c ~/.config/niri/config.kdl && niri msg action load-config-file
```

顺带纠正两个常见误解（源码依据）：

- `variable-refresh-rate true` 解析成 **`VrrRule::On` = 窗口可见即启用，不要求全屏**；
  只有 `"fullscreen-only"` 才是仅在当前全屏窗口时启用。
- XWayland 客户端日志里出现 `creating toplevel … fullscreen: false` 也**不影响** `On` 规则。

## 怎么确认好了

游戏跑着的时候再查一次，应该变成 True：

```bash
niri msg --json outputs | grep -o '"vrr_enabled":[a-z]*'
```

## 参考

- `niri-config/src/window_rule.rs`（`VrrRule` 定义与解析）
- `src/niri.rs` `refresh_on_demand_vrr()`（判定逻辑）
