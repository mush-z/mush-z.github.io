---
title: openstack-Train.md
date: 2022-04-17 14:47:45
tags: openstack-T
category: Linux
---

# OpenStack T版搭建

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
|     `PLACEMENT_PASS`     |            Placement的密码             |

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
[openstack-t]
name=openstack-t
baseurl=ftp://172.16.100.66/openstack-t
gpgcheck=0

#compute
[root@compute ~]# cat /etc/yum.repos.d/http.repo 
[centos]
name=centos
baseurl=ftp://172.16.100.66/centos
gpgcheck=0
[openstack-t]
name=openstack-t
baseurl=ftp://172.16.100.66/openstack-t
gpgcheck=0
```

>此处用的是配置本地源，一个Centos，一个OpenStack源
>
>附上国内阿里源https://mirrors.aliyun.com/centos/7/cloud/x86_64/openstack-train/

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

安装

```shell
yum -y install chrony
```

```shell
vi /etc/chrony.conf

server ntp.aliyun.com iburst
###大概26行左右###
allow 172.16.100.0/24
```

#### compute

安装

```shell
yum -y install chrony
```

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
grep '^[a-z\[]'  /etc/keystone/keystone.conf.bak > /etc/keystone/keystone.conf

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
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone


keystone-manage bootstrap --bootstrap-password ADMIN_PASS \
  --bootstrap-admin-url http://controller:5000/v3/ \
  --bootstrap-internal-url http://controller:5000/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne
```

### 配置Apache HTTP服务器

```shell
echo "ServerName controller" >>/etc/httpd/conf/httpd.conf
ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
```

#### 启动HTTP服务

```shell
systemctl enable httpd.service
systemctl start httpd.service
```

### 创建并编辑admin-openrc文件

```shell
[root@controller ~]# cat admin-openrc 
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

```shell
source admin-openrc
```

### 创建域,项目,用户,角色

```shell
openstack domain create --description "An Example Domain" example

openstack project create --domain default \
  --description "Service Project" service

openstack project create --domain default \
  --description "Demo Project" myproject
  
openstack user create --domain default \
  --password MYUSER myuser

openstack role create myrole

openstack role add --project myproject --user myuser myrole
```

### 取消前面设置的环境变量

```shell
unset OS_AUTH_URL OS_PASSWORD
```

### 作为admin用户和myuser用户请求身份令牌

```shell
openstack --os-auth-url http://controller:5000/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name admin --os-username admin token issue \
  --os-password ADMIN_PASS
openstack --os-auth-url http://controller:5000/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name myproject --os-username myuser token issue \
  --os-password MYUSER
```

### 创建并编辑demo-openrc文件

```shell
[root@controller ~]# cat demo-openrc 
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=myproject
export OS_USERNAME=myuser
export OS_PASSWORD=MYUSER
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

### 验证环境变量是否成功

```shell
source admin-openrc
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
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken www_authenticate_uri http://controller:5000
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_url http://controller:5000
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_type password
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken project_domain_name Default
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken user_domain_name Default
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken project_name service
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken username glance
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken password GLANCE_PASS
openstack-config --set /etc/glance/glance-api.conf  paste_deploy flavor keystone
openstack-config --set /etc/glance/glance-api.conf glance_store stores file,http
openstack-config --set /etc/glance/glance-api.conf glance_store default_store file
openstack-config --set /etc/glance/glance-api.conf glance_store filesystem_store_datadir /var/lib/glance/images/
```

```shell
chmod 777 /etc/glance/glance-registry.conf
```

### 同步glance数据库

```shell
su -s /bin/sh -c "glance-manage db_sync" glance
```

<!--同步后进入数据库查看glance库是否有数据-->

<!--忽略警告-->

### 启动服务

```shell
systemctl enable openstack-glance-api.service
systemctl start openstack-glance-api.service
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

## Placement

#### 在控制节点创建placement数据库(controller)

```shell
a.登录mysql数据库
	mysql -u root -p000000
