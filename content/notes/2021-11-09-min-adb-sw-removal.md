---
layout: post
title:  "Installing Minimal ADB and Removing Bloatware"
date:   2021-11-09 21:00:00 +0530
categories: tweaks
tags:
    - adb
    - android
---
[//]: # (The body)

The **Android Debug Bridge (ADB)** provides an interface between an Android device and a PC. The connection can be made using USB or Wi-Fi (if on the same network). From [this XDA guide][1]:

The internal structure of the Android Debug Bridge (ADB) is based on the classic client-server architecture. There are three components that make up the entire process.

  - The client, i.e. the PC you have connected to your Android device. We are sending commands to our device from the computer.
  - A daemon (`adbd`), which runs commands on a device. The daemon runs as a background process on each device.
  - A server, which manages communication between the client and the daemon and runs as a background process on the PC.

Typically, to install **ADB** one must install the **Android SDK**, which is *>400 MB* in size. It is sufficient to install the **Android SDK Platform-Tools**, but that's *>90 MB*. For someone not looking forward to full-scale Android development, this seems like an overkill (it is). This is where the **Minimal ADB** comes in. Found in [this XDA thread][2][^vernote], it provides all the necessary files in a neat package of *~3 MB*. I use the portable version, as I don't really need to install the program.

### Working with adb

To start working with **adb** we need to go to *Developer Options*[^dops] in Settings and enable USB debugging. Now, we can fire up a shell and type
```shell
adb devices
```
(or `./adb <commands>` in the folder where **adb** is, if not in *PATH*.) which shows the devices connected via **ADB**. Initially, there will be none. On a fresh start, this will start the daemon:
```
* daemon not running; starting now at tcp:5037
* daemon started successfully
List of devices attached

```
Now if we connect our phone via USB to the computer, we would prompted for permission to debug *on our phone* (if the adb server is running - if not, run `adb devices`). If allowed, our device will show up with a serial number on the list.

#### Connecting via Wi-Fi

It is also possible to connect to the device via Wi-Fi. 

  1. **On Android 10 and lower**, the initial setup requires a USB connection. According to [this guide][5],
      - We must first connect the device via USB.
      - We set the device to listen on a port (here, *5555*) for a TCP/IP connection:

        ```shell
        adb tcpip 5555
        ```
        and disconnect the USB cable.
      - We need to find the IP address of the phone on the network. This can be usually found at *About phone > Status info > IP address*.
      - We can now run

        ```shell
        adb connect ip_addr:5555
        ```
        to connect to the device. If successful, `adb devices` will list the device.[^wifidbg] To connect to this device anytime in future, we just need to run the `connect` command with the `ip_addr` *at that time* at port *5555*.

  2. **On Android 11 (and higher?)**, no USB connection is required. Following [this guide][6],
      - We must turn on *Wireless Debugging* in *Developer Settings* on our phone. On opening the *Wireless Debugging* settings, we will have the option to *"Pair device with pairing code"*, which will show a 6-digit pairing code alongwith the IP address of the phone and a port to connect to.
      - Run
      ```
      adb pair ip_addr:port
      ```
      using values from last step. We will be asked for the pairing code next. Once entered, pairing should succeed and our phone should list the name of the PC under *Paired devices*.
      - Now we connect:
      ```
      adb connect ip_addr:port
      ```
      *This port will in general NOT be same as the previous one!* This port number is listed under the *Wireless Debugging* settings. This completes the setup. Also, in this case, the port to connect to keeps changing - so when we need to connect to the device at a later time, we need to input the correct port number as well.

**Note 1:** If we want to disconnect, we can simply run `adb kill-server`. All devices will be disconnected. 


**Note 2:** If we want a detailed list of connected devices, we can run
```shell
adb devices -l
```
which displays more info like the model number and device name.

We can access the adb shell by typing `adb shell`. This will work only if a device is connected, as the shell opens at the device root `/`.

### Uninstalling packages

`pm` is the package manager of adb. In the adb shell, we can write
```shell
m30s:/ $ pm list packages
```
(the device name is displayed first: here `m30s`)
to list all installed packages (the list will be long!). A few Linux commands also work in the shell, hence we could, say, grab only the apps with the word 'samsung' in it:
```shell
m30s:/ $ pm list packages | grep 'samsung'
```
Finally, we can use
```shell
m30s:/ $ pm uninstall –k --user 0 <package-name>
```
to uninstall packages.[^unstnote] The `-k` switch keeps data and user cache. The `--user 0` option uninstalls only for the current user (of the device).






[//]: # (Footnotes, if any)

[^vernote]: This thread was last updated on *02-09-2018* and the ADB version packaged is old (v1.0.39). Here's a [thread][3] that provides **Tiny ADB and Fastboot** was updated on *August 2021* and uses v1.0.41 (size: *~7 MB*).
[^dops]: One needs to reveal the *Developer Settings* by tapping on *Build number* in phone software information 7 times quickly in succession.
[^wifidbg]: Interestingly, this method does not require turning on *Wireless debugging* in the phone.
[^unstnote]: Without rooting a device, we technically cannot uninstall system packages using adb. This is because one does not have write permissions on the */system* partition. ([More on this][4]) We hide them from the user so these apps do not run anymore. They're still on device storage.





[//]: # (Links, if any)

[1]: <https://www.xda-developers.com/install-adb-windows-macos-linux/>
[2]: <https://forum.xda-developers.com/t/tool-minimal-adb-and-fastboot-2-9-18.2317790/>
[3]: <https://forum.xda-developers.com/t/tool-windows-tiny-adb-fastboot-august-2021.3944288/>
[4]: <https://gitlab.com/W1nst0n/universal-android-debloater/-/wikis/FAQ>
[5]: <https://developer.android.com/studio/command-line/adb/?gclid=CjwKCAjw2dD7BRASEiwAWCtCbxKzwu63T-HpLaIp8ASO1aNA6aRl7_8Wvc2LqCvf3BI3umOlQQaOtBoCQrEQAvD_BwE&gclsrc=aw.ds#wireless>
[6]: <https://developer.android.com/studio/run/device#wireless>