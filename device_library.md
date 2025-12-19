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

Conversely some components can be used with different neurokraken.devices. For example our LED can be a `devices.direct_on(pin:int)` or a `devices.timed_on(pin:int)` that would default stay LOW but allow us to `get.send_out('led', 100)` to turn HIGH for a duration of exactly 100 milliseconds before turning LOW again.

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

[The add your own device page](middleman-arduino) has an example of adding an arduino as a teensy connected device
(modalities)=
## Modalities

(configurators-devices)=
## Configurators.devices

(wiring-guides)=
## Wiring guides