b.创建glance数据库
	CREATE DATABASE placement;
c.创建glance数据库用户，使其可以对glance数据库有完全控制权限
	GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' IDENTIFIED BY 'PLACEMENT_DBPASS';
	GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' IDENTIFIED BY 'PLACEMENT_DBPASS';
d.退出数据库
	exit
```

#### 在keystone创建系统用户关联角色

```shell
openstack user create --domain default --password PLACEMENT_PASS placement
openstack role add --project service --user placement admin
```

#### 在keystone上创建服务和注册api

```shell
openstack service create --name placement --description "Placement API" placement

openstack endpoint create --region RegionOne \
  placement public http://controller:8778

openstack endpoint create --region RegionOne \
  placement internal http://controller:8778
  
openstack endpoint create --region RegionOne \
  placement admin http://controller:8778
```

#### 安装软件包

```shell
yum -y install openstack-placement-api
```

#### 修改配置文件

**/etc/placement/placement.conf**

```shell
cp /etc/placement/placement.conf{,.bak}
grep '^[a-Z\[]' /etc/placement/placement.conf.bak >/etc/placement/placement.conf

openstack-config --set /etc/placement/placement.conf placement_database connection mysql+pymysql://placement:PLACEMENT_DBPASS@controller/placement 
openstack-config --set /etc/placement/placement.conf api auth_strategy keystone
openstack-config --set /etc/placement/placement.conf keystone_authtoken auth_url http://controller:5000/v3
openstack-config --set /etc/placement/placement.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/placement/placement.conf keystone_authtoken auth_type password
openstack-config --set /etc/placement/placement.conf keystone_authtoken project_domain_name Default
openstack-config --set /etc/placement/placement.conf keystone_authtoken user_domain_name Default
openstack-config --set /etc/placement/placement.conf keystone_authtoken project_name service
openstack-config --set /etc/placement/placement.conf keystone_authtoken username placement
openstack-config --set /etc/placement/placement.conf keystone_authtoken password PLACEMENT_PASS
```

#### 同步数据库

```shell
su -s /bin/sh -c "placement-manage db sync" placement
```

#### 编辑00-placement-api.conf文件

```shell
###在结尾添加
vim /etc/httpd/conf.d/00-placement-api.conf
....
<Directory /usr/bin>
<IfVersion >= 2.4>
   Require all granted
</IfVersion>
<IfVersion < 2.4>
   Order allow,deny
   Allow from all
</IfVersion>
</Directory>
```

#### 重启httd服务

```shell
systemctl restart httpd
```

#### 验证

```shell
[root@controller placement]# placement-status upgrade check
+----------------------------------+
| Upgrade Check Results            |
+----------------------------------+
| Check: Missing Root Provider IDs |
| Result: Success                  |
| Details: None                    |
+----------------------------------+
| Check: Incomplete Consumers      |
| Result: Success                  |
| Details: None                    |
+----------------------------------+
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
b.创建nova_api,nova,nova_cell0数据库
	CREATE DATABASE nova_api;
	CREATE DATABASE nova;
	CREATE DATABASE nova_cell0;
c.创建nova_api，nova数据库用户，使其可以对nova_api,nova数据库有完全控制权限
	GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';
	GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';
	GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';
	GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';
	GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';
	GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';
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
openstack service create --name nova --description "OpenStack Compute" compute
openstack endpoint create --region RegionOne \
  compute public http://controller:8774/v2.1
openstack endpoint create --region RegionOne \
  compute internal http://controller:8774/v2.1
openstack endpoint create --region RegionOne \
  compute admin http://controller:8774/v2.1
