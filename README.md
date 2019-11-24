# I2C-7311
## License: CERN Open Hardware Licence v1.2

IO Expander with 4x IO4 connections

### Introduction

This board was originally produced to help me build control panels for
my model railroad - I needed to connect many different buttons, switches
and lights to an Arduino, and also connect the Arduino to a
communications bus (Ethernet, CAN or Loconet), and I couldn\'t find
anything that had both high IO point density AND visual feedback.

I started with a simple I2C based 32-pin Arduino
[IOShield](IOShield "wikilink") with monitoring LEDs on each pin
(Active-LOW inputs, LED is on when pin is grounded, writing a \"1\" to
the port pulls it to ground). I was aiming to support up to the max I2C
devices that can be used together, with the 8-bit board
[I2C-8574-IO](I2C-8574-IO "wikilink") (deprecated), that was a total of
8x boards with 128 i/o points.

The 7311 IO expanders used here will support up to 64 boards, with 1024
potential I/O points.

This latest design moves from overly generic and/or overly specialized
designs to a generic design with daughterboards that encapsulate the
appliance-specific buffering and logic. The benefits provided by these
IOB boards include:

-   Improved I/O bank utilization (leftover capacity is now a multiple
    of 4-bit daughtercard instead of 16-bit I2C card\...)
-   Riser height eliminates having to custom cut 3M track
-   Allows deprecation of cabling intensive signal driver IO4 and I2C
    boards
-   Allows custom circuitry to be added to Signal, Detector and Turtle
    drivers

### Cautions

With several hundred IO loads, and up to 256 external breakout devices,
a suitable power supply (or supplies) must be used.

In practice, a 3A 9-12VDC supply a good choice for most small to medium
sized control points.

This supply needs to power the core processor, its direct IO loads, the
I2C expander chain AND provide \~100mA to each IO4 device. Instead of
relying on the Arduino\'s onboard regulated supply (which is only good
for about 100mA itself), each board has its own regulator with a common
supply feed that can handle up to 4A @ 12v. If more power is needed, the
daisy chained power feed thru can be replaced with a per-board
independent supply.

### Specifications

This IO board is based on the MAX7311/12 16 bit IO Expander, with a a
bank of 4x I/O Boards. These IOBs contain buffered LED drivers connected
to LEDs that show the status of the I/O lines in real time. They also
contain the needed circuitry to drive / sense the appliances connected
to them. See

-   [IOB-Inputs](IOB-Inputs "wikilink") 4x buffered inputs
-   [IOB-Outputs](IOB-Outputs "wikilink") 4x buffered outputs
-   [IOB-Turtle](IOB-Turtle "wikilink") 1x buffered output and 3x
    buffered inputs
-   [IOB-Signal](IOB-Signal "wikilink") 2x RGB signal head driver (Dark,
    Stop, Approach, Clear)

Each board provides a latched set of 16x IO points, with each point
being software selectable to be either an input or an output. The
individual points are reasonably protected from the environment, and can
sink 10 mA each (with the MAX7312, they can also drive \~20mA). By using
active low inputs and outputs, the effects of environmental noise are
reduced - remote sensors only need to ground an I/O point to register
activity.

#### Communication Protocol {#communication_protocol}

``` {.cpp}
Simple I2C 16-bit Reads and Writes:
uint16_t I2Cexpander::read7311() {
    uint16_t data = 0;
    Wire.beginTransmission(_i2c_address);
    Wire.write(REGISTER_7311INPUT);
    Wire.endTransmission();
    // Wire.beginTransmission(_i2c_address);
    Wire.requestFrom(_i2c_address, (uint8_t)2);
    if(Wire.available()) {
        data = Wire.read();
    }
    if(Wire.available()) {
        data |= (Wire.read() << 8);
    }
    // Wire.endTransmission();
    return data;
}

void I2Cexpander::write7311(uint16_t data) {
    data = data | _config;
    Wire.beginTransmission(_i2c_address);
    Wire.write(REGISTER_7311OUTPUT);
    Wire.write(0xff & data);  //  low byte
    Wire.write(data >> 8);    //  high byte
    Wire.endTransmission();
}
```

See [I2Cexpander](I2Cexpander "wikilink") for a more complete interface
library.
