---
layout: post
title:  "GStreamer 1.0 errors on DaVinci"
date:   2015-01-05 13:14:00
categories: openwrt
---

I got some erros with gstreamer on my DM36x platform.
And also default openwrt gstreamer don't include video4linux2 support, so it useless for me.

I created new [package feed][cwp] wich includes updated i2c-tools, libv4l and gstreamer.
Later i will add packages for encoder (dvsdk, gst-ce-plugins).

But i still don't recognize why `gst-inspect-1.0` blacklist some of plugins (video4linux2).

Usually:

{% highlight text %}
root@ipnc:~# gst-inspect-1.0
...
Total count: 25 plugins (10 blacklist entries not shown), 136 features
root@ipnc:~# gst-inspect-1.0 -b
Blacklisted files:
  libgstvolume.so
  libgstvideo4linux2.so
  libgsttypefindfunctions.so
  libgstsubenc.so
  libgstrtpmanager.so
  libgstmpegtsmux.so
  libgstmatroska.so
  libgstinterleave.so
  libgstbayer.so
  libgstautodetect.so

Total count: 10 blacklisted files
{% endhighlight %}


With enabled debug messages i got:

{% highlight text %}
root@ipnc:~# gst-inspect-1.0 --gst-debug-level=4
...
0:00:00.696318725  1159  0x11a0e80 ERROR     GST_PLUGIN_LOADING gstpluginloader.c:673:write_one: Packet magic number is missing. Memory corruption detected
0:00:00.755484889  1159  0x11a0e80 ERROR     GST_PLUGIN_LOADING gstpluginloader.c:277:plugin_loader_replay_pending: Plugin file /usr/lib/gstreamer-1.0/libgstautodetect.so failed to load. Blacklisting
...
{% endhighlight %}


But i fonud workaround:

{% highlight text %}
root@ipnc:~# rm -rf .cache/gstreamer-1.0/
root@ipnc:~# gst-inspect-1.0 --gst-disable-registry-fork
...
Total count: 25 plugins, 301 features
{% endhighlight %}


Still in todo:

- test composite video output
- build TI codecs and [gstreamer plugin from RidgeRun][gce]
- choose video captude driver and enable it (linux-3.18 have regular and stagging versions)


[cwp]: https://github.com/CamWRT/packages
[gce]: https://github.com/RidgeRun/gst-ce-plugin