```

#### 安装相关服务软件包

openstack-nova-conductor 负责数据库
openstack-nova-novncproxy  负责云主机连接
openstack-nova-scheduler  负责调度调度

```shell
yum -y install openstack-nova-api openstack-nova-conductor openstack-nova-novncproxy openstack-nova-scheduler
```

#### 修改配置文件

**/etc/nova/nova.conf**

```shell
cp /etc/nova/nova.conf{,.bak}
grep '^[a-Z\[]' /etc/nova/nova.conf.bak >/etc/nova/nova.conf

openstack-config --set /etc/nova/nova.conf DEFAULT enabled_apis osapi_compute,metadata
openstack-config --set /etc/nova/nova.conf api_database connection mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api
openstack-config --set /etc/nova/nova.conf database connection mysql+pymysql://nova:NOVA_DBPASS@controller/nova
openstack-config --set /etc/nova/nova.conf DEFAULT transport_url rabbit://openstack:RABBIT_PASS@controller:5672/
openstack-config --set /etc/nova/nova.conf api auth_strategy keystone
openstack-config --set /etc/nova/nova.conf keystone_authtoken www_authenticate_uri http://controller:5000/
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_url http://controller:5000/
openstack-config --set /etc/nova/nova.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_type password
openstack-config --set /etc/nova/nova.conf keystone_authtoken project_domain_name Default
openstack-config --set /etc/nova/nova.conf keystone_authtoken user_domain_name Default
openstack-config --set /etc/nova/nova.conf keystone_authtoken project_name service
openstack-config --set /etc/nova/nova.conf keystone_authtoken username nova
openstack-config --set /etc/nova/nova.conf keystone_authtoken password NOVA_PASS
openstack-config --set /etc/nova/nova.conf DEFAULT my_ip 172.16.100.28		###替换自己的IP!!!
openstack-config --set /etc/nova/nova.conf DEFAULT use_neutron True
openstack-config --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
openstack-config --set /etc/nova/nova.conf vnc enabled true
openstack-config --set /etc/nova/nova.conf vnc server_listen '$my_ip'
openstack-config --set /etc/nova/nova.conf vnc server_proxyclient_address '$my_ip'
openstack-config --set /etc/nova/nova.conf glance api_servers http://controller:9292
openstack-config --set /etc/nova/nova.conf oslo_concurrency lock_path /var/lib/nova/tmp
openstack-config --set /etc/nova/nova.conf placement region_name RegionOne
openstack-config --set /etc/nova/nova.conf placement project_domain_name Default
openstack-config --set /etc/nova/nova.conf placement project_name service
openstack-config --set /etc/nova/nova.conf placement auth_type password
openstack-config --set /etc/nova/nova.conf placement user_domain_name Default
openstack-config --set /etc/nova/nova.conf placement auth_url http://controller:5000/v3
openstack-config --set /etc/nova/nova.conf placement username placement
openstack-config --set /etc/nova/nova.conf placement password PLACEMENT_PASS
```

#### 同步数据库

```shell
su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
su -s /bin/sh -c "nova-manage db sync" nova
```

<!--同步后进入数据库查看nova库是否有数据-->

<!--忽略警告-->

#### 验证 nova cell0 和 cell1 是否正确注册

```shell
[root@controller ~]# su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova
+-------+--------------------------------------+------------------------------------------+-------------------------------------------------+----------+
|  Name |                 UUID                 |              Transport URL               |               Database Connection               | Disabled |
+-------+--------------------------------------+------------------------------------------+-------------------------------------------------+----------+
| cell0 | 00000000-0000-0000-0000-000000000000 |                  none:/                  | mysql+pymysql://nova:****@controller/nova_cell0 |  False   |
| cell1 | e9c5a4d7-880f-4783-958b-85d44bad22b1 | rabbit://openstack:****@controller:5672/ |    mysql+pymysql://nova:****@controller/nova    |  False   |
+-------+--------------------------------------+------------------------------------------+-------------------------------------------------+----------+
```

#### 启动服务

```shell
systemctl enable \
    openstack-nova-api.service \
    openstack-nova-scheduler.service \
    openstack-nova-conductor.service \
    openstack-nova-novncproxy.service
 systemctl start \
    openstack-nova-api.service \
    openstack-nova-scheduler.service \
    openstack-nova-conductor.service \
    openstack-nova-novncproxy.service
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

