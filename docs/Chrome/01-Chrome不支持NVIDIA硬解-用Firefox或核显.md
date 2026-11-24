# Chrome 在 NVIDIA 上根本不会硬解：独显直连就换 Firefox，有核显才指定核显

!!! info "一句话"
    Chromium 直接跳过 nvidia-drm，独显上永远是软解；Firefox 才支持 NVIDIA 硬解。

## 现象

`chrome://gpu` 里 `Video Decode: Hardware accelerated`，但
`Video Acceleration Information → Decoding:` **一条都不列**；播放时 CPU 高、风扇转，
`nvidia-smi` 的 NVDEC 占用为 0%。

## 环境

Chrome 158 / Wayland / 混合显卡（4060 + 780M）。

## 排查

```bash
# Chrome 自己的日志直接说了
# WARNING:media/gpu/vaapi/vaapi_wrapper.cc:131 : Should skip nVidia device named: nvidia-drm
```

也就是说：**`Hardware accelerated` 这一行是假象**，判断硬解有没有生效只看 `Decoding:` 列表。

## 原因

Chromium 的 VA-API 实现里写死了跳过 NVIDIA 设备。它不是「配错了」，而是**不支持** ——
所以 NVIDIA 上无论怎么加 flag，`<video>` 都会回退到软解。

对比：Firefox 的 VA-API（以及 NVDEC）**支持 NVIDIA**，在纯 NVIDIA 环境里能真正硬解。

## 解决办法

按你的机器形态选一条路：

**A. 独显直连、系统里只有 NVIDIA** → 用 **Firefox** 看视频。
Firefox 支持 NVIDIA 硬解，不需要任何「绕路」，也不会有跨卡问题：

```
about:support → Media → Hardware Video Decoding: enabled / 看 FFmpegVpxVideoDecoder 之类
```

**B. 有核显（本机这种）** → 让 Chrome 跑在核显上做硬解：

```
# ~/.config/chrome-flags.conf
--render-node-override=/dev/dri/renderD128      # 核显 render 节点，编号用 lspci/by-path 确认
```

这条路能拿到完整硬解（h264 / vp9 / hevc / av1），但**代价是跨卡合成**：
Chrome 在 780M 渲染、niri 在 4060 扫描输出 → 撕裂纹/残影 →
按 [第一类：跨卡合成](../显卡/01-跨卡合成-闪屏撕裂卡顿花屏.md) 那套处理。

常见误区：`--use-gl=egl` 在 Chrome 158 是**非法值**，会让 GPU 初始化失败、全软件渲染
（`gl_factory.cc:110 … not found in allowed implementations`）。

## 怎么确认好了

`chrome://gpu`：

```
GL_RENDERER: ANGLE (AMD, AMD Radeon 780M Graphics (radeonsi phoenix ACO), …)
Decoding: h264 baseline/main/high、vp9 profile0/2、hevc main/main10、av1 main
```

本机实测（B 站视频 + 弹幕）：Chrome 合计 CPU 51%、CPU 52–54 ℃、风扇 3700–3800 RPM。
**别把「CPU 降低」当优化证据** —— GPU 被整关掉时 CPU 反而可能更低，先确认 `GL_RENDERER` 是真显卡。

## 参考

- `media/gpu/vaapi/vaapi_wrapper.cc:131`（跳过 nvidia-drm）
- [显卡/第一类：跨卡合成](../显卡/01-跨卡合成-闪屏撕裂卡顿花屏.md)
