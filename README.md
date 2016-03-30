#### pslinuxstorma
#####partition disks
create partitions  
MBR-2TB GUID-8ZB / logical partitions, paimary 4, max 128 partitions
```
fdisk, gdisk parted
```

######fdisk
```
fdisk -l /dev/xvda
```
delete partition
```
dd id=/dev/zaro of=/dev/xvda count=1 bs=1024 //Or omit count parameter
```

press w to save and exit
######parted
```
parted
select /dev/xvdab
mklabel labelname
```
partition. from 1mb to 200mb(size=199mb)
```
mkpart primary 1 200
```
use remaining space
```
mkpart extended 201 -1
mkpart logical 202 300
p
quit
```
######scripting
```
#! /bin/bash
DISK="/dev/xvda"
parted -s $DISK -- mklabel name mkpart extended 1m -1m
#create a swap partition
parted -s $DISK mkpart logical swapname 2m 100m
#create logic and LVM
parted -s $DISK mkpart logical 101m 200m
parted -s $DISK set 10 lvm on
parted -s $DISK mkpart logical 901m 1000m
parted -s $DISK print
```
######summary
83 linux-82swap-8e lvm-fd raid  
GPT label:
8300 linux-8200 swap
