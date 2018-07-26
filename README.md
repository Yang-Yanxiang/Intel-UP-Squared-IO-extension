# Extend Intel UP squared board GIO pin by using MCP23017
![connection](https://github.com/Yang-Yanxiang/Intel-UP-Squared-IO-extension/blob/master/connection.jpg)
Picture is from Adafruit.
![connection](https://github.com/Yang-Yanxiang/Intel-UP-Squared-IO-extension/blob/master/pinout.png)
* The yellow line is SDA(pin27)
* The green line is SCL(pin28)
* The three black lines on top are the address pins
* The brown pin is RESET which must be pulled high for normal operation(connect using 5k resistor to 5v)
* Red is 5v
* Black is GND.

### Testing the hardware

i2c_designware.0 -> I2C channel 0 on hat (ID_SD ID_SCL)
```
ls /sys/bus/pci/devices/*/i2c_designware.0/ | grep i2c
i2c-4
```
i2c_designware.1 -> I2C channel 0 on hat (pin 3,5 on HAT)
```
ls /sys/bus/pci/devices/*/i2c_designware.1/ | grep i2c
i2c-5
```
So the linux device node for the first i2c channel is /dev/i2c-4
To detect all the peripherals on the first i2c bus do the following
```
$ sudo apt install i2c-tools
$ sudo i2cdetect -y -r 4
    0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: 50 -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
60: -- 61 -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
70: -- -- -- -- -- -- -- --
```

### Using command tool
i2cset is a small helper program to set registers visible through the I2C bus.
```
i2cset [-y] i2cbus chip-address data-address value [mode] [mask]
i2cset -V
```
OPTIONS

-V Display the version and exit.

-y Disable interactive mode. By default, i2cset will wait for a confirmation from the user before messing with the I2C bus. When this flag is used, it will perform the operation directly. This is mainly meant to be used in scripts.

There are four required options to i2cset. i2cbus indicates the number of the I2C bus to be scanned. This number should correspond to one of the busses listed by i2cdetect -l. chip-address specifies the address of the chip on that bus, and is an integer between 0x00 and 0x7F. data-address specifies the address on that chip to write to, and is an integer between 0x00 and 0xFF. value is the value to write to that location on the chip.

The mode parameter, if specified, is one of the letters b or w, corresponding to a write size of a single byte or a 16-bit word, respectively. If the mode parameter is omitted, i2cset defaults to byte mode. The value provided must be within range for the specified data type (0x00-0xFF for bytes, 0x0000-0xFFFF for words).

The mask parameter, if specified, describes which bits of value will be actually written to data-address. Bits set to 1 in the mask are taken from value, while bits set to 0 will be read from data-address and thus preserved by the operation.

Here is the memory map for the mcp23017
![MCP23017memory_map](https://github.com/Yang-Yanxiang/Intel-UP-Squared-IO-extension/blob/master/mcp23017_mm.png)

First we configure Port A pins GPA0-7 as outputs. Remember 0x20 is the I2C address of the mcp23017, in the table above you can see that 0x00 is IODIRA and sending 0x00 sets all of the pins to be outputs. If for example you wanted the first 7 pins to be outputs and pin 8 to be an input then the last value would be 0x80.
```
\\ let GPA0 be output
sudo i2cset -y 1 0x20 0x00 0x00
\\ set GPA0 to a logic high
sudo i2cset -y 1 0x20 0x14 0x01
\\ set GPA0 to be low
sudo i2cset -y 1 0x20 0x14 0x00
```
### Using node.js
blink LED on GPA0
```
"use strict";

const mraa = require('mraa');

let i2cDevice = new mraa.I2C(1);

i2cDevice.address(0x20);
i2cDevice.writeReg(0x00, 0x00);

setInterval(()=>{
    i2cDevice.address(0x20);
    i2cDevice.writeReg(0x14, 0x00);
    setTimeout( () => {
        i2cDevice.address(0x20);
        i2cDevice.writeReg(0x14, 0x01);
    }
}, 2000)
```
