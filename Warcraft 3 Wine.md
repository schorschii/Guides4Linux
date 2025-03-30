# Warcraft III on Linux

Prerequisites: original Warcraft III CDs

1. Create ISO images from your CDs using K3B. Important: check "clone copy" checkbox, otherwise copy protection will not work.

2. Mount the created ISO files using gCDEmu. In gCDEmu settings, check "DPM Emulation", "Rate Emulation", "Bad Sector Emulation".
   (Note: the original CDs are multisession discs - when mounting with the Gnome integrated ISO mounter we get the OSX session, but we need the Windows installer)

3. Install newest Wine version as described in https://gitlab.winehq.org/wine/wine/-/wikis/Download

4. Install necessary video codecs: `apt install gstreamer1.0-plugins-good:i386`

5. Create a 32bit wineprefix for WC3 and install necessary Windows libs:
   ```
   WINEPREFIX=/home/USERNAME/.wine-wc WINEARCH=win32 winetricks corefonts vcrun6 ffdshow xvid wsh57 wmp9 l3codecx lavfilters binkw32

   # open wine settings and uncheck box "Graphic" -> Allow window manager..." to make videos work
   WINEPREFIX=/home/USERNAME/.wine-wc winecfg
   ```

6. Install Reign Of Chaos:
   ```
   # install!
   WINEPREFIX=/home/USERNAME/.wine-wc wine /path/to/your/iso/install.exe

   # play!
   WINEPREFIX=/home/USERNAME/.wine-wc wine "/home/USERNAME/.wine-wc/drive_c/Program Files (x86)/Warcraft III/Warcraft III.exe" -nativefullscreen -graphicsapi OpenGL2
   ```

7. Install The Frozen Throne
   ```
   # install!
   WINEPREFIX=/home/USERNAME/.wine-wc wine /path/to/your/iso/install.exe
   
   # play!
   WINEPREFIX=/home/georg/.wine-wc wine "/home/USERNAME/.wine-wc/drive_c/Program Files/Warcraft III/Frozen Throne.exe"
   ```

8. Install patch 1.27 to play without CD/ISO mounted and to get the bonus campaign. Use 1.27 only if The Frozen Throne is installed. 1.27 will brick the game if only Reign of Chaos is installed.

   https://www.moddb.com/games/warcraft-iii-frozen-throne/downloads?filter=t&kw=&category=4&categoryaddon=&timeframe=
   
   ```
   WINEPREFIX=/home/USERNAME/.wine-wc wine patch.exe
   ```
