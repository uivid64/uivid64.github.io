### Change Source

This is the most important step after the installation is completed, or because the conda warehouses are all abroad, so the access speed is very slow. We need to replace the address of the warehouse with a domestic mirror source in order to use it normally. (download at normal speed instead of tortoise speed)

### Quick Start for Win Users

1. Open the start menu, you will find the program of conda's prompt (that is, in the conda folder, the program with prompt in the name, and the icon is a black console)

2. Execute the command: `conda config --set show_channel_urls yes` (copy and paste the command, press Enter to execute)

3. Go to the C drive and find the Users folder, then find the folder with your user name (for example, my name is Joe, you may be Administrator or something else by yourself), you can see the file named .condarc (Assuming your system is installed on the C drive)

4. Open the .condarc file, delete everything in it, then go to https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/, copy all the content in this box, and paste it into .condarc save and exit

5. Go back to the dark console window of the prompt just now (you won't close it, just reopen it after closing it), then execute `conda clean -i` and it will be ok

Then it is normal use. By default, conda will be the `base` environment. Of course, you can install any package you need here. If you want to create multiple isolated virtual environments, you also need to master the creation and switching of conda environments.

### Create an environment

Generally, we create by name, execute conda `create -n python` with the name you want to create.
The places where this command can be changed are the blue and red parts. The conda commands all start with conda, then create means to create an environment, then `-n` means to give the environment a name, just follow the name with a space, and then The python in the red part is the name of the module to be installed. Only one python is installed here. You can also specify the version, such as `python=3.8`.

 If you want to install anything else, you can continue to follow
Example:

`conda create -n tf python=3.8 tensorflow=2.2`

In this way, you get a conda environment named tf containing two packages python and tensorflow.

### View environment

You can execute `conda info -e` to view all environments.

### Activate (toggle) environments

After it is created, we can execute the command to activate the specified environment by name. For example, I can execute: `conda activate BTSer` to switch to my environment

If you just followed along, you can switch to your tensorflow environment with `conda activate tf`

### Install the module in the environment

Before installing, make sure that you are currently in the environment you want.
If you don't switch it first, you will not need to install in the wrong environment.
After confirmation, execute conda install module name to install. (such as `conda install scipy`)

### Delete environment

If you don't want an environment anymore, you can remove this environment by `conda remove -n environment name --all`