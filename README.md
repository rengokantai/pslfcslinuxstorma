#### pslfcslinuxstor
#####partitioning
######fdisk
basic
```
lsblk
fdisk -l /dev/xvdf
```
fdisk
```
fdisk /dev/xvdf
n->p->enter->enter->+20M
```
change partition type: t HExcode82=>swap partition, L=list all type w=write d=delete
```
dd if=/dev/zero of=/dev/xvdf1 count=1 bs=512
```
######gdisk
```
gdisk /dev/xvdf
```
?=help, then create 2 partition:
```
n->enter->enter->+20M->8300
n->enter->enter->+20M->8200
w->Y
```
wipe out GPT header
```
dd if=/dev/zero of=/dev/xvdf count=2 bs=16K
```
######parted
print info
```
parted /dev/xvdf print
```
begin
```
parted
select /dev/xvda
mklabel msdos
mklabel gpt(destroy table)
p(print)
mkpart primary 1 200 (mega)
mkpart extended 201 300  (or 201 -1)
mkpart logical 202 300
quit
```
destroy:
```
dd if=/dev/zero of=/dev/xvdf1 count=1 bs=512
```
using parted with script like:
```
#!/bin/bash
disk="/dev/xvdf"
parted -s $disk -- mklabel msdos mkpart extended 1m -1m
parted -s $disk mkpart logical linux-swap 2m 100m #5
parted -s $disk mkpart logical 101m 200m #6
parted -s $disk mkpart logical 201m 300m 
parted -s $disk mkpart logical 301m 400m 
parted -s $disk mkpart logical 401m 500m #9
parted -s $disk mkpart logical 501m 600m 
parted -s $disk mkpart logical 601m 700m 
parted -s $disk mkpart logical 701m 800m #12

parted -s $disk set 10 lvm on
parted -s $disk set 11 lvm on
parted -s $disk set 12 lvm on
parted -s $disk mkpart logical 801m 900m 
parted -s $disk mkpart logical 901m 1000m 
parted -s $disk set 13 raid on
parted -s $disk set 14 raid on
parted -s $disk print
```

######summary
83 linux-82swap-8e lvm-fd raid  
GPT label:
8300 linux-8200 swap

#####creating linux file system
######ext4
```
fdisk -l /dev/xvda
```

```
mkfs.  tab   //show all file system
```

```
mkfs.ext4 -t /dev/xvdf1
mkfs -t ext4 /dev/xvdf1
```
assign label
```
mkfs.ext4 -t -L ke /dev/xvdf1 //assign label
```
tune: -c :count -i interval
```
tune2fs -L newlabel -c 0 -i 0 /dev/xvdf1
```
show info
```
dumpe2fs /dev/xvdf1
```

######xfs
```
mkfs.xfs -b size=1k -l (log)  size=10m /dev/xvdf1
```
see all com:
```
xfs_ tab
```
check info
```
xfs_db -x /dev/xvdf7
uuid //get
label//get
```

set label:
```
label labelname
```
mount ext4
```
mount /dev/xvdf1 /mnt
ls /mnt
umount /mnt
mkdir -p /data/test
mount /dev/xvdf1 /data/test
mount | grep test
mount -o remount,noexec /dev/xvdf1 /data/test
umount /data/test
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
UUID=" " /data ext4 noexec 0 2     //2 filecheck   noexec/defaults   nobackup, filesystemcheck
```

then reload and mount
```
mount -a
```
######Using mount command and XFS file system
```
vim /etc/fstab
```
```
UUID=" " /data ext4 defaults 0 0
```
xfs_info
```
xfs_info /dev/xvdf2
```
#####managing SWAP

######
```
mkswap /dev/xvdf4
swapon -s  //summary
swapon /dev/xvdf4 (low priority)
```

turn off (remove swap space)
```
swapoff /dev/xvdf4
```

check by using```free -m```

######config priority
```
swapoff -a //turn off all swap space
```

in vim console,type
```
!rblkid /dev/xvda
```
edit fstab
```
UUID="" swap swap sw,pri=5 0 0
```
turn on all
```
swapon -a
```
######raid
raid levels:  
linear: partitions/disk not the same size, volume is expanded across all disks in array,spare disk not supported
raid0: similar to above but disks or partitions are of same size  
raid1: mirror data between devices of similar size
```
cat /proc/mdstat
```
using mdadm  (multi device adminis)
```
yum install mdadm
```
create 2 raid partitions using parted,
```
mdadm --create --verbose /dev/md0 --level=mirror --raid-device=2 /dev/xvdf5 /dev/xvdf6
```
```
mkfs.xfs /dev/md0
mdadm --detail --scan
mdadm --stop /dev/md0
mdadm --assamble --scan
```

