---
title: 如何创建一个debian6的镜像源

date: 2020-05-14 19:49

tags: 
    - debian 6
    - apt 仓库
    - mirror
    - 镜像源
--

[TOC]

## 1 需求

公司有一套 ```PXE``` 网络安装环境，基于 ```debian 6``` 的 ```ODSP V4.1``` 都是通过网络安装（PXE+TFTP+Apache+preseed.cfg），没有制作 ```ISO``` 光盘，现场设备出问题需更换系统盘时，即时售后在现场，也需要通过邮寄系统盘的方式来，因为 ```debian 6``` 仓库的体积实在是太大（超120G），要想搭建迷你版本的 ```PXE``` 虚拟机来实现现场批量替换，就必须定制化 ```debian 6``` 的源。

## 2 解决方案
结合已有环境，同时参考了部分文档，总结如下：
1. [debian 制作本地源](http://blog.chinaunix.net/uid-20119707-id-3357690.html)
2. [如何创建一个简单 APT 仓库](https://www.cnblogs.com/bamanzi/p/create-simple-apt-repo.html)
3. [iPXE 引导 ubuntu 并使用 preseed 自动安装系统](https://lvii.github.io/system/2019-08-07-ipxe-auto-install-ubuntu-server-with-preseed/)
4. [Debian GNU/Linux 安装手册](https://d-i.debian.org/manual/)

## 3 搭建步骤
### 3.1 获取需要的包
安装包分为两类。
**第一类包**：
是 ```debian installer``` 需要的包，也就是安装过程中需要的包，此类包基本都是 ```.udeb```结尾的包，从 ```debian 6``` 仓库的 ```dists/squeeze/main/debian-installer/binary-amd64/``` 目录里的 ```Packages.gz``` 文件可以获取到所有 ```Section``` 为 ```debian-installer``` 的包及路径，通过 ```sublime``` 工具将所有路径匹配出，写一个脚本将其从仓库里拷贝出来```。

**备注**：sublime 通过正则表达式匹配所有路径
```Packages.gz``` 文件里的内容基本如下：

```bash
Package: anna
Priority: standard
Section: debian-installer
Installed-Size: 212
Maintainer: Debian Install System Team <debian-boot@lists.debian.org>
Architecture: amd64
Version: 1.39
Depends: libc6-udeb (>= 2.11), libdebconfclient0-udeb, libdebian-installer4-udeb (>= 0.76), cdebconf-udeb
Filename: pool/main/a/anna/anna_1.39_amd64.udeb
Size: 71470
MD5sum: fb45ad870cfb95a10e3350eac32f7943
SHA1: 88a200620807a389b8e810e5a74ab88a1e5c1c30
SHA256: d2dfd84e77346979788119e69a4694c582e614a6c5015fdb60211d8041614fce
Description: anna's not nearly apt, but for the Debian installer, it will do

Package: apt-cdrom-setup
Priority: extra
Section: debian-installer
Installed-Size: 328
Maintainer: Debian Install System Team <debian-boot@lists.debian.org>
Architecture: all
Source: apt-setup
Version: 1:0.53+squeeze2
Provides: apt-mirror-setup
Depends: cdrom-detect, cdebconf-udeb
Filename: pool/main/a/apt-setup/apt-cdrom-setup_0.53+squeeze2_all.udeb
Size: 88708
MD5sum: 85f585c11ea440d09309cc20ebeb4043
SHA1: 5c8d7d4f3d8f0b2d1d5d6a008c62ae6309d442c5
SHA256: 10fed94d42a0bca03127fba60993c83d49c0a47fe0ae1b83f2ae027773efc8f5
Description: set up a CD in sources.list
...
```

按 ```CTRL+F```，选中 ```Regular Expression```，输入 ```Filename:\s*(.*.udeb)$```，便可将所有 ```.udeb``` 文件路径选中，拷贝到一个新的文件中，按 ```CTRL+A``` 全选，按 ```CTRL+L``` 多行编辑，将 ```Filename:``` 删除，剩下的就是所有文件的路径。然后前面输入 ```cp ```，按 ```end``` 到所有行尾，输入 ``` /root/pools/debian-installer/```，这里自行定义，我只是举例说明如何将所有文件获取到。 

```dist/squeeze``` 目录结构如下：

```bash
[root@localhost squeeze]# tree 
.
├── Contents-amd64.gz
├── Contents-i386.gz
├── contrib
│   ├── binary-amd64
│   │   ├── Packages
│   │   ├── Packages.gz
│   │   └── Release
│   └── binary-i386
│       ├── Packages
│       ├── Packages.gz
│       └── Release
├── main
│   ├── binary-amd64
│   │   ├── Packages
│   │   ├── Packages.gz
│   │   └── Release
│   ├── binary-i386
│   │   ├── Packages
│   │   ├── Packages.gz
│   │   └── Release
│   └── debian-installer
│       ├── binary-amd64
│       │   ├── Packages
│       │   ├── Packages.gz
│       │   └── Release
│       └── binary-i386
│           ├── Packages
│           ├── Packages.gz
│           └── Release
├── non-free
│   ├── binary-amd64
│   │   ├── Packages
│   │   ├── Packages.gz
│   │   └── Release
│   └── binary-i386
│       ├── Packages
│       ├── Packages.gz
│       └── Release
├── Release
└── Release.gpg
```

**第二类包**：
需要安装的软件包，网络安装的 ```preseed.cfg``` 里有安装软件包的步骤，但无法获取其依赖关系（无知无畏），那就笨点，先安装一个系统，再通过 ```dpkg -l``` 命令把安装的包导出即可。

方法一：
笨办法，获取到包名后在仓库里一个个搜索获取。

方法二：
用 ```sudo apt-get install --reinstall -d <package-name>``` 来把所有的包下载到 ```/var/cache/apt/archives/``` 目录，然后将其拷贝到指定目录。

### 生成仓库
#### 目录结构
```bash
├── dists
│   ├── squeeze
│   │   ├── InRelease
│   │   ├── main
│   │   │   ├── binary-amd64
│   │   │   │   ├── Packages
│   │   │   │   ├── Packages.gz
│   │   │   │   └── Release
│   │   │   ├── debian-installer
│   │   │   │   └── binary-amd64
│   │   │   │       ├── Packages
│   │   │   │       ├── Packages.gz
│   │   │   │       └── Release
│   │   │   └── Release
│   │   ├── Release
│   │   └── Release.gpg
│   └── stable -> squeeze
├── pools
│   ├── debian-installer
│   │   ├── acpi-modules-2.6.32-5-amd64-di_1.76+squeeze6_amd64.udeb
│   │   ├── ai-console-setup-udeb_1.9_all.udeb
│   │   ├── ......
│   │   └── zlib-modules-2.6.32-5-amd64-di_1.76+squeeze6_amd64.udeb
│   └── odsp
│       ├── acl_2.2.49-4_amd64.deb
│       ├── acpi_1.5-2_amd64.deb
│       ├── .....
│       └── zlib1g_1.2.3.4.dfsg-3_amd64.deb
└── update.sh
```

```update.sh``` 脚本用来动态生成 ```Packages.gz```、```Release``` 等文件，内容如下：
```bash
dpkg-scanpackages pools/odsp > dists/squeeze/main/binary-amd64/Packages
dpkg-scanpackages pools/odsp | gzip > dists/squeeze/main/binary-amd64/Packages.gz
cat > dists/squeeze/main/binary-amd64/Release << EOF
Archive: stable
Origin: Debian
Label: Debian
Version: 6.0.4
Component: main
Architecture: amd64
EOF

dpkg-scanpackages -t udeb pools/debian-installer > dists/squeeze/main/debian-installer/binary-amd64/Packages
dpkg-scanpackages -t udeb pools/debian-installer | gzip > dists/squeeze/main/debian-installer/binary-amd64/Packages.gz
cat > dists/squeeze/main/debian-installer/binary-amd64/Release << EOF
Archive: stable
Origin: Debian
Label: Debian
Version: 6.0.4
Component: main
Architecture: amd64
EOF

rm -rf dists/squeeze/Release
cat > dists/squeeze/Release << EOF
Origin: Debian
Label: Debian
Suite: stable
Version: 6.0.4
Codename: squeeze
Architectures: amd64
Components: main
Description: Debian 6.0.4 Released 28 January 2012
EOF
apt-ftparchive release dists/squeeze/ >> dists/squeeze/Release
gpg --armor --detach-sign --sign -o dists/squeeze/Release.gpg dists/squeeze/Release
```

## 补充：
### apt一键下载所有依赖的包
无外网的局域网安装软件一个烦人的事件就是明明安装包下好了，但有时候就是安装不上，因为缺少相应依赖的包。

那么如何将一个软件依赖的包、库之类的下载下来呢。这里就用到apt的相关功能。
#### 方法
1. 找包
    找到依赖的包用apt-cache depends packname来获取。
2. 下载
    用 ```apt-get install dependpackname --reinstall -d``` 来下载所依赖的包。 -d是表示只下载。
3. 批量安装
    用shell命令组合来一键下载所有所依赖的包。

```bash
#有些包名中有<>符号，用tr将其删除
sudo apt-get install --reinstall -d `apt-cache depends packname | grep depends | cut -d: f2 |tr -d "<>"`
```

注意上面命令中反引号。 即 ` 。

示例：找到fcitx所依赖的包。


```apt-cache depends``` 会列出3类包。如果只想安装 ```Depends``` 的，用 ```grep``` 过滤下。如果想提取出包的名字，用 ```cut``` 做一个分割。剩下的就是调用下载相应包的指令了。

#### 进阶
下载所依赖的包所依赖的包

比如A依赖B，B又依赖C，那如何递归呢。此时就需要一个脚本文件，用函数来实现了。

递归3次下载所依赖包的脚本如下：
```bash
#!/bin/bash

logfile=/home/perrin/Desktop/log
ret=""
function getDepends()
{
   echo "fileName is" $1>>$logfile
   # use tr to del < >
   ret=`apt-cache depends $1|grep Depends |cut -d: -f2 |tr -d "<>"`
   echo $ret|tee  -a $logfile
}
# 需要获取其所依赖包的包
libs="gnome-shell"                  # 或者用$1，从命令行输入库名字

# download libs dependen. deep in 3
i=0
while [ $i -lt 3 ] ;
do
    let i++
    echo $i
    # download libs
    newlist=" "
    for j in $libs
    do
        added="$(getDepends $j)"
        newlist="$newlist $added"
        apt install $added --reinstall -d -y
    done

    libs=$newlist
done
```
