# How to get DaVinci Resolve running on Ubuntu 24.04

## Installation
```
sudo apt install libapr1 libaprutil1 libglib2.0-0

sudo SKIP_PACKAGE_CHECK=1 ./DaVinci_Resolve_Studio_19.0_Linux.run -i
```

## Startup
By default, it produces the error:
```
/opt/resolve/bin/resolve: symbol lookup error: /lib/x86_64-linux-gnu/libpango-1.0.so.0: undefined symbol: g_once_init_leave_pointer
```

Solution:
```
sudo mkdir /opt/resolve/libs/obsolete

sudo mv /opt/resolve/libs/libgio* /opt/resolve/libs/obsolete/
sudo mv /opt/resolve/libs/libglib* /opt/resolve/libs/obsolete/
sudo mv /opt/resolve/libs/libgmodule* /opt/resolve/libs/obsolete/
```

## Adjustments for HiRes-Displays (4K)
Start Resolve with the follwing environment variables (e.g. by editing `/usr/share/applications/com.blackmagicdesign.resolve.desktop`).

```
QT_DEVICE_PIXEL_RATIO=2
QT_AUTO_SCREEN_SCALE_FACTOR=true
```

## DeckLink Cards
Normally, DeckLink cards are detected automatically after installing the [Desktop Video](https://www.blackmagicdesign.com/de/support/family/capture-and-playback) software. In case of problems, try downgrading to a older kernel version if possible. Support for newer Linux kernels always comes with a delay.

```
# should show your card
lspci | grep Black

# check service status
service DesktopVideoHelper status

# check status of corresponding kernel module
sudo dkms status
sudo dkms build blackmagic/11.4a14
sudo dkms build blackmagic/11.4a15

# read the log to get why build has failed
nano /var/lib/dkms/blackmagic-io/11.4a14/build/make.log

apt remove desktopvideo
sudo dkms status
```
