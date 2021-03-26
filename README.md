# HP_Chromebook_15
Custom kernel/etc for the Intel KabyLake based HP Chromebook 15 (SYNDRA). Much of this likely applies to the other "NAMI" ChromeOS devices.  

All of this is currently based on booting the device with the stock BIOS with legacy boot firmware from MrChromebox.tech. I have not made any attempt to replace the stock BIOS with the custom EFI image though that is supposedly possible.

This has all targeted Debian Buster but should be applicable to other Linux distrobutions.

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
some additional features of the graphics card can be utilized if you install additional firmware blobs (HuC,GuC,dmc). I beleive because I'm using a newer kernel this requires a newer version of the firmware-misc-nonfree which I manually installed from the debian unstable repo.


### Keyboard backlight
It appears that they removed support for the keyboard backlight in the bootloader/BIOS somewhere around ChromeOS 77, even when it was supported I wasn't able to find any way to adjust it within Chrome OS.

I was able to make it work on my device by: 
- downloading a ChromeOS 75 recovery image from https://cros-updates-serving.appspot.com
- restore ChromeOS 75 using the recovery image (enable debug options when it asks and set a root password).
- before connecting to wifi use alt-F2 (right arrow) to connect to a terminal and log in with the root password you just set
- disable autoupdates `rm /etc/update_manager.conf` then run chromeos-setdevpassword so that you can run root commands within ChromeOS
- restart so that you have a session with network that isn't trying to autoupdate.
- run through the process to enable legacy boot and then install linux as normal

With a working bootloader/BIOS the cros_kbd_led_backlight driver works as designed. The brightness can be set with:

`echo 50 > "/sys/class/leds/chromeos::kbd_backlight/brightness"`

I tried dumping the ACPI table from ChromeOS 75's BIOS and using it to override the table from ChromeOS 87 but that didn't help. Looking at a diff of the two tables the defferences between them don't appear to be related to the keyboard backlight anyway. For now downgrading before installing linux is the only solution I'm aware of.

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
I still need to look at how to make the backlight/etc buttons work (rather than as function keys). I have a triggerhappy script set up to handle the keyboard backlight adjustments but a nott sure if that is the right direction to go.

### Sound
This requires some firmware files you need to extract from a chromos recovery image and probably some additional dev work. I'm not aware of anyone who has successfully done this for this device

This link has some information that might be helpful:

https://github.com/GalliumOS/galliumos-distro/issues/536

I've tried adapting the method for this device but haven't made much progress yet.

Personally, I just use bluetooth audio for now.

It appeared hdmi audio was set up on GalliuOS, I have not yet checked if that actually worked or how to get it working on Debian. 

<br><br><br>


I may bundle all of the above into a Debian installer image since I've done that on other projects and have experience setting up github CI/CD jobs to keep them updated. I normally wouldn't bother for a non-embedded device like this, but getting a kernel in place that supports the TPM right away is probably worth the hassle. 

Let me know if you have any questions or something to contribute. I don't know if anyone will actually run across this repo or not, I don't plan to promote it unless I discover something interesting (like how to enable the keyboard backlight).



To enable "legacy" boot to allow custom OS install:
https://mrchromebox.tech/

Backlight issue:
https://bugs.chromium.org/p/chromium/issues/detail?id=997415
https://bugs.chromium.org/p/chromium/issues/detail?id=1176224&q=HP%20chromebook%2015&can=2
