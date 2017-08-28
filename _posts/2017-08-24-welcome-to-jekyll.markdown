---
layout: post
title:  "Yi Home Camera"
date:   2017-08-24 21:48:39 +0200
categories: reversing
---
# Getting started
You can download the camera firmware from [www.yitechnology.com][yi-firmware-url].

You can unpack the firmware using [binwalk][github-binwalk] available from github and brew for the mac.

{% highlight bash %}
brew install binwalk
{% endhighlight %}

When unpacking using the -e switch you will require [jefferson][github-jefferson] to read and extract the contents of the JFFS filesystem, this can be installed after cloning from github, on the mac you will need to install the following packages to get jefferson working.

{% highlight bash %}
pip install cstruct
pip install PylibLZMA
{% endhighlight %}

After extracting the filesystem, you can identify that the camera is a HI3518, there is more information about the internals here.

# Tweaking the startup
From the extracted files copy the init.sh to an SD card and place it in /test and rename it to equip_test.sh .
Edit the file and comment out the following lines:
{% highlight bash %}
if [ -f "/home/hd1/test/equip_test.sh" ]; then
	/home/hd1/test/equip_test.sh
	exit
fi
{% endhighlight %}

At the end of the file include the following lines:
{% highlight bash %}
### Get a copy of the etc files
mkdir /home/hd1/test/etc
cp /etc/* /home/hd1/test/etc/

### Dump the MTD blocks
mkdir /home/hd1/test/data

mount -o remount,ro /
dd if=/dev/mtdblock0 of=/home/hd1/test/data/mtdblock0;
dd if=/dev/mtdblock1 of=/home/hd1/test/data/mtdblock1;
dd if=/dev/mtdblock2 of=/home/hd1/test/data/mtdblock2;
dd if=/dev/mtdblock3 of=/home/hd1/test/data/mtdblock3;
dd if=/dev/mtdblock4 of=/home/hd1/test/data/mtdblock4;
dd if=/dev/mtdblock5 of=/home/hd1/test/data/mtdblock5;
dd if=/dev/mtdblock6 of=/home/hd1/test/data/mtdblock6;
mount -o remount,rw /

### Launch Telnet server 
log "Start telnet server..."
telnetd &

{% endhighlight %}

When telnet gives you a login prompt, you are ready to power off and cillect your files. Use [John the Ripper][john-url] to brute force the /etc/passwd file and you will be able to log into the unit.

{% highlight bash %}
brew install john-jumbo
{% endhighlight %}

{% highlight log %}
Linux version 3.0.8 (rock07@Server) (gcc version 4.4.1 (Hisilicon_v100(gcc4.4-290+uclibc_0.9.32.1+eabi+linuxpthread)) ) #1 Wed Apr 30 16:56:49 CST 2014
CPU: ARM926EJ-S [41069265] revision 5 (ARMv5TEJ), cr=00053177
CPU: VIVT data cache, VIVT instruction cache
Machine: hi3518
Memory policy: ECC disabled, Data cache writeback
AXI bus clock 200000000.
On node 0 totalpages: 10240
free_area_init_node: node 0, pgdat c0539a60, node_mem_map c0558000
  Normal zone: 80 pages used for memmap
  Normal zone: 0 pages reserved
  Normal zone: 10160 pages, LIFO batch:1
pcpu-alloc: s0 r0 d32768 u32768 alloc=1*32768
pcpu-alloc: [0] 0
Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 10160
Kernel command line: mem=40M console=ttyAMA0,115200 root=/dev/mtdblock4 rootfstype=jffs2 mtdparts=hi_sfc:256k(boot)ro,128k(env),128k(conf),3072k(os),3584k(rootfs),9088k(home),128k(vd)
PID hash table entries: 256 (order: -2, 1024 bytes)
Dentry cache hash table entries: 8192 (order: 3, 32768 bytes)
Inode-cache hash table entries: 4096 (order: 2, 16384 bytes)
Memory: 40MB = 40MB total
Memory: 35080k/35080k available, 5880k reserved, 0K highmem
Virtual kernel memory layout:
    vector  : 0xffff0000 - 0xffff1000   (   4 kB)
    fixmap  : 0xfff00000 - 0xfffe0000   ( 896 kB)
    DMA     : 0xffc00000 - 0xffe00000   (   2 MB)
    vmalloc : 0xc3000000 - 0xfe000000   ( 944 MB)
    lowmem  : 0xc0000000 - 0xc2800000   (  40 MB)
    modules : 0xbf000000 - 0xc0000000   (  16 MB)
      .init : 0xc0008000 - 0xc0029000   ( 132 kB)
      .text : 0xc0029000 - 0xc0515000   (5040 kB)
      .data : 0xc0516000 - 0xc053a0e0   ( 145 kB)
       .bss : 0xc053a104 - 0xc0557380   ( 117 kB)
SLUB: Genslabs=13, HWalign=32, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
NR_IRQS:32 nr_irqs:32 32
sched_clock: 32 bits at 100MHz, resolution 10ns, wraps every 42949ms
Console: colour dummy device 80x30
Calibrating delay loop... 218.72 BogoMIPS (lpj=1093632)
pid_max: default: 32768 minimum: 301
Mount-cache hash table entries: 512
CPU: Testing write buffer coherency: ok
NET: Registered protocol family 16
Serial: AMBA PL011 UART driver
uart:0: ttyAMA0 at MMIO 0x20080000 (irq = 5) is a PL011 rev2
console [ttyAMA0] enabled
uart:1: ttyAMA1 at MMIO 0x20090000 (irq = 5) is a PL011 rev2
bio: create slab <bio-0> at 0
SCSI subsystem initialized
usbcore: registered new interface driver usbfs
usbcore: registered new interface driver hub
usbcore: registered new device driver usb
cfg80211: Calling CRDA to update world regulatory domain
Switching to clocksource timer1
NET: Registered protocol family 2
IP route cache hash table entries: 1024 (order: 0, 4096 bytes)
TCP established hash table entries: 2048 (order: 2, 16384 bytes)
TCP bind hash table entries: 2048 (order: 1, 8192 bytes)
TCP: Hash tables configured (established 2048 bind 2048)
TCP reno registered
UDP hash table entries: 256 (order: 0, 4096 bytes)
UDP-Lite hash table entries: 256 (order: 0, 4096 bytes)
NET: Registered protocol family 1
RPC: Registered named UNIX socket transport module.
RPC: Registered udp transport module.
RPC: Registered tcp transport module.
RPC: Registered tcp NFSv4.1 backchannel transport module.
NetWinder Floating Point Emulator V0.97 (double precision)
VFS: Disk quotas dquot_6.5.2
Dquot-cache hash table entries: 1024 (order 0, 4096 bytes)
JFFS2 version 2.2. (NAND) © 2001-2006 Red Hat, Inc.
fuse init (API version 7.16)
yaffs: yaffs built Apr 30 2014 16:56:01 Installing.
msgmni has been set to 68
Block layer SCSI generic (bsg) driver version 0.4 loaded (major 253)
io scheduler noop registered
io scheduler deadline registered (default)
io scheduler cfq registered
brd: module loaded
loop: module loaded
Spi id table Version 1.22
Hisilicon Spi Flash Controller V350 Device Driver, Version 1.10
Spi(cs1) ID: 0xC8 0x40 0x18 0xC8 0x40 0x18
SPI FLASH start_up_mode is 3 Bytes
Spi(cs1):
Block:64KB
Chip:16MB
Name:"GD25Q128"
spi size: 16MB
chip num: 1
7 cmdlinepart partitions found on MTD device hi_sfc
Creating 7 MTD partitions on "hi_sfc":
0x000000000000-0x000000040000 : "boot"
0x000000040000-0x000000060000 : "env"
0x000000060000-0x000000080000 : "conf"
0x000000080000-0x000000380000 : "os"
0x000000380000-0x000000700000 : "rootfs"
0x000000700000-0x000000fe0000 : "home"
0x000000fe0000-0x000001000000 : "vd"
Fixed MDIO Bus: probed
himii: probed
usbcore: registered new interface driver rt2500usb
usbcore: registered new interface driver rt73usb
ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
hiusb-ehci hiusb-ehci.0: HIUSB EHCI
hiusb-ehci hiusb-ehci.0: new USB bus registered, assigned bus number 1
hiusb-ehci hiusb-ehci.0: irq 15, io mem 0x100b0000
hiusb-ehci hiusb-ehci.0: USB 0.0 started, EHCI 1.00
hub 1-0:1.0: USB hub found
hub 1-0:1.0: 1 port detected
ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
hiusb-ohci hiusb-ohci.0: HIUSB OHCI
hiusb-ohci hiusb-ohci.0: new USB bus registered, assigned bus number 2
hiusb-ohci hiusb-ohci.0: irq 16, io mem 0x100a0000
hub 2-0:1.0: USB hub found
hub 2-0:1.0: 1 port detected
usbcore: registered new interface driver cdc_acm
cdc_acm: USB Abstract Control Model driver for USB modems and ISDN adapters
usbcore: registered new interface driver cdc_wdm
Initializing USB Mass Storage driver...
usbcore: registered new interface driver usb-storage
USB Mass Storage support registered.
usbcore: registered new interface driver ums-alauda
usbcore: registered new interface driver ums-datafab
usbcore: registered new interface driver ums-freecom
usbcore: registered new interface driver ums-isd200
usbcore: registered new interface driver ums-jumpshot
usbcore: registered new interface driver ums-sddr09
usbcore: registered new interface driver ums-sddr55
usbcore: registered new interface driver mdc800
mdc800: v0.7.5 (30/10/2000):USB Driver for Mustek MDC800 Digital Camera
mousedev: PS/2 mouse device common for all mice
usbcore: registered new interface driver usbhid
usbhid: USB HID core driver
TCP cubic registered
Initializing XFRM netlink socket
NET: Registered protocol family 10
NET: Registered protocol family 17
NET: Registered protocol family 15
lib80211: common routines for IEEE802.11 drivers
lib80211_crypt: registered algorithm 'NULL'
Registering the dns_resolver key type
registered taskstats version 1
drivers/rtc/hctosys.c: unable to open rtc device (rtc0)
mmc0: new SDHC card at address 0007
usb 1-1: new high speed USB device number 2 using hiusb-ehci
mmcblk0: mmc0:0007 SL32G 28.9 GiB
 mmcblk0: p1
VFS: Mounted root (jffs2 filesystem) on device 31:4.
Freeing init memory: 132K
udevd (636): /proc/636/oom_adj is deprecated, please use /proc/636/oom_score_adj instead.
ADDRCONF(NETDEV_UP): eth0: link is not ready

[CPLD_PERIPH] cpld_init ok, [ ver=Feb  8 2017, 11:23:09 ]
Hisilicon Media Memory Zone Manager
hi3518_base: module license 'Proprietary' taints kernel.
Disabling lock debugging due to kernel taint
Hisilicon UMAP device driver interface: v3.00
pa:82800000, va:c3200000
load sys.ko ...OK!
acodec inited!
==>[0]:PreBuff:0xc0e90000, DmaAddr:0x80e90000
==>[1]:PreBuff:0xc0ec0000, DmaAddr:0x80ec0000
==>[2]:PreBuff:0xc0f10000, DmaAddr:0x80f10000
==>[3]:PreBuff:0xc0f20000, DmaAddr:0x80f20000
==>[4]:PreBuff:0xc0f80000, DmaAddr:0x80f80000
==>[5]:PreBuff:0xc1a20000, DmaAddr:0x81a20000
==>[6]:PreBuff:0xc1a21000, DmaAddr:0x81a21000
==>[7]:PreBuff:0xc0cd8000, DmaAddr:0x80cd8000
==>[8]:PreBuff:0xc1b98000, DmaAddr:0x81b98000
==>[9]:PreBuff:0xc1ca0000, DmaAddr:0x81ca0000
==>[10]:PreBuff:0xc0e88000, DmaAddr:0x80e88000
==>[11]:PreBuff:0xc0ed8000, DmaAddr:0x80ed8000
==>[12]:PreBuff:0xc0ef8000, DmaAddr:0x80ef8000
==>[13]:PreBuff:0xc0f08000, DmaAddr:0x80f08000
==>[14]:PreBuff:0xc0f30000, DmaAddr:0x80f30000
==>[15]:PreBuff:0xc1b02800, DmaAddr:0x81b02800
install prealloc ok
STA : @@@@@@ rtusb init rt2870 --->


=== pAd = c347d000, size = 1567864 ===

[0] pHTTXContext TransferBuffer=0xc0e90000,DMA=0x80e90000
[1] pHTTXContext TransferBuffer=0xc0ec0000,DMA=0x80ec0000
[2] pHTTXContext TransferBuffer=0xc0f10000,DMA=0x80f10000
[3] pHTTXContext TransferBuffer=0xc0f20000,DMA=0x80f20000
Alloc TX null/PsPoll
Alloc TX null/PsPoll
[0] pRxContext TransferBuffer=0xc0cd8000,DMA=0x80cd8000
[1] pRxContext TransferBuffer=0xc1b98000,DMA=0x81b98000
[2] pRxContext TransferBuffer=0xc1ca0000,DMA=0x81ca0000
[3] pRxContext TransferBuffer=0xc0e88000,DMA=0x80e88000
[4] pRxContext TransferBuffer=0xc0ed8000,DMA=0x80ed8000
[5] pRxContext TransferBuffer=0xc0ef8000,DMA=0x80ef8000
[6] pRxContext TransferBuffer=0xc0f08000,DMA=0x80f08000
[7] pRxContext TransferBuffer=0xc0f30000,DMA=0x80f30000
pCmdRspEventContext TransferBuffer=0xc1b02800,DMA=0x81b02800
 RTMPAllocTxRxRingMemory, Status=0
 RTMPAllocAdapterBlock, Status=0
RTMP_COM_IoctlHandle():pAd->BulkOutEpAddr=0x8
RTMP_COM_IoctlHandle():pAd->BulkOutEpAddr=0x4
RTMP_COM_IoctlHandle():pAd->BulkOutEpAddr=0x5
RTMP_COM_IoctlHandle():pAd->BulkOutEpAddr=0x6
RTMP_COM_IoctlHandle():pAd->BulkOutEpAddr=0x7
RTMP_COM_IoctlHandle():pAd->BulkOutEpAddr=0x9
==>WaitForAsicReady MAC_CSR0=0x76010500
==>WaitForAsicReady MAC_CSR0=0x76010500
NVM is EFUSE
Endpoint(8) is for In-band Command
Endpoint(4) is for WMM0 AC0
Endpoint(5) is for WMM0 AC1
Endpoint(6) is for WMM0 AC2
Endpoint(7) is for WMM0 AC3
Endpoint(9) is for WMM1 AC0
Endpoint(84) is for Data-In
Endpoint(85) is for Command Rsp
usbcore: registered new interface driver rt2870
================> UP : RTMP_SEM_EVENT_WAIT(STA)
1. LDO_CTR0(6c) = a64799, PMU_OCLEVEL c
2. LDO_CTR0(6c) = a6478d, PMU_OCLEVEL 6
==>WaitForAsicReady MAC_CSR0=0x76010500
FW Version:0.1.00 Build:7640
Build Time:201308201452____
ILM Length = 46472(bytes)
DLM Length = 0(bytes)
Loading FW....
#
RTMP_TimerListAdd: add timer obj c35a24f4!
RTMP_TimerListAdd: add timer obj c35a2524!
RTMP_TimerListAdd: add timer obj c35a2554!
RTMP_TimerListAdd: add timer obj c35a24c4!
RTMP_TimerListAdd: add timer obj c35a2434!
RTMP_TimerListAdd: add timer obj c35a2464!
RTMP_TimerListAdd: add timer obj c3536e6c!
RTMP_TimerListAdd: add timer obj c3523768!
RTMP_TimerListAdd: add timer obj c352379c!
RTMP_TimerListAdd: add timer obj c3536f0c!
RTMP_TimerListAdd: add timer obj c3526118!
RTMP_TimerListAdd: add timer obj c3525958!
RTMP_TimerListAdd: add timer obj c35260e4!
RTMP_TimerListAdd: add timer obj c3526434!
RTMP_TimerListAdd: add timer obj c352614c!
RTMP_TimerListAdd: add timer obj c3526180!
RTMP_TimerListAdd: add timer obj c35261b4!
RTMP_TimerListAdd: add timer obj c348132c!
RTMP_TimerListAdd: add timer obj c3480b6c!
RTMP_TimerListAdd: add timer obj c34812f8!
RTMP_TimerListAdd: add timer obj c3481648!
RTMP_TimerListAdd: add timer obj c348157c!
RTMP_TimerListAdd: add timer obj c34b6564!
RTMP_TimerListAdd: add timer obj c34b5da4!
RTMP_TimerListAdd: add timer obj c34b6530!
RTMP_TimerListAdd: add timer obj c34b6880!
RTMP_TimerListAdd: add timer obj c34b6598!
RTMP_TimerListAdd: add timer obj c34b65cc!
RTMP_TimerListAdd: add timer obj c34b6600!
RTMP_TimerListAdd: add timer obj c3536e0c!
RTMP_TimerListAdd: add timer obj c3536edc!
RTMP_TimerListAdd: add timer obj c35f9924!
RTMP_TimerListAdd: add timer obj c35f9954!
RTMP_TimerListAdd: add timer obj c35f9984!
RTMP_TimerListAdd: add timer obj c35f99b4!
RTMP_TimerListAdd: add timer obj c35f99e4!
RTMP_TimerListAdd: add timer obj c35f9a18!
RTMP_TimerListAdd: add timer obj c35a2494!
RTMP_TimerListAdd: add timer obj c3522784!
RTMP_TimerListAdd: add timer obj c3522754!
RTMP_TimerListAdd: add timer obj c3522724!
RTMP_TimerListAdd: add timer obj c3536e3c!
P2pGroupTabInit .
P2pScanChannelDefault <=== count = 3, Channels are 1, 6,11 separately
P2pCfgInit::
==>WaitForAsicReady MAC_CSR0=0x76010500
cfg_mode=9
cfg_mode=9
wmode_band_equal(): Band Equal!
Key1Str is Invalid key length(0) or Type(0)
Key2Str is Invalid key length(0) or Type(0)
Key3Str is Invalid key length(0) or Type(0)
Key4Str is Invalid key length(0) or Type(0)
1. Phy Mode = 14
2. Phy Mode = 14
NVM is Efuse and its size =1d[1e0-1fc]
ERROR!!! MT7601 E2PROM: WRONG VERSION 0xc, should be 9
3. Phy Mode = 14
AntCfgInit: RxAntDiversityCfg 6
AntCfgInit: primary/secondary ant 0/1
AntCfgInit: RxAntDiv 0/0
---> InitFrequencyCalibration
InitFrequencyCalibrationMode:Unknow mode = 3
InitFrequencyCalibration: frequency offset in the EEPROM = 103(0x67)
- InitFrequencyCalibration
RTMPSetPhyMode: channel is out of range, use first channel=1
MCS Set = ff 00 00 00 01
== STA : rt28xx_init, Status=0
0x1300 = 00064300
RTMPDrvOpen(1):Check if PDMA is idle!
RTMPDrvOpen(2):Check if PDMA is idle!
============== UP : RTMP_SEM_EVENT_UP(STA)
elevator: type anticipatory not found
elevator: switch to anticipatory
 failed
elevator: type anticipator not found
elevator: switch to anticipator failed
elevator: type anticipator not found
elevator: switch to anticipator failed
load viu.ko ...OK!
ISP Mod init!
load vpss.ko ....OK!
load venc.ko ...OK!
load group.ko ...OK!
load chnl.ko ...OK!
load h264e.ko ...OK!
load jpege.ko ...OK!
load rc.ko ...OK!
load region.ko ....OK!
load vda.ko ....OK!
hi_i2c init is ok!
Hisilicon Watchdog Timer: 0.01 initialized. default_margin=60 sec (nowayout= 0, nodeamon= 0)
Kernel: ssp initial ok!
cfg_mode=9
wmode_band_equal(): Band Equal!
ra0: no IPv6 routers present
Load hi_cipher.ko success.
===>rt_ioctl_giwscan. 25(25) BSS returned, data->length = 4007
==>rt_ioctl_siwfreq::SIOCSIWFREQ(Channel=2)
PeerBeaconAtJoinAction(): Set CentralChannel=2
{% endhighlight %}

[yi-firmware-url]: https://www.yitechnology.com/support/firmware_home/id/4.html
[github-binwalk]: https://github.com/devttys0/binwalk
[github-jefferson]: https://github.com/sviehb/jefferson
[john-url]: http://www.openwall.com/john/

<!-- 
Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
 -->