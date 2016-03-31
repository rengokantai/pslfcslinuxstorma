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

#####creating linux file system
######ext4
```
mkfs.  tab   //show all file system
```

```
mkfs -t /dev/xvda
mkfs -t -L data /dev/xvda //assign label
```
tune: -c :count -i interval
```
tune2fs -L newlabel -c 0 -i 0 /dev/xvda
```
show info
```
dumpe2fs /dev/xvda
```

######xfs
```
mkfs.xfx -b size=1k -l (log)  size=10m /dev/xvda
```
see all com:
```
xfs_ tab
```
check info
```
xfs_db -x /dev/xvda
uuid //get
label//get
```

set label:
```
label labelname
```
mount ext4
```
mount /dev/xvda /mnt
ls /mnt
umount /mnt
mount | grep xvda
```

```
mount -o remount,noexec /dev/xvda /data
```


load all mount
```
cat /proc/mounts
```
showid
```
blkid /dev/xvda
```
change
```
vim /etc/fstab
```
edit
```
UUID=" " /data ext4 noexec 0 2     //2 filecheck   noexec/defaults
```

then reload and mount
```
mount -a
```
#####managing SWAP

######
```
mkswap /dev/xvda
swapon -s
swapon /dev/sdb5 (low priority)
```

turn off
```
swapoff /dev/sdb5
```

check by using```free -m```

###config priority
```
swapoff -a //turn off all
```

in vim console,type
```
!rblkid /dev/xvda
```
edit fstab
```
UUID="" swap swap sw,pri=5 0 0
```
######raid
```
cat /proc/mdstat
```
using mdadm  (multi device adminis)
```
yum install mdadm
mdadm --create --verbose /dev/md0 --level=mirror --raid-device=2 /dev/db12 /dev/db13
mdadm --detail --scan
mdadm --stop /dev/md0
```

#####extending permission with ACL
```
grep ACL /boot/config tab
```
y means from system, m means module

#listing acls
```
ls -A do not list . .. 
```
```
ls -l (last . show acl support)
```
get acl info
```
getfacl filename
```

######set default acls
```
setfacl -m d:o:--- test-acl/        //others cannot do anything  (u,g,o)
```
set to peculiar user
```
setfacl -dm u:bob:rw test-acl/      //user bob can rw
```
#remove acls
```
setfacl -b filaname
```
remove entry
```
setfacl -x u:bob filaname
```
######security issue
```
chage -l username
```

change selinux context
```
chcon -t admin_home_t /etc/shadow
```
