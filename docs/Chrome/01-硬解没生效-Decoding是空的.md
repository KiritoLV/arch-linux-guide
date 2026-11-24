# chrome://gpu 写着 Hardware accelerated，但 `Decoding:` 是空的

!!! info "一句话"
    Chromium 主动跳过 NVIDIA 的 VA-API，独显上的「硬件加速」是假象。

## 现象

`chrome://gpu` 里 `Video Decode: Hardware accelerated`，但 `Video Acceleration Information → Decoding:`
一条都不列。播放视频时 CPU 高、风扇转，`nvidia-smi` 的 NVDEC 占用 0%。

## 环境

Chrome 158 / 混合显卡（4060 + 780M）/ Wayland；Chrome 被钉在 4060 的 render 节点上。

## 排查

```bash
# 1) 看 GPU 进程到底是哪张卡
grep -oE '/dev/dri/renderD[0-9]+' /proc/$(pgrep -f 'chrome --type=gpu-process' | head -1)/cmdline | sort -u
# 2) 看 Chrome 自己的日志
WARNING:media/gpu/vaapi/vaapi_wrapper.cc:131 : Should skip nVidia device named: nvidia-drm
# 3) 对照 codec 支持
Decoding: （空）    →  实际是软解
```

## 原因

Chromium 的 VA-API 实现里写死了跳过 `nvidia-drm`。即使 4060 有 VDPAU/NVDEC 能力，
Chrome 也不会用它做 `<video>` 解码。AMD 的 `radeonsi_drv_video.so` 则被认可。

## 解决办法

让 Chrome 跑在核显的 render 节点上：

```
# ~/.config/chrome-flags.conf
--render-node-override=/dev/dri/renderD128
```

（该文件由 `/usr/bin/google-chrome-stable` 读取，脚本用 `grep -v '^#'` 过滤注释，写中文注释安全。）

注意：**节点编号别照抄**，`/dev/dri/renderD12X` 对应哪张卡用 `lspci -nn` + `by-path` 确认。

## 怎么确认好了

`chrome://gpu` 应该变成：

```
GL_RENDERER: ANGLE (AMD, AMD Radeon 780M Graphics (radeonsi phoenix ACO), OpenGL ES 3.2 Mesa …)
Decoding: h264 baseline/main/high、vp9 profile0/2、hevc main/main10、av1 main 全部列出
```

本机实测（B 站视频 + 弹幕）：Chrome 合计 CPU 51%、CPU Tctl 52.4–53.8 ℃、风扇 3700–3800 RPM
（GPU 被整关时是 102% / 73–75 ℃ —— **别把那种「降 CPU」当优化证据**，先确认 GPU 没被关掉）。

## 参考

- `media/gpu/vaapi/vaapi_wrapper.cc:131`（跳过 nvidia-drm 的那行）
