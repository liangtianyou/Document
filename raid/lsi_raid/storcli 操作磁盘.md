[TOC]

## 1 查看控制器信息
```bash
# storcli show all
```
结果如下：
```bash
[root@Storage ~]# storcli show all
Status Code = 0
Status = Success
Description = None

Number of Controllers = 1
Host Name = Storage
Operating System  = Linux4.4.163

System Overview :
===============

-------------------------------------------------------------------------------------
Ctl Model                   Ports PDs DGs DNOpt VDs VNOpt BBU sPR DS  EHS ASOs Hlth  
-------------------------------------------------------------------------------------
  0 AVAGOMegaRAIDSAS9361-8i     8   4   1     1   1     1 Opt On  1&2 Y      3 NdAtn 
-------------------------------------------------------------------------------------

Ctl=Controller Index|DGs=Drive groups|VDs=Virtual drives|Fld=Failed
PDs=Physical drives|DNOpt=DG NotOptimal|VNOpt=VD NotOptimal|Opt=Optimal
Msng=Missing|Dgd=Degraded|NdAtn=Need Attention|Unkwn=Unknown
sPR=Scheduled Patrol Read|DS=DimmerSwitch|EHS=Emergency Hot Spare
Y=Yes|N=No|ASOs=Advanced Software Options|BBU=Battery backup unit
Hlth=Health|Safe=Safe-mode boot


ASO :
===

----------------------------------------------------
Ctl Cl SAS MD R6 WC R5 SS FP Re CR RF CO CW HA SSHA 
----------------------------------------------------
  0 X  U   X  U  U  U  X  U  X  X  X  X  X  X  X    
----------------------------------------------------

Cl=Cluster|MD=Max Disks|WC=Wide Cache|SS=Safe Store|FP=Fast Path|Re=Recovery
CR=CacheCade(Read)|RF=Reduced Feature Set|CO=Cache Offload
CW=CacheCade(Read/Write)|X=Not Available/Not Installed|U=Unlimited|T=Trial
|HA=High Availability |SSHA=Single server High Availability 


Need Attention :
==============

Controller 0 :
============

-----------------------------------------------------------------------
EID:Slt DID State DG     Size Intf Med SED PI SeSz Model            Sp 
-----------------------------------------------------------------------
163:13  188 Offln  0 3.637 TB SAS  HDD N   N  512B ST4000NM003A     U  
-----------------------------------------------------------------------


EID-Enclosure Device ID|Slt-Slot No.|DID-Device ID|DG-DriveGroup
DHS-Dedicated Hot Spare|UGood-Unconfigured Good|GHS-Global Hotspare
UBad-Unconfigured Bad|Onln-Online|Offln-Offline|Intf-Interface
Med-Media Type|SED-Self Encryptive Drive|PI-Protection Info
SeSz-Sector Size|Sp-Spun|U-Up|D-Down|T-Transition|F-Foreign
UGUnsp-Unsupported|UGShld-UnConfigured shielded|HSPShld-Hotspare shielded
CFShld-Configured shielded|Cpybck-CopyBack|CBShld-Copyback Shielded
```
可以看到该设备只有一个RAID卡，Ctl只有一个，并且Ctl编号为0。

## 2.查看RAID卡挂载的磁盘信息
```bash
# storcli /c0 /eall /sall show
```
输出结果如下：
```bash
[root@Storage ~]# storcli /c0 /eall /sall show
Controller = 0
Status = Success
Description = Show Drive Information Succeeded.


Drive Information :
=================

-----------------------------------------------------------------------
EID:Slt DID State DG     Size Intf Med SED PI SeSz Model            Sp 
-----------------------------------------------------------------------
163:13  188 Offln  0 3.637 TB SAS  HDD N   N  512B ST4000NM003A     U  
163:14  187 Onln   0 3.637 TB SAS  HDD N   N  512B ST4000NM003A     U  
163:15  186 Onln   0 3.637 TB SAS  HDD N   N  512B ST4000NM003A     U  
163:16  185 Onln   0 3.637 TB SAS  HDD N   N  512B ST4000NM003A     U  
-----------------------------------------------------------------------

EID-Enclosure Device ID|Slt-Slot No.|DID-Device ID|DG-DriveGroup
DHS-Dedicated Hot Spare|UGood-Unconfigured Good|GHS-Global Hotspare
UBad-Unconfigured Bad|Onln-Online|Offln-Offline|Intf-Interface
Med-Media Type|SED-Self Encryptive Drive|PI-Protection Info
SeSz-Sector Size|Sp-Spun|U-Up|D-Down|T-Transition|F-Foreign
UGUnsp-Unsupported|UGShld-UnConfigured shielded|HSPShld-Hotspare shielded
CFShld-Configured shielded|Cpybck-CopyBack|CBShld-Copyback Shielded
```

## 3 磁盘操作
控制磁盘上线、下线、离线等
```bash
# storcli /cx /ex /sx set good
# storcli /cx /ex /sx set offline
# storcli /cx /ex /sx set online
# storcli /cx /ex /sx set missing
# storcli /cx /ex /sx set jbod
```

通过storcli设置raid卡下挂载硬盘的状态为JBOD的前提是：

1. bios支持；
2. Raid Card支持

## 4 磁盘擦除
```bash
# storcli /cx /ex /sx start erase
# storcli /cx /ex /sx stop erase
# storcli /cx /ex /sx secureerase
```

## 5 升级raid卡固件
需要重启
```bash
# storcli /cx download file=filepath 
```