---
title: PXE安装Debian6

date: 2020-05-15 13.29

tags: 
    - PXE
    - debian6
--

[TOC]

## 1 配置相关
### 1.1 dhcpd.conf 配置文件内容
```bash
[root@bogon ~]# cat /etc/dhcp/dhcpd.conf 
option space pxelinux;
option pxelinux.magic code 208 = string;
option pxelinux.configfile code 209 = text;
option pxelinux.pathprefix code 210 = text;
option pxelinux.reboottime code 211 = unsigned integer 32;
option arch code 93 = unsigned integer 16; # RFC4578
subnet 172.168.0.0 netmask 255.255.0.0{
    range 172.168.100.1 172.168.120.255;
    class "pxeclients" {
        match if substring (option vendor-class-identifier, 0, 9) = "PXEClient";
        next-server 172.168.1.15;
        filename "pxelinux.0";
    }
    ### 预设路由 ###
    option routers 172.168.1.1;
    option broadcast-address 172.168.255.255;
    ### 域名 ###
    option domain-name "perabytes.com";
    option domain-name-servers 172.168.1.1;
    ### 指定租约更新时间 ###
    default-lease-time 600;
    max-lease-time 86400;
    ### 其他
    ddns-update-style none;
    ignore client-updates;
    authoritative;
    allow booting;
    allow bootp;
    allow unknown-clients;
}
```
### 1.2 pxelinux.cfg 配置文件内容
```bash
[root@bogon ~]# cat /var/lib/tftpboot/pxelinux.cfg/default
default installer/default/vesamenu.c32
prompt 0
timeout 0

label install
    menu label ^1) ODSP V4.1-Debian6.0.3-US
    menu default
    kernel installer/linux/debian/debian-6/linux
    append vga=normal initrd=installer/linux/debian/debian-6/initrd.gz -- quiet preseed/url=http://172.168.1.15/odsp/ks/odsp_v4.1_debian6.0.3_us.ks debian-installer/locale=en_US console-keymaps-at/keymap=us netcfg/choose_interface=auto netcfg/get_hostname=localhost netcfg/get_domain=d --
```

### 1.3 odsp_v4.1_debian6.0.3_us.ks 配置文件内容
```bash
[root@bogon ~]# cat /data/odsp/ks/odsp_v4.1_debian6.0.3_us.ks 
d-i netcfg/choose_interface select auto

# 允许本地源
d-i debian-installer/allow_unauthenticated string true
d-i mirror/country string enter information manually
d-i mirror/http/hostname string 172.168.1.15
d-i mirror/http/directory string /debian6
d-i mirror/suite string squeeze
d-i mirror/http/proxy string

# 分区
d-i partman-auto/method string lvm
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-auto/purge_lvm_from_device boolean true
d-i partman-lvm/confirm boolean true

d-i partman-auto/expert_recipe string                         \
      boot-root ::                                            \
              256 1000 100 ext2                                  \
                      $primary{ } $bootable{ }                \
                      method{ format } format{ }              \
                      use_filesystem{ } filesystem{ ext2 }    \
                      mountpoint{ /boot }                     \
              .                                               \
              512 500000 1000000000 ext4                       \
                      $defaultignore{ }                    \
                      $primary{ }                           \
                      method{ lvm }                         \
                      vg_name{ localhost }                    \
              .                                             \
              512 500000  1000000000  ext4                            \
                      $lvmok{ }                             \
                      in_vg{ localhost }                            \
                      lv_name{ root }                       \
                      method{ format }                              \
                      format{ }                             \
                      use_filesystem{ } filesystem{ ext4 }    \
                      mountpoint{ / }                         \
		      options/noatime{ noatime }	\
              .                                               \
              512  100000 1000000000  ext4                                 \
                      $lvmok{ }                               \
                      in_vg{ localhost }                              \
                      lv_name{ var }                         \
                      method{ format } format{ }                \
                      use_filesystem{ } filesystem{ ext4 }    \
                      mountpoint{ /var }                         \
		      options/noatime{ noatime }	\
              .                                                 \
              512  1024 4096 linux-swap                          \
                      $lvmok{ }                               \
                      in_vg{ localhost }                              \
                      lv_name{ swap }                         \
                      method{ swap } format{ }                \
              .


d-i partman/confirm_write_new_label boolean true
d-i partman/choose_partition select Finish 
d-i partman/confirm boolean true

# 时区配置
d-i time/zone string Asia/Shanghai
d-i clock-setup/utc boolean true
d-i clock-setup/ntp boolean false


# 配置源
d-i apt-setup/security-updates boolean false
d-i apt-setup/non-free boolean false
d-i apt-setup/contrib boolean false
base-config apt-setup/security-updates boolean false
base-config apt-setup/services-select multiselect security
base-config apt-setup/security_host string 172.168.1.15
base-config apt-setup/security directory string /debian6

# 密码
passwd passwd/root-password password 123456
passwd passwd/root-password-again password 123456
passwd passwd/make-user boolean false

# 安装设置
d-i grub-installer/only_debian boolean true
d-i finish-install/reboot_in_progress note
d-i base-installer/install-recommends boolean false

tasksel tasksel/first multiselect Standard system

# 安装包
d-i pkgsel/include string ssh rsync nfs-common nfs-kernel-server portmap mini-httpd mdadm lvm2 quota mgetty ntpdate psmisc ifenslave hdparm ethtool lib32stdc++6  libstdc++6 lm-sensors libsensors4-dev openipmi ipmitool libopenipmi-dev libopenipmi0 vsftpd ftp scsitools python-chardet lighttpd sudo python-flup vim sysstat smartmontools acl beep samba samba-common samba-common-bin snmp snmpd at python-serial arping python-pexpect python-simplejson python-imaging

d-i pkgsel/upgrade select none

# Some versions of the installer can report back on what software you have
# installed, and what software you use. The default is not to report back,
# but sending reports helps the project determine what software is most
# popular and include it on CDs.
popularity-contest popularity-contest/participate boolean false

# 安装完成后操作
d-i preseed/late_command       string cd /target/; wget http://172.168.1.15/odsp/ps/odsp_v4.1_debian6.0.3_us.ps; chmod +x ./odsp_v4.1_debian6.0.3_us.ps; chroot ./ ./odsp_v4.1_debian6.0.3_us.ps; rm -f ./odsp_v4.1_debian6.0.3_us.ps

exim4-config exim4/dc_eximconfig_configtype select no configuration at this time
exim4-config exim4/no_config boolean true
exim4-config exim4/no_config boolean true
exim4-config exim4/dc_postmaster string
```

