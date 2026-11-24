# pacman 缓存占了 11 G：它同时是降级的唯一退路

!!! info "一句话"
    清之前先想清楚 —— 缓存删空后，翻车只能靠 Arch Linux Archive。

## 现象

`/var/cache/pacman/pkg` 到 11 G（约 2993 个文件），系统盘吃紧。

## 环境

Arch / pacman。

## 排查

```bash
du -sh /var/cache/pacman/pkg
ls /var/cache/pacman/pkg | wc -l
```

## 原因

pacman 默认保留所有历史版本包（包括已卸载的），方便 `-U` 降级。

## 解决办法

按需清，别一把梭：

```bash
sudo pacman -Sc            # 清掉不在仓库里的旧包（回收最多）
sudo paccache -rk2         # 只留最近 2 个版本（需 pacman-contrib）
paru -Sc                   # AUR 构建缓存
```

降级两条路：

```bash
# ① 本地缓存
ls /var/cache/pacman/pkg | grep 包名
sudo pacman -U /var/cache/pacman/pkg/包名-旧版本.pkg.tar.zst
# ② Arch Linux Archive（缓存里没有时）
# /etc/pacman.d/mirrorlist 临时改为
#   Server=https://archive.archlinux.org/repos/2026/11/20/$repo/os/$arch
sudo pacman -Syyuu          # 双 u 允许降级
```

## 怎么确认好了

`du -sh /var/cache/pacman/pkg` 降下来，且至少还留着当前版本的包（能 `pacman -U` 回滚）。

## 参考


