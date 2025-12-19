# Add your own device

This is a guide to add custom arduino code to Neurokraken. This covers electronics and libraries that have not yet been considered in the existing device libary.

To add a piece of arduino code as a device that neurokraken can read as a sensor or control, the code needs to be wrapped in a simple simple Arduino class that inherits from a Control, Sensor, or Process base class depending on its utility.

Neurokraken's arduino codebase provides useable functions like `int8ToBytes(i)`, `int16Tobytes(i)`, and `longToBytes(i)` that allow sending values to neurokraken as a read_in sensor with int8, int16 and long meaning a maximum value of 255, 65,535 or 4,294,967,295 respectively.

Correspondingly `b = boolFromByte()`, `i = intFromByte()` and `i = int16FromBytes()` can allow reading different precision control values from the python side to act upon.

## Sensors, Controls and Processes

This subchapter will be available soon

## Config.h

`config2teensy.py` automatically generates a file Config.h matching the python side device dictionary. This file enables the arduino side to know which devices to run on which pins.
As it is automatically generated one generally does not have to interact with it. As it can however help illuminate how neurokraken works, this subchapter will walk through its content.

A minimal Config.h is structured like this:

```cpp
#include "Sensors.h"

namespace config{
  StartStop* startStop = new StartStop();
  MillisReader* msRead = new MillisReader();

  Control* controls[] = {startStop};

  Sensor* sensors[] = {msRead};

  Process* processes[] = {};
}
```

The instructions for manually filling out Config.h (as config2teensy would automatically do) are as follows:

add your controlled classes to this array. Your class should:
- subclass Control, so that it can access the received bytes as a entry of its data property, i.e. class MyValve : public Control{}
- contain a void act(){} function that utilizes its updated data to run your written arduino code
- if you are using more than 1 byte of information, overwrite the classe's default numCtrlBytes, i.e. by running numCtrlBytes = 2 in its constructor;
- This array should follow the same order you chose in your python side config.py's serial_out. i.e. startStop, valve0, valve1, rotary encoder control

add your data reading classes to this array. Your class should:
- subclass Sensor, so that it can provide send its read data, i.e. class MyAnalogSensor : public Sensor{}
- contain a void read(){} function that fills up its sensBytes[] with your readings - you can use functions like longToBytes(value) or int8ToByte(value) to do this
- if you are reading more than 1 byte of information, overwrite the classe's default numSensBytes, i.e. by running numBytes = 2 in its constructor;
- This array should follow the same order you choose in your python side config.py's serial_in. i.e. milliseconds, microseconds, rotary encoder, pulseclock

add your classes that have to run a function at high frequency to this array. Your class should:
- subclass Process, so that the arduino void loop() can identify and run them i.e. class MyPulseSignalGenerator : public Process{}
- contain a void step(){} that contains the code you want to run
- The .act(){} or .read(){} of controls and sensors is only executed following communication while a Process's .task() enables running code at much higher frequency.

sensor -> read()
control -> act()
process -> step()

A task's filled in Config.h can then look like this.

```cpp
#include "TimedOn.h"
#include "RotEnc.h"
#include "PulseClock.h"
#include "Sensors.h"
#include "ServoMotor.h"

namespace config{
  // controls
  StartStop* startStop = new StartStop();
  DirectOn* valve0 = new DirectOn(40);
  DirectOn* valve1 = new DirectOn(41);
  ServoMotor* servo = new ServoMotor(15);
  Control* controls[] = {startStop, valve0, valve1, rotEnc, pulseClock, step};

  // Sensors
  MillisReader* msRead = new MillisReader();
  RotEnc* rotEnc = new RotEnc(31, 32);
  PulseClock* clock_100ms = new PulseClock(1,    100);
  PulseClock* clock_1s = new PulseClock(   2,   1000);
  PulseClock* clock_10s = new PulseClock(  3,  10000);
  PulseClock* clock_5min = new PulseClock( 4, 300000);
  AnalogSensor* lightSens = new AnalogSensor(39);
  Sensor* sensors[] = {msRead, clock_100ms, clock_1s, clock_10s, clock_5min, rotEnc, lightSens, pulseClock};

  // These classes also have a process .step() functionality
  Process* processes[] = {clock_100ms, clock_1s, clock_10s, clock_5min, pulseClock, step};
}
```