### 1.4 apache 配置文件
```bash
[root@bogon ~]# cat /etc/httpd/conf/httpd.conf
...
<Directory "/var/www">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
...
```

## 2 目录结构
### 2.1 /var/www/
```
[root@bogon ~]# tree /var/www
/var/www
├── debian6 -> /data/debian6/
├── html
└── odsp -> /data/odsp/
```

### 2.2 /data
```
[root@bogon ~]# tree /data
/data
├── debian6
│   ├── dists
│   │   ├── squeeze
│   │   │   ├── InRelease
│   │   │   ├── main
│   │   │   │   ├── binary-amd64
│   │   │   │   │   ├── Packages
│   │   │   │   │   ├── Packages.gz
│   │   │   │   │   └── Release
│   │   │   │   ├── debian-installer
│   │   │   │   │   └── binary-amd64
│   │   │   │   │       ├── Packages
│   │   │   │   │       ├── Packages.gz
│   │   │   │   │       └── Release
│   │   │   │   └── Release
│   │   │   ├── Release
│   │   │   └── Release.gpg
│   │   └── stable
│   │       ├── InRelease
│   │       ├── main
│   │       │   ├── binary-amd64
│   │       │   │   ├── Packages
│   │       │   │   ├── Packages.gz
│   │       │   │   └── Release
│   │       │   ├── debian-installer
│   │       │   │   └── binary-amd64
│   │       │   │       ├── Packages
│   │       │   │       ├── Packages.gz
│   │       │   │       └── Release
│   │       │   └── Release
│   │       ├── Release
│   │       └── Release.gpg
│   ├── pools
│   │   ├── debian-installer
│   │   │   ├── acpi-modules-2.6.32-5-amd64-di_1.76+squeeze6_amd64.udeb
│   │   │   ├── ai-console-setup-udeb_1.9_all.udeb
│   │   │   ├── ...
│   │   │   ├── zlib1g-udeb_1.2.3.4.dfsg-3_amd64.udeb
│   │   │   └── zlib-modules-2.6.32-5-amd64-di_1.76+squeeze6_amd64.udeb
│   │   └── odsp
│   │       ├── acl_2.2.49-4_amd64.deb
│   │       ├── acpi_1.5-2_amd64.deb
│   │       ├── ...
│   │       ├── xz-utils_5.0.0-2_amd64.deb
│   │       └── zlib1g_1.2.3.4.dfsg-3_amd64.deb
│   └── update.sh
└── odsp
    ├── 4.1
    │   └── ODSP-V4.1-Debian6.0.3-US
    ├── ks
    │   └── odsp_v4.1_debian6.0.3_us.ks
    └── ps
        └── odsp_v4.1_debian6.0.3_us.ps
```

### 2.3 /var/lib/tftpboot/
```bash
[root@bogon ~]# tree /var/lib/tftpboot/
/var/lib/tftpboot/
├── installer
│   ├── default
│   │   ├── chain.c32
│   │   ├── mboot.c32
│   │   ├── memdisk
│   │   ├── menu.c32
│   │   ├── pxelinux.0
│   │   ├── pxelinux.cfg
│   │   │   └── default
│   │   └── vesamenu.c32
│   └── linux
│       └── debian
│           └── debian-6
│               ├── initrd.gz
│               └── linux
├── pxelinux.0 -> installer/default/pxelinux.0
└── pxelinux.cfg -> installer/default/pxelinux.cfg
```
