---
title: Linux扩容根磁盘空间
date: 2022-04-17 14:55:26
tags: linux-根扩容
category: Linux
---

1.添加磁盘空间

2.查看磁盘信息

```shell
fdisk -l
```

3.新建磁盘分区

4.创建新分区(vdb1)。

```shell
fdisk /dev/vdb
```

5.创建物理卷。

```shell
pvcreate /dev/vdb1
```

6.将添加新的物理卷，加载到centos卷组。

```shell
vgextend centos /dev/vdb1
```

7.增加/dev/mapper/centos-root大小，增加100G。

```shell
lvresize -L +100G /dev/mapper/centos-root
```

8.同步文件系统。

```shell
xfs_growfs /dev/mapper/centos-root
```

9.查看扩容后的大小。

```shell
df -h
```
