# Power8Board V1

48V → 12V Power Distribution & Current-Sensing Board for 8 Thrusters

## Overview

Power8Board is a custom power distribution and sensing PCB intended for robotics applications (e.g. ROVs / AUVs) where 6 DoF control is needed from eight thrusters. 

The board converts 48V DC into 5V and 12V rails, distributes power to eight ESCs, and provides per-thruster current measurement and voltage rail monitoring (3.3, 5, 12, 48) over I²C.

---

## Power Architecture

Two [MPQ860-12V72-L48NBMC](https://www.digikey.com/en/products/detail/murata-power-solutions-inc/MPQ860-12V72-L48NBMC/26005878?s=N4IgTCBcDaILYAcCOAOAbABhAXQL5A) 860W power converters are used to step down 48 to 12V with high efficiency. This 12V power is then routed to eight XT30U-M connectors which ESCs can then connect to for power. These were chosen due to having four power pins, allowing for better heat distribution, and for their active current sharing capabilities. With ~1500W (not 1720 due to drop in max wattage output when current sharing), eight standard T200 thrusters can be driven at full power. This 12V can also be used to power other 12V devices using the [MR30 connector](https://www.tme.com/us/en-us/details/mr30pw-m/dc-power-connectors/amass/) on the board. 

Additionally, a [UWS-5/10-Q48N-C](https://www.digikey.com/en/products/detail/murata-power-solutions-inc/UWS-5-10-Q48N-C/3041704?s=N4IgTCBcDaIKoHUDKBaArAegIwAYUEUAWADgDkUBhEAXQF8g) 100W power converter is used to step down 48V to 5V. This 5V is outputted through the MR30 and can be used to power a Raspberry Pi for instance.

---

## Current Sensing

Monitor thruster load and detect abnormal conditions (e.g. over-current):

### Current-Sense Amplifiers
  * [INA240A2](https://www.ti.com/product/INA240) (analog)
  * One per ESC channel
  * 50x amplification value (50V/V gain)
* 0.0033 $\Omega$ shunt resistor placed in series with each 12V output
  * Scale factor: 1 / (SHUNT_VALUE * AMP_FACTOR) = 1 / (0.0033 * 50) = 6.061
  * current = 6.061 * (output of INA240)
* Analog output routed to ADC inputs with RC-filter (680 $\Omega$, 2.2 uF)

### Channel Organization

* Left side: 4 thrusters
* Right side: 4 thrusters
* Each side feeds into its own ADC, an [ADS1015](https://www.ti.com/product/ADS1015) (12-bit, I²C interfacing)
* I²C address:
  * Left: 0x48
  * Right: 0x49

---

## Voltage Sensing

Monitor that voltage rails are within normal bounds.

### Power Rails

| Rail  | Purpose                    |
| ----- | -------------------------- |
| 48 V  | Main input power           |
| 12 V  | ESC & thruster power       |
| 5 V   | Miscellaneous              |
| 3.3 V | ADCs, current sensors, I²C |

### ADC

* [ADS1015](https://www.ti.com/product/ADS1015) (12-bit, I²C interfacing)
  * Used for measuring the voltages
  * Address: 0x4A

### Voltage Dividers

<img width="1240" height="581" alt="image" src="https://github.com/user-attachments/assets/e06c1844-2f70-43fd-a123-f4c59d74d12f" />

### Scalar Values
  * 48V: (470+25)/25 = 19.8
  * 12V: (470+110)/110 = 5.273
  * 5V: (470+470)/470 = 2
  * 3.3V: (470+470)/470 = 2

Multiply these values read from the ADC by these scalars respectively to find their true voltages.

---

## LED Indicators

| Rail  | Color                      |
| ----- | -------------------------- |
| 48 V  | Green                      |
| 12 V  | Yellow                     |
| 5 V   | Red                        |
| 3.3 V | Amber                      |

---

## PCB Layout

### 6-layer stackup

| Layer | Type                      |
| ----- | ------------------------- |
| 1     | Signal                    |
| 2     | Ground                    |
| 3     | Signal - Power (3.3V)     |
| 4     | Power (12V & 48V)         |
| 5     | Ground                    |
| 6     | Signal                    |

<img width="1825" height="1088" alt="image" src="https://github.com/user-attachments/assets/89e401bc-1455-4b4c-9c96-b00cc128263f" />

### Design notes

* Nearly all analog traces are routed on layer 3 to reduce effects of digital noise.
* Copper pours help with thermal dissipation.
* Via-in-pad is used to condense routing. It's also a standard option for 6-layer boards when ordering with JLCPCB.
* Ground on layer 2 & 5 provides low-impedance return path and reduce EMI.
* Power converters connect from the backside. 

---

## Ordering

To order, navigate to Power8Board V1/Export. Use Power8Board V1.zip for ordering the PCB and Power8Board V1 Stencil.zip for ordering a stencil (very helpful for soldering). Here are the options I chose when ordering from JLCPCB:

<img width="1427" height="1173" alt="image" src="https://github.com/user-attachments/assets/ee1c1c0e-c497-439d-b427-3278b853226f" />

For ordering the components, navigate to Power8Board V1/bom and click on ibom.html. This BOM is interactive, making it great for ordering and knowing where components go while soldering. **Note that this does not include all components for making the board.** This is because pin receptacles are soldered onto the pads for all power converters. Here are the two types of pin receptacles used on the board:

Pin #1: https://www.digikey.com/en/products/detail/mill-max-manufacturing-corp/0372-0-15-15-13-27-10-0/433860

Pin #2: https://www.digikey.com/en/products/detail/mill-max-manufacturing-corp/0312-0-15-15-34-27-10-0/433567

Pin #1 is used on pads 4, 5, 7, and 8 on both 48V-12V converters and on pads 4 and 8 on the 48V-5V converter (10 in total)

Pin #2 is used on pads 1, 2, and 3 on both 48V-12V converters and on pads 1, 2, 3, 5, 6, and 7 on the 48V-5V converter (12 in total)

---

Email me at harry.c.ohagin@gmail.com if you have any questions. 
