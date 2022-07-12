Win + R to Run :

msconfig

In System Configuration , select the Services :

pick 'Hide all Microsoft services', then Disable all the services you don't need.

Enter the following commands in this text document:

DEL /F /A /Q \\?\%1

RD /S /Q \\?\%1

For Windows network connection, use cmd:
`ipconfig/flushdns`
`ipconfig /displaydns`

For Windows icon refresh, follow:
Press the `WIN+R` key combination on the keyboard,
and type `[%USERPROFILE%\AppData\Local]` in the pop-up window to confirm.
Open the folder window to delete the `IconCache.db` icon cache file in the hidden state, and you're done.

When the icon of a program locked on the taskbar of
the computer is displayed in white, while other icons are displayed normally,follow:
Press `Windows key + R key` at the same time to open the Run dialog box,
enter `ie4uinit.exe -show` and press `Enter` to repair

For Windows Defender's DetectionHistory, use explorer.exe:(Use `administrator` privileges)
`C:\ProgramData\Microsoft\Windows Defender\Scans\History\Service\DetectionHistory`

## Regarding Windows Explorer crashes, you can try the following solutions to check the integrity of your system components and see if that resolves the issue:
## Type the following commands at an administrator command prompt:

## Type following tips:
sfc /SCANNOW and
Dism /Online /Cleanup-Image /ScanHealth

## This command will scan all system files and compare them with the official system files, scanning the computer for inconsistencies.

Dism /Online /Cleanup-Image /CheckHealth

## This command must be used when the system file is found to be damaged after the execution of the previous command.

DISM /Online /Cleanup-image /RestoreHealth

## This command restores those various system files to the official system source files.
## Reboot after completion, and then type the following command: sfc /SCANNOW,
## Check if system files are repaired.

## Opening a folder in Windows Explorer will automatically pop up a new window, how to set it to open in the same window?
1. Repair damaged system files
Run CMD as administrator: Start Menu -> All Programs -> Accessories -> Right-click the command line prompt and select Run as administrator
Enter sfc /scannow and press Enter. It is best to restart after checking.
2. Register the invalid DLL file
Open Run: Start Menu -> All Programs -> Accessories -> Run
Enter regsvr32 "%SystemRoot%\System32\actxprxy.dll" and press Enter. After the prompt is successful, enter regsvr32 "%ProgramFiles%\Internet Explorer\ieproxy.dll" and press Enter. After the prompt is successful, check whether it is back to normal.

## Open Windows PowerShell as administrator (right-click) in the Windows PowerShell folder in the Start menu.

### 1. View the app

Enter the command Get-AppxPackage to view all installed apps. Be sure to do this step, the content will be used later. After querying, copy and paste them into Notepad for subsequent steps (use the mouse to select the content to be copied, and the selected color disappears after ctrl+C, but it has actually been copied).

### 2. App uninstall

#### 2.1 Uninstall all apps at once

Enter the command Get-AppxPackage * | Remove-AppxPackage, as shown in the figure below, the red in the figure below indicates that some applications cannot be uninstalled, such as Cortana, Edge, etc.

#### 2.2 Uninstall a single app

Enter the command Get-AppxPackage *application name* | Remove-AppxPackage, as shown below. The red in the picture below is when the Xbox is uninstalled. Although this error occurs, the Xbox has basically been uninstalled. The name of the application is the last word of the "name" of the searched application. For example, in the first application in the first picture, "name" is "Microsoft.MicrosoftEdge", then the name of the application is "MicrosoftEdge" when uninstalling (this is an example , actually Edge cannot be uninstalled).

### 3. Recover deleted apps

#### 3.1 Restore all applications at once
Enter the command:
Get-AppxPackage -AllUsers | Foreach {Add-AppxPackage -DisableDevelopmentMode -Register "$($_.InstallLocation)\AppXManifest.xml"} to restore all apps.

#### 3.2 Recover some apps individually
Enter the command:
Add-appxpackage -register "C:\Program Files\WindowsApps\Microsoft.BingWeather_4.9.51.0_x86__8wekyb3d8bbwe\appxmanifest.xml" -disabledevelopmentmode
The above command is to install the weather application, which can be divided into three parts:
Add-appxpackage -register "application installation location\appxmanifest.xml" -disabledevelopmentmode
for all applications, the first and third parts are fixed Yes, the second part is the installation location of the application, which is determined according to the "InstallLocation" of the application found in the first step. The red mark in the figure below is equivalent to the above C:\Program Files\WindowsApps\Microsoft.BingWeather_4.9.51.0_x86__8wekyb3d8bbwe, so if you install a picture application, replace this part of the content with the red line in the picture.

Check the SoftwareDistribution folder. First open: C:\Windows, check the SoftwareDistribution folder. Verify the size of this folder to see if the Windows 11 update files are downloading.

There is also the obvious possibility that the update may hang longer, even if the file is not installed, in which case the SoftwareDistribution folder needs to be cleared. Specific steps are as follows:

1. Press the win+r shortcut key to open the run menu, type cmd, and run as administrator to open the command prompt window. Type the following command: net stop wuauserv and press Enter.

2. Then enter net stop bits and press Enter.

3. Navigate to C:\Windows\SoftwareDisrtibution and delete all folders.
Then continue to enter at the command prompt: net start wuauserv and press Enter.

4. Continue to enter net start bits and press Enter.
This can solve the problem that Windows 11 is stuck when downloading and updating to 100%, and you can try to update the system again.