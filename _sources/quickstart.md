# Quickstart

## Installation

````{Dropdown} installation instructions using python conda
<!-- :open:  -->

install the JDK from [Adoptium JDK download](https://adoptium.net/) as a requirement for the python py5 package.

- Ensure "Add to PATH" and "Set JAVA_HOME variable" options are checkmarked during install.
- If successful, running `java --version` in a new console will confirm the Java installation

Move a command line into a folder where you want to set up the Neurokraken codebase. Then run the following commands to create a conda environment and install required packages into it.

```bash
git clone https://github.com/PasseckerLab/neurokraken
cd neurokraken
conda create --name neurokraken python==3.13
conda activate neurokraken
pip install .
```

You are now generally ready to run and develop Neurokraken tasks using your new neurokraken environment.

---

To run tasks with connected actual electronics rather than just in keyboard/agent mode you further need to install the arduino IDE with the teensyduino add-on. This allows you to upload neurokraken's auto-generated arduino code to your teensy microcontroller.

- [The arduino IDE](https://www.arduino.cc/en/software/) 
- [teensyduino](https://www.pjrc.com/teensy/teensyduino.html)

---

For more detailed information and alternatives (how to install python, using venv or uv instead of conda, how to install git or download the codebase without git, ...) look up the [Installation instructions chapter](installation)
````

To progress you should have a cloned or downloaded copy of the Neurokraken codebase from github.com/PasseckerLab/Neurokraken and set up a python environment with the package dependencies installed.

## Run examples

Within the codebase you will find a folder `examples` which contains example tasks that can help you get started with different neurokraken use cases. Examples are documented in [the examples chapter](examples) and within their code use `mode='keyboard'` allowing them to directly run without connected electronics.

```bash
cd examples
conda activate neurokraken # if using conda activate the environment
python corridor_3d.py
```

In the following sections we will walk through how to develop a simple task from the beginning. In the end we cover how to leave keyboard mode and run the task with actual connected electronics and where to go next to explore neurokraken's capabilities.

## Create a minimal starting task

For a minimal Neurokraken task create a new .py file in a directory of your choice and add the following. We will cover elements and ways to add to it in the next steps.

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

## Extend your task

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

## Run with real devices

You should have a cloned or downloaded copy of the neurokraken codebase from https://github.com/PasseckerLab/Neurokraken on your computer. Two components of this folder are relevant to autocreate and upload your task's microcontroller side code, `config2teensy.py` to create the code, and the `teensy` folder containing the resulting code ready to upload with the arduino IDE.

1. Connect your electronics to the teensy according to the [device library wiring guide](device_library)
    - i.e. for the task above 1 light sensor, 1 LED, and 1 reward valve.
2. Run the neurokraken repository's `config2teensy.py`. It will ask you to drag and drop your task task .py script into the console and then press enter.
    - The repository's `teensy` folder now contains ready-to-upload arduino code for your task through an automatically created/updated `Config.h` file.
    - Devices added to your scripts serial_in and serial_out dictionaries are automatically included.
3. With the arduino IDE open the teensy folder/its `teensy.ino`, install potential missing libraries for your devices, select the teensy's USB connection in the dropdown menu at the top of the IDE and <u>press Upload</u> in the IDE. You should see the teensy's built-in orange LED flickering and then remaining on while the IDE console confirms a successful upload.
4. In your script change the Neurokraken's `mode='keyboard'` into `mode='teensy'` and run your task script. 
    - Your peripheral controls and sensors are now logged and accessed from your task python code 

## Where to next

[Examples](examples) covers more examples across the range of neurokraken applications.

[Controls](controls) provides a good introduction to task development and the reference to `get` the central interface element to neurokraken's managed elements.

Depending on your specific use case you can also dive deeper into the reference documentation for [States](states), the [Configuration](configuration), or the [Device Library](device_library)
