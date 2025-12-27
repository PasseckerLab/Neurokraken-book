# Controls and Task development

## task development

A task consists of 3 main steps. You can see practical examples within the [Quickstart](quickstart) and [Examples](examples).

1. The configuration of your experiment setup elements and creating of a `Neurokraken()` object using them.
2. Creating a task in the form of at least one `neurokraken.State` and loading the task into the Neurokraken.
3. Run the task.

Optionally additional code to run in parallel like a UI can be launched as step 2.5

### Configuration

```python
from neurokraken import Neurokraken
from neurokraken.configurators import devices, Display, Camera, Microphone
```

This part is where you decide on the physical components to utilize in your setup using Neurokraken.configurators. This includes electronic components to use (sensors, stimuli) as well as displays, cameras, and microphones. Further you can decide where to save the log and wether to run with electronics connected or in simulated keyboard or agent mode.

You can find a walkthrough of configuration options in the [configurations chapter](configuration) with [cameras](cameras.md), [microphones](microphones) and [modes](modes) having their own pages.

Arduino-Side devices are covered in the [device library](device_library)

### The task

```python
from neurokraken import State
from neurokraken.controls import get
```

After creating the Neurokraken object with the configurators of interest, the next step is to create a task consisting of at least one neurokraken.State. 

The state only serves as a flexible container for your task's python code. However, more advanced tasks can benefit from the organization, compartimentalization and flexibility multiple states or even blocks of states that one can switch between can provide.
States can contain `on_start()`, `on_end()` and `loop_main()` functions allowing you to distribute your task's python code accordingly.

Within your task's python code, the main interface to neurokraken's live and logged data as well as controls for its managed devices is `controls.get`. While `controls.get` is an essential interface for any task, the features of states can be explored as they become useful to you and your developing task.

You can find the documentation for states in the [states](states) chapter while `get` is documented below in [controls.get](controls-get).

(controls-get)=
## controls.get

Neurokraken's controls.get servers as your main interface to neurokraken's managed functionalities enabling to
- read live sensor values defined in serial_in
- change the state of controlled devices defiend in serial_out
- read and add to the automated log dictionary of previous sensor/control value changes and task events.
- access current camera frames for display or live-analysis
- override the current state or switch to an alternative block of states
- read the current experiment time
- start, stop, and quit the experiment
- ...

Due to its central position in neurokraken data and controls, get and its managed elements are the main way to direct events and conditions of your task complemented by states for optional compartimentalization of more advanced experiment progressions.

`get` can be accessed across components, i.e. you can turn on an LED as part of a tasks states' loop_main() or on_start() as well as from a UI element.

```python
# in your experiment code following creation of the Neurokraken object
from neurokraken.controls import get

class My_state(State):
    def on_start(self):
        get.send_out('my_led': True)

...

# in your UI code
def light_on():
    get.send_out('my_led': True)

gui.button(label='shine', pos=(50, 50), on_click=light_on)
```
This unified importable live access to neurokraken enables adding arbitrary novel elements to your task like observer protocols scheduling task progression based on logged performance.

As get is a universally used component in neurokraken tasks even for tasks split over multiple files, it can serve as a useful dumping ground for global variables you want to access across elements. For example, we might define a probability for a reward state actually providing a reward at the start of our script. If we store this probability in `get` we can access it within our state to run the probability as well as from a UI element to live update the probability.

```python
# in your experiment code following creation of the Neurokraken object
from neurokraken.controls import get
import random

get.probability = 0.5

class Reward(State):
    def on_start(self):
        if random.random() < get.probability:
            get.send_out('reward_valve': 70)

...

# in your UI code
def update_probability(value):
    get.probabilities = float(value)

gui.Text_Input(pos=(50, 50), on_enter=update_probability)
```

Practically in this example we might further want to wrap the float conversion with a try statement to guard against crashes from accidental non-numeric inputs.

```{Note}
While a controls.get object exists before the creation of the Neurokraken() object, as its content depends on the configuration provided to Neurokraken it is not yet populated and should only be imported after the creation of Neurokraken().
```

(log)=
## The log

A key element for live tasks and their successive analysis is the log dictionary. During the task it can be accessed with `get.log`. After the task you will find the same data as a log.json file in the log_dir you set for the Neurokraken object. A json file can easily imported into a python dictionary allowing you to work with exactly the same data structure within the experiment and the following analysis.

### log organization

While some log entries are automatically added and maintained by neurokraken depending on the components of your task (see below) the log is a standard python dictionary allowing you to manually add entries or extend existing ones depending on your task requirements.

