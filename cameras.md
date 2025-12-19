(Cameras)=
# Cameras

(camera_examples)=
## Adding cameras to your task 

Adding a camera can be as simple as adding a Camera element to your config cameras list: 

```python
from utils.configurators import Camera

cameras = [Camera()]
```

Neurokraken supports USB connectable cameras ranging from minimal embedded application camera modules and webcams to specialized high framerate, high resolution GenICam protocol cameras.

Here are 3 more typical example with specified parameters 

```python
Camera(name='Kayeton_mono_2.8-12mm',
       idx=0, width=1280, height=720,
       max_capture_fps=100, vid_fps=100,
       ui_view_enabled=True, ui_view_step=3, ui_view_scale=0.4)

import cv2
Camera(name='main_view',
       idx=0, width=1920, height=1080,
       cv2_backend=cv2.CAP_MSMF,
       max_capture_fps=40, vid_fps=40,
       color2grey=False, # a color camera
       ui_view_enabled=True, ui_view_step=1, ui_view_scale=0.3)

Camera(name='scientific_camera',
       idx=0, # 2,592 x 1,944
       capturer='harvesters', harvesters_path_GenTL_cti='C:/path/to/.cti/file',
       max_capture_fps=40, vid_fps=40,
       ui_view_enabled=True, ui_view_step=1, ui_view_scale=0.2)
```

```{note}
You can provide multiple cameras to the cameras list to use them together in the same experiment
```

## Camera() arguments

(camera_name)=
### name
- The name of the camera in the log. Defaults to `_`
### idx
- use the nth camera connected to your pc. Defaults to 0.
    - for cameras using `capturer='harvester'` idx refers to the nth connected genICam protocol camera
### width, height
- Cameras typically support different width/height combinations at different framerates. These combinations are often noted on the packaging and store webpage but can also easily be checked with the windows build-in "Camera" application (open the settings in the top left corner, then open the entry "video settings" which will show you supported resolutions, i.e. `1080p 16:9 30fps` and `720 16:9 60fps` and `480p 4:3 60fps`).
- If width and height are unspecified you may end up with resolution video data than your camera can provide.
### max_capture_fps
- The maximum rate of image aquisition. Neurokraken will capture images as fast as your camera, and computer resources allow. Setting a maximum fps can help when you i.e. only want to use a 120fps camera at 60fps. Default to 5_000.

(vid_fps)=
### vid_fps
- The framerate for the saved video. Neurokraken logs each frame time so the playback speed of these frames does not matter much, but keeping it in the neighborhood of your capturing framerate creates a reasonable speed when the video file is watched on its own. Defaults to 30.
- You can check the fps your cameras practically achieve on a given setup with `ui_utils.framerates_to_string()`
### ui_view_enabled
- Prepare a py5image for live camera display in the ui. Default to False.
- With ui_view_enabled=True you can access the current frame for display in your py5-based gui.
```python
# in your ui.py
preview = get.camera(0, preview=True)
self.image(preview, 300, 100, 200, 200)
```
- `frame = get.camera(idx, preview=False)` retrieves the full size current numpy array frame image for live processing applications/computer vision or displaying it in non-py5-based UIs.
  - This numpy array data is also available with cameras where preview=False, however high framerate, high resolution camera create a lot of pixel data which can be demanding to process at this full scale. Thus when the purpose is only a live preview in the ui, reducing the data to be displayed can save resources and allow higher framerates on limited systems.
#### ui_view_step and ui_view_scale
-  When ui_view_enabled=True, only update the the ui preview image every nth frame and reduce the size to a lower resolution.

### cv2 backend
- Neurokraken uses `cv2.CAP_DSHOW` as default backend.
- cv2 provides different capturing backends that can work differently well or not with different accessed cameras. `cv2.CAP_DSHOW` was found to be most suitable for most cameras tested in our lab.
- Try `cv2.CAP_MSMF` if you encounter issues with `cv2.CAP_DSHOW`, i.e. a 60fps camera only producing frames at 10fps.

### capturer
- the capturer defaults to `capturer='cv2'` but you can use a different capturer for different purposes or even [hack in new capturing sources](hacking_cameras)
- Cameras using the GenICam protocol require a different capturer than cv2 and can be used with `capturer='harvesters'`. See [Advanced cameras (GenICam)](GenICam) on how to configure and use these cameras effectively.
- `capturer='iio'` uses imageiio\[ffmpeg\] for frame aquisition. You need to `pip install imageio[ffmpeg]` to use it.

### color2grey (using color cameras)
- By default neurokraken processes images as single-channel greyscale (`color2grey=True`).
- If you are using a color camera and want to preserve the color channels set `color2grey=False`.
- `color2grey_use_single_RGB_channel=None` will merge channels to greyscale intelligently and is the default approach when using `color2grey=True`. If you want to extract a specific channel using `color2grey=True`, provide the respective channel index, i.e. for blue `color2grey_use_single_RGB_channel=2`.

#### turn_image
- rotate the captured image by 180 degrees - useful when you have to mount the camera inverted. Carries a small performance cost.

### save_as_vid
- Save the capture to a video file in the log folder throughout the experiment duration. Defaults to True.
- The filename will be the provided [name](camera_name) of this camera.
- The video will use the framerate provided to [vid_fps](vid_fps)

