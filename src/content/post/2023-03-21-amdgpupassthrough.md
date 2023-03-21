+++
title= "amdgpupassthrough"
date = "2023-03-21T11:53:13+08:00"
description = "amdgpupassthrough"
keywords = ["Technology"]
categories = ["Technology"]
+++
Install virt-io drivers:    

![/images/2023_03_21_11_53_28_763x591.jpg](/images/2023_03_21_11_53_28_763x591.jpg)

![/images/2023_03_21_11_53_54_593x400.jpg](/images/2023_03_21_11_53_54_593x400.jpg)

Then the amd gpu:    

![/images/2023_03_21_11_54_24_330x162.jpg](/images/2023_03_21_11_54_24_330x162.jpg)

Get the driver:    

![/images/2023_03_21_11_56_45_732x511.jpg](/images/2023_03_21_11_56_45_732x511.jpg)

![/images/2023_03_21_11_57_03_847x445.jpg](/images/2023_03_21_11_57_03_847x445.jpg)

Copy the file into :    

![/images/2023_03_21_14_59_59_792x170.jpg](/images/2023_03_21_14_59_59_792x170.jpg)

Then add it into shutdown items:    

![/images/2023_03_21_15_00_37_919x622.jpg](/images/2023_03_21_15_00_37_919x622.jpg)

Edit its sequence:    

![/images/2023_03_21_15_01_51_470x422.jpg](/images/2023_03_21_15_01_51_470x422.jpg)

eject script:    

```
Function Check-RunAsAdministrator()
{
  #Get current user context
  $CurrentUser = New-Object Security.Principal.WindowsPrincipal $([Security.Principal.WindowsIdentity]::GetCurrent())
  
  #Check user is running the script is member of Administrator Group
  if($CurrentUser.IsInRole([Security.Principal.WindowsBuiltinRole]::Administrator))
  {
       Write-host "Script is running with Administrator privileges!"
  }
  else
    {
       #Create a new Elevated process to Start PowerShell
       $ElevatedProcess = New-Object System.Diagnostics.ProcessStartInfo "PowerShell";
 
       # Specify the current script path and name as a parameter
       $ElevatedProcess.Arguments = "& '" + $script:MyInvocation.MyCommand.Path + "'"
 
       #Set the Process to elevated
       $ElevatedProcess.Verb = "runas"
 
       #Start the new elevated process
       [System.Diagnostics.Process]::Start($ElevatedProcess)
 
       #Exit from the current, unelevated, process
       Exit
 
    }
}
 
#Check Script is running with Elevated Privileges
Check-RunAsAdministrator
 
#Place your script here.
     $deviceName="AMD Radeon(TM) Graphics"
foreach ($dev in (Get-PnpDevice | Where-Object{$_.Name -eq $deviceName})) {
  &"pnputil" /remove-device $dev.InstanceId 
}
# sleep for 8 seconds
Start-Sleep -Seconds 8

#Read more: https://www.sharepointdiary.com/2015/01/run-powershell-script-as-administrator-automatically.html#ixzz7wTuWTBny% 
```
