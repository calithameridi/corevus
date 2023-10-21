# Corevus-G alpha program user's guide
Read the entirety of this document before unboxing the board and proceeding with any integration and testing.

# Safety precautions
So far, I have verified most functionalities on the alpha hardware to function nominally. Nonetheless, the possibility of failures — including catastrophic failures threatening life and property — cannot be ruled out. Please exercise the following precautions while handling or operating the provided hardware:
* Be able to cut power at a moment's notice, such as powering the board / printer from a power strip with an **easily accessible** off switch.
* Maintain a fire extinguisher by the board at all times.
* Do not allow the board to operate near any flammable materials.

**Participation in the alpha program is conditional on you acknowledging the risk of prototype hardware failing catastrophically and making sufficient preparations to mitigate suck risk.** I will be following up with each alpha tester individually to review these terms.

# Feature description

- Motors M0, M1, M2, M3 are TMC2208 drivers supporting ~1A of drive current each. They are connected to the main 24V bus.
- Motors M4, M5 are TMC2160 drivers supporting ~3A of drive current each. The DIAG1 pins on the driver ICs are connected to the microcontroller and can be used for sensorless homing with Trinamic StallGuard. They can be connected to either the main 24V bus or an auxiliary 48V supply (see below).
- Two driver expansion ports are available to mount additional motor drivers to the board. Each driver port exposes signals for 2 drivers, though the included TMC2160 driver boards only uses one of them. They can be connected to either the main 24V bus or an auxiliary 48V supply (see below).
    - Switching drivers between 24V and 48V is done by replacing certain fuses on the board. Feeding 24V into a certain driver group (e.g. M4-M5) is done by installing the fuse in the 24V position for that group. **Do not connect both fuses at once** as this shorts the 24V and 48V supplies together, potentially causing damage or fire.
- The board contains nine fan ports. Two (F0, F1) are four-pin while the remaining seven (F2-F8) are two-pin. The first six (F0-F5) ports can be switched between 24V and 12V using jumpers, while the last three (F6-F8) are 24V only.
- There are four endstop ports (E0-E3) plus an additional inductive probe port (E4, defective, see errata below).
- There are six thermistor ports, all have 4.7kΩ pullup resistors. I have had good results directly connecting PT1000 sensors to them.
- The Pi Zero dock and USB hub is designed to mount and expose the connections on a Pi Zero or Zero 2, such as running the USB port into a hub. A dummy splice board is provided that exposes the upstream port of the hub as USB-C.

## Errata
- Endstop port E4 (inductive probe) is not electrically connected to the microcontroller.
    - _Workaround_: Bodge the output pin of the opto-isolator to another endstop input pin. Ask me if you're using an inductive probe and need this fix to be performed.
- Plugging in the board to a Raspberry Pi or other computer, while the board itself is not powered from 24V, causes the large electrolytic capacitors to charge off the supplied 5V power. this may cause the Pi to reboot as the onboard PMIC detects an overcurrent and shuts down the processor until the capacitors are charged. Fix is planned for next revision.
    - _Workaround_: Ensure the board and the Pi are connected prior to powering up the Pi.  
## Installing Klipper

Installing the Klipper firmware should not be unfamiliar if you have already gone through this process. You may want to disconnect power and all peripherals from the board while installing firmware.

Connect the board via a USB cable to your Raspberry Pi or other Klipper host (**warning:** this may cause a hard power cycle as described above).

On your Pi, `cd ~/klipper` and run `make clean` to remove previously generated firmware, and `make menuconfig` to open the configuration menu. Enter the relevant parameters:
```
        Klipper Firmware Configuration
[*] Enable extra low-level configuration options
    Micro-controller Architecture (STMicroelectronics STM32)  --->
    Processor model (STM32F407)  --->
    Bootloader offset (No bootloader)  --->
    Clock Reference (12 MHz crystal)  --->
    Communication interface (USB (on PA11/PA12))  --->
    USB ids  --->
()  GPIO pins to set at micro-controller startup
```
Run `make` to compile the firmware (this will take a while). Once complete, load the firmware onto the microcontroller:
1. Find the _Kill Heaters_ switch and push it to the left (towards the edge of the board) to disable all heaters. 
    >In practice, this doesn't matter as the board does not erroneously enable heaters in DFU (unlike a certain competing manufacturer), but nonetheless it is wise to reduce the possibility of catastrophic failure modes.
2. On the board, enable the DFU switch (it should light up) and push the (gold-colored) reset button.
3. Check that the microcontroller is in DFU mode and connected to the Pi by running `lsusb`. The board should appear in the resulting list as `0483:df11 STMicroelectronics STM Device in DFU Mode`.
4. Load the compiled Klipper firmware by running `make flash FLASH_DEVICE=0483:df11`. This will take a while.
5. On the board, disable the DFU switch and push the reset button.
6. Verify the firmware has been successfully loaded by running `lsusb` again. The board should now appear as `1d50:614e OpenMoko, Inc. stm32f407xx` or similar.
7. At this point you can run `ls /dev/serial/by-id/*` and copy the resulting directory path into your Klipper printer.cfg file.
    > Remember to use the **directory path** (e.g. `/dev/serial/by-id/usb-Klipper_stm32f407xx_3F0046000351323531343534-if00`) in your printer.cfg. The 24-character string in the device name is the 96-bit unique ID of the STM32 and varies per device.
8. Remember to re-enable heaters with the _Kill Heaters_ switch when you are ready to use them.

# Klipper configuration
For ease of configuration, please download the included pin table [ADD LINK HERE] and copy-paste it into your printer.cfg. It should provide human-readable names for all functionalities provided by the board.

For example, the `[extruder]` on my development machine is specified with the following configuration section. Note the use of `H0`, `T5`, `M0_STEP`, etc. to denote the physical peripherals the extruder and hotend are connected to, as opposed to specifying pin names. 
```
[extruder]
step_pin: M0_STEP
dir_pin: M0_DIR
enable_pin: !M0_EN
microsteps: 32
rotation_distance: 22.55665 
gear_ratio: 80:20
nozzle_diameter: 0.4
filament_diameter: 1.75
pressure_advance: 0.5
pressure_advance_smooth_time: 0.04
pwm_cycle_time: 0.02
control: pid
pid_kp: 10
pid_ki: 1
pid_kd: 20
heater_pin: H0
sensor_type: PT1000
sensor_pin: T5
min_temp: 0
max_temp: 300
max_power: 1
min_extrude_temp: 190
```
Writing your own configuration file should be a straightforward matter of specifying the connected hardware on your machine.
