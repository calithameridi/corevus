# Corevus Toolboard v1.0 user's manual

Work in progress.

## Specifications
- Designed for high temperature operation. Tested to 75 degC chamber temperature so far with no issues. Designed to survive 100+ degC, but haven't run it that hot yet.
- 150 MHz STM32G4 microcontroller
- TMC2209 extruder driver
- 1x 24V heater port, 2-pin Molex Micro-Fit, 5A capability
- 1x thermistor port, 2-pin JST-XH, 2.2kΩ pullup resistor  
- 1x endstop port, 3-pin JST-XH
- 1x heater fan port, 3-pin JST XH, fixed 5V fan port with tachometer to run everyone's favorite [25mm fan](https://www.digikey.com/en/products/detail/delta-electronics/ASB02505SHA-AY6B/7491489). Compatible with all 5V fans though, whether with tachometer or without.
- 2x print fan ports, 2-pin JST-XH, 12/24V voltage selectable via jumper
- Breakout pin header for additional I/O



## Firmware



## Configuration




###
```
[board_pins corevus-toolboard-v1.0] # Corevus Toolboard v1.0 pin alias table. THIS MUST MATCH YOUR VERSION NUMBER!
mcu: ctb # specify name given to the corevus toolboard microcontroller, typically 'ctb'
aliases:
    M0_DIR = PA0, M0_THERM = PA1, M0_STEP = PA2, M0_UART = PA3, M0_EN = PA8, F0_FG = PA9, S0 = PA10, H0 = PA13, F0 = PA14, ACCEL_CS = PA15, T0 = PB0, ACCEL_INT1 = PB6, F1 = PB7
```

### Temperature sensors
The `temperature_mcu` reads out the temperature of the `ctb` microcontroller `CvBoardTherm` is a 10kΩ, 3950K SMD thermistor that I use on various parts of the
```
[temperature_sensor Toolboard]
sensor_type: temperature_mcu
sensor_mcu: ctb
max_temp: 140

[thermistor CvBoardTherm]
temperature1: 25.0
resistance1: 10000.0
beta: 3950

[temperature_sensor Extruder_Driver] # toolboard TMC2209 temp
sensor_type: CvBoardTherm
pullup_resistor: 4700
sensor_pin: ctb:M0_THERM
min_temp: -20
max_temp: 125

#[temperature_sensor Heater_Driver] # toolboard heater MOSFET temp
#sensor_type: CvBoardTherm
#pullup_resistor: 4700
#sensor_pin: ctb:PA4
#min_temp: -20
#max_temp: 125
```
