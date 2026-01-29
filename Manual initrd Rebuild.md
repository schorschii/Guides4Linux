# Manual initrd/initramfs Rebuild
Rebuild an existing Ubuntu initrd/initramfs with modifications.

An initrd is a file of multiple cpio archives directly concatenated onto each other - crazy!

1. extract initrd with: `unmkinitramfs initrd.img...-generic extracted`
2. `cd extracted`
3. make changes
4. execute this script to repack

```
#!/bin/bash
set -e

if [ "$1" == "" ]; then
        echo "Enter target file name as first parameter!"
        exit 1
fi

cd early
find . -print0 | cpio --null --create --format=newc > "../$1"

cd ../early2
find . -print0 | cpio --null --create --format=newc >> "../$1"

cd ../early3
find . -print0 | cpio --null --create --format=newc >> "../$1"

cd ../main
find . -print0 | cpio --null --create --format=newc >> "../$1"
```
