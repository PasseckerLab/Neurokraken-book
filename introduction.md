# Neurokraken Introduction and Overview

<!-- ```{image} ./images/neurokraken.png
:alt: The neurokraken
:width: 400px
:align: center
``` -->

```{figure} ./images/neurokraken.png
---
height: 400px
name: directive-fig
---
Many arms for all your experiment tasks, devices and controls 
```

## A fully flexible, open-source, python-based behavior platform for the extended neurosciences

Neurokraken is a new open and flexible easy-use behavior platform enabling easy pursuit of new experiment designs.

## Features

- 🦑 Tasks in unrestricted standard python with the python package ecosystem available for usage
- 🦑 Arduino code automatically created from the python code, ready to upload and run
- 🦑 keyboard and agent mode to develop and run tasks in simulation
- 🦑 Easy to extend for new applications
- 🦑 automatic high-precision logging of task events and sensors
- 🦑 cameras, microphones, displays, electronic components, ...
- 🦑 live data and interactions

## Where to start

Depending on your preference there are 3 good places to start

- [Quickstart](quickstart) for a quick start with a minimal example
- [Examples](examples) for examples ranging from minimal to 3D environments, live-tracking and artificial agent subjects
- [Overview](overview) below on this page covers a comprehensive list of neuroraken features to explore

## About the Kraken

Neurokraken is free open-source software developed at the [Passeckerlab](https://lab.jpassecker.com/) by Alexander Wallerus. Please feel to use it in your research, get in touch, and contribute to the project by joining discussions on github.

---

(overview)=
## Neurokraken Overview

Neuroscience experiments require a wide range capabilities. After all, we want to replicate specific experiences from the breadth of our lives and of our capabilities to investigate how the brain's processes these situations.

As a result Neurokraken contains a wide range of abilities listed below, beyond what is needed by any individual experiment - however all you need to get started is a minimal State with a loop_main() as shown in our [quickstart](quickstart) and [examples](examples)

The following overview summarizes neurokraken capabilities that may become useful to your task while reserving the details for the respective documentation pages.
 
Your experiment may not need a microphone or benefit from from a multi-state sequence organization. Start simple, and when a given additional component becomes relevant, i.e. adding a display to show your subject custom visuals, explore the corresponding documentation and examples.

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

### [FAQ](faq)

Frequently asked questions
