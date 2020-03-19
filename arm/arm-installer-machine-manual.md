# ARM安装机使用说明
[TOC]

**说明**：
1. 系统类型分为 **单控** - **DK** 和 **双控** - **SK**。
2. 这里 **DK** 和 **SK** 是 **单控** 和 **双控** 简拼的首字母组合的简写。
3. **List disk** 项只列出系统盘以外的磁盘。
4. 选择 **磁盘** 和 **系统** 时，可以使用 **序号**，也可以使用 **名称**，使用 **名称** 时需要和列表中的名称完全一致
5. 操作过程中任何步骤，都可以输入 **q** 退出当前操作。

## 1 主界面
连接显示器后，界面显示如下：
```bash
----------------------- install R.E.D.S ------------------------
1. Install
2. List os
3. List disk
----------------------------------------------------------------
Enter you choise: 
```
一共3个选项，说明如下：
| 选项| 说明 | 输入|
|--|--|--|
| Install   | 安装系统             | **1** 或者 **install** |
| List os   | 列出本机可以安装的系统 | **2** 或者 **ls**      |
| List disk | 列出系统盘以外的磁盘   | **3** 或者 **ld**      |

## 2 安装系统
在 ```Enter you choise:``` 输入 **1** 或者 **install** ，就会进入 **安装** 选项。

### 2.1 第一步 - 选择系统类型
```bash
----------------------------------------------------------------
Enter you choise: install
----------------------------------------------------------------
1. DK
2. SK
----------------------------------------------------------------
n: next page, p: pre page, q: exit
Select OS type: 
```

这里以 **双控** 为例，输入 **SK** 或者 **2** ，进入可安装的双控系统列表界面：

**注意**：大小写须完全一样
### 2.2 第二步 - 选择系统
```bash
Select OS type: SK
SK
----------------------------------------------------------------
1. REDS-FTD_V3.1-7-kernel4.4.163
2. REDS3.3-16-kernel4.4.163
----------------------------------------------------------------
n: next page, p: pre page, q: exit
Select OS: 
```

以安装 **REDS-FTD_V3.1-7-kernel4.4.163** 为例，输入 **1** 或者 **REDS-FTD_V3.1-7-kernel4.4.163**，进入选择磁盘界面。

### 2.3 第三步 - 选择磁盘
```bash
Select OS: 1
REDS-FTD_V3.1-7-kernel4.4.163
----------------------------------------------------------------
id | name  | model                         | sn                  
----------------------------------------------------------------
1  | sdb   | INTEL SSDSCKKI128G8           | PHLA911102WU128I    
----------------------------------------------------------------
n: next page, p: pre page, q: exit
Select disk(e.g. 'sdb sdc', 'all'): 
```
磁盘选择方式：
1. 选择单块磁盘，直接输入 **序号** 或者 **磁盘名**，如 **1** 或者 **sdb**。
2. 多块磁盘，如果想全部安装，直接输入 **all**。
3. 多块磁盘，只想安装部分磁盘，选择输入 **序号** 或者 **磁盘名**，中间用 **空格** 分开，如**sdb sdc**。

这里输入序号 **1** ，开始在 **sdb** 上安装双控系统 **REDS-FTD_V3.1-7-kernel4.4.163**。

### 2.4 第四步 - 开始安装
```bash
Select disk(e.g. 'sdb sdc', 'all'): 1
2009-11-05 11:45:43
sdb install start
/dev/sdb clear
/dev/sdb clear complete
/dev/sdb parted
/dev/sdb1 create fs
/dev/sdb1 create fs complete
/dev/sdb2 create fs
/dev/sdb2 create fs complete
/dev/sdb parted complete
/dev/sdb mount
/dev/sdb mount complete
/dev/sdb copy
/dev/sdb1 copy
/dev/sdb1 copy complete
/dev/sdb2 copy
/dev/sdb2 copy complete
/dev/sdb copy complete
sdb install complete
/dev/sdb install succ
2009-11-05 11:50:41
```

## 3 列出系统
在 ```Enter you choise:``` 输入 **2** 或者 **ls** ，就会进入 **列出可安装的系统** 选项。

