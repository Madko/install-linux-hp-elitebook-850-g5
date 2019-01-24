# Running Linux (Fedora 29) on HP Elitebook 850 G5

Sharing some experiences about running Fedora on an HP Elitebook 850 G5 laptop.


Right now this document is only half-way finished. Hopefully I will find time to provide some more information.

## What works

Basic functionality is working :

* WIFI
* HDMI to an external monitor (even with the lid closed). Tested with a Dell UZ2715Hb monitor that has 1080p max resolution.
* Trackpad
* Mouse button (to steer the mouse pointer)
* Adaptive brightness (as weird as with _Windows_)
* Fan are quieter with _Linux_

I haven't tested much more than that.

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

* RAM : 8 GB RAM
* Operating system : Windows 10 Pro
* 512 GB SSD
* 15.6" inch screen (resolution 3840x2160)

## Installation

The computer came with _Windows 10 Pro_ installed with 3 partitions. I split the C: in half to install Linux.

Here is the steps to reduce C:\ volume :

* In Windows, right click on _Start menu_ logo > Disks Management
* Right click on C:
* Reduce this volume, by default it's set on half its size
* Accept the changes and reboot

Download Fedora 29 Workstation Live x86_64 and write it on an USB drive.

If _Windows_ was pre-installed, _secure boot_ is enabled and you will get "Image did not authenticate", preventing you to boot any other OS.

Here is the steps to disable _secure boot_ :

* Boot the computer
* On the boot screen displaying HP Logo "Sure boot by HP" press F10
* Go to _Advanced_ > _Secure Boot Options_
* Choose _Legacy support enabled and secure boot disabled_
* Press F10
* Yes to save the changes
* Computer boots on a warning screen, you have to enter the displayed code :
** Press Num lock
** Enter the code
** Press "Enter"
** Computer reboots with _secure boot_ disabled

Here is the steps to boot from the USB drive :

* Boot the computer
* On the boot screen displaying HP Logo "Sure boot by HP" press F9
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



### BIOS update

TO BE EDITED:

I rebooted the laptop and went into the BIOS. From there it was possible to download
and install a newer version of the BIOS.

An even newer version of the BIOS is available from the HP web page

    HP Firmware Pack (Q78)  01.03.00 Rev.A	 16.5 MB	22 aug 2018

https://support.hp.com/us-en/drivers/selfservice/hp-elitebook-850-g5-notebook-pc/18491276

but I wonder how to install it:

https://github.com/eriksjolund/install-linux-hp-elitebook-850-g5/issues/2

## Output from some commands

### sudo lshw -sanitize

    linux@laptop:~$ sudo lshw -santize > output/lshw.txt
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
