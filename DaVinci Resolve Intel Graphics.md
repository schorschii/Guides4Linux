# How to get DaVinci Resolve running with Intel (integrated or PCIe graphics card) GPU on Linux

You need to install the Intel OpenCL libraries with [this patch](https://github.com/intel/compute-runtime/pull/673).

Installing the Ubuntu package `intel-opencl-icd` makes DaVinci start but it does not render any Video since the version shipped in Ubuntu 24.04 is too old and does not contain the mentioned patch.

Install the newest versions from the Intel repo:  
https://github.com/intel/compute-runtime/releases  
https://github.com/intel/intel-graphics-compiler/releases

If you are using older hardware, you need to download the packages with "legacy1" in the file name, see [here](https://github.com/intel/compute-runtime/blob/master/LEGACY_PLATFORMS.md).

---

The Ubuntu package `intel-media-va-driver-non-free` is propably interesting for you too. It can decrease CPU load on video playback on Intel hardware.
