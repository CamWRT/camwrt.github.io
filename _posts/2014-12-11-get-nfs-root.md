---
layout: post
title:  "SIP-1080P booting to NFS root"
date:   2014-12-11 09:08:00
categories: hacking
---


Getting original kernel and rootfs
----------------------------------

1. Download firmware update package `firmware_TS38ABFG006-ONVIF-P2P-V2.5.1.5_20140821190333.bin`
2. Use [`tpsee_hack`][tphack] to unpack
3. Place 01kernel to your TFTP server (i use `/srv/tftp/ipnc/`)
4. Unpack 02cramfs to your NFS root (`/srv/nfs/ipnc/`) with root privilegies (`cramfsck -x /srv/nfs/ipnc 02cramfs`)


Modifying rootfs
----------------

1. Modify `/etc/inittab`: uncomment string with ttyS0 login.
2. Comment out all lhonnyling additions to `/etc/init.d/rcS` (this change will disable camera software, but prevents from NFS error 101).


Modify U-Boot env
-----------------

Note: save somewhere current environment (`printenv`).

{% highlight text %}
DM368 IPNC :>setenv bootcmd tftp 0x80700000 ipnc/01kernel; bootm 0x80700000
DM368 IPNC :>setenv bootargs mem=55M console=ttyS0,115200n8 rw root=/dev/nfs rootfstype=nfs nfsroot=192.168.1.22:/srv/nfs/ipnc ip=192.168.1.80
DM368 IPNC :>saveenv
{% endhighlight %}


New information
---------------

### /proc/mtd

{% highlight text %}
dev:    size   erasesize  name
mtd0: 00300000 00020000 "bootloader"
mtd1: 00200000 00020000 "params"
mtd2: 00200000 00020000 "kernel"
mtd3: 01800000 00020000 "filesystem"
mtd4: 00200000 00020000 "data1"
mtd5: 02000000 00020000 "data2"
mtd6: 00200000 00020000 "bk-kernel"
mtd7: 01800000 00020000 "bk-filesys"
mtd8: 02500000 00020000 "update"
{% endhighlight %}


### /proc/devices

{% highlight text %}
Character devices:
  1 mem
  4 /dev/vc/0
  4 tty
  4 ttyS
  5 /dev/tty
  5 /dev/console
  5 /dev/ptmx
  7 vcs
 10 misc
 13 input
 14 sound
 21 sg
 81 video4linux
 89 i2c
 90 mtd
 93 TS800
 95 ADC_CH0
108 ppp
116 alsa
128 ptm
136 pts
180 usb
189 usb_device
254 rtc

Block devices:
  1 ramdisk
  8 sd
 31 mtdblock
 65 sd
 66 sd
 67 sd
 68 sd
 69 sd
 70 sd
 71 sd
128 sd
129 sd
130 sd
131 sd
132 sd
133 sd
134 sd
135 sd
254 mmc
{% endhighlight %}


### /proc/cpuinfo

{% highlight text %}
Processor       : ARM926EJ-S rev 5 (v5l)
BogoMIPS        : 215.44
Features        : swp half thumb fastmult edsp java 
CPU implementer : 0x41
CPU architecture: 5TEJ
CPU variant     : 0x0
CPU part        : 0x926
CPU revision    : 5
Cache type      : write-back
Cache clean     : cp15 c7 ops
Cache lockdown  : format C
Cache format    : Harvard
I size          : 16384
I assoc         : 4
I line length   : 32
I sets          : 128
D size          : 8192
D assoc         : 4
D line length   : 32
D sets          : 64

Hardware        : DaVinci DM368 IPNC
Revision        : 3650000
Serial          : 0000000000000000
{% endhighlight %}


### /proc/davinci\_clocks

{% highlight text %}
ADCIF_CLK 170000000 0
KEYSCAN_CLK 24000000 0
RTC_CLK 24000000 0
VOICECODEC_CLK 20571428 0
USBCLK 24000000 1
PWM3_CLK 24000000 0
PWM2_CLK 24000000 0
PWM1_CLK 24000000 0
PWM0_CLK 24000000 0
AEMIFCLK 170000000 3
gpio 170000000 1
SPICLK 170000000 0
MMCSDCLK1 48571428 0
MMCSDCLK0 48571428 1
McBSPCLK 170000000 1
I2CCLK 24000000 1
EMACCLK 170000000 1
HPI 170000000 0
UART1 24000000 1
UART0 24000000 1
ARMCLK 432000000 0
{% endhighlight %}

### /proc/gio/registers

{% highlight text %}
GPIO Module:
  0xfbc67000: 0x44830105 PID                
  0xfbc67008: 0x0000007f BINTEN             
GPIO Bank0 and Bank1:
  0xfbc67010: 0xc3efffff Direction          
  0xfbc67014: 0x20100000 Output Data        
  0xfbc67018: 0x20100000 Set Data           
  0xfbc6701c: 0x20100000 Clear Data         
  0xfbc67020: 0x63000000 Input Data         
  0xfbc67024: 0x00000000 Set Rising edge   
  0xfbc67028: 0x00000000 Clear Rising edge 
  0xfbc6702c: 0x00000000 Set Falling edge  
  0xfbc67030: 0x00000000 Clear Falling edge
  0xfbc67034: 0x00000000 Interrupt Status   
GPIO Bank2 and Bank3:
  0xfbc67038: 0x3ef3fff7 Direction          
  0xfbc6703c: 0x000c0008 Output Data        
  0xfbc67040: 0x000c0008 Set Data           
  0xfbc67044: 0x000c0008 Clear Data         
  0xfbc67048: 0x000c001a Input Data         
  0xfbc6704c: 0x00000000 Set Rising edge   
  0xfbc67050: 0x00000000 Clear Rising edge 
  0xfbc67054: 0x00000000 Set Falling edge  
  0xfbc67058: 0x00000000 Clear Falling edge
  0xfbc6705c: 0x00000000 Interrupt Status   
GPIO Bank4:
  0xfbc67060: 0xe67dfffe Direction          
  0xfbc67064: 0x0d800001 Output Data        
  0xfbc67068: 0x0d800001 Set Data           
  0xfbc6706c: 0x0d800001 Clear Data         
  0xfbc67070: 0x09880061 Input Data         
  0xfbc67074: 0x00000000 Set Rising edge   
  0xfbc67078: 0x00000000 Clear Rising edge 
  0xfbc6707c: 0x00000000 Set Falling edge  
  0xfbc67080: 0x00000000 Clear Falling edge
  0xfbc67084: 0x00000000 Interrupt Status   
{% endhighlight %}


[tphack]: https://github.com/CamWRT/tpsee_hack
