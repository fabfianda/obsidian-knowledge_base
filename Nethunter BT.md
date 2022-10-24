ref: https://www.reddit.com/r/NetHunter/comments/u316qs/internal_bluetooth_radio_possible_as_hci0/i5ywgqa?utm_medium=android_app&utm_source=share&context=3

The use of the internal bluetooh radio has been worked out with the help of one of the Nethunter developers. Without their help this would have never been figured out! There is a great community on the gitlab page referenced below incase anyone else has issues. Below is just a quick summary of the steps taken. For a detailed discussion and a thread which provides some examples of the below with picutures see: [https://gitlab.com/kalilinux/nethunter/apps/kali-nethunter-app/-/issues/316](https://gitlab.com/kalilinux/nethunter/apps/kali-nethunter-app/-/issues/316)

Summary of all the steps completed to make the internal BT recognized as hci0:

1. Ensure you are using a kernel with VHCI, UART H4, BinderFS enabled.
2. Mount binderfs in Kali Chroot. Add the below code to the bootkali\_init under the SET SYSTEM section, then stop/start kali chroot in chroot manager in NH app.

\######### MOUNT BINDERFS ########

if \[ ! "$($BUSYBOX mountpoint $MNT/dev/binderfs 2> /dev/null | grep 'is a')" \]; then\[ -d $MNT/dev/binderfs \] && rm -rf $MNT/dev/binderfs\[ ! -d $MNT/dev/binderfs \] && mkdir -p $MNT/dev/binderfs$BUSYBOX mount -o bind /dev/binderfs $MNT/dev/binderfs && bklog "\[+\] mounted /dev/binderfs"

fi

3. Check for a successful link in Kali chroot using ls -al /dev

4. Then install bluebinder. Run this script in NH terminal [https://forum.xda-developers.com/attachments/bluebinder\_installer-zip.5365101/](https://forum.xda-developers.com/attachments/bluebinder_installer-zip.5365101/)

5. Turn off Bluetooth in android, then in kali terminal run:

`service dbus start`

`bluebinder`

Keep it running in that window, open another Kali terminal and run:

`hciconfig hci0 up`

Note: Can close this terminal window if you want.

Interface should now be available and ready to collect using Kismet. Open kismet and verify interface is available and collecting data. Note: Can make the changes to the kismet configuration files as indicated by yesimxev to have the wlan0 and hci0interfaces be automatically selected on Kismet startup.