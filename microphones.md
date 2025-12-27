# Microphones

You can add recording microphones to your task using `configurators.Microphone` when creating the Neurokraken object.

You can list all microphones currently connected to your computer and their indices with the `list_connected_recording_devices()` tool.

```python
from neurokraken.tools import list_connected_recording_devices
list_connected_recording_devices()
```

A microphone can be added to the neurokraken object to create a recording of the task interval. You can also provide a list of microphones to record from multiple sources. After the end of your task you will find the audio file(s) in the log directory.

```python
from neurokraken.configurators import Microphone

mic = Microphone(name='mic', idx=0, sample_rate=44100, num_channels=1)
nk = Neurokraken(..., microphones=mic)
```

To use a higher end 200khz microphone:

```python
mic = Microphone(name='mic', idx=1, sample_rate=200_000, num_channels=1)
```

## name :str

Microphone name or identifier. The .wav file and log entries will be saved under this name. Defaults to '_'.

## idx :int

Microphone device index. Your first microphone is 0, the 2nd, 1, etc. Defaults to 0.
        
## sample_rate :int|None

the sample rate to be used, i.e. 44100. At `sample_rate=None` the microphone's default sample rate will be used. Defaults to None.

## num_channels :int:

The number of channels to be recorded (Mono/Stereo). Defaults to 1.