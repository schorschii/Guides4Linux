# How to get DaVinci Resolve running on Ubuntu 24.04


## Installation
```
sudo apt install libapr1 libaprutil1 libasound2 libglib2.0-0

sudo SKIP_PACKAGE_CHECK=1 ./DaVinci_Resolve_Studio_19.0_Linux.run -i
```

## Startup
By default, it produces the error:
```
/opt/resolve/bin/resolve: symbol lookup error: /lib/x86_64-linux-gnu/libpango-1.0.so.0: undefined symbol: g_once_init_leave_pointer
```

Solution:
```
sudo mkdir /opt/resolve/libs/obsolete

sudo mv /opt/resolve/libs/libgio* /opt/resolve/libs/obsolete/
sudo mv /opt/resolve/libs/libglib* /opt/resolve/libs/obsolete/
sudo mv /opt/resolve/libs/libgmodule* /opt/resolve/libs/obsolete/
```
