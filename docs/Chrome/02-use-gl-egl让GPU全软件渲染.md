# 加了 `--use-gl=egl` 之后 Chrome 变成全软件渲染

!!! info "一句话"
    Chrome 158 允许的 GL 实现里没有 `gl=egl`，写了就直接放弃 GPU。

## 现象

`chrome://gpu` 全是 `Software only`，`GL_VENDOR / GL_RENDERER / GL_VERSION` 都是 `Disabled`，
`GPU0 VENDOR=0x0000 DEVICE=0x0000`；页面明显变卡、CPU 飙升。

## 环境

Chrome 158（Linux Wayland）。

## 排查

用日志实例复现，直接把原因打出来：

```bash
google-chrome-stable --enable-logging=stderr --v=1 2>&1 | grep -i "gl_factory"
# ui/gl/init/gl_factory.cc:110 Requested GL implementation (gl=egl-gles2, angle=none)
#   not found in allowed implementations:
#   [(gl=egl-angle,angle=opengl),(gl=egl-angle,angle=opengles),(gl=egl-angle,angle=vulkan)]
```

## 原因

`--use-gl=egl` 是旧版参数，Chrome 158 只接受 `gl=egl-angle` 三档。传了非法值不是回退，
而是**整个 GPU 初始化失败**。

## 解决办法

把那一行从 `~/.config/chrome-flags.conf` 删掉，然后**完全退出 Chrome**（含后台进程）重新启动。

```bash
rm -f ~/.config/chrome-flags.conf.bak   # 或手动删掉 --use-gl=egl 那行
pgrep -a chrome | head                  # 确认没有残留进程
```

## 怎么确认好了

`chrome://gpu` 恢复 `Compositing / Rasterization / Video Decode: Hardware accelerated`，
`GL_RENDERER` 回到真实显卡名。

## 参考


