# HP_Chromebook_15
Custom kernel/etc for HP Kaby Lake Chromebook

I got this device at a discount and am mostly happy with it for the price and am now looking at improving linux support for it however I can. 

So far I have:

Set up an automated script to build a custom kernel for this device based on Debian's amd64 kernel for Bullseye. So far the only change from Debian's is to enable support for the TPM. Without TPM support the device will automatically reflash the boot loader with the stock version if you allow it to suspend....which can wreck your day. 


Things I am looking into:

Figuring out how to enable the keyboard backlight. When I bought the thing I expected I could figure this out from ChromeOS....but it doesn't work there either. As of today HP hasn't posted any GPL code for the device either. I've spent some time poking at gpios and ACPI entries but haven't had any luck yet.

A proper keymapping that includes the numpad.

Proper defaults for the trackpad. GalliumOS's defaults work great, Debian's were not very good. When I move from GalliumOS to Debian I'll figure out how to make it work the same on Debian and post info here in some form.

GalliumOS has the sound working with their kernel, but it does not with mine. I beleive part of the reason they've kept an older kernel/base is that something changed that makes this harder to support for these chipsets. I've read some things that make it sound like this is possible to fix, i'll look into that at somepoint.

I may bundle all of the above into a Debian installer image since I've done that on other projects and have experience setting up github CI/CD jobs to keep them updated. I normally wouldn't bother for a non-embedded device like this, but getting a kernel in place that supports the TPM right away is probably worth the hassle. 

Let me know if you have any questions or something to contribute. I don't know if anyone will actually run across this repo or not, I don't plan to promote it unless I discover something interesting (like how to enable the keyboard backlight).



To enable "legacy" boot to allow custom OS install:
https://mrchromebox.tech/

Backlight issue:
https://bugs.chromium.org/p/chromium/issues/detail?id=997415
