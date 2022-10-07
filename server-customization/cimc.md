# CIMC

## CIMC KVM

```
https://<CIMC_IP>/kvm.jnlp?cimcAddr=<CIMC_IP>&tkn1=<CIMC_username>&tkn2=<CIMC_Password>
```

## Workaround FLASH-player

Use Pale Moon 32-bits browser with flash before 2021 (portable version downloaded from [here ](https://ia600407.us.archive.org/view\_archive.php?archive=/14/items/Pale-Moon-25.x-Files/Pale-Moon-25.x-files.zip)works fine), otherwise Flash kill-switch will show

![](../.gitbook/assets/flash\_kill\_switch.png)

Another workaround could be tried ([source link](https://community.cisco.com/t5/cisco-bug-discussions/cscvs11682-c220-c240-m3-server-need-html5-support-for-cimc-webui/td-p/4144939))

> Here is how I was able to get it to work...
>
> &#x20;
>
> Download and install K-Meleon browser (I used the installer from PortableApps\[.]com it should also work from the direct installer)
>
> Download and install HxD (freeware hex editor)
>
> Download and install Flash (I used FlashPlayer 32.0.0.465)
>
> &#x20;
>
> Once you have the software installed, open K-Meleon and navagate to "about:plugins" this will show you which version and path of flash that the browser is using. in my case it was "C:\WINDOWS\SysWOW64\Macromed\Flash\NPSWF32\_32\_0\_0\_465.dll"
>
> &#x20;
>
> Run HxD as administrator and open "NPSWF32\_32\_0\_0\_465.dll"
>
> &#x20;
>
> Search for:
>
> 00 00 40 46 3E 6F 77 42
>
> replace it with:
>
> 00 00 00 00 00 00 F8 7F
>
> &#x20;
>
> Then disable the EOL Uninstall Warnings
>
> Run notepad as administrator and Edit "C:\Windows\SysWOW64\Macromed\Flash\mms.cfg"
>
> Add a line with "EOLUninstallDisable=1"
>
> save and close
>
> After that it should work just fine.
>
> I had a very old CIMC that an error SSL\_ERROR\_NO\_CYPHER\_OVERLAP
>
> to Fix this, I had to change some of the settings in "about:config"I don't remember which ones did the trick but where there is a will there is a way..

## HDD info&#x20;

```
scope chassis
show storageadapter
```

```
PCI Slot     Health         Controller Status ROC Temperature Product Name                       Serial Number  Firmware Package Build Product ID Battery Status Cache Memory Size Boot Drive     Boot Drive is PD 
------------ -------------- ----------------- --------------- ---------------------------------- -------------- ---------------------- ---------- -------------- ----------------- -------------- -------------- 
SLOT-2       Good           Optimal           49 degrees C    LSI MegaRAID SAS 9266-8i           SV31736026     23.33.1-0058           LSI Logic  Optimal        866 MB            0              false          
```

Line (3) make note of a PCI Slot name

```
scope storageadapter SLOT-2
show physical-drive
```

```
Physical Drive Number Controller Device ID      Health         Status                 Boot Drive Manufacturer   Model          Predictive Failure Count Drive Firmware Type  Block Size Physical Block Size Negotiated Link Speed Locator LED FDE Capable FDE Enabled FDE Secured FDE Locked Coerced Size   
--------------------- ---------- -------------- -------------- ---------------------- ---------- -------------- -------------- ------------------------ -------------- ----- ---------- ------------------- ---------- ----------- ----------- ----------- ----------- ---------- -------------- 
1                     SLOT-2     17             Good           Online                 false      ATA            ST9500620NS    0                        CC02           HDD   512        512                 6.0 Gb/s   false       0           0           0           0          475883 MB      
2                     SLOT-2     18             Good           Online                 false      ATA            ST9500530NS    0                        CC04           HDD   512        512                 3.0 Gb/s   false       0           0           0           0          475883 MB      
3                     SLOT-2     20             Good           Online                 false      ATA            ST9500620NS    0                        CC02           HDD   512        512                 6.0 Gb/s   false       0           0           0           0          475883 MB      
4                     SLOT-2     22             Good           Online                 false      ATA            ST9500620NS    0                        CC02           HDD   512        512                 6.0 Gb/s   false       0           0           0           0          475883 MB      
5                     SLOT-2     19             Good           Online                 false      ATA            ST9500620NS    0                        CC02           HDD   512        512                 6.0 Gb/s   false       0           0           0           0          475883 MB      
6                     SLOT-2     23             Good           Online                 false      ATA            ST9500620NS    0                        CC02           HDD   512        512                 6.0 Gb/s   false       0           0           0           0          475883 MB      
7                     SLOT-2     16             Good           Online                 false      ATA            ST9500530NS    0                        CC04           HDD   512        512                 3.0 Gb/s   false       0           0           0           0          475883 MB      
8                     SLOT-2     21             Good           Online                 false      ATA            ST91000640NS   0                        CC03           HDD   512        512                 6.0 Gb/s   false       0           0           0           0          952720 MB      
```

Lines (3) and below, look at the Health column

## CIMC upgrade with HUU on USB

Proper CIMC upgrade method is using KVM.&#x20;

USB of any kind: USB-stick, internal FlexFlash RAID will fail with message:&#x20;

![](<../.gitbook/assets/image (7).png>)

This is the bug: [https://bst.cloudapps.cisco.com/bugsearch/bug/CSCup62091](https://bst.cloudapps.cisco.com/bugsearch/bug/CSCup62091)

Problem with incorrect way of writing ISO on USB. Very good explanation of workaround is here: [http://vstudio.zohosites.com/blogs/post/create-cisco-huu-usb-disk](http://vstudio.zohosites.com/blogs/post/create-cisco-huu-usb-disk)

As well discussion: [https://community.cisco.com/t5/unified-computing-system/cisco-standalone-c-series-host-update-utility-usb-image-utility/ta-p/3638625/page/3/show-comments/true?attachment-id=158721](https://community.cisco.com/t5/unified-computing-system/cisco-standalone-c-series-host-update-utility-usb-image-utility/ta-p/3638625/page/3/show-comments/true?attachment-id=158721)

#### My case

I accidentally downgraded CIMC down to 1.5(7f). After that CIMC got broken , I can't even get to it from BIOS -> F8. CIMC throw error message smth like "I'm broke". No web interface as well.

I've tried:

* HUU image on USB stick - failed with message 906
* HUU image on SCU partition of FlexFlash - failed with message 906
* tried SCU to update HUU image - failed with accessing to HUU.iso either on cisco.com or locally
* HUU 3.0(4a) to USB-stick with above method - silently reboot during load

#### Solution&#x20;

Upgrade from 1.5(7f) to 2.0(latest) using above method, then upgrade to latest version with KVM

#### Details

I used Ubuntu with latest script with two modifications described in vstudio link.

After upgrade CIMC reset password to default **password** (I saw another option "Cisco1234", but in my case it was "password").

```
#!/bin/bash

#syntax : sh create_util_usb.sh <device> <scu iso> <huu iso> <driver.iso>
#example: sh create_util_usb.sh /dev/sdb scu.iso huu.iso ucs-cxxx-1.4.3-driver.iso

if [ $# -lt 2 ];then
        echo "Missing argurments"
        echo "syntax: sh create_util_usb.sh <device> <huu-iso-image>"
        echo "example:"
        echo "  sh create_util_usb.sh /dev/sdb ucs-c220m4-huu-2.0.12.31.iso" 
        exit 1;
fi

USB_DEV=$1
SCU_ISO=$2

if [ "$3" == "debug" ]; then
        set -x
fi

#check for syslinux

#rpm -qa | grep syslinux 
#if [ $? -ne 0 ]; then
#       echo "Error:  syslinux not installed"
#       exit 1;
#fi

#Script input validation

if [ -z $USB_DEV ]; then
        echo "Error: USB Device not provided"
        exit 1;
fi
if [ -z $SCU_ISO ]; then
        echo "Error: SCU ISO not provided"
        exit 1;
fi

losetup -f
if [ $? -ne 0 ]; then
        echo "Error: No free loop device found. "
        exit 1
fi

#HUU ISO name validation
basename $SCU_ISO | grep -i "huu"
if [ $? -ne 0 ]; then
        echo "The second parameter does not look HUU iso. Do you want to proceed (y/n)?"
        read ans
        ans=`echo $ans | tr [:upper:] [:lower:]`
        if [ $ans = "y" ]; then
                echo "Proceeding..."
        else
                exit 1;
        fi
fi

TMP_TEST_DIR=/tmp/scu_test.$$
mkdir -p $TMP_TEST_DIR
#SCU ISO validation
losetup -f
if [ $? -ne 0 ]; then
        echo "Error: No free loop device found. "
        exit 1
fi

mount -o loop $SCU_ISO $TMP_TEST_DIR
if [ $? -ne 0 ]; then
        echo "Error: Unable to mount SCU iso for validation"
        rm -rf $TMP_TEST_DIR
        exit 1;
fi
umount $TMP_TEST_DIR


echo " "

rm -rf $TMP_TEST_DIR


TMP_PART_SC=/tmp/partscr.sh

PART1_SIZE=8192
PART1_DEV=$USB_DEV"1"
USB_DIR=/tmp/usb2.$$
#MBR=/usr/share/syslinux/mbr.bin
MBR=/usr/lib/syslinux/mbr/mbr.bin

SYSLINUX_FILE=syslinux.cfg
MENU_FILE=menu.txt
TMP_SCU=/tmp/SCU

SCU_LCD_SUM=0


create_part ()
{
        echo "## Creating partitions... Please wait (Will take a few minutes)"

        umount $PART1_DEV
        dd if=/dev/zero of=$USB_DEV bs=4096 count=10
        echo "Zeroing of USB done....."
echo "d
n
p
1

+1024M
w" | fdisk $USB_DEV

        if [ $? -ne 0 ]; then
                echo "Error: Partition creation failed."
                exit 1;
        fi

        #Mark Partition as active
echo "a
1
w
" | fdisk $USB_DEV

        echo "####### creating partition Done ######"
}

format_part () 
{
        echo "###### Formating the partitions ######"
        mkdosfs -F 32 $PART1_DEV
        if [ $? -ne 0 ]; then
                echo "Error: Formatting of first USB partition failed"
                exit 1;
        fi
        echo "##### Formating partition Done ######"
}


add_syslinux ()
{
        echo "##### add_syslinux #####"
        dd if=$MBR of=$USB_DEV
        if [ $? -ne 0 ]; then
                echo "Error: dd of mbr.bin failed ";
                #rm -rf $USB_DIR
                exit 1;
        fi
        syslinux $PART1_DEV 
        if [ $? -ne 0 ]; then
                echo "Error: syslinux failed [ $PART1_DEV ] Failed";
                #rm -rf $USB_DIR
                exit 1;
        fi

        echo "##### add_syslinux DONE  ######"
}


copy_files ()
{
        echo "##### copy_files  ######"
        mkdir -p $USB_DIR
        mount $PART1_DEV $USB_DIR
        if [ $? -ne 0 ]; then
                echo "Error: Mounting of partition [ $PART1_DEV ] on Dir [ $USB_DIR ] Failed";
                rm -rf $USB_DIR
                exit 1;
        fi


        mkdir -p $TMP_SCU

        losetup -f
        if [ $? -ne 0 ]; then
                echo "Error: No Free loop device found. Exiting";
                umount $USB_DIR
                rm -rf $USB_DIR
                rm -rf $TMP_SCU
                exit 1;

        fi

        mount -o loop $SCU_ISO $TMP_SCU
        if [ $? -ne 0 ]; then
                echo "Error: Mounting of SCU ISO [ $SCU_ISO ] on Dir [ $TMP_SCU ] Failed";
                umount $USB_DIR
                rm -rf $USB_DIR
                rm -rf $TMP_SCU
                exit 1;
        fi

        cp -rvpf $TMP_SCU/* $USB_DIR/.
        if [ $? -ne 0 ]; then
                echo "Error: Copying of SCU Files to USB failedFailed";
                umount $USB_DIR
                rm -rf $USB_DIR
                rm -rf $TMP_SCU
                exit 1;
        fi
 
        mv $USB_DIR/isolinux/ $USB_DIR/syslinux
        mv $USB_DIR/syslinux/isolinux.cfg $USB_DIR/syslinux/syslinux.cfg

        # Set the UUID of SCU SD device as part of syslinuf.cfg
        SCU_UUID=`blkid $PART1_DEV | cut -d" " -f2 | sed 's/\"//g'`
        sed -ir "s/root=live:CDLABEL=.*-huu-[0-9]*/root=live:${SCU_UUID}/g" $USB_DIR/syslinux/syslinux.cfg
                sed -ir "s/root=live:CDLABEL=.*-huu-[0-9]*/root=live:${SCU_UUID}/g" $USB_DIR/EFI/BOOT/grub.cfg

        umount $TMP_SCU
        umount $USB_DIR

        rm -rf $TMP_SCU $USB_DIR
        echo "##### copy_files Done  ######"
}

#delete_part
create_part
format_part
add_syslinux
copy_files 

fdisk -l $USB_DEV


exit 0

```

