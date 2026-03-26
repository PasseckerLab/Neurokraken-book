# UI Examples

## Minimal examples

The following chapters show 4 UI frameworks (krakengui, dearpygui, tkinter, pyside6) used with a minimal Neurokraken task. The respective UI contains containing a text input to set a tone device's frequency and a text field showing the current task milliseconds.

UI window can be inserted into a task just before `nk.run()`. A common pattern in UI frameworks is that the code element of a button, slider, text input, ... is provided with a callback function whose code will be executed when the user triggers the element.

### Krakengui

Kraken-gui is a set of simple gui elements and plotting functionalities for py5. Utilizing a py5 sketch enables custom visualizations beyond existing gui elements while also providing great flexibility in how to place elements. Py5's high compute efficiency also makes it well suited to run alongside/as part of neurokraken tasks.

You can find krakengui at https://github.com/alexanderwallerus/kraken-gui.

Minimal UI:

```python
from neurokraken import Neurokraken, State
from neurokraken.configurators import devices
from neurokraken.controls import get

serial_out = {'frequency': devices.tone(pin=32)}

nk = Neurokraken(serial_in={}, serial_out={}, mode='keyboard', log_dir=None)

class Nothing(State):
    pass

nk.load_task({'my_state': Nothing(next_state='my_state')})

# ----------------------- UI -----------------------

from py5 import Sketch
import krakengui as gui

def change_freq(freq:str):
    get.send_out('freq', int(freq))

class UI(Sketch):
    def settings(self):
        self.size(400, 400)

    def setup(self):
        gui.Text_Input(pos=(20, 30), label='frequency', on_enter=change_freq)

    def draw(self):
        self.background(0)
        self.fill(255)
        self.text(f'time (ms): {get.time_ms}', 20, 20)
        if get.quitting:
            self.exit_sketch()

    def key_pressed(self, e):
        pass

ui = UI()
ui.run_sketch(block=False)

nk.run()
```

### dearpygui