### 3.1 第一步 - 选择系统类型
```bash
----------------------------------------------------------------
Enter you choise: ls
----------------------------------------------------------------
1. DK
2. SK
----------------------------------------------------------------
n: next page, p: pre page, q: exit
Select OS type: 
```
这里以 **双控** 为例，输入 **SK** 或者 **2** ，进入可安装的双控系统列表界面：

```bash
Select OS type: SK
SK
----------------------------------------------------------------
1. REDS-FTD_V3.1-7-kernel4.4.163
2. REDS3.3-16-kernel4.4.163
----------------------------------------------------------------
n: next page, p: pre page, q: exit
List OS: 
```

### 3.2 第二步 - 选择系统
选择对应的系统，如果系统文件目录下包含 ```readme.md``` 或者 ```安装说明.md```，会打印如下：
```bash
n: next page, p: pre page, q: exit
List OS:: 1
REDS-FTD_V3.1-7-kernel4.4.163
Manual Start------------------------------------------------
# 安装说明

## 操作系统安装
1. 使用任意Linux系统，将REDS-2.0.3-*.iso挂载到某个目录下（比如/mnt目录）。
如：mount REDS-2.0.3-10(20181018) /mnt -o loop
2. 将要安装系统的系统盘接入Linux系统，系统将识别为新的磁盘（比如sdb）。
3. 进入挂载镜像的目录，执行 install 脚本，显示如下界面，选择 1 安装系统，根据提示会要求你选择要安装的磁盘，输入对应的磁盘（如sdb），回车开始安装。
--------------------- install R.E.D.S ----------------------
1.Install
2.List disks
3.quit
------------------------------------------------------------
Enter you choise: 2
------------------------------------------------------------
name  | model                         | sn                  
------------------------------------------------------------
sdb   | InnoDisk Corp. - mSATA 3ME3   | B2A11703010420146   
------------------------------------------------------------
--------------------- install R.E.D.S ----------------------
1.Install
2.List disks
3.quit
------------------------------------------------------------
Enter you choise: 1
------------------------------------------------------------
name  | model                         | sn                  
------------------------------------------------------------
sdb   | InnoDisk Corp. - mSATA 3ME3   | B2A11703010420146   
------------------------------------------------------------
Select disk (e.g. sdb sdc, q for exit): sdb
2018-15-06 18:46:52
/dev/sdb  install
/dev/sdb  clear
/dev/sdb  clear complete
/dev/sdb  parted
/dev/sdb1 create fs
/dev/sdb1 create fs complete
/dev/sdb2 create fs
/dev/sdb2 create fs complete
/dev/sdb  parted complete
/dev/sdb  mount
/dev/sdb  mount complete
/dev/sdb  copy
/dev/sdb1 copy sda1
/dev/sdb1 copy sda1 complete
/dev/sdb2 copy sda2
/dev/sdb2 copy sda2 complete
/dev/sdb  install complete
2018-15-06 18:48:07
--------------------- install R.E.D.S ----------------------
1.Install
2.List disks
3.quit
------------------------------------------------------------
Enter you choise: 

## 管理系统安装
1. 操作系统对拷完成后，将制作好的硬盘装入ARM操作系统，启动设备。
2. 系统初次启动会自动安装管理系统并重启设备，请耐心等待，安装完成会出现如下界面。
------------ Configuration REDS V2.0.3-10-C70 -------------
1.Network manage
2.Set WEB admin password
S/N :   Model : 
-----------------------------------------------------------
Enter you choise:
Manual End-------------------------------------------------
```

## 4 列出磁盘
在 ```Enter you choise:``` 输入 **3** 或者 **ld** ，就会进入 **列出系统盘以外的磁盘** 选项。
```bash
----------------------------------------------------------------
Enter you choise: 3
----------------------------------------------------------------
id | name  | model                         | sn                  
----------------------------------------------------------------
1  | sdb   | INTEL SSDSCKKI128G8           | PHLA911102WU128I    
----------------------------------------------------------------
```
可以通过磁盘的 **型号**（model）和 **序列号**（SN）来定位磁盘。