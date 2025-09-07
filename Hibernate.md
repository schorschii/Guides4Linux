
# Enable Hibernate Ubuntu 20-24

**First, you need a swap space (partition or file) as big as your RAM (or more).**

If you already have a swap partition, you can resize it (and shrink your system partiton) using e.g. Gparted a live system. If you have installed your system inside LVM, you can use the [lvreduce and lvextend](LVM.md) commands.

Test it with: `sudo systemctl hibernate`.

## (Tell initramfs where to resume from)
If your computer does not restore the previous state when starting it again, you need to explicitely tell where the swap to resume from is.
```
sudo nano /etc/initramfs-tools/conf.d/resume
```
```
RESUME=/dev/xxx resume_offset=xxx
```
`resume_offset` is only required if your swap space is located in a file. Find the offset then using `sudo filefrag -v /swapfile` (`physical_offset` first line).

```
sudo update-initramfs -c -k all
```

## Enable Hibernate option in GUI
If everything works, hibernate needs to be enabled for the (Gnome) desktop to make the button visible in the power menu.

### Ubuntu 24.04 or newer
```
sudo nano /etc/polkit-1/rules.d/10-enable-hibernate.rules
```

```
polkit.addRule(function(action, subject) {
    if (action.id == "org.freedesktop.login1.hibernate" ||
        action.id == "org.freedesktop.login1.hibernate-multiple-sessions" ||
        action.id == "org.freedesktop.upower.hibernate" ||
        action.id == "org.freedesktop.login1.handle-hibernate-key" ||
        action.id == "org.freedesktop.login1.hibernate-ignore-inhibit")
    {
        return polkit.Result.YES;
    }
});
```

### Ubuntu 22.04 or older
```
sudo mkdir -p /etc/polkit-1/localauthority/50-local.d
sudo nano /etc/polkit-1/localauthority/50-local.d/com.ubuntu.enable-hibernate.pkla
```

```
[Re-enable hibernate by default in upower]
Identity=unix-user:*
Action=org.freedesktop.upower.hibernate
ResultActive=yes

[Re-enable hibernate by default in logind]
Identity=unix-user:*
Action=org.freedesktop.login1.hibernate;org.freedesktop.login1.handle-hibernate-key;org.freedesktop.login1;org.freedesktop.login1.hibernate-multiple-sessions;org.freedesktop.login1.hibernate-ignore-inhibit
ResultActive=yes
```
