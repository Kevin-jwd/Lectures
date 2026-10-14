___
## 온도/기압 측정 흐름

1. Start
2. Start temperature measurement
3. Wait 4.5ms
4. Read UT (uncompensated temperature)
5. Start pressure measurement
6. Wait 4.5ms (depend on mode) (oss = 0 ->ultra low power)
7. Read UP (uncompensated pressure)
8. Calculate pressure and temperature (in driver)

※ 현재 bmp180 의 read()는 온도 전용 경로이므로 5,6,7 과정을 수행하지 않음

___
## Calibration coefficients

- EEPROM contains 11 calibration coefficients.
- Before the first calculation, master should read these values from EEPROM
- AC1, AC2, AC3, AC4, AC5, AC6, B1, B2, MB, MC, MD

___
## Calculation of pressure and temperature

1. Start
2. Read Calibration data from EEPROM
3. Read Uncompensated temperature value
	- write 0x2E into reg 0xF4
	- wait 4.5ms
4. Read Uncompensated pressure value (sampling mode (00): single)
	- write 0x34 + (oss<<6) into reg 0xF4
	- wait 4.5ms
5. calculate true temperature
6. calculate true pressure
7. return to step 3.

___
## Global Memory Map

- **out_xlsb** (Data register, readonly)
- **out_lsb** (Data register, readonly)
- **out_msb** (Data register, readonly)
- **ctrl_meas** (Control register, r/w)
	- sco <5> : conversion start, value stays `1` during conversion, reset to `0` after conversion completed
	- oss <7:6> : controls the oversampling ratio of the pressure measurement (in our project, 00 which means single)
- **soft_reset** (Control register, r/w)
	- if set to 0xB6, will perform as power on reset
- **id** (Fixed, readonly)
	- fixed to 0x55 and can be used to check whether communication is functioning
- **calib21** (Calibration register, readonly)