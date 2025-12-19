# Common task patterns

These are elements of tasks that may frequently be needed and hopefully can be of help gathered as patterns in this document.

This document is heavily in progress and will be completed soon.

## Logging an event from the UI

## Ending the task when the UI is closed

## Ask for a specific input to be logged before the experiment start, i.e. the subject

## Look up and add subject-specific data to the subject entry, i.e. from a network file or server

## Create variations of the same State through custom \_\_init\_\_() arguments

```python
class Delay(State):
    def __init__(self, background=(0,0,0), **kwargs):
        super().__init__(**kwargs)
        self.background = background
        
    def loop_visual(self, sketch):
        sketch.background(*self.background)

    def loop_main(self):
        return False, 0

blocks = {
'my_block': {
    'red':  Delay(max_t=10, next_state='cyan', background=(255,0,0)),
    'cyan': Delay(max_t=10, next_state='red', background=(0,255,255)),
    }
}
```

## Override an experiment variable from a UI text input

## Store variables in .get for shared access across files

## Using Threading.thread() for repeated timed actions

## Extending the task visual across multiple displays

## Adding more visual sketches or windows.

## Create a variation of a block with changed variables

```python
# create a State that has a specific property, i.e. probabilities
# (In a full example we might add a def loop_main() that checks 2 buttons 
#  and provides a reward with the respective probability)
class Choice(State):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.probabilities = [0.5, 0.5]

# We can start out creating an experiment block that uses this state
task = {'equal_good': 'choice': Choice(next_state='choice')}

# Now we create an independent copy of that entire block with all its states and add it as a 2nd block
import copy
task['better_left'] = copy.deepcopy(task['equal_good'])
# Then we update the copies probabilities
task['better_left']['choice'].probabilities = [0.8, 0.2]
```

We can now `get.switch_block('better_left')` to change to this alternative experiment block

## start with a specific state depending on conditions

```python
task = {'better right': ...,
        'better left': ...}

start_block = 'better right' if random.random() < 0.5 else 'better left'
nk.load_experiment(task, ..., start_block=start_block)
```

## Set a peripheral like a valve to a specific final state upon the task end using run_at_quit

## Switching parameters or blocks based on performance

## Run the same task with alternative switchable device options

## Pause task execution by switching to a standby State