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

(extra-arduino)=
## Adding an extra microcontroller

A neurokraken device can also be another microcontroller, a 2nd teensy, an arduino, or ESP32. while one generally can liberally add new sensors and controlled devices to the main teensy, there are situations where adding another microcontroller as a middleman to further devices can be useful.

First, while the growing range of arduino-compatible components and libraries opens powerful new possibilities for neurokraken, not all devices or libraries are designed for the millisecond precision sampling that neurokraken targets. Rare cases might hold up the microcontroller preventing other connected devices from running with their millisecond or sub-millisecond duties as sensors and/or processes respectively. If you connect them to an extra intermediary arduino or teensy, your main teensy can still rush through its duties while also utilizing the troublesome device to its full capabilities.

Devices currently included in `configurators.devices` guarantee high precision even when we use 30 of them together in our tests, however when your task requires adding a new arduino module/library as a device, and millisecond precision is important for your experiment, it can be useful to run a quick performance test to verify that your teensy remains fast or whether you should add an extra microcontroller. To performance test, you can run the task with the Neurokraken object argument `log_performance=True`. After the task you can run `toolkit/performance_test/performance_test.py` and provide it with the created log folder - if the "sensor sampling" graph shows you a flat 1 millisecond difference between datapoints you are good.

Second, an arduino library may utilize custom features of the arduino architecture and a teensy-compatible alternative is not easily found. Third you may yourself want to write arduino-side device code that requires delays or large compute.

### Code Example

In this example we are connecting another microcontroller that collects 2 numbers (they could be created by sensor readings and/or a library) and through a wire connections provides the current readings to to the main teensy upon request. On the main teensy side the approach is the same as adding any other Sensor/Control/Process, only that we use the arduino Wire library within the respective read() or act() function to update values shared with the extra microcontroller. Here we create the device as a sensor and provide the average to our task.

We first upload the standalone code of the extra microcontroller.

```cpp
// extra teensy/arduino code
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
  a = random(-127,128);
  b = random(-127,128);
}

void requestEvent(){
  Wire.write(a);
  Wire.write(b);
}
```

The following device file is to be added to neurokraken's teensy folder - we are treating the connected extra microcontroller as a sensor.

```cpp
// ExtraArduino.h
#include <Wire.h>

class Extra : public Sensor{
  public:
    int pin;

    Extra(){
      Wire.begin();
    }

    void read(){
      Wire.requestFrom(0x08, 2); // request 2 bytes from the arduino at this address
      if(Wire.available() >= 2){
        int a = Wire.read();
        int b = Wire.read();
        int c = (a + b)  / 2;
        valueToBytes(c);         // provide the value to the python side
      }
    }
};
```

On the python side, we add a serial_in entry with the name chosen for our arduino class (Extra) and parameters for our sensor value range.

```python
serial_in = {
  'extra': {'value': 0, 'encoding': 'int', 'byte_length': 1, 'logging': True, 'arduino_class': 'Extra'}
}
# Our task code may later access this sensor with get.read_in('extra')
```

Lastly we can run `config2teensy.py` to update the code within the teensy folder, upload it and start our task. The 2ndary microcontroller here acts as any other sensor, and if it can provide you with 1000 different values per second you will see this data density reflected in your log.

### Wiring

Microcontrollers have 2 competing standards with one set running at 3.3V like the teensy and the the other set including many arduino models running at 5V.
The two wiring diagrams show the connection with another 3.3V device and a 5V device.

````{mermaid}
flowchart TB;
subgraph main teensy4.1
GND
SDA_teensy["SDA(18)"]
SCL_teensy["SCL(19)"]
end

subgraph extra teensy4.1
GND_extra["GND"]
SDA_extra["SDA(18)"]
SCL_extra["SCL(19)"]
end

GND --- GND_extra
SDA_teensy --- SDA_extra
SCL_teensy --- SCL_extra
````

To interact with a 5V microcontroller like the arduino nano we can decide the Voltage to be used for I2C communication by connecting the teensy's 3.3V to our I2C data and clock pins through 4.7kOhm pullup resistors.

````{mermaid}
flowchart TB;
subgraph teensy4.1
3.3V
GND
SDA_teensy["SDA(18)"]
SCL_teensy["SCL(19)"]
end

subgraph Arduino Nano
GND_arduino["GND"]
SDA_nano["SDA(A4)"]
SCL_nano["SCL(A5)"]
end

GND --- GND_arduino

3.3V --4.7kOhm--> SDA_teensy
3.3V --4.7kOhm--> SCL_teensy
SDA_teensy --- SDA_nano
SCL_teensy --- SCL_nano
```` 

In the examples above both microcontrollers are powered by USB. If we want to reduce cables and be more effective we can also power the extra microcontroller from the teensy's 3.3V or 5V pins depending on the Voltage used by the targeted device.

<u>Use either a USB connection for the extra microcontroller or the 5V/3V from the main teensy - don't have both approaches connected at the same time.</u>

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