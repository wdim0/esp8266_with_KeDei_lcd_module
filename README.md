# ESP8266 with 3.5" KeDei LCD module

This repository describes an easy hack (depending on your DIY skills) of cheap KeDei LCD 3.5" module (320x480 pixels), originally sold as a display option for Raspberry Pi, to simulate more standard LCD connection (seen in LCD modules more often) using lines: /CS, D/C, SCLK, MOSI, (MISO), sometimes called as "4-wire 8-bit data serial interface II" or "4-line serial interface".<br>
After the hack, the module can be used with my WLCD driver for superfast drawing possibility (superfast in ESP8266 terms).

Why? Because KeDei is cheap, nice, but it's not optimized for speed at all. At least for ESP8266 and it's HSPI interface and it's long transactions (max. 512 bits / 512 CLK long). Why they just didn't set the LCD controller into serial interface mode? Surely the Raspberry Pi could also manage that .. or not?

So basically, if you want to connect KeDei module to ESP8266, you have 3 options:
- use SW bit-banging. Sooo sloooow, useless with 320x480 (153.6 k) pixels
- use HSPI to perform some basic HW SPI transactions, but the transactions have to be short - number of transmitted bits must be exactly as KeDei internal shift register (3 x 74HC565 shift registers = 24 bits). Also not optimal for 320x480 pixels at all
- hack the LCD module to get more standard LCD connection type (4-wire 8-bit data serial interface II) and use entire HSPI buffer. This gives us speed. If you're looking for suitable driver, you can use my WLCD driver for superfast drawing possibility (in ESP8266 terms). And that's what we're doing here ;)

<b>ESP8266 with hacked 3.5" KeDei LCD module (480x320, SPI, CLK 20 MHz) - video</b><br>
[![ESP8266 with hacked 3.5" KeDei LCD module (480x320, SPI, CLK 20 MHz)](http://img.youtube.com/vi/NzYD4sONz20/1.jpg)](http://www.youtube.com/watch?v=NzYD4sONz20)

TODO - image of hacked module with high-speed 74VLS4040 module (just ordered 74VLS4040 ... pending) (max 40 MHz CLK I hope)

Hack with 74HC4040 and without propagation delay compensation (max 13.3 MHz CLK)
![3.5" KeDei LCD module hacked - no propagation delay compensation](https://raw.githubusercontent.com/wdim0/esp8266_with_KeDei_lcd_module/master/module_hacked_no_compensation.jpg)

Hack with 74HC4040 and propagation delay compensation (max 20 MHz CLK).<br>
Use new CLK input, don't connect CLK to original connector.
![3.5" KeDei LCD module hacked - with propagation delay compensation](https://raw.githubusercontent.com/wdim0/esp8266_with_KeDei_lcd_module/master/module_hacked_with_compensation.jpg)
<br>13.3 MHz CLK vs 20 MHz CLK comparison using WLCD demo:<br>
1000 lines in 1833 ms vs 1595 ms => +15% faster<br>
1000 images 50x50 in 3345 ms vs 2345 ms => +42% faster

Limitations:
- The KeDei display should be v4.0 (see image) - with this version of the module I've done the hack and tested everything. Maybe also different versions will work (most probably they should work), but it's not guaranteed
- In WLCD driver we're using "only" 65 k colors (R5G6B5) mode, because this gives us speed. A lot of it! Many things can be optimized then and we can use 32-bit copy instructions. Everything is nicely aligned when using R5G6B5 and ESP8266's HSPI

##The hack

Original 3.5" KeDei LCD module
![Original 3.5" KeDei LCD module](https://raw.githubusercontent.com/wdim0/esp8266_with_KeDei_lcd_module/master/module_original.jpg)

U1 removed (hot air soldering station is your friend), interrupt PCB trace leading signal L_CS to U2 and U3's STCP input
![U1 removed, interrupt PCB trace](https://raw.githubusercontent.com/wdim0/esp8266_with_KeDei_lcd_module/master/module_U1_removed_interrupt_PCB_trace.jpg)

Schema of the hack (not all parts of the display module are drawn (power suppy, touch, connectors, ...). It's not important for the idea of the hack). By the "propagation delay" I mean the delay between the input and output change. For 74HC4040's CP input and Q2 output, it's someting around 40 ns (according to datasheet).
![Schema of the hack](https://raw.githubusercontent.com/wdim0/esp8266_with_KeDei_lcd_module/master/schema.jpg)

Timing is becomming important for 74HC... family when speed exceeds tenths of MHz ...<br>
This is an explanation, why we need propagation delay compensation when we want to achieve higher speeds.
![Timig](https://raw.githubusercontent.com/wdim0/esp8266_with_KeDei_lcd_module/master/timing.jpg)

## Thanks

This hack would never be possible without the great starting point created by saper-2:<br>
https://github.com/saper-2/rpi-spi-lcd35-kedei<br>
https://www.raspberrypi.org/forums/viewtopic.php?f=44&t=124961

Thanks!
