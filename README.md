# pyRevit-DevHooks-Logger-Path-Fix
Certain pyRevit extension hooks fail on launch or journal updates because the logging directory hardcodes a lowercase `desktop`.

A quick fix for an `IOError: [Errno 2] Could not find a part of the path` crash within the `pyRevitDevHooks` extension. 

## The Problem
Certain pyRevit extension hooks fail on launch or journal updates because the logging directory hardcodes a lowercase `desktop` path (`%userprofile%\desktop`). This causes `.NET` and IronPython file handlers to throw a `System.IO.DirectoryNotFoundException` on Windows systems where the folder is strictly mapped to capitalized `Desktop` or redirected via OneDrive.

## How to Fix (Choose One)

### Method 1: 1-Click Automated Fix (Easiest for End-Users)
1. Download **`Fix_pyRevit_Logs.bat`** from this repository.
2. Double-click the `.bat` file to run it.
3. The script automatically targets your local AppData directory and updates the path case configuration.
4. Reload pyRevit or restart Autodesk Revit.

### Method 2: Manual Code Replacement
If you prefer to update the source extension file manually:

1. Navigate to your local pyRevit hooks library path:
   ```text
   %appdata%\pyRevit-Master\extensions\pyRevitDevHooks.extension\lib\
   
Open hooks_logger.py in a text editor (e.g., VS Code or Notepad).

Locate line 5:

Python
USER_DESKTOP = op.expandvars('%userprofile%\desktop')

4. Replace it with this robust fallback block to handle redirected paths (like OneDrive) or system casing automatically:
   ```python
   try:
       import winreg
       key = winreg.OpenKey(winreg.HKEY_CURRENT_USER, r"Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders")
       USER_DESKTOP, _ = winreg.QueryValueEx(key, "Desktop")
       USER_DESKTOP = op.expandvars(USER_DESKTOP)
   except Exception:
       onedrive_path = op.expandvars('%userprofile%\\OneDrive\\Desktop')
       if op.exists(onedrive_path):
           USER_DESKTOP = onedrive_path
       else:
           USER_DESKTOP = op.join(op.expanduser("~"), "Desktop")
Save the file and reload your pyRevit extension tab inside Revit.
