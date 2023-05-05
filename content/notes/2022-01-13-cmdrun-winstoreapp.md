---
layout: post
title:  "Run Windows Store App from command line"
date:   2022-01-13 14:10:00 +0530
categories: tweaks
tags:
    - windows-store
    - commandline
---
[//]: # (The body)

The procedure to launch Windows Store apps manually seems to be convoluted, as there are a host of permission issues associated with it. Windows Store apps are usually stored in the folder *C:\Program Files\WindowsApps*. The folder is hidden and permissions are restricted. Apps within this folder cannot be run manually (atleast in my experience?). 

The method outlined here helps us launch such an application manually from the command line, which might be handy. We follow [this guide][1] loosely.

The command that achieves it has the format
```
explorer shell:appsfolder\<PackageFamilyName>!<ApplicationID>
```
Time to explain!

The *shell:appsfolder* location points to all the installed applications on the system. Indeed, if we type this out in the File Explorer address bar or in the Windows Run dialog, we're taken to the folder.[^accessmethod] Now we need to find the *PackageFamilyName* and *ApplicationID* of the Windows Store app of interest. 

To do so, we create a shortcut (on the Desktop) of the app of interest by right-clicking on it in the *shell:appsfolder*. Once, the shortcut is created, we open its **Properties** and under the **Shortcut** tab, we can see the **Target** field which lists exactly the `<PackageFamilyName>!<ApplicationID>` format, but this being an [advertised shortcut][2], the field will be greyed out and since the entry is long, it might not be possible to read all of it. If one can, however - our task's accomplished - just put it in the format and the application runs! Otherwise, read on.

We run a powershell instance and run the following command:
```powershell
Get-AppxPackage -Name "<app-name-wildcard>"
```
where `<app-name-wildcard>` is a wildcard string that is related to our application of interest. Basically, we use a part of the application name (which we of course, know) along with wildcard characters to find the relevent app entry using `Get-AppxPackage`. For example, this displays the entry of the **Microsoft Photos** app (bundled by default):
```powershell
Get-AppxPackage -Name "*Photo*"


Name              : Microsoft.Windows.Photos
Publisher         : CN=Microsoft Corporation, O=Microsoft Corporation, L=Redmond, S=Washington, C=US
Architecture      : X64
ResourceId        :
Version           : 2021.21110.8005.0
PackageFullName   : Microsoft.Windows.Photos_2021.21110.8005.0_x64__8wekyb3d8bbwe
InstallLocation   : C:\Program Files\WindowsApps\Microsoft.Windows.Photos_2021.21110.8005.0_x64__8wekyb3d8bbwe
IsFramework       : False
PackageFamilyName : Microsoft.Windows.Photos_8wekyb3d8bbwe
PublisherId       : 8wekyb3d8bbwe
IsResourcePackage : False
IsBundle          : False
IsDevelopmentMode : False
NonRemovable      : False
Dependencies      : {Microsoft.Photos.MediaEngineDLC_1.0.0.0_x64__8wekyb3d8bbwe,
                    Microsoft.UI.Xaml.2.6_2.62112.3002.0_x64__8wekyb3d8bbwe,
                    Microsoft.NET.Native.Framework.2.2_2.2.29512.0_x64__8wekyb3d8bbwe,
                    Microsoft.NET.Native.Runtime.2.2_2.2.28604.0_x64__8wekyb3d8bbwe...}
IsPartiallyStaged : False
SignatureKind     : Store
Status            : Ok
```
This method displays both the *PackageFamilyName* (*Microsoft.Windows.Photos_8wekyb3d8bbwe* in this case) and the location of the executable (*C:\Program Files\WindowsApps\Microsoft.Windows.Photos_2021.21110.8005.0_x64__8wekyb3d8bbwe*). A less hit-and-miss method would be to use keywords we found in the target field of the app shortcut (preferably, the beginning of the string, which we can always see) and use the `Where-Object` cmdlet to filter results using the *PackageFamilyName* itself[^shortcut]... like so:
```powershell
Get-AppxPackage | Where-Object {$_ -like "*Photo*"}
```
with that out of the way, we can now go to the folder containing the app and find the *AppxManifest.xml* file there. In this file, we need to find the *ApplicationID*.[^appid] For **Microsoft Photos**, for instance:
```xml
<Application Id="App" Executable="Microsoft.Photos.exe" EntryPoint="AppStubCS.Windows.App" ResourceGroup="AppGroup">
```
The *ApplicationID* is just "App" in this case. So, the command to launch **Microsoft Photos** from the command line then becomes:
```powershell
explorer shell:appsfolder\Microsoft.Windows.Photos_8wekyb3d8bbwe!App
```







[//]: # (Footnotes, if any)

[^accessmethod]: I personally use **Microsoft Powertoys** for the same - `> shell:appsfolder` does the trick.
[^shortcut]: The first method, if it works reliably, then creating the shortcut or going to *shell:appsfolder* is redundant.
[^appid]: A simple document search should do. We should search for the *Executable=* property of the application, as one application package may contain more than one application. 





[//]: # (Links, if any)

[1]: <https://www.tenforums.com/software-apps/57000-method-open-any-windows-10-apps-command-line.html>
[2]: <https://stackoverflow.com/a/1270833/12983399>