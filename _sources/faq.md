# FAQ

## trying to upload (or verify) my teensy code in the arduino IDE results in "internal error in mingw32_gt_pch_use_address..."
Your temp folder is very large causing an issue for arduino. Go to C:\Users\%USERPROFILE%\AppData\Local\Temp and delete everything you can in this folder. Those are temporary files so their deletion shouldn't impact your system. You may have to restart your PC for your deletions to be registered.

## Why is my video so fast?

Videos are being saved using the cv2 VideoWriter which is very efficient, but requires a framerate to be provided at the start of the recording. Following the camera configuration the video may have been recorded at `max_capture_fps=90` but saved with the default `vid_fps=30`. Or if you provided a `max_capture_fps=1000` the recording proceeded as fast as your system allows, while still using `vid_fps` as the file's fps. In your log you can find the actual experiment time frames under an entry with the name of your camera which should be treated as ground truth for video frame time. We are working towards an option for automatic reencoding of video (and audio) into task time.

## after trying out some custom cv2 camera properties the camera remains in a unwanted configuration, i.e. super high contrast

There is a chance this can be reset through windows' pre-installed camera app by within the settings switching between 60 and 50hz
