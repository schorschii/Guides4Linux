# Install PPDs System-Wide for Selection in `system-config-printer` GUI

If you want to deploy special/custom CUPS PPD driver files onto managed computers, you need to create a special archive using PyPPD.

Assuming your PPD files are in a temporary directory named `epson_tmt88/`, execute the following:
```
apt install pyppd
pyppd epson_tmt88/ -o epson_thermal
```
Move `epson_thermal` into `/usr/lib/cups/driver/`.

Now, when adding a printer via `system-config-printer` GUI, the new PPDs will be shown in the model selection. No need to manually select the path to the PPD anymore.
