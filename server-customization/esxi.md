# ESXi

## ESXI license

{% embed url="https://customerconnect.vmware.com/group/vmware/evalcenter?p=free-esxi6" %}

## ESXI monitoring with RRD

Enable SSH on ESXi.

Enable SNMP on ESXi. (if error after running it is problem with snmp.conf on ESXi, look here: [https://kb.vmware.com/s/article/2056832](https://kb.vmware.com/s/article/2056832))

RRD template and installation guide here: [https://forums.cacti.net/viewtopic.php?f=12\&t=52122](https://forums.cacti.net/viewtopic.php?f=12\&t=52122)

## ESXI auto-shutdown at night (ESXi before 6.5)

Crontab is not installed. Use workaround:

```
/bin/echo "40 04 * * * /bin/poweroff" >> /var/spool/cron/crontabs/root
```

Time in UTC TZ.

## ESXI auto-shutdown at night (ESXi after 6.5)

Crontab is not installed. Use workaround:

1\) vi /etc/rc.local.d/local.sh

```
/bin/kill $(cat /var/run/crond.pid)
/bin/echo "40 04 * * * /bin/poweroff" >> /var/spool/cron/crontabs/root
/usr/lib/vmware/busybox/bin/busybox crond
```

Time in UTC TZ.

2\) Re-run this script

```
/etc/rc.local.d/local.sh
```

3\) Make it persistent

```
/bin/auto-backup.sh
```



## ESXi installation on Gigabyte brix

{% embed url="https://mobiletiger.jorba.de/vmware-esxi-6-0-n3150-itx-intel-celeron-braswell-platform-problem-solved" %}

It may stuck in the middle of installation process because doesn't support "interactive installation". It actually alive w/o actively output to console.

1. Create custom ESXi6.5

VMWare PowerCLI> .\ESXi-Customizer-PS-v2.4.x.ps1 -v65 -vft - load net-e1000e,sata-xahci -outDir .\output

2\. Install ESXi to USB stick using VMWare workstation

3\. Boot from USB stick and configure ESXi (IP address / gw, enable SSH)

&#x20;       Use https://www.plop.at/de/bootmanager/download.html to bootup VM from an USB

4\. Plug USB-stick to BRIX and boot-up from flash

## ESXi 6.7 installation on UCS-C240 M3

&#x20;If you see error: “can’t have a partition outside the disk”.

