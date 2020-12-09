# Brushless motor position control

This is a beginners guide to motor position control using the [ODrive motor controller](https://github.com/madcowswe/ODrive) meant to be a great first-read.

ODrive has its own great documentation page and getting started guide, but I found it easy to get lost in the plethora of information as a beginner.

Based from my experiences and struggles, I was inspired to create a guide meant for users who have zero experience with ODrive and encoders in general. The guide goes into detail on how set-up the ODrive board, how to interface it with an encoder through ABI, and how to tune the control loop.

## Table of contents

* [Prerequisites](#Prerequisites)
* [Recommended resources](#Recommended-resources)
* [Materials](#Materials)
* [Setting up the testbed (optional)](#Setting-up-the-testbed-(optional))
* [Wiring the encoder using ABI](#Wiring-the-encoder-using-ABI)
* [Preparing the motor](#Preparing-the-motor)
* [Which operating system to use](#Which-operating-system-to-use)
* [Updating the ODrive firmware](#Updating-the-ODrive-firmware)
* [Configuring the ODrive](#Configuring-the-ODrive)
* [Tuning](#Tuning)
* [What's next](#What's-next)

## Prerequisites

No prerequisite knowledge is assumed. The guide will walk through every single detail needed to set-up positional control for a brushless motor. However, as it is a beginners guide, it is not comprehensive, so I highly encourage taking a look at the recommended resources to gain more familiarity with how everything works.

## Recommended resources

[ODrive Getting Started](https://docs.odriverobotics.com) - This guide is only meant to be an introductory guide and is not meant to replace the full, comprehensive ODrive documentation. This getting started page from ODrive, as well as other pages in the docs, is the best resource for the ODrive user.

It is a good idea to also look over the datasheet of your encoder. Shown below are resources for the AS5047 encoder:

[AS5047 Eval Board Manual](https://media.digikey.com/pdf/Data%20Sheets/Austriamicrosystems%20PDFs/AS5047P-TS_EK_AB.pdf) - AS5047P evaluation board manual. Great for getting familiar with its usage.

[AS5x47y Interfaces](https://ams.com/documents/20143/36005/AS5x47y_AN000298_1-00.pdf/38a3a86b-6ee1-3843-f410-f79e1479f49b) - Documentation on the communication interfaces of the AS5x47 encoder family. Good for understanding how to connect the AS5047 encoder with the ODrive.

## Materials

* ODrive motor controller
  * Comes with a 50W power resistor. This guide will assume the 24V variant.
* Encoder
  * In this guide, any ABI capable encoder would do. The ODrive does provide support for other interfaces, but we would use ABI here. The one used is the ASM AS5047P magnetic rotary sensor evaluation board.
* Brushless motor
  * Any 12-24V BLDC motor would work as long as it does not exceed the ODrive peak current capability. For this guide, a small motor would be recommended. The one used is a Sunnysky V3508 580KV.
* Power supply
  * Any 12-24V DC power supply or lipo battery that can supply the current needed for the motor.
* Soldering iron
* Male to female jumper cables
* Epoxy
  * Any adhesive to attach the magnet to the motor's shaft would do.

For the optional testbed frame:

* 3D printer
* M2.5 screws and nuts

## Setting up the testbed (optional)

To make it easier to use the ODrive while testing it with the motor, I designed a 3D printed frame to hold the ODrive and the power resistor, as well as have space to mount the motor and encoder. The STL file for the design is available in the *designs* folder.

![CAD-design](https://raw.githubusercontent.com/aarondls/motor-position-control/main/Images/design_transparent.png)

The frame was printed at 20% infill. Adjust this infill, the screw hole sizes, and the motor mounting holes to match your own set-up. The motor mount is specifically designed for the Sunnysky V3508 brushless motor. The actual Fusion 360 file is available to edit the design.

The ODrive attaches via the included standoffs in the kit and regular M3 nuts. The included power resistor also attaches to the frame using regular M3 screws.

To accomodate your own motor, design the motor mount, and simply adjust the holes in the testbed frame to match the screw holes on your mount. Note when designing the motor mount that the distance of the magnet from the encoder is specified in the datasheet.

### Renderings

![Render-transparent](https://raw.githubusercontent.com/aarondls/motor-position-control/main/Images/render_transparent.png)

### Actual print

![actual-print](https://github.com/aarondls/motor-position-control/blob/main/Images/actual_print.png?raw=true)

## Wiring the encoder using ABI

This portion shows the AS5047P magnetic position sensor, but the instructions generalize for any ABI capable encoder.

For this guide, ABI was chosen over SPI since it is easy to get started with and straightforward to expand to two encoders. Each motor terminal also has its own set of terminals needed for ABI. If you prefer another interface such as SPI, you could look over the ODrive encoder page for guidance on that.

The index signal of the encoder is then used to avoid having to calibrate everytime the ODrive starts up. This is one advantage of absolute position encoders.

To prepare the AS5047P evaluation board, first solder the included header pins. Then, short the correct pins for either 5V or 3.3V usage. Note which one you selected, as this would be important later on when connecting to the ODrive.

I shorted mine by soldering the pads together to keep the form factor thin, but the encoder also comes with a jumper device that plugs onto the presoldered header.

Now, we can examine the pins on the AS5047 evaluation board. This portion of the datasheet nicely summarizes everything we need to know:

![Datasheet P6](https://raw.githubusercontent.com/aarondls/motor-position-control/main/Images/datasheet_p6.png)

This portion of the datsheet nicely shows what we need for ABI (and other interfaces like SPI should you prefer those):

![AS5047P Block Diagram](https://raw.githubusercontent.com/aarondls/motor-position-control/main/Images/AS5047P_block_diagram.png)

On the ODrive board, there are pins marked A, B, Z, 5V, and GND for each motor. Since we are using motor 0, we use the pins close to M0. A, B, GND, and 5V (or 3.3, depending on what voltage was selected earlier) all go to the corresponding pins of the same name on the AS5047 board. The Z pin then goes to the I (index) pin of the AS5047 board.

The wiring scheme for ABI, from ODrive --> encoder, would therefore require 5 wires and are:

* A &nbsp; &nbsp; &nbsp; &nbsp;--> &nbsp; &nbsp; A
* B &nbsp; &nbsp; &nbsp; &nbsp;--> &nbsp; &nbsp; B
* Z &nbsp; &nbsp; &nbsp; &nbsp;--> &nbsp; &nbsp; I
* 5V &nbsp;&nbsp; &nbsp; --> &nbsp; &nbsp; 5V
* GND &nbsp; --> &nbsp; &nbsp; GND

## Preparing the motor

The magnet has to be attached to the shaft of the motor, which sounds easier than it actually is. The magnet does not snap on the shaft perfectly down the center and snaps around to other portions of the shaft easily. While positioning the magnet onto the shaft, you would feel the center position. Be careful as a slight nudge in any direction caused the magnet to snap to an off-center position.

I found that using a 2-part epoxy worked well in holding the magnet in place even while it was still curing. Some people use a jig to precisely center the magnet and hold it in place while the adhesive set, but I felt it was not needed as the epoxy was strong enough to hold it and easy enough to eyeball the center.

## Wiring everything together

This diagram from the ODrive documentation provides a great overview of how everything connects together:

![ODrive wiring diagram](https://docs.google.com/drawings/d/e/2PACX-1vTpJziAisrkvV1kTL4vckAJkmJ-BAvTwN1GeZZNCNwpTHv47Cf8bpz-gJqK2Z3un6FCHT4E-rcuUg6c/pub?w=1716&h=1281)

The power supply is wired directly to the terminals on the short side marked DC.

The three cables from the motor go to the three terminals on the long side. Choose either of the three-phase terminals, which are either for motor 0 (M0) or motor 1 (M1). Note which one you chose, as this will be which axis you would be using for commands later on.

We would also need to wire the included power resistor (or your own, if you have calculated a specific value that you need) to the ODrive. If you are using a battery as the power supply, not wiring it wouldn't be a problem since the ODrive would just feed power back to the battery and recharge it. However, problems could arise if a power supply is used. The ODrive documentation details this problem more in-depth.

## Which operating system to use

I highly recommend using a Windows or Linux device. If you prefer macOS, I recommend updating the firmware and tuning using a non-Mac device. After this, everything else can be done on a Mac.

## Updating the ODrive firmware

First, it is a good idea to update the ODrive firmware to the latest one. The board most likely comes shipped with the latest firmware, but it is good to know how to update it in the future.

It might be as easy as following the steps outlined at [ODrive Tool](https://docs.odriverobotics.com/odrivetool#upgrading-firmware-with-a-different-dfu-tool), but a lot of errors could also go your way.

I tried this instructions on both Windows and macOS. The Windows instruction worked smoothly, but the macOS did not, which is why I recommend not using a Mac for this potion.

On the Mac, running the command odrivetool dfu should have done it, but for me it got stuck at putting the device into DFU mode.

```shell
$ odrivetool dfu
Putting device into DFU mode...
```

Switching the DIP switch into DFU mode on the board makes no difference either.

Since the python DFU tool doesn't work, we can instead convert the binary file through the ARM development tools, and flash the firmware using dfu-util.

Note that previously we could install gcc-arm-embedded using

```bash
brew cask install gcc-arm-embedded
```

but this no longer works after ARM depreciated the use of PPA.

To get around this fact that gcc-arm-embedded is no longer on homebrew, we can instead tap into osx-cross/arm and install from there.

```bash
brew tap osx-cross/arm
brew install arm-gcc-bin
```

At the moment I did this, macOS Big Sur still has issues, and you may run into CLT not supported errors. To get around this, simply download and update the Command Line Tools for Xcode at [Apple's website](https://developer.apple.com/download/more/).

Finally, after all this, we can finally follow the second step on the macOS portion of the [ODrive Tool page](https://docs.odriverobotics.com/odrivetool#upgrading-firmware-with-a-different-dfu-tool), replacing the filenames with the correct ones when needed.

## Configuring the ODrive

Now, follow the ODrive getting started page starting on the "Downloading and Installing Tools" portion for your operating system. It does a great job of explaining everything and I found it to be extremely helpful and beginner friendly, so there is no need to rewrite another guide for that portion.

Some useful things to note while doing the getting started guide:

* For the Sunnysky V3508 motors used
  * I set the maximum calibration current to 5A, but you should select any current you are comfortable with (the default is 10A).
  * I left vel_limit to its default value.
  * The motor has 14 magnets in the rotor, so pole_pairs should be set to 7. An easy way to find this is to simply count the number of magnets (if visible) of the motor and divide it by 2.
  * The motor is rated at 580KV, so torque_constant should be set to 0.01425862069. With earlier firmware versions, you might get the error "attribute not found", which is why we updated it.
  * The motor is a hobby brushless motor, so set motor_mode to MOTOR_TYPE_HIGH_CURRENT.
* For the AS5047 encoder used
  * Count per revolution is 4000, so set cpr to 4000. Note that while the data sheet says the default is 4096, I observed the actual value to be different after much testing and frustration. Other users on the ODrive forum seems to have the same problem too.

After finishing the getting started guide, everything should be set-up and ready for tuning.

## Tuning

For this part, I highly recommend using Windows or Linux over macOS, since the liveplotter tool currenly crashes on macOS. Without it, it is extremely difficult, if not impossible, to properly tune the controller.

The tuning guide on the ODrive docs is quite unfriendly to the first time user. However, it is still worth a read before following this guide to see where the overall procedure came from.

### Preparation for tuning

Before starting the tuning process, make sure that the axis state is in closed loop control mode.

```bash
<axis>.requested_state = AXIS_STATE_CLOSED_LOOP_CONTROL
```

Note that *\<axis\>* is replaced with the axis number you chose earlier (0 for motor 0 and 1 for motor 1). The odrive number is 0 for the first odrive plugged. For example, when setting up M0 and for a single ODrive board plugged in:

```bash
odrv0.axis0.requested_state = AXIS_STATE_CLOSED_LOOP_CONTROL
```

To return to the idle state at any point, we can set the state to *AXIS_STATE_IDLE*.

```bash
<axis>.requested_state = AXIS_STATE_IDLE
```

The liveplotter tool is used to dial in the gain values. To graph the position setpoint against the measured position:

```bash
start_liveplotter(lambda:[odrv0.axis0.encoder.pos_estimate, odrv0.axis0.controller.pos_setpoint])
```

You can read more about the liveplotter tool [here](https://docs.odriverobotics.com/odrivetool#liveplotter).

Finally, when anything goes wrong, the state goes back to idle or undefined and the whole thing stops. Follow the procedure as described in the getting started guide, which is redescribed here for convenience:

Read the errors by

```bash
dump_errors(odrv0)
```

then figure out what is causing it from the [error code documentation](https://docs.odriverobotics.com/troubleshooting.html#error-codes). 

After addressing the issue, clear the error slate.

```bash
dump_errors(odrv0, True)
```

### The tuning process

Now, begin the tuning process by setting all the gain values to zero.

```bash
<axis>.controller.config.pos_gain = 0
<axis>.controller.config.vel_gain = 0
<axis>.controller.config.vel_integrator_gain = 0
```

Increase *vel_gain* by around 30% per iteration until the motor exhibits some vibration. The vibrations are noticable and make a distinct reverberating noise.

We can also easily see through liveplotter than some vibration is occuring:

![Vibrations](https://raw.githubusercontent.com/aarondls/motor-position-control/main/Images/vibrations.png)

At this point, there is no need yet to send input_pos commands. Simply disturbing the motor slighly with your fingers would induce vibrations if *vel_gain* is too high, or the motor would vibrate even without any disturbance.

Sometimes, the motor spins too quickly and causes the whole thing to stop. Read and clear the errors as described above, then increase *vel_limit*.

At the point where the motor exhibits some vibration, decrease *vel_gain* to half of its vibrating value.

Now, increase *pos_gain* by 30% per iteration until you see overshoot.

To see if the controller overshoots, turn the motor by hand and let go, then view the liveplotter graph.

Alternatively, you could send input position commands using

```bash
<axis>.controller.input_pos = position
```

where *position* could be 1 turn, 1.5 turns etc.

and view the liveplotter graph.

For example, setting *input_pos* equal to 2 generates this graph on liveplotter:

![Undershoot](https://raw.githubusercontent.com/aarondls/motor-position-control/main/Images/undershoot.png)

It is clear in this case that undershoot is present, so we need to increase *pos_gain* until overshoot occurs. Increasing *pos_gain* by 30% per iteration yields this graph in liveplotter, where we see overshoot starts to occur.

![Overshoot](https://raw.githubusercontent.com/aarondls/motor-position-control/main/Images/overshoot.png)

At the point where overshoot occurs, back down *pos_gain* until the overshoot disappears.

Getting rid of the overshoot and sending another *input_pos* command generates this graph on liveplotter:

![No overshoot](https://raw.githubusercontent.com/aarondls/motor-position-control/main/Images/no_overshoot.png)

Now, we can set *vel_integrator_gain* using the formula 0.5 * bandwidth * *vel_gain*.

From the ODrive docs

> Bandwidth is the overall resulting tracking bandwidth of your system. Say your tuning made it track commands with a settling time of 100ms (the time from when the setpoint changes to when the system arrives at the new setpoint); this means the bandwidth was 1/(100ms) = 1/(0.1s) = 10hz. In this case you should set the vel_integrator_gain = 0.5 * 10 * vel_gain.

Thus, to find the bandwidth value, set a new *input_pos* and view the time it takes to settle into the set position using the liveplotter graph.

After setting *vel_integrator_gain*, disturbing the control loop by manually turning the motor yields this liveplotter graph:

![Disturbing tuned control loop](https://raw.githubusercontent.com/aarondls/motor-position-control/main/Images/disturbing_tuned.png)

After this, congratulations! The control loop should now be tuned.

## What's next

At this point, everything should be set-up and ready to go.

I recommend looking over the [ODrive documentation](https://docs.odriverobotics.com) for a comprehensive tour of its capabilities.

Have fun working on your projects!
