# Windows Package Manager

The winget command line utility enables installing applications and other packages from the command line.

## winget command
usage: winget [<command>] [<options>]

The following commands are available:
* `install`   - Installs the given package
* `show`      - Shows information about a package
* `source`    - Manage sources of packages
* `search`    - Find and show basic info of packages
* `list`      - Display installed packages
* `upgrade`   -  Upgrades the given package
* `uninstall` - Uninstalls the given package
* `hash`      - Helper to hash installer files
* `validate`  - Validates a manifest file
* `settings`  - Open settings or set administrator settings
* `features`  - Shows the status of experimental features
* `export`    - Exports a list of the installed packages
* `import`    - Installs all the packages in a file

For more details on a specific command, pass it the help argument. [-?]

The following options are available:
  -v,--version  Display the version of the tool
  --info        Display general info of the tool

More help can be found at: https://aka.ms/winget-command-help