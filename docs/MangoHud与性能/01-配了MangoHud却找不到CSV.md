# 配了 MangoHud，玩完却找不到 CSV

!!! info "一句话"
    输出目录不存在、没人按 Shift+F2、或者抓错了显卡 —— 三个原因，任何一个都会让你一张都拿不到。

## 现象

`~/.config/MangoHud/MangoHud.conf` 明明配了 `output_folder`，玩完游戏目录里是空的。

## 环境

mangohud 0.8.4 / Steam + Proton 游戏 / 混合显卡。

## 排查

```bash
ls -ld <output_folder 指向的目录>          # ① 目录存在吗
grep -E 'autostart_log|toggle_logging|pci_dev' ~/.config/MangoHud/MangoHud.conf
grep "Set renderD" ~/某次启动日志           # ② 抓到的是哪张卡
```

## 原因

三个独立条件，全满足才落盘：

1. `output_folder` 指向的目录**必须已存在**（MangoHud 不会帮你建）；
2. 只配 `toggle_logging=Shift_L+F2` 时需要人手动按 —— 没人按就永远没有文件；
3. 混合显卡下可能抓到核显，指标全错（甚至不写文件）。

## 解决办法

```
# ~/.config/MangoHud/MangoHud.conf
output_folder=/home/<user>/mangologs     # 先 mkdir -p
autostart_log=5                          # 启动 5 秒后自动开始记录
pci_dev=0000:01:00.0                     # 用 lspci -nn 拿独显地址，别让 MangoHud 抓错卡
```

加载成功的证据（日志里会出现）：

```
[MANGOHUD] [info] [gpu.cpp:90] Set renderD129 as active GPU (driver=nvidia id=10de:28e0 pci_dev=0000:01:00.0)
```

## 怎么确认好了

玩完**正常退出**游戏，目录里出现 `<程序名>_<日期>_<时间>.csv` 和 `_summary.csv`：

```
v1
v0.8.4
---------------------SYSTEM INFO---------------------
os,cpu,gpu,ram,kernel,driver,cpuscheduler
--------------------FRAME METRICS---------------------
fps,frametime,cpu_load,cpu_power,gpu_load,cpu_temp,gpu_temp,gpu_core_clock,gpu_mem_clock,
gpu_vram_used,gpu_power,ram_used,swap_used,process_rss,cpu_mhz,elapsed
```

`elapsed` 是纳秒。

## 参考


