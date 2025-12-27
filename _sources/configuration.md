# Configuration - The Neurokraken Object

A minimal empty task that also contains imports for optional components and forever runs an empty state named "state_name" looks like the following: 

```python
from neurokraken import Neurokraken, State
from neurokraken.configurators import Display, Microphone, Camera, devices
from neurokraken.controls import get

nk = Neurokraken(serial_in={}, serial_out={}, log_dir='./', mode='keyboard')

nk.load_task('state_name': State(next_state='state_name'))

nk.run()
```

In the first step we create a Neurokraken object here named `nk`. In the following part we can add our experiment logic using states and run the task.
The Neurokraken object manages communication with hardware components including serial interfaces, camera systems, and data logging. It handles task execution, real-time data streaming and setup configuration management.

The setup configuration is defined during the creation of the Neurokraken object through respective arguments, including involved components (displays=, cameras=, microphones=, arduino-side electronics serial_in=/serial_out=) and options for how the task should be run (run_mode=, autostart=, log_dir=, ...)

The topics of arduino side devices `serial_in` and `serial_out` can be found in the [device library chapter](device_library), while [cameras](cameras.md) and [microphones](microphones) also are documented in their own chapters.

**Audio Stimuli**

Neurokraken does not have a built-in approach to audio stimuli, however python features a range of packages capable of playing sound files, frequencies and series from chosen computer speakers. Tasks can be extended with audio components by utilzing/calling functions of these packages at relevant times from your main loop. As some implementations of live audio may be blocking the flow of your code, it is reccommended to wrap their usage in a python Threading.thread if you encounter delays in your main loop with a given audio package of interest.

<!-- ```{contents}
:local:
:depth: 1
``` -->

## serial_in :dict

A dictionary of named arduino-side sensors used in the task. Common sensor options are available in `configurators.devices`.
If no arduino side sensors are used you can provide an empty dictionary `serial_in={}`.

```python
from neurokraken.configurators import devices

serial_in = {
    'steering': devices.rotary_encoder(pins=(31, 32), keys=['down', 'up'])
}

nk = Neurokraken(serial_in=serial_in, ...)
```

## serial_out :dict

A dictionary of named arduino-side controlled devices used in the task. Common options are available in `configurators.devices`
If no arduino side controls are used you can provide an empty dictionary `serial_out={}`.

```python
from neurokraken.configurators import devices

serial_out = {
    'reward': devices.timed_on(pin=40),
}

nk = Neurokraken(..., serial_out=serial_out, ...)
```

(display)=
## display :configurators.Display

Settings for the Subjects view using a confugrators.Display class

```python
display=Display(size=(800, 600))

nk = Neurokraken(..., display= display)
```

### size :(int,int)

the width and height of the window in pixels. Defaults to `(800, 600)`.

Depending on your system drawing many changing pixel can be a surprisingly large sink for your system's performance. If you want to optimize compute requirements of your task and your subjects don't require HD you can switch to a lower resolution.

For this in your windows settings -> display settings you can set your subject monitor's resolution from something high like 1k or 2k to something lower like (800, 600) and match the resolution in the display configuration.

```python
nk = Neurokraken(..., Display(size=(800, 600)) )
```

This resolution will be used for the loop_visual py5 sketch, meaning that in this case `sketch.ellipse(0, 0, 10, 10)` would draw a 10 pixel ellipse on the top left end of the window and `sketch.ellipse(790, 0, 10, 10)` would draw an ellipse on the top right end of the window.

To span multiple displays you can also choose a size larger than a single display like `size=(1600, 600)`

### position :(int,int)

The position relative to the top left corner of your primary display. Defaults to (0, 0).

On windows can find the organizations your displays in the Windows Display Settings.
For example when you have a 2ndary 800x600 monitor for your subject set directly to the left of your main display, you can move the position this 800 pixels width to the left to fit your task visuals perfectly onto that display.

```python
nk = Neurokraken(..., Display(size=(800, 600), pos=(-800, 0)) )
```

### borderless :bool

Whether the visual window should be displayed without a taskbar, i.e. as a fullscreen visual. Defaults to False.

`borderless=False` can be useful to have a window that can be dragged around when testing.

`borderless=True` is useful in the applied task to provide a focused task visual.

```python
nk = Neurokraken(..., Display(size=(800, 600), pos=(-800, 0), borderless=True) )
```

### renderer :str

The rendering backend to use by py5. One of `'JAVA2D'`, `'P2D'` and `'P3D'`. Defaults to 'JAVA2D'.

To enable drawing 3D environments `renderer='P3D'` is required.

```python
display = Display(renderer='P3D') 
```

## log_dir :str|None

The director where the log will be saved.

A log folder will be created at this location and contain the session's log.json as well as video and audio files from camera and microphone captures. 

Use `log_dir=None` to not save a log.

Defaults to the current folder `'./'`.

During the task you can find the used log dir with `get.log_dir` to manually add additional files of interest.

