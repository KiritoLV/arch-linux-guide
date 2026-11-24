# Arch 排错笔记

混合显卡 + niri 的机器上，显卡问题基本只有**两大类**，其余都是变体：

1. **跨卡合成** —— niri 一块卡扫描输出、客户端另一块卡渲染，缺显式同步：闪屏 / 撕裂 / 残影 / 单帧花屏。
2. **静默偶发卡顿** —— 频率掉进深睡档，升频来不及：几秒顿一下，无日志。

Chrome 的硬解是常见触发器：Chromium 不支持 NVIDIA 硬解（独显直连只能软解，得用 Firefox）；
有核显时改指核显能硬解，但立刻引入跨卡合成。

**在线看 → https://kiritolv.github.io/arch-linux-guide/**

## 目录

- **显卡**：第一类：跨卡合成 —— 闪屏 / 撕裂 / 残影 / 单帧花屏、第二类：静默偶发卡顿 —— 频率太低（锁频解决）
- **Chrome**：Chrome 在 NVIDIA 上根本不会硬解：独显直连就换 Firefox，有核显才指定核显
- **存储**：NTFS 大文件读到一半 EIO、目录读到某个 inode 就中断

## 新增一条

1. 在对应分类目录建 `NN-问题标题.md`，模板：现象 / 环境 / 排查 / 原因 / 解决办法 / 怎么确认好了 / 参考。
2. 预览：`./.venv/bin/mkdocs serve` → http://127.0.0.1:8000
3. 发布：`./.venv/bin/mkdocs gh-deploy --force --no-history`

## 环境

Arch Linux / 内核 `7.3.5-zen1-1-zen` / niri `26.04` / nvidia-open-dkms `620.73.04` /
Chrome `158` / ASUS TUF A15 FA507XV（R9 7940HS + RTX 4060 Laptop + 780M）。

License: MIT
