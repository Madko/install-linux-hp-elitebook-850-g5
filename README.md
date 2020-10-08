# Running Linux (Fedora 29 or newer) on HP Elitebook 850 G5

Sharing some experiences about running Fedora on an HP Elitebook 850 G5 laptop.

Right now this document is only half-way finished. Hopefully I will find time to provide some more information.

I first was using Fedora 29, then

 * upgraded to Fedora 30 => same experience
 * upgraded to Fedora 31 => same experience, almost everything is OK
 * upgraded to Fedora 32 => same experience, almost everything is OK

I will try to keep this document up to date.

## What works

Basic functionality is working (kernel 5.8.13-200.fc32.x86_64) :

* WIFI
* Keyboard, keyboard backlight and shortcuts
* Touchpad
* Mouse button (to steer the mouse pointer)
* Adaptive brightness (as weird as with _Windows_)
* Webcam
* HDMI external port (up to 4K)
* Thermal sensors
* Thunderbolt 3 port

What is not working :

* Fingerprint sensor
* Mic mute button

## Customization of the laptop

The laptop configuration was like this :

    HP EliteBook 850 G5
    CPU Intel i7-8550U
    Windows 10 PRO
    Webcam + IR
    15.6 inch UHD Anti-Glare LDE
    8GB (1x8GB) DDR4 2400 
    512GB PCIe NVMe Three Layer Cell Solid State Drive 
    No Near Field Communication (No NFC) 
    Intel 8265 ac 2x2 nvP +Bluetooth 4.2 WW with 2 Antennas 
    No WWAN ??
    Fingerprint Sensor 
    No Active SmartCard 
    3 Cell 56 WHr Long Life 
    45 Watt Smart nPFC Right Angle AC Adapter 
    C5 1.8m Sticker Conventional Power Cord SE/FI 
    3/3/0 Warranty EURO 
    No vPro AMT supported 
    DIB HP Basic Carrying Case 
    Country Localization SE/FI 
    Dual Point with numeric keypad spill-resistant Collaboration SE/FI 
    EU RED Pictogram Label 
    Core i7 G8 Label 
    HP 1 year Priority Management Service for PCs (1000+ seats) 
    HP 3 year Next Business Day Onsite Notebook Only Hardware Support 
    HP Operations Service

The laptop was delivered to me in the end of January 2019.

Short summary of hardware configuration :

* RAM : 8 GB
* Operating system : Windows 10 Pro
* Storage : 512 GB SSD
* Display : 15.6" inch screen (resolution 3840x2160)

Upgrade I made :

* RAM : 32 GB 
* Operating system : Fedora
* Storage : 1 TB SSD
* WWAN LTE : Huawei ME906s bought in october 2020 for ~30€

## Installation

The computer came with _Windows 10 Pro_ installed with 3 partitions :

* EFI (360 MB) + hidden partition maybe for Windows restoration (128 MB)
* Windows C:\ (~475 GB)
* Restore tools (915 MB)

Just split the C:\ in half (~235 GB) to install Linux.

Here is the steps to reduce C:\ volume :

* In Windows, right click on _Start menu_ logo > _Disks Management_
* Right click on C:\
* Reduce this volume, by default it's set on half its size
* Accept the changes and reboot

Download **Fedora 31 Workstation Live x86_64** and write it on an USB drive.

If _Windows_ was pre-installed, _secure boot_ is enabled and you will get "Image did not authenticate", preventing you to boot any other OS.

Here is the steps to disable _secure boot_ :

* Boot the computer
* On the boot screen displaying HP Logo "Sure boot by HP" press **F10**
* Go to _Advanced_ > _Secure Boot Options_
* Choose _Legacy support enabled and secure boot disabled_
* Press **F10**
* Yes to save the changes
* Computer boots on a warning screen, you have to enter the displayed code :
  * Press Num lock
  * Enter the code
  * Press "Enter"
  * Computer reboots with _secure boot_ disabled

Here is the steps to boot from the USB drive :

* Boot the computer
* On the boot screen displaying HP Logo "Sure boot by HP" press **F9**
* Choose the USB UEFI entry

Proceed with usual Fedora installation :

* Review the partition scheme (default/automatic was fine)
* Launch the installation
* Reboot

User creation will be at first boot.

Here is the partition table :

    [edouard@hp850g5 ~]$ lsblk -o NAME,MOUNTPOINT,LABEL,FSTYPE
    NAME            MOUNTPOINT LABEL            FSTYPE
    nvme0n1                                     
    ├─nvme0n1p1     /boot/efi  SYSTEM           vfat
    ├─nvme0n1p2                                 
    ├─nvme0n1p3                Windows          ntfs
    ├─nvme0n1p4                Windows RE Tools ntfs
    ├─nvme0n1p5     /boot                       ext4
    └─nvme0n1p6                                 LVM2_member
      ├─fedora-root /                           ext4
      ├─fedora-swap [SWAP]                      swap
      └─fedora-home /home                       ext4


