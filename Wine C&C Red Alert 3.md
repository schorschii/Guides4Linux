# Command & Conquer Red Alert 3 on Linux via Wine
With local multiplayer support!

Prerequisites: original C&C Red Alert 3 DVD + Patch 1.012  
https://www.united-forum.de/downloads/alarmstufe-rot-3-patch-1-012.52/  
https://www.moddb.com/games/cc-red-alert-3/downloads/cc-red-alert-3-v112-german-patch

1. Install with patch:
```
WINEPREFIX=/home/USERNAME/.wine-ra3 WINEARCH=win32 wine AutoRun.exe
WINEPREFIX=/home/USERNAME/.wine-ra3 wine RedAlert3_german_patch1.012.exe
```

2. Start & configure
   - do not set resolution higher than 1920x1080
   - do not set graphics too high - what worked for me was:
     - Anti Alias: Off
     - Terrain details: Ultra High
     - Water details: High
     - Model details: High
     - Shader details: Mid
     - VFX details: Mid
     - Shadows: High
     - Vertical Sync: On
     - Texture quality: High

   With higher settings, I experience black screens and freezes.
