# DotNet tools

## Document Information

- **Last Updated:** March 11, 2026
- **.NET SDK Version:** 10.0.103
- **PowerShell Version:** 7.5.4

## handle.exe

Allows to find which process is locking a file or directory.

```powershell
& 'C:\Users\mpokrzywinski\Downloads\Handle\handle.exe' 'C:\R\I\OS\ProjectTemplates\MpTest.App'
```

And now you can use the PID to kill the process:

```powershell
Stop-Process -Id 1234 -Force
```
