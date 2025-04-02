# Capture MiniDV Camera Signal via FireWire on Linux
Some Pinacle PCI FireWire Video Cards are not supported.

```
apt install libavc1394-0 libavc1394-tools libraw1394-11 libraw1394-tools kino dvgrab
reboot

# load kernel modules
sudo modprobe firewire-core firewire-net firewire-ohci firewire-sbp2

# should show FireWire devices
ls /dev/fw*

# GUI for recording
kino

# CLI for recording
dvgrab
```
