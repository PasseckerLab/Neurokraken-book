# States - Structuring task steps

States are how you can structure which happens when in a task.

States are typically written after the configuration of your setup.
The following example shows **5 optional methods** that can be added to a state to run at a specific context.

```python
from neurokraken import Neurokraken, State
from neurokraken.configurators import devices, Display
from neurokraken.controls import get

serial_out = {'led': devices.direct_on(pin=3, start_value=False)}

nk = Neurokraken(serial_in={}, serial_out=serial_out, log_dir='./', display=Display())

get.white_background = False # we will make use of this created variable later

# States and task code starts here

class My_state(State):
    def pre_task(self):
        pass

    def on_start(self):
        pass

    def loop_main(self):
        return False, 0
    
    def loop_visual(self, sketch):
        pass

    def on_end(self):
        pass

task = {'state_0': My_state(next_state='state_0')}

nk.load_task(task)

nk.run()
```

The line `class My_state(State):` does 2 things:
- We decide on the class name "`My_state`". This allows use to later use it as part of the task with `My_state(next_state=<str>)`
- `(State)` ensures that neurokraken can the class as a state, by automatically adding internal properties.

Afterwards we can add one or more of the shown methods to add context-specific code to our task. The (self) argument enables access to a shared `self.` variable space across the class.

At the end we create a task dictionary where we map names (like here `'state_0'`) to the states we want to use (`My_state()`)
Even in single state tasks, States require an argument `next_state=<str>` that names the successor state or list of possible successor states, with the progression typically decided within loop_main() or by a maximum time provided to the state `My_state(..., max_time_s=3)`.

Each method has access to `self`, allowing you to share variables and their usage across methods of the same state, and access to `get.` allowing you to access and interact with global variables like the `get.white_background = False` defined in the example above. Additionally, if you are familiar with python's `global` keyword or specified file imports and namespacing, these approaches can provide further ways of accessing individual variables across different functions.

## pre_task(self)

Code in here is executed once before the task begins.

This is useful for setting up variables used later later in the state as self.<variable_name>, as well as loading files like state-related images to be used in the task.
A good pattern is to move compute-intensive code that only needs to be run once (like loading files) outside of repeatedly run methods like loop_main() or on_start().

```python
def pre_task(self):
    self.number = 5
```

pre_task can be run with a second argument `sketch`, which will provide your code access to py5 sketch functions. This is useful for loading images, or 3D models to later be used in the loop_visual().

```python
def pre_task(self, sketch):
    self.image = sketch.load_image('C:/Path/To/My/Image.png')
```

## on_start(self)

This method is run every time the state is entered - typically following the previous state's end.

This is a good spot to set or reset a variable for the subsequent loop_main() or loop_visual() and to change a controlled device like a servo motor or valve to a different value for this state or a followup state.

```python
class My_state(State):
    def on_start(self):
        get.send_out('led', True)
        self.choice = 'none' # if a choice is made we will update it to 'left' or 'right' in loop_main()
```

## loop_main(self)

This method is where your main state logic takes place - reading sensors, deciding outcomes, logging custom data, and then either continueing to the next iteration of loop_main() or to to on_end() and the next state.

loop_main runs as fast as your system allows, expectably thousands of times per second, allowing for tight control of task progression.

Notably loop_main() has to return a bool and a number, i.e. `return False, 0`

The bool True/False signals whether the state is completed and you want to move on to the next state i.e. when you get.read_in() a touch choice from the subject, or find its position entered a target radius.

The number decides which follow-up state option to continue to when a list of options is provided. Thus if there is only a single follow-up option we are always returning index 0.

```python
class Choice(State):
    def loop_main(self):
        if get.read_in(...):
            # bad choice made, continue to the timeout penalty state 0
            return True, 0
        elif get.read_in(...):
            # good choice made, continue to the reward state 1
            return True, 1
        else:
            # no choice made, rerun loop_main()
            return False, 0

class Reward(State):
    ...

class Timeout(State):
    ...

task = {'choosing': Choice(next_state=['timeout', 'reward']),
        'timeout': Timeout(next_state='choosing'),
        'reward':   Reward(next_state='choosing')}
```