openstack-config --set /etc/nova/nova.conf DEFAULT enabled_apis osapi_compute,metadata
openstack-config --set /etc/nova/nova.conf DEFAULT transport_url rabbit://openstack:RABBIT_PASS@controller
openstack-config --set /etc/nova/nova.conf api auth_strategy keystone
openstack-config --set /etc/nova/nova.conf keystone_authtoken www_authenticate_uri http://controller:5000
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_url http://controller:5000/
openstack-config --set /etc/nova/nova.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_type password
openstack-config --set /etc/nova/nova.conf keystone_authtoken project_domain_name Default
openstack-config --set /etc/nova/nova.conf keystone_authtoken user_domain_name Default
openstack-config --set /etc/nova/nova.conf keystone_authtoken project_name service
openstack-config --set /etc/nova/nova.conf keystone_authtoken username nova
openstack-config --set /etc/nova/nova.conf keystone_authtoken password NOVA_PASS
openstack-config --set /etc/nova/nova.conf DEFAULT my_ip 172.16.100.29   ###替换自己compute IP
openstack-config --set /etc/nova/nova.conf DEFAULT use_neutron True
openstack-config --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
openstack-config --set /etc/nova/nova.conf vnc enabled True
openstack-config --set /etc/nova/nova.conf vnc vncserver_listen 0.0.0.0
openstack-config --set /etc/nova/nova.conf vnc vncserver_proxyclient_address '$my_ip'
openstack-config --set /etc/nova/nova.conf vnc novncproxy_base_url http://172.16.100.28:6080/vnc_auto.html
###服务器组件监听所有的 IP 地址，而代理组件仅仅监听计算节点管理网络接口的 IP 地址。基本的 URL 指示您可以使用 web 浏览器访问位于该计算节点上实例的远程控制台的位置。
openstack-config --set /etc/nova/nova.conf glance api_servers http://controller:9292
openstack-config --set /etc/nova/nova.conf oslo_concurrency lock_path /var/lib/nova/tmp
openstack-config --set /etc/nova/nova.conf placement region_name RegionOne
openstack-config --set /etc/nova/nova.conf placement project_domain_name Default
openstack-config --set /etc/nova/nova.conf placement project_name service
openstack-config --set /etc/nova/nova.conf placement auth_type password
openstack-config --set /etc/nova/nova.conf placement user_domain_name Default
openstack-config --set /etc/nova/nova.conf placement auth_url http://controller:5000/v3
openstack-config --set /etc/nova/nova.conf placement username placement
openstack-config --set /etc/nova/nova.conf placement password PLACEMENT_PASS
```

#### 确定您的计算节点是否支持虚拟机的硬件加速

```shell
egrep -c '(vmx|svm)' /proc/cpuinfo
###如不为0，则您的计算节点支持硬件加速，这通常不需要额外的配置

