# HP_Chromebook_15
Custom kernel/etc for the Intel KabyLake based HP Chromebook 15 (SYNDRA). Much of this likely applies to the other "NAMI" ChromeOS devices.  

If this project helps you click the Star at the top of the page to let me know!

All of this is currently based on booting the device with the stock BIOS with legacy boot firmware from MrChromebox.tech. I have not made any attempt to replace the stock BIOS with the custom EFI image though that is supposedly possible.

This has all targeted Debian Buster but should be applicable to other Linux distrobutions.

I'm considering creating pacakges to automate setting some of these up and possibly creating a custom Debian installer image to further automate that process. Let me know if that would be of interest to you, it will probably only reach the top of my todo list if there is interest.

## Things I have done/recommend

### Kernel with needed drivers 
Set up an automated script to build a custom kernel for this device based on Debian's amd64 kernel for Bullseye. So far the only change from Debian's is to enable support for the TPM. Without TPM support the device will automatically reflash the boot loader with the stock version if you allow it to suspend....which can wreck your day. 

To setup my repo add install the kernel:
```
apt-get install -y apt-transport-https gnupg
wget -qO - https://raw.githubusercontent.com/1000001101000/HP_Chromebook_15/master/PPA/key.GPG | apt-key add -
echo "deb https://raw.githubusercontent.com/1000001101000/HP_Chromebook_15/master/PPA/ testing main" > /etc/apt/sources.list.d/hp15_kernel.list
apt-get update
apt-get install linux-image-hp15

```

### Microcode
I believe the CPU in this device is one that was affected by a hyperthreading bug:
https://www.extremetech.com/computing/251499-major-hyper-threading-flaw-can-destabilize-intel-cpus-based-kaby-lake-skylake

This can be corrected by installing the intel-microcode package and rebooting.

### Intel graphics firmware
some additional features of the graphics card can be utilized if you install additional firmware blobs (HuC,GuC,dmc). I beleive because I'm using a newer kernel this requires a newer version of the firmware-misc-nonfree which I manually installed from the debian unstable repo.

These can then be enabled via some module parameters:

/etc/modprobe.d/i915.conf
`options i915 enable_fbc=1 enable_guc=2`