#####extending permission with ACL
######ACL support within the kernel
support xfs md4
```
grep ACL /boot/config tab
grep ACL /boot/config-$(uname -r)
```
y means from system, m means module  

######listing acls
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
acl will override `chmod`
```
setfacl -m d:o:--- test-acl/        //d=default:others cannot do anything  (u,g,o)
```
set to peculiar user
```
setfacl -dm u:bob:rw test-acl/      //user bob can rw
```
######remove acls
remove all acl entry
```
setfacl -b filaname
```
remove single acl entry
```
setfacl -x u:bob filaname
```
######security issue
```
chage -l username
```

change selinux context
```
chcon -t admin_home_t /etc/shadow  //enhance security
```
then
```
chage -l username  //permission denied
```
```
ls -Z /etc/shadow
```
check recent
```
ausearch -m AVC -ts recent   //-ts = timestamp
```
restore /etc/shadow
```
restorecon /etc/shadow
```
test apache syntax
```
apachectl configtest
```
semanage
```
semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
restorecon /web
restorecon /web/*     //= restorecon -R /web
```

#####managing logical valumes
######LVMs
using pvscan
```
yum install -y lvm2
pvscan  //scan all disks from physical volumes
vgscan //  scan all disks for volume groups and rebuild caches
lvscan //scan all disks from logical volumes
```

create phy volume
```
pvcreate /dev/xvdf10
pvcreate /dev/xvdf11
pvcreate /dev/xvdf12
```

create volume group
```
vgcreate vg1 /dev/xvdf10 /dev/xvdf11
```
get more info
```
vgs
```

create logical volume
```
lvcreate -n <name e.x lv1> -L <size> <gpname e.x. vg1>
lvcreate -n lv1 -L 184m vg1
mkfs.xfs /dev/vg1/lv1
```
scan
```
lvscan
```

######resize logical volume
extend vg1, add another volume
```
vgextend vg1 /dev/sdb12
```
in home folder,
```
mkdir /lvm
```
then, in fstab
```
/dev/vg1/lv1 /lvm xfs defaults 0 0
```
test
```
mount -a
mount
```
copy test
```
find /usr/share/doc -name '*.pdf' -exec cp {} /lvm \;
ls /lvm
```
######resizing logical volume
extend volume group, add another physical volume
```
vgextend vg1 /dev/xvdf12
```
extend logical volume
```
lvextend -L +50m /dev/vg1/lv1
xfs_growfs /lvm
df -h /lvm
```

######lvm snapshots
```
lvcreate -L 30m -s -n backup /dev/vg1/lv1
```
ro = readonly
```
mount /dev/vg1/backup /mnt -o nouuid,ro
```
restore:
```
tar -cf /root/backup.tar /mnt
umount /mnt
lvremove /dev/vg1/backup
```
######migrate pv to new storage
connect another hard drive(sdc), then
```
fdisk /dev/sdc->n->e->enter enter
n->l->enter->+300M->t->5->8e->w
```
```
lsblk
pvcreate /dev/sdc5
vgextend vgl /dev/sdc5
pvmove (-b background) /dev/xvdf10 /dev/sdc5
```
remove:
```
vgdisplay /dev/xvdf10
vgreduce vg1 /dev/xvdf10   //remove /dev/xvdf10 from vg1
pvremove /dev/xvdf10
umount /lvm
mount -a
```

#####config iscsi block storage server
######Install iSCSI Target and Configure Firewall
server1
```
yum install targetd targetcli
systemctl enable targetd
yum install firewalld
firewall-cmd --add-service=iscsi-target --permanent
firewall-cmd --reload
firewall-cmd --list-services
```
######create lvm on share
```
lvreduce -L -100m /dev/vg1/lv1
vgs
lvcreate -L 100m -n web_lv vg1
lvscan
```
######configure iscsi target
```
targetcli
>ls
> backstores/block create web_store /dev/vg1/web_lv

```

