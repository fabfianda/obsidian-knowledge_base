• At your grub menu press "e" to edit boot options.
• You'll see a simple text edtor.  Find the line that starts with "linux" and at the end of that line add a space and then ```
init=/bin/bash.```

• Hit F10 to exit the editor.  The machine will boot.
• Remount your root filesystem read/write like this:
```mount -o remount,rw /```
• Change root's password by running ```
passwd root