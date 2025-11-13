# Linux Kiosk System
Linux suits perfectly for embedded systems, e.g. for (public) kiosk browser workstations, room meeting schedule displays etc.

## Basic Configuration
To only start one specific application (and not the entire desktop), you can modify your Xsession (`/usr/share/xsessions/`) or Wayland session (`/usr/share/wayland-sessions/`) file to look like this:

```
[Desktop Entry]
Name=Chrome Kiosk
Comment=Start Chrome in Kiosk mode (no desktop environment)
Exec=google-chrome --kiosk https://example.com
Icon=
Type=Application
```

Make sure that no session files for normal desktop sessions are available anymore (otherwise, the user could switch to the desktop session on the login screen).

When the Chrome process terminates, the user is automatically logged out and back on the login screen.

## OpenKiosk
For a more advanced kiosk browser experience (URL whitelists & blacklists etc.), have a look at [OpenKiosk](https://openkiosk.mozdevgroup.com/).

## Auto Profile Reset
For a more complex public Linux workstation with automatic profile (home dir) reset after x minutes, have a look at [Linux-Public-Workstation](https://github.com/slub/Linux-Public-Workstation).