#### vid_codec
- The codec used by the videowriter. Defaults to 'mp4v'

#### vid_container
- The container used by the videowriter. Defaults to 'mp4'

#### save_as_images
- Save the frames as individual .png images. Impacts performance. Defaults to False

### stream_active
- Stream the capture to your local network
- Experimental feature, impacts performance.

#### stream_port
- When stream_active=True, Port number for streaming. Defaults to 50_000. You can access your stream through PC.IP.ADD.RESS:stream_port

#### stream_step and stream_scaling
-  When stream_active=True, only stream every nth captured frame and reduce the size to a lower resolution like 0.5
- Defaults to 10 and 0.5

(GenICam)=
## Advanced cameras (GenICam)
- advanced scientific cameras (the kind you find on sources like edmund optics) often use the GenICam protocol (Generic Interface for Cameras)
- These cameras can be accessed and used in neurokraken through the python harvesters package [https://github.com/genicam/harvesters](https://github.com/genicam/harvesters)

### Installing harvesters:
- [instructions from harvesters](https://github.com/genicam/harvesters/blob/master/docs/INSTALL.rst)
- Harvesters relies on the genicam package. Official python genicam support for newer python packages takes a while, so depending on your python version you have 2 choices as of the time of this writing with python3.13 being the most current version
  - `pip install genicam` if you run python <= 11 
  - `pip install -i https://test.pypi.org/simple/ genicam==1.5.0` if you run a newer python version,
  - see the [harvesters github issues](https://github.com/genicam/harvesters/issues/472) for information on which python the official genicam package currently supports and alternatives for newer python versions
- `pip install harvesters`
- .cti file:
  - Harvesters requires a .cti file (often referred to as the GenTL Producer)
  - While harvesters reccommends using a .cti file from Matrix Vision, they also note that some camera manufacturers may only enable their cameras to work with their own .cti files.
  - As an example of how to proceed if if the general .cti file won't work for you; For a camera from Allied Vision, the manufacturer provides camera software (Vimba 6.0) that will install itself into a chosen folder. Looking for .cti files within this folder, we can find Vimba_6.0\VimbaUSBTL\Bin\Win64\VimbaUSBTL.cti which can be used by harvesters and thus Neurokraken with harvesters. Other manufacturers should have a similar usable file amidst their respective camera software.

## on video/audio frame times

Due to a technical limitation video and audio file time can be run at a different speed from from the task's current milliseconds. The file framerate has to be decided when the file/recording is started while the capabilities of a system may still be unclear or capturing intentionally is performed as fast as the system can process at a non-standard framerate. Neurokraken saves video and audio files with timestamps to ensure that frames are accurately relateable to the respective task time and events.

We are working on an optional feature to automatically re-encode video and audio post-experiment completion for easier analysis.

(hacking_cameras)=
## Hacking cameras\.py
- Higher end manufacturers may also create an individual python api for their products that might provide more control or performance. If you want to provide neurokraken with a new video input source feel free to have a look at `utils/cameras.py`. You will find the `self.capturer` variable which defines the backend (i.e. 'cv2' or 'harvesters') used in 3 places; setup, frame retrieval, and closedown. To add a new backend add a new case in these 3 places and implement your capturer's intended behaviors.

## Troubleshooting

(autoexposure)=
### My camera is suddenly having a slow framerate or extreme contrast. It wasn't having issues before
Many cameras automatically adjust their exposure time depending on the received brightness. If you are testing a camera pointed at a dark area, the framerate may go down as a result of the camera increasing its exposure time. To reset a camera stuck with a long exposure time, move point the camera towards a brighter area or if it has a light aperture turn it more open and back to stimulate exposure adjustments. Opening the camera in the windows built-in "Camera" application also can help reset stuck camera settings. Cameras may also automatically increase their exposure time or internal processing in insufficient light conditions like night conditions or lack of suitable wavelengths in cameras that utilize a specific spectrum like near-visibleInfrared. If your camera exposes its settings you can also manually adjust the exposure to a fitting value.

## Improving camera performance
- When using high resolution/high framerate cameras like [the examples above](camera_examples) computers may reach their processing limit leading to reduced capturing speed. More modern PCs and PCs with more CPU cores will generally do better than repurposed old PCs in their capabilities. We found modern mini PCs to be surprisingly capable experiment hardware for their price. However when requiring more frames per second or a higher resolution from an important but limited camera the following steps can be taken:
  - reduce the `max_capture_fps` or `width`/`height` of other less important cameras.
  - When `ui_view=True`: increase the `ui_view_step` and lower the `ui_view_scale`
  - Process color camera data in greyscale `color2grey=True`
  - You can trade `max_capture_fps` against the resolution. i.e. on one pc a camera with `width=1920, height=1080` may achieve 60 fps but when used with `width=1280, height=720` run at 90fps. Note that cameras have a specific set of width/height combinations can will only work properly using one of these combinations.
  - GenICam cameras have a lower impact than typical USB cameras.
  - You can check the performance of your last run experiment with `python tools/performance_test.py --last`
    - If you observe latency spikes (tens of milliseconds) when using your camera revisit the cv2_backend choice and test your camera in bright light conditions against issues caused by autoexposure [(see Troubleshooting)](autoexposure)
