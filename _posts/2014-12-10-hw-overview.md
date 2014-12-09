---
layout: post
title:  "SIP-1080P / TS38F2 hardware overview"
date:   2014-12-10 02:09:57
categories: hacking
---

This camera module has two board construction.


Board A - CPU
-------------

![board A top](/files/hw-overview/brd_a_top.jpg)

* <span style="color:red">&#x25cf;</span> CPU: DM368
* <span style="color:orange">&#x25cf;</span> RAM: [Hynix HY5PS1G1631C](https://www.google.ru/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0CB4QFjAA&url=http%3A%2F%2Fwww.hynix.com%2Fdatasheet%2Fpdf%2Fdram%2FHY5PS1G4%25288.16%252931C%2528L%2529FP%2528Rev0.3%2529.pdf&ei=pCCDVO3hE-HRywPrioA4&usg=AFQjCNFzxSw8tQDbj9IMpbyNHim1wm2qAw&sig2=_0JJbRDA_31JLqiBA6PPOg&bvm=bv.80642063,d.bGQ)
* <span style="color:yellow">&#x25cf;</span> NAND: [Macronix MX30LF1G08AA](http://www.macronix.com/en-us/Product/Pages/ProductDetail.aspx?PartNo=MX30LF1G08AA)
* <span style="color:green">&#x25cf;</span> ETH PHY: [Micrell KSZ8041NL](http://www.micrel.com/_PDF/Ethernet/datasheets/ksz8041nl.pdf)
* <span style="color:blue">&#x25cf;</span> [SOT23 C08K - SN74VC1G08](http://www.ti.com/lit/ds/symlink/sn74lvc1g08.pdf)
* <span style="color:violet">&#x25cf;</span> TS81 - ???


![board A botton](/files/hw-overview/brd_a_bottom.jpg)

* <span style="color:red">&#x25cf;</span> L13602-ADJ - ??? (National Semi)
* <span style="color:orange">&#x25cf;</span> A14E - ???
* <span style="color:yellow">&#x25cf;</span> XM51 - ???
* <span style="color:green">&#x25cf;</span> 4337UF - looks like 24AA05 I2C EEPROM


Board B - Sensor
----------------

![board B top](/files/hw-overview/brd_b_top.jpg)

![board B botton](/files/hw-overview/brd_b_bottom.jpg)

* <span style="color:red">&#x25cf;</span> [AD8051ART (OPAMP)](http://www.digikey.com/product-detail/en/AD8051ART-REEL7/AD8051ART-REEL7CT-ND/656161)

