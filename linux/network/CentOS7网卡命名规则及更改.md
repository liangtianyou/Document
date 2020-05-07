# CentOS7网卡命名规则及更改
[TOC]
从 CentOS7 开始，系统默认的网卡命名有已经不是 ethX 方式了，会变成如 ens enp 等开头的网卡名称。

原linux内核启动过程中，会默认给网卡以 ethX 方式随机命名，然后再通过 systemd 去 rename 成其他名称。

## 1 如何rename
默认rename流程：
1. 依据/usr/lib/udev/rules.d/60-net.rules， 查看是否有ifcfg-xx配置文件（路径在/etc/sysconfig/network-scripts/)， 是否有定义了指定MAC地址的配置文件（ifcfg-xx ，xx必须和配置文件的内容DEVICE一致）， 如果有， 则命名改网卡；
2. 依据/usr/lib/udev/rules.d/71-biosdevname.rules，如果biosdevname使能了（安装了biosdevname这个包，且内核启动参数显式设置为1），且网卡没有在step1中定义，则按照biosdevname命名规则rename网卡；（注意，如果没有安装biosdevname这个包，就没有这个文件）
3. 依据/lib/udev/rules.d/75-net-description.rules，将udev工具会根据device属性将填写网卡的属性命名，可能一个网卡会有多个维度的名称；
4. udev 根据第三步中的赋值，按照指定的scheme规则，去给在第一步，第二步中没有命名的网卡命名；
5. 这个顺序是在我们没有自定义自己的rules的前提下，如果用户自定义了自己的rules，则用户自定义为优先级最高；

## 2 scheme次序
上面第四步中提到，按照指定的scheme规则，这个规则是什么呢？   

首先说第三步中，提到按照网卡的不同属性命名，系统识别网卡有好几种维度：  
* 比如 BIOS provided index numbers for on-board devices (example: eno1),
* 比如：BIOS provided PCI Express hotplug slot index numbers (example: ens1) 
* 比如：physical location of the connector of the hardware (example: enp2s0), 
* 比如：mac地址
同一个网卡通常同时具有多个维度的名称，systemd在选取的时候，按照有先后次序，使用先命中的。
顺序可以简单理解为 ```eno1-ens1-enp1```
```bash
udevadm info /sys/class/net/eth2 | grep ID_NET_NAME
```

## 3 用户自定义网卡名称

1. 在用户没有自定义rules文件前提下，第一步中的网卡命名方式也可认为是 一种用户自定义的网卡命名，即在/etc/sysconfig/network-scripts/ifcfg-xx 文件，xx就是这个网卡名称，文件内容中体现MAC_ADDRESS、NAME，这种情况下，则会按照配置文件中指定的名称来命名网卡；
2. 如果用户自定义了rules文件，放在/etc/udev/rules.d/目录下，则这个优先级是最高的；比1中ifcfg-xx方式优先级更高，但是如果两者不一致，则在重启network服务时，会依据ifcfg-xx，所以用户不应该同时采用里两种方式给同一个网卡命不同的名称；

## 4 内核启动参数
biosdevnane、net.ifnames

默认就是内核启动参数没有 ```biosdevname``` 也没有 ```net.ifnames``` 参数（其实默认是 ```net.ifnames=1,biosdevname=0```），这种情况下就按照一中进行网卡命名：eno-ens-enp的方式逐个匹配；

但是如果使能了 ```biosdevname```，则会使用 ```biosdevname``` 的命名第一步没有命名的网卡；

bios命名规则：
要么是em开头，要么是p开头；
怎么样使能 ```biosdevname``` 呢？2个条件：
1. 安装biosdevname包
2. 且在内核启动参数中明确 biosdevname=1。
否则使能不了。

内核启动参数net.ifname,

如果在启动参数中增加 net.ifname=0，这个文件会在/lib/udev/rule.d/80-net-name-slot.rules体现使用价值，则告诉系统不用scheme的方式来命名，这个时候，会恢复ethx这种命名方式。

## 5 修改命名规则为ethx规则
1. 安装时修改
在centos7安装时，启动到boot main，按tab或e进入编辑，在quit前增加"net.ifnames=0 biosdevname=0",安装后默认就会采用ethx命名规则。

2. 安装后修改
在 ```/etc/sysconfig/grub``` 文件，在倒数第二行加 ```net.ifnames=0 biosdevname=0```
```bash
[root@anntec ~]# cat  /etc/sysconfig/grub 
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quiet net.ifnames=0 biosdevname=0"
GRUB_DISABLE_RECOVERY="true"
```
执行 ```grub2-mkconfig -o /boot/grub2/grub.cfg``` 生成新的grub.cfg文件
```bash
[root@anntec ~]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-693.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-693.el7.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-327.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-327.el7.x86_64.img
Found linux image: /boot/vmlinuz-0-rescue-eaef47d4b7d140bbb95217c73cd4d2e9
Found initrd image: /boot/initramfs-0-rescue-eaef47d4b7d140bbb95217c73cd4d2e9.img
done
```
重命名网卡名称：执行 ```ifcfg-eno16777736 ifcfg-eth0``` 修改 ```ifcfg-eth0``` 文件中 DEVICE=eth0
```bash
[root@anntec ~]# cd /etc/sysconfig/network-scripts/
[root@anntec network-scripts]# ls
[root@anntec network-scripts]# mv ifcfg-eno16777736 ifcfg-eth0
[root@anntec network-scripts]# vi ifcfg-eth0
DEVICE=eth0
```
重启服务器，验证

参考链接：
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/sec-troubleshooting_network_device_naming
