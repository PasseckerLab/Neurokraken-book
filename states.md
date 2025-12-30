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
- we decide on the name "`My_state`". This allows use to later use it in the task with `My_state(next_state=\<str\>)`
- `(State)` ensures that neurokraken can use it as a state, by automatically adding internal properties to the class.

Afterwards we can add one or more of the shown methods to add context specific code to our task.

At the end we create a task dictionary where we map names (like here `'state_0'`) to the states we want to use (`My_state()`)
Even in single state tasks, States require an argument `next_state=\<str\>` that names the successor state or list of possible successor states, with the progression typically decided within loop_main() or by a maximum time provided to the state `My_state(..., max_time_s=3)`.

Each method has access to `self` allowing you to share variables and their usage across methods of the same state, and access to `get.` allowing you to access and interact with global variables like the `get.white_background = False` defined in the example above.

## pre_task(self)

Code in here is executed once before the task even begins.

This is useful for setting up variables used later later in the state, as well as or loading files like images to be used in the task.
A good pattern is to move compute-intensive code that only needs to be run once (like loading files) outside of repeatedly run methods like loop_main() or on_start().

```python
def pre_task(self):
    self.number = 5
```

pre_task can be run with a second argument `sketch` providing will provide your code access to py5 sketch functions. This is useful for loading images, or 3D models to later be used in the loop_visual().

```python
def pre_task(self, sketch):
    self.image = sketch.load_image('C:/Path/To/My/Image.png')
```

## on_start(self)

This method is run every time the state is entered - typically following the previous state's end.

<!-- 

set/reset a variable for the subsequent loop_main() and loop_visual()
serial_out led from empty example

```python
class My_state(State):
    def on_start(self):
```

## loop_main(self)

## loop_visual(self, sketch)

## on_end(self)

## further methods

As the code of this class is under your control you can add further methods to call from scheduled methods like loop_main() or on_start() to organize your code.

```python
class My_state(State):
    def sum(self, a, b):
        return a + b

    def on_start(self):
         number = self.sum(5, 3)
         print(number)
```

If you are familiar with python classes and the `__init__()` method, you can also add one to your class to create different variations of the same State with specific differences. I.e. a task might contain 2 versions of a state, one with a rewarding `self.probability=0.9` and one with a less positive `self.probability=0.1`. An example of this can be found in the [frequently used patterns](patterns).

## Organizing state flow - The task dictionary



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

## State arguments



<!-- 
## run_at_visual_start

When using py5's shader functions it was found that load_shader() needs to be run by the same sketch that will display it. A method provided to run_at_visual_start allows its code access to the task's loop_visual() sketch during setup for this niche situation.

```python
def method_on_visual_setup(sketch):
    pass

nk.load_task(..., run_at_visual_start=method_on_visual_setup)
```
 -->
