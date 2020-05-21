**环境说明：**

|   主机名    |   os version    |      ip       | docker version | docker-compose  version | ansible version | awx version |   备注   |
| :---------: | :-------------: | :-----------: | :------------: | :---------------------: | :-------------: | :---------: | :------: |
| ansible-awx | Centos 7.6.1810 | 172.27.34.50  | Docker 19.03.9 |         1.25.5          |      2.9.9      |   10.0.0    | 管理节点 |
|  centos01   | Centos 7.6.1810 | 172.27.34.28  |       ×        |            ×            |        ×        |      ×      | 被管节点 |
|  centos02   | Centos 7.6.1810 | 172.27.34.35  |       ×        |            ×            |        ×        |      ×      | 被管节点 |
|  centos03   | Centos 7.3.1611 | 172.27.34.161 |       ×        |            ×            |        ×        |      ×      | 被管节点 |


&nbsp;

## 一、简介

### ansible简介

Ansible是一个非常简单的IT自动化平台，使程序和系统更易于部署。Ansible本质上是一个进行了封装的Shell，优点在于它是去中心化的工具，可以直接通过ssh管理远程主机，实现无Agent的部署。

### AWX简介

AWX提供了一个基于web的用户界面、REST API和构建在Ansible之上的任务引擎。 图形化的AWX 能够更方便的编排和部署 Ansible Playbook，并提供集中的日志记录、审计和系统跟踪。AWX是商业版Ansible Tower  的开源版本。

**awx项目地址：**https://github.com/ansible/awx/

## 二、ansible安装

### 1.安装EPEL源

```bash
[root@ansible-awx ~]# yum -y install epel-release
```

