# Chapter 03 使用 tuned 优化系统

[返回](../)

[TOC]

## 调优配置

tuned 提供了不同的配置文件来对系统进行调优。

```sh
[root@server130 ~]# tuned-adm list
Available profiles:
- accelerator-performance     - Throughput performance based tuning with disabled higher latency STOP states
- balanced                    - General non-specialized tuned profile
- desktop                     - Optimize for the desktop use-case
- hpc-compute                 - Optimize for HPC compute workloads
- intel-sst                   - Configure for Intel Speed Select Base Frequency
- latency-performance         - Optimize for deterministic performance at the cost of increased power consumption
- network-latency             - Optimize for deterministic performance at the cost of increased power consumption, focused on low latency network performance
- network-throughput          - Optimize for streaming network throughput, generally only necessary on older CPUs or 40G+ networks
- optimize-serial-console     - Optimize for serial console use.
- powersave                   - Optimize for low power consumption
- throughput-performance      - Broadly applicable tuning that provides excellent performance across a variety of common server workloads
- virtual-guest               - Optimize for running inside a virtual guest
- virtual-host                - Optimize for running KVM guests
Current active profile: virtual-guest
```

### 从命令行管理调优配置

`tuned-adm` 用于更改 tuned 守护进程的设置

```sh
# 查看当前的调优配置文件
[root@server130 ~]# tuned-adm active
Current active profile: virtual-guest

# 列出可用的调优配置文件
[root@server130 ~]# tuned-adm list

# 切换调优配置文件
[root@server130 ~]# tuned-adm profile balanced
[root@server130 ~]# tuned-adm active
Current active profile: balanced

# 使用系统推荐的调优配置文件
[root@server130 ~]# tuned-adm recommend
virtual-guest
```

