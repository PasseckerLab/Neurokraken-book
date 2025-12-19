# Quickstart/Overview

(quickstart)=
## Quickstart

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

---

(overview-list)=
## Neurokraken Overview

Neuroscience experiments require a wide range capabilities. After all, we want to replicate specific experiences from the breadth of our lives and our capabilities to investigate how the brain's processes these situations.

As a result Neurokraken contains a wide range of abilities listed below, beyond what is needed by any individual experiment.
 
This however is nothing to be intimidated by - all you need to get started is a minimal State with a loop_main() as shown above. 
 
Your experiment may not need a microphone or benefit from from a multi-state sequence organization. Start simple, and when a given additional component becomes relevant, i.e. adding a display to show your subject custom visuals, explore the corresponding documentation and examples.

The following overview summarizes neurokraken capabilities that may become useful to your task while reserving the details for the respective documentation pages.

If you encounter any questions, please feel free to join our github discussions. Neurokraken is developed to be an easy and helpful toolkit and community for neuroscience research.

```{contents}
:local:
```

### [Controls](controls)

Your task code's main interface to the system - Neurokraken's live status, controls and interactions:
- Read sensors
- Control devices
- The log dictionary
    - Autologging sensor/task updates to its history
    - Accessible and extendable for in-task usage
- Camera frames
- Task controls: start/stop/active/quit/quitting
- State/Block overrides

### {doc}`configuration`

Used in 3 stages to define and run a task: Initialization, .load_task(), .run()
- initialization: The Computer and task electronics setup
- same task can run with different configurations.
- serial_in: arduino-side sensors, experiment accessible with `get.read_in("<device_name>")`
- serial_out: arduino-side controllable peripherals, experiment accessible with `get.send_out("<device_name>", <value>)`
- Adding one or more displays to your task
- Cameras and Microphones
- log_folder

### {doc}`states`

States can be used to guide your experiment flow in a well compartimentalized easy to develop way. 
Alternatively your task can be organized as a single eternal state.

- loop_main, on_start, on_end
- loop_visual - adding custom interactive visuals to your task with py5
- state flow: state finished, next_state= and progress_state()
- copy and modify states for distinct purposes
- adding callbacks: run_at_start, run_at_end, permanent_states
- using trials and live-analysis
- blocks organize seperate state flows and switching between within one task.

### [device_library](device_library)

Documented electronic peripherals for neuroscience tasks
- Neurokraken.configurators.devices
- Self-creating arduino-side code
- Direct simple access from python
- Combinable and remixable according to task needs
- Example applications
- Arduino wiring guides
- Extendable

### {doc}`cameras`

Neurokraken works with standard USB cameras as well as advanced laboratory grade cameras
- live frame access for preview or live-processing
- automatic frame time logging
- configurable resolution and target fps

### [microphones](microphones)

Microphones connected to your PC can be used to record experiment audio.  This also works with 200khz specialized microphones.

### [modes](modes)

Special modes for neurokraken:
- keyboard mode: Develop tasks without connected teensy/electronics using keyboard keys as inputs
- agent mode: Run tasks with a script controlled agent/AI instead of a physical subject

### [examples](examples)

A range of examples within the repository's tasks/examples covering a wide range of applications
- steering wheel
- olfactory stimulus decision
- live-AI video tracking
- reinforcement learning agent
- 3D blender created corridor
- VIZDOOM reinforcement learning package
- python eye-tracking package
- Pong
- joystick game
- User Interfaces
- USB device optogenetics control
- ...

### [common_patterns](patterns)

Example code snippets for common task use cases: 
- UI overrides, logging text from a UI, scheduling repeated actions, loading subject information before task start,...

### [AI/ML/CV](ai_ml_cv)

Running high compute data processing as part of neurokraken tasks
- computer vision, machine learning, AI, ...
- Cutils Examples: live position tracking using Cutie

### [UIs](UIs)

Add a window to display and plot experiment data, camera views and override experiment variables

### [tools](tools)

toolkit: self-contained utilities for neurokraken tasks:
- config2teensy.py: autogenerate teensy/arduino code from a task script using its serial_in/serial_out 
- Cutils: functions for automated cutie tracking using folders or live numpy image arrays
- Neurokraken performance testing
- valve drop size/opening time calibration

tools:
- experiment helper functions

### [runner mode](runner_mode)

A runner for laboratories with an extensive number of tasks and variations
- arguments to select task, device configuration, experiment, subject, mode, ask for datapoints on startup, skip log creation, ...
- useful modularization and control for advanced tasks

### [outside devices](outside_devices)

Integrate USB devices and existing neuroscience behavior equipment into neurokraken tasks

### [installation](installation)

Quick install and detailed installation instructions

### [building_your_setup](building_your_setup)

- Hardware library: 3D printed modular elements for custom neuroscience experiment setups
- Device assembly instructions: Printable XYZ stage, odor delivery, ...
- Building a behavior setup walkthrough
- Table of affordable and useful components for different neuroscience setups

### [arduino_custom](arduino_custom)

Adding a new custom self-made addition to the device library 

If you are interested in adding a new device and/or its Arduino code to your task, this is the place to start.

### [for_developers](for_developers)

How the Neurokraken works - useful for hacking-in new functionalities.
- the networker: Automatic live updating of python side sensor input, automatic logging, execution of python side controls on arduino.
- the main loop and other threads. Internal timing of processes

### [troubleshooting](troubleshooting)

Some issues like Arduino's too-large windows temp folder are hard to recognize. Here we gather known difficulties and workarounds



