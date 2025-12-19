# AI/ML/Computer Vision

## The AI processing loop

## Tracking movements with Cutie

[Cutie](https://github.com/hkchengrex/Cutie) [[2310.12982] Putting the Object Back into Video Object Segmentation](https://arxiv.org/abs/2310.12982) is a Segmentation Model that we found to be very powerful for tracking the body, limbs, pupils, and other elements related to our experiment subjects. Cutie can run on a existing recorded video or with some minor modifications live within the context of your experiment. For example you can have an experiment state be depending on where a freely moving subject is currently positioned in an open or labyrinth environment, or when 2 tracked elements interact, or where a hand is placed. Or you can run Cutie on an existing video to to find correlations post the original experiment. Cutie doesn't have to be trained/fine-tuned on its targets and can work from once-made reference images with click selected targets. Unlike posenet models that provide you with skeleton points, a segmentation model provides you with the entire pixel area of your tracked target, i.e. an arm or snout. A target center can still easily be calculated, however having access to the occupied area could possibly also enable infering further information like a subject's respiration cycle.

### Installation

Prerequisites: 

- A nvidia cuda gpu with suffient VRAM. 16GB should be ok for most tasks. (Cutie can also run on the CPU but will be very slow - skip the pytorch(CUDA) install step to go for the CPU approach)
- We tested our install for windows, though the original repository will guide you through the install for ubuntu
- conda (i.e. miniconda), pip, git

open the command line in your target folder for cutie and clone the respository

`git clone https://github.com/hkchengrex/Cutie`

move into the cloned cutie folder

`cd Cutie`

create and activate conda environment for cutie's python packages
(For usage in another project you will want to instead activate its existing environment i.e. `conda activate neurokraken` and install cutie in there)

`conda create --name cutie python==3.12`

activate the conda environment

`conda activate cutie`

Go to https://pytorch.org/get-started/locally/ and copy the command to pip install pytorch (with CUDA) into your environment, i.e.

`pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128`

Within the cutie folder you will find the package dependencies in a file `pyproject.toml`. Two packages in this list can cause trouble when installing on windows, so replace these lines to use a alternatives:

  `'cchardet >= 2.1.7',` into `'faust-cchardet',`
  `'pyqtdarktheme',` into `'pyqtdarktheme-fork',`

(in case the install then errors on netifaces, also replace the following line: `  'netifaces >= 0.11.0',` into `  'netifaces-plus',`)

Install the dependencies noted in pyproject.toml

`pip install .`

Download the pretrained models

`python cutie/utils/download_models.py`

Test your install by running the interactive gradio demo

`python interactive_demo.py --video ./examples/example.mp4 --num_objects 1`

--num_objects defines how many distinct elements you want to track.

If you receive an error about the line `qdarktheme.setup_theme("auto")` in interactive_demo.py you can comment out this line within the file and try again. Or try installing pyqtdarktheme-fork

See [Cutie/docs/INTERACTIVE.md at main · hkchengrex/Cutie · GitHub](https://github.com/hkchengrex/Cutie/blob/main/docs/INTERACTIVE.md) for how to provide a video or image-folder of your own or track multiple num_objects

### Typical Cutie Usage

A standard cutie workflow can look like this: Load a video => Select the active mask/element in the bottom left, then in the frame click-select targets of interest and right-click-unselect elements until the starting masks are good. => Commit that frame to memory as an important reference => propagate forward. If masks diverge from the target, pause propagation, click-correct how the masks should be => Commit that corrected frame to memory as an important reference => Propagate backward/forward until the neighboring frames fit the corrected tracking => keep propagating forward until the entire video is tracked or the next niche situation causes masks to diverge.

#### Parameter options

<details><summary>If you encounter a a masks sperading from the original target over extended time periods, you can follow this segment to adjust parameters to better fit for your data.
</summary>

Cutie's default parameters work very well for complete objects of the human world. However in research we often want to track more niche targets over long periods of time. With the default settings an arm-mask of a mouse for example can gradually minute by minute spread over the body. If we encounter this behavior, we want cutie to put more focus on permanent- and long-term memory and less on the short term working-memory development that might cause drift over long time periods. To shift towards long-term focus you can edit the parameters in the interactive_demo to:

`Min, working memory frames 1`

`max working memory frames 2`

`memory every 1`

`max long-term memory size 5000`

You can then scroll through your video, find representative frames of the various positions your target(s) can be in, click select your target(s) in those frames, and commit those frames to permanent memory. Those specific frames can be considered your reference images, representing the range of how your targets can look throughout your video data. After committing these frames with click selected masks to permanent memory, you can now propagate forward/backward to fill in the video.
</details>

### Tracking with reference images

In research you often have a setup that gets repeated multiple times, i.e. you have a series of videos of a rodent exploring an open environment. You can use your reference images and masks from the first rodent video to automatically track all future rodents in the same setup. When you load a video into cutie it saves the images and masks into the Cutie/workspace/{videoname}/images and /masks folder. This includes masks that cutie propagated as well as the original masks you click-created in the interactive_demo.py - so you can copy and back-up representative frames/masks for reusage. I.e. your video is the images 0000000.jpg, 0000001.jpg, 0000002.jpg...  with the corresponding masks 0000000.png, 0000001.png, 0000002.png...

To avoid click-selecting targets anew for every video, we want to replace those first few frames and masks with our representative reference frames and masks. For example we can copy the reference image/mask file pairs from a previous run into the new workspace images and masks folders, rename them into the starting frames 0000000.jpg, 0000001.jpg, 0000002.jpg and 0000000.png, 0000001.png, 0000002.png, and now when we start the interactive_demo those will be our new first video frames and masks. After committing them to permanent memory we can propagate forward and for example track a new different rodent with the once created reference from the first run. This approach works very well when you have a repeating experiment setup in which you want targets tracked.

Neurokraken's `toolkit/cutie/cutils.py` has functions that automate this process for running cutie with a folder of frames or even live video using a provided folder of reference images and a corresponding folder of reference masks to run as additional frames committed to memory before the actual video. You can find application examples in `toolkit/cutie/process_folder.py` and `live_webcam_example` and the [tracked_shuttle example task](examples).

### Considerations

There is a tradeoff between short-term and long-term focus. Short term-focus has smoother frame to frame edge transitions but comes with the risk of subelement masks slowly diffusing over the entire object with time requiring manual corrections. Long term focus prioritizes your perment-memory committed frames over recent developments but has a bit more jittery frame to frame edges.

Another tradeoff is that every frame you commit to permanent memory will increase your utilized GPU VRAM, so you cannot provide infinite reference frames and will want to choose frames that are representative of the range of how your tracked object can look throughout the video files.

### Further parameter options 

You can find the default parameters and extra options that the interactive_demo uses in cutie/config/gui_config.yaml. There you can find and edit entries corresponding to the parameters noted above as: max_mem_frames, min_mem_frames, mem_every, and max_num_tokens. Further parameters that can tried to edit for different performance/quality are amp: True and max_overall_size.

### Troubleshooting

If cutie crawls to a halt and you find your GPU memory maxed out close GPU VRAM heavy applications like other local AI.
