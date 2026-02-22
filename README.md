# Flash LSI SAS 3008 TO IT MODE

Guide for converting LSI SAS 3008 Controller from IR mode to IT mode, in order to run ZFS on a linux environment.

Adapted from original Github repository: https://github.com/EverLand1/9300-8i_IT-Mode , and combined with this video: https://www.youtube.com/watch?v=A54VuVNVX5M


## 1. Getting the Files
Download all five of these files from this repo and put them on a USB drive. They are necessary for this guide.

- bootx64.efi&emsp;        :&emsp;      UEFI Shell Boot File
- mptsas3.rom&emsp;        :&emsp;      Legacy BIOS OROM
- mpt3x64.rom&emsp;        :&emsp;      UEFI BIOS OROM
- sas3flash.efi&emsp;      :&emsp;      Flashing tool

## 2. Preparing the USB Drive
1. To access the UEFI shell later on with the required files, you must format your USB flash drive into a bootable Free-DOS drive using Rufus. Launch Rufus, select the correct drive under **"Device"**, select **"Free-DOS"** under **"Boot Selection"**, and click **"Start"**. This should only take a couple seconds.

#### &emsp;&emsp; WARNING: ALL FILES ON THE SELECTED DRIVE WILL BE DELETED.

&emsp;&emsp;&emsp;![Rufus Free-DOS](images/rufus.png)

2. After the drive has completed formatting to Free-DOS, create a folder on the root of the USB named "EFI".
3. Inside that folder, create another folder named "Boot".
4. Move the bootx64.efi file into the "Boot" folder. **(Resulting filepath is EFI/Boot/bootx64.efi)**
5. Now add all required files from this repo to the root of the USB.

#### &emsp;&emsp; Resulting filesystem after formatting drive in Rufus and adding the required files.

&emsp;&emsp;&emsp;![USB Files](images/USB.png)

## 3. Booting the Server into UEFI Shell

1. Before powering on the machine, unplug any external drives that are on the system. This is not necessary, but it may make the process easier later.
2. Next, plug in the USB drive.
3. Access the BIOS/UEFI menu to make sure you can boot to the UEFI shell from this USB drive. This can be done through several ways, depending on your system's manufacturer, as well as the operating system or lack there of. It can usually be triggered by spamming a function key during startup.

&emsp;&emsp;- [This article](https://www.tomshardware.com/reviews/bios-keys-to-access-your-firmware,5732.html) includes UEFI/BIOS keys for popular brands, as well as Windows and Linux specific ways to access the UEFI menu.

5. Enable the system to boot to the drive in BIOS. Once again these settings vary depending on manufacturer. 

&emsp;&emsp;- "Secure Boot" to "Disabled"

&emsp;&emsp;- "Boot Mode" from "Legacy" to "UEFI" or "Auto"

  _Note: On my system, a Lenovo ThinkServer, this setting was found under "Boot Manager" then "Boot Mode". Again, manufacturer specific. THIS MAY NOT BE AVAILABLE OR NECESSARY ON ALL SYSTEMS._
  
5. Save these changes and restart the system.
6. Boot to the USB drive. If possible, you can access the boot menu during startup similarly to the BIOS menu. However, you can also go back to the BIOS menu and change the boot sequence to make the UEFI USB Drive the first device the system boots to upon startup. You will now be able to boot to the USB.

## 5. Resetting the Adapter
1. You should find yourself at a ```Shell>``` command line prompt. If you are experienced with using the terminal or are adventerous, here are some interesting [commands](https://manuais.iessanclemente.net/images/a/a6/EFI-ShellCommandManual.pdf) you can play around with while in this shell.
2. Now, to find all available drives on the system, type ```map```. This may already show up when you are first brought into the shell.
3. Your USB drive should be named something along the lines of "Removable Hard Disk". If not, choose the first drive that shows up first. Mount to the alias of that drive. For example, if the drive is named ```fs0```, type ```fs0:``` (remember the colon). Your terminal prompt should now read ```fs0:\>``` as you have successfully mounted the drive.
4. Type ```dir``` to see the filesystem on that drive (Equivalent of using ```ls``` in Linux.). If you do not see the files you put on the USB drive, retry the previous commands beginning with ```map``` until you find the correct drive.
5. Type: ```sas3flash.efi -list```

&emsp;&emsp;&emsp;This shows the SAS Address as mentioned earlier. ENSURE TO TAKE A PICTURE OF THIS SCREEN AND RECORD THE SAS Address if you haven't already. 

**You may get an error stating "Controller is not operational. A firmware download is required." Type ```quit``` to exit that prompt and continue with this guide.**

&emsp;![List](images/list.png)

6. Type this command: ```sas3flash.efi -c 0 -o -e 5```. After it completes, the firmware currently on it will be cleared

&emsp;&emsp;&emsp;This erases the running flash/bios on the RAID controller. 

&emsp;&emsp;![Noreset](images/noreset.jpg)

## 6. Flashing the Controller
1. Boot back into the UEFI shell through the USB drive.
2. Type this command: ```sas3flash.efi -o -e 6```


&emsp;![Erase](images/erase.jpg)

3. Next, type: ```sas3flash.efi -c 0 -o -f SAS9300_8i_IT.bin -b mptsas3.rom -b mpt3x64.rom
- This flashes the new UEFI and Legacy BIOS firmware with ROM

&emsp;![Flash Firmware](images/flash.jpg)

4. Lastly, type this command:```sas3flash.efi -list```
- Verify that all information including the SAS Address are correct.

## 5. Complete!
- You are now able to reboot the system and have completed this guide.

&emsp;&emsp; - Command in UEFI shell: ```reset```

## Please leave any feedback or questions!

# Resources
- MAIN TUTORIAL - [How to flash a LSI SAS 3008 HBA (e.g. IBM M1215) to IT mode - ServeTheHome Forum](https://www.servethehome.com/flash-lsi-sas-3008-hba-e-g-ibm-m1215-mode/)
- [Crossflashing of LSI 9341-8i to LSI 9300-8i - ServeTheHome Forum](https://forums.servethehome.com/index.php?threads/crossflashing-of-lsi-9341-8i-to-lsi-9300-8i-success-but-no-smart-pass-through.3522/)
- [Detailed newcomers' guide to crossflashing LSI 9211/9300/9305/9311/9400/94xx HBA and variants - TrueNAS Forum](https://www.truenas.com/community/resources/detailed-newcomers-guide-to-crossflashing-lsi-9211-9300-9305-9311-9400-94xx-hba-and-variants.54/)
- [[HOW-TO] Flashing LSI SAS HBA Controller [EFI/UEFI] - TrueNAS Forum](https://www.truenas.com/community/threads/how-to-flashing-lsi-sas-hba-controller-efi-uefi.78457/)
- Older Card Versions - [LSI SAS 2008 RAID Controller/HBA Information - ServeTheHome Forum](https://www.servethehome.com/lsi-sas-2008-raid-controller-hba-information/)
- [Outdated Firmware Version Notice - iXsystems](https://www.truenas.com/docs/files/LSI9300xxFirmwareUpdatev1_0.pdf)
