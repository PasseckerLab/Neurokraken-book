# Neurokraken Tools

This chapter is currently in development and will be available soon.
You can find covered elements in neurokraken's toolkit folder.


## How to run the performance tests

### benchmarking task requirements:
- the following 2 adaptations have to be made when running a task for Performacne testing:
- the task includes an explicit logged milliseconds time in its serial_in, as well as a 100ms pulse_clock, and a microseconds time.
```python
serial_in = {'t_ms': devices.time_millis(logging=True),
             'pulse_clock': devices.pulse_clock(pin=22, change_periods_ms=100),
             ...
             't_us': devices._time_micros(logging=True)
}
```
- the Neurokraken is defined with log_performance=True `nk = Neurokraken(... log_performance=True)`

### pulse-clock recipient device stand-in teensy
- upload `test_pulse_clock/test_pulse_clock.ino` to a 2ndary teensy. This teensy will validate the stability of pulse clock signals as an external device will receive them. Connect its pin 1 to the task teensy's 100ms pulse clock pin and connecte both teensys' grounds.
- before starting the task run `test_pulse_clock/test_pulse_clock.py` => Once the task/pulses start you will see the millisecond and microsecond of every received pulse which should be 100ms apart from another minus a small drift between both teensys' clocks directly visible in the microseconds. This is not an issue and expected - independent clocks being slightly out of phase from another is why we create a ground truth time signal with the pulse clock after all. After the task has been finished press Ctrl+Alt+P to save the data as `pulse_clock.csv` and create a plot of relevant metrics.

### stimulus
- As the system only logs values that have changed we have to provide artificial changing stimulation to mimick realistic experiment conditions. For this we use an ESP32S3 qt-py, though it is easily exchangeable for other 3.3V microocontrollers. (! the arduino nano outputs 5V and directly wiring 5V to teensy inputs without voltage conversion to 3.3V can damage the teensy)
- upload `stimulus/stimulus.ino` to the ESP.
- Connect its GND to the teensy GND and digital outputs to the teensy digitalRead pins. The ESP will move servos to actuate task rotary encoders. For convenience we use a PCA9685 servo driver over individual circuits for which 5V->V+, 3.3V->VCC, GND, SDA, and SCL need to be connected.
- The ESP can be powered through USB or by wiring 5V from the teensy to its 5V. Once powered it will provide digital and rotation stimuli for the task

### main teensy
- connect the teensy according to the devices connected in the respective task configuration
- run config2teensy.py and select the task .py file to create an uploadable configuration in the teensy folder. Then upload teensy.ino to your task teensy. (! When multiple teensy's are connected/disconnected it can be difficult to see which is which from the arduino IDE => the easiest way to upload to the pulse-clock or task teensy specifically is to disconnect the other one)
- after starting test_pulse_clock.py run your task .py file. 

### test_pulse_clock

- The freedom to use any arduino library in your experiment also can mean the danger of this library halting your important experiment functions
- Most libraries won't require enough compute to delay your pulse clock meaningfully. However since the synchronization pulse signal is vital for aligning the time axis with other experiment devices like electrophysiological recordings, it is a good idea to confirm the consistency of your clock signal when using arduino libraries that are likely to be compute heavy.
  - Compute heavy devices can be outsourced to an additional connected microcontroller should this situation occur.
- To test your pulse clock performance connect your experiment teensy to a 2nd data collection teensy as shown in the following circuit.
- This 2nd teensy will listen for changes in the pulse clock signal, mirror it on its built-in LED, and upon a change send its own clock's millisecond and microsecond readings to python
- Connecting a 100ms pulse Clock to the first pin 3 will give a good idea of whether the the clock quality is stable.
- Over time the single digit position of your logged pulse microseconds can be observed to have a constant increase or decrease. This is expected and a result of small variations in the clock crystals between the first and 2nd teensy. Drifts like these are why multi-device experiments use a shared ground truth signal for aligning time intervals.
  - as an example, the us column of the running code might show a drift like this with a 100ms clock: `3194432414, 3194532414, 3194632414, 3194732413, 3194832413, 3194932413, 3195032413, 3195132412, 3195232412, 3195332412, 3195432411, 3195532410, 3195632410, 3195732410`
- To run a test start run test_pulse_clock.py and then start an experiment task. After enough data has been gathered stop your experiment and press Ctrl+Alt+P to see a plot visualization of the microsecond differences between pulses
- Variations on the scale of several microseconds generally won't be experiment-relevant. However, when you plan to interpolate your ephys/behavior time using 100ms clock pulses, 100ms should last around 100ms.

#### Example circuit: Connecting a clock pin 2 to a test teensy

````{mermaid}
flowchart TB;
subgraph experiment_teensy["Experiment Teensy"]
100msclock["clock pin 2\n100ms"]
GND
USB
end

subgraph clocktest_teensy["Clock Test Teensy"]
clocktestInput["input pin 3"]
clocktestGND["GND"]
clocktestUSB["USB"]
end

100msclock --> 10kOhm
10kOhm --> clocktestInput
GND --- clocktestGND
PC --- USB
PC --- clocktestUSB
````
