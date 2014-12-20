---
layout: post
title:  "Dump DM368 PINMUX states."
date:   2014-12-13 23:08:00
categories: hacking
---

I wrote simple program to determine how original drivers configure GPIOs.

Repository: [dm36x_debug_tools][dtools].
PINMUX description: [sprufg5a.pdf][sprufg5a].

SIP-1080P PINMUX
----------------

{% highlight text %}
# ./d_pinmux 
DEVICE_ID: 0x8B83E02F
        DEVREV: 8
        PARTNUM: B83E
        MFGR: 017
PINMUX0: 0x00FC0000
        MMC/SD0: 0
        GIO49: 1
        GIO48: 1
        GIO47: 1
        GIO46: 1
        GIO45: 1
        GIO44: 1
        GIO43: 0
        C_WE_FIELD: 0
        VD: 0
        HD: 0
        YIN0: 0
        YIN1: 0
        YIN2: 0
        YIN3: 0
        YIN4: 0
        YIN5: 0
        YIN6: 0
        YIN7: 0
PINMUX1: 0x00430000
        VCLK: 1
        EXTCLK: 0
        FIELD: 0
        LCD_OE: 1
        HVSYNC: 1
        COUT0: 0
        COUT1: 0
        COUT2: 0
        COUT3: 0
        COUT4: 0
        COUT5: 0
        COUT6: 0
        COUT7: 0
PINMUX2: 0x00001980
        EM_CLK: 1
        EM_ADV: 1
        EM_WAIT: 0
        EM_WE_OE: 0
        !EM_CE1: 1
        !EM_CE0: 1
        EM_D15_8: 0
        EM_A7: 0
        EM_A3: 0
        EM_AR: 0
PINMUX3: 0x615AFFFF
        GIO26: 0
        GIO25: 3
        GIO24: 0
        GIO23: 0
        GIO22: 0
        GIO21: 2
        GIO20: 2
        GIO19: 1
        GIO18: 1
        GIO17: 1
        GIO16: 1
        GIO15: 1
        GIO14: 1
        GIO13: 1
        GIO12: 1
        GIO11: 1
        GIO10: 1
        GIO9: 1
        GIO8: 1
        GIO7: 1
        GIO6: 1
        GIO5: 1
        GIO4: 1
        GIO3: 1
        GIO2: 1
        GIO1: 1
PINMUX4: 0x0030C000
        GIO42: 0
        GIO41: 0
        GIO40: 0
        GIO39: 0
        GIO38: 0
        GIO37: 3
        GIO36: 0
        GIO35: 0
        GIO34: 3
        GIO33: 0
        GIO32: 0
        GIO31: 0
        GIO30: 0
        GIO29: 0
        GIO28: 0
        GIO27: 0
ARM_INTMUX: 0x0003C401
        INT0: 0
        INT7: 0
        INT8: 0
        INT62: 0
        INT61: 0
        INT59: 0
        INT58: 0
        INT57: 0
        INT56: 0
        INT55: 1
        INT54: 1
        INT53: 1
        INT52: 1
        INT43: 0
        INT38: 0
        INT30: 0
        INT29: 1
        INT28: 0
        INT26: 0
        INT24: 0
        INT20: 0
        INT19: 0
        INT18: 0
        INT17: 0
        INT13: 0
        INT10: 1
EDMA_EVTMUX: 0x0003C000
        EVT63: 0
        EVT62: 0
        EVT61: 0
        EVT60: 0
        EVT59: 0
        EVT58: 1
        EVT57: 1
        EVT56: 1
        EVT55: 1
        EVT54: 0
        EVT53: 0
        EVT43: 0
        EVT42: 0
        EVT41: 0
        EVT40: 0
        EVT26: 0
        EVT19: 0
        EVT18: 0
        EVT12: 0
        EVT3: 0
        EVT2: 0
