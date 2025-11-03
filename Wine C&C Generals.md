# Command & Conquer Generals on Linux via Wine
With local multiplayer support!

Prerequisites: original C&C Generals CDs + Patch 1.08

1. Install with patch and vcrun2005:
```
WINEPREFIX=/home/USERNAME/.wine-generals wine setup.exe
WINEPREFIX=/home/USERNAME/.wine-generals wine Generals-108-german.exe

WINEPREFIX=/home/USERNAME/.wine-generals winetricks vcrun2005
```

2. Create config file `/home/USERNAME/Documents/Command and Conquer Generals Data/Options.ini`:
```
AntiAliasing = 0
BuildingOcclusion = yes
CampaignDifficulty = 1
DynamicLOD = no
ExtraAnimations = yes
GameSpyIPAddress = 0.0.0.0
Gamma = 50
HeatEffects = yes
IPAddress = 0.0.0.0
IdealStaticGameLOD = High
LanguageFilter = false
MaxParticleCount = 5000
MusicVolume = 55
Resolution = 1920 1080
Retaliation = yes
SFX3DVolume = 79
SFXVolume = 71
ScrollFactor = 50
SendDelay = no
ShowSoftWaterEdge = yes
ShowTrees = yes
StaticGameLOD = Custom
TextureReduction = 0
UseAlternateMouse = no
UseCloudMap = yes
UseDoubleClickAttackMove = no
UseLightMap = yes
UseShadowDecals = yes
UseShadowVolumes = yes
VoiceVolume = 70
```

Adjust resolution as desired (this enables you to play in full HD - this resolution is not selectable in game).

3. Start game.dat (not game.exe) - this bypasses copy protection checks - no need to insert the CD  :))
```
cd "/home/USERNAME/.wine-generals/drive_c/Program Files/EA Games/Command and Conquer Generals"
WINEPREFIX=/home/USERNAME/.wine-generals wine game.dat
```
