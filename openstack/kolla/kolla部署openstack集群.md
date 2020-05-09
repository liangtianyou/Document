# 使用kolla部署openstack

> 使用版本为stein版本

[TOC]

## 1 环境配置

### 1.1 安装系统并进行配置

> 安装Centos7.6以上的系统

#### 1.1.1 配置网络

配置固定IP地址，且与其他主机在同一网段

```bash
vim /etc/sysconfig/network-scripts/ifcfg-ensxx
```

#### 1.1.2 配置hosts与hostname

```bash
hostnamectl set-hostname xxx 

vim /etc/hosts
10.10.166.202 s2
10.10.166.203 s3
10.10.166.204 s4
10.10.166.205 s5
```

#### 1.1.3 配置免密互信

生成秘钥并将秘钥传输给其他节点

```bash
ssh-keygen -t rsa
ssh-copy-id -i ~/.ssh/id_rsa.pub s1
```

### 1.2 下载所需的安装包

下载所需软件

```bash
yum install python-devel ansible docker libffi-devel gcc openssl-devel libselinux-python -y
```

下载pip并更新

```bash
yum upgrade python-setuptools
yum install python-pip
pip install --upgrade pip
```

### 1.3 下载安装kolla和kolla-ansible的代码

创建放置代码的文件夹
```bash
mkdir /root/openstack
cd /root/openstack
```

```bash
cd /root/openstack
git clone https://github.com/openstack/kolla
git clone https://github.com/openstack/kolla-ansible
```
#### 1.3.1 切换代码分支

> 分别进入kolla和kolla-ansible进行分支切换


查看代码分支

```bash
git branch -a
```

切换到指定远程分支

```bash
git checkout -b stable/stein origin/stable/stein
```

#### 1.3.2 源码安装kolla和kolla-ansible

```bash
pip install ./kolla
pip install ./kolla-ansibl
```

### 1.4 配置kolla和kolla-ansible的配置文件
#### 1.4.1 创建etc/kolla目录并复制配置文件

创建kolla配置文件目录
```bash
mkdir -p /etc/kolla
```

将项目中的配置文件复制到```/etc/kolla```
```bash
cp -r kolla-ansible/etc/kolla/* /etc/kolla
```
#### 1.4.2 获取角色配置文件并进行配置

 - 配置文件共两个为```all-in-one```和```multinode```
 - ```all-in-one```为配置所有服务只在单台主机运行的环境，可以直接使用默认配置进行部署
 - ```multinode```为配置多节点的角色清单，需要安装前期规划进行配置


将角色配置文件复制到当前目录
```bash
cp kolla-ansible/ansible/inventory/* .
```

对角色文件进行配置,只需要将对应各项的hostname进行修改即可
```bash
vim multinode

[control]
s2
s3
[network]
s2
[compute]
s4
s5
[monitoring]
s2
[storage]
s2
s3
s4
s5
[deployment]
localhost       ansible_connection=local
```

可使用kolla-ansible对各节点的连通情况进行验证
```bash
ansible -i multinode all -m ping
```
#### 1.4.3 生成随机密码以及设置登录密码
生成随机密码
```bash
kolla-genpwd
```
修改登录密码
```bash
vim /etc/kolla/passwords.yml
keystone_admin_password: 123456
```
#### 1.4.4 修改部署配置文件
> 执行```kolla-ansible```相关命令时，默认调用的下面的配置文件以及上述的密码文件

配置文件位置
```bash
vim /etc/kolla/globals.yml
```

**部分关键参数修改**

指定安装的版本
```bash
# 指定使用centos为基础系统的镜像
kolla_base_distro: "centos"
kolla_install_type: "binary"
# 指定安装openstack版本
openstack_release: "stein"
```

网络配置参数
```bash
# 此IP地址为部署完成后，页面访问的IP地址
kolla_internal_vip_address: "10.10.166.210"
```

docker参数配置
```bash
# 如果不使用本地镜像库进行部署，将参数注释掉即可
docker_registry: "10.10.166.202:4000"
docker_namespace: "kolla"
```

### 1.5 配置本地镜像仓库安装
> 如果网络条件比较好，可以不使用本地镜像仓库部署

#### 1.5.1 获取kolla部署所需的openstack镜像

不使用本地镜像仓库，注释```globals.yml```中的docker参数，执行以下命令将镜像下载到各个节点
```bash
cd /root/openstack
kolla-ansible -i ./multinode pull
```

