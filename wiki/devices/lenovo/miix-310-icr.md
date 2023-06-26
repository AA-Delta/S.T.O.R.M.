# Device info

## What's Working:

| **Hardware**     | **PCI/USB ID** |   **Works?**   |
| ---------------- | -------------- | -------------- |
| Keyboard         | 258a:1015      | Yes            |
| Touchpad         |                | Yes            |
| Touchscreen      |                | Yes            |
| GPU              | 8086:22b0      | Yes            |
| Screen           |                | Depends\*      |
| Webcam           | 8086:22b8      | Not tested     |
| Audio            |                | Yes            |
| Wireless          | 024c-b723      | Depends\*      |
| Modem (optional) | 12d1:15c1      | Not tested     |
| Bluetooth        |                | Depends\*      |
| TF/SIM reader    |                | Not tested     |
| TPM              |                | Yes            |
| Gyroscope        |                | Depends\*      |

\* It depends on the distribution, the kernel, and other factors. You can search below to see if it has a solution.

# Troubleshooting

## Ubuntu / Debian / Linux

### Gyroscope

The motion sensor or gyroscope works by default in various distributions/kernels. However, the sensor reports its orientation incorrectly to the operating system. To obtain the correct configuration for the motion sensor within Linux, you need to apply the following changes:

* Create file / Append to file: **/etc/udev/hwdb.d/60-sensor.hwdb** :

>
    sensor:modalias:*
      ACCEL_MOUNT_MATRIX=0, 1, 0; 1, 0, 0; 0, 0, 1

* Run `sudo systemd-hwdb update` to update hardware configuration.
* Restart your system.

If you still have orientation issues with your screen, you can use  `xrandr` manually in X.org to rotate the screen if you need to:
>
    xrandr -o right

Or:

>
    xrandr --output DSI-1 --rotate right

You can use at startup with your window manager startup configuration. Here is an example using Lightdm on Linux Mint 21:

* Create or edit: **/etc/lightdm/lightdm.conf.d/80-display-setup.conf**
* Append the startup script under **\[SeatDefaults]** like this:
>
    [SeatDefaults]
    # (This line is a comment, you can omit it) Another configs.
    display-setup-script=xrandr -o right

* Restart your system.

For troubleshooting or fixing screen orientation issues in GRUB or text mode, you can use the following solutions:

* Edit **/etc/default/grub** and append to **GRUB\_CMDLINE\_LINUX\_DEFAULT** one of the next launch options depending of what your needs are :

>
    quiet splash fbcon=rotate:1 video=efifbh

>

    video=DSI-1:800x1280e acpi_osi= i915.modeset=1 fbcon=rotate:1 video.use_native_backlight=1 i915.enable_fbc=1 i915.enable_rc6=1 i915.semaphores=1 nospalsh quiet

* Run `sudo update-grub` to update your GRUB config.
* Restart your system.

### Screen:

Sometimes, screen backlight might not work correctly. To fix the issue you can try this:

* Apend this to the initramfs modules file **/etc/initramfs-tools/modules**

>
    pwm_lpss_platform

* Update initramfs with `sudo update-initramfs -k all -u`
* Restart your system.

You can use this if you're having issues in text mode or when using GRUB

* Edit **/etc/default/grub** and append to **GRUB\_CMDLINE\_LINUX\_DEFAULT**:

>
    pwm-lpss pwm-lpss-platform

* Run `sudo update-grub` to update your GRUB config.
* Restart your system.

### GPU

If your screen freezes, becomes unresponsive or goes dark, you can try this (X.org only):

* Append to or create intel config file **/etc/X11/xorg.conf.d/20-intel-graphics.conf** with:

>
	Section "Module"
	    Load "dri3"
	EndSection

	Section "Device"
	    Identifier  "Intel Graphics"
	    Driver      "intel"
	    Option      "DRI"   "3"
	EndSection

* Restart your device.

After rebooting, GPU acceleration will exhibit improved performance, eliminating "screen tearing" and preventing device freezing. Additionally, the display manager will accurately reflect the device's native resolutions.

**Warning:**

Enabling DRI 3 on your graphics card will increase input latency in your 3D applications.

### Wireless:

* Devices that require specific kernel versions or additional drivers:

>
    rtl8723bs | Wifi and Bluetooth | Works under Kernel 5.15.0-75-generic

The blobs for this device became unavaliable when merged into new versions of the kernel, so for newer versions you should enable compile flags for this device if it doesn't work in your distro; This meaning, you probably need to recompile the kernel for your device. Be aware that if you have Secure Boot enabled, you need to sign everything you do.

### Sources

* https://forums.lenovo.com/t5/Ubuntu/Xubuntu-successfully-installed-on-Miix-310-10ICR-everything-but-camera-works/m-p/5208218
* https://gist.github.com/dotsh/7e647e2f4fec06d085a6902ea0f0a4cb
* https://wiki.archlinux.org/title/Lenovo\_IdeaPad\_Miix\_310-10ICR
* https://askubuntu.com/questions/1072645/screen-rotation-on-ubuntu-18-04-lts-lenovo-ideapad-miix-510
* https://community.intel.com/t5/Graphics/Difficulty-Enabling-DRI3-on-Debian-bullseye/td-p/1344602?profile.language=es\&countrylabel=Argentina
