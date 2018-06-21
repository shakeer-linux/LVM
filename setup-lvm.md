
**LVM**    
**L**ogical **V**olume **M**angaer is widely used technique for deploying logical rather than physical storage. With LVM "logical" partitons can span across physical harddrive and can be resized.   


LVM permits having one logical filesystem span multiple physical volumes and partitions while appearing as a simple partition for normal usage. Disk partitions are converted into physical volumes and multiple physical volumes are grouped into a volume group. Then the volume group is subdivided into logical volumes.


* Logical volumes can be resized, enabling you to shift space between filesystem without reinstalling the system.  
* Logical volumes can span multiple physical volumes, enabling the use of filesystems that are larger than one physical disk.
* Additional storage can be added to existing filesystems *--for example, you can add a new disk drive and add that storage space to the home filesystem.
* Data can be migrated from one drive to other.

#### i) Adding a Logical Volume:

Create two logical partitions inside an extended partition and set type to 8e:

```sh
fdisk /dev/sdb
Type ‘t‘ to set partition type to 8e  (Linux LVM)

partprobe -s /dev/sdb
parted /dev/sdb print 
```
Create two Physical Volumes from the partitions:
```
pvcreate    /dev/sdb5    /dev/sdb6
pvdisplay; pvs
```
Create a Volume Group:

```
vgcreate vg1   /dev/sdb5   /dev/sdb6
vgdisplay; vgs
```
Allocate a Logical Volume from the volume group:
```
lvcreate   -n [–name] lv1  -L [–size]  300M  vg1
lvdisplay; lvs
```
Format the Logical Volume:
```
mkfs.ext4   /dev/vg1/lv1
```
Mount the Logical Volume:
```
mkdir /mnt/lv1; chmod a+rw /mnt/lv1
echo “/dev/vg1/lv1  /mnt/lv1  ext4   defaults  0  0″ >> /etc/fstab
mount -a; df -hT
```

#### ii) Extend and Reduce logical volume:
Extending logical volumes:
Extend both logical volume and file-system (ext4, xfs):
```
lvextend –resizefs  –size [+|-]100M   /dev/vg1/lv1
Alternatively you may run the following commands:
lvextend -L +100M /dev/vg1/lv1    (First resize volume) 
resize2fs  /dev/vg1/lv1                     (Then resize file-system)
```
Reducing logical volumes:
Reduce both logical volume and file-system (ext4):
```
lvredue  –resizefs  –size [+|-]200M  /dev/vg1/lv1
```
For xfs file-system follow these steps: (automatic reducing is not supported)
```
xfsdump -f   /tmp/lv1.dump  /mnt/lv1
umount   /mnt/lv1
lvreduce  -L 200M   /dev/vg1/lv1
mkfs.xfs  -f  /dev/vg1/lv1
mount   /dev/vg1/lv1
xfsrestore -f  /tmp/lv1.dump  /mnt/lv1
```
#### iii) Extend and Reduce volume group:

Extend a volume group:
```
vgextend vg1 /dev/sdb7
Reduce a volume group:
pvmove  /dev/sdb6  [dest PV]  (Relocate PEs from /dev/pv6 to unused PVs in vg1)
vgreduce  vg1  /dev/pv6           (Removes /dev/pv6 from vg1)
```
#### iv) Create LVM snapshot:
When resizing volumes it is useful to create a snapshot of logical volumes to ensure that data is not lost. To do so there must be enough room on the volume group first.
```
vcreate   -s [–snapshot]  -L [–size] 100M   -n [–name] lv1snap  /dev/vg1/lv1
mkdir /mnt/lv1snap
mount   /dev/vg1/lv1snap   /mnt/lv1snap 
```
#### v) Remove Physical Volumes, Volume Group and Logical Volume:
```
umount   /dev/vg1/lv1
lvremove   /dev/vg1/lv1
vgremove   /dev/vg1
pvremove   /dev/sdb5   /dev/sdb6
```
#### vi) Miscellaneous commands:

Physical Volumes:
```
pvcreate PV: Initializes physical volume(s) for use by LVM
pvremove PV:  Remove LVM label(s) from physical volume(s)
pvdisplay | pvs | pvscan:  List all physical volume(s)
pvmove PV_src [PV_dest]:  Move extents from one physical volume to another
```
Volume Group:
```
vgcreate VG PV:  Creates a volume group
vgremove VG: Removes a volume group
vgextend VG PV: Add physical volume(s) to a volume group
vgreduce VG PV: Removes physical volume(s) from a volume group
vgdisplay | vgs | vgscan: List all volume groups
vgrename VG VG_new: Rename a volume group name
```
Logical Volume:
```
lvcreate –size –name LV VG: Create a logical volume
lvremove /dev/VG/LV: Remove logical volume(s) from the system
lvextend –resizefs –size /dev/VG/LV: Extend an LV by a specified size
lvreduce –resizefs –size /dev/VG/LV: Reduce the size of a logical volume
lvdisplay | lvs: Display information about a logical volume
lvscan | lvmdiskscan: List all logical volumes in all volume groups
lvrename VG LV  LV_new: Rename logical volume name
```
