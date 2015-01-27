---
layout: post
title:  "CamWRT project suspended"
date:   2015-01-27 13:45:00
categories: openwrt
---

I have not any time to start video processing, so i decide to restore factory firmware.

What i got:

- Working vanilla and openwrt 3.18.1 kernel
- Working base openwrt operating system
- OpenWRT target for DaVinci DM365 and DM368 SOC

What need to do:

- Setup MT9P031 sensor driver
- Setup IPIPE driver
- Make gstreamer streaming chain and test performance.
- Check what wrong with setup of VPBE analog video out driver


Restoring factory firmware
--------------------------

For restoration used file [firmware\_TS38ABFG006-ONVIF-P2P-V2.5.1.5\_20140821190333.bin][fw].


### Prepare files for tftp

1. Unpack firmware using [tpsee\_hack unpack.sh][th]
2. Copy files `01kernel` & `02cramfs` to your tftp folder (my `/srv/tftp/ipnc`).


### Flash firmware via tftp

{% highlight sh %}
#
# U-Boot restore steps, do only commands
#

# erase CamWRT
nand erase 400000

# load & flash kernel
tftp 82000000 ipnc/01kernel
nand write 82000000 500000 200000

# load & flash rootfs
tftp 82000000 ipnc/02cramfs
nand write 82000000 700000 c67000

# restore original u-boot environment
printenv
setenv bootargs mem=55M console=ttyS0,115200n8 root=/dev/mtdblock3 rootfstype=cramfs ip=off eth=\$(ethaddr)
setenv bootcmd nboot 0x80700000 0 0x500000\;bootm 0x80700000
printenv
saveenv

# reboot
reset
{% endhighlight %}


### Restore ASCIINema

[asciinema.org/a/15780][as]

<script type="text/javascript" src="https://asciinema.org/a/15780.js" id="asciicast-15780" async></script>


[fw]: /files/fw/firmware_TS38ABFG006-ONVIF-P2P-V2.5.1.5_20140821190333.bin
[th]: https://github.com/datacompboy/tpsee_hack.git
[as]: http://asciinema.org/a/15780
