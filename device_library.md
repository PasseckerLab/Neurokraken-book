# Device Library - Sensors, Stimuli, Peripheral Devices, Arduino

## Adding and using arduino-side devices

As the ecosystem of arduino compatible components and libraries is steadily growing, so is the number of components that can be incorporated into neuroscience experiments through neurokraken.

Neurokraken's `configurators.devices` allow you to mix and match arduino side devices according to experiment needs and to automatically create the arduino side code ready to upload to the teensy microcontroller.

For example the following start of a task script is already sufficient for a task that requires the the ability to read a rotation value and to control an LED light. It is also sufficient to run `config2teensy.py` to create the arduino side code for these devices connected to the teensy's pins 1, 2 and 3 respeticely without any further manual work required.

This ability to easily add and switch devices of tasks allows for fast development, testing and application of neurokraken tasks while maintaining flexibility for varying and updating task setups.

```python
from neurokraken import Neurokraken
from neurokraken.configurators import devices

serial_in = {'rotation': devices.rotary_encoder(pins=(1,2))}

serial_out = {'led': devices.direct_on(pin=3, start_value=False)
}
nk = Neurokraken(serial_in=serial_in, serial_out=serial_out)
```

In your task code you can then easily read the current rotation and set the state of the LED by using `neurokraken.controls.get` with the respective device names provided in serial_in and serial_out above.

```python
current_rotation = get.read_in('rotation')

get.send_out('led', True)
```

## Non-arduino-side devices

The [configuration documentation](configuration) can guide you towards integrating computer-side devices like displays, cameras, microphones and speakers.

## The device library

Many `configurators.devices` can be used multi-functionally. For example `devices.direct_on(pin:int)` allows you to set a HIGH or LOW voltage on the provided pin from your task code irrespective of the component receiving this HIGH/LOW signal. Thus a direct_on can be used to control an LED as well as a valve, an audible buzzer, a solenoid piston, a lamp, a heater,... 

Conversely some components can be used with different neurokraken.devices. For example our LED can be a `devices.direct_on(pin:int)` or a `devices.timed_on(pin:int)` that would default stay LOW but allow us to `get.send_out('led', 100)` to turn HIGH for a duration of exactly 100 milliseconds before turning LOW again, or it could even be run through a `devices.servo(pin:int)`.

Lastly, some modalities can be implemented with alternative components. For example a rotation angle can be sensed by a rotary encoder (`devices.rorary_encoder`) or a potentiometer (`devices.anaolg_read`) that trades resolution, continuous multi-rotation angles, and a ready to use low-friction axle for smaller size and cost.

Thus the device library is organized as follows:

1. [Modalities](modalities)
2. [Configurators.devices](configurators-devices)
3. [Wiring guides](Wiring-guides)

[Modalities](modalities) covers options for the different modalities one may want to implement and links to the corresponding [Configurators.devices](configurators-devices).

Further [Wiring guides](Wiring-guides) contains corresponding wiring diagrams for each device.

Lastly Ordering Options in the hardware chapter contains a table of affordable and useful components for different neuroscience setups with links and prices for the respective items.

## Adding your own device

While the existing neurokraken configurators.devices cover the dominant rabge of electronics that we can think of being useful in behavior setup, there will always be unanticipated ideas and needs.

Research progresses through the adoption of new ideas, so neurokraken makes it easy to add a new device we haven't considered yet.

If you have standalone arduino code for your device, this code can be wrapped within a Sensor, Control, or Process context (or a combination of these) for neurokraken to understand how to interact with it. [Our page on adding your own devices](arduino_custom) guides you through how to do this.

Outside of neurokraken's ability to add your own device we strive to continously extend the pre-existing configurators.devices according to popular needs of the research community.

## Adding an arduino as a device

[The add your own device page](extra-arduino) has an example of adding an arduino as a teensy connected device

(modalities)=
## Modalities

### Reading modalities

#### Rotation

