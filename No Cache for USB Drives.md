# Disable Write Cache for USB drives
Disable filesystem write cache on USB devices with a simple udev rule. Caching is great but causes long unmounting time which is annoying on USB devices. 

Create `/etc/udev/rules.d/99-usb-sync.rules`:
```
KERNEL!="sd[a-z]|sd[a-z][0-9]", GOTO="usb_limit_write_cache_end"
ENV{ID_USB_TYPE}!="disk", GOTO="usb_limit_write_cache_end"
ACTION!="add|change", GOTO="usb_limit_write_cache_end"

ENV{ID_FS_USAGE}=="filesystem", ENV{UDISKS_MOUNT_OPTIONS_DEFAULTS}+="sync", ENV{UDISKS_MOUNT_OPTIONS_ALLOW}+="sync", GOTO="usb_limit_write_cache_end"

LABEL="usb_limit_write_cache_end"
```

USB drives are now automatically mounted using the `sync` option to directly write to disk. Unounting those drives will now be very fast after a copy process finished.
