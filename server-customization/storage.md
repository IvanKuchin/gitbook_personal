# Storage

## Cheatsheet

Information about disk mapper (devices like: dm-0, dm-1, dm-2, ...)

```
sudo dmsetup ls
sudo dmsetup info /dev/dm-0
```

Information about LVM

```
sudo pvs
sudo vgs
sudo lvs
```

Example of `lvs`

```
$ sudo lvs
  LV      VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root-lv ubuntu-vg -wi-ao----  64.00g                                                    
  snap-lv ubuntu-vg -wi-ao---- 256.00g                                                    
  var-lv  ubuntu-vg -wi-ao----  16.00g   
```

If Attr-column contains "s" (for ex: swi-ao) means snapshot fs. Which means temporary fs and write errors are possible to it.

Hierarchy of file systems

```
lsblk
```

```
NAME                MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0                   2:0    1    4K  0 disk 
sda                   8:0    0   64G  0 disk 
├─sda1                8:1    0  487M  0 part /boot
├─sda2                8:2    0    1K  0 part 
└─sda5                8:5    0 63.5G  0 part 
  ├─www--vg-root    252:0    0 59.5G  0 lvm  /
  └─www--vg-swap_1  252:1    0    4G  0 lvm  [SWAP]
sdb                   8:16   0  920G  0 disk 
└─backup--vg-backup 252:2    0  920G  0 lvm  /storage/ikuchin
sr0                  11:0    1 1024M  0 rom  
```

## CIMC RAID configuration

![](../.gitbook/assets/server\_raid.jpg)



## How to fix bad block on WD My cloud