Rotation can be read with a rotary encoder ([devices.rotary_encoder](devices.rotary_encoder)) or a potentiometer ([devices.analog_read](devices.analog_read)). Rotary encoders are useful for high precision low friction rotation readings like on a steering wheel or treading wheel/disc, while potentiometers are small and cheap.

Rotation can be useful for measuring a steering wheel, or progress on a walking disc.

#### Linear position

Slider-shaped potentiometers can be read as a potentiometer ([devices.analog_read](devices.analog_read)), as can joysticks like the arduino psp2 module.

#### Brightness/Beam breaking

Light and its blockage can be read with a photoresistor or a beam breaking sensor as a ([devices.analog_read](devices.analog_read)) or ([devices.binary_read](devices.binary_read))

#### Touch

Touch can be read through a capacitive touch breakout board as a ([devices.binary_read](devices.binary_read)) or in the shape of a beam break as a ([devices.analog_read](devices.analog_read)). As capacitive touch is based on rapidly charging and discharging the touchable element a beam break can be preferable for electrophysiology experiment.

#### Buttons

Buttons and switches can be read as a ([devices.binary_read](devices.binary_read))

#### Vibration

Vibration can be sensed through a shake switch as a ([devices.analog_read](devices.analog_read)).

#### Additional reading sensors

While the readings above cover most use cases, the range of readings gatherable from neurokraken tasks is only limited by arduino-compatible sensors allowing for many new experiment designs. The most difficult cases can even be integrated through an [extra-arduino](extra-arduino) if required. 

Examples of arduino sensors that could be added include `distance (time of flight) sensors`, `flex sensors`, `accelerometers`, `temperature sensors`, ...

### Controlling modalities

#### Motoric movement

