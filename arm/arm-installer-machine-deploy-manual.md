# ARM安装机镜像部署说明
[TOC]

## 1 安装机目录结构
```bash
/data
├── DK
│   └── REDS-V2.0.3-10
└── SK
    ├── REDS3.3-16-kernel4.4.163
    └── REDS-FTD_V3.1-7-kernel4.4.163
```
/data目录下有 **DK** 和 **SK** 两个子目录，分别用来存放 **单控** 和 **双控** 的系统。
* DK - 单控系统目录
* SK - 双控系统目录

## 2 安装镜像部署
### 2.1 创建目录
在对应系统目录创建和安装镜像名称相同的目录。

比如一个 **双控** 的 **安装镜像** 名称为 **REDS-FTD_V3.1-7-kernel4.4.163.iso**，那就在 **SK** 目录下创建目录 **REDS-FTD_V3.1-7-kernel4.4.163**。

### 2.2 拷贝镜像内容
两种方式：
1. 将镜像的内容拷贝到第一步创建的目录下。
2. 将镜像挂载到第一步创建的目录下。

#### 2.2.1 拷贝内容
把镜像挂载：
```bash
mount -o loop REDS-FTD_V3.1-7-kernel4.4.163.iso /mnt/
```
拷贝镜像内容：
```bash
cp -a /mnt/* /data/SK/REDS-FTD_V3.1-7-kernel4.4.163/
```
####  2.2.2 直接挂载
把镜像直接挂载到第一步创建的目录下。

假设镜像放置在 **/root/iso/** 目录下。
```bash
mount -o loop /root/iso/REDS-FTD_V3.1-7-kernel4.4.163.iso /data/SK/REDS-FTD_V3.1-7-kernel4.4.163
```
记得写入 ```/etc/fstab```，这样下次系统启动时会自动挂载该镜像。
```bash
echo '/root/iso/REDS-FTD_V3.1-7-kernel4.4.163.iso /data/SK/REDS-FTD_V3.1-7-kernel4.4.163 iso9660 rw,loop=/dev/loop0	0 0' >> /etc/fstab
```