# FT1500a uboot 配置
```bash
# printenv
baudrate=115200
bootargs=console=ttyS0,115200 earlyprintk=uart8250-32bit,0x28001000 root=/dev/sda2 initrd=0x95000000,32M rootwait rw KEYBOARDTYPE=pc KEYTABLE=us
bootcmd=run scsi_boot
bootdelay=3
eth1addr=00:00:00:00:00:00
ethact=mii0
ethaddr=00:00:00:00:00:00
fdt_addr=0x100000
fdt_high=0xa0000000
gatewayip=202.197.66.1
initrd_addr=0xa00000
init_size=0x1000000
ipaddr=202.197.66.222
kernel_addr=0x200000
ncip=202.197.66.76
netdev=eth0
netmask=255.255.255.0
preboot=usb xhci start
scsi_boot=run scsi_load_kern;run scsi_load_fdt;run scsi_load_initrd;eq close c0;eq close c1;eq close c4;eq close c5;pci enum;bootm 0x90100000 0x95000000:0x2000000 0x90000000
scsi_load_fdt=ext4load scsi 0:1 0x90000000 dtb/u-boot-general.dtb
scsi_load_initrd=ext4load scsi 0:1 0x95000000 initramfs.img
scsi_load_kern=ext4load scsi 0:1 0x90100000 uImage
scsidevs=1
serverip=202.197.66.116
stdin=usbkbd,serial,nc
```