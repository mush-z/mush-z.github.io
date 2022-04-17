---
title: openstack-Mitaka
date: 2022-04-17 14:47:38
tags: openstack-M
category: Linux
---

# OpenStack M版搭建

![img](https://i.bmp.ovh/imgs/2022/04/17/3f71c9177e94cb22.png)

## 密码表：(密码均为密码名称)

|         密码名称         |                  描述                  |
| :----------------------: | :------------------------------------: |
| 数据库密码(不能使用变量) |            数据库的root密码            |
|       `ADMIN_PASS`       |            `admin` 用户密码            |
|   `CEILOMETER_DBPASS`    |       Telemetry 服务的数据库密码       |
|    `CEILOMETER_PASS`     | Telemetry 服务的 `ceilometer` 用户密码 |
|     `CINDER_DBPASS`      |       块设备存储服务的数据库密码       |
|      `CINDER_PASS`       |     块设备存储服务的 `cinder` 密码     |
|      `DASH_DBPASS`       |  Database password for the dashboard   |
|       `DEMO_PASS`        |           `demo` 用户的密码            |
|     `GLANCE_DBPASS`      |          镜像服务的数据库密码          |
|      `GLANCE_PASS`       |      镜像服务的 `glance` 用户密码      |
|      `HEAT_DBPASS`       |     Orchestration服务的数据库密码      |
|    `HEAT_DOMAIN_PASS`    |         Orchestration 域的密码         |
|       `HEAT_PASS`        | Orchestration 服务中``heat``用户的密码 |
|    `KEYSTONE_DBPASS`     |          认证服务的数据库密码          |
|     `NEUTRON_DBPASS`     |          网络服务的数据库密码          |
|      `NEUTRON_PASS`      |     网络服务的 `neutron` 用户密码      |
|      `NOVA_DBPASS`       |          计算服务的数据库密码          |
|       `NOVA_PASS`        |      计算服务中``nova``用户的密码      |
|      `RABBIT_PASS`       |        RabbitMQ的guest用户密码         |
|       `SWIFT_PASS`       |    对象存储服务用户``swift``的密码     |

## 主机配置

|   主机名   |      IP       | 内存 | 硬盘 | 网卡数 |
| :--------: | :-----------: | :--: | :--: | :----: |
| controller | 172.16.100.28 |  8G  | 80G  |   1    |
|  compute   | 172.16.100.29 |  8G  | 80G  |   1    |

### 修改主机名

#### controller

```shell
hostnamectl set-hostname controller
```

#### compute

```shell
hostnamectl set-hostname compute
```

<!--注意重新连接 否则后面创建Rabbitmq用户可能会报错-->

## 网络配置

### controller配置

```shell 
vi /etc/sysconfig/network-scripts/ifcfg-eth0

ONBOOT=yes
BOOTPROTO=static
IPADDR=172.16.100.28
NETMASK=255.255.255.0
GATEWAY=172.16.100.254
DNS1=114.114.114.114
```

###  compute配置

```shell
vi /etc/sysconfig/network-scripts/ifcfg-eth0

ONBOOT=yes
BOOTPROTO=static
IPADDR=172.16.100.29
NETMASK=255.255.255.0
GATEWAY=172.16.100.254
DNS1=114.114.114.114
```

：wq 保存退出

### 重启网卡

<!--在controller和compute均执行-->

```shell
systemctl restart network
```

#### controller

```shell
[root@controller ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:00:db:9c brd ff:ff:ff:ff:ff:ff
    inet 172.16.100.28/24 brd 172.16.100.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::6010:d73a:2158:5fe8/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

#### compute

```shell
[root@compute ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:3c:b9:4c brd ff:ff:ff:ff:ff:ff
    inet 172.16.100.29/24 brd 172.16.100.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::c374:72b:eb12:5916/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

## 初始化配置

### 关闭防火墙

<!--在controller和compute均执行-->

```shell
systemctl stop firewalld
systemctl disable firewalld
```

### 关闭网卡守护进程

<!--在controller和compute均执行-->

```shel
systemctl stop NetworkManager
systemctl disable NetworkManager
```

### 关闭SElinux

<!--在controller和compute均执行-->

```shell
setenforce 0

vi /etc/selinux/config

修改 SELINUX=enforcing 为 SELINUX=disabled

:wq 保存退出
```

### 查看SElinux状态

<!--在controller和compute均执行-->

```shell
[root@controller ~]# getenforce 
Permissive

[root@compute ~]# getenforce 
Permissive

```

### 配置yum源

```shell
将目录下的源删除
rm -rf /etc/yum.repos.d/C*

#controller
[root@controller ~]# cat /etc/yum.repos.d/http.repo 
[centos]
name=centos
baseurl=ftp://172.16.100.66/centos
gpgcheck=0
[openstack-m]
name=openstack-m
baseurl=ftp://172.16.100.66/openstack-m
gpgcheck=0

#compute
[root@compute ~]# cat /etc/yum.repos.d/http.repo 
[centos]
name=centos
baseurl=ftp://172.16.100.66/centos
gpgcheck=0
[openstack-m]
name=openstack-m
baseurl=ftp://172.16.100.66/openstack-m
gpgcheck=0
```

>此处用的是配置本地源，一个Centos，一个OpenStack源
>
>附上国内阿里源https://mirrors.aliyun.com/centos/7/cloud/x86_64/openstack-queens/

### 配置host解析

```shell
[root@controller ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

172.16.100.28 controller
172.16.100.29 compute
```

```shell
[root@compute ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

172.16.100.28 controller
172.16.100.29 compute
```

### 测试各节点之间与Internet的通信

```shell
[root@controller ~]# ping -c 2 compute
PING compute (172.16.100.29) 56(84) bytes of data.
64 bytes from compute (172.16.100.29): icmp_seq=1 ttl=64 time=0.250 ms
64 bytes from compute (172.16.100.29): icmp_seq=2 ttl=64 time=0.447 ms

--- compute ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.250/0.348/0.447/0.100 m
```

```shell
[root@compute ~]# ping -c 2 controller
PING controller (172.16.100.28) 56(84) bytes of data.
64 bytes from controller (172.16.100.28): icmp_seq=1 ttl=64 time=0.378 ms
64 bytes from controller (172.16.100.28): icmp_seq=2 ttl=64 time=0.391 ms

--- controller ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.378/0.384/0.391/0.020 ms
```

```shell
[root@controller ~]# ping -c 2 baidu.com
PING baidu.com (220.181.38.251) 56(84) bytes of data.
64 bytes from 220.181.38.251 (220.181.38.251): icmp_seq=1 ttl=47 time=174 ms
64 bytes from 220.181.38.251 (220.181.38.251): icmp_seq=2 ttl=47 time=173 ms

--- baidu.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 173.205/173.637/174.070/0.600 ms
```

```shell
[root@compute ~]# ping -c 2 baidu.com
PING baidu.com (220.181.38.251) 56(84) bytes of data.
64 bytes from 220.181.38.251 (220.181.38.251): icmp_seq=1 ttl=47 time=174 ms
64 bytes from 220.181.38.251 (220.181.38.251): icmp_seq=2 ttl=47 time=171 ms

--- baidu.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 171.844/173.164/174.484/1.320 ms
```

### 修改时间同步配置文件

#### controller

```shell
vi /etc/chrony.conf

server ntp.aliyun.com iburst
###大概26行左右###
allow 172.16.100.0/24
```

#### compute

```shell
vi /etc/chrony.conf

server controller iburst
```

#### 重启chrony服务

<!--在controller和compute均执行-->

```shell
systemctl restart chronyd
```

#### 验证是否同步成功

```shell
[root@controller ~]# chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* 203.107.6.88                  2   6   367    79   +140us[ +789us] +/-   95ms
```

```shell
[root@compute ~]# chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* controller                    3   6   377     9    -34us[ +664us] +/-   96ms

```

### 所有节点安装openstack客户端和openstack-selinux

```shell
yum -y install python-openstackclient openstack-selinux
```

### 安装Mariadb数据库(controller)

```shell
yum -y install mariadb mariadb-server python2-PyMySQL
```

创建和编辑`/etc/my.cnf.d/openstack.cnf`文件并添加以下：

```shel
[mysqld]
bind-address = 172.16.100.28 #controller IP地址
default-storage-engine = innodb
innodb_file_per_table
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```

#### 启动mariadb

```shell
systemctl start mariadb
systemctl enable mariadb
```

#### 数据库默认初始化 (设置密码000000)

```shell
[root@controller ~]# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

### 安装rabbitmq并创建用户(controller)

1、功能:协调操作和状态信息服务

2、常用的消息代理软件

​			RabbitMQ

​			Qpid

​			ZeroMQ

3.在controller节点安装RabbitMQ

```shell
a.安装RabbitMQ软件包
	yum -y install rabbitmq-server
b.启动服务并设置开机自启动
	systemctl enable rabbitmq-server
	systemctl start rabbitmq-server
查看运行状态
	systemctl status rabbitmq-server
c.创建用户
	rabbitmqctl add_user openstack RABBIT_PASS
	rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```

### 安装memcached缓存(controller)

```shell
a.安装memcached软件包
	yum -y install memcached python-memcached
b.修改配置文件的地址否则其他网络无法访问
	sed -i 's#127.0.0.1#172.16.100.28#g' /etc/sysconfig/memcached  ##注意IP更换成自己的
c.启动服务并设置开机自启动
	systemctl start memcached.service
	systemctl enable memcached.service
```

## Keystone认证服务

keystone认证方式:UUID,PKI,Fernet   随机生成字符串的方法

功能:认证管理,授权管理,服务目录

### 在控制节点创建认证服务数据库(controller)

```shell
a.登录mysql数据库
	mysql -u root -p000000
b.创建keystone数据库
	CREATE DATABASE keystone;
c.创建keystone数据库用户，使其可以对keystone数据库有完全控制权限
	GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'KEYSTONE_DBPASS';
	GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'KEYSTONE_DBPASS';
d.退出数据库
	exit
```

### 安装keystone相关软件包 和openstack-utils.noarch软件包

```shell
yum -y install openstack-keystone httpd mod_wsgi openstack-utils.noarch
```

### 修改配置文件

```shell
cp /etc/keystone/keystone.conf /etc/keystone/keystone.conf.bak
grep -Ev '^$|#' /etc/keystone/keystone.conf.bak > /etc/keystone/keystone.conf

openstack-config --set /etc/keystone/keystone.conf DEFAULT admin_token ADMIN_TOKEN
openstack-config --set /etc/keystone/keystone.conf  database connection mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone
openstack-config --set /etc/keystone/keystone.conf  token provider fernet
```

### 同步keystone数据库

```shell
su -s /bin/sh -c "keystone-manage db_sync" keystone
```

<!--同步后进入数据库查看keystone库是否有数据-->

```shell
如果同步失败,将/var/log/keystone/keystone.log修改属组,重启httpd

chown -R keystone.keystone /var/log/keystone/keystone.log
systemctl restart httpd
```

### 初始化fernet

```shell
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
```

### 配置Apache HTTP服务器

```shell
ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
echo "ServerName controller" >>/etc/httpd/conf/httpd.conf
```

配置以下文件

###将原有是删除

vim /etc/httpd/conf.d/wsgi-keystone.conf 

```shell
Listen 5000
Listen 35357

<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /usr/bin/keystone-wsgi-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/httpd/keystone-error.log
    CustomLog /var/log/httpd/keystone-access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>

<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/httpd/keystone-error.log
    CustomLog /var/log/httpd/keystone-access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>
```

#### 启动HTTP服务

```shell
systemctl enable httpd.service
systemctl start httpd.service
```

### 创建服务和注册api:

```shell
export OS_TOKEN=ADMIN_TOKEN
export OS_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3

openstack service create \
  --name keystone --description "OpenStack Identity" identity

openstack endpoint create --region RegionOne \
  identity public http://controller:5000/v3
  
openstack endpoint create --region RegionOne \
  identity internal http://controller:5000/v3　　
 #前两个为5000端口，专门处理内部和外部的访问
  　　　　　　　　　　　　　　　　　　　　　　　　　　　　
openstack endpoint create --region RegionOne \
  identity admin http://controller:35357/v3
 #35357端口，专门处理admin
```

### 创建域,项目,用户,角色

```shell
openstack domain create --description "Default Domain" default

openstack project create --domain default \
   --description "Admin Project" admin 

openstack user create --domain default \
   --password ADMIN_PASS admin　　　　

openstack role create admin  

openstack role add --project admin --user admin admin

openstack project create --domain default \
--description "Service Project" service
```

### 创建环境变量脚本

```shell
[root@controller ~]# cat admin-openrc 
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

### 取消前面设置的环境变量

```shell
unset OS_TOKEN
```

### 使环境变量生效

```shell
source admin-openrc
```

### 验证环境变量是否成功

```shell
openstack token issue
```

\### 注意环境变量问题:先取消前面设置的环境变量,否则会报错###

## Glance镜像服务

**glance-api**

接收镜像API的调用，诸如镜像发现、恢复、存储。

**glance-registry**

存储、处理和恢复镜像的元数据，元数据包括项诸如大小和类型。

### 在控制节点创建镜像服务数据库(controller)

```shell
a.登录mysql数据库
	mysql -u root -p000000
b.创建glance数据库
	CREATE DATABASE glance;
c.创建glance数据库用户，使其可以对glance数据库有完全控制权限
	GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'GLANCE_DBPASS';
	GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'GLANCE_DBPASS';
d.退出数据库
	exit
```

### 在keystone创建glance用户关联角色

```shell
openstack user create --domain default --password GLANCE_PASS glance
openstack role add --project service --user glance admin

验证是否成功
openstack role assignment list
```

### 在keystone创建服务和注册api

```shell
openstack service create --name glance \
--description "OpenStack Image" image
openstack endpoint create --region RegionOne \
  image public http://controller:9292
openstack endpoint create --region RegionOne \
  image internal http://controller:9292
openstack endpoint create --region RegionOne \
  image admin http://controller:9292
```

### 安装glance软件包

```shell
yum -y install openstack-glance
```

### 修改配置文件

**/etc/glance/glance-api.conf**

```shell
cp /etc/glance/glance-api.conf{,.bak}
grep '^[a-z\[]' /etc/glance/glance-api.conf.bak >/etc/glance/glance-api.conf

openstack-config --set /etc/glance/glance-api.conf database connection mysql+pymysql://glance:GLANCE_DBPASS@controller/glance
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_type password
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken project_name service
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken username glance
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken password GLANCE_PASS
openstack-config --set /etc/glance/glance-api.conf  paste_deploy flavor keystone
openstack-config --set /etc/glance/glance-api.conf glance_store stores file,http
openstack-config --set /etc/glance/glance-api.conf glance_store default_store file
openstack-config --set /etc/glance/glance-api.conf glance_store filesystem_store_datadir /var/lib/glance/images/
```

**/etc/glance/glance-registry.conf**

```shell
cp  /etc/glance/glance-registry.conf{,.bak}
grep '^[a-z\[]' /etc/glance/glance-registry.conf.bak >/etc/glance/glance-registry.conf

openstack-config --set /etc/glance/glance-registry.conf database connection mysql+pymysql://glance:GLANCE_DBPASS@controller/glance
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_type password
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken project_name service
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken username glance
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken password GLANCE_PASS
openstack-config --set /etc/glance/glance-registry.conf paste_deploy flavor keystone
```

### 同步glance数据库

```shell
su -s /bin/sh -c "glance-manage db_sync" glance
```

<!--同步后进入数据库查看glance库是否有数据-->

<!--忽略警告-->

### 启动服务

```shell
systemctl enable openstack-glance-api.service openstack-glance-registry.service
systemctl start openstack-glance-api.service openstack-glance-registry.service
```

### 下载测试镜像cirros

```shell
wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img
```

###下载不了多试几次###

### 上传镜像测试

```shell
openstack image create "cirros"  --file cirros-0.3.4-x86_64-disk.img  --disk-format qcow2 --container-format bare  --public
```

## Nova计算服务

nova-api:				接受响应所有的计算服务请求，管理虚拟机(云主机)生命周期

nova-compute (多个):	真正管理虚拟机(nova-compute调用libvirt)

nova-scheduler:		nova调度器（挑选出最合适的nova-compute来创建虚机)

nova-conductor:		帮助nova-compute代理修改数据库中虚拟机的状态

nova-network			早期openstack版本管理虚拟机的网络(已弃用,neutron)

nova-consoleauth和nova-novncproxy: web版的vnc来直接操作云主机

novncproxy:			web版vnc客户端

nova-api-metadata:		接受来自虚拟机发送的元数据请求(配合neutron-metadata-agent,来虚拟机定制化)

### controller

#### 在控制节点创建nova服务数据库(controller)

```shell
a.登录mysql数据库
	mysql -u root -p000000
b.创建nova_api,nova数据库
	CREATE DATABASE nova_api;
	CREATE DATABASE nova;
c.创建nova_api，nova数据库用户，使其可以对nova_api,nova数据库有完全控制权限
	GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';
	GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';
	GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';
	GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';
d.退出数据库
	exit
```

#### 在keystone创建系统用户（glance，nova，neutron）关联角色

```shell
openstack user create --domain default --password NOVA_PASS nova
openstack role add --project service --user nova admin
```

#### 在keystone上创建服务和注册api

```shell
openstack service create --name nova \
  --description "OpenStack Compute" compute
openstack endpoint create --region RegionOne \
  compute public http://controller:8774/v2.1/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  compute internal http://controller:8774/v2.1/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  compute admin http://controller:8774/v2.1/%\(tenant_id\)s
```

#### 安装相关服务软件包

```shell
yum -y install openstack-nova-api openstack-nova-conductor \
  openstack-nova-console openstack-nova-novncproxy \
  openstack-nova-scheduler
```

#### 修改配置文件

**/etc/nova/nova.conf**

```shell
cp /etc/nova/nova.conf{,.bak}
grep '^[a-Z\[]' /etc/nova/nova.conf.bak >/etc/nova/nova.conf

openstack-config --set /etc/nova/nova.conf DEFAULT enabled_apis osapi_compute , metadata
openstack-config --set /etc/nova/nova.conf api_database connection mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api
openstack-config --set /etc/nova/nova.conf database connection mysql+pymysql://nova:NOVA_DBPASS@controller/nova
openstack-config --set /etc/nova/nova.conf DEFAULT rpc_backend rabbit
openstack-config --set /etc/nova/nova.conf oslo_messaging_rabbit rabbit_host controller
openstack-config --set /etc/nova/nova.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/nova/nova.conf oslo_messaging_rabbit rabbit_password RABBIT_PASS
openstack-config --set /etc/nova/nova.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/nova/nova.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_type password
openstack-config --set /etc/nova/nova.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/nova/nova.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/nova/nova.conf keystone_authtoken project_name service
openstack-config --set /etc/nova/nova.conf keystone_authtoken username nova
openstack-config --set /etc/nova/nova.conf keystone_authtoken password NOVA_PASS
openstack-config --set /etc/nova/nova.conf DEFAULT my_ip 172.16.100.28		###替换自己的IP!!!
openstack-config --set /etc/nova/nova.conf DEFAULT use_neutron True
openstack-config --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
openstack-config --set /etc/nova/nova.conf vnc vncserver_listen '$my_ip'
openstack-config --set /etc/nova/nova.conf vnc vncserver_proxyclient_address '$my_ip'
openstack-config --set /etc/nova/nova.conf glance api_servers http://controller:9292
openstack-config --set /etc/nova/nova.conf oslo_concurrency lock_path /var/lib/nova/tmp
```

#### 同步数据库

```shell
su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage db sync" nova
```

<!--同步后进入数据库查看nova库是否有数据-->

<!--忽略警告-->

#### 启动服务

```shell
systemctl enable openstack-nova-api.service \
  openstack-nova-consoleauth.service openstack-nova-scheduler.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service
systemctl start openstack-nova-api.service \
  openstack-nova-consoleauth.service openstack-nova-scheduler.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service
```

### compute

#### 安装软件包

```shell
yum -y install openstack-nova-compute openstack-utils.noarch
```

#### 修改配置文件

**/etc/nova/nova.conf**

```shell
cp /etc/nova/nova.conf{,.bak}
grep '^[a-Z\[]' /etc/nova/nova.conf.bak >/etc/nova/nova.conf

openstack-config --set /etc/nova/nova.conf DEFAULT rpc_backend rabbit
openstack-config --set /etc/nova/nova.conf oslo_messaging_rabbit rabbit_host controller
openstack-config --set /etc/nova/nova.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/nova/nova.conf oslo_messaging_rabbit rabbit_password RABBIT_PASS
openstack-config --set /etc/nova/nova.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/nova/nova.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_type password
openstack-config --set /etc/nova/nova.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/nova/nova.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/nova/nova.conf keystone_authtoken project_name service
openstack-config --set /etc/nova/nova.conf keystone_authtoken username nova
openstack-config --set /etc/nova/nova.conf keystone_authtoken password NOVA_PASS
openstack-config --set /etc/nova/nova.conf DEFAULT my_ip 172.16.100.29   ###替换自己IP
openstack-config --set /etc/nova/nova.conf DEFAULT use_neutron True
openstack-config --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
openstack-config --set /etc/nova/nova.conf vnc enabled True
openstack-config --set /etc/nova/nova.conf vnc vncserver_listen 0.0.0.0
openstack-config --set /etc/nova/nova.conf vnc vncserver_proxyclient_address '$my_ip'
openstack-config --set /etc/nova/nova.conf vnc novncproxy_base_url http://172.16.100.28:6080/vnc_auto.html
###服务器组件监听所有的 IP 地址，而代理组件仅仅监听计算节点管理网络接口的 IP 地址。基本的 URL 指示您可以使用 web 浏览器访问位于该计算节点上实例的远程控制台的位置。
openstack-config --set /etc/nova/nova.conf glance api_servers http://controller:9292
openstack-config --set /etc/nova/nova.conf oslo_concurrency lock_path /var/lib/nova/tmp
openstack-config --set /etc/nova/nova.conf libvirt cpu_mode none
openstack-config --set /etc/nova/nova.conf libvirt virt_type qemu
```

#### 启动服务

```shell
systemctl enable libvirtd.service openstack-nova-compute.service
systemctl start libvirtd.service openstack-nova-compute.service
```

### 查看nova是否正常

```shell
[root@controller ~]# nova service-list
+----+------------------+------------+----------+---------+-------+----------------------------+-----------+
| Id | Binary           | Host       | Zone     | Status  | State | Updated_at                 | Disabled Reason |
+----+------------------+------------+----------+---------+-------+----------------------------+-----------+
| 1  | nova-consoleauth | controller | internal | enabled | up    | 2021-11-23T12:50:21.000000 | -               |
| 2  | nova-conductor   | controller | internal | enabled | up    | 2021-11-23T12:50:22.000000 | -               |
| 3  | nova-scheduler   | controller | internal | enabled | up    | 2021-11-23T12:50:21.000000 | -               |
| 9  | nova-compute     | compute    | nova     | enabled | up    | 2021-11-23T12:50:25.000000 | -               |
+----+------------------+------------+----------+---------+-------+----------------------------+-----------+

```

## Neutron网络服务

neutron-serverT端(9696)	api:接受和响应外部的网络管理请求

neutron-linuxbridge-agent :	负责创建桥接网卡

neutron-dhcp-agent:		负责分配IP

neutron-metadata-agent:	配合nova-metadata-api实现虚拟机的定制化操作

L3-agent：				实现三层网络vxlan(网络层)

### controller

#### 在控制节点创建neutron服务数据库(controller)

```shell
a.登录mysql数据库
	mysql -u root -p000000
b.创建neutron数据库
	CREATE DATABASE neutron;
c.创建neutron数据库用户，使其可以对neutron数据库有完全控制权限
	GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'NEUTRON_DBPASS';
	GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'NEUTRON_DBPASS';
d.退出数据库
	exit
```

#### 在keystone创建系统用户(glance, nova, neutron)关联角色

```shell
openstack user create --domain default --password NEUTRON_PASS neutron
openstack role add --project service --user neutron admin
```

#### 在keystone上创建服务和注册api

```shell
 openstack service create --name neutron \
  --description "OpenStack Networking" network
openstack endpoint create --region RegionOne \
  network public http://controller:9696
openstack endpoint create --region RegionOne \
  network internal http://controller:9696
openstack endpoint create --region RegionOne \
  network admin http://controller:9696
```

#### 安装服务相应软件包

```shell
yum  -y install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables
```

#### 修改配置文件

**配置服务组件**

**a：/etc/neutron/neutron.conf**

```shell
cp /etc/neutron/neutron.conf{,.bak}
grep '^[a-z\[]' /etc/neutron/neutron.conf.bak >/etc/neutron/neutron.conf

openstack-config --set /etc/neutron/neutron.conf database connection mysql+pymysql://neutron:NEUTRON_DBPASS@controller/neutron
openstack-config --set /etc/neutron/neutron.conf DEFAULT core_plugin ml2
openstack-config --set /etc/neutron/neutron.conf DEFAULT service_plugins  
openstack-config --set /etc/neutron/neutron.conf DEFAULT  rpc_backend rabbit
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_host controller
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_password RABBIT_PASS
openstack-config --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_name service
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken username neutron
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken password NEUTRON_PASS
openstack-config --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_status_changes True
openstack-config --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_data_changes True
openstack-config --set /etc/neutron/neutron.conf  nova auth_url http://controller:35357
openstack-config --set /etc/neutron/neutron.conf  nova auth_type password
openstack-config --set /etc/neutron/neutron.conf  nova project_domain_name default
openstack-config --set /etc/neutron/neutron.conf  nova user_domain_name default
openstack-config --set /etc/neutron/neutron.conf  nova region_name RegionOne
openstack-config --set /etc/neutron/neutron.conf  nova project_name service
openstack-config --set /etc/neutron/neutron.conf  nova username nova
openstack-config --set /etc/neutron/neutron.conf  nova password NOVA_PASS
openstack-config --set /etc/neutron/neutron.conf  oslo_concurrency lock_path /var/lib/neutron/tmp
```

**配置 Modular Layer 2 (ML2) 插件**

**b: /etc/neutron/plugins/ml2/ml2_conf.ini**

```shell
cp /etc/neutron/plugins/ml2/ml2_conf.ini{,.bak}
grep '^[a-z\[]' /etc/neutron/plugins/ml2/ml2_conf.ini.bak >/etc/neutron/plugins/ml2/ml2_conf.ini

openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 type_drivers flat,vlan
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 tenant_network_types 
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers linuxbridge
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 extension_drivers port_security
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_flat flat_networks provider
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini securitygroup enable_ipset True
```

**配置Linuxbridge代理**

**c: /etc/neutron/plugins/ml2/linuxbridge_agent.ini**

```shell
cp /etc/neutron/plugins/ml2/linuxbridge_agent.ini{,.bak}
grep '^[a-Z\[]' /etc/neutron/plugins/ml2/linuxbridge_agent.ini.bak >/etc/neutron/plugins/ml2/linuxbridqe_agent.ini

openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini linux_bridge physical_interface_mappings provider:eth0 ###替换自己的网卡名称
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan False
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

**配置DHCP代理**

**d: /etc/neutron/dhcp_agent.ini**

```shell
cp /etc/neutron/dhcp_agent.ini{,.bak}
grep '^[a-Z\[]'  /etc/neutron/dhcp_agent.ini.bak >/etc/neutron/dhcp_agent.ini

openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT interface_driver   neutron.agent.linux.interface.BridgeInterfaceDriver
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT dhcp_driver   neutron.agent.linux.dhcp.Dnsmasq
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT enable_isolated_metadata   True
```

**配置元数据代理**

**e：/etc/neutron/metadata_agent.ini**

```shell
cp /etc/neutron/metadata_agent.ini{,.bak}
grep -EV '^$|#' /etc/neutron/metadata_agent.ini.bak > /etc/neutron/metadata_agent.ini

openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT nova_metadata_ip controller
openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT metadata_proxy_shared_secret METADATA_SECRET
```

**添加控制节点网络服务**

**f：/etc/nova/nova.conf**

```shell
openstack-config --set  /etc/nova/nova.conf neutron url    http://controller:9696
openstack-config --set  /etc/nova/nova.conf neutron auth_url   http://controller:35357
openstack-config --set  /etc/nova/nova.conf neutron auth_type  password
openstack-config --set  /etc/nova/nova.conf neutron project_domain_name    default
openstack-config --set  /etc/nova/nova.conf neutron user_domain_name    default
openstack-config --set  /etc/nova/nova.conf neutron region_name    RegionOne
openstack-config --set  /etc/nova/nova.conf neutron project_name    service
openstack-config --set  /etc/nova/nova.conf neutron username    neutron
openstack-config --set  /etc/nova/nova.conf neutron password    NEUTRON_PASS
openstack-config --set  /etc/nova/nova.conf neutron service_metadata_proxy    True
openstack-config --set  /etc/nova/nova.conf neutron metadata_proxy_shared_secret    METADATA_SECRET
```

#### **同步neutron数据库**

```shell
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

<!--可以ok即成功-->

<!--同步后进入数据库查看neutron库是否有数据-->

<!--忽略警告-->

#### 启动服务

```shell
systemctl restart openstack-nova-api.service
systemctl enable neutron-server.service \
  neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
  neutron-metadata-agent.service
systemctl start neutron-server.service \
  neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
  neutron-metadata-agent.service
```

### compute

#### 在计算节点安装组件(compute)

```shell
yum -y install openstack-neutron-linuxbridge ebtables ipset
```

#### 修改配置文件

**a./etc/neutron/neutron.conf**

```shell
cp /etc/neutron/neutron.conf{,.bak}
grep '^[a-Z\[]' /etc/neutron/neutron.conf.bak >/etc/neutron/neutron.conf

openstack-config --set /etc/neutron/neutron.conf DEFAULT rpc_backend    rabbit
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_host    controller
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_userid    openstack
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_password RABBIT_PASS
openstack-config --set /etc/neutron/neutron.conf DEFAULT auth_strategy    keystone
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_type    password
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name  default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name    default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_name    service
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken username    neutron
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken password    NEUTRON_PASS
openstack-config --set /etc/neutron/neutron.conf oslo_concurreny lock_path    /var/lib/neutron/tmp
```

**配置Linuxbridge代理**

**b: /etc/neutron/plugins/ml2/linuxbridge_agent.ini**

```shell
cp /etc/neutron/plugins/ml2/linuxbridge_agent.ini{,.bak}
grep '^[a-Z\[]' /etc/neutron/plugins/ml2/linuxbridge_agent.ini.bak >/etc/neutron/plugins/ml2/linuxbridqe_agent.ini

openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini linux_bridge physical_interface_mappings provider:eth0  ###替换自己的网卡名称
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan False
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

**添加计算节点网络服务**

**c：/etc/nova/nova.conf**

```shell
openstack-config --set  /etc/nova/nova.conf neutron url    http://controller:9696
openstack-config --set  /etc/nova/nova.conf neutron auth_url    http://controller:35357
openstack-config --set  /etc/nova/nova.conf neutron auth_type    password
openstack-config --set  /etc/nova/nova.conf neutron project_domain_name    default
openstack-config --set  /etc/nova/nova.conf neutron user_domain_name    default
openstack-config --set  /etc/nova/nova.conf neutron region_name    RegionOne
openstack-config --set  /etc/nova/nova.conf neutron project_name    service
openstack-config --set  /etc/nova/nova.conf neutron username    neutron
openstack-config --set  /etc/nova/nova.conf neutron password    NEUTRON_PASS
```

#### 启动服务

```shell
重新启动计算服务：
	systemctl restart openstack-nova-compute.service
启动Linuxbridge代理并配置它开机自启动：
	systemctl enable neutron-linuxbridge-agent.service
	systemctl start neutron-linuxbridge-agent.service
```

## Dashboard服务

(根据情况确定安装控制节点还是计算节点）

> 此次安装在compute上

### 安装openstack-dashboard

```shell
yum -y install openstack-dashboard
```

### 修改配置文件

**/etc/openstack-dashboard/local_settings**

```shell
#在 controller 节点上配置仪表盘以使用 OpenStack 服务
OPENSTACK_HOST = "controller"
#允许所有主机访问仪表板
ALLOWED_HOSTS = ['*', ]
#配置 memcached 会话存储服务
CACHES = {
	 	 'default': {
        		'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
 	  	 	     'LOCATION': 'controller:11211',
			   }
}
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
#启用第3版认证API
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
#启用对域的支持
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
#配置API版本
OPENSTACK_API_VERSIONS = {
		   "identity": 3,
	 	   "image": 2,
	   	   "volume": 2,
		}
#通过仪表盘创建用户时的默认域配置为 default
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "default"
#通过仪表盘创建的用户默认角色配置为 user
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
#如果您选择网络参数1，禁3层网络服务：
OPENSTACK_NEUTRON_NETWORK = {
	    ...
'enable_router': False,
'enable_quotas': False,
'enable_distributed_router': False,
'enable_ha_router': False,
'enable_lb': False,
'enable_firewall': False,
'enable_vpn': False,
'enable_fip_topology_check': False,
		}
#可以选择性地配置时区：
TIME_ZONE = "Asia/Shanghai"
```

### 启动服务

```shell
systemctl restart httpd.service
```

### 控制节点

```shell
systemctl restart memcached.service
```

\###如服务无法启动,在/etc/httpd/conf.d/openstack-dashboard.conf文件下添加以下字段###

​				WSGIApplicationGroup %{GLOBAL}

## Cinder块存储服务

cinder-volume LVM,nfs,gfs,ceph



cinder-api:				接收外部的api请求

cinder-volume:		提供存储空间

cinder-scheduler	调度器,决定将要分配的空间由哪一个cinder-volume提供

cinder-backup		备份存储

### controller

#### 在控制节点创建Cinder服务数据库(controller)

```shell
a.登录mysql数据库
	mysql -u root -p000000
b.创建cinder数据库
	CREATE DATABASE cinder;
c.创建cinder数据库用户，使其可以对cinder数据库有完全控制权限
	GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' IDENTIFIED BY 'CINDER_DBPASS';
	GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' IDENTIFIED BY 'CINDER_DBPASS';
d.退出数据库
	exit
```

#### 在keystone创建系统用户(glance, nova, neutron,cinder)关联角色

```shell
openstack user create --domain default --password CINDER_PASS cinder
openstack role add --project service --user cinder admin
```

#### 在keystone上创建服务和注册api

```shell
 openstack service create --name cinder \
  --description "OpenStack Block Storage" volume
openstack service create --name cinderv2 \
  --description "OpenStack Block Storage" volumev2
  
openstack endpoint create --region RegionOne \
  volume public http://controller:8776/v1/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  volume internal http://controller:8776/v1/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  volume admin http://controller:8776/v1/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  volumev2 public http://controller:8776/v2/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  volumev2 internal http://controller:8776/v2/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  volumev2 admin http://controller:8776/v2/%\(tenant_id\)s
```

#### 安装服务相应软件包

```shell
yum -y install openstack-cinder
```

#### 修改配置文件

**/etc/cinder/cinder.conf**

```shell
cp /etc/cinder/cinder.conf{,.bak}
grep -Ev '^$|#' /etc/cinder/cinder.conf.bak >/etc/cinder/cinder.conf

openstack-config --set /etc/cinder/cinder.conf database connection    mysql+pymysql://cinder:CINDER_DBPASS@controller/cinder
openstack-config --set /etc/cinder/cinder.conf DEFAULT rpc_backend    rabbit
openstack-config --set /etc/cinder/cinder.conf oslo_messaging_rabbit rabbit_host    controller
openstack-config --set /etc/cinder/cinder.conf oslo_messaging_rabbit rabbit_userid    openstack
openstack-config --set /etc/cinder/cinder.conf oslo_messaging_rabbit rabbit_password    RABBIT_PASS
openstack-config --set /etc/cinder/cinder.conf DEFAULT auth_strategy    keystone
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_uri    http://controller:5000
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_url    http://controller:35357
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken memcached_servers    controller:11211
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_type    password
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken project_domain_name    default
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken user_domain_name    default
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken project_name    service
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken username    cinder
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken password    CINDER_PASS
openstack-config --set /etc/cinder/cinder.conf DEFAULT my_ip  172.16.100.28 ###替换自己IP
openstack-config --set /etc/cinder/cinder.conf oslo_concurrency lock_path    /var/lib/cinder/tmp
```

#### 同步cinder数据库

```shell
su -s /bin/sh -c "cinder-manage db sync" cinder
```

<!--同步后进入数据库查看cinder库是否有数据-->

<!--忽略警告-->

修改nova配置文件

**/etc/nova/nova.conf**

```
openstack-config --set /etc/cinder/cinder.conf cinder os_region_name  RegionOne
```

#### 启动服务

```shell
systemctl restart openstack-nova-api.service
systemctl enable openstack-cinder-api.service openstack-cinder-scheduler.service
systemctl start openstack-cinder-api.service openstack-cinder-scheduler.service
```

### compute

#### 在计算节点安装存储节点(compute)

```shell
#安装支持的工具包
yum -y install lvm2
systemctl enable lvm2-lvmetad.service
systemctl start lvm2-lvmetad.service
```

```shell
在计算节点新建一块硬盘
###如果在VM添加的硬盘没有扫描出来,用以下命令
echo '- - -' >/sys/class/scsi_host/host{0,1,2}/scan

[root@compute ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0              11:0    1 1024M  0 rom  
vda             252:0    0   80G  0 disk 
├─vda1          252:1    0    1G  0 part /boot
└─vda2          252:2    0   79G  0 part 
  ├─centos-root 253:0    0 49.9G  0 lvm  /
  ├─centos-swap 253:1    0  4.8G  0 lvm  [SWAP]
  └─centos-home 253:2    0 24.3G  0 lvm  /home
vdb             252:16   0   80G  0 disk  ###新添加的硬盘
```

#### 创建LVM 物理卷

```shell
[root@compute ~]#  pvcreate /dev/vdb
  Physical volume "/dev/vdb" successfully created.
```

#### 创建 LVM 卷组 cinder-volumes

```shell
[root@compute ~]# vgcreate cinder-volumes /dev/vdb
  Volume group "cinder-volumes" successfully created
```

#### 添加一个过滤器(只有实例可以访问块存储卷组)

**/etc/lvm/lvm.conf** 

```shell
###大概文件130行左右
filter = [ "a/vdb/", "r/.*/"]
######多盘######filter = [ "a/vdb/", "a/vdc/", "r/.*/"]
```

#### 安装服务相应软件包

```shell
yum -y install openstack-cinder targetcli python-keystone
```

#### 修改配置文件

**/etc/cinder/cinder.conf**

```shell
cp /etc/cinder/cinder.conf{,.bak}
grep -Ev '^$|#' /etc/cinder/cinder.conf.bak >/etc/cinder/cinder.conf

openstack-config --set /etc/cinder/cinder.conf database connection mysql+pymysql://cinder:CINDER_DBPASS@controller/cinder
openstack-config --set /etc/cinder/cinder.conf DEFAULT rpc_backend rabbit
openstack-config --set /etc/cinder/cinder.conf oslo_messaging_rabbit rabbit_host controller
openstack-config --set /etc/cinder/cinder.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/cinder/cinder.conf oslo_messaging_rabbit rabbit_password RABBIT_PASS
openstack-config --set /etc/cinder/cinder.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_type password
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken project_name service
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken username cinder
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken password CINDER_PASS
openstack-config --set /etc/cinder/cinder.conf DEFAULT my_ip 172.16.10.29
openstack-config --set /etc/cinder/cinder.conf lvm volume_driver cinder.volume.drivers.lvm.LVMVolumeDriver
openstack-config --set /etc/cinder/cinder.conf lvm volume_group cinder-volumes
openstack-config --set /etc/cinder/cinder.conf lvm iscsi_protocol  iscsi
openstack-config --set /etc/cinder/cinder.conf lvm iscsi_helper lioadm
openstack-config --set /etc/cinder/cinder.conf DEFAULT enabled_backends lvm
openstack-config --set /etc/cinder/cinder.conf DEFAULT glance_api_servers http://controller:9292
openstack-config --set /etc/cinder/cinder.conf oslo_concurrency lock_path /var/lib/cinder/tmp
```

#### 启动服务

```shell
systemctl enable openstack-cinder-volume.service target.service
systemctl start openstack-cinder-volume.service target.service
```

#### 在controller检查是否成功

```shell
[root@controller ~]# cinder service-list
+------------------+-------------+------+---------+-------+----------------------------+-----------------+
|      Binary      |     Host    | Zone |  Status | State |         Updated_at         | Disabled Reason |
+------------------+-------------+------+---------+-------+----------------------------+-----------------+
| cinder-scheduler |  controller | nova | enabled |   up  | 2021-11-24T02:01:16.000000 |        -        |
|  cinder-volume   | compute@lvm | nova | enabled |   up  | 2021-11-24T02:01:23.000000 |        -        |
+------------------+-------------+------+---------+-------+----------------------------+-----------------+
```

## 启动一个实例

### 1.创建网络

```shell
neutron net-create --shared --provider:physical_network provider  --provider:network_type flat zzz
neutron subnet-create --name  zzz123\
	  --allocation-pool start=172.16.100.101,end=172.16.100.250 \
	  --dns-nameserver 114.114.114.114 --gateway 172.16.100.254 \
 	  zzz 172.16.100.0/24
```

### 2.创建云主机的硬件配置方案

```shell
 openstack flavor create --id 0 --vcpus 1 --ram 64 --disk 1 m1.nano
```

### 3.创建密钥对

```shell
ssh-keygen -q -N "" -f ~/.ssh/id_rsa
openstack keypair create --public-key ~/.ssh/id_rsa.pub mykey
验证是否添加:  openstack keypair list
```

### 4.创建安全组规则

```shell
允许 ICMP (ping)：
openstack security group rule create --proto icmp default
允许安全 shell (SSH) 的访问：
openstack security group rule create --proto tcp --dst-port 22 default
```

### 5.启动一个实例

```shell
查看网络id:  neutron net-list
###网络id=PROVIDER_NET_ID
创建实例:
openstack server create --flavor m1.nano --image cirros \
  --nic net-id=PROVIDER_NET_ID --security-group default \
  --key-name mykey provider-instance
  
  
###查看已创建实例###
[root@controller ~]# openstack server list
+--------------------------------------+-------------------+--------+--------------------+
| ID                                   | Name              | Status | Networks           |
+--------------------------------------+-------------------+--------+--------------------+
| f6fa8d1a-13e7-46c9-b5e3-78cfdc3497d5 | provider-instance | ACTIVE | zzz=172.16.100.102 |
+--------------------------------------+-------------------+--------+--------------------+
```

