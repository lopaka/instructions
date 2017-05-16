# Instructions to CLEAN install Windows 10 version 1703 (aka Creators Update) (1703) on ASUS EeeBook X205TA

## Required items

* A working system to download software and create install media
* Bootable USB flash drive at least 8GB (ALL DATA WILL BE REMOVED FROM THIS DRIVE!) or a USB DVD-ROM and DVD-R media
* USB keyboard (or keyboard/mouse combo) - Windows 10 install media does not have the drivers for keyboard or trackpad

## Obtain and create install media

Go to https://www.microsoft.com/software-download/windows10 to either:

**NOTICE: THE X205TA only supports 32-bit Windows 10! It will not allow you to install 64-bit**

* Download and use the media creation tool to create a bootable USB install drive

or

* Download the ISO file to create a bootable DVD-ROM

Follow instructions to create desired install media.

## Install Windows 10

* Make sure the X205TA is off
* Connect the USB flash drive or USB DVD-ROM player with DVD.
* Connect the USB keyboard
* Start the X205TA and continue to press `F2` to get into BIOS.
* Under `Security` tab, `Secure Boot menu` -> `Secure Boot Control` set to `Disabled`, otherwise, you may get a **SECURE BOOT VIOLATION** on boot. 
* Under `Save & Exit` tab, `Save Changes` (NOT `Save Chances and Exit`)
* Lastly, while still in `Save & Exit` tab, under `Boot Override`, select the USB flash drive or DVD-ROM drive.

Install as normal using tab and space to press UI buttons if you do not have a keyboard/mouse combo.

## Post Installation

You will need to obtain a few Windows 10 32-bit drivers from [ASUS support site](https://www.asus.com/us/Laptops/ASUS_EeeBook_X205TA/HelpDesk_Download/). Minimum required are:

1. Chipset driver: Intel INF Update Driver - will get the keyboard and mouse working [(direct link at time of doc creation)](http://dlcdnet.asus.com/pub/ASUS/EeeBook/Drivers_for_Win10/chipset/Chipset_Intel_Baytrail_X205TA_Win10_32_VER112.zip)
2. ATK: ATKPackage ATKACPI driver and hotkey-related utilities [(direct link at time of doc creation)](http://dlcdnet.asus.com/pub/ASUS/EeeBook/Apps_for_Win10/ATKPackage/ATKPackage_Win10_32_VER100050.zip)
3. TouchPad: ASUS Smart Gesture [(direct link at time of doc creation)](http://dlcdnet.asus.com/pub/ASUS/EeeBook/Apps_for_Win10/SmartGesture/SmartGesture_WIN10_32_VER405.zip)
