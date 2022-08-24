---
title: mysql安装
date: 2022-08-24 18:00:16
tags: mysql安装
category: Linux
---

## MySQL安装

```shell
yum -y install mysql
```

```shell
wget https://dev.mysql.com/get/mysql80-community-release-el7-6.noarch.rpm
```

```shell
yum -y install mysql80-community-release-el7-6.noarch.rpm
```

```shell
yum -y install mysql-community-server --nogpgcheck
```

```shell
systemctl start mysqld
systemctl enable mysqld
systemctl status mysqld
```

```shell
grep 'temporary password' /var/log/mysqld.log
```