Automatically maintained and extended data includes the following, generally time-stamped, elements.
- the data, time and subject data provided to Neurokraken() (dict or simple ID string)
- progressions of the current state, block and trial
- millisecond precision changes in sensor values 
- enacted changes to controlled devices
- time of camera frames
- time of microphone keypoints

```
get.log
├── experiment_data
│   ├── datetime: str(datetime.now())
│   └── subject data
├── states {time/name}: []
├── blocks {time/name}: []
├── trials {time}: []
├── events: []
├── read_in_device_0_name (t ms/value): []
├── read_in_device_1_name (t ms/value): []
├── ...
├── controls
|   ├── send_out_device_0_name (t ms/value): []
|   ├── send_out_device_1_name (t ms/value): []
|   ... 
├── camera_0_name: [] (frame time ms/#idx/vid time)
├── camera_1_name: [] (frame time ms/#idx/vid time)
│   ...
├── microphone_0_name: [] (frame time ms/audio time)
|   ...
├── Your added log list A: []
├── Your added log list B: []
├── Your added log dict C: {}
└── ...
```

### extending the log

Outside of these elements, you can add any data to the log that helps your task or even extend automatically created entries.
For example, your task might include a state where a choice left or right is made each trial. Within this state you could add the `'choice'` as an entry to the current trial (which is the last so far and thus [-1] in the list).

```python
get.log['trials'][-1]['choice'] = 'left'
```

And then in live or post-experiment analysis you can iterate through the trials to count up specific events like left choices
```python
left_trials = [t for t in get.log['trials'] if t['choice'] == 'left']
num_left_trials = len(left_trials)
```
Alternatively you could add a new entry to the log solely managed by you.
```python
# before you begin the task with neurokraken.start()
get.log['choices'] = []
# Within the task loop, after making a choice
if ...:
    get.log['choices'].append([get.time_ms], 'left')
```

## Controls (.get)

<!-- ```{contents}
:local:
``` -->

### time_ms:int
The current experiment time in milliseconds

### log:dict
The log dictionary of your experiment so far. See [the log](log) for a list of automatic content

### read_in(\<device name\>:str)
access the current value of a peripheral electronic sensor defined in serial_in.

```python
# for the Neurokraken() initialization
from neurokraken.configurators import devices
serial_in = {'beam_break': devices.analog_read(pin=5)}

# in your task specific code
if get.read_in('beam_break') < 400:
    ...
```

If needed you can also access the serial_in dictionary itself as `get.serial_in`

### send_out(\<device name\>:str, value)
Control a peripheral electronic device defined in serial_out to a different value. Open a Valve, turn on an LED, move a Motor Arm,...

```python
# for the Neurokraken() initialization
from neurokraken.configurators import devices
serial_out = {'reward_valve': devices.timed_on(pin=5)}

# in your task specific code
get.send_out('reward_valve', 70)
```

If needed you can also access the serial_in dictionary itself as `get.serial_out`

### camera(i:int, preview=False)
Access the current frame from the i-th camera, i.e. `frame = get.camera(0)` for the first camera of your setup.