create in iscsi
```
iscsi/ create iqn.2016-06.com.ke.server1:web
cd iscsi/iqn.2016-06.com.ke.server1:web/tpg1/
luns/ create /backstores/block/web_store
acls/ create iqn.2016-06.com.ke.server2:web   //set client
exit
netstat -ltn  //port 3260
```

######configure iscsi initiator
start another machine server2
```
yum list available |grep iscsi
yum install iscsi-initiator-utils
vim /etc/iscsi/initiatorname.iscsi
```
edit
```
InitiatorName=iqn.2016-06.com.ke.server2:web
```

```
iscsiadm --mode discovery --type sendtargets --portal (server1ip) --discover //put ip address
```
connect
```
iscsiadm --mode node --targetname iqn.2016-06.com.ke.server1:web --portal (server1ip) --login
```
test in server2:
```
lsblk
```
#####HA Clusters
######pacemaker
create two servers, do these on both servers
```
yum install pacemaker pcs resource-agents
```
```
echo 'hacluster:pass' |chpasswd
```
```
firewall-cmd --permanent --add-service=high-availability
firewall-cmd --reload
```
######creating cluster
do on both machine
```
systemctl enable pcsd && systemctl start pcsd
```
on machine 1:
```
pcs cluster auth 172.31.117.145 172.31.16.122 -u hacluster -p pass
pcs cluster setup --name peanut 172.31.117.145 172.31.16.122
pcs cluster start --all
pcs status
```
do on both
```
systemctl enable corosync pacemaker
pcs status
```
######understand stonith quorum
shoot the other node in the head
on machine 1
```
pcs property set stonith-enabled=false
pcs property set no-quorum-policy=ignore
pcs status
less /etc/corosync/corosync.conf
```
######clustering ip
machine1
```
pcs config
pcs resource create ip ocf:heartbeat:IPaddr2 ip=172.31.127.100 cidr_netmask=24 op monitor interval=20s //delete:pcs resource delete ip
pcs status
ip a s
ping 172.31.127.100
```
machine2
```
pcs cluster standby 172.31.117.145
pcs cluster unstandby 172.31.117.145
```

######install apache
both machine
```
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```

```
vi /etc/httpd/conf/httpd.conf
```
edit
```
DocumentRoot "/var/www/html"
```
Add plugin conf:
```
vi /etc/httpd/conf.d/status.conf
```
edit
```
<Location /server-status>
SetHandler server-status
Require ip 127.0.0.1
</Location>
```

######clustering apache
machine1
```
pcs resource create web-server ocf:heartbeat:apache configfile=/etc/httpd/conf/httpd.conf statusurl="http://127.0.0.1/server-status" op monitor interval=20s
pcs constraint colocation add web-server cluster_ip INFINITY6
w3m 172.31.127.100
```
machine2
```
pcs cluster standby server1.ex.com  //traffic will go server2
pcs cluster unstandby server1.ex.com    //back to server1
```
#####glusterfs
(I created another 2 servers)
do on both machine:
```
parted /dev/xvdf -- mklabel msdos mkpart primary 1m -1m
mkfs.xfs /dev/xvdf1
mkdir /gfs
vi /etc/fstab
```
edit  (5dw W d$)
```
UUID= .... /gfs xfs defaults 0 0 
```
######install glusterfs and firewall
both machine:
```
yum install epel-release -y
cd /etc/yum.repos.d/
yum install wget
wget -P /etc/yum.repos.d http://download.gluster.org/pub/gluster/glusterfs/LATEST/RHEL/glusterfs-epel.repo
scp glusterfs-epel.repo user@172.31.101.252:/tmp/
ls /usr/lib/firewalld/services
```
enable machine1
```
yum install glusterfs-server glusterfs glusterfs-fuse
```
machine2
```
yum install glusterfs-server
```
enable on both machine
```
yum install -y firewalld && systemctl start firewalld && firewall-cmd --permanent --add-service=glusterfs && firewall-cmd --reload && systemctl start glusterd.service
```
#####Implementing Distributed Volumes
both machine
```
mkdir /gfs/vol
```
machine1
```
gluster peer probe 172.31.101.252 ( server 2 ip)
```
machine2
```
gluster peer status
```
machine1
```
gluster volume create volume_dist transport tcp 172.31.114.21:/gfs/vol 172.31.101.252:/gfs/vol 
gluster volume start volume_dist
gluster volume info
mount -t glusterfs 172.31.114.21:/volume_dist /mnt
touch /mnt/file{1..100}  //each server will get 50 files
```
test in server1 and server2 respectively:
```
ls /gfs/vol
```

