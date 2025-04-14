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
```

## Logical Volumes
```
# list all
lvs
# info about one logical volume
lvdisplay

# resize
lvextend --resizefs -L 1G /dev/mylvm/data          # specific size
lvextend --resizefs -l +100%FREE /dev/mylvm/data   # use all avail space
```