VDAC_CONFIG: 0x081141CF
        TVSHORT: 0
        TVINT: 0
        PDTVSHORTZ: 0
        XDMODE: 0
        PWDNZ_TVDETECT: 0
        PWDNBUFZ: 1
        PWD_C: 1
        PWD_B: 1
        PWD_A: 1
USB_PHY_CTRL: 0x000021E0
        PHYCLKFREQ: 2
        DATAPOL: 0
        PHYCLKSRC: 0
        PHYCLKGD: 1
        SESNDEN: 1
        VBDTCTEN: 1
        VBUSENS: 1
        PHYPLLON: 0
        OTGPDWN: 0
        PHYPDWN: 0
VPSS_CLK_CTRL: 0x00000038
        VPSS_CLKMD: 0
        VENC_CLK_SRC: 1
        DACCLKEN: 1
        VENCCLKEN: 1
        PCLK_INV: 0
        VPSS_MUXSEL: 0
{% endhighlight %}


Peripheral device pins
----------------------

* McBSP (but i don't see any codec on board)
  - GIO49..GIO44 - i2s/codec signals
* Video In - all enabled (YIN, default?)
* Vidio Out - all disabled (COUT, GIO mode)
* I2C
  - GIO21 - I2C_SDA
  - GIO20 - I2C_SCL (note some magic in dmesg with this pin)
* UART1
  - GIO25 - UART1_TXD
  - GIO34 - UART1_RXD
* UART0 (console)
  - GIO19 - UART0_RXD
  - GIO18 - UART0_RXD
* EMAC (Ethernet)
  - GIO17..GIO1 - MII signals
* Clock output (image sensor clock)
  - GIO37 - CLKOUT0


Trying to guess GPIOs
---------------------

Using d_gpio utility i found what GIOs set to output.

{% highlight text %}
# /tmp/ipnc/d_gpio 
DIR 01: 0xC3EFFFFF: I I O O O O I I  I I I O I I I I  I I I I I I I I  I I I I I I I I  
IN  01: 0x63000000: 0 1 1 0 0 0 1 1  0 0 0 0 0 0 0 0  0 0 0 0 0 0 0 0  0 0 0 0 0 0 0 0  
DIR 23: 0x3EF3FFF5: O O I I I I I O  I I I I O O I I  I I I I I I I I  I I I I O I O I  
IN  23: 0x000C001A: 0 0 0 0 0 0 0 0  0 0 0 0 1 1 0 0  0 0 0 0 0 0 0 0  0 0 0 1 1 0 1 0  
DIR 45: 0xE67DFFFE: I I I O O I I O  O I I I I I O I  I I I I I I I I  I I I I I I I O  
IN  45: 0x09880061: 0 0 0 0 1 0 0 1  1 0 0 0 1 0 0 0  0 0 0 0 0 0 0 0  0 1 1 0 0 0 0 1  
DIR  6: 0xFFFFFFFF: I I I I I I I I  I I I I I I I I  I I I I I I I I  I I I I I I I I  
IN   6: 0x00000000: 0 0 0 0 0 0 0 0  0 0 0 0 0 0 0 0  0 0 0 0 0 0 0 0  0 0 0 0 0 0 0 0  
{% endhighlight %}

Then i try to change GIO state via `echo 1 > /proc/gio/gioX` and found IR CUT control lines:

- GIO26: state 1 (need to determine filter type)
- GIO27: state 2

Just send 1 to one of that GIO and filter will flip, then disable coils by setting all to 0.


Findings:

- GIO28: controls IR LEDs power (side connector on module)
- GIO33: red status LED (inverted).
- GIO50: possible PHY reset (when i pull it 0, eth link led is off, ping fails)
- GIO51: ??? maybe sensor reset? need more testings.
- GIO92: immediately reboot after setting 1!


*Update 20.12.2014*: Add intmux/evtmux/vdac values.

[sprufg5a]: http://www.ti.com/lit/ug/sprufg5a/sprufg5a.pdf
[dtools]: https://github.com/CamWRT/dm36x_debug_tools
