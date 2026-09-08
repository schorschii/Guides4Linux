# CUPS (PIN-protected) locked print with Ricoh devices

I recently needed to set up PIN-protected print with a Ricoh M 320FSE.

Unfortunately, there is no exactly matching PPD file on [openprinting.org](https://www.openprinting.org/download/PPD/Ricoh/) available and the PPD files on the [Ricoh website](https://support.ricoh.com/bb/html/dr_ut_e/re2/model/m320/m320.htm) have a very limited feature set (no PIN option).

So I tested PS/PXL PPD files from other Ricoh devices. They basically work but the locked print option is ignored by the Ricoh printer, it just prints the document without PIN prompt... this means: it's time for Wireshark again!

Capturing a print job from the Windows driver shows the following PXL preamble:
```
@PJL JOB NAME="LibreOffice - Dokument1"
@PJL SET JOBNAME="LibreOffice - Dokument1"
@PJL SET LOCKEDPRINT="2222"
@PJL SET USERID="17:07 <windows-username>"
@PJL SET PCHOSTNAME="BDV020"
@PJL SET TIME="17:07:27"
@PJL SET DATE="2026/09/08"
@PJL SET HOSTLOGINNAME="<windows-username>"
@PJL SET DISPCHARSET="iso-8859-1"
@PJL SET HOSTCHARSET="iso-8859-1"
@PJL SET DUPLEX=ON
@PJL SET BINDING=LONGEDGE
@PJL SET MEDIATYPE=PAPER
@PJL SET RESOLUTION=600
@PJL SET BITSPERDOT=1
@PJL SET ECONOMODE=OFF
@PJL SET FRONTCOVERPRINT=OFF
@PJL SET DATAMODE=BW
@PJL SET BLACKOVERPRINT=OFF
@PJL SET IGNOREBLANKPAGES=OFF
@PJL ENTER LANGUAGE=PCLXL
```

The interesting line is `@PJL SET LOCKEDPRINT="2222"`: every PPD file I've seen so far sets the password as `@PJL SET JOBPASSWORD="2222"`. I don't know what Ricoh tought here, but luckily this leads us to the simple solution: just change the variable name.

So all you need to do is to replace the (5) occurences of `JOBPASSWORD` in the PPD file with `LOCKEDPRINT`. Then, select the PPD file for your printer in the CUPS GUI, and locked printing will work fine!
