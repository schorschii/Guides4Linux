# ELO Touchscreen Calibration on Linux

## Option 1: evdev
```
sudo apt-get install xserver-xorg-input-evdev xinput_calibrator

/usr/share/X11/xorg.conf.d/40-libinput.conf
# "Touchscreen": libinput -> evdev

xinput_calibrator
# write output to /usr/share/X11/xorg.conf.d/99-calibration.conf

reboot
```

## Option 2: libinput
```
xinput list

# default
xinput set-prop 10 "Coordinate Transformation Matrix"  1 0 0  0 1 0  0 0 1

# invert y
xinput set-prop 10 "Coordinate Transformation Matrix"  1 0 0  0 -1 1  0 0 1

# invert y + scale touches to border (for ELO)
xinput set-prop 10 "Coordinate Transformation Matrix"  1.2 0 -0.1  0 -1.2 1.1  0 0 1
```

see: https://en.wikipedia.org/wiki/Transformation_matrix#Affine_transformations
