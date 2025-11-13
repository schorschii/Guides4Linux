# Apple SuperDrive on Linux

This is the perfect example why this proprietary Apple shit is annoying. You need to send a non-standard command every time after power on to activate the drive.

```
sudo apt install sg3-utils
sudo sg_raw /dev/sr* EA 00 00 00 00 00 01
```

To automate this, create an udev rule `/etc/udev/rules.d/99-superdrive.rules`:
```
ACTION=="add", ATTRS{idVendor}=="05ac", ATTRS{idProduct}=="1500", SUBSYSTEM=="block", RUN+="/usr/bin/sg_raw $root/$kernel EA 00 00 00 00 00 01"
```
```
udevadm control --reload-rules
```
