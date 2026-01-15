# Integrating Outside Devices

## Time Alignment between neuroscience devices - TTL pulse signal for time synchronization

A common difficulty in neuroscience is that experiments may require many individual devices that without a ground truth time signal would have no way to later time-align their respective data for analysis. Experiments might use a behavior setup, a data aquision unit, one or more electrophysiological recording devices, TTL-triggerable cameras,... Often an additional device is brought in to create a ground truth synchronization TTL signal for all of these devices.

Neurokraken automatically saves timestamped sensor and control events removing the need for a separate data aqusision unit. Additionally digital or analog signals from outside devices can be added and connected to neurokraken as a devices.digital_read and devices.analog_read to log millisecond precision event timestamp of respective devices.

Neurokraken's device libary also includes devices.pulse_clock(\<pin\>, \<change_period_millis\>) which can create alignment time signals to connect to outside devices without the need for adding specialized signal generation equipment. For example one might run a neurophysiological recording with a pulse 100ms pulse clock connected to the recording device. A task could also include multiple pulse clocks, i.e. changing state at 100ms, 1s, 10s, and 5minute intervals.

A devices.pulse_clock could also be used to trigger TTL devices like specialized cameras.

Pulse clock pins are HIGH until begin of the task, will switch to LOW at time 0 and then repeatedly change state after the provided interval. Internally devices.pulse_clock is linked to neurokraken's start() and stop()/quit() controls allowing you to i.e. configure neurokraken with `autostart=False` to postpone the start of your task states and pulse clock signals until get.start() and then use get.stop() or get.quit() to end your task at which time all pulse clock pins switch to HIGH. This creates a clear experiment interval in connected devices with direct timestamps of the corresponding experiment time being 0ms, 100ms, 200ms,... when for example using a 100ms pulse clock.

Neurokraken's toolkit/performance_test folder contains utilities and instructions to confirm the precision of your created pulse clock signal.

## USB devices

### Devices with python packages/APIs
Some categories of USB devices (cameras, microphones, ...) are in categories neurokraken possesses integrated interfaces for. 

Other devices may possess python packages that allow interactions which can be integrated into your task code for usability. (Our integration of cameras and microphones actually also use python packages; cv2, harvesters, and sounddevice to facilitate neurokraken's access to the respective devices)

### Additional Microcontrollers - Arduino, ESP, ...

A common example of microcontroller information exchange is through serial communication using the arduino IDE console. The python package pyserial can allow you to move the same communication from the arduino IDE into your python code. A task could besides its neurokraken-managed teensy also exchange information with several UBSB-connected arduinos.

A common approach for this in neurokraken is to start a communication loop before neurokraken.run() which can be seen within our cyclops example. Cyclops is a teensy-based optogenetics control device that can be programmed with arduino and thus be serial communicated with to control the device from the experiment task code.

Alternatively to a direct PC USB connection, 2ndary microcontroller can be connected to the teensy as described in [Add your own device](extra-arduino).

### Integrating commercial devices

Some commercial devices with USB connections send their data as a simple series of values through Serial communication.

One can test this by using the arduino IDE serial monitor but instead of selecting the COM USB connection of an arduino the COM connection of the device is selected. If the received values directly reflect sensor changes, i.e. they show clear HIGH/LOW states or continously update one or two position numbers, integration is straightforward.

The python package `pyserial` can be used to read or send this data just like the arduino IDE Serial Monitor, thus allowing you to read from existing commercial devices through Neurokraken rather than having to buy or assemble new equipment, and to extend the original imited device integration with the capabilities of neurokraken.

