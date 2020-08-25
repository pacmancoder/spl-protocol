SPL protocol
==========================

This document describes specification of the Open Source single-wire data transmissing protocol for
constrained devices.

### Table of contents
* [Overview](#overview)
* [Hardware layer](#hardware-layer)  
    * [Short length bus](#short-length-bus)
    * [Long length bus](#long-length-bus)
    * [Controlling bus signal](#controlling-bus-signal)
        * [MCU open-drain](#mcu-open-drain)
        * [MCU general-purpose pins](#mcu-general-purpose-pins)
        * [External driver](#external-driver)
* [Protocol specification](#protocol-specification)
    * [SPL frames](#spl-frames)


### Overview
SPL (Originally named after **S**tock**P**i**L**er project) protocol is an open source alternative
for proprietary single-wire interfaces for applications where speed is not a huge priority.

This protocol features the following:
- Simple to implement
- Low device requirements (Easily runs even on Padauk™ PMS150C)
- Simple hardware layer (single pull-up resistor)
- Deterministic frame size (10 bytes)
- Simple checksum
- Big device id address space (2^31 addresses)
- Broadcast address
- Pre-defined commands for _ping_ and _sleep_
- Optional acknowledge
- Medium distance (e.g. 3 meters of single ribbon cable wire for 5V CMOS levels should be fine.
  For more long lines more high logic levels may be required to withstand the noise and the voltage
  drop in the wire)

### Hardware layer
Hardware layer is very simple: Single wire + pull-up resistor (For short lines even normal)

#### Short length bus
For application with bus length up to ~50cm single 5k-10k resistor should be sufficient.

For example, we can connect multiple simple devices on the single bus using 3 wires:
* +5V
* GND
* SPL bus signal line

![Short bus schematics](img/hw_bus_1m.png)

#### Long length bus
If we are talking about 3-4 meter bus and a few dozens of the devices while using 5V CMOS logic,
There are a number of additional considerations that need to be taken into account when long bus
with a few dozens of the devices is required and we have only 5V logic levels at our disposal:

* Because of the potential huge voltage drop in the wires and the signal reflections inside the bus,
  lower pull-down resistors value should be selected. Multiple pull-down resistors may be used, evenly
  distributed throughout all bus length.

![Long bus schematics](img/hw_bus_long.png)

#### Controlling bus signal
To only read the bus data, normal input mode can be used on the most MCU's.

However, writing data is more complicated. But do not fear! It is not SO complicated!

##### MCU open-drain
Ideally, If your MCU has open-drain pin (Like PA5 on Padauk PMS150C), it can be used to drive SPL
bus without and hassle, just by toggling MCU pin from 1 to 0 in output mode 

![Mcu native open-drain schematics](img/hw_bus_mcu.png)

##### MCU general-purpose pins
More dangerous way of controlling the bus (but still, very simple) is to use GPIO pin only in two
modes:
* Input Hi-Z
* Output Low

**CAUTION:** If one device on the bus will set its SPL pin to Output High, while some other device
pull its SPL pin low, it **WILL** cause immediate short circuit condition and damage your devices
on the bus. Please use this mode VERY carefully.

![Mcu native open-drain schematics](img/hw_bus_mcu_gpio.png)

For example, for Arduino, it will translate to something like this:

```c++
void setup() {
  // Pin should be INPUT by default, but let's show that here explicitly anyway
  pinMode(SPL_BUS_PIN, INPUT);
  // Make sure that even when pin mode will be changed to OUTPUT, it will be pulled
  // to ground (sink current)
  digitalWrite(SPL_BUS_PIN, LOW); 
}

// Let's wrap pin control code into separate functions to minimize mistakes and improve
// readability

void spl_pull_down() {
  pinMode(SPL_BUS_PIN, OUTPUT);
}

void spl_pull_up() {
  pinMode(SPL_BUS_PIN, INPUT);
}

// Please note that digitalWrite is only used once, in the setup() function!
// (!!!) This will potentially produce a short circuit: digitalWrite(SPL_BUS_PIN, HIGH); 
```
##### External driver
If it is not possible to guarantee that selected GPIO pin never will be set to active high mode, or
selected pull-up resistors are sourcing more current than GPIO can candle in active log (sink) mode,
then the following two-pin configuration with external N-Channel MOSFET can be used:
![External driver schematics](img/hw_bus_mcu_external_driver.png)

Of course, in this case SPL output should be controlled with the inverted signal

### Protocol specification
This chapter describes actual protocol implementation details
#### SPL Frames
TODO