In [runner mode](runner_mode) the save folder will always be the /logs directory of your cloned codebase. Runner mode will also copy a backup of your executed task code to the log_folder.

```python
# example directories
log_dir = './'                          # current folder
log_dir = 'C:/path/to/my/save/folder'   # specified absolute folder
log_dir = None                          # don't save a log

nk = Neurokraken(..., log_dir=log_dir)
```

## cameras :configurators.Camera|list

The camera or list \[ \] of cameras used to capture the task using configurators.Camera.
You can either provide a single Camera instance or a list of instances when using multiple cameras.
Cameras are documented in the [cameras chapter](cameras.md). Defaults to [ ]

```python
from utils.configurators import Camera

cameras = Camera(name='my_camera', idx=0, width=1280, height=720, max_capture_fps=60)
nk = Neurokraken(..., cameras=cameras)
```

## microphones :configurators.Microphone|list

The microphpne or list \[ \] of microphones used to capture the task using configurators.Microphone.
You can either provide a single Microphone instance or a list of instances when using multiple microphones.
Cameras are documented in the [microphones chapter](microphones). Defaults to [ ]

```python
from utils.configurators import Microphone

microphone = Microphone(name='my_microphone', idx=0)
nk = Neurokraken(..., microphones=microphone)
```

## mode :str

Operating mode (`'teensy'`, `'keyboard'` or `'agent'`). Defaults to `'teensy'`

In teensy mode neurokraken will search for a USB connected device identifying itself as `KRAKEN` (or a custom set other [serial_key](serial_key) to run multiple neurokraken and their electronics in parallel on the same computer)
This is the dominant way to execute experiments in the real world. However trying to run this mode without a teensy connected will result in an error message.

Keyboard mode can be used to run tasks without any connected teensy and with keyboard inputs replacing the sensors respective serial_in entries. This mode is useful for flexible task development and testing.
Similarly agent mode allows you to automate providing task inputs like a subject or keyboard would. This can be useful for automated testing or research using AI agents as subjects.
keyboard and agent mode are covered in detail in the [modes](modes) chapter

```python
nk = Neurokraken(..., mode='keyboard')
```

## agent :class|None

A class with a def act() method to run when `mode='agent'`

(serial_key)=
## serial_key :str

Serial communication key identifier. Defaults to `'KRAKEN'`

In teensy mode you can run multiple neurokraken instances with respective teensy and electronics in parallel on the same computer. For this each Neurokraken needs to be able to identify its respective teensy. By default this identifier is `'KRAKEN'` and is provided with the teensy upload of the code.

To update this identifier you have to update it both here in the string provided to the Neurokraken object and in the teensy code before uploading. Within the teensy folder content to be uploaded you can find a file `_Utils.h` containing line 24 `{'K', 'R', 'A', 'K', 'E', 'N', '0', '0', '0', 0},` which defines this identifier on the teensy side.

To change a task to the identifier `SETUP2` we can edit the python and arduino side as follows. This new Neurokraken identifying its connection as `SETUP2` can then run on the same device and time as one using the original identifier `KRAKEN`.

```python
nk = Neurokraken(..., serial_key='SETUP2')
```

```cpp
{'S', 'E', 'T', 'U', 'P', '2', '0', '0', '0', 0},
```

## subject :dict|str

Subject identification information to be added to `get.log.experiment_data`. Defaults to `{"ID": "_"}`

The saved log_folder will also be named after the provided ID entry, or `Neurokraken_` if ID is `_`.

If only a string is provided the result is equivalent to providing `{"ID": "<provided string>"}`

```python
nk = Neurokraken(..., subject={'ID': 'Alpha', 'group': '7'})
```

## max_framerate :int

Maximum frame rate for the main loop. Defaults to 8000

## autostart :bool

Whether to automatically start the experiment or wait for `get.start()`. Defaults to True.

Delaying running your task time and states with `autostart=False` until a custom time can be useful for tasks requiring extended setup work, as cameras and controls.get interfaces are already available before triggering `get.start()`

Alternatively to `get.start()` you can start a `autostart=False` task with the keyboard combination `Ctrl + Alt + S`

```python
nk = Neurokraken(..., autostart=False)
``` 

## log_performance :bool

Set to True to have the main loop log iteration and networking times. Defautls to False.

Can be used to measure the impact of UIs, and live-analysis on the the speed and regularity of main loop iterations.

For the most accurate time information it is reccommended to run tasks with `mode='teensy'`

The main loop performance is separate from the teensy millisecond precision sampling which can be confirmed by adding a devices.time_millis with logging=True to your serial_in.

## runner mode arguments

Internal arguments used by the runner mode to enable its features.

### config

Useful in runner mode to develop config-dependent experiments. The provided container (i.e. config.py file) will be accessible as get.config

### task_path

This folder (i.e. tasks/my_task) will be copied to the log folder as a backup of the experiment run. This path will also be searched for a file launch.py to import and run before the experiment start.

### import_pre_run

Path to a .py file to import just before starting the run, i.e. to start a GUI
