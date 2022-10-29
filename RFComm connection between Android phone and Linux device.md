ref: https://unix.stackexchange.com/posts/92256/timeline


Connect/turn on your bluetooth adapter on Linux. Ensure your adapter is visible (can be set in gnome-bluetooth -- you should see a bluetooth system tray icon).

Add a serial posrt:

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


Back on Android:

If hci0 not up

```
bluebinder
```

then

```
hciconfig hci0 up
```

Finally, if for example PC BT address is: D0:B1:28:94:2D:50 and Android BT address is: D0:B1:28:94:2D:50

```
rfcomm connect hci0 34:02:86:F9:B3:38 22
```

You should now get:

```
Connected /dev/rfcomm0 to 34:02:86:F9:B3:38 on channel 22
Press CTRL-C for hangup
```

