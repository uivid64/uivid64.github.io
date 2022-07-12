`Manage with wslconfig command`
1. Set the default running linux system

#### wslconfig /setdefault <DistributionName>

  As mentioned above, if you execute wslconfig /setdefault ubuntu, then executing the bash command will run ubuntu

2. Uninstall the linux system

#### wslconfig /unregister <DistributionName>

  When there is a problem with the system, we can uninstall it and then reinstall it. Such as: wslconfig /unregeister ubuntu

3. View the installed linux system

#### wsl -l

4. Change the default user to root, here is an example of Ubuntu

#### ubuntu config --default-user root

5. WSL 2 enables Linux GUI applications to feel native and natural to use on Windows.

Launch Linux apps from the Windows Start menu
Pin Linux apps to the Windows task bar
Use alt-tab to switch between Linux and Windows apps
Cut + Paste across Windows and Linux apps

  rerequisites
You will need to be on Windows 11 Build 22000 or higher to access this feature. You can join the Windows Insiders Program to get the latest preview builds.

Installed driver for vGPU

To run Linux GUI apps, you should first install the preview driver matching your system below. This will enable you to use a virtual GPU (vGPU) so you can benefit from hardware accelerated OpenGL rendering.

Intel GPU driver for WSL
AMD GPU driver for WSL
NVIDIA GPU driver for WSL