###如果返回值为0，则您的计算节点不支持硬件加速，您必须配置libvirt为使用 QEMU 而不是 KVM，执行以下命令！
openstack-config --set /etc/nova/nova.conf libvirt virt_type qemu
```



#### 启动服务

```shell
systemctl enable libvirtd.service openstack-nova-compute.service
systemctl start libvirtd.service openstack-nova-compute.service
```

### 在控制节点查看

```shell
[root@controller ~]# openstack compute service list --service nova-compute
+----+--------------+---------+------+---------+-------+----------------------------+
| ID | Binary       | Host    | Zone | Status  | State | Updated At                 |
+----+--------------+---------+------+---------+-------+----------------------------+
| 10 | nova-compute | compute | nova | enabled | up    | 2021-12-17T06:59:11.000000 |
+----+--------------+---------+------+---------+-------+----------------------------+
```

### 发现计算节点主机

**###添加新计算节点时，您必须在控制器节点上运行以注册这些新计算节点**

```shell
[root@controller ~]# su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting computes from cell 'cell1': e9c5a4d7-880f-4783-958b-85d44bad22b1
Checking host mapping for compute host 'compute': b5a4c6b3-ad48-4e74-b5f4-76cb6f55f8d3
Creating host mapping for compute host 'compute': b5a4c6b3-ad48-4e74-b5f4-76cb6f55f8d3
Found 1 unmapped computes in cell: e9c5a4d7-880f-4783-958b-85d44bad22b1
```

**###或者配置适当的间隔执行** 

#这里配置的是300秒，可以自行设置

```shell
openstack-config --set /etc/nova/nova.conf scheduler discover_hosts_in_cells_interval 300
```

### 查看nova是否正常

```shell
[root@controller ~]# nova service-list
+--------------------------------------+----------------+------------+----------+---------+-------+----------------------------+-----------------+-------------+
| Id                                   | Binary         | Host       | Zone     | Status  | State | Updated_at                 | Disabled Reason | Forced down |
+--------------------------------------+----------------+------------+----------+---------+-------+----------------------------+-----------------+-------------+
| c606ddf6-2213-459e-b07e-e8219c9c24fc | nova-conductor | controller | internal | enabled | up    | 2021-12-17T07:13:17.000000 | -               | False       |
| 6608eb09-0130-4cdd-8311-f53082e14508 | nova-scheduler | controller | internal | enabled | up    | 2021-12-17T07:13:18.000000 | -               | False       |
| 2012503b-9c09-4575-96e1-991b1f8dbf5e | nova-compute   | compute    | nova     | enabled | up    | 2021-12-17T07:13:21.000000 | -               | False       |
+--------------------------------------+----------------+------------+----------+---------+-------+----------------------------+-----------------+-------------+

```

## Neutron网络服务

neutron-server端(9696)	api:接受和响应外部的网络管理请求

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
openstack-config --set /etc/neutron/neutron.conf DEFAULT transport_url rabbit://openstack:RABBIT_PASS@controller
openstack-config --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken www_authenticate_uri http://controller:5000
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://controller:5000
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_name service
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken username neutron
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken password NEUTRON_PASS
openstack-config --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_status_changes True
openstack-config --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_data_changes True
openstack-config --set /etc/neutron/neutron.conf  nova auth_url http://controller:5000
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

openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini linux_bridge physical_interface_mappings provider:ens3 ###替换自己的网卡名称
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan False
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

**配置DHCP代理**

**d: /etc/neutron/dhcp_agent.ini**

```shell
cp /etc/neutron/dhcp_agent.ini{,.bak}
grep '^[a-Z\[]'  /etc/neutron/dhcp_agent.ini.bak >/etc/neutron/dhcp_agent.ini

openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT interface_driver linuxbridge
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT dhcp_driver neutron.agent.linux.dhcp.Dnsmasq
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT enable_isolated_metadata True
```

#### 配置内核

```shell
echo -e "net.bridge.bridge-nf-call-iptables = 1\nnet.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf

[root@controller ~]# modprobe br_netfilter
[root@controller ~]# sysctl -p
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
```

**配置元数据代理**

**e：/etc/neutron/metadata_agent.ini**

```shell
cp /etc/neutron/metadata_agent.ini{,.bak}
grep '^[a-Z\[]'  /etc/neutron/metadata_agent.ini.bak > /etc/neutron/metadata_agent.ini

openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT nova_metadata_host controller
openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT metadata_proxy_shared_secret METADATA_SECRET
```

**添加控制节点网络服务**

**f：/etc/nova/nova.conf**

```shell
openstack-config --set  /etc/nova/nova.conf neutron url http://controller:9696
openstack-config --set  /etc/nova/nova.conf neutron auth_url http://controller:5000
openstack-config --set  /etc/nova/nova.conf neutron auth_type  password
openstack-config --set  /etc/nova/nova.conf neutron project_domain_name  default
openstack-config --set  /etc/nova/nova.conf neutron user_domain_name  default
openstack-config --set  /etc/nova/nova.conf neutron region_name   RegionOne
openstack-config --set  /etc/nova/nova.conf neutron project_name  service
openstack-config --set  /etc/nova/nova.conf neutron username  neutron
openstack-config --set  /etc/nova/nova.conf neutron password  NEUTRON_PASS
openstack-config --set  /etc/nova/nova.conf neutron service_metadata_proxy  True
openstack-config --set  /etc/nova/nova.conf neutron metadata_proxy_shared_secret  METADATA_SECRET
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

openstack-config --set /etc/neutron/neutron.conf DEFAULT transport_url rabbit://openstack:RABBIT_PASS@controller
openstack-config --set /etc/neutron/neutron.conf DEFAULT auth_strategy  keystone
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken www_authenticate_uri http://controller:5000
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://controller:5000
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

openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini linux_bridge physical_interface_mappings provider:ens3  ###替换自己的网卡名称
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan False
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroupfirewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

#### 配置内核

```shell
echo -e "net.bridge.bridge-nf-call-iptables = 1\nnet.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf

[root@controller ~]# modprobe br_netfilter
[root@controller ~]# sysctl -p
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
```

**添加计算节点网络服务**

**c：/etc/nova/nova.conf**

```shell
openstack-config --set  /etc/nova/nova.conf neutron url  http://controller:9696
openstack-config --set  /etc/nova/nova.conf neutron auth_url http://controller:5000
openstack-config --set  /etc/nova/nova.conf neutron auth_type  password
openstack-config --set  /etc/nova/nova.conf neutron project_domain_name  default
openstack-config --set  /etc/nova/nova.conf neutron user_domain_name   default
openstack-config --set  /etc/nova/nova.conf neutron region_name   RegionOne
openstack-config --set  /etc/nova/nova.conf neutron project_name  service
openstack-config --set  /etc/nova/nova.conf neutron username  neutron
openstack-config --set  /etc/nova/nova.conf neutron password  NEUTRON_PASS
```

#### 启动服务

```shell
	systemctl restart openstack-nova-compute.service
	systemctl enable neutron-linuxbridge-agent.service
	systemctl start neutron-linuxbridge-agent.service
```

### 查看服务（controller）

```shell
[root@controller ~]# openstack network agent list
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host       | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| 1a758c8f-87b1-4e43-9f7b-445a8d38c454 | DHCP agent         | controller | nova              | :-)   | UP    | neutron-dhcp-agent        |
| 590d524d-6188-4e09-a5f8-25aa83568192 | Linux bridge agent | controller | None              | :-)   | UP    | neutron-linuxbridge-agent |
| add2e0e2-71ba-4e29-b454-77435f1266e2 | Metadata agent     | controller | None              | :-)   | UP    | neutron-metadata-agent    |
| fad9d9ca-85b4-4d12-9bf8-852f59e1ed0f | Linux bridge agent | compute    | None              | :-)   | UP    | neutron-linuxbridge-agent |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+

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
	   	   "volume": 3,
		}
#WEB界面配置
WEBROOT = "/dashboard"
#通过仪表盘创建用户时的默认域配置为 default
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default"
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

**/etc/httpd/conf.d/openstack-dashboard.conf**

```shell
sed -i "4i WSGIApplicationGroup %{GLOBAL}" /etc/httpd/conf.d/openstack-dashboard.conf
```

### 启动服务

```shell
systemctl restart httpd.service
```

### 控制节点

```shell
systemctl restart memcached.service
```

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
openstack service create --name cinderv2 \
  --description "OpenStack Block Storage" volumev2
openstack service create --name cinderv3 \
  --description "OpenStack Block Storage" volumev3
  
openstack endpoint create --region RegionOne \
  volumev2 public http://controller:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne \
  volumev2 internal http://controller:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne \
  volumev2 admin http://controller:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne \
  volumev3 public http://controller:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionOne \
  volumev3 internal http://controller:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionOne \
  volumev3 admin http://controller:8776/v3/%\(project_id\)s
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
openstack-config --set /etc/cinder/cinder.conf DEFAULT transport_url rabbit://openstack:RABBIT_PASS@controller
openstack-config --set /etc/cinder/cinder.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken www_authenticate_uri http://controller:5000
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_url http://controller:5000
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken memcached_servers   controller:11211
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_type  password
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken project_domain_name   default
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken user_domain_name   default
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken project_name   service
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken username    cinder
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken password    CINDER_PASS
openstack-config --set /etc/cinder/cinder.conf DEFAULT my_ip  172.16.100.28 ###替换自己IP
openstack-config --set /etc/cinder/cinder.conf oslo_concurrency lock_path  /var/lib/cinder/tmp
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
yum -y install lvm2 device-mapper-persistent-data
systemctl enable lvm2-lvmetad.service
systemctl start lvm2-lvmetad.service
```

