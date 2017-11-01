# orange-pi-pwm
pwm control for Orange Pi

This code is based on code published on www.orangepi.org/orangepibbsen/forum.php?mod=viewthread&tid=1153 by user snowman with text *You can play with it by modifying the code*

Conversation from thread is the following

```
I was looking into hardware pwm and thought I could use the sunxi_pwm module, 
I had a look in the H3 manual, and I think only port PA5 is usable (serial boot connector middle pin)
I had no luck executing the sunxi_pwm module (filesystem with PWM0 is not created).
The fex file does not set the port correctly to pwm nor the pwm register.
WiringOP is not configured correctly for PWM (yet)

But you can always set the ports manually ;)
(p 317 H3 manual  011 PWM0 dataregister for port PA5)

in the H3 manual P.188 the state that the control register for PWM is at 0x01C21400 (here you can set the frequency with prescalers for 24Mhz)
At an offset of 0x04 you can find the control register for the period (duty cycle --upper 16bit is number of cycles -- lower 16bit is number of active cycles -- upper has to be greater or equal)
in the manual they only refer to channel 0...

I attached an example with comment, which blinks a LED attached to port PA05. You can play with it by modifying the code (see comment)

I've check your code and PWM0 is working. I also tried to run PWM1 (I've changed information about register in code to PA6) but it didn't worked. So only one hadrware PWM is available on OP?


Yes. You can read about this in  (H3 datasheet v1.1](http://linux-sunxi.org/H3)
SoC H3 has only one port PWM

How do you calculate your desired frequency?

You said that comment in your code, 

        //data = (0x6EEE00EE);     //control register 987 hz (no prescaler)
        //  data = (0x000E0006);   //control register 1.6 Mhz max
        //   data = (0x00020001);  //control register 8 Mhz max

Your desired frequency is 987 hz and you calculate data value (0x6EEE00EE)

How do you calculate it?

Some findings to pass on:

WiringOP does not appear to have working PWM support at this time. I tested both the gpio app and the lib (via the pwm example program) and neither work due to missing H3 pin/mode configuration.

On my Orange PI PC, the GPIO PA5 PWM0 is accessible via the middle pin of the UART header. I could not get PWM0 or PWM1 working via the 40-pin header.

The Orange Pi header documentation and the OPI-PC schematics indicate that header pin 7 is H3 GPIO PA6 with PWM1 available on MUX 3, but the H3 datasheet v1.2 does not show PWM1 available on GPIO PA6, and setting PA6 to func 3 did not result in a PWM signal on pin 7.

The PWM0 function only appears to be available on the UART header (GPIO PA5: PWM0). The H3 datasheet shows the PWM function is also available on PL10, but the OPI-PC uses that pin for the power LED.


As to performance, I am evaluating the Orange Pi boards vs the Raspberry Pi 3 for GPIO at 1MHz+ speeds.

Here are my findings using C/assembly:

RPI3
===
~7.5ns : Set or clear up to 28 GPIO pins on 40-pin header. Setting some pins & clearing others takes 2x GPIO writes.
~52ns : Read the level of 28 GPIO pins on 40-pin header.

The RPI3 can bit-bang output to the GPIO at about 66MHz (using 2x GPIO writes to set and clear lines).
Reading is considerably slower at 19.2MHz.



Orange Pi PC (Armbian 5.14 Jessie 3.4.112 desktop)
========
~50ns: Set the level of all GPIO pins <on one port>.
~120ns: Read the level of all GPIO pins <on one port>.

The OPI-PC can bit-bang output to the GPIO at about 20MHz.
Reading is considerably slower at ~8MHz.



Because the OPI-PC uses GPIO from H3 ports A,C,D,G on the 40-pin header, it is very slow to read (4x 120ns) or write (4x 50ns) all 28 GPIO pins on the 40-pin header, and it is not possible to change them simultaneously.

```
