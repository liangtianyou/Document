---
title: Gerrit 代码服务器安装
date: 2019-10.12 11:00
comments: true
tags: 
    - Gerrit
    - drbd
    - CentOS 7
---

# 代码服务器安装
使用了两台设备来备份代码，两台机器的配置一样。创建RAID，挂载到/data目录，gerrit和project目录都部署到/data上。

## 修改源
```bash
rm -rf /etc/yum.repos.d/*
cat > /etc/yum.repos.d/c7-digiocean.repo << EOF
[c7-digiocean]
name=c7-digiocean
baseurl=http://10.10.1.8/centos-7.0/
gpgcheck=0
enabled=1
EOF
```
## 安装依赖包
yum -y install git mariadb-server postfix dovecot httpd

## 安装drbd
yum install DRBD/drbd-km-3.10.0_123.el7.x86_64-8.4.10-1.x86_64.rpm DRBD/drbd-utils-9.0.0-1.el7.centos.x86_64.rpm

## 配置drbd
* 配置文件：/etc/drbd.d/data.res，内容如下：
```bash
resource data {
	on EServer1 {
		device /dev/drbd0;
		disk /dev/centos_bogon/data;
		address 10.10.1.21:7788;
		meta-disk internal;
	}
	on EServer3 {
		device /dev/drbd0;
		disk /dev/centos_bogon/data;
		address 10.10.1.23:7788;
		meta-disk internal;
	}
}
```
* 创建drbd
```bash
drbdadm create-md data
drbdadm up data
drbdadm -- --overwrite-data-of-peer primary data
```
* 开机启动服务
```bash
systemctl enable drbd
systemctl start drbd
```
Tip: 创建drbd后无法连接上，检查是否防火墙开启。

## 挂载drbd
* 格式化
```bash
mkfs.xfs /dev/drbd0
``
* 写入fstab
vim /etc/fstab，写入内容如下：
​```bash
/dev/drbd0 /data                   xfs     defaults        1 2
```
* 挂载
```bash
mount /data
```
## 设置开机启动
* 开机脚本：/etc/init.d/startgerrit，内容如下：

```bash
#!/bin/bash
#
# start drbd and gerrit after boot
# 
# chkconfig: 2345 20 20
# description: start drbd and gerrit after boot

. /etc/rc.d/init.d/functions

start() {
    #up drbd
    modprobe drbd
    drbdadm up data
    drbdadm primary data
    #mount /data
    mount /dev/drbd0 /data
    #start gerrit
    /data/gerrit/bin/gerrit.sh start
    #start httpd
    systemctl start httpd
    return 0
}

stop() {
	systemctl stop httpd
    /data/gerrit/bin/gerrit.sh stop
    flush
    umount /data
    drbdadm secondary data
    drbdadm down data
    rmmod drbd
    return 0
}

restart() {
    stop
    start
}

case "$1" in
    start)
        $1
        ;;
    stop)
        $1
        ;;
    restart)
        $1
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac
```
* 设置开机启动
```bash
systemctl enable startgerrit
```

## 初始化数据库
```bash
systemctl enable mariadb
systemctl start mariadb
mysqladmin -uroot password 'xxx'
mysql -uroot -pxxx -e "CREATE DATABASE reviewdb CHARACTER SET 'utf8' COLLATE 'utf8_general_ci'"
```

## 安装gerrit
```bash
java -jar gerrit-2.14.6.war init -d /data/gerrit
```

### 安装后配置文件
* 配置文件：/data/gerrit/etc/gerrit.config，内容如下：
```bash
[gerrit]
	basePath = /data/project
	serverId = 3cf32b37-2598-4441-a34c-4c7dc8d4ef56
	canonicalWebUrl = http://gerrit1.perabytes.com/
[database]
	type = mysql
	hostname = localhost
	database = reviewdb
	username = root
[index]
	type = LUCENE
[auth]
	type = HTTP
[receive]
	enableSignedPush = false
[sendemail]
	smtpServer = EServer1.perabytes.com
	smtpUser = admin
[container]
	user = root
	javaHome = /usr/java/jdk1.8.0_152/jre
[sshd]
	listenAddress = *:29418
[httpd]
	listenUrl = proxy-http://*:8081/
[cache]
	directory = cache
```
* 密码配置文件：/data/gerrit/etc/secure.config，内容如下：
```bash
[database]
	password = ***
[auth]
	registerEmailPrivateKey = ***
[sendemail]
	smtpPass = ***
```
密码内容我隐藏了，根据实际情况填写。

### 创建管理员
```bash
htpasswd -c /data/gerrit/etc/passwd admin
```

### 启动gerrit
```bash
/data/gerrit/bin/gerrit.sh start
```

### 停止gerrit
```bash
/data/gerrit/bin/gerrit.sh stop
```

### 配置httpd
* 配置文件：/etc/httpd/conf.d/gerrit.conf，内容如下：
```bash
<VirtualHost *:80>
    ServerName gerrit1.perabytes.com

    ProxyRequests Off  
    ProxyVia Off  
    ProxyPreserveHost On  
  
    <Proxy *>  
          Order deny,allow  
          Allow from all  
    </Proxy>  
  
    <Location /login/>  
        AuthType Basic  
        AuthName "Gerrit Code Review"  
        AuthBasicProvider file  
        AuthUserFile /data/gerrit/etc/passwd  
        Require valid-user  
    </Location>
  
    AllowEncodedSlashes On  
    ProxyPass / http://10.10.1.21:8081/    
</VirtualHost>
```
* 设置开机启动
```bash
systemctl enable httpd
systemctl start httpd
```