`preview=True` provides a py5image ready to display in py5-based UIs. This is only possible when your camera uses `ui_view_enabled=True`. (`preview=True` will apply the [Camera's](Cameras) set `ui_view_scale` and `ui_view_step`.)

`preview=False` Will return the full scale numpy image.

`preview=True` is reccommended for live usage in the UI due to its lower performance impact. `preview=False` is reccommended for computer vision applications that can make full use of the entire numpy pixel aray. Live displayed camera frames are the dominant impact on compute that is easy to reduce in typical tasks when higher performance is needed.

`get.cameras` can be used to access a list of the neurokraken-internal Camera classes if needed.

### log_dir :Path
The log Path - can be used to save additional files with the rest of the log.

### threads_info :dict
A dictionary containing `'framerate_main'`, `'framerate_visual'` and `'framerate_cams'` useful for development and maximizing a camera's viable fps. framerate_cams is a dict[str:float] with individual cameras accessible by their configured name

```python
# example live framerate within a py5 UI's draw()
self.text(f'main loop fps: {get.threads_info["framerate_main"]}', 10, 25)
```

### mode :str
The mode in which the task is run, one of 'teensy', 'keyboard' or 'agent'. You can use get.mode to change task behavior depending on how the task is run, i.e. to skip parts of your task code if you are in keyboard mode.

### start()
Begin your task. Only needs to be called if your config is set to autostart=False.
A manually timed start can be useful in cases where extended preparation and the highest accuracy of the first few seconds is required.
Components outside of the actual state loops and automatic logging are already executing before get.start() allowing you to monitor camera feeds and sensor values before starting the actual task. 

### stop()
manual non-program-quitting task end

### quit()
stop the state machine, save the log and exit the program. 
A practical use case is to bind `get.quit()` to the shutdown function of your UI or when a specific time point is surpassed.

### quitting :bool
Check whether a shutdown has been triggered - useful for shutting down self-developed parallel code

### active :bool
get.active can be read by elements like cameras or your code to fit their activity with the active experiment time. If you configure Neurokraken with autostart=False, active will be False until get.start() and be inactive again after get.stop()

### permanent_states :list
The list of your permanent states provided for your task

### blocks :dict
Your task states.

You can dynamically access states and modify their variables through get.block, i.e. `get.blocks['my_block']['my_state'].variable = 5`

In the common case that you provided a non-nested dictionary of task states to neurokraken.load_task() this dict will contain a single default block (fittingly named get.blocks['block']) containing your dictionary of states.

### switch_block(\<next block name\>:str)
switch to a different block/state flow of your task by its name.
Useful for manual override of the general task or its varations.

Synonymous with get.current_block = 'x'

```python
from neurokraken import State
class State_A(State):
    ...
class State_B(State):
    ...
class State_C(State):
    ...
task = {
'consistent_block':  {'single_state': State_A(next_state='single_state')},
'alternating_block':  {'B': State_B(next_state='C')}
                      {'C': State_C(next_state='B')}
}
nk.load_task(task)
...

# Switch within a UI-triggered element of code
get.switch_block('alternating_block')
```
### progress_state((\<next state name\>:str))
Override the task flow to progress to a different state of this block now.
When overriding progressing from a state, its on_end() and run_at_end() won't be called.

Synonymous with get.current_state = 'x'

### current_block :str
Property to get and set the current block in your task.

This property provides convenient access to the current block of the state machine,
allowing both reading and modification of the current block depending on task conditions or manual intervention.
Setting a block can alternatively be performed with get.switch_block(block_name)

```python
def loop_main(self):
    ...
    if num_rewards > 200 and get.current_block == 'easy_difficulty_block':
        get.current_block = 'medium_difficulty_block'
```

### current_state :str
Property to check and override the current state outside of the existing state_flow.
Progressing to a new state can alternatively performed with get.progress_state(state_name)

```python
def key_pressed(e):
    if e.get_key() == 'm':
        print(f'old state was {get.current_state}. Override moving to "reward_state"')
        get.current_state = 'reward_state'
```

Please note that during a state's `def on_start()` or a state's provided `run_at_start=` get.current_state still points
to the previous state rather than the one currently starting up. This is so that parallel code like loop_visual 
does not try to already run a state that may be reliant on a variable set up during on_start() and that the
`current_state` accessed by elements of your task will always have its `def on_start()` and `run_at_start=` executed
before becoming available as the `get.current.state`. If you require access to the state during these 2 startup functions,
utilize the respective self/first argument; def on_start(self): and `def my_at_start_function(state):

### current_block_trials_count :int
The numebr of trials completed within the current block. 

To access the number of total trials so far you can access your log: `len(get.log['trials])`

### config
Runner mode only

Access and/or update your config.py variables from other files like task.py or launch.py. Useful if your experiment requires information you provide for a specific setup in config.py like setup-specific thresholds for a given setup.

Alternatively values to be shared across files can be maintained in .get itself or within the log

### experiment
Runner mode only
Access and/or update your experiment.py variables from other files

Alternatively values to be shared across files can be maintained in .get itself or within the log

## tools

```python
from neurokraken import tools
```

tools are additional helper utilities

### list_connected_recording_devices()

prints out which microphones and cameras are detected at which idx and what respective camera resolutions are supported.

Does not yet check for GenICam type cameras

### import_file(\<file_path\>:str, add_to_syspath=False)

Import a python script from a file path.
    
This can be useful to integrate 3rd party scripts/tools into neurokraken tasks, that have not yet been made into proper packages.

add_to_syspath is an optional argument to automatically add the file's directory to sys.path which enables imports of the imported script itself to work.

```python
# We want to import cutils.py from a different location on our pc to run live image analysis
from neurokraken import tools
cutils_path = str(Path('C:/path/to/cutils.py'))
cutils = tools.import_file(cutils_path)

cutils.create_cutie(cutie_config_path)
```