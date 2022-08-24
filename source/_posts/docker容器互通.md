---
title: docker容器互通
date: 2022-08-24 17:58:21
tags: docker容器互通
category: Linux
---

### 1.创建自定义网桥

```shell
docker network create  --subnet=172.101.0.0/24 test
```

###网段可以自己定义，最后是网桥名称也可以自己定义



### 2.查看自定义网桥信息

```shell
docker network inspect test
```



### 3.启动容器

```shell
docker run -itd --net=test --ip=172.101.0.101 --name server centos
```



### 4.添加路由

```shell
ip route add **172.102.0.0/24** via 192.168.100.130 dev ens33
```



### 5.添加永久路由

```shell
###编辑文件vim /etc/rc.local

###在最后添加

ip route add 172.102.0.0/24 via 192.168.100.130 dev ens33
```



