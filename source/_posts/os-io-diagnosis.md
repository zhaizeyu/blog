---
title: 磁盘 I/O 分析与排查笔记
date: 2025-05-09 00:00:00
categories:
  - 操作系统
tags:
  - IO定位
  - 磁盘
  - 性能分析
---

# 磁盘 I/O 分析与排查笔记

## atop 指标说明

* **busy**：磁盘在采样周期内的忙碌时间百分比，表示设备处于活动状态的比例。
* **read / write**：每秒的读写请求次数。
* **KiB/r / KiB/w**：每次读写操作的数据量（KiB）。
* **MBr/s / MBw/s**：每秒的读写数据量（MB/s）。
* **avio**：平均每次 I/O 操作的响应时间（毫秒）。

### DSK 列

* **RDDSK**：每秒读取的磁盘数据量（KB/s，逻辑请求，不代表真实落盘）。
* **WRDSK**：每秒写入的磁盘数据量（KB/s，逻辑请求，不代表真实落盘）。
* **DSK**：总磁盘 I/O（RDDSK + WRDSK），该进程占系统当前已使用 I/O 的比例。

### 常见分析场景

* **吞吐量低但 busy 高**

  * 可能是 **大量小 I/O**，导致 avio 高。
* **avio 持续 > 10ms**

  * 可能存在存储性能瓶颈。
* **avio 持续 > 100ms**

  * 可能涉及硬件故障或链路问题。

---

## 常用辅助命令

* `iotop`：实时查看进程 I/O 占用情况。
* `pidstat -d 1`：按进程维度统计 I/O。
* `iostat -dx 1`：磁盘整体 I/O 指标，`await` 反映平均响应时间（类似 avio）；util 磁盘利用率低，wait 高可能是总体磁盘使用少，单次磁盘读写慢。

---

## 读写性能测试

```bash
# 写入性能测试（1 GiB）
dd if=/dev/zero of=/tmp/testfile bs=1M count=1024 oflag=direct

# 读取性能测试
dd if=/tmp/testfile of=/dev/null bs=1M iflag=direct

# 查看磁盘延迟
iostat -dx 1
```

---

## 硬件层面检查

* `smartctl -a /dev/sdX`
  查看磁盘 SMART 信息，判断是否存在硬件错误。

* `dmesg | grep -i error`
  或
  `grep -i error /var/log/messages`
  检查内核日志中是否有 I/O 错误。

---

## 文件系统检查

```bash
# 检查所有挂载分区的文件系统状态
df | grep ^/dev | cut -d" " -f1 | while read line; do
  echo -n "$line "
  dumpe2fs -h $line 2>/dev/null | grep 'Filesystem state' | cut -d":" -f2
done
```

---

## I/O 路径理解（自上而下）

1. **应用层**：`read()` / `write()` 系统调用
2. **VFS**：虚拟文件系统抽象
3. **文件系统层**：ext4 / xfs / btrfs 等
4. **通用块层**：合并请求、调度算法、blk-mq
5. **设备驱动层**：SCSI / NVMe / virtio-blk
6. **主机总线与适配器**：SAS / SATA / PCIe 控制器
7. **存储设备固件**：SSD FTL、HDD Firmware
8. **物理介质**：NAND 闪存、磁盘盘片

👉 每一层都可能成为性能瓶颈：

* 文件系统：日志/锁
* 块层：调度队列积压
* 驱动/控制器：队列溢出、reset
* 设备固件：GC、掉盘

---
