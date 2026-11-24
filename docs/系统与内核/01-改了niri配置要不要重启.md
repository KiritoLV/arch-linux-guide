# 改了 niri 配置，哪些要重登、哪些热重载就够

!!! info "一句话"
    debug 选项是每帧现读的，改完 reload 立刻生效；但输出/显卡拓扑类必须重登。

## 现象

改完 `config.kdl` 不知道要不要退出登录；有时改完没反应，以为是改错了。

## 环境

niri 26.04。

## 排查

源码里 `debug` 选项是在渲染循环里**现读**的：

```rust
// src/backend/tty.rs
let needs_sync = res.needs_sync()
    || self.config.borrow().debug.wait_for_frame_completion_before_queueing;
```

`self.config` 是活配置（`Rc<RefCell<Config>>`），reload 后下一帧就按新值跑。

## 原因

niri 区分两类配置：影响渲染/行为的（可 reload）与影响输出拓扑、DRM 设备占用的（需重登）。

## 解决办法

```bash
niri validate -c ~/.config/niri/config.kdl      # 校验（本身也会触发一次 reload）
niri msg action load-config-file               # 手动 reload
journalctl --user -b | grep "niri_config: loaded" | tail -3   # 确认真的读了
```

- **热重载够**：`debug {}` 里的选项、窗口规则、binds、布局参数、输入设置。
- **必须退出登录重进**：`outputs`（模式/位置/VRR 开关本身）、`render-drm-device`、`ignore-drm-device`。

**顺序铁律**：先改配置 → 再换线 → 最后退出登录重进。反过来会黑屏（目标口拿不到 modeset）。

## 怎么确认好了

`journalctl` 里出现新的 `niri_config: loaded config from …`，且配置校验通过。

## 参考

- niri wiki: Configuration / Debug Options