如果要使用本地镜像库部署，可按照以下清单获取docker镜像（所拉取镜像均为stein版本）并上传
```bash
kolla/centos-binary-chrony
kolla/centos-binary-collectd
kolla/centos-binary-cron
kolla/centos-binary-fluentd
kolla/centos-binary-glance-api
kolla/centos-binary-haproxy
kolla/centos-binary-heat-api
kolla/centos-binary-heat-api-cfn
kolla/centos-binary-heat-engine
kolla/centos-binary-horizon
kolla/centos-binary-influxdb
kolla/centos-binary-ironic-inspector
kolla/centos-binary-iscsid
kolla/centos-binary-keepalived
kolla/centos-binary-keystone
kolla/centos-binary-keystone-fernet
kolla/centos-binary-keystone-ssh
kolla/centos-binary-kolla-toolbox
kolla/centos-binary-mariadb
kolla/centos-binary-memcached
kolla/centos-binary-mongodb
kolla/centos-binary-neutron-dhcp-agent
kolla/centos-binary-neutron-l3-agent
kolla/centos-binary-neutron-metadata-agent
kolla/centos-binary-neutron-openvswitch-agent
kolla/centos-binary-neutron-server
kolla/centos-binary-nova-api
kolla/centos-binary-nova-compute
kolla/centos-binary-nova-conductor
kolla/centos-binary-nova-consoleauth
kolla/centos-binary-nova-libvirt
kolla/centos-binary-nova-novncproxy
kolla/centos-binary-nova-scheduler
kolla/centos-binary-nova-ssh
kolla/centos-binary-openvswitch-db-server
kolla/centos-binary-openvswitch-vswitchd
kolla/centos-binary-placement-api
kolla/centos-binary-prometheus-cadvisor
kolla/centos-binary-prometheus-memcached-exporter
kolla/centos-binary-prometheus-mysqld-exporter
kolla/centos-binary-rabbitmq
kolla/centos-binary-swift-container
kolla/centos-binary-swift-proxy-server
kolla/centos-binary-swift-rsyncd
kolla/centos-binary-telegraf
kolla/centos-binary-tgtd
```

#### 1.5.2 创建本地镜像仓库
> 在部署节点上创建本地镜像仓库

拉取registry镜像
```bash
docker pull registry
```

运行registry容器
```bash
docker run -d -v /edc/images/registry:/var/lib/registry -p 4000:5000 --restart=always --name registry registry
```

#### 1.5.3 上传镜像到本地仓库

修改配置文件指定使用本地仓库
```bash
vim /etc/docker/daemon.json
{"insecure-registries" : [ "10.10.166.202:4000" ]}
```

重新加载配置文件
```bash
systemctl daemon-reload && systemctl restart docker && systemctl status docker
```

修改本地镜像标签指向本地仓库
```bash
for i in `docker images|grep -v registry|grep -v R|grep kolla/centos-binary|awk '{print $1}'`;do docker image tag $i:stein 10.10.166.202:4000/${i#*/}:stein;done
```

将修改好标签的镜像上传到本地仓库
```bash
for i in `docker images|grep 10.10.166.202|awk '{print $1}'`;do docker push $i:stein;done
```

查看已上传的镜像
```bash
curl http://s2:4000/v2/_catalog
```

将本地仓库中的镜像分发到各节点
```bash
cd /root/openstack
kolla-ansible -i ./multinode pull
```

### 1.6 进行环境检查

```bash
cd /root/openstack
kolla-ansible -i ./multinode prechecks
```

## 2 部署openstack

### 2.1 部署引导服务器

```bash
cd /root/openstack
kolla-ansible -i ./multinode bootstrap-servers
```

### 2.2 自动化部署openstack

```bash
kolla-ansible -i ./multinode deploy
```

### 2.3 清理opensack环境
> 如果未出现配置文件错误，可以不清理环境重复执行自动化部署命令

因配置文件出错，需要清理环境重新部署
```bash
kolla-ansible -i ./multinode cleanup 
```

## 3 错误汇总以及解决方法

### 3.1 pip安装代码出错
pip安装kolla和kolla-ansible时，出现无法卸载包的问题


在以下路径中找到对应的包，手动删除后，重新执行pip安装
```bash
/usr/lib/python2.7/site-packages/
usr/lib64/python2.7/site-packages/
```
### 3.2 端口占用错误
在执行```kolla-ansible -i ./multinode prechecks```命令时，出现执行失败


根据错误信息，查看所使用的端口，将占用端口的进程清除
查看端口命令
```bash
netstat -ntulp | grep xxx
```

### 3.3 未配置docker共享参数
在执行```kolla-ansible -i ./multinode prechecks```命令时，未配置docker共享参数

创建对应的文件并进行配置
```bash
mkdir -pv /etc/systemd/system/docker.service.d

vim /etc/systemd/system/docker.service.d/kolla.conf
[Service]
MountFlags=shared

systemctl daemon-reload && systemctl restart docker && systemctl status docker