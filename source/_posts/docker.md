---
title: docker
date: 2022-04-17 14:06:29
tags: Docker learn
category: Linux
---

# Docker

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的镜像中，然后发布到任何流行的Linux或Windows操作系统的机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。

## docker安装

```shell
yum -y install yum-utils device-mapper-persistent-data lvm2
```

```shell
yum -y install gcc
```

```shell
yum-config-manager	--add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

```shell
yum makecache fast
```

```shell
yum -y install docker-ce
```

### 配置加速

```shell
mkdir -p /etc/docker	#创建文件夹
```

```shell
vim /etc/docker/daemon.json

{
  "registry-mirrors": ["https://hxqazmaw.mirror.aliyuncs.com"]
}
```

### 查看docker进程

```shell
ps -ef|grep docker
```

### 启动docker

```shell
systemctl daemon-reload
systemctl start docker
systemctl enable docker
```

## 帮助命令

```shell
docker version				
docker info
docker --help
```

## 镜像命令

```shell
docker images	
-a：列出本地所有的镜像（含中间映像层）
-q：只显示镜像ID
--digests:显示镜像的摘要信息
--no-trunc:显示完整的镜像信息
```

```shell
docker search	镜像名字（查看镜像）
--no-trunc:显示完整的镜像描述
-s:列出收藏数不小于指定值的镜像
--automated:只列出automated build类型的镜像
```

```shell
docker pull	镜像名字（下载镜像）
docker pull	镜像名字[:TAG](版本号，否则为最新版本)
```

### 删除镜像

```shell
删除镜像
删除单个	docker rmi -f 镜像ID
删除多个	docker rmi -f 镜像名1：TAG 镜像名2：TAG
删除全部	docker rmi -f $(docker images -qa)
```

### 镜像commit

```shell
Docker commit -a=”作者” -m=”提交的描述信息” 容器ID 要创建的目标镜像名:[标签名]
```

## 容器命令

### docker run

```shell
Docker run
--name=“容器新名字”：为容器指定一个名称
-d:后台运行容器，并返回容器ID，也即启动守护式容器
-i:以交互模式运行容器，通常与-t同时使用
-t:为容器重新分配一个伪输入终端，通常与-i同时使用
-P:随机端口映射
-p:指定端口映射，有以下四种格式
Ip：hostPort:containerPort
Ip::containerPOrt
HostPort:containerPort
containerPort
```

### docker ps

```shell
Docker ps
-a:列出当前所有正在运行的容器+历史上运行过的
-l:上一个运行的
-n:显示最近n个创建的容器
-q:静默模式，只显示容器编号
--no-trunc:不截断输出
```

### 操作容器

```shell
启动容器：docker start 容器ID或容器名
重启容器：docker restart 容器ID或容器名
停止容器：docker stop 容器ID或容器名
强制停止容器：docker kill 容器ID或容器名
删除已停止容器：docker rm 容器ID	（-f 强制）
```

### 一次性删除多个容器

```shell
Docker rm -f$(docker ps -a  -q)
Docker ps -a -q |xargs docker rm
```

### 退出容器

```shell
Exit
Ctrl+P+Q
```

### 启动守护式容器

```shell
docker run -d  容器名
```

### 查看容器日志

```shell
docker logs -t -f --tail 容器ID

-t是加入时间戳
-f跟随最新的日志打印
--tail 数字 显示最后多少条
```

### 查看容器内部细节

```shell
docker inspect 容器ID
```

### 进入正在运行的容器以命令行交互

```shell
Docker exec -it 容器ID 
docker attach 容器ID

Attach：直接进入容器启动命令终端，不会启动新的进程
Exec：是在容 器中打开新的终端，并且可以启动新的进程
```

### 从容器内拷贝文件到主机上

```shell
docker cp 容器ID：容器内路径 目的主机路径
```

# Dockerfile

Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明

## 参数

| Build         | Both    | Run        |
| ------------- | ------- | ---------- |
| FROM          | WORKDIR | CMD        |
| MAINTAINER    | USER    | ENV        |
| COPY          |         | EXPOSE     |
| ADD           |         | VOLUME     |
| RUN           |         | ENTRYPOINT |
| ONBUILD       |         |            |
| .dockerignore |         |            |

## 命令详解

```shell
FROM 						基础镜像,当前新镜像是基于哪个镜像的
```

```shell
MAINTAINER 					镜像维护者的姓名和邮箱地址
```

```shell
RUN 						容器构建时需要运行的命令
```

```shell
EXPOSE						当前容器对外暴露出的端口
```

```shell
WORKDIR						指定在创建容器后,终端默认登入进来的工作目录,一个落脚点
```

```shell
ENV							用来在构建镜像过程中设置环境变量
```

```shell
ADD							将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包
```

```shell
COPY						类似ADD,拷贝文件和目录到镜像中
```

```shell
VOLUME						容器数据卷,用于数据保存和持久化工作
```

```shell
CMD							指定一个容器启动时要运行的命令,可以有多个CMD,但只有最后一个生效
```

```shell
ENTRYPOINT					指定一个容器启动时要运行的命令,目的和CMD一样,但多个都会生效
```

```shell
ONBUILD						当构建一个被继承的Dockerfile时运行命令,父镜像在被子继承后父镜像的onbuild被触发
```

