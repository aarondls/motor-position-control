# Switching to SPI

Also includes saving motor and encoder calibration to avoid calibration at every start-up

## SPI Wiring

For multiple encoders communicating through SPI, refer to multiple slave portion of the [AS5047 datasheet](https://ams.com/documents/20143/36005/AS5x47y_AN000298_1-00.pdf/38a3a86b-6ee1-3843-f410-f79e1479f49b)

Shared, ODrive to encoder:

* MOSI -> MOSI
* MISO -> MISO
* SCK  -> SCK

Separated, ODrive to encoder:

* GPIO pin (preferably not 0 or 1) -> CSn1
* GPIO pin (preferably not 0 or 1) -> CSn2

## Calibration and Saving

Start with motor calibration. Refer to [ODrive AXIS_STATE page](https://docs.odriverobotics.com/api/odrive.axis.axisstate):

Set motor.config.pre_calibrated to False:
```
<odrv>.<axis>.motor.config.pre_calibrated  = False
```

Then, calibrate motor:
```
<odrv>.<axis>.requested_state = AXIS_STATE_MOTOR_CALIBRATION
```

Then, set motor.config.pre_calibrated to True:
```
<odrv>.<axis>.motor.config.pre_calibrated = True
```

Then, save configuration and reboot:
```
<odrv>.save_configuration()
<odrv>.reboot();
```
 
// try ABI for now

Follow "Encoder with index signal" part.

// Note might want to manually search for index at start-up

Then prepare for the encoder offset calibration. Refer to [ODrive encoder page](https://docs.odriverobotics.com/encoders#encoder-without-index-signal). Make sure "Startup sequence notes" part is good, then go to "SPI Encoders" part. 

First, set:

```
<odrv>.<axis>.encoder.config.abs_spi_cs_gpio_pin = 4  # or which ever GPIO pin you choose
```

Now since we are using an AMS encoder:

```
<odrv>.<axis>.encoder.config.mode = ENCODER_MODE_SPI_ABS_AMS
<odrv>.<axis>.encoder.config.cpr = 2**14
// note this might have to be set to 4000 for AS5047P
```

Now save and reboot:
```
<odrv>.save_configuration()
<odrv>.reboot()
```

Then, set encoder.config.pre_calibrated to false:
```
<odrv>.<axis>.encoder.config.pre_calibrated = False
```

Now, run the offset calibration:
```
<odrv>.<axis>.requested_state = AXIS_STATE_ENCODER_OFFSET_CALIBRATION 
```

Verify everything went well by checking the following variables:


* <axis>.error should be 0.
* <axis>.encoder.config.offset - This should print a number, like -326 or 1364.
// change to encoder.config.phase_offset
* <axis>.encoder.config.direction - This should print 1 or -1.

Then, set encoder.config.pre_calibrated to true:
```
<odrv>.<axis>.encoder.config.pre_calibrated = False
```


Finally, save and reboot:
```
<odrv>.save_configuration()
<odrv>.reboot()
```


Every time the motor cables are changed, go through the calibration process.