loop_main() can be filled with your own python code to make complex decisions and changes based on a subject's current `get.read_in` actions, the history `get.log`, get.time_ms, or state or global variables.

A common pattern is for the main loop to change state `self.` or global `get.` variables like the starting example's `get.white_background=False` that are then utilized by other code like loop_visual() with their updating values. 

Returning `False` means the state was not yet completed and loop_main() will loop run again. If you return `True` the task will move on to the next state, decided by the returned number.

Notably a state can also be provided with a maximum duration, and if this many seconds have passed the state will progress regardles of whether loop_main() returned True or False.  

```python
'choosing': Choice(next_state=['timeout', 'reward'], max_time_s=10)
```

You can manually read the state's `self.start_time` variable for the task millisecond when the state started. The duration the state has already been running thus is `get.time_ms - self.start_time`  

## loop_visual(self, sketch)

When using a [display](display) in your task, this function allows you to draw custom visual stimuli using py5's drawing function through the sketch argument.

```python
def loop_visual(self, sketch):
    if get.white_background:
        sketch.background(255, 255, 255) # R, G, B
    else:
        sketch.background(0, 0, 0)
```

You can find the py5 reference [here](https://py5coding.org/reference/summary.html). Py5 makes it easy to draw interactive custom 2D or 3D stimuli, animations, and environments using simple shapes, text, or loaded images and 3d models.

## on_end(self)

This function runs after conclusion of a state, due to `loop_main()` returning True to signal the state being finished, or due to a state's duration exceeding the provided seconds `max_time_s=`.

```python
def on_end(self):
    print('the state has concluded')
```

## Further methods

As the code of the state class is under your control you can add further methods to call from scheduled methods like loop_main() or on_start() to organize your code.

```python
class My_state(State):
    def sum(self, a, b):
        return a + b

    def on_start(self):
         number = self.sum(5, 3)
         print(number) # 8
```

If you are familiar with python classes and the `__init__()` method, you can also add one to your class to create different variations of the same State with specific differences. I.e. a task might contain 2 versions of a state, one with a rewarding `self.probability=0.9` and one with a less positive `self.probability=0.1`. An example of this can be found in the [frequently used patterns](patterns).

## Organizing state flow - The task dictionary

The task dictionary contains named states of our task, with each state also naming its successor or list of possible succesors.
We can use this to create an example task that blinks an LED every 5 seconds the following way.

```python
class Light_on(State):
    def on_start(self):
        get.send_out('led', True)
    
class Light_off(State):
    def on_start(self):
        get.send_out('led', False)

task = {'on' :  Light_on(next_state='off', max_time_s=5),
        'off': Light_off(next_state='on',  max_time_s=5)}

nk.load_task(task)
```

Often a subject will make a choice and based on the choice we want to progress to splitting outcome states. For this we can provide a list of next_state options.

For example we can reorganize our slow blinking LED example into a trial starting with a 2 second "Start" state that decides whether the next state will be Light_on or Light_off, with either of these states returning to another decidiging Start state after their 5 seconds.

```python
import random

class Light_on(State):
    def on_start(self):
        get.send_out('led', True)
    
class Light_off(State):
    def on_start(self):
        get.send_out('led', False)

class Start(State):
    def on_start(self):
        self.light_on = True if random.random() > 0.5 else False
    
    def loop_main(self):
        next_state_idx = 1 if self.light_on else 0
        return False, next_state_idx

task = {'start': Start(next_state=['off', 'on'], max_time_s=2),
        'off': Light_off(next_state='start', max_time_s=5),
        'on' :  Light_on(next_state='start', max_time_s=5)}
        
nk.load_task(task)
```

The state flow can be as extensive, branching and complex as you want it to be, i.e. a task may progress through a sequence of 7 states before returning to the first one with the outcome of a middle state determining a normal progression, a reset to the first state or skipping ahead of the next states.

You can also add a single forever state or a group of states which are unreachable from the next_state= options of the core task. These state's can still manually be switched to or from with `get.progress_state('<state_name>')` from a UI button to create a manual standby state or special use states within the same task, to enable use cases similar to [blocks](blocks).  

## State arguments

States always have to receive their next_state argument. Further they can receive 4 optional arguments to extend their functionality.

### next_state

*str or list*

The next_state or list of next state options to progress to upon the main loop returning True or a state's provided max_time_s running out.

```python
task = {'start': Start(next_state=['off', 'on'], max_time_s=2),
        'off': Light_off(next_state='start', max_time_s=5),
        'on' :  Light_on(next_state='start', max_time_s=5)}
```

### max_time_s

*float or tuple\[float,float\] or Callable->float*

The maximum duration in seconds for which a state will run before progressing regardless of wether the main loop returns the trial to be finished True. Defaults to 1_000_000.

You can provide either a number (float/int), a 2 entry interval (tuple/list) for a random time within the interval every time the state runs, or a function returning a float for a customized dynamic timing.

```python
Light_on(next_state='start', max_time_s=5)

Light_on(next_state='start', max_time_s=[2.5, 7.5])

def five_seconds():
    return 5
Light_on(next_state='start', max_time_s=five_seconds)
```

### run_at_start and run_at_end

*Callable or list*

These 2 arguments allow you to provide a function or list of functions to run after on_start() or after on_end() respectively.

This can be useful to organize complex task code, i.e. variables and logic might be defined in on_start() and on_end() but all logging and complex analysis is defined in run_at_start= and run_at_end= allowing you to separate the task itself from calculations for live plots or post-experiment analysis.

```python
class Start(State):
    def on_start(self):
        get.light_on = True if random.random() > 0.5 else False
        self.light_on = get.light_on # extra copy to the state self for later in this example

# add an empty list to log light decisions of this example task
get.log['light_decisions'] = []

def log_light():
    # append an entry with the time and decision to this list
    get.log['light_decisions'].append([get.time_ms, get.light_on])

task = {'pause': Start(next_state=['off', 'on'], max_time_s=3, run_at_start=log_light),
        ... }
```

A function provided to run_at_start= or run_at_end= can optionally contain an argument to access the respective state/variables added to its self.

```python
def log_light(state):
    # add the time and decision to this list
    get.log['light_decisions'].append([get.time_ms, state.light_on])
```

run_at_end functions can also be provided with a 2nd argument, a boolean to read True/False whether a state was ended by the main loop or timed out.

(trials)=
## Trials

States can provide basic trial management.

Frequently tasks are organized in a repeating pattern of states and it makes sense to organize and log data accordingly. `get.log` contains a list of dictionaries under `get.log['trials']` that can be used for this purpose.

You can provide one or more states at the end of your task pattern with the argument `trial_complete=True` and every time one of these states culminates another entry will be added to the log trials list. 

You can access the current trial number as `len(get.log['trials'])`, the current (thus most recent and last) trial with `get.log['trials'][-1]`, and the most recent 5 trials with `get.log['trials'][-5:]`.

Each trial by default contains a `['start']` time entry, i.e. `get.log['trials'][-1]['start']` for the start of the most recent trial. As a flexible python dictionary the trial entry can manually be extended with every information relevant to the task or successive analysis.

Finally, `neurokraken.load_task()` can be provided with a function as `run_post_trial=`, that will run at the end of a trial. This time point is well suited for updating trial specific values and lists relevant for the future of the task or live plots and visualizations. For example one might maintain a list of time-value-pairs of a subject's reaction times, or actions during the last 30 trials to then regularly plot this list in a live-UI.

```python
class Light_on(State):
    ...

class Light_off(State):
    ...

class Start(State):
    def on_start(self):
        print(f'entering trial number {len(get.log['trials'])}')
        get.light_on = True if random.random() > 0.5 else False
        # save the information to the current trial dictionary
        get.log['trials'][-1]['light'] = get.light_on

task = {'start': Start(next_state=['off', 'on'], max_time_s=2),
        'on' :  Light_on(next_state='start', max_time_s=5, trial_complete=True),
        'off': Light_off(next_state='start', max_time_s=5, trial_complete=True)}

get.average_light = 0.5

def recalculate_average_light():
    was_light_on = []
    for trial in get.log['trials']:
        was_light_on.append(trial['light'])
    true_count = sum(was_light_on) # True counts as 1, False as 0
    get.average_light = true_count / len(get.log['trials'])

nk.load_task(task, run_post_trial=recalculate_average_light)
```

(blocks)=
## Alternative blocks of states

typically a task is loaded as a dictionary of states. However you can also provide a nested dictionary allowing you to organize groups of states and their progressions into individual state blocks.

As a `next_state=` can only be within the same block the only way to switch between blocks is manually through `get.switch_block('<block_name>')`.

```python
from neurokraken import State
class State_A(State):
    ...
class State_B(State):
    ...
class State_C(State):
    ...

task = {
'consistent_block' :  {'single_state': State_A(next_state='single_state')}
'alternating_block':  {'B': State_B(next_state='C'),
                       'C': State_C(next_state='B')}
}
nk.load_task(task)
...

# within a UI-triggered element switch the active block
get.switch_block('alternating_block')
```

The first block will be the one executed when you run your task, unless a different one was provided to `neurokraken.load_task(..., start_block='<block_name>'`.

An interesting use case of blocks is to use python's deepcopy to create an independent copy of a block and to manually change variables within selected states of the copy in order to create distinct and switchable versions of a given task.

## Autologging

Progressions in the current state, trial, and block are automatically saved in get.log['states'], get.log['trials'] and get.log['blocks'].

State and block changes are maintained as a list of (time, state/block-name) tuples.

Trials are maintained as a list of extendable dictionaries with a preexisting `start` entry.

## load_task arguments

Neurokraken.load_task() can receive optional arguments besides the task dictionary for extended control of task elements, as shown in the usage of [trials](trials) above.

### start_block :str

The name of the block to start the run with. If not provided the task will begin with the first block

### run_at_start :Callable

The provided function will be called at the very beginning of the task before the states.

The run_at_start function will be followed by an additional communication loop before the task begins, allowing you to start the task with controlled devices in a desired state.

```python
def move_back_servo():
    get.send_out('my_servo', 160)

nk.load_task(..., run_at_start=move_back_servo)
```

### run_at_quit :Callable

THe provided function will be run once the task is ended, i.e. through `get.quit()` or `Ctrl+Alt+Q`.

After the function an additional communication loop will be performed, allowing to set the final position of teensy-side controlled devices at shutdown regardless of the state during which the task was quit.

```python
def close_valve():
    get.send_out('my_valve', False)

nk.load_task(..., run_at_quit=close_valve)
```

### run_post_trial :Callable

This function will be called upon trial completion as shown in the [trials](trials) subchapter.

### specialized arguments

#### main_as_sketch :bool

Whether to run the main loop as a py5 sketch or a python while-True: - loop. Defaults to True for a py5 sketch.

It was found that using a py5 sketch allows for more stable main loop timing during high workload tasks (cameras, visual, live-AI). The ability to run as a while-True: loop is still available with `main_as_sketch=False`.

#### permanent_states :list

A list of classes containing a .run() function. The respective .run() function will be executed at the end of each main loop iteration.
 
#### run_at_visual_start :Callable

When using py5's shader functions it was found that load_shader() needs to be run by the same sketch that will display it. A method provided to run_at_visual_start allows its code access to the task's loop_visual() sketch during setup for this niche situation.

```python
def method_on_visual_setup(sketch):
    pass

nk.load_task(..., run_at_visual_start=method_on_visual_setup)
```

#### experiment :Container

Used by the runner mode to provide get.experiment access to the chosen experiment .py file and variables therein.