A servo motor [devices.servo](devices.servo) can move things through a rotating arm. A 3D printabel "rack and pinion" mechanism can turn this rotation into linear movement. Please note that for a servo to work one of the following pins has to be used: 0-15, 18-19, 22-25, 28-29, 33, 36-37 (https://www.pjrc.com/teensy/td_libs_Servo.html). Rapid linear movement can also be achieved through a solenoid piston which can be either a [devices.direct_on](devices.direct_on) or a [devices.timed_on](devices.timed_on).

#### Liquid/Odor delivery (Valves)

A solenoid-operated pinch valve is an easy way to open or close rubber tubing providing liquid or air to a subject. Depending on the use case they can exist in Normally-Closed or Normally-Open and will change to the alternative when powered. This can be done through signal from either a a [devices.direct_on](devices.direct_on) or a [devices.timed_on](devices.timed_on). A common use case is to provide precisely sized task-reward droplets of sugar water as a reward through a short valve opening.

#### LEDs

LEDs can be controlled through a [devices.direct_on](devices.direct_on) or a [devices.timed_on](devices.timed_on), or as adjustable brightness is controlled through the same signal as a servo's position, a [devices.servo](devices.servo).

#### Tone

Auditory frequencies from a connected passive buzzer or piezo can be generated through a [devices.tone](devices.tone).

#### Synchronization pulse signals

Regular synchronization signals with a chosen frequency can be created as a (devices.pulse_clock)[devices.pulse_clock]. These signals are useful to create a common time signal across separate pieces of experiment equipment, like the neurokraken behavior setup and an external electrophysiology recording board. Some external cameras also utilize this type of signal to time their frame captures.

#### Displays and Audio speakers

More advanced stimuli like computer display graphics, or audio playback can best be run from the computer/python side as described in [Configurators](configuration.md).

(configurators-devices)=
## Configurators.devices

Configurators.devices provides functions to create, mix and match devices for a given task. A task may utilize 1 devices.direct_on() and 1 devices.servo(), or it may utilize 5 devices.servo() all on different pins.

The devices can be added to the `serial_in` dictionary for sensors, and the `serial_out` dictionary for controlled devices, each entry with a chosen key string identifier that can later in your task be used to address sensors to read from and controls to update.

```python
from neurokraken import Neurokraken, State
from neurokraken import get
from neurokraken.configurators import devices

serial_in =  {'light_beam': devices.analog_read(pin=3)}
serial_out = {'reward_valve': devices.timed_on(pin=2)}

nk = Neurokraken(serial_in=serial_in, serial_out=serial_out)

class My_state(State):
    def loop_main(self):
        light_strength = get.read_in('light_beam')
        if light_strenght < 400:
            get.send_out('reward_valve', 100) # open for 100ms

nk.load_task({'A': My_state(next_state='A')})
nk.run()
```

Commonly a device will require one or two pins as argument. Sometimes a specific optional argument can be provided, like the `start_value=True/False` for a `devices.direct_on`.

### Adding and extending beyond configurators.devices

configurators.devices functions return standardized dictionaries, which can exchangeably be used directly in as entries in serial_in and serial_out. This way a new custom experiment device can directly be added as a dictionary matching the device's communicated data. configurators.devices are convenient for rapid development with established devices, but are not a limitation if more flexibility for new devices is required.

### A device as both sensor and control

Neurokraken can support a device being a sensor as well as a control. This is currently exemplified in configurators.devices.rotary_encoder which is a sensor but depending on the optional argument `control=` (defaults to False) can return the optional control entry for serial_out instead. Notably the key name provided in serial_in and serial_out has to match so that this device is recognized as the same.

### Sensors

Sensors have optional arguments for keyboard mode to assign keys and their effect on the sensor value (`keys=` and `keys_control=`) as detailed in the [modes chapter](modes.md). A second optional sensor argument is `logging=True/False` (defaults to True) which allows you to disable logging for a specific sensor.

```python
serial_in =  {'light_beam': devices.analog_read(pin=3, keys=['s', 'w'], logging=False)}
```

(devices.binary_read)=
#### binary_read(pin:int)

A digital read of a pin (HIGH or Low), (True or False). Many devices can provide a suitable input to an arduino pin from simple buttons to touch sensor modules. Provides 0 or 1 upon get.read_in(<device name>).

```python
serial_in = {'licking_sensor': true_false_sensor(pin=3)}
...
print(get.read_in('licking_sensor')) # 0 or 1
```

(devices.analog_read)=
#### analog_read(pin:int)

Analog read of a pin. Provides a continuous value 0 to 1023 upon get.read_in(<device name>).

Analog reads can be used with devices like an angle measuring potentiometer or light sensitive photoresistor.

```python
serial_in = {'rotation_potentiometer': continuous_sensor(pin=3)}
...
print(get.read_in('rotation_potentiometer')) # 387
```

(devices.rotary_encoder)=
#### rotary encoder(pins:list)

Rotation position of rotary encoder. Returns -2,147,483,648 to +2,147,483,647 upon get.read_in(<device name>).

Rotary encoders are useful for high precision low friction rotation readings like on a steering wheel or treading wheel/disc.

The optional argument `controls=True/False` (defaults to False) can be used to return the optional control entry for serial_out instead of the sensor_value for serial_in. When adding the control running get.send_out(<device_name>) allows resetting the wheel position to 0.

```python
serial_in = {'wheel_pos': rotary_encoder(pins=[3,4])}
...
print(get.read_in('wheel_pos')) # -5738
```

Example with controls enabled.

```python
serial_in = {'wheel_pos': rotary_encoder(pins=[3,4])}
serial_out = {'wheel_pos': rotary_encoder(pins=[3,4], controls=True)} # optional
...
print(get.read_in('wheel_pos')) # -5738
get.send_out('wheel_pos', True) # reset the wheel position to 0 (only if control provided to serial_out)
```

(devices.pulse_clock)=
#### pulse_clock(pin:int, change_period:int)

`change_period:int` - period in milliseconds for the pulse clock to switch HIGH/LOW

A periodically changing HIGH/LOW clock signal will be provided at the selected pin.
This is useful to align events with external devices like neural recording hardware that may not have access to the context of the task executing python but can log changes in connected digital signals.
I.e. a pulse_clock might be created to switch its HIGH/LOW state every 1000 milliseconds and its output pin connected to a recording device. To match the external device events at behavior time 500 seconds, we now just have to look at the time of its 500th digital signal change. 

Multiple pulse clocks at different frequencies can run in parallel for redundancy, i.e. 4 clocks switching HIGH/LOW every 100ms, 1s, 10s, 5 minutes respectively.

The pulse clock is a special device that is integrated with the start_stop control and will only be counting time when the neurokraken is started/active and until it is stopped/inactive.
At inactive state all pins will be HIGH. Upon get.start() pulse clock pins start at 0.

```python
serial_in = {'clock_100ms': pulse_clock(2,     100),
             'clock_1s':    pulse_clock(3,    1000),
             'clock_10s':   pulse_clock(4,  10_000),
             'clock_10min': pulse_clock(5, 300_000)}
...
print(get.read_in('clock_1s')) # 1 => This pin is currently HIGH 1 and not LOW 0"""
```

### Controls

(devices.direct_on)=
#### direct_on(pin:int, start_value:bool=False)

`start_value:bool` : The initial value of the pin. Defaults to False (LOW).

Directly turn a pin HIGH (i=1) or LOW (i=0) upon get.send_out(<device_name>, i)

This can be used to control valves, buzzers, LEDs, ... anything you can control with a electric logic signal. These devices can also be time-limited activated with the timed_on device as an alternative.

```python
serial_out = {'odor_valve': direct_on_off(pin=3)}
...
get.send_out('odor_valve', 0) # turn the signal off / shut down the valve/LED/buzzer/...
```

(devices.timed_on)=
#### timed_on(pin:int)

Similar to direct_on_off for the same devices, but only turns the digital signal HIGH for the provided milliseconds upon get.send_out(<device_name>, <milliseconds>) before turning the signal LOW again. This can be used to easily provide well timed stimulus lengths and reward sizes. The providable millisecond range is 0 to 65536.

```python
serial_out = {'reward_valve': timed_on(pin=3)}
...
get.send_out('reward_valve', 60) # open the valve for 60 milliseconds
```

(devices.servo)=
#### servo(pin:int)

A servo motor to dynamically move experiment elements upon get.send_out(<device_name>, angle)
The angle value can range from 0 to 255.
This device can also be used to control the strength of an LED.
The pin has to among 0-15, 18-19, 22-25, 28-29, 33, 36-37 

```python
serial_out = {'servo': servo(3)}
...
get.send_out('servo', 100)
```

(devices.tone)=
#### tone(pin:int)

Set the tone frequency to be output by a passive buzzer or piezo device.
This device uses the arduino tone function. Send 0 to not provide a tone.
The maximum providable frequency is 65_535.

```python
serial_out = {'frequency': tone(pin=3)}
get.send_out('frequency', 500)
```

(wiring-guides)=
## Wiring guides

The following diagrams show the wiring for common components in neuroscience experiment setups. As we are using common arduino parts and circuits, you can generally find the wiring for new components beyond what we have listed by web image searching for `arduino circuit <device_name>`.

### 12V Power

Many devices (valves, servos) will require more Volts and/or Amps than the teensy's 3.3V logic provides to operate. For this you can simply buy a "12V power supply" from common stores like amazon and cut through its cable. Inside the cable you will find 2 wires, red (12V+) and black (GND) that you can directly use in your circuit as the required power source and its respective ground.

### LED

````{mermaid}
flowchart LR;
subgraph teensy4.1
outputPin["output pin"]
GND
end

subgraph LED
LED+
LED-
end

outputPin --220Ohm--> LED+
LED- --> GND
````

### Photoresistor

For the analog pin use one of the beige Axx pins shown in the teensy 4.1 pinout at https://www.pjrc.com/store/teensy41.html

````{mermaid}
flowchart TB;
subgraph teensy4.1
analogPin["analog pin"]
v3_3["3.3V"]
GND
end

subgraph Photoresistor
photoresistor+
photoresistor-
end

v3_3 --> photoresistor+
photoresistor- --10kOhm--> GND
photoresistor- --> analogPin
````

### Beam break/photo interrupter

You can find beam breaks or photo interrupter components that combine an LED and a photoresistor in one device

````{mermaid}
flowchart TB;
subgraph teensy4.1
analogPin["analog pin"]
v3_3["3.3V"]
GND
end

subgraph LED
LED+
LED-
end

v3_3 --220Ohm--> LED+
LED- --> GND

subgraph Photoresistor
photoresistor+
photoresistor-
end

v3_3 --> photoresistor+
photoresistor- --10kOhm--> GND
photoresistor- --> analogPin
````

### Potentiometer

````{mermaid}
flowchart TB;
subgraph teensy4.1
analogPin["analog pin"]
v3_3["3.3V"]
GND
end

subgraph Potentiometer
signal["signal (center)"]
end1["end 1"]
end2["end 2"]
end

v3_3 --> end1
end2 --> GND
signal --> analogPin
````

### Valve

A Valve will require a control pin and an additional power supply, i.e. 12V. The easiest approach is to buy a 4 channel valve driver board that can directly be connected to your output pin, the power V+ and GND, and the valve+/- itself. The shown board can be found on sites like amazon. Notably, to ensure proper operation with a 3.3V device like the Teensy, you have to desolder/solder over the small LED as shown in the image.

#### 4 Channel Valve Driver Board

This board has 4 lanes A, B, C, D where the signal at A drives the opposite valve A. You can go beyond this circuit to connect 4 output pins with 4 lanes to 4 valves.

````{mermaid}
flowchart TB;
subgraph teensy4.1
output["output pin"]
v3_3["3.3V"]
GND
end

subgraph power supply 12V
power_12V+
power_Gnd["GND"]
end

subgraph Valve
valve+["Valve +"]
valve-["Valve -"]
end

subgraph valve driver board

subgraph power
board_12v["12V+"]
boardpow_GND["GND"]
end

subgraph lane A
subgraph Signal["Signal Side"]
board_a_signal["Signal +"]
board_a_gnd["Signal GND"]
end
subgraph Valve_side["Valve Side"]
board_a_valve+["Valve +"]
board_a_valve-["Valve -"]
end
end

end

power_12V+ --> board_12v
boardpow_GND --> power_Gnd

output --> board_a_signal
board_a_gnd --> GND
board_a_valve+ --> valve+
valve- --> board_a_valve-
````

#### Manual Valve Circuit

The following circuit manually builds a lane of the valve driver board above allowing you to drive a single valve with common electronics parts.

````{mermaid}
flowchart TB;
subgraph teensy4.1
outputPin["output pin"]
GND
end

subgraph IRLZ44
IRLZ44Left
IRLZ44Center
IRLZ44Right 
end
outputPin --1kOhm--> IRLZ44Left
Valve- --> IRLZ44Center
IRLZ44Right --> 12VGND 
IRLZ44Center -.- IRLZ44Right

subgraph Valve
12V+ --- Capacitor
Capacitor --- 12VGND["12V GND"]
12V+ --> Valve+
Valve+ --> Valve-
Valve- --> Diode["Diode\n(1n4007)"]
Diode --> Valve+
end
12VGND --- GND
````

The capacitor stores charge to support the high current draw required when the valve is activated.

When the valve solenoid gets powered this will induce movement of its piston. Conversely, when the piston springs back upon loss of power this induces a voltage. The diode will feed back this voltage to prevent it from ending up in the microcontroller.

### Rotary Encoder

There are 2 common types of rotary encoder - both will have 4 wires for black GND, red 3.3V, and 2 signals A and B (i.e. in green and white): The more common pull-up Rotary encoder, and a more uncommon typically expensive direct rotary encoder.

#### Pull-up Rotary Encoder

The most common rotary encoders often don't provide voltage on their signal pins, but depending on their state short the respective signal pins to GND or not and can wired like this.

````{mermaid}
flowchart TB;
subgraph teensy4.1
GND
3.3V
inputPinA["pin A"]
inputPinB["pin B"]
end

subgraph Rotary Encoder
encV["V+"]
encGND["GND"]
encA["A"]
encB["B"]
end

3.3V --> encV
encGND --> GND
3.3V --4.7kOhm--> inputPinA
inputPinA --> encA
3.3V --4.7kOhm--> inputPinB
inputPinB --> encB
````

These signals can be read by connecting them to Teensy input pins with pull-up resistors to 3.3V. If the encoder shorts the signal to GND, the pin reads LOW; otherwise, the 3.3V will make its way to the teensy pin, which will be pulled-up to HIGH (3.3V).

#### Direct Rotary Encoder

This type of encoder is generally more expensive, sold by specialized electronics stores, maybe more compact and can have additional optional wires.

This is the circuit for a 5V rotary encoder whose signal pins switch between 5V and 0V during rotation. If your rotary encoder functions at 3.3V you can use 3.3V and leave out the voltage divider resistors.

````{mermaid}
flowchart TB;
subgraph teensy4.1
5V
GND
3.3V
inputPinA["pin A"]
inputPinB["pin B"]
end

subgraph Rotary Encoder
enc5V["5V"]
encGND["GND"]
encA["A"]
encB["B"]
end

A(( ))
B(( ))
5V --> enc5V
encGND --> GND
encA -- 10kOhm --> A
encB -- 10kOhm --> B
A -- 20kOhm --> GND
B -- 20kOhm --> GND
A --> inputPinA
B --> inputPinB
````

The manual should indicate which wires correspond to V+, GND, A, and B. As the teensy should only receive 3.3V not the 5V the rotary encoder delivers, we use 2 stages of resistors to lower the A and B voltage from 5V to 3.3V and then 0V/GND. We can then connect the 3.3V stages to our reading pins. (Voltage Dividier)

### Servo

For the servo signal wire, use a pin designated for PWM on the Teensy pinout in https://www.pjrc.com/store/teensy41.html

````{mermaid}
flowchart TB;
subgraph teensy4.1
outputPin["PWM output pin"]
GND
end

subgraph power supply 12V
power_12V+
power_Gnd["GND"]
end

Capacitor
Diode["1N4007 diode"]

subgraph Servo_Motor
servo_power["Power +"]
servo_signal["Signal"]
servo_gnd["Ground"]
end

power_12V+ --- Capacitor
Capacitor --- power_Gnd

power_12V+ --> servo_power
servo_power --> Diode
Diode --- power_Gnd

outputPin --> servo_signal
servo_gnd --> power_Gnd
servo_gnd --> GND
````

### Tone Passive Buzzer

````{mermaid}
flowchart LR;
subgraph teensy4.1
outputPin["output pin"]
GND
end

subgraph Buzzer
Buzzer+
Buzzer-
end

outputPin --> Buzzer+
Buzzer- --> GND
````

### Capacitive Touch Sensor

This circuit uses a capacitive touch sensor module available on common stores like amazon.

````{mermaid}
flowchart TB;
subgraph teensy4.1
sensorPin["sensor pin"]
v3_3["3.3V"]
GND
end

subgraph Capacitive Touch sensor
module_VCC["VCC"]
module_GND["GND"]
module_SIG["SIG"]
end

v3_3 --> module_VCC
module_GND --> GND
module_SIG --> sensorPin
````

### Pulse clock

````{mermaid}
flowchart TB;
subgraph experiment_teensy["Experiment Teensy"]
clock_pin["clock pin"]
GND
USB
end

subgraph external_device["External Device"]
external_input["input pin"]
external_GND["GND"]
end

clock_pin -- 10kOhm --> external_input
GND --- external_GND
PC --- USB
````
