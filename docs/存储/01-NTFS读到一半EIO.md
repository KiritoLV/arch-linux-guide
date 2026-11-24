# NTFS 大文件读到一半 EIO、目录读到某个 inode 就中断

!!! info "一句话"
    内核 `fs/ntfs`（新驱动）7.3.5 的已知 bug，7.3.6 修了；与 NTFS 卷是否 dirty 无关。

## 现象

整盘顺序读到 52 GiB 左右开始 EIO，47 个文件读不出来；同样的文件用 `ntfs-3g` 读完全正常。

## 环境

内核 7.3.5-zen1-1-zen / 内核自带 `fs/ntfs`；NTFS 卷 1.23 TiB 挂在 `/mnt/winshare`。

## 排查

```bash
dmesg | grep -i -E "ntfs|I/O error" | tail
# 对照：同一批文件用 ntfs-3g 挂载读一遍（能读通 → 驱动问题，不是盘问题）
```

## 原因

7.3.5 的 `fs/ntfs/attrib.c` 在计算属性长度时对 sparse/压缩属性少算了一个字段，
越界后返回 EIO。7.3.6 的修复：

```c
- if (a->flags & (ATTR_IS_SPARSE | ATTR_COMPRESSION_MASK))
+ if ((a->flags & (ATTR_IS_SPARSE | ATTR_COMPRESSION_MASK)) &&
+     !a->data.non_resident.lowest_vcn)
        min_len += sizeof(a->data.non_resident.compressed_size);
```

## 解决办法

两条路：

1. **升级到含修复的内核**（7.3.6 起）。
2. 临时验证可以只热替换模块（不装整内核、重启即回滚）：用 7.3.6 的 `fs/ntfs` 源码，
   `make LOCALVERSION=-zen1-1-zen` 保证 vermagic 一致，`insmod` 自编的 `ntfs.ko`。

判别加载的是哪个模块**看 `coresize`，不看 vermagic**：修复版 466944，原版 495616。

## 怎么确认好了

同一批文件重跑不再出现 EIO。本机实测：139.2 GiB / 77.4 s / 1.80 GiB/s / **0 EIO**，
原先 47 个 EIO 全部读通，原 50 个正常文件 0 回归。

## 参考

- 内核 `fs/ntfs/attrib.c`（7.3.6 修复）
