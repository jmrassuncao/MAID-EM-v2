## Library for reading SDM120 SDM220 SDM230 SDM630 Modbus Energy meters. ##

### SECTIONS: ###
#### 1. [INTRODUCTION](#introduction) ####
#### 2. [SCREENSHOTS](#screenshots) ####
#### 3. [INITIALIZING](#initializing) ####
#### 4. [READING](#reading) ####
#### 5. [DEBUGING](#debuging) ####
#### 6. [CREDITS](#credits) ####

---

### Introduction: ###
This library allows you reading SDM module(s) using:
- [x] Hardware Serial (<i>recommended option, smallest number of reads errors</i>) <b><i>or</i></b>
- [x] Software Serial (<i>library for ESP8266</i> https://github.com/plerup/espsoftwareserial)

you also need rs232<->rs485 converter:
- [x] with automatic flow direction control (<i>look at images below</i>) <b><i>or</i></b>
- [x] with additional pins for flow control, like MAX485</br>
     (<i>in this case MAX485 DE and RE pins must be connected together to one of esp pin</br>
     and this pin must be passed when initializing the library</i>)

_Tested on Wemos D1 Mini with Arduino IDE 1.8.3-1.9.0b & ESP8266 core 2.3.0-2.4.1_

---

### Screenshots: ###
<img src="https://github.com/reaper7/SDM_Energy_Meter/blob/master/img/hardware_sdm220_1.jpg" height="330"><img src="https://github.com/reaper7/SDM_Energy_Meter/blob/master/img/hardware_sdm220_2.jpg" height="330"></br>
<p align="center">
  <img src="https://github.com/reaper7/SDM_Energy_Meter/blob/master/img/livepage.gif"></br>
  <i>live page example (extended) screenshot</i>
</p>

---

### Initializing: ###
```cpp
//lib init when Software Serial is used:
#include <SDM.h>
//      ______________________baudrate
//     |    __________________rx pin
//     |   |    ______________tx pin
//     |   |   |    __________dere pin(optional for max485)
//     |   |   |   |
SDM<4800, 13, 15, 12> sdm;

//lib init when Hardware Serial is used:
#define USE_HARDWARESERIAL
#include <SDM.h>
//      ______________________baudrate
//     |    __________________dere pin(optional for max485)
//     |   |    ______________swap hw serial pins from 3/1 to 13/15(default false)
//     |   |   |
SDM<4800, 12, false> sdm;
```
NOTE: <i>when GPIO15 is used (especially for swapped hardware serial):</br>
some converters (like mine) have built-in pullup resistors on TX/RX lines from rs232 side,</br>
connection this type of converters to ESP8266 pin GPIO15 block booting process.</br>
In this case you can replace the pull-up resistor on converter with higher value (100k),</br>
to ensure low level on GPIO15 by built-in in most ESP8266 modules pulldown resistor.</br></i>

---

### Reading: ###
List of available registers for SDM120/220/230/630:</br>
https://github.com/reaper7/SDM_Energy_Meter/blob/master/SDM.h#L36
```cpp
//reading voltage from SDM with slave address 0x01 (default)
//                                      __________register name
//                                     |
float voltage = sdm.readVal(SDM220T_VOLTAGE);

//reading power from 1st SDM with slave address ID = 0x01
//reading power from 2nd SDM with slave address ID = 0x02
//useful with several meters on RS485 line
//                                      __________register name
//                                     |      ____SDM device ID  
//                                     |     |
float power1 = sdm.readVal(SDM220T_POWER, 0x01);
float power2 = sdm.readVal(SDM220T_POWER, 0x02);
```
NOTE: <i>if you reading multiple SDM devices on the same RS485 line,</br>
remember to set the same transmission parameters on each device,</br>
only ID must be different for each SDM device.</i>

---

### Debuging: ###
Sometimes <b>readVal</b> return <b>NaN</b> value (not a number),</br>
this means that the requested value could not be read from the sdm module for various reasons.</br>

__Please check out open and close issues, maybe the cause of your error is explained or solved there.__

The most common problems are:
- weak or poorly filtered power supply / LDO, causing NaN readings and ESP crashes</br>
  https://github.com/reaper7/SDM_Energy_Meter/issues/13#issuecomment-353532711</br>
  https://github.com/reaper7/SDM_Energy_Meter/issues/13#issuecomment-353572909</br>
  https://github.com/reaper7/SDM_Energy_Meter/issues/8#issuecomment-381402008</br>
- faulty or incorrectly prepared converter</br>
  https://github.com/reaper7/SDM_Energy_Meter/issues/16#issue-311042308</br>
- faulty esp module</br>
  https://github.com/reaper7/SDM_Energy_Meter/issues/8#issuecomment-381398551</br>
- many users report that between each readings should be placed <i>delay(50);</i></br>
  https://github.com/reaper7/SDM_Energy_Meter/issues/7#issuecomment-272080139</br>
  (I did not observe such problems using the HardwareSerial connection)</br>
- using GPIO15 without checking signal level (note above)</br>
  https://github.com/reaper7/SDM_Energy_Meter/issues/17#issue-313606825</br>
  https://github.com/reaper7/SDM_Energy_Meter/issues/13#issuecomment-353413146</br>
  https://github.com/reaper7/SDM_Energy_Meter/issues/13#issuecomment-353417658</br>

You can get last error code using function:
```cpp
//get last error code
//                                      __________optional parameter,
//                                     |          true -> read and reset error code
//                                     |          false or no parameter -> read error code
//                                     |          but not reset stored code (for future checking)
//                                     |          will be overwriten when next error occurs
uint16_t lasterror = sdm.getErrCode(true);

//clear error code also available with:
sdm.clearErrCode();
```
Errors list returned by <b>getErrCode</b>:</br>
https://github.com/reaper7/SDM_Energy_Meter/blob/master/SDM.h#L128</br>

You can also check total number of errors using function:
```cpp
//get total errors counter
//                                       _________optional parameter,
//                                      |         true -> read and reset errors counter
//                                      |         false or no parameter -> read errors counter
//                                      |         but not reset stored counter (for future checking)
uint16_t cnterrors = sdm.getErrCount(true);

//clear errors counter also available with:
sdm.clearErrCount();
```

---

### Credits: ###

:+1: ESP SoftwareSerial library by Peter Lerup (https://github.com/plerup/espsoftwareserial)</br>
:+1: crc calculation by Jaime Garc??a (https://github.com/peninquen/Modbus-Energy-Monitor-Arduino)</br>
:+1: new registers for SDM120 and SDM630 by bart.e (https://github.com/beireken/SDM220t)</br>

---

**2016-2018 Reaper7**

[paypal.me/reaper7md](https://www.paypal.me/reaper7md)
