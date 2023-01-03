# lsi-9211-8i-firmware-upgrade
Firmware upgrade instructions for LSI 9211-8i chip based boards and EFI instructions for gigabyte based systems

This guide teaches how to upgrade firmware on the LSI 9211-8i based hba/sas controller cards. It also shows how to manage more than just one board if you have multiple installed in your system that you need to modify.

references:

https://linustechtips.com/topic/779052-lsi-9211-8i-p20-firmware/?_fromLogin=1#replyForm

http://www.0x00.to/post/2013/04/07/Flash-IBM-ServeRAID-M1015-to-LSI9211-8i-with-UEFI-mainboard

https://www.truenas.com/community/threads/lsi-sas2008-hba-aka-9211-8i-q-firmware-upgrade-to-stop-bios-detection-fix-hotswap-not-working.84689/

https://www.truenas.com/community/resources/detailed-newcomers-guide-to-crossflashing-lsi-9211-9300-9305-9311-9400-94xx-hba-and-variants.54/

https://nguvu.org/freenas/Convert-LSI-HBA-card-to-IT-mode/

https://www.reddit.com/r/AMDHelp/comments/d51z2g/boot_to_efi_shell_on_gigabyte_x570_master/

https://docs.broadcom.com/doc/12353205


Create boot disk:

The boot disk that I've created in the zip in the repo is comprised from the following:

M1015_to_LSI9211-8i.zip (12.70 mb): http://www.0x00.to/FILES%2f2013%2f04%2fM1015_to_LSI9211-8i.zip.axdx

FreeDOS 1.1: http://derek.chezmarcotte.ca/?p=340

Win32 disk image creator(Used to write FreeDOS 1.1: http://sourceforge.net/projects/win32diskimager/

broadcom latest driver 20.00.07.00: https://docs.broadcom.com/docs-and-downloads/host-bus-adapters/host-bus-adapters-common-files/sas_sata_6g_p20/9211-8i_Package_P20_IR_IT_FW_BIOS_for_MSDOS_Windows.zip

SAS2FLASH efi: https://docs.broadcom.com/docs-and-downloads/host-bus-adapters/host-bus-adapters-common-files/sas_sata_6g_p20/Installer_P20_for_UEFI.zip

The following steps were used to create the disk:

NOTE: If you'd like to simply take the repo file and extract it to your freeDOS USB disk you can do so so that you don't have to do steps 2 to 5.

Step 1. Extracted FreeDOS image and wrote to usb stick via using win32 disk image creator

Step 2. Took M1015_to_LSI9211-8i.zip and unzipped it's contents to the usb drive.

Step 3. Took latest drivers from "broadcom latest driver 20.00.07.00" and overwrote the 2118it.bin on the usb drive and same for mptsas2.rom and sas2flash_dos_rel\sas2flsh.exe file placed in root of the usb drive and overwrote the old one.

Step 4. Took the sas2flash.efi from Installer_P20_for_UEFI\sas2flash_efi_ebc_rel\sas2flash.efi and placed in root of usb drive, overwriting the old one.

step 5. Setup the EFI boot for gigabyte based systems. I took the shellx64.efi in the usb drive and renamed it to bootx64.efi. I also placed it in a new folder: efi\boot\bootx64.efi. This allows for gigabyte based systems to see the efi shell.

Creating boot disk instructions are complete. 

Clear the cards bios/firmware:

step 1. Boot from USB drive but do not boot via EFI, go into your BIOS boot loader upon boot and select boot from USB (You should see two options, one is for regular non UEFI boot and the other would be for UEFI boot. We want the non UEFI boot first.)

Step 2. List adapters and flash: 


```
### Show all adapters ###
Magarec -adplist
#### clean first card ###
megarec -writesbr 0 sbrempty.bin
megarec -cleanflash 0
### clean second card ###
megarec -writesbr 1 sbrempty.bin
megarec -cleanflash 1
```

Step 3. reboot and boot into UEFI mode of USB.

Once UEFI loader loaded, you'll need to find the disk drive letter that has the files. Usually fs0 or fs1 etc. type 
```fs0:``` or ```fs1:``` then list directory content with the ```dir``` command, and you should be seeing the main flash drives directory.

Step 4. List cards ```sas2flash.efi -listall```

Step 5. Flash first card with bios included (don't have to do this if you don't want the bios on a card. go to step 6 if you don't want any boot bios loaded onto any cards but want to flash the firmware.)
```sas2flash.efi -o -f 2118it.bin -b mptsas2.rom```

Step 6. Flash all cards with latest firmware ```sas2flash.efi -o -fwall 2118it.bin```

you should now be done!
