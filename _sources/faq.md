# FAQ

## trying to upload (or verify) my teensy code in the arduino IDE results in "internal error in mingw32_gt_pch_use_address..."
Your temp folder is very large causing an issue for arduino. Go to the following windows explorer directories; %USERPROFILE%\AppData\Local\Temp as well as %USERPROFILE%\AppData\Local\arduino\sketches and delete everything you can in these folder. Those are temporary files so their deletion shouldn't impact your system. You may have to additionally restart your PC.

## after trying out some custom cv2 camera properties the camera remains in a unwanted configuration, i.e. super high contrast

There is a chance this can be reset through windows' pre-installed camera app by within the settings switching between 60 and 50hz