######create replicated volume
1
```
umount /mnt
```
1 and 2
```
mkdir /gfs/rep
```
1
```
gluster volume create volume_replicated replica 2 172.31.114.21:/gfs/rep 172.31.101.252:/gfs/rep
gluster volume start volume_replicated
mount -t glusterfs 172.31.114.21:/volume_replicated /mnt
touch /mnt/file1
```
test in server1 and server2 respectively:
```
ls /gfs/rep
```
#####encrypted volume
######shredding disk
```
vgs
lvcreate -L 80m -n name vg1 
shred -v --iterations=1 /dev/vg1/name
```

######encrypting (LUKS support is by way of the dm_crypt kernel module)
```
grep -i ACL /boot/config-$(uname -r)
grep -i DM_CRYPT /boot/config-$(uname -r)   //m=maybe
lsmod | grep dm_crypt
modprobe dm_crypt   //enable this module
yum install cryptsetup
rpm -qf $(which cryptsetup)
cryptsetup -y luksFormat /dev/vg1/name
cryptsetup luksDump /dev/vg1/name
cryptsetup isLuks /dev/vg1/name
echo $? 0=t 1=f
```


######opening encrypted and formatting
```
cryptsetup luksOpen /dev/vg1/name name_vol
ls /dev/mapper
\ls /dev/mapper
mkfs.xfs /dev/mapper/name_vol
```
######Mounting at Boot
```
cryptsetup luksClose name_vol
cryptsetup luksOpen /dev/vg1/name name_vol
\ls /dev/mapper, copy uuid from /dev/mapper/vg1-name
\ls   //not alias
```
then
```
vi /etc/crypttab
```
edit
```
luks-data UUID="" 
```
then
```
vi /etc/fstab
```
edit
```
/dev/mapper/luks-data /luks-data xfs defaults 0 0
```
```
mkdir /luks-data
(ctrl r to search previous commands)
cryptsetup luksOpen /dev/vg1/name luks_data
mount -a
mount
```
#####Auto mounter
######using default autofs option
```
yum list installed
yum list installed | grep autofs
yum list available | grep autofs
yum intall autofs -y
ls /etc/auto*
vim /etc/autofs.conf
systemctl start autofs
ls /misc
cd cd
ls
```
######auto mounting enc partition
```
umount /luks-data
```
and edit /etc/fstab
```
/dev/mapper/luks-data /luks-data xfs defaults 0 0   (delete this line)
```
```
cryptsetup luksClose luks-data
vim /etc/auto.misc
```
edit
```
luks -fstype=xfs :/dev/mapper/luks-data
```
```
systemctl restart autofs
cd /misc/luks
```
######config nfs on server2
machine2
```
yum list nfs*
firewall-cmd --add-service=nfs --permanent && firewall-cmd --reload
systemctl start rpcbind nfs-server && systemctl enable rpcbind nfs-server
mkdir /share
find /usr/share/doc -name '*.pdf' -exec cp {} /share \;
vim /etc/exports
```

edit
```
/share *(ro)
```
then
```
exportfs -r
exportfs -s
```
######auto mount remote mounts
machine1
```
mount -t nfs server2.ex.com:/share /mnt
```
change config
```
vi /etc/auto.misc
```
edit
```
luks -fstype=xfs :/dev/mapper/luks-data
pdf -ro,soft,intr server2.ex.com:/share
```

#####group quota
```
df -hT
rpm -fq $(which quota)
vi /etc/fstab
```
edit
```
UUID="" /data/mydata ext4 noatime,noexec,usrquota 0 2
```
```
quotacheck -mau  //init quota database
```
```
quotaon /dev/sdb6
```
######set user quota
```
repquota -uv /dev/sdb6
```
```
setquota -u bob 20000  //soft limit=softli20mb  25000  //hard limit=25mb 0 0  (group quota) /dev/sdb6
```
```
edquota bob
```
apply other account's quota
```
edquota -u newuser -p bob
```
######xfs quotas
```
xfs_quota -xc 'limit -u bsoft=30m bhard=35m bob' /data/data2
xfs_quota -c 'quota -h bob'  //see report
```

