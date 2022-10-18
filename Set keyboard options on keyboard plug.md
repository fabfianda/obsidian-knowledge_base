Use `lsusb` to discover the Hardware ID that you need to set in the udev rule. My das keyboard looks like this `Bus 002 Device 009: ID 04d9:2013 Holtek Semiconductor, Inc.`

I first set up the udev rule to autodetct the keyboard is by creating a udev rule:

In the file `/etc/udev/rules.d/00-usb-keyboards.rules`:

```
ACTION=="add", ATTRS{idVendor}=="04d9", ATTRS{idProduct}=="2013", RUN+="/home/rizumu/bin/kbd_udev", OWNER="rizumu"
```


Depending on your distro, you may already have a udev rule for keyboards in /lib/udev/rules.d/64-xorg-xkb.rules. On Ubuntu, this imports /etc/default/keyboard, which has options roughly like this:

`XKBMODEL="pc105"   XKBLAYOUT="us"   XKBVARIANT=""   XKBOPTIONS=""`

For my setup, I found that this built-in rule was executing after my custom udev rule, and was overriding my settings. Instead I changed XKBOPTIONS in /etc/default/keyboard to be:

`XKBOPTIONS="ctrl:nocaps"`

To get the "Caps Lock is Control" behavior I wanted on all keyboards.