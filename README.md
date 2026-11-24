# Arch 排错笔记

在 Arch Linux（混合显卡 + niri 合成器）上真实碰到的问题：**现象 → 排查 → 原因 → 解决办法**。

**在线看 → https://kiritolv.github.io/arch-linux-guide/**

## 为什么这么写

- 只写**实测**结论，推断一律标注；
- 每条给**判据命令**，能自己复核，而不是「试这个」；
- 写因果链：因为 A 导致 B，在满足 C 时出现 D。

## 目录

- **显示与显卡**（4）：Chrome 跑核显、niri 跑独显 → 跨卡合成导致撕裂/残影、VRR 配了却一直不生效：Proton 游戏的 app_id 不是 `steam_app_*`、高刷下窗口偶发单帧花屏（NVIDIA）、桌面滑动一顿一顿，过几秒就卡一下（显存掉到 405 MHz）
- **Chrome**（3）：chrome://gpu 写着 Hardware accelerated，但 `Decoding:` 是空的、加了 `--use-gl=egl` 之后 Chrome 变成全软件渲染、看视频时弹幕像低刷新率、晃鼠标才顺
- **MangoHud与性能**（3）：配了 MangoHud，玩完却找不到 CSV、分析帧时间时最容易犯的错：把连续分布读成「VSync 量化」、游戏只有 70 fps 上不去：GPU 满载，不是参数没调好
- **系统与内核**（3）：改了 niri 配置，哪些要重登、哪些热重载就够、内核参数里两项各出现两次、pacman 缓存占了 11 G：它同时是降级的唯一退路
- **存储**（1）：NTFS 大文件读到一半 EIO、目录读到某个 inode 就中断
- **音频**（1）：录不到系统声音 / `pw-record --target` 指定了却没生效
- **网络**（1）：测延迟测出 0.0002 s：透明代理劫持了连接

## 新增一条

1. 在对应分类目录下建 `NN-问题标题.md`，照模板写：现象 / 环境 / 排查 / 原因 / 解决办法 / 怎么确认好了 / 参考。
2. 预览：`./.venv/bin/mkdocs serve` → http://127.0.0.1:8000
3. 发布：`./.venv/bin/mkdocs gh-deploy --force --no-history`

## 环境

Arch Linux / 内核 `7.3.5-zen1-1-zen` / niri `26.04` / nvidia-open-dkms `620.73.04` /
mesa `26.3.1` / Chrome `158` / ASUS TUF A15 FA507XV（R9 7940HS + RTX 4060 Laptop + 780M）。

硬件不同结论未必成立，但判据命令照样能用。

License: MIT
