## Delete a file with `.bat` file

Create new `doument.txt`, write in:
```
DEL /F /A /Q \\?\%1

RD /S /Q \\?\%1
```
Save as `.bat` file. Move the file to the .bat file and be deleted