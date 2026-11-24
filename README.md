# Arch Linux 个人使用指南

一台 ASUS TUF Gaming A15 FA507XV（R9 7940HS + RTX 4060 Laptop + Radeon 780M）上的
Arch Linux 实战笔记：混合显卡、niri、Chrome 硬件解码、VRR、性能调优、排错。

**在线阅读 → https://kiritolv.github.io/arch-linux-guide/**

## 写作原则

1. 只写**实测**结论，推断一律标注；
2. 每条结论给**判据命令**，半年后能自己复核；
3. 写因果链（因为 A 导致 B，在 C 条件下出现 D），不写「试这个」。

## 内容

| 章节 | 内容 |
|---|---|
| [系统基线](docs/01-系统基线.md) | 硬件、内核参数逐条、GRUB、snapper、zram、常驻服务 |
| [包管理](docs/02-包管理.md) | pacman/paru、降级（缓存 + ALA）、清理、升级后必查 |
| [显卡与显示](docs/03-显卡与显示.md) | 混合显卡拓扑、两档切换、Chrome 硬解、跨卡撕裂、VRR、锁频、MangoHud |
| [排错手册](docs/09-排错手册.md) | 症状 → 第一动作 |

存储（btrfs + NTFS）、音频、Wine/游戏、网络代理四章在写。

## 本地构建

```bash
python3 -m venv .venv
./.venv/bin/pip install mkdocs-material
./.venv/bin/mkdocs serve            # http://127.0.0.1:8000
```

## 发布

```bash
./.venv/bin/mkdocs gh-deploy --force --no-history   # 推到 gh-pages，Pages 自动更新
```

## 免责

个人记录。照着做之前先确认自己的硬件拓扑（`lspci -nn`、`niri msg outputs`）。
涉及 `sudo`、引导参数、分区与快照的操作，后果自负。

License: MIT