Full description could be found [here](https://www.smartmontools.org/wiki/BadBlockHowto#footnote6)

Symptoms: super slow response from WebGUI on any operation.

How to fix:

1. enable SSH
2. SSH to disk shell

```
smartctl -a /dev/sda
```

> ID# ATTRIBUTE\_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN\_FAILED RAW\_VALUE
>
> &#x20; 5 Reallocated\_Sector\_Ct   0x0033   100   100   005    Pre-fail  Always       -       0
>
> 196 Reallocated\_Event\_Count 0x0032   100   100   000    Old\_age   Always       -       0
>
> 197 <mark style="background-color:yellow;">Current\_Pending\_Sector</mark>  0x0022   100   100   000    Old\_age   Always       -      <mark style="background-color:yellow;">1</mark>
>
> 198 Offline\_Uncorrectable   0x0008   100   100   000    Old\_age   Offline      -       1

If current pending sectors higher than 0, then there are sectors that need to be fixed.

To find broken sectors either:

```
smartctl -l selftest /dev/hda:
```

> SMART Self-test log structure revision number 1
>
> Num  Test\_Description    Status                  Remaining  LifeTime(hours)  LBA\_of\_first\_error
>
> \# 1  Extended offline    Completed: read failure       90%       217         <mark style="background-color:yellow;">0x016561e</mark>9

Or

> Error 51790 \[13] occurred at disk power-on lifetime: <mark style="background-color:yellow;">9395</mark> hours (391 days + 11 hours)
>
> &#x20; When the command that caused the error occurred, the device was doing SMART Offline or Self-test.
>
> &#x20;
>
> &#x20; After command completion occurred, registers were:
>
> &#x20; ER -- ST COUNT  LBA\_48  LH LM LL DV DC
>
> &#x20; \-- -- -- == -- == == == -- -- -- -- --
>
> &#x20; 40 -- 51 00 80 00 00 06 f9 c1 80 e6 00  Error: UNC 128 sectors at <mark style="background-color:yellow;">LBA = 0x06f9c180 = 117031296</mark>
>
> &#x20;
>
> &#x20; Commands leading to the command that caused the error were:
>
> &#x20; CR FEATR COUNT  LBA\_48  LH LM LL DV DC  Powered\_Up\_Time  Command/Feature\_Name
>
> &#x20; \-- == -- == -- == == == -- -- -- -- --  ---------------  --------------------
>
> &#x20; c8 00 00 00 80 00 00 06 f9 c1 80 e6 08     01:36:05.380  READ DMA
>
> &#x20; 27 00 00 00 00 00 00 00 00 00 00 e0 08     01:36:05.372  READ NATIVE MAX ADDRESS EXT
>
> &#x20; ec 00 00 00 00 00 00 00 00 00 00 a0 08     01:36:05.372  IDENTIFY DEVICE
>
> &#x20; ef 00 03 00 46 00 00 00 00 00 00 a0 08     01:36:05.372  SET FEATURES \[Set transfer mode]
>
> &#x20; 27 00 00 00 00 00 00 00 00 00 00 e0 08     01:36:05.372  READ NATIVE MAX ADDRESS EXT

Match 9395 hours against Power\_On\_Hour to identify how long-ago error did appear.

> SMART Attributes Data Structure revision number: 16
>
> Vendor Specific SMART Attributes with Thresholds:
>
> ID# ATTRIBUTE\_NAME          FLAGS    VALUE WORST THRESH FAIL RAW\_VALUE
>
> &#x20; 1 Raw\_Read\_Error\_Rate     POSR-K   131   131   051    -    22522
>
> &#x20; 3 Spin\_Up\_Time            POS--K   233   196   021    -    7316
>
> &#x20; 4 Start\_Stop\_Count        -O--CK   084   084   000    -    16834
>
> &#x20; 5 Reallocated\_Sector\_Ct   PO--CK   200   200   140    -    0
>
> &#x20; 7 Seek\_Error\_Rate         -OSR-K   200   200   000    -    0
>
> &#x20; 9 <mark style="background-color:yellow;">Power\_On\_Hours          -O--CK   088   088   000    -    9395</mark>
>
> &#x20;10 Spin\_Retry\_Count        -O--CK   100   100   000    -    0
>
> &#x20;11 Calibration\_Retry\_Count -O--CK   100   253   000    -    0
>
> &#x20;12 Power\_Cycle\_Count       -O--CK   100   100   000    -    29
>
> 192 Power-Off\_Retract\_Count -O--CK   200   200   000    -    9
>
> 193 Load\_Cycle\_Count        -O--CK   195   195   000    -    17279
>
> 194 Temperature\_Celsius     -O---K   112   098   000    -    40
>
> 196 Reallocated\_Event\_Count -O--CK   200   200   000    -    0
>
> 197 Current\_Pending\_Sector  -O--CK   200   200   000    -    22
>
> 198 Offline\_Uncorrectable   ----CK   100   253   000    -    0
>
> 199 UDMA\_CRC\_Error\_Count    -O--CK   200   200   000    -    0
>
> 200 Multi\_Zone\_Error\_Rate   ---R--   100   253   000    -    0
>
> &#x20;                           ||||||\_ K auto-keep
>
> &#x20;                           |||||\_\_ C event count
>
> &#x20;                           ||||\_\_\_ R error rate
>
> &#x20;                           |||\_\_\_\_ S speed/performance
>
> &#x20;                           ||\_\_\_\_\_ O updated online
>
> &#x20;                           |\_\_\_\_\_\_ P prefailure warning

Using `hdparm –read-sector` identify sectors that are failed. For example:

```
hdparm –-read-sector 117031296 /dev/sda
```

To fix the sector write all zeroes in it

```
hdparm --yes-i-know-what-i-am-doing –-write-sector 117031296 /dev/sda
```



Script below is a small automation to find and recover bad sectors

```shell
export i=117027504
while [ $i -le 117029504 ]
do echo $i
output=$(hdparm --read-sector $i /dev/sda)
result=$?
if [ $result -eq 0 ]; then
echo ok
else
echo got to be fixed
hdparm --yes-i-know-what-i-am-doing --write-sector $i /dev/sda
fi
let i+=1
done
```

## Clean-up used space

Useful post can be found [here](https://serverfault.com/questions/315181/df-says-disk-is-full-but-it-is-not)

> ikuchin@dev:\~$ df -h
>
> Filesystem                   Size  Used Avail Use% Mounted on
>
> udev                         7.9G     0  7.9G   0% /dev
>
> tmpfs                        1.6G   26M  1.6G   2% /run
>
> <mark style="background-color:yellow;">/dev/mapper/ubuntu--vg-root   38G   36G    0G 100% /</mark>
>
> tmpfs                        7.9G     0  7.9G   0% /dev/shm
>
> tmpfs                        5.0M     0  5.0M   0% /run/lock
>
> tmpfs                        7.9G     0  7.9G   0% /sys/fs/cgroup
>
> /dev/sda1                    236M  155M   69M  70% /boot
>
> cgmfs                        100K     0  100K   0% /run/cgmanager/fs
>
> tmpfs                        1.6G     0  1.6G   0% /run/user/1000
>
> 192.168.168.32:/nfs/ikuchin  3.6T  1.2T  2.5T  33% /storage/ikuchin
>
> //192.168.168.46/ikuchin     4.6T  799G  3.8T  18% /storage2/ikuchin

{% hint style="warning" %}
Root cause: CIFS mount /storage2/xxxxx has hide actual directory that eat all space. To fix it: unmount and remove everything there.
{% endhint %}

Another avenue:

Check open files that are hold by running processes.

```
lsof +L1 (which is equal to lsof | grep deleted)
```

Expected output:

```
COMMAND     PID     USER   FD   TYPE DEVICE SIZE/OFF NLINK    NODE NAME
mysqld     1208    mysql    4u   REG  252,0        0     0 1572867 /tmp/ib6WVq6B (deleted)
mysqld     1208    mysql    5u   REG  252,0      441     0 1572870 /tmp/ibPY6w0i (deleted)
mysqld     1208    mysql    6u   REG  252,0        0     0 1572871 /tmp/ibxVIDUZ (deleted)
mysqld     1208    mysql    7u   REG  252,0        0     0 1572873 /tmp/ibo7QUQn (deleted)
mysqld     1208    mysql   11u   REG  252,0        0     0 1573498 /tmp/ibM7uSY5 (deleted)
dovecot    1328     root  124u   REG   0,18        0     0     678 /run/dovecot/login-master-notify4a62ef1ffba130b5 (deleted)
dovecot    1328     root  141u   REG   0,18        0     0     679 /run/dovecot/login-master-notify6bd4c1b8cd3efa38 (deleted)
dovecot    1328     root  144u   REG   0,18        0     0     680 /run/dovecot/login-master-notify1605a39481d441bb (deleted)
apache2    1557 www-data   38u   REG  252,0        0     0 1573790 /tmp/.ZendSem.PnoKte (deleted)
apache2    1562 www-data   38u   REG  252,0        0     0 1573790 /tmp/.ZendSem.PnoKte (deleted)
apache2    3286 www-data   38u   REG  252,0        0     0 1573790 /tmp/.ZendSem.PnoKte (deleted)
... output cut ...
```

{% hint style="warning" %}
Those files are not always an issue. Services can remove files that they opened.
{% endhint %}

Actual PID is the second column, but it might be sub-process of another parent process (for example docker service). To see the process tree and identify the process use:

```
sudo systemctl status
```

```
    State: degraded
     Jobs: 0 queued
   Failed: 1 units
    Since: Wed 2021-05-26 10:25:01 MSK; 5 months 10 days ago
   CGroup: /
           ├─init.scope
           │ └─1 /sbin/init
           ├─system.slice
           │ ├─mdadm.service
           │ │ └─1103 /sbin/mdadm --monitor --pid-file /run/mdadm/monitor.pid --daemonise --scan --syslog
           │ ├─dbus.service
           │ │ └─978 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
           │ ├─cron.service
           │ │ └─24563 /usr/sbin/cron -f
           │ ├─lvm2-lvmetad.service
           │ │ └─467 /sbin/lvmetad -f
           │ ├─vgauth.service
           │ │ └─999 /usr/bin/VGAuthService
           │ ├─nagios-nrpe-server.service
           │ │ └─1319 /usr/sbin/nrpe -c /etc/nagios/nrpe.cfg -d
           │ ├─dovecot.service
           │ │ ├─1328 /usr/sbin/dovecot
... output cut ...
```

## LVM

![LVM architecture](<../.gitbook/assets/image (10).png>)

```
fdisk /dev/sdb
create ext partition (press n)
change type to Linux LVM 8e (press t)
write (press w)
```



```
pvcreate /dev/sdb1
vgcreate backup-vg /dev/sdb1
lvcreate –L 67G –n backup backup-vg
mkfs.ext4 /dev/backup-vg/backup
```

To mount it after reboot ad below snippet to /etc/fstab

```
/dev/mapper/backup--vg-backup   /storage/ikuchin/       ext4    errors=remount-ro 0       0
```

## ESXi CLI

List of disks and partitions:

```
ls -lah /dev/disks/
```

HW RAID-controller / firmware / driver

```
esxcli storage san sas list
```

RAID type , number of disks and /dev/disks/UUID

```
esxcli storage core device list
```

Mapping /dev/disks/UUID to hba and controller name:&#x20;

```
esxcfg-scsidevs -l
```
