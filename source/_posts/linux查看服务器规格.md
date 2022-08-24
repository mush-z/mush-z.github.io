---
title: linux查看服务器规格
date: 2022-08-24 17:57:04
tags: linux查看服务器规格
category: Linux
---

### 1.查看物理CPU个数：

```shell
cat /proc/cpuinfo |grep "physical id"|sort |uniq|wc -l
```



### 2.查看每个CPU具有几个核

```shell
cat /proc/cpuinfo |grep "cores"|uniq
```



### 3.查看逻辑CPU个数  

```shell
cat /proc/cpuinfo |grep "processor"|wc -l
```





```shell
df -h |grep -v 'Filesystem' | awk 'BEGIN {printf "文件系统使用情况： \n \n"} {printf $1} {printf "文件系统使用率："} {print $5}'
```

