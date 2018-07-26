# Extend Intel UP squared board GIO pin by using MCP23017
![connection](https://github.com/Yang-Yanxiang/Intel-UP-Squared-IO-extension/blob/master/connection.jpg)
Picture is from Adafruit.
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
