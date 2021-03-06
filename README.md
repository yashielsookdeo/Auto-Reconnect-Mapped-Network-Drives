# Auto-Reconnect-Mapped-Network-Drives

## Symptoms

You probably experience the following issues in Windows 10, version 1809 - 1903:

	In Windows Explorer, a red X appears on the mapped network drives.
	
	Mapped network drives are displayed as Unavailable when you run the net use command at a command prompt.
	
	In the notification area, a notification displays the following message:
		Could not reconnect all network drives.

## Workaround

Microsoft developed a solution in late November 2018, however it is buggy and is unreliable. Currently, you can work around this issue by running scripts to automatically reconnect mapped network drive when you log on the device. To do this, create two script files, and then use one of the workarounds, as appropriate.

### Scripts
#### Create a script file named MapDrives.cmd

The file should be run at a regular but not at an elevated command prompt because it should be run at the same privilege as Windows Explorer:

	PowerShell -Command "Set-ExecutionPolicy -Scope CurrentUser Unrestricted" >> "%TEMP%\StartupLog.txt" 2>&1 
	PowerShell -File "%SystemDrive%\Scripts\MapDrives.ps1" >> "%TEMP%\StartupLog.txt" 2>&1


#### Create a script file named MapDrives.ps1

The file should be run at a regular but not at an elevated command prompt because it should be run at the same privilege as Windows Explorer:

	$i=3
	while($True){
    $error.clear()
    $MappedDrives = Get-SmbMapping |where -property Status -Value Unavailable -EQ | select LocalPath,RemotePath
    foreach( $MappedDrive in $MappedDrives)
    {
        try {
            New-SmbMapping -LocalPath $MappedDrive.LocalPath -RemotePath $MappedDrive.RemotePath -Persistent $True
        } catch {
            Write-Host "There was an error mapping $MappedDrive.RemotePath to $MappedDrive.LocalPath"
        }
    }
    $i = $i - 1
    if($error.Count -eq 0 -Or $i -eq 0) {break}

    Start-Sleep -Seconds 30

	}


### Workarounds

All workarounds should be executed in standard user security context. Executing scripts in an elevated security context will prevent mapped drivers from being available in the standard user context.

#### Workaround 1: Create a startup item

[Note] This workaround works only for the device that has network access at logon. If the device has not established a network connection by the time of logon, the startup script won't automatically reconnect network drives.

   Copy the script file (MapDrives.cmd) to the following location:

    %ProgramData%\Microsoft\Windows\Start Menu\Programs\StartUp

   Copy the script file (MapDrives.ps1) to the following location:

    %SystemDrive%\Scripts\
   A log file (StartupLog.txt) will be created in the %TEMP%\ folder.
   Log off, and then log back on to the device to open the mapped drives.


#### Workaround 2: Create a scheduled task

   Copy the script file MapDrives.ps1 to the following location:

    %SystemDrive%\Scripts\
   In Task Scheduler, select Action > Create Task.
   On the General tab in the Create Task dialog box, type a name (such as "Map Network Drives") and description for the task.
   Select Change User or Group, select a local user or group (such as LocalComputer\Users) and then select OK.
   On the Triggers tab, select New, and then select At log on for the Begin the task field.
   On the Actions tab, select New, and then select Start a program for the Action field.
   Type Powershell.exe for the Program/script field.

   In the Add arguments ([optional]) field, type the following:

    -windowsstyle hidden -command .\MapDrives.ps1 >> %TEMP%\StartupLog.txt 2>&1

   In the Start in ([optional]) field, type the location (%SystemDrive%\Scripts\) of the script file.
   On the Conditions tab, select the Start only if the following network connection is available option, select Any connection, and then select OK.
   Log off, and then log back on to the device to run the scheduled task.

