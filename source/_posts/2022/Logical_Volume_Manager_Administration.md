---
title: 配置和管理逻辑卷管理器 (Logical Volume Manager)
date: 2022-08-25 14:22:08
updated: 2022-08-25 14:22:08
tags: [Linux, LVM]
categories: Linux
---

# 1. 逻辑卷管理概述

逻辑卷管理 (LVM) 在物理存储上创建抽象层，帮助您创建逻辑存储卷。这比直接使用物理存储的方式具有更大的灵活性。

此外，硬件存储配置在软件中隐藏，因此可以调整大小并移动，无需停止应用或卸载文件系统。这可降低操作成本。

<!-- more -->

## 1.1 LVM 组件

 LVM 具有以下基本组件：

- **物理卷 (Physical Volume, PV)** ：物理卷 (PV) 是指定为 LVM 使用的物理硬盘分区或整个硬盘。如需更多信息，请参阅[管理 LVM 物理卷](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_logical_volumes/managing-lvm-physical-volumes_configuring-and-managing-logical-volumes)。
- **卷组 (Volume Group, VG)** ：卷组 (VG) 是物理卷 (PV) 的集合，它会创建一个磁盘空间资源池，可以从中分配逻辑卷 (LV)。如需更多信息，请参阅[管理 LVM 卷组](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_logical_volumes/managing-lvm-volume-groups_configuring-and-managing-logical-volumes)。
- **逻辑卷 (Logical Volume, LV)** ：逻辑卷 (LV) 代表可挂载的存储设备，用于创建文件系统。如需更多信息，请参阅[管理 LVM 逻辑卷](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_logical_volumes/managing-lvm-logical-volumes_configuring-and-managing-logical-volumes)。

![LVM 逻辑卷组件](basic-lvm-volume-components.png)

- **物理扩展 (Physical Extend, PE)** ：物理扩展 (PE) 是增大或者减小逻辑卷容量的最小值。当使用物理卷创建卷组时，默认情况下每个物理扩展 (PE) 的大小为 4MB。如果默认物理扩展 (PE) 大小不合适，可以使用 `-s` 选项为 `vgcreate` 命令指定范围大小。

![VG和PE关系图](pe_vg.gif)

如上图所示，VG 中的 PE 分配给虚线部分的 LV。如果未来要扩充 VG ，为这个 VG 添加其他的 PV 即可。如果要扩充 LV，也是通过分配 VG 中未使用的 PE 给 LV 实现的。

## 1.2 LVM 的优点
与直接使用物理存储相比，逻辑卷具有以下优势：

- **灵活的容量** ：使用逻辑卷时，您可以将设备和分区聚合到一个逻辑卷中。借助此功能，文件系统可以扩展到多个设备中，就像它们是一个单一的大型设备一样。
- **存储卷大小** ：您可以使用简单的软件命令扩展逻辑卷或减小逻辑卷大小，而无需重新格式化和重新分区基础设备。
- **在线数据重新定位** ：部署更新、更快或者更弹性的存储子系统，可以在系统活跃时移动数据。在磁盘处于使用状态时可以重新分配磁盘。例如，您可以在删除热插拔磁盘前将其清空。
- **方便的设备命名** ：逻辑卷可以使用用户定义的名称和自定义名称进行管理。
- **条带化卷** ：您可以创建一个在两个或者多个设备间条带化分布数据的逻辑卷。这可显著提高吞吐量。
- **RAID 卷** ：逻辑卷为您对数据配置 RAID 提供了一种便捷的方式。这可防止设备故障并提高性能。
- **卷快照** ：您可以对数据进行快照（逻辑卷在一个特点时间点上的副本）用于一致性备份或测试更改的影响，而不影响实际数据。
- **精简卷** ：逻辑卷可以使用精简模式置备。这可让您创建大于可用物理空间的逻辑卷。
- **缓存卷** ：缓存逻辑卷使用快速块设备，如 SSD 驱动器，以提高更大、较慢的块设备的性能。