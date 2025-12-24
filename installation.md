# Installation

## Installation Windows

### Quick Install:

Seee below for more detailed informations and alternative installation approaches if encountering issues/questions.

install the JDK from [Adoptium JDK download](https://adoptium.net/)
- make sure to checkmark "set JAVA_HOME" variable during the install

```bash
git clone https://github.com/PasseckerLab/neurokraken
cd neurokraken
conda create --name neurokraken python==3.13
conda activate neurokraken
pip install .
```

You are now ready to run and develop Neurokraken tasks using your new neurokraken environment. To run peripheral electronics further install [arduino](install-arduino)

### Detailed Install:

#### requirements:
- python
- Java JDK (for py5)
- Arduino with teensyduino (not needed for keyboard/agent mode)

##### Python

If you are new to python, the easiest way to install it is through miniconda.

- install python/miniconda from [Download-Link](https://repo.anaconda.com/miniconda/Miniconda3-latest-Windows-x86_64.exe). This link is a bit burried in a command from ([anaconda webpage](https://www.anaconda.com/docs/getting-started/miniconda/install#quickstart-install-instructions)).

```{note}
When installing miniconda, make sure to checkmark __"Add Miniconda to the system PATH environment variable"__ despite the installer reccommending against it or your system won't be able to find miniconda and python! (Registering it as default python can also be useful but is not required)
```

##### Java JDK

- Download and install the jdk (latest LTS Release jdk-21) from [Adoptium JDK download](https://adoptium.net/). Within the installer keep "Add to PATH" and "Set JAVA_HOME variable" checked. Be sure to have administrator rights for your PC or the path might not be set properly.

* If the install was successful running `java -version` in a new command line should provide you with the found version.

<details>
<summary>alternative automatic java jdk install (works only sometimes)</summary>

There is  chance you can autoinstall the jdk using the following to lines in the command prompt, this however has not yet been shown to relaiably work compared to the manual way above:

```bash
pip install install-jdk
python -c "import jdk; print('Java installed to', jdk.install('17'))"
```
</details>

(install-arduino)=
##### Arduino

- To run with your electronics instead of just keyboard/agent mode further install:
  - The [arduino IDE](https://www.arduino.cc/en/software/) 
  - [teensyduino](https://www.pjrc.com/teensy/teensyduino.html)

#### Get the repository

If you have git installed, run: 

`git clone github.com/Passeckerlab/Neurokraken`

Otherwise go to [github.com/Passeckerlab/Neurokraken](https://github.com/Passeckerlab/Neurokraken) and in the upper right you can find a green button Code --> Download ZIP. Unzipping this file will provide you with the same Neurokraken folder that the `git clone` command would and allow you to install neurokraken.

For the following steps open or move the command line to in the folder of your local neurokraken repository, i.e. after the git clone  command:

`cd Neurokraken`

#### Installation using conda (recommended)

```bash
conda create --name neurokraken python==3.13
conda activate neurokraken
pip install .
```

pip install additional packages into this environment as they become relevant in your task.

```{Note}
When using runner mode you and using a conda environment named __"neurokraken"__ you can directly run `kraken.bat` instead of `kraken.py`. It will autoactivate this environment and run kraken.py with all arguments forwarded to it. Otherwise the name you give the environment doesn't matter.
```

#### Alternative Installation Methods (venv, uv, ...)

````{toggle} test
##### venv install

Using a venv works like a conda environment, but it doesn't require a conda install, only a python.exe.

```bash
python -m venv venv_neurokraken
venv_neurokraken\Scripts\activate
pip install .
```

to activate this environment you can run the windows file path of activate.bat within that created venv folder - afterwards it can be used like an activated conda environment.
`C:\path\to\my\venv_neuerokraken\Scripts\activate.bat`

```{Note}
For automated setups you can download a python executable with `bitsadmin.exe /transfer "JobName" https://www.python.org/ftp/python/3.12.3/python-3.12.3-amd64.exe C:\path\to\save\file.exe`
```

##### uv install

Coming soon

##### single-step conda install
You can also conda create a neurokraken environment and pip install the packages in one step with the following command which also requires git as an additional dependency
```bash
conda env create -f environment.yml
```

````

##### git installation
- Git may become a dependency for some packages aquired from github instead of the pip package index.
- Git typically comes preinstalled with windows, but if it is missing you can download it from [git-scm.com](https://git-scm.com/). The installer will ask about many settings, but the default options should be perfectly fine.
- Once you have git installed (you can confirm this by typing `git` in the command line), you can use anaconda/conda to setup a python environment which includes the dependencies for the python code by either running:

##### Editable install

If your project requires changes to the neurokraken codebase, you can create an editable install. This way the executed neurokraken codebase will directly be your cloned repository rather than a copy of it within your conda environment folder and changes you perform on on the cloned codebase will directly be reflected in your executed code without needing to reinstall neurokraken after every change to the codebase.

For an editable install with your environment activated add the editable `-e` -flag to the pip install command.
`pip install -e .`

To remove the current version of neurokraken run `pip uninstall neurokraken`.

### Installation Linux

In development