[https://definitelyalive.com/2017/03/18/fixing-cant-have-a-partition-outside-the-disk/](https://definitelyalive.com/2017/03/18/fixing-cant-have-a-partition-outside-the-disk/)

When deploying newer versions of ESXi on Cisco C-series servers that use Cisco’s FlexFlash SD card storage (although, admittedly this could be any server vendor) you may run into this fantastic error message that reads “Can’t have a partition outside the disk! Unable to read partition table for device.” This essentially means that the format in which the volume on the storage isn’t able to be used for an ESXi installation.

In looking into this issue we found Cisco KB CSCus51007 and countless blogs that tell you to do one of four things:

·         Install ESXi 5.5U1 Customized Image and then upgrade to the flavor of ESXi of your choice.

·         Take the SD card out, put it in a different computer, re-partition it (or make it blank) and then install ESXi.

·         Boot into GParted and re-partition the SD card.

·         Insert the SD card into a working ESXi host and use the recovery console shell to format the SD card.

I thought about this for a moment and came to a hypothesis. Is it possible to get to the recovery shell from the ESXi installer? Knowing what I know about how ESXi works, it just loads everything into memory at boot. The installer has to work the same way, right? So, when I booted up my image of 6.0 U3, I got to the window where you select your disk and wrote down the C#:T#:L# of the SD card volume and hit ALT+F1.

This was a triumph. One may even call it a huge success. The recovery console was available. A coworker of mine came over to see what I was up to and I explained what I was doing. He was as intregued as I was, as storage is his jam. I logged in as root (which in the installer, has no password set) and punched in ls. LS showed us that there was a /vmfs/ directory, just like a live version of ESXi.. In /vmfs/devices/disks/ we found our device.

The command you need to run to convert the volume is as follows: partedUtil mklabel “/dev/disks/deviceID” gpt. An example would look like this: partedUtil mklabel “/dev/disks/mpx.vmhba11:C0:T0:L0” gpt. Wait, why isn’t /vmfs/devices/ in the path name? /vmfs/devices/ is actually a symlink to /dev/. The command, from our testing, doesn’t even work when you use the symlink.

From there, hit ALT-F2 and re-scanned the storage for the installer. ESXi 6.0 U3 installed without issue. I hope this helps some admins in the future, as the resources out there on this problem aren’t great! A TL;DR is below with steps. Enjoy!

&#x20;

TL;DR: just give me the fix!

1. Boot into ESXi’s installer
2. Get to the disk selection screen and take note of the disk identifier
3. Hit ALT+F1 (login root / pass ())
4. Navigate to /dev/disks/
5. Use ls to find your disk identifier
6. Use the following command to convert the disk into GPT: partedUtil mklabel “/dev/disks/deviceID” gpt
7. Install ESXi

## ESXi partitioning

{% embed url="http://www.vmwarearena.com/what-i-wish-everyone-knew-about-esxi-partition" %}

## Create VMFS on SD card (opt 1)

{% hint style="warning" %}
If there is a separate partition on SD card that represents to ESXi as an independent disk. For example: FlexFlash split into 4 partitions in CIMC. Each partition represents to ESXI as separate disk: mpx.vmhba33 / 34 / 35 /36
{% endhint %}

* In this example ESXi installed on mpx.vmhba33
* New partition will be created on mpx.vmhba34

Check available disks in ESXi and try to match it against CIMC FlexFlash partitions

```
[root@localhost:~] ls -lah /dev/disks
total 4114379440
drwxr-xr-x    2 root     root         512 Dec 23 21:01 .
drwxr-xr-x   13 root     root         512 Dec 23 21:01 ..
-rw-------    1 root     root        3.0G Dec 23 21:01 mpx.vmhba33:C0:T0:L0
-rw-------    1 root     root        4.0M Dec 23 21:01 mpx.vmhba33:C0:T0:L0:1
-rw-------    1 root     root      250.0M Dec 23 21:01 mpx.vmhba33:C0:T0:L0:5
-rw-------    1 root     root      250.0M Dec 23 21:01 mpx.vmhba33:C0:T0:L0:6
-rw-------    1 root     root      110.0M Dec 23 21:01 mpx.vmhba33:C0:T0:L0:7
-rw-------    1 root     root      286.0M Dec 23 21:01 mpx.vmhba33:C0:T0:L0:8
-rw-------    1 root     root        6.5G Dec 23 21:01 mpx.vmhba33:C0:T0:L1
-rw-------    1 root     root        6.0G Dec 23 21:01 mpx.vmhba33:C0:T0:L1:1
-rw-------    1 root     root        1.3G Dec 23 21:01 mpx.vmhba33:C0:T0:L2
-rw-------    1 root     root        7.6G Dec 23 21:01 mpx.vmhba33:C0:T0:L2:1
```

Lines (5) - (10) is ESXi disks, refer to ESXi partitioning for details

Lines (11) - (12) and (13) - (14) additional partitions exposed by FlexFlash controller.

![](<../.gitbook/assets/image (4).png>)

By turning on and off (enabling/disabling) those partitions and rebooting ESXi you'll get to know which one is which.&#x20;

Identify what partition will be converted to Datastore then check that partition table can be retrieved. If not, chose another partition.

Keep below numbers for future reference to avoid calculations:

```
[root@localhost:~] partedUtil getptbl /dev/disks/mpx.vmhba33\:C0\:T0\:L1
msdos
848 255 63 13631488
1 63 12578894 11 0
```

&#x20;This output means that vmhba33\_\_\_\_L1 is primary partition

Line (2) tells msdos, it means that partition table sits there

Line (4) shows extended partition size , the one that we intended to change

* 1 is partition id
* 63 is start sector
* 12578894 is end sector
* 11 is partition type refer here for valid partition types ([https://kb.vmware.com/s/article/1036609](https://kb.vmware.com/s/article/1036609)) or use `partedUtil showGuids`
* 0 is it bootable

Label primary partition as `gpt`

```
partedUtil setptbl /dev/disks/mpx.vmhba33\:C0\:T0\:L1 gpt
```

Check

```
[root@localhost:~] partedUtil getptbl /dev/disks/mpx.vmhba33\:C0\:T0\:L1 
gpt
848 255 63 13631488
```

Note that partition :1 disappears

Recreate partition with the same size but different ID

```
partedUtil setptbl /dev/disks/mpx.vmhba33\:C0\:T0\:L1 gpt "1 63 12578894 AA31E02A400F11DB9590000C2911D1B8 0"
```

* 1 is partition id
* 63 is start sector
* 12578894 is end sector
* AA31E02A400F11DB9590000C2911D1B8 is partition type refer here for valid partition types ([https://kb.vmware.com/s/article/1036609](https://kb.vmware.com/s/article/1036609)) or use `partedUtil showGuids`
* 0 is it bootable

After that step new partition will appear :1 . Make VMFS5 file system in new partition

```
vmkfstools -C vmfs5 -S SD-Datastore /dev/disks/mpx.vmhba33\:C0\:T0\:L1:1
```

Check devices:

```
[root@localhost:~] ls -lah /dev/disks
total 4114379440
drwxr-xr-x    2 root     root         512 Dec 23 21:01 .
drwxr-xr-x   13 root     root         512 Dec 23 21:01 ..
-rw-------    1 root     root        3.0G Dec 23 21:01 mpx.vmhba33:C0:T0:L0
-rw-------    1 root     root        4.0M Dec 23 21:01 mpx.vmhba33:C0:T0:L0:1
-rw-------    1 root     root      250.0M Dec 23 21:01 mpx.vmhba33:C0:T0:L0:5
-rw-------    1 root     root      250.0M Dec 23 21:01 mpx.vmhba33:C0:T0:L0:6
-rw-------    1 root     root      110.0M Dec 23 21:01 mpx.vmhba33:C0:T0:L0:7
-rw-------    1 root     root      286.0M Dec 23 21:01 mpx.vmhba33:C0:T0:L0:8
-rw-------    1 root     root        6.5G Dec 23 21:01 mpx.vmhba33:C0:T0:L1
-rw-------    1 root     root        6.0G Dec 23 21:01 mpx.vmhba33:C0:T0:L1:1
-rw-------    1 root     root        1.3G Dec 23 21:01 mpx.vmhba33:C0:T0:L2
-rw-------    1 root     root        7.6G Dec 23 21:01 mpx.vmhba33:C0:T0:L2:1
```

Line (12) shows new partition again

Check Datastores

```
[root@localhost:~] ls /vmfs/volumes/ -la
total 2820
drwxr-xr-x    1 root     root           512 Dec 23 21:04 .
drwxr-xr-x    1 root     root           512 Dec 23 20:52 ..
drwxr-xr-x    1 root     root             8 Jan  1  1970 00e0e4e6-8a1caab3-1f13-2923665b65f5
drwxr-xr-x    1 root     root             8 Jan  1  1970 5ae37990-5a1b12a8-2e59-6c416a1ecb66
drwxr-xr-t    1 root     root          2240 Dec 23 17:06 5ae37d98-00f6a405-5382-6c416a1ecb66
drwxr-xr-x    1 root     root             8 Jan  1  1970 619be0eb-c8f91c6b-d441-ee2cfc487126
drwxr-xr-t    1 root     root          1260 Dec 23 21:01 61c4e3c2-e04ca8ec-b12c-6c416a1ecb66
lrwxr-xr-x    1 root     root            35 Dec 23 21:04 SD-Datastore -> 61c4e3c2-e04ca8ec-b12c-6c416a1ecb66
lrwxr-xr-x    1 root     root            35 Dec 23 21:04 datastore1 -> 5ae37d98-00f6a405-5382-6c416a1ecb66
```

Line (10) shows new datastore. Now datastore will appear in GUI.

## Create VMFS on SD card (opt 2)

{% hint style="warning" %}
If SD-card is a single partition and represented to ESXi as a single disk.&#x20;
{% endhint %}

ESXI will partition that disk into multiple partitions and will consume around 4 GB for all partitions all leftovers will stay intact. We want to slice some amount of unpartitioned space for another VMFS disk.&#x20;

ESXi doesn't allow to change partition table of the disk where it is installed, therefore we have to load some OS that have partedUtil. I'm using ESXi installer. When you are on a step accepting EULA you can press Alt+F1 and get shell with "root/" credentials. From there do some math described on the below picture and add one more partition.&#x20;

![](<../.gitbook/assets/image (11).png>)

After that create normal filesystem in that partition as described in previous example.

## Move ESXi scratch location

{% embed url="https://kb.vmware.com/s/article/1033696" %}

## Determining Network/Storage firmware/driver version

When checking compatibility for devices make sure you refer to VMware HCL , not to vendor recommendations (in our case Cisco)

{% embed url="https://kb.vmware.com/s/article/1027206" %}

Once hba/nic card model identified , go to HCL: [https://www.vmware.com/resources/compatibility/search.php?deviceCategory=io](https://www.vmware.com/resources/compatibility/search.php?deviceCategory=io)

In my case HCL recommended previous version of drivers, while Cisco bundle came with latest version. Sometime drivers MUST be downgraded.

![](<../.gitbook/assets/image (17).png>)

## Activate storage controller driver

{% embed url="https://kb.vmware.com/s/article/58844?lang=en_US&queryTerm=lsi_mr3+version+7.708.07.00-3vmw" %}

{% hint style="warning" %}
Be careful , you may activate legacy driver
{% endhint %}