(middleman-arduino)=
## Adding a new heavy-compute arduino-side device

While the growing range of arduino-compatible components and libraries opens powerful new possibilities for neurokraken, not all devices or libraries are designed for the millisecond precision sampling that neurokraken targets.

Some interesting devices may be implemented in a way that holds up the microcontroller for a considerable time, which would prevent others from running their millisecond or sub-millisecond duties as sensors and/or processes respectively. Devices currently included in `configurators.devices` guarantee high precision even when we use 30 devices together in our tests, however this may not hold true for new devices one may try out. One example of this is the SSD 1306 display.

An easy way to test the impact of a new device is to add the milliseconds clock manually in your task's serial_in with `logging=True`

`serial_in = {'t_ms': devices.time_millis(logging=True), ...`

This will log every change on the current millisecond to your log.json allowing you to see if sampling periods have been missed due to a tested new device.

If your experiments do not care about millisecond precision and are still are able to achieve sufficiently high resolution this of course is not a concern.

If you need to retain millisecond precision and your new device however this is also not a problem, as you can easily add a second microcontroller like an arduino nano as a middleman between your teensy and the new device. The arduino can run the device and absorb delays, while still interacting with the teensy as fast as the added device allows as a prxoy sensor/control without affecting the teensy's timing.



Sometimes it can be useful to outsource computations to an arduino as a secondary microcontroller. For example in rare cases we might have found a great module with an arduino library for an experiment of ours, but the library implementation causes delays of multiple milliseconds in our teensy main loop that we want to avoid, or the library utilizes custom features of the arduino architecture and we couldn't find a teensy-compatible alternative. In these cases the simplest approach is to run our module from an arduino nano, and forward its reading to the central teensy through I2C.



### Code Example

In this example we are using the arduino to collect 2 numbers (they could be created by sensor readings and/or a library). While this example is minimal standalone, you can easily apply it to writing a neurokraken teensy device, by calling a readArduino function like shown here from within the read() function of a neurokraken device's arduino code. Respectively Wire can also be used to provide instructions to the arduino

```arduino
// arduino nano code
#include <Wire.h>

// Being specific with the datatype allows Wire to correctly reinterpret the 
// received number. int8_t for -127 to 127 and uint8_t for 0 to 255
int8_t a = 0;
int8_t b = 0;

void setup() {
  Wire.begin(0x08);             // Join the I2C bus as with address 0x08
  Wire.onRequest(requestEvent); // Run this function when the teensy calls
}

void loop() {
  // constantly update the data points
  a = random(-127,128)
  b = random(-127,128)
}

void requestEvent(){
  Wire.write(a);
  Wire.write(b);
}
```

```arduino
//teensy side code
#include <Wire.h>

int8_t a = 0;
int8_t b = 0;

void setup(){
  Wire.begin();
}

void loop(){
  readArduino();

  Serial.print(a);
  Serial.print(", ");
  Serial.print(b);
  Serial.println();
}

void readArduino(){
  Wire.requestFrom(0x008, 2); // request 2 bytes from the arduino at this address
  if(Wire.available() >= 2){
    a = Wire.read();
    b = Wire.read();
  }
}
```

### Wiring

Most Arduinos run on 5V which would damage the teensy. However we can decide the Voltage to be used for I2C communication by connecting the teensy's 3.3V to our I2C data and clock pins through 4.7kOhm pullup resistors.

<u>Use either a USB connection for the arduino nano or the connection to the teensy's 5V.</u>
For development it can be beneficial to first stay with a USB connection for the nano, add Serial.print() calls mirroring the teensy side to confirm that the data is interpreted the same on both sides.

````{mermaid}
flowchart TB;
subgraph teensy4.1
3.3V
5V
GND
SDA_teensy["SDA(18)"]
SCL_teensy["SCL(19)"]
end

subgraph Arduino Nano
GND_arduino["GND"]
Vin
SDA_nano["SDA(A4)"]
SCL_nano["SCL(A5)"]
end

GND --- GND_arduino
5V -- ! only if no USB is powering or connected to the arduino already ! --> Vin

3.3V --4.7kOhm--> SDA_teensy
3.3V --4.7kOhm--> SCL_teensy
SDA_teensy --- SDA_nano
SCL_teensy --- SCL_nano
```` 

This will work analogous for other arduinos. Some might have other pins for their SDA and SCL though.