![Screenshot](img/screenshot.png)

# Dell XPS 15 9570 Catalina
A collection of all resources needed to run macOS Catalina on a Dell XPS 15 9570

## 🔍 Overview
This is more of a compilation of information and configs from various repositories and forums than a place where real development happens. This repository should contain everything needed to get Catalina up and running on your specific Dell XPS 9570 configuration.

## What works and what doesn't

| Feature | Status | Notes |
| ------------- | ------------- | ------------- |
| **Intel iGPU** | ✅ Working | Fully supported, needs some tuning if you have 1080p screen [here](#📺-display)  |
| **iMessages and App Store** | ✅ Working | Just follow the  [guide below](#ℹ️-changing-serial-number,-board-serial-number-and-smuuid) |
| **Speakers and Headphones** | ✅ Working | To fix headphones follow the [guide below](#🔈-audio)  |
| **Built-in Microphone** | ✅ Working |  |
| **Webcam** | ✅ Working | It gets recognized as internal webcam  |
| **Handoff** | 🔵 Testing |  |
| **Airdrop** | 🔵 Testing |  |
| **Unlock with Watch** | 🔵 Testing |  |
| **Wifi/BT Card** | 🔶 To replace | The stock Killer Wifi must be replaced, see [here](#📶-Wi-Fi/Bluetooth) |
| **Thunderbolt** | 🔶 Partially working | It needs to be plugged before booting up. USB-C port for energy supply can be used in hotplug too, but probably won't delivery sufficient power (XPS needs 130W, almost no usb charger can supply that amount of power).  |
| **Touchscreen** | 🔶 Works but | The touchscreen capability seems to work but the power management isn't good. [Battery drain](#1) is very high. You can [disable](#👈-touchscreen) it |
| **NVIDIA GPU** | ❌ Not working | Will never work on > High Sierra because Apple doesn't support NVIDIA GPUs anymore |
| **PM981 SSD** | ❌ Not working | Even if [NVMeFix](https://github.com/acidanthera/NVMeFix) 1.0.2 promises to fix the PM981 Kernel Panics, it doesn't. Just replace it with a SATA M.2 drive or a supported NVMe one. |
| **Fingerprint reader** | ❌ Not working | Probably will never work. NoTouchId kext disables it automatically to save power |
| **SD Reader** | ❌ Not working | Works on Windows, if you don't dualboot disable it in bios to save power |

## ⬇️ Installation
Follow this guide if you have never set up a Hackintosh before.

### Creating a bootable installer
To start you need a USB flash drive with at least 12GB of available storage and a local copy of macOS. The installer for macOS Catalina can be obtained from [here](https://itunes.apple.com/de/app/macos-catalina/id1466841314?ls=1&mt=12).

Next, you want to format the USB flash drive using Disk Utility. Click on “View” in the toolbar and choose “Show All Devices” to see all physical disks instead of only partitions. Select your USB flash drive, name it “MyVolume” and format it HFS+/Mac OS Extended (Journaled) with GUID Partition Map.

Now you are ready to create the installation media. Use the following command to start the process. It may take a while depending on the USB flash drive you are using.

```sudo /Applications/Install\ macOS\ Catalina.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume```

If your USB flash drive has a different name, replace ```MyVolume``` with the name of your volume.

After the installer says that it's done, the volume now contains the macOS Catalina installer and is ready to boot on a real Mac. However, because we are building a Hackintosh, we have to take an additional step and install the OpenCore bootloader. To do this, you first have to mount the EFI partition. This is where OpenCore and all its configuration files go. Use the following command to list all disks.

```sudo diskutil list```

Find the EFI partition of your USB flash drive. Normally its entry 1 under /dev/disk2. Use the following command to mount the EFI partition - in this case, ```disk2s1```.

```sudo diskutil mount disk2s1```

Now that you have access to the EFI partition, the real fun starts.

### Configuring EFI
Clone this repository to get the base EFI folder as well as all additional kexts and patches. Now you will have to prepare the EFI folder for your exact hardware configuration. Read through the [configuration section](#configuration) to learn more. Once everything is configured properly, copy the folder into the EFI partition you have mounted in the previous step.

### Booting the installer
After having created the installer USB flash drive, you are ready to install macOS on your XPS. Make sure SSD mode is set to AHCI mode instead of RAID in BIOS otherwise, macOS won't be able to detect your SSD. Select your USB flash drive as boot media and go through the macOS installer like you would on a real mac. Once you have come to the desktop, advance to the next step.

### Post Installation
Congratulations! You have successfully booted and installed macOS. At this point, you just have to copy the EFI folder you have prepared in a previous step to the SSD. Mount the EFI partition of your SSD with

```sudo diskutil mount disk0s1```

and copy your customized EFI folder into the newly mounted EFI partition. You should now be able to boot your computer without the USB flash drive attached. If you're having issues with specific parts like Wi-Fi, Bluetooth or Audio, have a look at the corresponding sections in this repository and open an issue if you are unable to solve them.

## 🛠 Configuration
This section talks about configuring the EFI folder for your exact hardware.

Almost all changes are done inside the OpenCore configuration file. Use the provided version of [ProperTree](https://github.com/corpnewt/ProperTree) to edit ```EFI/OC/config.plist```.

### 🔈 Audio
By default, the audio can be a bit buggy. When using headphones, after some time the audio randomly stops. Sometimes un- and replugging the headphones works, but that's pretty annoying and unreliable. To permanently fix this issue you will have to install [ComboJack](https://github.com/hackintosh-stuff/ComboJack/tree/master/ComboJack_Installer) from the ```tools``` folder by running ```install.sh```.

### 📶 Wi-Fi/Bluetooth
The stock Killer Wi-Fi card will never be supported in macOS. So to use Wi-Fi, you will have to replace it for a supported card. This repository is configured to work out-of-the-box with both the Dell DW1830 and DW1560 wireless adapter.

Another option for a fraction of the price is the [Dell DW1820a](https://www.aliexpress.com/item/32918457901.html). It is a bit slower than the cards mentioned above and in my experience has problems with some 5GHz networks, but with a minor modification, you can make it work pretty reliably. The only change that has to be done is disabling “Active State Power Management”. This can be done by inserting the following entry under ```Root/DeviceProperties/Add```.

```xml
<key>PciRoot(0x0)/Pci(0x1c,0x0)/Pci(0x0,0x0)</key>
<dict>
    <key>pci-aspm-default</key>
    <integer>0</integer>
</dict>
```

### 📺 Display
This repository is configured to work properly with the 4K display. If you have a FHD screen, you should apply two minor changes:

- Change ```dpcd-max-link-rate``` in ```Root/DeviceProperties/Add/PciRoot(0x0)/Pci(0x2,0x0)``` from ```14000000``` to ```0A000000```
- Change ```UIScale``` in ```Root/NVRAM/Add/4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14``` from ```02``` to ```01```

Also, I strongly suggest enabling subpixel antialiasing for the FHD screen.

```defaults write -g CGFontRenderingFontSmoothingDisabled -bool NO```

### 🔋 Power management
Hibernation is not supported on a Hackintosh and everything related to it should be completely disabled. Disabling additional features prevents random wakeups while the lid is closed. After every update, these settings should be reapplied manually.

```
sudo pmset -a hibernatemode 0
sudo rm -f /var/vm/sleepimage
sudo mkdir /var/vm/sleepimage
sudo pmset -a standby 0
sudo pmset -a autopoweroff 0
sudo pmset -a powernap 0
sudo pmset -a proximitywake 0
sudo pmset -b tcpkeepalive 0 (optional)
```

### ⚡️ Performance
CPU power management is done by ```CPUFriend.kext``` while ```CPUFriendDataProvider.kext``` defines how it should be done. ```CPUFriendDataProvider.kext``` is generated for a specific CPU and power setting. The one supplied in this repository was made for the i7-8750H and is optimized for maximum performance. In case you have another CPU or would rather sacrifice some performance in trade for more battery life, you can use the script [one-key-cpufriend](https://github.com/stevezhengshiqi/one-key-cpufriend) from the ```tools``` folder to generate your own ```CPUFriendDataProvider.kext```.

### 👈 Touchscreen
If you don't need the touchscreen you can disable it to save power. Replace ```SSDT-TPDX.aml``` in ```EFI/OC/ACPI``` with the one inside the ```resources``` folder.

### ℹ️ Changing Serial Number, Board Serial Number and SmUUID
NOTE: With the stock Killer Wi-Fi card, iMessage will never work.

To use iMessage and other Apple services, you need to generate your own serial numbers. This can be done using [Hackintool](https://www.tonymacx86.com/threads/release-hackintool-v3-x-x.254559/). Go to the “Serial“ tab and make sure model is set to ```MacBookPro15,1```. Use the barcode-with-apple button to check your generated serial numbers. If the website tells you that the serial number isn't valid, everything is fine. Otherwise, you have to generate a new set.

Next you will have to copy the following values from Hackintool to your ```config.plist```:
- Serial Number -> ```Root/PlatformInfo/Generic/SystemSerialNumber```
- Board Number -> ```Root/PlatformInfo/Generic/MLB```
- SmUUID -> ```Root/PlatformInfo/Generic/SystemUUID```

Reboot and Apple services should work.

## 🔧 Tweaks
This section talks about various optional tweaks that enhance your experience

### ⤵️ Undervolting
Undervolting your CPU can reduce heat, improve performance and provide longer battery life. However, if done incorrectly, it may cause an unstable system. The ```tools``` folder contains a patched version of [VoltageShift](https://github.com/sicreative/VoltageShift).

Using ```./voltageshift offset <CPU> <GPU> <CPUCache>``` you can adjust the voltage offset for the CPU, GPU, and cache. Safe starting values are ```-100, -75, -100```. From there you can start gradually lowering the values until your system gets unstable.

## 🤔 Frequently Asked Questions

### Why my trackpad doesn't work?
In macOS, the “PrintScreen“ button (PrtScr, to the right of F12) disables/enables the trackpad instead of disabling/enabling wifi as it says. Probably you just pressed that button by accident.

### I have a Samsung PM981 SSD, will it work?
The Samsung PM981 (or more precise the controller it uses) is known to cause random kernel panics in macOS. Up until now, there was no way to even install macOS on the PM981 and the only option was to replace it with either a SATA or a known working NVMe SSD. However, recently a new set of patches, namely [NVMeFix](https://github.com/acidanthera/NVMeFix) was released. It greatly improves compatibility with non-apple SSDs including the PM981. Thanks to those patches, you can now install macOS, but there is still a chance for kernel panics to occur while booting. In this case, you can clear the NVRAM from the OpenCore boot menu as a workaround to get it booting again. Since I don't own a PM981 myself, I can't debug this issue, but [frbuccoliero](https://github.com/frbuccoliero) is currently testing a potential fix.

### My XPS reboots once the loading bar with the apple has reached approximately 90%
You probably have a Samsung PM981 SSD. As stated above, we're working on a permanent fix.  In the meantime, you will have to reset NVRAM from the OpenCore menu before each boot.

## Acknowledgments
- [acidanthera](https://github.com/acidanthera) for providing almost all kexts and drivers
- [alexandred](https://github.com/alexandred) for providing VoodooI2C
- [daliansky](https://github.com/daliansky) for providing the awesome hotpatch guide [OC-little](https://github.com/daliansky/OC-little/) and always up-to-date Hackintosh solutions in [XiaoMi-Pro-Hackintosh](https://github.com/daliansky/XiaoMi-Pro-Hackintosh)
- [RehabMan](https://github.com/RehabMan) for providing many laptop [hotpatches](https://github.com/RehabMan/OS-X-Clover-Laptop-Config/tree/master/hotpatch) and guides
- [knnspeed](https://www.tonymacx86.com/threads/guide-dell-xps-15-9560-4k-touch-1tb-ssd-32gb-ram-100-adobergb.224486) for providing ComboJack, well-explained hotpatches and a working USB-C hot plug solution
- [bavariancake](https://github.com/bavariancake/XPS9570-macOS) and [LuletterSoul](https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS-Mojave) for providing detailed installation guides and configurations for the XPS 9570
- [xxxzc](https://github.com/xxxzc/xps15-9550-macos) for providing OpenCore support for the XPS 9570
- [frbuccoliero](https://github.com/frbuccoliero) for PM981 related testing and extending the guide
- Everyone else involved in Hackintosh development