```shell
在计算节点新建一块硬盘
###如果在VM添加的硬盘没有扫描出来,用以下命令
echo '- - -' >/sys/class/scsi_host/host0/scan
echo '- - -' >/sys/class/scsi_host/host1/scan
echo '- - -' >/sys/class/scsi_host/host2/scan

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
openstack-config --set /etc/cinder/cinder.conf DEFAULT transport_url rabbit://openstack:RABBIT_PASS@controller
openstack-config --set /etc/cinder/cinder.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken www_authenticate_uri http://controller:5000
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_url http://controller:5000
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_type password
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken project_name service
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken username cinder
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken password CINDER_PASS
openstack-config --set /etc/cinder/cinder.conf DEFAULT my_ip 172.16.100.29 ###替换自己IP
openstack-config --set /etc/cinder/cinder.conf lvm volume_driver cinder.volume.drivers.lvm.LVMVolumeDriver
openstack-config --set /etc/cinder/cinder.conf lvm volume_group cinder-volumes
openstack-config --set /etc/cinder/cinder.conf lvm target_protocol  iscsi
openstack-config --set /etc/cinder/cinder.conf lvm target_helper lioadm
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
[root@controller ~]#  cinder service-list
+------------------+-------------+------+---------+-------+----------------------------+---------+---------+
| Binary           | Host        | Zone | Status  | State | Updated_at                 | Cluster | Disabled Reason | Backend State |
+------------------+-------------+------+---------+-------+----------------------------+---------+---------+
| cinder-scheduler | controller  | nova | enabled | up    | 2021-12-17T12:06:39.000000 | -       | -               |               |
| cinder-volume    | compute@lvm | nova | enabled | up    | 2021-12-17T12:06:37.000000 | -       | -               | up            |
+------------------+-------------+------+---------+-------+----------------------------+---------+---------+

```

## 启动一个实例

### 1.创建网络

```shell
neutron net-create --shared --provider:physical_network provider  --provider:network_type flat zzz
neutron subnet-create --name  zzz123\
	  --allocation-pool start=172.16.100.101,end=172.16.100.250 \
	  --dns-nameserver 114.114.114.114 --gateway 172.16.100.254 \
 	  zzz 172.16.100.0/24

openstack network create --share --provider-physical-network provider --provider-network-type flat zzz
openstack subnet create --ip-version 4 --allocation-pool start=172.16.100.101,end=172.16.100.250 --dns-nameserver 114.114.114.114 --gateway 172.16.100.254 --subnet-range 172.16.100.0/24 --network zzz zzz123
```

### 2.创建云主机的硬件配置方案

```shell
 openstack flavor create --id 0 --vcpus 1 --ram 1024 --disk 10 m1.nano
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

