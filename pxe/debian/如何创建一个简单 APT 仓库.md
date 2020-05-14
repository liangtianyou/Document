# 如何创建一个简单 APT 仓库

[TOC]

## 1 无废话版本

### 1.1 需求
有一堆 ```.deb``` 包，想把它们做成一个 ```APT``` 仓库，这样就可以用 ```apt install pkgname``` 进行安装了，这样一方面自己可以规避 ```dpkg -i xxx.deb``` 时候的依赖问题，另一方面也方便了其他人。

### 1.2 解决方法
```bash
# 服务端
mkdir -p /opt/raspi-apt-repos/raspbian8
cp ~/Downloads/raspbian8/*.deb /opt/raspi-apt-repos/raspbian8
cd /opt/raspi-apt-repos/raspbian8
# scan *.deb and create Packages file
dpkg-scanpackages -m . > Packages
# create  Release  file 
apt-ftparchive release . &gt; Release

# TODO: create a GPG key with ‘gpg --gen-key’ if you have no one
gpg --list-keys  || gpg --gen-key
# create Release.gpg file
gpg --armor --detach-sign --sign -o Release.gpg Release
# create InRelease file
gpg --clearsign -o InRelease Release

# export your public key
cd /opt/raspi-apt-repos/
gpg --export --armor <uid> -o my-repo.gpg-key.asc

# start web server (you'd better use nginx instead)
python -m SimpleHTTPServer 4000

# ===== 8&lt; ==========================================
# 客户端
wget http://210.32.142.88:4000/my-repo.gpg-key.asc && sudo apt-key add my-repo.gpg-key.asc
echo “deb http://foo.example.com/repos/raspbian8 ./” | sudo tee /etc/apt/sources.list.d/my-repo.list
```
## 2 什么叫＂简单 APT 仓库"
这里说的是如何创建一个简单的 ```APT``` 仓库（根据 [DebianRepository/Setup - Debian Wiki](https://wiki.debian.org/DebianRepository/Setup) ，这种简单的仓库称为 ```trivial archive```, 而复杂的那种称为 ```official archive```），之所以说＂简单＂，主要是指仓库不采用 ```pool``` 结构，里面只有一个```suite```（jessie, jessie-backport, trusty, trusty-updates 这些东西叫做 suite)，一个 ```component```（main, nonfree, universe 这些东西叫做 component），也就是说 ```Packages``` 文件只有一个（也不提供 ```Contents-amd64.gz``` 这些可有可无的文件）。

这个仓库的目录结构如下：
```bash
.
├── vim_8.0.1420-0york0~14.04_amd64.deb
├── vim_8.1.1575-0york0~14.04_amd64.deb
├── vim_8.1.1575-0york0~16.04_amd64.deb
├── vim_8.1.1575.orig.tar.gz
├── vim-common_8.0.1420-0york0~14.04_all.deb
├── vim-common_8.1.1575-0york0~16.04_all.deb
├── Packages
└── Release
```
使用时需要在 ```/etc/apt/sources.list``` 里面添加的配置是：
```bash
deb http://foo.example.com/repos/raspbian8 ./
```
在这个配置里, ```suite``` 的值是 ```./```, 当 ```suite``` 是路径的情况下 ```component``` 必须为空．对于 ```sources.list``` 的格式可以参考 [sources.list (5)](https://manpages.debian.org/sources.list)。 
### 2.1 参考: "不简单"的 APT仓库
```Debian/Ubuntu``` 的官方仓库（以及 ```Launchpad``` 上面的 ```ppa```）跟上面的结构不一样，典型特征是顶层有个 ```dists``` 目录和 ```pool``` 目录．目录结构如下:
```bash
├── dists/
│   ├── buster/
│   ├── jessie/
│   │   ├── InRelease
│   │   ├── main/
│   │   │   ├── binary-aarmhf/
│   │   │   │   ├── Packages
│   │   │   │   └── Release
│   │   │   ├── binary-all/
│   │   │   │   ├── Packages
│   │   │   │   └── Release
│   │   │   └── binary-amd64/
│   │   │   └── Contents-amd64.gz
│   │   │   └── Contents-armhf.gz
│   │   ├── Release
│   │   └── Release.gpg
│   └── stretch/
└── pool/
    └── main/
        ├── v/
        │   └── vim/
        │       ├── vim_8.0.1420-0york0~14.04_amd64.deb
        │       ├── vim_8.1.1575-0york0~14.04_amd64.deb
        │       ├── vim_8.1.1575-0york0~16.04_amd64.deb
        │       ├── vim_8.1.1575.orig.tar.gz
        │       ├── vim-common_8.0.1420-0york0~14.04_all.deb
        │       └── vim-common_8.1.1575-0york0~16.04_all.deb
        └── z/
            └── zim/
```
对应的 ```/etc/apt/sources.list``` 配置大致如下：
```bash
deb http://foo.example.com/repos/raspbian8 jessie main
```
在这个配置里，```suite=jessie, component=main```，一行可以配置多个 ```component```，但不能配置多个 ```suite```。

可以看到，```dists``` 下面按 ```suite``` 分了多个目录(stretch, jessie, buster)，```suite``` 下面又按 ```CPU``` 架构有不同的 ```binary-${arch}``` 目录和 ```Contents-${arch}.gz``` 文件；而真正的deb包都放在 ```pool``` 目录下，对于 ```vim``` 这样一个软件来说多个 ```suite/cpu``` 的deb包是放在一起的。

这样的好处是:
1. 将所有类型CPU的包列表(Packages或者Packages.gz文件)放在一个文件里面，这样每个机器要获取的包列表就比较小（考虑到 ```Debian/Ubuntu``` 所收录的软件数，这个节省是比较可观的；而 ```Contents-${arch}.gz``` 里面存放的是所有包里面的文件列表（ ```apt-file``` 程序会使用到这个信息），这个文件的体积就更大了。
2. 不同套件/不同```CPU```可共用的 ```deb``` 包（主要是那些 ```_all.deb```）和源代码包，也只在 ```pool/v/vim``` 这样的目录下存放一份，不同存放多份。
3. 源代码包(```.dsc, orig.tar.xz```)有路径存放，这样 ```dget / apt source``` 可以取到源代码包。

不过缺点就是：
这样的仓库很复杂，不靠工具是无法确定构建出一个 ```deb```　之后要往哪里放，要更新哪些索引文件（```Packages```, ```Contents-${arch```}, ```Release```, ```InRelease```...）

## 3 创建简单 APT 仓库的方法

根据 [DebianRepository/Setup - Debian Wiki](https://wiki.debian.org/DebianRepository/Setup?action=show&redirect=HowToSetupADebianRepository) ，有很多种工具可以用来创建 ```APT``` 仓库，但创建简单仓库(```trivial archive```)的方法在该页面就只看见一个，于是我就用了:
### 3.1 创建
1. 将所有 deb 放到同一个目录（假设这个目录为 ```/opt/raspi-apt-repos/raspbian8```）
2. ```cd /opt/raspi-apt-repos/raspbian8 && dpkg-scanpackages -m . > Packages```

这就完成了，是不是很简单？(其实事儿没完呢) 可以看到唯一的变化就是多了一个 ```Packages``` 文件，里面是各个 ```deb``` 包的相对路径＼描述＼大小（也就是都是客户端用 ```apt show xxx``` 可以查到的内容）

### 3.2 使用
1. 如果仅仅是本机使用，那么可以在 ```/etc/apt/sources.list``` 里面添加一行 ```deb file:/opt/raspi-apt-repos/raspbian8 ./``` 然后就可以 ```apt update && apt install vim``` 了．
2. 如果要提供给其它机器使用，那么需要安装配置 ```nginx``` 之类的 ```web server``` (临时试验可以用 ```python -m SimpleHttpServer``` 启动一个 ```web server```)．假设在 ```web server```里面将 ```/opt/raspi-apt-repos/``` 映射到了 ```http://foo.example.com/repos```, 那么对方需要在 ```/etc/apt/sources.list``` 里面添加的配置就是 ```deb http://foo.example.com/repos/raspbian8 ./```　

### 3.3 补充说明
1. ```dpkg-scanpackages``` 这个工具在 ```dpkg-dev``` 这个包里面，需要先安装这个包才能使用．这里有个好处是 ```Centos``` 的```EPEL``` 库里面也有这个包，所以 ```RHEL / CentOS``` 上也就可以很方便地装上这个工具再按上述方法创建一个 ```APT``` 仓库（其实我工作中同时要给 ```Ubuntu 14.04/16.04/18.04```, ```CentOS 6/7```, ```SLES 12.4/15``` 同时提供包仓库，服务器是一台 ```CentOS 7```)
2. 上面命令行中给 ```dpkg-scanpackages``` 加了 ```-m``` 选项，这个选项的作用是如果同名的包有多个版本，都把它们采集到 ```Packages``` 文件中去，这样用户可以自己用 ```apt show vim``` 看到所有的版本，然后用 ```apt install foo=1.12``` 这样的方式来安装指定的版本．（如果你的目录内有同名的包，而又没有使用 ```-m``` 选项的话，后扫描到的包会被忽略，不进入 ```Packages``` 文件．另外需要注意的是，```abc_1.1-1_i386.deb``` 和 ```abc_2.0-1_amd.deb``` 这种针对不同 ```CPU``` 也会被认为是同名的包，其中一个会被忽略掉！）

## 3 Ubuntu 16.04 / Debian 8 以上版本拒绝上述简单仓库的解决办法
### 3.1 Release 文件
如果用户端是 ```Ubuntu 16.04 / Debian 8 (jessie)``` 或者更高版本的话， 这个仓库会被 ```apt``` 拒绝接受，并报告如下的错误信息:
```bash
$ sudo apt update
Get:1 file:/opt/raspi-apt-repo/ubuntu18 ./ InRelease
Ign:1 file:/opt/raspi-apt-repo/ubuntu18 ./ InRelease
Get:2 file:/opt/raspi-apt-repo/ubuntu18 ./ Release
Err:2 file:/opt/raspi-apt-repo/ubuntu18 ./ Release
  File not found - /opt/raspi-apt-repo/ubuntu18/./Release (2: No such file or directory)
Hit:3 http://ports.ubuntu.com bionic InRelease
Hit:4 http://ports.ubuntu.com bionic-updates InRelease
Hit:5 http://ports.ubuntu.com bionic-security InRelease
Hit:6 http://ppa.launchpad.net/ubuntu-pi-flavour-makers/ppa/ubuntu bionic InRelease
Hit:7 http://ports.ubuntu.com bionic-backports InRelease
Reading package lists... Done
E: The repository 'file:/opt/raspi-apt-repo/ubuntu18 ./ Release' does not have a Release file.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.
```
可以看到，```apt``` 由于没有从仓库读取到 ```Release``` 文件，于是不再读取 ```Packages``` 文件，这个仓库被拒绝了 (虽然前面也试图读取 ```InRelease``` 文件并且失败了，但这个错误是可以忽略的（行首是 ```Ign```，而读取 ```Release``` 那行是 ```Err```）

出现这个问题的原因在于, ```apt``` 对了加强安全性， 需要仓库生成一个 ```Release``` 文件，这个文件里面包含了 ```Packages``` 等文件的大小和校验和(包含 **MD5/SHA1/SHA256/SHA512** 多种值)．如果这个文件里面所描述的 ```Packages``` 大小与校验和与实际读取到的文件不一致，```apt``` 也会拒绝这个仓库．
```
Date: Sat, 29 Jun 2019 07:18:59 +0000
MD5Sum:
 7a487761b83fc9326ba4a3186df3b950             5384 Packages
 256a066585672866ffd831926ab42b77               38 Release
SHA1:
 92c867ff5789ae6efdcc500d59d220b7e30bc952             5384 Packages
 d1c369af68084f30a6d3cf8d98066d4a723a459c               38 Release
SHA256:
 a9ae09e62999cd2b7b7f7fab48ae04e28ac3d2a6a6613df5e3c93bffe11d9166             5384 Packages
 2124019cd02360d2f5466d40bb097920f2415e997a9fb20c78380bb04d5c33ff               38 Release
SHA512:
 68da75f205561ce06b327502611c0f01d6401e048e17dcf2960d96c04397fbdf20252d2ccb0e997570834572ede7a6376162caec3411560bc5523a06413f634d             5384 Packages
 b5b3f6a48013239e70c34f623f5e69594c6b11f27f8da8325067f97f5801c73208c9adb66b0d1cce0186d1102c1013684c615077d8dcebb590027390969e0b54               38 Release
```
生成这个 Release 文件的方法是: ```apt-ftparchive release . > Release```

### 3.2 Release.gpg 和 InRelease 文件
生成了 ```Release``` 文件之后，用户端运行 ```apt update```，还是会碰到问题:
```bash
$ sudo apt update
Get:1 file:/opt/raspi-apt-repo/ubuntu18 ./ InRelease
Ign:1 file:/opt/raspi-apt-repo/ubuntu18 ./ InRelease
Get:2 file:/opt/raspi-apt-repo/ubuntu18 ./ Release [816 B]
Get:2 file:/opt/raspi-apt-repo/ubuntu18 ./ Release [816 B]
Get:3 file:/opt/raspi-apt-repo/ubuntu18 ./ Release.gpg                                        
Ign:3 file:/opt/raspi-apt-repo/ubuntu18 ./ Release.gpg                                   
Hit:4 http://ports.ubuntu.com bionic InRelease                                                               
Hit:5 http://ports.ubuntu.com bionic-updates InRelease                                             
Hit:6 http://ports.ubuntu.com bionic-security InRelease                                            
Hit:7 http://ports.ubuntu.com bionic-backports InRelease                 
Hit:8 http://ppa.launchpad.net/ubuntu-pi-flavour-makers/ppa/ubuntu bionic InRelease
Reading package lists... Done                      
E: The repository 'file:/opt/raspi-apt-repo/ubuntu18 ./ Release' is not signed.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.
```
可以看到，```Release``` 文件读到了，但 ```apt``` 认为 ```*Release is not signed```，于是认为从这个仓库安装软件还是不安全的，```Packages``` 文件还是没有被读取。

这个 ```Release.gpg``` 又是什么东西，该怎样生成它呢?

根据 [DebianRepository/Format - Debian Wiki](https://wiki.debian.org/DebianRepository/Format?action=show&redirect=RepositoryFormat#A.22Packages.22_Indices) 的说法, ```The file "Release.gpg" contains a GPG signature```，并且与那个 ```InRelease``` 文件有关系:
```
    The file "dists/$DIST/InRelease" shall contain meta-information about the distribution and checksums for the indices, possibly signed with a GPG clearsign signature (for example created by "gpg -a -s --clearsign"). For older clients there can also be a "dists/$DIST/Release" file without any signature and the file "dists/$DIST/Release.gpg" with a detached GPG signature of the "Release" file, compatible with the format used by the GPG options "-a -b -s".

    InRelease files are signed in-line while Release files should have an accompanying Release.gpg file.
```
这里说 ```Release.gpg``` 是个签名文件 (Technically speaking, this is an ascii-armored detached gpg signature.) ，随同  ```Release``` 出现的，比较老的客户端只认这两个文件; 而 ```InRelease``` 是内嵌签名的（也就是说，将原来 ```Release``` 的内容和 ```Release.gpg``` 的内容揉到一起了 － 注意这里不是简单地拼到一起），新的客户端才支持这个这个文件．观察一下 ```Debian``` 和 ```Ubuntu``` 的仓库 ( [http://mirrors.ustc.edu.cn/debian/dists/jessie/](http://mirrors.ustc.edu.cn/debian/dists/jessie/), [http://mirrors.ustc.edu.cn/ubuntu/dists/xenial/](http://mirrors.ustc.edu.cn/ubuntu/dists/xenial/) ) , 可以看到 ```Debian``` 的仓库只有 ```Release``` 和 ```Release.gpg``` 这两个文件，而 ```Ubuntu``` 仓库里面这三个文件都有．

下面说说如何生成这两个文件：
1. 如果你跟我一样不知道什么是GPG数字签名，那么先仔细读一下 [GPG入门 - 简书](https://www.jianshu.com/p/1257dbf3ed8e) 或者 [GnuPG 入门教程 | IterNull Blog](https://blog.iternull.com/posts/2016/07/16/GPG-Getting-Started-Tutorial.html) ，大致知道密钥/公钥, 加密/签名, 文本签名/二进制签名，分离式签名等等概念
2. 按照上面文档描述的方法，生成自己的 gpg key: ```gpg --gen-key```
3. 生成 Release.gpg: ```gpg --armor --detach-sign --sign -o Release.gpg Release```
4. 生成 InRelease: ```gpg --clearsign -o InRelease Release```

### 3.3 公钥不被信任问题
以为问题就都解决了吗？用户方再执行一遍试试
```
$ sudo apt update
[sudo] password for bamanzi: 
Get:1 file:/opt/raspi-apt-repo/ubuntu18 ./ InRelease
Ign:1 file:/opt/raspi-apt-repo/ubuntu18 ./ InRelease
Get:2 file:/opt/raspi-apt-repo/ubuntu18 ./ Release [816 B]
Get:2 file:/opt/raspi-apt-repo/ubuntu18 ./ Release [816 B]
Get:3 file:/opt/raspi-apt-repo/ubuntu18 ./ Release.gpg [659 B]
Get:3 file:/opt/raspi-apt-repo/ubuntu18 ./ Release.gpg [659 B]
Ign:3 file:/opt/raspi-apt-repo/ubuntu18 ./ Release.gpg                                          
Hit:4 http://ports.ubuntu.com bionic InRelease                                                               
Get:5 http://ports.ubuntu.com bionic-updates InRelease [88.7 kB]                                                                               
Hit:6 http://ppa.launchpad.net/ubuntu-pi-flavour-makers/ppa/ubuntu bionic InRelease                                                            
Hit:7 http://ports.ubuntu.com bionic-security InRelease                                                                                        
Hit:8 http://ports.ubuntu.com bionic-backports InRelease                                                                                       
Get:9 http://ports.ubuntu.com bionic-updates/main armhf Packages [508 kB]                                                                      
Get:10 http://ports.ubuntu.com bionic-updates/universe armhf Packages [812 kB]                                                                 
Reading package lists... Done                                                                                                                  
W: GPG error: file:/opt/raspi-apt-repo/ubuntu18 ./ Release: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 722D2AFAD8BAD548
E: The repository 'file:/opt/raspi-apt-repo/ubuntu18 ./ Release' is not signed.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.
```
可以看到, ```apt``` 还是没有读取 ```Packages``` 文件，原因是 ```The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 722D2AFAD8BAD548```
也就是说 ```InRelease / Release.gpg``` 虽然签名了，但由于这个签名所用的公钥没有被接受（```NO_PUBKEY 722D2AFAD8BAD548```)

解决办法有三个:
1. 用户端在执行 ```apt update``` 的时候，添加 ```--allow-insecure-repositories``` 选项；在执行 ```apt install pkg``` 的时候，添加 ```--allow-unauthenticated``` 选项
    后面这个--allow-unauthenticated不加也可以，但 apt 会报告如下告警:
    ```
    WARNING: The following packages cannot be authenticated!
      foo bar baz
    Install these packages without verification [y/N]?
    ```
    需要输入 y 才能成功安装软件包（如果是自动化脚本执行安装的话， ```apt update -y foo``` 这样只添加 ```-y``` 是不够的，必须增加上面的选项）

2. 第二个方法是在用户侧修改仓库的配置，改为 ```deb [trusted=yes] http://foo.example.com/repos/raspbian8 ./``` ，注意这里添加了 ```trusted=yes``` 选项。（这里的修改方法是只对此仓库添加信任选项；网上也有人说在 ```/etc/apt/apt.conf.d``` 下面某个文件里面添加 ```APT::Get::AllowUnauthenticated "true"``` 和 ```Acquire::AllowInsecureRepositories "true"```　这样两行，但这样影响的所有仓库，从安全的角度来说并不推荐）
3. 第三个方法首先要服务侧将其公钥导出 (```gpg --export --armor <uid> -o my-repo.gpg-key.asc``` )并且以某种方式将这个 ```my-repo.gpg-key.asc``` 文件提供给用户( 比如公布在仓库根目录让用户手工下载)；然后用户将此公钥导入表示他信任此公钥: ```sudo apt-key add my-repo.gpg-key.asc```

## 4 链接
[Create Your Own Local apt Repository to Avoid “Dependency Hell” - Stiri It ,Servere Linux , Servicii Linux](https://servers-linux.ro/create-your-own-local-apt-repository-to-avoid-dependency-hell/)
[Easy APT Repository · Iain R. Learmonth](https://iain.learmonth.me/blog/2017/2017w383/)
[DebianRepository/Format - Debian ](https://wiki.debian.org/DebianRepository/Format?action=show&amp;redirect=RepositoryFormat#A.22Packages.22_Indices)
[DebianRepository/Setup - Debian Wiki](https://wiki.debian.org/DebianRepository/Setup?action=show&amp;redirect=HowToSetupADebianRepository)
[DebianRepository/UseThirdParty - De](https://wiki.debian.org/DebianRepository/UseThirdParty?action=show&amp;redirect=RepositoryInstructions)
[SecureApt - Debian Wiki](https://wiki.debian.org/SecureApt)
[apt-secure(8) — apt — Debian stretch — Debian Manpages](https://manpages.debian.org/stretch/apt/apt-secure.8.en.html)
[GPG入门教程 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2013/07/gpg.html)
[GnuPG 入门教程 | IterNull Blog](https://blog.iternull.com/posts/2016/07/16/GPG-Getting-Started-Tutorial.html)
[GPG入门 - 简书](https://www.jianshu.com/p/1257dbf3ed8e)
[Setting up a personal secure apt repository](https://debian-administration.org/article/717/Setting_up_a_personal_secure_apt_repository)
[15.3. Creating a Package Repository for APT - The Debian nistrator's Handbook](https://debian-handbook.info/browse/stable/sect.setup-apt-package-repository.html)
[Aptly - A Debian Repository Management Tool](https://www.unixmen.com/introducing-aptly-a-debian-repository-management-tool/)
