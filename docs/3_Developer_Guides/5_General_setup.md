---
title: Generic devices setup
---

## Camera setup on RPi4

This assumes that you'll be using the Raspberry Pi Camera Module, which can
easily hooked on the board using the LVDS connector. Further more information
about how to connect it, could be found at
[Installing a Raspberry Pi camera](https://www.raspberrypi.com/documentation/accessories/camera.html#installing-a-raspberry-pi-camera)

With the camera installed, you'll need to enable it by editing /boot/config.txt
file (if you're booting an AGL image over a sd-card it will be there by
default, otherwise -- in case of doing netboot, you'll have to create
manually) and add the following entry:


	## start_x
	##     Set to "1" to enable the camera module.
	##
	##     Enabling the camera requires gpu_mem option to be specified with a value
	##     of at least 128.
	##
	##     Default 0
	##
	start_x=1

And reboot your device. Afterwards, after logging it, make sure that you have
the /dev/video0 device created. You could also use v4l2-ctl to verify
that is indeed usable as a capture device:

	$ v4l2-ctl -d /dev/video0 --all

In order to test out video playback, use the following gstreamer pipeline:

	$ gst-launch-1.0 v4l2src device=/dev/video0 ! \
						video/x-raw,width=640,height=480 ! waylandsink fullscreen=true

Alternatively, using [camera-gstreamer](https://gerrit.automotivelinux.org/gerrit/gitweb?p=apps/camera-gstreamer.git;a=summary)
application could might be another possibility, but you'll have to build it
yourself and add it in the image.

This includes an example on how to create a xdg-toplevel surface and create a
gstreamer pipeline, instead of relaying on waylandsink to create one for you,
in a programmatic fashion.

## Display setup on RPi4

This assumes that you'll be using the Raspberry Pi 7'' display. Installation
can be found at [Rpi Display page](https://www.raspberrypi.com/documentation/accessories/display.html).

If booting over the network, the dtb should already contain the ft5406 dtb,
while booting over sd-card the following show be in the /boot/config.txt
file:

	dtoverlay=rpi-ft5406-overlay

Once the board boots up, you should get a rainbow like effect and afterwards
booting up would display debug scrolling over. You'll need to adjust the
orientation as the 800x480 is in portrait.  Edit /etc/xdg/weston/weston.ini and
add a new output entry, like the following:

	[output]
	name=DSI-1
	transform=90

Note that for the Qt platform, the homescreen application, together with the
other demo applications, is tailored specifically for a 1080p display and it
will display incorrectly on such a smaller display.

## Testing out video camera without a real device

While the above requires having a real video camera device, one can use out
the [vivid module](https://docs.kernel.org/admin-guide/media/vivid.html?highlight=vivid#the-virtual-video-test-driver-vivid)
to try out your custom application or just testing out camera functionality in AGL.

You should normally have the module present, not loaded, for either **rpi4** or for
**h3ulcb** boards. Load the module, like in the following command:

	modprobe vivid allocators=0x1

Have a look at the kernel ring buffer to see what capture devices are created
and use as source:

	$ gst-launch-1.0 v4l2src device=/dev/videoXX ! \
				video/x-raw,width=640,height=480 ! waylandsink fullscreen=true

making sure to replace /dev/videoXX with correct capture device. You can check
that easily using the v4l2-ctl mentioned above.

Note that using camera-gstreamer will attempt to find the first available
capture device, so if you happen to have to another one before the one created
by this module it might not work. You can bypass that, by just making a symlink
from /dev/videoXX to /dev/video0 to make first available capture device.
