# Corevus-G v0.4 hardware overview

Document incomplete (but for the most part ready), work in progress

*See also*: [Configuration and Setup](./CvG-v0.4-config-setup.md)

## Errata
- The USB-A connector labeled 5V PWR, designed to supply power to the RPi, is unusable due to a wiring error. I have shipped a cable harness with each order to bodge this while I push the fix to the next revision.
- The H0 heater turns on in DFU mode because I was an idiot and forgot to check the H723 boot pins when migrating this board from the F407. The H0 connector should be taped off as a precautionary measure.
- I haven't finalized my decision on the pinout for the 8 pin toolhead cable, and it may end up changing to be incompatible with the pinout on this board (such that plugging in the wrong hardware may result in electrical damage). The connector is taped off for now, when I get around to sending out testing units for corevus toolboard and you would like to receive one please contact me so I can send you the correct cable that won't set your machine on fire

## Wiring the 5V power harness

The JST-XH connector should be plugged into the S2 endstop port, as shown in the below image. 
![image1](/assets/images/bodge-harness-1.jpg) 
The pin header side of the harness should be plugged in as shown in the below image, with the red 5V wire over pin 4 and the black gnd wire over pin 6. **Failure to install this correctly may result in destruction of hardware** and a bad user experience. 
![image2](/assets/images/bodge-harness-2.jpg)

## Board hardware

### Dimensions 

Board is 135 × 75 mm, with M3 mounting holes inset 3mm from the corners.

![aeiou](/assets/images/CvG-v0.4-dimensions.png)

### Power inlet

Connect a 24V power supply to the screw terminal pair labeled '24V MAIN / GND'. If you are using a dedicated power supply for motors, connect it to the screw terminal pair labeled '48V MOTOR / GND'. 

![aeiou](/assets/images/CvG-v0.4-power-inlet.jpg)

### High power motors

M0 and M1 are high-power TMC2160 stepper drivers, conventionally used for driving the XY stage on the printer. 

The MOSFETs attached to the TMC2160s are good enough so as to not require heatsinks — even in excess of 3A drive current — although providing cooling airflow is always useful.

### Driver expansion ports

Each of the two driver expansion ports M2 and M3 is provisioned to accommodate signals for one- or two-driver modules that screw onto the ports by means of screw standoffs (for example, a common configuration is to add one TMC2160 driver per port to drive a cross-gantry or 4-motor corexy stage).

To install additional drivers, use a 2 mm hex key to pop out the orange cover tape on the screw standoff, and fasten the drivers using the included M3x4 screws. **Make sure the screws are secured tightly** — the standoffs are current bearing connections for motor voltage. In addition, fasten a screw on the rear screw standoff to clamp down the drivers completely and ensure reliable signal connections.

![aeiou](/assets/images/CvG-v0.4-drivers-uninstalled.jpg)
![aeiou](/assets/images/CvG-v0.4-drivers-installed.jpg)

### *High voltage operation*

A dedicated motor voltage supply can be used to power M0, M1, M2, and M3 for certain high performance applications. 

Voltage switching is accomplished by using pairs of fuse slots to connect a motor driver bank to either the main 24V supply or the dedicated (up to 48V) motor supply, which should be wired up to the power-in screw terminals labeled '24-48V MOTOR / GND'. The fuse pair marked [0-1] connects power to the two integral TMC2160 drivers (M0, M1) and the fuse pair marked [2-3] connects power to the driver expansion mounting slots (M2, M3).

**Do not** attach both fuses in a fuse pair together as this will short the main and motor power supplies together and may destroy power supplies connected to both. In addition, **do not** connect the M2-M3 ports to an external power supply that exceeds the voltage ratings of the installed drivers — it's very likely that if you have drivers other than TMC2160 on a 48V power supply that they will be destroyed and you will not have a good hardware experience.

*Tip: to remove fuses, gently pull them out of the socket with pliers. To install fuses, first install them partially by hand, then use a large flat object like a table edge to push the fuse down into its fully seated position.*

### Low power motors

The board contains four TMC2208 drivers for powering low power stepper motors, such as Z-axis drives and extruders (up to 1A). They are run off the primary 24V power supply. 

![aeiou](/assets/images/CvG-v0.4-tmc2208.jpg)

### Thermistors 
The board has eight thermistor connectors T0-T7 with 4.7kΩ 1% pullup resistors (will be replaced with 0.1% in next revision). In addition, embedded thermistors are placed near the 12V and 5V dc-dc converters as well as on the M1 and M4 motor drivers to sense their temperatures during operation, and the M2/M3 driver expansion ports are provisioned to accommodate thermistors on attached drivers.

The T0 port appears to not work, perhaps due to a bug with the stm32h7 ADC code.

Testing has shown thermistor noise to be fairly stable, even with directly-connected Pt1000-type sensors.

### Fans 

- F0 and F1 are 4-pin, 12V fans (with frequency generator output and PWM input). 
- F2-F5 are 2-pin fans that can be jumpered to 12V or 24V. 
- F6-F9 are fixed voltage 2-pin fan ports. F6-F8 are 12V only, and F9 is 5V only.

![aeiou](/assets/images/CvG-v0.4-fans.jpg)

### USB and toolhead ports 

The microcontroller USB, accessible through the type-C connector labeled 'MCU USB', is the recommended way to control the board. It can also be used to flash the board firmware in DFU mode. 

The USB-A port labeled 5V PWR is designed to power an attached RPi or other SBC, but it is not usable due to a wiring error and should be taped off. 

The 'TOOLHEAD' connector makes available a 24V, 12V and USB differential pair connection for an in-development tool board, of which the USB is broken out as a separate type-C connector. As I have not finalized the pinout for this connector it is taped off for now, if in the future you order or obtain a corevus toolboard then please contact me for a cable assembly before using this connector.

# Heaters

The H0-H3 heater ports can be used to drive 24V heaters at up to 5A per heater. The HB heater port supports a higher current (~10A); a heatbed or other resistive load can be connected across the HB POS and HB NEG screw terminals.

Although H0 is functional, it is connected to a pin that is pulled on when loading the stm32 bootrom during DFU, it may be wise to not connect that to anything.

Alternatively, you can use these ports or additional fan ports to drive relays / SSRs for larger loads such as mains powered bed heaters. 

![aeiou](/assets/images/CvG-v0.4-heaters.jpg)

### Endstops 
S0 through S3, with signal / gnd / 5V pinout. S3 can be switched between 5V and 24V using an adjacent jumper and has a built-in protection diode for use with inductive probes.
