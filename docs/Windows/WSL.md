# Windows Subsystem for Linux

## wslconfig command

* `wslconfig /setdefault <DistributionName>` - Set the default running linux system. As mentioned above, if you execute wslconfig /setdefault ubuntu, then executing the bash command will run ubuntu
* `wslconfig /unregister <DistributionName>` - Uninstall the linux system. When there is a problem with the system, we can uninstall it and then reinstall it. Such as: wslconfig /unregeister ubuntu
* `wsl -l` - View the installed linux system.
* `ubuntu config --default-user root` - Change the default user to root, here is an example of Ubuntu.
