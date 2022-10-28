ref: https://unix.stackexchange.com/posts/92256/timeline

I seem to have gotten this working now. Bluetooth seems a bit finicky. I'm recapping my steps in full in case someone else finds it useful (though its pretty much what I tried initially). This is for Android JB (4.2.2) on a Nexus 4 and Arch Linux 3.6.7-1, with bluez 4.101 on Gnome 3.6 (w/ gnome-bluetooth).

_(this step may not do anything useful)_ Turn Bluetooth on Android off and disconnect your USB/Bluetooth Adapter from your Linux machine (or if you have an in built one, reset it using hcitool devname reset)

Connect/turn on your bluetooth adapter on Linux. Ensure your adapter is visible (can be set in gnome-bluetooth -- you should see a bluetooth system tray icon).

Turn on bluetooth on your Android device. Use Android to pair to the adapter (I was unable to pair the other way around from Linux). A dialog will come up asking you for a key. Put in any PIN you want. Gnome should pop up a notification asking you for a key; put in the same PIN you entered earlier. Your Android device and the key should be paired at this point.

In Linux, open up a terminal and check what bluetooth services are available by typing in

```
sdptool browse local
```

If you already have a serial port service, make a note of what channel it is. If you don't, you can add the service:

```
sdptool add --channel=22 SP
```

Now listen on this channel using rfcomm:

```
sudo rfcomm listen /dev/rfcomm0 22
```

rfcomm will block, listening for a connection with a message like

```
Waiting for connection on channel 22
```

Back on Android, I used the BlueTerm application ([http://pymasde.es/blueterm/](http://pymasde.es/blueterm/), also available freely on the google play store) though any similar application should work. Open up BlueTerm, go to options > Connect Device: select the paired adapter.

Hopefully, the application was able to connect. You'll see additional verification in the terminal where you blocked listening with a message like:

```
Waiting for connection on channel 22
Connection from 22:22:22:22:22:22 to /dev/rfcomm0
Press CTRL-C for hangup
```

Anything you type into the BlueTerm app should be going to /dev/rfcomm0. You can see stuff show up as you type by opening up a new terminal and doing something like:

```
cat /dev/rfcomm0
```

