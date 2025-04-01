# Command & Conquer Tiberian Sun on Linux via Wine
With local multiplayer support via IPX!

Prerequisites: original C&C Tiberian Sun CDs + Patch 2.03  
https://cnc-comm.com/tiberian-sun/downloads/patches

1. Compile IPX kernel module and userspace tools:
```
apt install git build-essential

git clone https://github.com/pasis/ipx.git
cd ipx/
make
sudo modprobe p8022
sudo modprobe psnap
sudo insmod ipx.ko

cd ..
git clone https://github.com/pasis/ipx-utils.git
cd ipx-utils/
apt install autoconf
./autogen.sh
./configure
make
sudo ./ipx_interface add -p eth0 802.2
ifconfig # should display the IPX address
```

2. Install Tiberian Sun:
```
sudo -i # you need to execute wine as root to make IPX work
export WINEPREFIX=/root/.wine-ts WINEARCH=win32
winecfg # set OS to Win98
wine regedit
```

Import the following into the registry:
```
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Wine\AppDefaults\setup.exe\Direct3D]
"Renderer"="opengl"
"RenderTargetLockMode"="readtex"
"VideoMemorySize"="2048"

[HKEY_CURRENT_USER\Software\Wine\AppDefaults\game.exe\Direct3D]
"Renderer"="gdi"
"RenderTargetLockMode"="readtex"
"VideoMemorySize"="2048"
```

Insert CD or mount image with gCDemu and install game + latest patch
```
wine setup.exe
wine patch.exe
```