The EXT4 file system is used with LVM, ie default partitionning scheme from Fedora.

## Tuning

### Hanging reboots

It seems that by default, Fedora (or systemd) is not sending the right kind of signal this machine is waiting for reboot. To fix that, you need to add the type of signal at the boot options. Here are the steps, first add _reboot=efi_ at the very end of the line **GRUB_CMDLINE_LINUX=** :

    sudoedit /etc/default/grub

You should have this content :

    GRUB_TIMEOUT=5
    GRUB_DISTRIBUTOR="$(sed 's, release .\*$,,g' /etc/system-release)"
    GRUB_DEFAULT=saved
    GRUB_DISABLE_SUBMENU=true
    GRUB_TERMINAL_OUTPUT="console"
    GRUB_CMDLINE_LINUX="resume=/dev/mapper/fedora-swap rd.lvm.lv=fedora/root rd.lvm.lv=fedora/swap rhgb quiet reboot=efi"
    GRUB_DISABLE_RECOVERY="true"

Then you need to regenerate grub2 actual configuration file¹ :

    sudo grub2-mkconfig -o /etc/grub2-efi.cfg

Then to validate that's has been taken, you reboot and check cmdline :

    cat /proc/cmdline

You must see the _reboot=efi_ there.

[¹] If for any reason you are not using EFI and secure boot, the right output file is _/etc/grub2.cfg_.

### Sound mute button _LED_

The shortcut button to mute sound is working fine but the _LED_ on it doesn't switch on to indicate its state.

To activate the _LED_, create a file to _/etc/modprobe.d/sound.conf_ with the following content :

    options snd-hda-intel model=mute-led-gpio

Reboot the computer (or unload/reload module _snd-hda-intel_).

### Power Management 

Powertop utiliy is a great tool to fix some power consumption issues.

    sudo dnf install powertop
    sudo powertop

In _tunables_ section a few setups by default are bad. Select them and press _space bar_ to activate them. Press _ESC_ to exit. Then enable _powertop_ service at boot to make these changes permanent :

    sudo systemctl enable powertop

Voilà !

### CPU frequencies and _TurboBoost_

To have more control over performance, the Gnome Shell Extension [_CpuFreq_](https://extensions.gnome.org/extension/1082/cpufreq/) is perfect. With this extension you can check the actual frequencies and choose which governor you want (powersave, performance etc).

I prefere to disable _TurboBoost_, because I don't really see any difference on performance. But with _TurboBoost_ off, fans are off too !

## _BIOS_ update

I rebooted the laptop and went into the _BIOS_. From there it was possible to download
and install a newer version of the _BIOS_.

Actual version of the _BIOS_ :

    Version: Q78 Ver. 01.06.00
    Release Date: 01/03/2019 

An even newer version of the _BIOS_ is available from the HP web page :

    HP Firmware Pack (Q78) 01.06.00 Rev.A	16.6 MB	Feb 1, 2019

https://support.hp.com/us-en/drivers/selfservice/hp-elitebook-850-g5-notebook-pc/18491276

How to install it :

https://github.com/eriksjolund/install-linux-hp-elitebook-850-g5/issues/2

## Output from some commands

### sudo lshw -sanitize

    linux@laptop:~$ sudo lshw -sanitize > output/lshw.txt
    [sudo] password for linux:
    linux@laptop:~$

See the output in [output/lshw.txt](output/lshw.txt)

### lsmod

    linux@laptop:~$ lsmod > output/lsmod.txt
    linux@laptop:~$

See the output in [output/lsmod.txt](output/lsmod.txt)

### cat /proc/cpuinfo

    linux@laptop:~$ cat /proc/cpuinfo > output/cpuinfo.txt
    linux@laptop:~$

See the output in [output/cpuinfo.txt](output/cpuinfo.txt)

### cat /proc/meminfo

    linux@laptop:~$ cat /proc/meminfo > output/meminfo.txt
    linux@laptop:~$

See the output in [output/meminfo.txt](output/meminfo.txt)

### dmesg

    linux@laptop:~$ dmesg > output/dmesg.txt
    linux@laptop:~$

See the output in [output/dmesg.txt](output/dmesg.txt)

### Thermal sensors 

    linux@laptop:~$ sensors > output/sensors.txt
    linux@laptop:~$

See the output in [output/sensors.txt](output/sensors.txt)

## Other references

Original version of this document was forked from [eriksjolund](https://github.com/eriksjolund/install-linux-hp-elitebook-850-g5) (thanks).
I've found this link on [reddit](https://www.reddit.com/r/Hewlett_Packard/comments/8ygk16/linux_on_hp_elitebook_850_g5/).

Gentoo wiki also have some information [here](https://wiki.gentoo.org/wiki/HP_EliteBook_850_G5).
