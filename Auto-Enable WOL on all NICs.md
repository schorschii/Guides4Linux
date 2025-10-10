# Auto-Enable WOL on all Network Interfaces
WOL needs to be enabled after every system boot again to make it work after shutting down the computer. This script will do it automatically for all network interfaces via a systemd service.

Make sure `ethtool` is installed!

`/usr/sbin/wol-enabler`:
```
#!/bin/bash

if [ "$EUID" -ne 0 ]; then
	echo "Das musst du als root ausführen, du Depp!!"
	exit
fi

for iface in $(ip addr list | awk -F': ' '/^[0-9]/ {print $2}'); do
	if [ "$iface" == "lo" ]; then continue; fi
	echo "Enabling WOL on interface: $iface"
	ethtool -s "$iface" wol g || true
done
```

`/etc/systemd/system/wol-enabler.service`:
```
[Unit]
Description=Enable WOL on all Interfaces
After=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/sbin/wol-enabler

[Install]
WantedBy=basic.target
```

```
systemctl enable wol-enabler.service
```
