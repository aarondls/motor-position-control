# Brushless motor position control

This is a beginners guide to motor position control using the [ODrive motor controller](https://github.com/madcowswe/ODrive) and the [AMS AS5047](https://ams.com/as5047p) magnetic encoder.

ODrive has its own great documentation page and getting started guide, but I found it easy to get lost in the plethora of information as a beginner. The AS5047 rotary position sensor is also a popular choice for motor encoders, and I found guides specifically on its usage with ODrive to be unfriendly to beginners.

Based from my experiences and struggles, I was inspired to create a guide meant for users who have zero experience with ODrive and encoders in general. The guide also goes into detail on how to interface the AS5047 encoder and ODrive, a popular set-up I imagine most beginners would follow.

## Prerequisites

No prerequisite knowledge is assumed. The guide will walk through every single detail needed to set-up positional control for a brushless motor. However, as it is a beginners guide, it is not comprehensive, so I highly encourage taking a look at the recommended resources to gain more familiarity with how everything works.

## Recommended resources

[ODrive Getting Started](https://docs.odriverobotics.com) - The getting started page from ODrive. One of the best resources for the first-time ODrive user. A must-read.

[AS5047 Eval Board Manual](https://media.digikey.com/pdf/Data%20Sheets/Austriamicrosystems%20PDFs/AS5047P-TS_EK_AB.pdf) - AS5047P evaluation board manual. Great for getting familiar with its usage.

[AS5x47y Interfaces](https://ams.com/documents/20143/36005/AS5x47y_AN000298_1-00.pdf/38a3a86b-6ee1-3843-f410-f79e1479f49b) - Documentation on the communication interfaces of the AS5x47 encoder family. Good for understanding how to connect the AS5047 encoder with the ODrive.

## Materials

* ODrive motor controller
  * Comes with a 50W power resistor. This guide will assume the 24V variant.
* ASM AS5047P magnetic rotary sensor evaluation board
* Brushless motor
  * Any 12-24V as long as it does not exceed the ODrive peak current capaility. For this guide, a small motor would be recommended. The one used is a Sunnysky V3508.
* Power supply
  * Any 12-24V DC power supply or lipo battery that can supply the current needed for the motor.
* Soldering iron
* Female to female ribbon cables
* Epoxy
  * Any adhesive to attach the magnet to the motor's shaft would do.

For the optional testbed frame:

* 3D printer
* M2.5 screws and heatset inserts


## Setting up the testbed (optional)

To make it easier to use the ODrive while testing it with the motor, I designed a 3D printed frame to hold the ODrive and the power resistor, as well as have space to mount the motor and encoder. The STL file for the design is available in the *designs* folder.

The frame was printed at 20% infill. Adjust this infill, the screw hole sizes, and the motor mounting holes to match your own set-up. The motor mount is specifically designed for the Sunnysky V3508 brushless motor. The actual Fusion 360 file is available to edit the design.

The ODrive attaches via the included standoffs in the kit and regular M3 nuts. The included power resistor also attaches to the frame using regular M3 screws.

To accomodate your own motor, design the motor mount, and simply adjust the holes in the testbed frame to match the screw holes on your mount. Note when designing the motor mount that the distance of the magnet from the encoder is specified in the datasheet.

## Wiring the encoder

The AS5047 evaluation board package makes it easy to work with the encoder, avoiding having to design your own PCB.

To prepare the encoder, first solder the included header pins. Then, short the correct pins for either 5V or 3.3V usage. Note which one you selected, as this would be important later on when connecting to the ODrive.

I shorted mine by soldering the pads together to keep the form factor thin, but the encoder also comes with a jumper device that plugs onto the presoldered header.

Now, we can examine the pins on the AS5047 evaluation board. This portion of the datasheet nicely summarizes everything we need to know:

![Datasheet P6](https://raw.githubusercontent.com/aarondls/motor-position-control/main/Images/datasheet_p6.png)

For this guide, we would be using SPI to interface with the ODrive. This allows us to avoid having to calibrate everytime the ODrive and encoder starts up.

For a single device application, we have the option for a 4-wire mode, which requires the MOSI, MISO, SCK, and CSn pins. This section of the AS5x47y Interfaces document sums it all up:

![Interface_document_2.2](https://raw.githubusercontent.com/aarondls/motor-position-control/main/Images/Interfaces_2.2.png)

On the ODrive board, there are pins marked SCK, MISO, MOSI, 5V, and GND. MISO, MOSI, and 5V (or 3.3, depending on what voltage was selected earlier) goes to the corresponding pins of the same name on the AS5047 board. SCK goes to CLK (for those wondering, SCK stands for Serial CLocK).

We can choose any GPIO pin to connect to CSn, but keep in mind that pins 1 and 2 are usually used by UART. I went with pin 4.

The connections for SPI, from ODrive -> encoder, are therefore:

    SCK    -->  CLK
    MISO   -->  MISO
    MOSI   -->  MOSI
    5V     -->  5V
    GND    -->  GND
    GPIO 4 -->  CSn

## Preparing the motor

The magnet has to be attached to the shaft of the motor, which sounds easier than it actually is. The magnet does not snap on the shaft perfectly down the center and  snaps around to other portions of the shaft easily. While positioning the magnet onto the shaft, you would feel the center position. Be careful as a slight nudge in any direction caused the magnet to snap to an off-center position.

I found that using a 2-part epoxy worked well in holding the magnet in place even while it was still curing. Some people used a jig to precisely center the magnet and hold it in place while the adhesive set, but I did not need to use one since the epoxy was strong enough to hold it.

## Wiring everything together

The power supply is wired directly to the temrinals on the short side marked DC.

The three cables from the motor go to the three terminals on the long side. Choose either of the three-phase terminals, which are either for motor 1 or motor 2.

We would also need to wire the included power resistor (or your own, if you have calculated a specific value that you need) to the ODrive. If you are using a battery as the power supply, not wiring it wouldn't be a problem since the ODrive would just feed power back to the battery and recharge it. Howveer, problems could arise if a power supply is used. The ODrive documentation details this problem more in-depth.
