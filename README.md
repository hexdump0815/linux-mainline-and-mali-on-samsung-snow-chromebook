some notes regarding the chromebook snow:

- active support from google for the snow chromebooks (samsung xe303c12) ended beginning of 2019, so they might be around for cheap
- they are very nice to use with linux: small, light (just a bit above 1kg), battery lasts for 4-6 hours at least
- what does not work with mainline:
  - usb3 port gives trouble with suspend, so i disabled it
  - webcam will not work (the usb bus it is connected to is somehow not seen in mainline)
  - 3g modem (if installed) will not work (situation similar to the webcam i guess, no idea if there even exists a driver for it)
  - full suspend/resume/hibernation do not work properly, but light suspend works perfectly (see below)
- what works: everything else, i.e. sound, wifi, external monitor, gles/opengl with mali and hopefully soon panfrost etc. ...
- all this was tested on a chromebook snow with xubuntu 18.04
- see readme.cbe for instructions to build the kernel from sources and patches (this is not a shell script)
- also ucm files (for rev4 snow - might need adjustment for snow rev5 chromebooks)
  cd / ; tar xzf snow-rev4-ucm.tar.gz
  alsaucm -c Snow-I2S-MAX98095
- make pulseaudio work:
  - /etc/asound..conf
  ```
  pcm.!default {
	type hw
	card 0
  }

  ctl.!default {
	type hw           
	card 0
  }
  ```
  - /etc/pulse/default.pa
  ```
  ...
  # local additions
  load-module module-alsa-sink device=sysdefault
  #load-module module-alsa-source device=sysdefault
  #load-module module-alsa-sink device=hw:0,0
  #load-module module-alsa-source device=hw:0,0  
  ```
- disable usb3 (does not work with sleep/wakeup) - done by disable-usb3.patch
  - set status = "disabled"; for usb3-vbus-en, usb_dwc3, phy@12100000 (samsung,exynos5250-usbdrd-phy) - leave in: ohci, ehci & usb2-phy
- only light sleep/wake as full does not work properly (will survive about a day with full battery)
  - put "echo s2idle > /sys/power/mem_sleep" into /etc/rc.local
- xfce -> settings -> power manager -> display -> brightness reduction
  - lower the display unused not lower than about 35% as at some point it is simply black (even above 0% already)
- enable all cortex a15 arm errata - done in the kernel config
  - otherwise mesa compile often gave internal compiler errors
- the wireless connection seems to drop from time to time
  - reloading the wifi module usually helps to bring it back (rmmod mwifiex_sdio mwifiex; modprobe mwifiex_sdio)
- mali support was taken from this tree https://github.com/mihailescu2m/linux/tree/odroidxu4-5.0.y
  - some commits not relevant were reverted (c8dee8578a8342bc33c48384e198aadd76ac72f6 699b5f261f2106b1314a293784b94a186c786b9b)
  - some changes required for v5.2 were added
  - the gpu node in the dts was taken from a suggestion of the panfrost irc
  - clock and iterrupt names were changed to match the suggested dts gpu node for panfrost (so that both can use the same dtb)
- on sd cards and emmc better create filesystems with journaling disabled (this should extend their life time)
  - mkfs -t ext4 -O ^has_journal -L somelabel /dev/somedevice
- better touchpad support - /etc/X11/xorg.conf.d/50-touchpad.conf:
```
Section "InputClass"
        Identifier              "touchpad"
        MatchIsTouchpad         "on"
        Option                  "FingerHigh"    "5"
        Option                  "FingerLow"     "5"
EndSection

Section "InputClass"
	Identifier "touchpad catchall"
	Driver "synaptics"
	MatchIsTouchpad "on"
	MatchDevicePath "/dev/input/event*"
	Option "TapButton1" "1"
	Option "TapButton2" "2"
	Option "TapButton3" "3"
	Option "EmulateThirdButton" "1"
	Option "EmulateThirdButtonTimeout" "750"
	Option "EmulateThirdButtonThreshold" "30"
EndSection
```

using gles or opengl with special rk3288 armsoc xorg server:
- cd / ; tar xzf armsoc-xorg-ubuntu-18.04.tar.gz (for ubuntu 18.04)
- cd / ; tar xzf opt-mali-exynos5250.tar.gz
- export LD_LIBRARY_PATH=/opt/mali-exynos5250
- start your gles or opengl app
- opengl is provided via included gl4es libGL.so.1 (https://github.com/ptitSeb/gl4es)
- use 01-armsoc.conf in /etc/X11/xorg.conf.d

using opengl with the default modesetting xorg server and gl4es hack (slower):
- cd / ; tar xzf opt-mali-exynos5250-fbdev-r5p0.tar.gz
- export LD_LIBRARY_PATH=/opt/mali-exynos-fbdev-r5p0
- export LIBGL_FB=3
- start your opengl app
- opengl is provided via included gl4es libGL.so.1 (https://github.com/ptitSeb/gl4es)
- use 01-modesetting.conf in /etc/X11/xorg.conf.d

different existing mali blob versions are provided: default is r12p0 - also provided r4p0, r5p0, r6p0

mainline u-boot:
- can be built following doc/README.chromium-chainload from https://gitlab.denx.de/u-boot/u-boot
- a prebuilt u-boot 2017.09 is provided
  - first partition the disk properly for chromeos - see misc.cbe/boot/cb.dd-single-part.readme
  - dd if=uboot.kpart.cbe-v2017.09 of=/dev/somedevicep1 (p1 is the kernel partition)

nicer bootup for the snow chromebook:
- see misc.cbe/boot/cb-gbb.txt

see the release section of this github repo for a precompiled kernel, modules etc.
