# Corevus-G v0.4 beta test manual

*See also*: [Hardware Overview](../CvG-v0.4-hardware-overview.md)


# Flashing Klipper

The Corevus-G v0.4 boards are tested and flashed with the klipper mcu firmware prior to shipout. However in the event that you need to flash klipper, use the following procedure:
- Unplug everything from the board except the USB connection to the klipper host (or any machine with the klipper build chain), you may find it helpful to take a photo before dismantling attached cables
- on the klipper host, run `cd ~/klipper` to enter the klipper source directory
- Run `make clean` to delete previous builds and `make menuconfig`
- In the configuration menu, select the following parameters: `STMicroelectronics STM32`, `STM32H723`, `No bootloader`, and `12 MHz crystal`.
- Exit the configuration menu and save you
- Run `make` and wait for the firmware to finish compiling
- Switch the device into DFU mode by holding down the BOOT pushbutton and then pushing and releasing the RESET pushbutton
- Reinstall the board in accordance to your previous wiring scheme

# Configuration 


## Pin table
Add the following section to your configuration. 

```
### Corevus-G v0.4 Pin Aliases ###

[board_pins corevus-g]
mcu: # specify name given to the corevus microcontroller, typically 'mcu'
aliases:
# TMC2160 drivers
	M0_STEP = PG8,  M0_DIR = PG7, M0_EN = PG5,  M0_CS = PG6, M0_DIAG1 = PG4,
	M1_STEP = PD15, M1_DIR = PG2, M1_EN = PD14, M1_CS = PG3, M1_DIAG1 = PD13,
# Driver expansion ports
	M2A_STEP = PB12, M2A_DIR = PE12, M2A_EN = PE13, M2A_CS = PE14, M2A_DIAG = PE15,
	M2B_STEP = PD12, M2B_DIR = PD11, M2B_EN = PD10, M2B_CS = PD9,  M2B_DIAG = PD8,
	M3A_STEP = PF13, M3A_DIR = PF14, M3A_EN = PF15, M3A_CS = PG1,  M3A_DIAG = PG0,
	M3B_STEP = PE11, M3B_DIR = PE10, M3B_EN = PE9,  M3B_CS = PE8,  M3B_DIAG = PE7,
# TMC2208 drivers
	M4_STEP = PF11, M4_DIR = PC4,  M4_EN = PC5,  M4_UART = PF12,
	M5_STEP = PF3,  M5_DIR = PF1,  M5_EN = PF2,  M5_UART = PF4,
	M6_STEP = PC15, M6_DIR = PC13, M6_EN = PC14, M6_UART = PF0,
	M7_STEP = PE4,  M7_DIR = PE2,  M7_EN = PE3,  M6_UART = PE5,
# Heaters
	HB = PD2, H0 = PD1, H1 = PD0, H2 = PD3, H3 = PD4,
# Fans
	F0 = PA10, F1 = PA15,
	F2 = PB4, F3 = PB3, F4 = PB6, F5 = PB5, 
	F6 = PB9, F7 = PB8, F8 = PE6, F9 = PB7,
	F0_FG = PC11, F1_FG = PC10,
# Thermistors
	T0 = PA0, T1 = PA1, T2 = PC2, T3 = PC3,
	T4 = PC0, T5 = PC1, T6 = PF6, T7 = PF7,
	M1_THERM = PB1, M2_THERM = PB0, M3_THERM = PF8,
	M4_THERM = PA4, 5V_THERM = PF9, 12V_THERM = PF10,
# Endstops
	S0 = PG12, S1 = PG13, S2 = PG14, S3 = PG15
```



## Thermistors
Corevus-G features 4 onboard thermistors soldered on to measure temperature in key locations. These temperature sensors not only present a useful diagnostic tool but also provide an extra layer of safety from serious errors such as motor driver misconfigurations. 

Add the following section to your config file. If the default `max_temp` of 85°C is too low, and providing more cooling / airflow to the board is not an option, you may have to increase the `max_temp`. Do not increase the value past 100°C. 

The expansion motor drivers also feature built-in thermistors, if you have drivers installed in ports M2 and/or M3 their temperatures can be read out by uncommenting the relevant sections.

```
### Corevus-G v0.4 Thermistors ###

[thermistor CvBoardTherm]
temperature1: 25.0
resistance1: 10000.0
beta: 3950

[temperature_sensor M1_Driver]
sensor_type: CvBoardTherm
sensor_pin: M1_THERM
min_temp: -20
max_temp: 85

[temperature_sensor M4_Driver]
sensor_type: CvBoardTherm
sensor_pin: M4_THERM
min_temp: -20
max_temp: 85

[temperature_sensor 12V]
sensor_type: CvBoardTherm
sensor_pin: 12V_THERM
min_temp: -20
max_temp: 85

[temperature_sensor 5V]
sensor_type: CvBoardTherm
sensor_pin: 5V_THERM
min_temp: -20
max_temp: 85

# Uncomment if using expansion ports M2 and/or M3

#[temperature_sensor M2_Driver]
#sensor_type: SDNT1005X103F3950FTF
#sensor_pin: M2_THERM
#min_temp: -20
#max_temp: 85

#[temperature_sensor M3_Driver]
#sensor_type: SDNT1005X103F3950FTF
#sensor_pin: M3_THERM
#min_temp: -20
#max_temp: 85
```


## Motors 

## Toolhead connector
