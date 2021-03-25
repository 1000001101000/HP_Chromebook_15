# HP_Chromebook_15
Custom kernel/etc for HP Kaby Lake Chromebook

I got this device at a discount and am mostly happy with it for the price and am now looking at improving linux support for it however I can. 


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

### intel graphics firmware
some additional features of the graphics card can be utilized if you install additional firmware blobs

### Keyboard backlight
It appears that they removed support for the keyboard backlight in the bootloader/BIOS somewhere around ChromeOS 77, even when it was supported I wasn't able to find any way to adjust it within Chrome OS.

I was able to make it work on my device by: 
- downloading a ChromeOS recovery image from https://cros-updates-serving.appspot.com
- restore ChromeOS using the recovery image (enable debug options when it asks and set a root password).
- disable network once in ChromeOS so that it doesn't auto-update and restore the current version
- use alt-F2 (right arrow) to connect to a terminal and disable autoupdates (remove update script from /etc/?) i'm having trouble finding the link I used.
- restart so that you have a session with network that isn't trying to autoupdate.
- run through the process to enable legacy boot and then install linux as normal

With a working bootloader/BIOS the cros_kbd_led_backlight driver works as designed. The brightness can be set with:

`echo 50 > "/sys/class/leds/chromeos::kbd_backlight/brightness"`

Eventually I want to see if I can make it work without needed to downgrade the bootlaoder. I suspect this should be possible by patching the ACPI table to add back in the proper entry. Admitedly I don't have any experience with that and don't really want to reflash my device now that it's working properly.


### Touchpad driver
The Synaptic and other Xorg input drivers don't work very well with this touchpad (at least not with any settings that I tried). The GalliumOS folks have ported the ChromeOS driver which works much better. 

You can install them by installing the DEBs for xserver-xorg-input-cmt, libgestures, and libevdevc from http://apt.galliumos.org. 

At least in XFCE this appears to remove the touchpad tab from mouse settings. I'm still looking at how to restore that or how to adjust things like the typing timout via Xorg.conf or xinput. 

### Grub menu
Setting the grub resolution to 1024x768 makes the best use of the screen of the modes available in the Buster version of Grub2
`GRUB_GFXMODE=1024x768`

### keyboard configuration
The keyboard for this device works well with the "Chromebook" mapping which is available in XFCE and I suspect other window managers as well. 

As usual Arch has a good guide describing how to automatically enable numlock:

https://wiki.archlinux.org/index.php/Activating_numlock_on_bootup

## Things I am looking into

### Suspend to memory
This works perfectly using my kernel under GalliumOS but is failing on Debian. I'm having trouble figuring out what the difference is, partly because switching between the two is a pain.

For now I've been using suspend to idle if I need to use suspend. 
`echo s2idle > /sys/power/mem_sleep`

### media keys
I still need to look at how to make the backlight/etc buttons work (rather than as function keys). I have a triggerhappy script set up to handle the keyboard backlight adjustments but a not sure if that is the right direction to go.

### Sound
I think GalliumOS has the sound working with their kernel, but it does not with mine. I beleive part of the reason they've kept an older kernel/base is that something changed that makes this harder to support for these chipsets. I've read some things that make it sound like this is possible to fix, i'll look into that at somepoint.

This link has some information that might be helpful:

https://github.com/GalliumOS/galliumos-distro/issues/536

I've tried adapting the method for this device but haven't made much progress yet

<br><br><br>


I may bundle all of the above into a Debian installer image since I've done that on other projects and have experience setting up github CI/CD jobs to keep them updated. I normally wouldn't bother for a non-embedded device like this, but getting a kernel in place that supports the TPM right away is probably worth the hassle. 

Let me know if you have any questions or something to contribute. I don't know if anyone will actually run across this repo or not, I don't plan to promote it unless I discover something interesting (like how to enable the keyboard backlight).



To enable "legacy" boot to allow custom OS install:
https://mrchromebox.tech/

Backlight issue:
https://bugs.chromium.org/p/chromium/issues/detail?id=997415
https://bugs.chromium.org/p/chromium/issues/detail?id=1176224&q=HP%20chromebook%2015&can=2
