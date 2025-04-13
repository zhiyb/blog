---
title: AS7262 Spectrometer
date: 2024-08-26 11:07:12
tags:
---

Some notes on this sensor: \
[AS7262 6-channel Spectral Sensor (Spectrometer) Breakout from Pimoroni](https://shop.pimoroni.com/products/as7262-6-channel-spectral-sensor-spectrometer-breakout)

<!--more-->

## Initialisation

AS7262 seems to require ~1s initialisation time after being powered-on or soft reset.

If another I2C transaction started before 1s wait, it seems the sensor's I2C may become stuck, I2C status register keeps returning 0x00 on read.

Same 1s delay must be applied after setting the soft reset bit in I2C control register.

## I2C synchronisation

AS7262's I2C bus seems rather weird with the virtual-register business, I suspect this is all implemented by firmware.

If controller has been reset while AS7262 sensor was in an I2C transaction, we'll need to synchronise the I2C / virtual register bus states.

Here is an idea:

1. Check status register
2. If read register has valid data, read it
3. If write register is free, write control register address 0x04
4. Repeat 1-3 until read register has been read after written control register address

## Firmware update

Firmware update is only possible via AT commands on the serial (UART) bus.

It is not accessible on the Pimoroni breakout board.
