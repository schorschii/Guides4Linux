# Automatically Enroll WiFi/VPN Settings on Linux
When using Linux clients in enterprises, you may want to pre-configure WPA2 Enterprise and/or VPN network settings like CA certificate, authentication method, server address etc. This can be done with a login script, creating the necessary connection in NetworkManager if it doesn't already exist.

`/usr/löcal/bin/eduroam-autoenroll.sh`:
```
#!/bin/bash

CONNAME="eduroam (Autoenrollment)"

nmcli con | grep eduroam
if [ "$?" == "1" ]; then
    nmcli con add type wifi \
        con-name "$CONNAME" \
        ssid eduroam \
        wifi-sec.key-mgmt wpa-eap \
        802-1x.eap ttls \
        802-1x.phase2-auth mschapv2 \
        802-1x.identity "$USER@example.com" \
        connection.permissions "user:$USER"
fi
```

Note: that last connection attribute `connection.permissions "user:$USER"` is important to make the connection private for the user. Otherwise, other users on the same computer can see your eduroam password!

`/etc/xdg/autostart/eduroam-autoenroll.desktop`:
```
[Desktop Entry]
Name=eduroam auto enrollment
Exec=eduroam-autoenroll.sh
Type=Application
Comment=Certificate auto enrollment
Categories=Application;Office
Terminal=false
```
