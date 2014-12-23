---
layout: post
title:  "OpenWRT first run on TS38F2/SIP-1080P camera module"
date:   2014-12-23 19:47:00
categories: openwrt
---

Finally i got OpenWRT on my SIP-1080P camera!
Sources in my fork: [openwrt-davinci.git][cwrt-gh].


Installation
============

After building OpenWRT we need to flash firmware via TFTP.
You must setup tftp server and place to tftpboot uImage and ubinized rootfs images.
Also current uImage require aligning to NAND erase size, done by `dd`.

{% highlight sh %}
$ sudo cp bin/davinci/openwrt-davinci-dm36x-TS38F2-uImage bin/davinci/openwrt-davinci-dm36x-TS38F2-squashfs-ubinized.bin /srv/tftp
$ dd if=bin/davinci/openwrt-davinci-dm36x-TS38F2-uImage of=/srv/tftp/openwrt-davinci-dm36x-TS38F2-uImage bs=128k conv=sync
{% endhighlight %}


Prepare installation script
---------------------------

{% highlight sh %}
#
# Automatic installation script for TS38F2
#

echo *** Installing OpenWRT to TS38F2

setenv kernelfile openwrt-davinci-dm36x-TS38F2-uImage
setenv rootfsfile openwrt-davinci-dm36x-TS38F2-squashfs-ubinized.bin

# this parameters must match built-in kernel mtd partitions
setenv kernelnaddr 0x400000
setenv kernelnsize 0x800000
setenv rootfsnaddr 0xc00000

echo *** Installing kernel: $(kernelfile)
tftp 0x82000000 $(kernelfile)
nand erase $(kernelnaddr) $(kernelnsize)
nand write $(fileaddr) $(kernelnaddr) $(filesize)

echo *** Installing UBI rootfs: $(rootfsfile)
tftp 0x82000000 $(rootfsfile)
nand erase $(rootfsnaddr)
nand write $(fileaddr) $(rootfsnaddr) $(filesize)

echo *** Original U-Boot environment:
printenv
echo
echo *** Modifying U-Boot environment.

setenv bootcmd  nboot 0x80700000 0 \$(kernelnaddr)\; bootm 0x80700000
setenv bootargs 55M console=ttyS0,115200n8 eth=$(ethaddr) ro rootfstype=squashfs
saveenv
echo *** New environment:
printenv
echo
echo *** Done. Enter \"reset\" to boot new firmware.
{% endhighlight %}

Make `autoscr` image:
{% highlight sh %}
$ grep -ve '^#' install-ts38f2.scr > /tmp/instwrt.scr
$ mkimage -A arm -O u-boot -T script -C none -n "OpenWRT Installer" -d /tmp/instwrt.scr instwrt.uscr
$
$ sudo cp instwrt.uscr /srv/tftp
{% endhighlight %}


Setup U-Boot network && save original bootargs
--------------------

{% highlight sh %}
DM368 IPNC :>setenv serverip 192.168.1.22
DM368 IPNC :>setenv ipaddr 192.168.1.80
DM368 IPNC :>setenv netmask 255.255.255.0
DM368 IPNC :>
DM368 IPNC :>setenv orig_bootargs $(bootargs)
DM368 IPNC :>setenv orig_bootcmd $(bootcmd)
{% endhighlight %}


Run installer
-------------

{% highlight sh %}
DM368 IPNC :>tftp 80700000 instwrt.uscr
DM368 IPNC :>autoscr
{% endhighlight %}


See ASCIINema's
---------------

### This stream shows `autoscr` (but i messed with bootargs)
[asciinema.org/a/14970](https://asciinema.org/a/14970)

<script type="text/javascript" src="https://asciinema.org/a/14970.js" id="asciicast-14970" async></script>


### And here i fix env, openwrt on TS38F2
[asciinema.org/a/14970](https://asciinema.org/a/14971)

<script type="text/javascript" src="https://asciinema.org/a/14971.js" id="asciicast-14971" async></script>


*Update*: script image generation and uImage aligning now added in build script.

[cwrt-gh]: https://github.com/CamWRT/openwrt-davinci
