# Logical Volume Manager
Group disks together to get a giant storage pool!

## Physical Volumes
```
# show/list
pvs
pvdisplay -m

# create
pvcreate /dev/sda3

# resize
# note: better use gparted which also directly updates the partition table!
pvresize --setphysicalvolumesize 5G /dev/sda3
```

## Volume Groups
```
# list all
vgs
# info about one volume group
vgdisplay mylvm

# create
vgcreate vg00 /dev/sdb1 /dev/sdc1
```

## Logical Volumes
```
# list all
lvs
# info about one logical volume
lvdisplay

# create
lvcreate -n data -L1G vg00
lvcreate -n data -l100%VG vg00
mkfs.ext4 /dev/vg00/data

# resize
lvextend --resizefs -L 1G /dev/mylvm/data          # specific size
lvextend --resizefs -l +100%FREE /dev/mylvm/data   # use all avail space
lvreduce --resizefs --size -50G /dev/mylvm/data    # shrink
```
