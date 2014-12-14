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
        MMC/SD0: 00
        GIO49: 01
        GIO48: 01
        GIO47: 01
        GIO46: 01
        GIO45: 01
        GIO44: 01
        GIO43: 00
        C_WE_FIELD: 00
        VD: 00
        HD: 00
        YIN0: 00
        YIN1: 00
        YIN2: 00
        YIN3: 00
        YIN4: 00
        YIN5: 00
        YIN6: 00
        YIN7: 00
PINMUX1: 0x00430000
        VCLK: 01
        EXTCLK: 00
        FIELD: 00
        LCD_OE: 01
        HVSYNC: 01
        COUT0: 00
        COUT1: 00
        COUT2: 00
        COUT3: 00
        COUT4: 00
        COUT5: 00
        COUT6: 00
        COUT7: 00
PINMUX2: 0x00001980
        EM_CLK: 01
        EM_ADV: 01
        EM_WAIT: 00
        EM_WE_OE: 00
        !EM_CE1: 01
        !EM_CE0: 01
        EM_D15_8: 00
        EM_A7: 00
        EM_A3: 00
        EM_AR: 00
PINMUX3: 0x615AFFFF
        GIO26: 00
        GIO25: 03
        GIO24: 00
        GIO23: 00
        GIO22: 00
        GIO21: 02
        GIO20: 02
        GIO19: 01
        GIO18: 01
        GIO17: 01
        GIO16: 01
        GIO15: 01
        GIO14: 01
        GIO13: 01
        GIO12: 01
        GIO11: 01
        GIO10: 01
        GIO9: 01
        GIO8: 01
        GIO7: 01
        GIO6: 01
        GIO5: 01
        GIO4: 01
        GIO3: 01
        GIO2: 01
        GIO1: 01
PINMUX4: 0x0030C000
        GIO42: 00
        GIO41: 00
        GIO40: 00
        GIO39: 00
        GIO38: 00
        GIO37: 03
        GIO36: 00
        GIO35: 00
        GIO34: 03
        GIO33: 00
        GIO32: 00
        GIO31: 00
        GIO30: 00
        GIO29: 00
        GIO28: 00
        GIO27: 00
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


[sprufg5a]: http://www.ti.com/lit/ug/sprufg5a/sprufg5a.pdf
[dtools]: https://github.com/CamWRT/dm36x_debug_tools