### Intel Graphics Drivers
I've seen pretty impressive performance improvements for both video and 3D games (Fallout New Vegas) by setting up the xserver-xorg-video-intel driver. I ran into some minor graphics issues in some gtk applications when using "uxa" and "sna" acceleration, "blt" seems at least as fast as the others and doesn't have that issue (admittedly I don't really understand the difference between them). The "TearFree" options prevents "Tearing" issues which can be easily deomnstrated by any of the 60fps tearing test videos on YouTube.


`apt-get install xserver-xorg-video-intel`

/etc/X11/xorg.conf.d/intel.conf:
```
Section "Device"
   Identifier  "Intel Graphics"
   Driver      "intel"
   Option "TearFree"    "true"
   Option "AccelMethod" "blt"
EndSection
```

The Arch wiki has lots of good inormation about this:  
https://wiki.archlinux.org/index.php/intel_graphics
https://www.x.org/releases/current/doc/man/man4/intel.4.xhtml


### Suspend to memory
The "deep" mode only seems to work when the Intel graphics drivers are used rather than the modeset drivers (see above). 

GalliumOS and most other sources I've looked at include this setting to avoid crashes, I haven't yet tested suspend without it:
`echo 0 > /sys/power/pm_async`

If "deep" mode isn't work for some reason I found that s2idle worked fairly well though I suspet the power savings aren't as good. 

`echo s2idle > /sys/power/mem_sleep`

### Keyboard backlight
It appears that they removed support for the keyboard backlight in the bootloader/BIOS somewhere around ChromeOS 77, even when it was supported I wasn't able to find any way to adjust it within Chrome OS.

I was able to make it work on my device by: 
- downloading a ChromeOS 75 recovery image from https://cros-updates-serving.appspot.com
- restore ChromeOS 75 using the recovery image (enable debug options when it asks and set a root password).
- before connecting to wifi use alt-F2 (right arrow) to connect to a terminal and log in with the root password you just set
- disable autoupdates `rm /etc/update_manager.conf` then run chromeos-setdevpassword so that you can run root commands within ChromeOS
- restart so that you have a session with network that isn't trying to autoupdate.
- run through the process to enable legacy boot and then install linux as normal

With a working bootloader/BIOS the cros_kbd_led_backlight driver works as designed. The brightness can be set manually with:

`echo 50 > "/sys/class/leds/chromeos::kbd_backlight/brightness"`

I tried dumping the ACPI table from ChromeOS 75's BIOS and using it to override the table from ChromeOS 87 but that didn't help. Looking at a diff of the two tables the defferences between them don't appear to be related to the keyboard backlight anyway. For now downgrading before installing linux is the only solution I'm aware of. I actually only looked at the DSDT table, I've since realized I should look at SSDT as well. At some point I'll take another look.....next time I feel like re-flashing my laptop.

I set up a hotkey to set the values using CTRL + the brightness keys using a triggerhappy script. I may move to another method eventually.


### Touchpad driver
The Synaptic and other Xorg input drivers don't work very well with this touchpad (at least not with any settings that I tried). The GalliumOS folks have ported the ChromeOS driver which works much better. 

You can install them by installing the DEBs for xserver-xorg-input-cmt, libgestures, and libevdevc from http://apt.galliumos.org. 

At least in XFCE this appears to remove the touchpad tab from mouse settings. I'm still looking at how to restore that or how to adjust things like the typing timout via Xorg.conf or xinput. 

### Grub menu
Setting the grub resolution to 1024x768 makes the best use of the screen of the modes available in the Buster version of Grub2

`GRUB_GFXMODE=1024x768` 

###  kexec for faster/remote reboots
I've been using kexec instead of a normal reboot to speed things up and to avoid the need for pressing CTRL+L at boot. 

To accomplish this I put together a simple script that parses the default kernel/initrd from the GRUB config and re-using the current kernel cmdline:

```
#!/bin/bash

## requires kexec-tools 
## /lib/systemd/system-shutdown/kexec.sh

if [ "$1" == "reboot" ]; then

   kernel="$(grep -e ^[[:space:]]*linux /boot/grub/grub.cfg | head -n 1 | awk '{print $2}')"
   initrd="$(grep initrd /boot/grub/grub.cfg | head -n 1 | awk '{print $2}')"

   kexec -l --initrd=$initrd  --reuse-cmdline $kernel
   kexec -e
fi
```

### keyboard configuration
The keyboard for this device works well with the "Chromebook" mapping which is available in XFCE and I suspect other window managers as well. 

As usual Arch has a good guide describing how to automatically enable numlock:

https://wiki.archlinux.org/index.php/Activating_numlock_on_bootup

## Things I am looking into

### media keys
I still need to look at how to make the backlight/etc buttons work (rather than as function keys). I have a triggerhappy script set up to handle the keyboard backlight adjustments but a not sure if that is the right direction to go.

### Sound
This requires some firmware files you need to extract from a chromos recovery image and probably some additional dev work. I'm not aware of anyone who has successfully done this for this device

This link has some information that might be helpful:

https://github.com/GalliumOS/galliumos-distro/issues/536

I've tried adapting the method for this device but haven't made much progress yet.

Personally, I just use bluetooth audio for now.

It appeared hdmi audio was set up on GalliuOS, I have not yet checked if that actually worked or how to get it working on Debian. 

<br><br><br>


To enable "legacy" boot to allow custom OS install:
https://mrchromebox.tech/

Backlight issue:
https://bugs.chromium.org/p/chromium/issues/detail?id=997415
https://bugs.chromium.org/p/chromium/issues/detail?id=1176224&q=HP%20chromebook%2015&can=2
