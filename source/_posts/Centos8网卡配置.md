---
title: Centos8网卡配置
date: 2022-08-24 18:03:34
tags: Centos8网卡配置
category: Linux
---

## Centos8网卡命令

### 1.查看网卡设备详细信息

```shell
nmcli device show / nmcli device show ens33
```

### 2.查看网卡设备状态

```shell
nmcli device status
```

### 3.查看网卡信息

```shell
nmcli device status
```

### 4.查看网卡具体信息

```shell
nmcli connection show ens33
```

### 5.查看所有活动连接

```shell
nmcli connection show --active
```

### 6.启动网卡

```shell
nmcli connection up ens33
```

### 7.停止

```shell
nmcli connection down ens33（可被自动激活）

nmcli device disconnect ens33（禁止被自动激活）
```

### 8.重启

```shell
nmcli connection reload
```

