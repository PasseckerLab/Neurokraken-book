# Quickstart

## A minimal starting task

- Follow the [Quick Install Instructions](installation) to clone the codebase into a local folder, create a python environment and install the Neurokraken python package
  - To go beyond keyboard mode and interact with electronics also install the Arduino IDE and in it the teensyduino addon.
- Within the codebase you will find a folder tasks/examples which contains example tasks that can help you get started with different neurokraken use cases.
- For a minimal Neurokraken task create a new .py file and add the following. We will cover elements and ways to add to it in the next steps.

```python
# setup configuration
from neurokraken import Neurokraken, State
from neurokraken.configurators import devices, Display, Camera, Microphone

serial_in = {}
serial_out = {}

nk = Neurokraken(serial_in=serial_in, serial_out=serial_out, log_dir='./', mode='keyboard')

# task design
from neurokraken.controls import get

class My_State(State):
    def loop_main(self):
        return False, 0
    
task = {
    'state_name': My_State(next_state='state_name'),
}

nk.load_task(task)

nk.run()
```

- You can directly run this python script - Since we are using keyboard mode we don't need to connect a teensy.
- `>>> python my_task.py`
- As its loop_main is empty (outside of the return) this task will continue doing nothing until manually closed.

### Extend your task

- In this simple example we want to extend the minimal task to reward a subject for poking a sensor.
  - After a reward there will be 10 seconds delay until the sensor will be responsive again
  - The Responsive state is indicated by a LED light.

```python
# setup configuration
from neurokraken import Neurokraken, State
from neurokraken.configurators import devices, Display, Camera, Microphone

serial_in =  {'light_beam': devices.analog_read(pin=3, keys=['s', 'w'])}
serial_out = {'reward_valve': devices.timed_on(pin=2),
              'LED': devices.direct_on(pin=3)}

nk = Neurokraken(serial_in=serial_in, serial_out=serial_out, log_dir='./', mode='keyboard')

# task design
from neurokraken.controls import get

class Poke_for_reward(State):
    def loop_main(self):
        if get.read_in('light_beam') > 512:
            get.send_out('LED', True)
            return True, 0
        else:
            get.send_out('reward_valve', 100)
            get.send_out('LED', False)
            return False, 0

task = {
    'poke': Poke_for_reward(next_state='delay'),
    'delay': State(next_state='poke', max_time_s=10)
}

nk.load_task(task)

nk.run()
```

---

**Task walkthrough:**

1. We added the new electronic devices to serial_in and serial_out
    - In simulated keyboard-mode, the light sensor input is controlled with LOW/HIGH values from keyboard keys *S / W*
---
2. loop_main() got extended to read the light sensor
    - if the light sensor is reading **>512** => not touched:
        - return that the state is not finished to enter the next loop iteration `return False, 0`
        - keep the LED on
    - else if the light sensor is reading **<=512**:
        - open the reward valve for 100 milliseconds.
        - turn the LED off until this state is reentered.
        - return that the state is finished and should move on to the 0th (or here single) possible next_state: `return True, 0`
---
3. Extend the task with a delay state
    - *poke* will lead to *delay* on touch, otherwise run forever.
    - *delay* is the un-extended default State, where nothing will change until it will timeout after 10 seconds and transition back to *poke*

---
```{note}
Combine `configurators`, `cameras`, `displays`, `devices`, `.get` access to `devices`, the `log` and camera frames together with `States` and their `loop_main()`, `on_start()`, `on_end()`, `loop_visual()` to create your experiment condition of interest.
```

### Run with real devices

1. Connect your electronics to the teensy according to the [device library wiring guide](device_library)
    - i.e. for the task above 1 light sensor, 1 LED, and 1 reward valve.
2. Run the neurokraken repository's `config2teensy.py`. It will ask you to drag and drop your task task .py script into the console before pressing enter.
    - The repository's `teensy` folder now contains ready-to-upload arduino code for your task through an automatically created/updated `Config.h` file.
    - Devices added to your scripts serial_in and serial_out dictionaries are automatically included.
3. With the arduino IDE open the teensy folder/its `teensy.ino`, install potential missing libraries for your devices, USB connect the teensy and <u>press Upload</u> in the IDE.
4. In your script change the Neurokraken's `mode='keyboard'` into `mode='teensy'` and run your task script. 
    - Your peripheral controls and sensors are now logged and accessed from your task python code 

### Where to next

[Examples](examples) covers more examples across the range of neurokraken applications.

[Controls](controls) provides a good introduction to task development and the reference to `get` the central interface element to neurokraken's managed elements.

Depending on your specific use case you can also dive deeper into the reference documentation for [States](states), the [Configuration](configuration), or the [Device Library](device_library)