[dearpygui](https://github.com/hoffstadt/dearpygui) is an easy python interface to the extensive component range of the Dear-ImGui C++ library.

`pip install dearpygui`

```python
from neurokraken import Neurokraken, State
from neurokraken.configurators import devices
from neurokraken.controls import get

serial_out = {'frequency': devices.tone(pin=32)}

nk = Neurokraken(serial_in={}, serial_out=serial_out, mode='keyboard', log_dir=None)

class Nothing(State):
    pass

nk.load_task({'my_state': Nothing(next_state='my_state')})

# ----------------------- UI -----------------------

import dearpygui.dearpygui as dpg

def update_text():
    dpg.set_value('displayed_text', f'Current time: {int(get.time_ms)}')

def input_callback(sender, text_input):
    if text_input.isdigit():
        get.send_out('frequency', int(text_input))
        print(f'updated frequency to {int(text_input)}hz')

def run_pygui():
    dpg.create_context()
    
    with dpg.window(label="Neurokraken", width=300, height=220):
         dpg.add_text(f'Current time: {int(get.time_ms)}', tag="displayed_text")
         dpg.add_input_text(label="new frequency", width=120,
                            callback=input_callback, on_enter=True)
    
    dpg.create_viewport(title='Neurokraken', width=300, height=150)
    dpg.setup_dearpygui()
    dpg.show_viewport()
    
    while dpg.is_dearpygui_running():
        update_text()
        dpg.render_dearpygui_frame()
    
    dpg.destroy_context()

# run pyside6 as a thread for non-blocking execution of the successive nk.run() code
import threading
threading.Thread(target=run_pygui, daemon=True).start()

nk.run()
```

### Pyside6

Pyside6 is a GUI framework based on the popular QT toolkit.

`pip install pyside6`

```python
from neurokraken import Neurokraken, State
from neurokraken.configurators import devices
from neurokraken.controls import get

serial_out = {'frequency': devices.tone(pin=32)}

nk = Neurokraken(serial_in={}, serial_out=serial_out, mode='keyboard', log_dir=None)

class Nothing(State):
    pass

nk.load_task({'my_state': Nothing(next_state='my_state')})

# ----------------------- UI -----------------------

from PySide6.QtWidgets import QApplication, QWidget, QVBoxLayout, QLabel, QLineEdit
from PySide6.QtCore import QTimer

class SimpleGUI(QWidget):
    def __init__(self):
        super().__init__()
        self.setGeometry(100, 100, 400, 150)
        layout = QVBoxLayout()
        
        self.display_label = QLabel(f'Current time: {int(get.time_ms)}')
        layout.addWidget(self.display_label)
        
        self.text_input = QLineEdit()
        self.text_input.setPlaceholderText("Set frequency:")
        self.text_input.editingFinished.connect(self.on_text_entered)
        layout.addWidget(self.text_input)
        
        self.setLayout(layout)
        
        self.timer = QTimer()
        self.timer.timeout.connect(self.update_label)
        self.timer.start(33) # update the label every 33ms (30fps)
    
    def update_label(self):
        self.display_label.setText(f'Current time: {int(get.time_ms)}')
    
    def on_text_entered(self):
        if self.text_input.text().isdigit():
            get.send_out('frequency', int(self.text_input.text()))
            print(f'updated frequency to {int(self.text_input.text())}hz')
            
def run_gui():
    app = QApplication()
    window = SimpleGUI()
    window.show()
    app.exec()

# run pyside6 as a thread for non-blocking execution of the successive nk.run() code
import threading
threading.Thread(target=run_gui, daemon=True).start()

nk.run()
```

### Tkinter

Tkinter is a UI framework pre-bundled with python which does not need to be separately installed.

```python
from neurokraken import Neurokraken, State
from neurokraken.configurators import devices
from neurokraken.controls import get

serial_out = {'frequency': devices.tone(pin=32)}

nk = Neurokraken(serial_in={}, serial_out=serial_out, mode='keyboard', log_dir=None)

class Nothing(State):
    pass

nk.load_task({'my_state': Nothing(next_state='my_state')})

# ----------------------- UI -----------------------

import tkinter as tk

def run_gui():
    text_field = None
    label = None
    def change_frequency(event):
        if text_field.get().isdigit():
            get.send_out('frequency', int(text_field.get()))
            print(f'updated frequency to {int(text_field.get())}hz')
            
    def update_label():
        label.config(text=f"Current time: {int(get.time_ms)}")
        # Schedule next update in 33ms
        root.after(33, update_label)

    # window parameters
    root = tk.Tk()
    root.geometry("140x70")

    # label showing the current time
    label = tk.Label(root, text=get.time_ms)
    label.pack(pady=10)
    update_label()

    # Text field that updates the frequency on Enter
    text_field = tk.Entry(root)
    text_field.pack()
    text_field.bind("<Return>", change_frequency)

    root.mainloop()

# run tkinter as a thread for non-blocking execution of the successive nk.run() code
import threading
threading.Thread(target=run_gui, daemon=True).start()

nk.run()
```

## live camera

You can access a live camera frame with [get.camera()](cameras.md).

Depending on which image datatype your gui of choice works with, you can define the Camera()'s ui_view_format as 'py5' or 'numpy' to retrieve the frame as a Py5Image or numpy ndarray respectively. In the following py5/krakengui-based example `ui_format=py5' makes sense, and the retrieved current frame is displayed in the draw loop. Other frameworks will typically require you to regularly update the displayed image in the same process used in the previous examples to update the displayed current experiment time.

```python
from neurokraken import Neurokraken, State
from neurokraken.configurators import devices, Camera
from neurokraken.controls import get

camera = Camera(name='my_camera', idx=0, width=1280, height=720, fps=60,
                ui_view_enabled=True, ui_view_format='py5',
                ui_view_step=2, ui_view_scale=0.3,)

nk = Neurokraken(serial_in={}, serial_out={}, cameras=camera, mode='keyboard', log_dir='./')

class Nothing(State):
    pass

nk.load_task({'my_state': Nothing(next_state='my_state')})

# ----------------------- UI -----------------------

from py5 import Sketch
import krakengui as gui

class UI(Sketch):
    def settings(self):
        self.size(400, 400)

    def setup(self):
        pass

    def draw(self):
        camera_frame = get.camera(0, preview=True)
        self.background(0)
        self.image(camera_frame, 0, 0)

        if get.quitting:
            self.exit_sketch()

ui = UI()
ui.run_sketch(block=False)

nk.run()
```

## UI interactions and live-plotting

The following example has the form of an alternating-side shape steering task performed by an artificial agent, which provides a rich range of possible UI-based interactions and live data visualizations.

A rectangle alternatingly has to be steered over a line displayed on the left or right side of the screen. Interactions are exemplified with a button, a slider, and a text input to interact with the task and the log as well as bindings to quit the task upon closing the UI.

The button toggles the color of the moved rectangle rendered between cyan and red. The slider controls the reward size, which can be confirmed in the log under the reward device's name `get.['controls']['reward']`. The text field will append its content and a timestamp to a log list `get.log['notes']` that was self-added beforehand. This can be useful to manually log observations directly as part of the running experiment log. When using text fields a common pitfall is that, their data type commonly is a string and when using it for a numerical entry like the reward size one will have to convert number to an int() or float() beforehand to avoid errors.

As variables need to be shared between the task and UI a common pattern shown here is to use `get.` as a global namespace accessible to all components.

(agent_steering)=
### UI-interacting task

```python
from neurokraken import Neurokraken, State
from neurokraken.configurators import Display, devices
from neurokraken.controls import get
import py5 # for py5's noise function to drive our agent's actions

serial_in = {'steering_wheel': devices.rotary_encoder(pins=(31, 32), keys=['left', 'right'])}
serial_out = {'reward': devices.timed_on(pin=40)}

class Agent:
    def __init__(self):
        self.act_freq = 100 # 100hz
      
    def act(self):
        # noise provides a series of random steering positions we can scale to the wheel range
        get.serial_in['steering_wheel']['value'] = (py5.noise(get.time_ms / 1000) - 0.5) * 400 

nk = Neurokraken(serial_in=serial_in, serial_out=serial_out, 
                 display=Display(size=(800, 600)), agent=Agent(), mode='agent')

# Task

get.colored_red = False # button controlled 
get.log['notes'] = []   # text field controlled
get.reward_size = 50    # slider controlled
get.log['history'] = [] # log of rewarded reached sides

class Steer_alternating(State):
    def on_start(self):
        self.target_right = True
        self.position = 0
    
    def loop_main(self):
        self.position = get.read_in('steering_wheel') # for this task the range is physically limited to [-150, 150]
        reached = None
        if self.target_right and self.position > 90:
            reached = 'right'
        elif not self.target_right and self.position < -90:
            reached = 'left'
        if reached is not None:
            self.target_right = not self.target_right
            get.log['history'].append([get.time_ms, reached])
            get.send_out('reward', get.reward_size)
        return False, 0
    
    def loop_visual(self, sketch):
        sketch.background(0)
        pos_in_visual = sketch.remap(self.position, -150, 150, 0, 800)
        sketch.rect_mode(sketch.CENTER)
        if get.colored_red:
            sketch.fill(255, 0, 0)
        else:
            sketch.fill(0, 255, 255)
        sketch.rect(pos_in_visual, 300, 50, 50)
        sketch.stroke(255)
        if self.target_right:
            sketch.line(640, 0, 640, 600)
        else:
            sketch.line(160, 0, 160, 600)

task = {'steer': Steer_alternating(next_state='steer')}

nk.load_task(task)

# UI code will be inserted here

nk.run()
```

### Interacting UI

```python
... # The entire task above until just before nk.run()

# UI-called functions
def change_color():
    get.colored_red = not get.colored_red

def change_reward_size(value):
    get.reward_size = int(min(max(value, 0), 100))
    print(f'updated reward to {get.reward_size}')

def add_note(text):
    print(f'adding timestamped note: {text} to the log')
    get.log['notes'].append([get.time_ms, text])

# UI
from py5 import Sketch
import krakengui as gui

class UI(Sketch):
    def settings(self):
        self.size(400, 400)

    def setup(self):
        with gui.Col(pos=(20, 20)) as col:
            col.add(gui.Button(label='switch color', on_click=change_color))
            col.add(gui.Slider(label='reward size', min=0, max=100, value=50, on_change=change_reward_size))
            col.add(gui.Text_Input(label='add note to log', on_enter=add_note))

    def draw(self):
        self.background(0)
        if get.quitting:
            self.exit_sketch()
    
    def key_pressed(self, e): pass # a minimal key_pressed function is required for krakengui's text input to work properly

    def exiting(self):
        get.quit()

ui = UI()
ui.run_sketch(block=False)

nk.run()
```

### Live plotting

This chapter extends the previous UI to include live plotting of the recent continuous steering positions as well as discrete data in the form of the rewarded left/right events.

For live plots we generally don't want to reprocess and plot the millions of datapoints of an advanced task every 60th of a second with the rest of the UI loop.
A common pattern is to use a repeating thread or to create and act upon a variable for the time since the last update at a lower frequency.
Similarly as neurokraken can collect 1000 datapoints per second per sensor a good practice is to reduce the amount of processed data to what is essential for the visualization, i.e. by limiting the time window and/or only using every xth datapoint.

For the live-plot we add 2 lines to the end of `def setup()` that will help us keep track of the time since we last updated the plot, and store the plot image, which starts out empty but will be regularly updated based on the former line.

At the end of `def draw()` we display the plot image, and then if sufficient time has passed, update the plot image.

Our log entries are lists of lists containing [time, value] pairs, thus at 5000 entries we find ourselves with a [5000,2] sized 2D array of lists. A nice trick to flip the axis of 2D lists like these so that we end up with one list of times and one with values to easily plot out is to use `list(zip(*<list_name>))`.

```python
def setup(self):
    ...
    self.last_plot_time = -4000
    self.plot_image = self.create_image(350, 200, self.RGB)

def draw(self):
    ...
    self.image(self.plot_image, 20, 180)  

    # update the plot every 5 seconds
    if get.time_ms - self.last_plot_time > 5_000:
        self.last_plot_time = get.time_ms
        # plot steering
        # only plot the recent time and save some compute by only checking every 3rd entry
        recent_steerings = [entry for entry in get.log['steering_wheel'][::3] if entry[0] > get.time_ms - 40_000]
        times, values = list(zip(*recent_steerings))
        times = [t / 1000 for t in times]
        plt = gui.Plot(w=350, h=200, sketch=self, x=0, y=0)
        plt.plot(times, values)

        # plot reached sides if sides have been reached
        recent_reached = [entry for entry in get.log['history'] if entry[0] > get.time_ms - 40_000]
        if len(recent_reached) != 0:
            times, sides = list(zip(*recent_reached))
            times = [t / 1000 for t in times]
            colors = [(0,255,255) if side=='left' else (255,0,255) for side in sides]
            plt.scatter(xs=times, ys=sides, color=colors, diameter=12, y_axis=1, order=['right', 'left'])
        self.plot_image = plt.show(to_py5image=True, xlabel='t_seconds', ylabel='position')      
```