![image-20200521134255867](https://i.loli.net/2020/05/21/g2AciW8P7a6Jpzm.png)

### 2.安装asnible

```bash
[root@ansible-awx ~]# yum -y install ansible
```

![image-20200521134332028](https://i.loli.net/2020/05/21/KdUrI7zlc3RtWJN.png)

**默认为最新版**

```bash
[root@ansible-awx ~]# ansible --version
```

![image-20200521134229179](https://i.loli.net/2020/05/21/JZ8W1IHO5iwBbAd.png)

## 三、docker安装

### 1.安装依赖包

```bash
[root@ansible-awx ~]# yum install -y yum-utils   device-mapper-persistent-data   lvm2
```

![image-20200521134158380](https://i.loli.net/2020/05/21/N5OK4VWTZ6eb2hA.png)

### 2.设置docker源

```bash
[root@ansible-awx ~]# yum-config-manager     --add-repo     https://download.docker.com/linux/centos/docker-ce.repo
```

![image-20200521134436056](https://i.loli.net/2020/05/21/HiynlpQu9T8Xo57.png)

**docker安装版本查看**

![image-20200521134559258](https://i.loli.net/2020/05/21/TDV3Rcmi9uygQhZ.png)

### 3.安装docker

```bash
[root@ansible-awx ~]# yum install -y docker-ce docker-ce-cli containerd.io
```

![image-20200521135004311](https://i.loli.net/2020/05/21/W9KOFoRvAlHBsgf.png)

未指定版本，默认为最新版

### 4.启动docker

```bash
[root@ansible-awx ~]# systemctl start docker
[root@ansible-awx ~]# systemctl enable docker
```

![image-20200521135139871](https://i.loli.net/2020/05/21/ziCUnFsIR93MepN.png)

### 5. 命令补全

#### 5.1 安装bash-completion

```bash
[root@ansible-awx ~]# yum -y install bash-completion
```

#### 5.2 加载bash-completion

```bash
[root@ansible-awx ~]# source /etc/profile.d/bash_completion.sh
```

![image-20200521135302964](https://i.loli.net/2020/05/21/CJLPgIQswXaKAYu.png)

### 6. 镜像加速

由于Docker Hub的服务器在国外，下载镜像会比较慢，可以配置镜像加速器。主要的加速器有：Docker官方提供的中国registry mirror、阿里云加速器、DaoCloud 加速器，本文以阿里加速器配置为例。

#### 6.1 登陆阿里云容器模块

登陆地址为：https://cr.console.aliyun.com ,未注册的可以先注册阿里云账户

![image-20200521135455943](https://i.loli.net/2020/05/21/MCsL97oBQ6yZbOi.png)

#### 6.2 配置镜像加速器

**配置daemon.json文件**

```bash
[root@centos7 ~]# mkdir -p /etc/docker
[root@centos7 ~]# tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://v16stybc.mirror.aliyuncs.com"]
}
EOF
```

**重启服务**

```bash
[root@centos7 ~]# systemctl daemon-reload
[root@centos7 ~]# systemctl restart docker
```

![image-20200521135601305](https://i.loli.net/2020/05/21/aLAMpIifNbFZqS5.png)
加速器配置完成

## 四、安装Python模块

### 1.安装pip3

```bash
[root@ansible-awx ~]# yum -y install python3-pip
```

![image-20200521141935172](https://i.loli.net/2020/05/21/96RyvHGnmcOosBz.png)

### 2.安装 docker-compose 的Python模块

```bash
[root@ansible-awx ~]# pip3 install docker-compose
```

![image-20200521141822008](https://i.loli.net/2020/05/21/tjuiMFVY2P3aXlb.png)

由于网络原因，安装过程中可能会失败，多试两次即可。

## 五、安装Docker Compose

### 1.下载二进制文件

各版本下载地址：https://github.com/docker/compose/releases

```bash
[root@ansible-awx ~]# curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

![image-20200521140329317](https://i.loli.net/2020/05/21/thzPaMg41sTUrBx.png)

### 2.赋权

```bash
[root@ansible-awx ~]# chmod +x /usr/local/bin/docker-compose
```

### 3.安装验证

```bash
[root@ansible-awx ~]# docker-compose --version
docker-compose version 1.25.5, build 8a1c60f6
```

## 六、awx安装

### 1.下载awx

awx下载地址：https://github.com/ansible/awx/releases ，本文选择版本为10.0.0

```bash
[root@ansible-awx ~]# wget https://github.com/ansible/awx/archive/10.0.0.tar.gz
```

**解压**

```bash
[root@ansible-awx ~]# wget https://github.com/ansible/awx/archive/10.0.0.tar.gz
[root@ansible-awx ~]# ll
总用量 10736
-rw-r--r--   1 root root 10983696 5月  21 14:36 10.0.0.tar.gz
-rw-------.  1 root root     1322 5月  21 11:43 anaconda-ks.cfg
drwxrwxr-x  11 root root     4096 3月  31 00:42 awx-10.0.0
```

### 2.修改配置

```bash
[root@ansible-awx ~]# cd awx-10.0.0/installer/
[root@ansible-awx installer]# ll
总用量 16
-rw-rw-r-- 1 root root  167 3月  31 00:42 build.yml
-rw-rw-r-- 1 root root  437 3月  31 00:42 install.yml
-rw-rw-r-- 1 root root 6131 3月  31 00:42 inventory
drwxrwxr-x 7 root root   99 3月  31 00:42 roles
[root@ansible-awx installer]# sed -i.bak 's/env python/env python3/g' inventory
[root@ansible-awx installer]# ll
总用量 24
-rw-rw-r-- 1 root root  167 3月  31 00:42 build.yml
-rw-rw-r-- 1 root root  437 3月  31 00:42 install.yml
-rw-rw-r-- 1 root root 6132 5月  21 14:41 inventory
-rw-rw-r-- 1 root root 6131 3月  31 00:42 inventory.bak
drwxrwxr-x 7 root root   99 3月  31 00:42 roles
```

修改配置文件inventory，使用python3，其他保持默认配置。

### 3.下载镜像

#### 3.1镜像下载

```bash
[root@ansible-awx ~]# docker pull registry.cn-hangzhou.aliyuncs.com/loong576/awx_web:10.0.0
[root@ansible-awx ~]# docker pull registry.cn-hangzhou.aliyuncs.com/loong576/awx_task:10.0.0
[root@ansible-awx ~]# docker pull registry.cn-hangzhou.aliyuncs.com/loong576/memcached:alpine
[root@ansible-awx ~]# docker pull registry.cn-hangzhou.aliyuncs.com/loong576/postgres:10
[root@ansible-awx ~]# docker pull registry.cn-hangzhou.aliyuncs.com/loong576/redis
```

awx的组件通过容器方式安装，分别下载对应版本镜像,，镜像下载需要些时间，请保持网络环境稳定。

#### 3.2打tag

```bash
[root@ansible-awx ~]# docker tag registry.cn-hangzhou.aliyuncs.com/loong576/awx_task:10.0.0 ansible/awx_task:10.0.0
[root@ansible-awx ~]# docker tag registry.cn-hangzhou.aliyuncs.com/loong576/awx_web:10.0.0 ansible/awx_web:10.0.0 
[root@ansible-awx ~]# docker tag registry.cn-hangzhou.aliyuncs.com/loong576/redis redis
[root@ansible-awx ~]# docker tag registry.cn-hangzhou.aliyuncs.com/loong576/postgres:10 postgres:10 
[root@ansible-awx ~]# docker tag registry.cn-hangzhou.aliyuncs.com/loong576/memcached:alpine memcached:alpine 
```

#### 3.3删除多余镜像

```bash
[root@ansible-awx ~]# docker rmi registry.cn-hangzhou.aliyuncs.com/loong576/awx_web:10.0.0 registry.cn-hangzhou.aliyuncs.com/loong576/redis registry.cn-hangzhou.aliyuncs.com/loong576/postgres:10 registry.cn-hangzhou.aliyuncs.com/loong576/memcached:alpine
```

#### 3.4镜像查看

```bash
[root@ansible-awx ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               latest              987b78fc9e38        2 days ago          104MB
postgres            10                  d92854c53ed1        5 days ago          200MB
memcached           alpine              35acd9837d07        7 days ago          9.28MB
ansible/awx_task    10.0.0              a968a1c4d9fd        7 weeks ago         2GB
ansible/awx_web     10.0.0              2cc33f01ffa7        7 weeks ago         1.96GB
```

### 4.awx安装

```bash
[root@ansible-awx installer]# pwd
/root/awx-10.0.0/installer
[root@ansible-awx installer]# ansible-playbook -i inventory install.yml
```

![image-20200521150341780](https://i.loli.net/2020/05/21/nMGWtOdkChNUYF6.png)

**容器查看**

```bash
[root@ansible-awx ~]# docker ps
CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS                  NAMES
6cbcc91766c2        ansible/awx_task:10.0.0   "/tini -- /bin/sh -c…"   2 hours ago         Up About an hour    8052/tcp               awx_task
d5b698ef603f        ansible/awx_web:10.0.0    "/tini -- /bin/sh -c…"   2 hours ago         Up About an hour    0.0.0.0:80->8052/tcp   awx_web
20f9e95f0c1c        postgres:10               "docker-entrypoint.s…"   2 hours ago         Up About an hour    5432/tcp               awx_postgres
35133c5b8b5f        redis                     "docker-entrypoint.s…"   2 hours ago         Up About an hour    6379/tcp               awx_redis
93d2bd24b6c5        memcached:alpine          "docker-entrypoint.s…"   2 hours ago         Up About an hour    11211/tcp              awx_memcached
```

## 七、登录awx

登陆地址：[http://172.27.34.50](http://172.27.34.50/)

![image-20200521152112098](https://i.loli.net/2020/05/21/3XqEBZWeIxCSkmA.png)

输入用户名admin，默认密码为password

![image-20200521152257510](https://i.loli.net/2020/05/21/YakM8TQgpCKvbIi.png)

## 八、awx实践

**新增清单**

![image-20200521152651127](https://i.loli.net/2020/05/21/vAeY7yxFwdUSIi4.png)

**清单名称为测试区**

![image-20200521152757474](https://i.loli.net/2020/05/21/E78R1rYkWOvdtTZ.png)

**创建主机**

![image-20200521152840557](https://i.loli.net/2020/05/21/7roi4vWstCzPY6U.png)

**分别新增被管主机172.27.34.28/35/161，指定登录用户和密码**

![image-20200521161107699](https://i.loli.net/2020/05/21/FBtrVwuDWCHmk1z.png)

**主机新增完毕**

![image-20200521153417213](https://i.loli.net/2020/05/21/qHyxphYw139sAEi.png)

**执行命令**

![image-20200521153442103](https://i.loli.net/2020/05/21/SP3h9NdQqknxZbG.png)

**查看各被管主机用户**

![image-20200521153628610](https://i.loli.net/2020/05/21/gmHQi719GflAC6x.png)

**参数为：cat /etc/passwd|grep -v 'nologin\|shutdown\|sync\|halt'|awk -F : '{print $1}'**

![image-20200521161228638](https://i.loli.net/2020/05/21/6tIJa8DEWNunbcQ.png)

**命令执行完成**

&nbsp;

&nbsp;



