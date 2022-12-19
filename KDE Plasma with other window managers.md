https://userbase.kde.org/Tutorials/Using_Other_Window_Managers_with_Plasma

# Tutorials/Using Other Window Managers with Plasma

## Contents

-   [1 Introduction](https://userbase.kde.org/Tutorials/Using_Other_Window_Managers_with_Plasma#Introduction)
-   [2 Using Another Window Manager with Plasma](https://userbase.kde.org/Tutorials/Using_Other_Window_Managers_with_Plasma#Using_Another_Window_Manager_with_Plasma)
    -   [2.1 Single User: Using System Settings](https://userbase.kde.org/Tutorials/Using_Other_Window_Managers_with_Plasma#Single_User:_Using_System_Settings)
    -   [2.2 System Wide: Adding an XSession](https://userbase.kde.org/Tutorials/Using_Other_Window_Managers_with_Plasma#System_Wide:_Adding_an_XSession)
-   [3 I3 configuration](https://userbase.kde.org/Tutorials/Using_Other_Window_Managers_with_Plasma#I3_configuration)
    -   [3.1 Installation](https://userbase.kde.org/Tutorials/Using_Other_Window_Managers_with_Plasma#Installation)
    -   [3.2 Configure i3](https://userbase.kde.org/Tutorials/Using_Other_Window_Managers_with_Plasma#Configure_i3)
    -   [3.3 Configure Plasma](https://userbase.kde.org/Tutorials/Using_Other_Window_Managers_with_Plasma#Configure_Plasma)
-   [4 Bspwm configuration](https://userbase.kde.org/Tutorials/Using_Other_Window_Managers_with_Plasma#Bspwm_configuration)
-   [5 Hints and Tips](https://userbase.kde.org/Tutorials/Using_Other_Window_Managers_with_Plasma#Hints_and_Tips)
    -   [5.1 DBus](https://userbase.kde.org/Tutorials/Using_Other_Window_Managers_with_Plasma#DBus)
-   [6 More Information](https://userbase.kde.org/Tutorials/Using_Other_Window_Managers_with_Plasma#More_Information)
-   [7 References](https://userbase.kde.org/Tutorials/Using_Other_Window_Managers_with_Plasma#References)

## Introduction

KWin is the default window manager (WM) in Plasma and has lot of features, but it only supports floating windows. Plasma lets you use another window manager, such as i3, bspwm or [any other tilling window manager](https://wiki.archlinux.org/index.php/Window_managers).

[![](https://userbase.kde.org/images.userbase/thumb/4/46/Plasma_i3.png/500px-Plasma_i3.png)](https://userbase.kde.org/File:Plasma_i3.png)

[](https://userbase.kde.org/File:Plasma_i3.png "Enlarge")

Plasma using i3 as window manager

![Warning.png](https://userbase.kde.org/images.userbase/thumb/c/cb/Warning.png/40px-Warning.png)

Warning

Other window managers are only available when using X.org. These changes cannot be made for Wayland sessions yet. With Wayland maturing and Wayland extensions being standardized, this feature could be implemented in Wayland[[1]](https://userbase.kde.org/Tutorials/Using_Other_Window_Managers_with_Plasma#cite_note-1)

  
Most window managers require few, if any, changes to their configuration files.

Some considerations to be aware of when using another window manager:

-   Tiling window managers may have problems with components like panels. For some configurations, certain options will help alleviate issues

-   Most window managers do not have their own compositors, and therefore lack features like animations and transparency. If such effects are desired, a compositor is required - refer to [Arch Wiki](https://wiki.archlinux.org/index.php/Xorg#Composite) for options.
    -   Compositors can also help with screen tearing, however check first if you can resolve it with your video drivers, as this will be a more performant solution in most cases.

## Using Another Window Manager with Plasma

### Single User: Using System Settings

This is the simplest method, but only affects the user for which it is added.

[![Kde-i3-integration.png](https://userbase.kde.org/images.userbase/1/1d/Kde-i3-integration.png)](https://userbase.kde.org/File:Kde-i3-integration.png)

-   Create a shell script (kde-i3.sh as an example), containing the following
    
    #!/bin/sh
    export KDEWM=/usr/bin/i3
    

Replace /usr/bin/i3 with the path to the window manager desired.

-   Start **System Settings**

-   Select **Startup and Shutdown**

-   Select **Autostart**

-   Click the **Add script** button below the list

-   Enter the location of the script created earlier, or use the folder icon to use a dialog to find it. Click **OK**, the script will be added to the list

-   In the **Run On** column, select **Before Session Startup** from the drop down list

-   The chosen window manager will be used the next time you log in

Note

In recent versions of Plasma the **Run On** and **Before Session Startup** options may not be available. You can obtain the same behavior by manually copying or symlinking the shell script above in `$HOME/.config/plasma-workspace/env`.

  

### System Wide: Adding an XSession

If superuser (root) access is available, a new session file for X.org can be created.

This has several advantages: the alternate session is available to all users of the system, and changing back to **KWin** is as simple as logging out and back in.

Note

The XSession file location varies between distributions, but is most of the time found in /usr/share/xsessions.

  
To add a session for an alternate window manager as superuser:

-   Copy the existing Plasma session file
    
    cp plasma.desktop plasma-i3.desktop
    

-   Using a text editor, open the file and change the Exec line, and optionally the Description
    
    [Desktop Entry]
    Type=XSession
    Exec=env KDEWM=/usr/bin/i3 /usr/bin/startplasma-x11
    DesktopNames=KDE
    Name=Plasma (i3)
    Comment=Plasma by KDE w/i3
    

![Warning.png](https://userbase.kde.org/images.userbase/thumb/c/cb/Warning.png/40px-Warning.png)

Warning

In versions prior to Plasma 5.17, use **startkde** in place of **startplasma-x11** in the Exec directive above.

  

-   The display manager must be restarted, most easily by rebooting

![Warning.png](https://userbase.kde.org/images.userbase/thumb/c/cb/Warning.png/40px-Warning.png)

Warning

Ensure the correct path and name of the window manager application are used when modifying the Exec directive. If KDE is unable to start the window manager, the session will fail and the user will be returned to the login screen.

  

![Warning.png](https://userbase.kde.org/images.userbase/thumb/c/cb/Warning.png/40px-Warning.png)

Warning

In some recent versions, SDDM checks for blank spaces in the `Exec` command. If blank spaces are found the Desktop Entry is not shown in the session menu. To workaround this, create a script named `plasma-i3.sh` containing the following:

#!/bin/sh
export KDEWM=/usr/bin/i3
/usr/bin/startplasma-x11

Set execution permissions for the script and reference it in **plasma-i3.desktop**, for example:

Exec=/usr/local/bin/plasma-i3.sh

  

## I3 configuration

### Installation

Before starting the configuration, you should make sure i3 or i3-gaps are installed on your system. If they are not, use your package manager or [Discover](https://userbase.kde.org/Special:MyLanguage/Discover "Special:MyLanguage/Discover") to install one of these two packages.

### Configure i3

When you first start i3, its default global configuration (at `/etc/i3/config`) will start i3-config-wizard, which will provide a dialog for Mod key selection and generate the user local configuration at `~/.i3/config`.

If you wish to maintain the same configuration across users (they can still override it with their own local config), remove the wizard invocation from the global configuration file and use it directly.

  
The following i3 window rules will help you with notifications and some other plasma windows:

for_window [title="Desktop — Plasma"] kill, floating enable, border none
for_window [class="plasmashell"] floating enable
for_window [class="Plasma"] floating enable, border none
for_window [title="plasma-desktop"] floating enable, border none
for_window [title="win7"] floating enable, border none
for_window [class="krunner"] floating enable, border none
for_window [class="Kmix"] floating enable, border none
for_window [class="Klipper"] floating enable, border none
for_window [class="Plasmoidviewer"] floating enable, border none
for_window [class="(?i)*nextcloud*"] floating disable
for_window [class="plasmashell" window_type="notification"] floating enable, border none, move right 700px, move down 450px
no_focus [class="plasmashell" window_type="notification"] 

If you are using a non-English installation of Plasma, you need to find out what the exact window title of the Desktop is. One way is to do this is with `wmctrl -l`.

Instead of adding

for_window [title="Desktop — Plasma"] kill; floating enable; border none

add the name of the window to the i3 config. This example uses the German Plasma installation.

for_window [title="Arbeitsfläche — Plasma"] kill; floating enable; border none

Notification positioning can be tricky, since not all notifications have the same size depending on their content. You can also use coordinates positioning, knowing your screen resolution. For example, for a 1920x1080 screen and notifications in the top-right corner we can use:

for_window [class="plasmashell" window_type="notification"] floating enable, border none, move position 1450px 20px

If you want to unlock KWallet automatically on login, configure PAM and then add the following line to your i3 configuration file:

exec --no-startup-id /usr/lib/pam_kwallet_init

### Configure Plasma

You may get into issues related with the "Activities" feature - activating it via keyboard shortcuts (some of which are also used by i3, like "Meta + Q") seems to make i3 unresponsive. Removing Activity related global shortcuts seems to resolve this.

To get the Plasma provided pager display desktops as i3bar, set **Pager Settings** > **General** > **Text display** to "Desktop name"

  

## Bspwm configuration

For the most part, bspwm requires little additional configuration.

-   A single Plasma panel, in most cases, is detected properly and bspwm will not place windows in its space. If the panel ends up presenting a problem, or when using multiple panels, the following may be added to **.bspwmrc**
    
    bspc config top_padding size
    
    where **size** is the size of the panel in pixels. Also valid are bottom_padding, left_padding, and right_padding. As many of these directives may be used as necessary for multiple panels.

## Hints and Tips

### DBus

Since Meta key handling is in part performed by KWin, you will have to handle these shortcuts manually when you switch to a different window manager.

You can open some Plasma components with DBus commands. You can use this to map corresponding keybindings to DBus commands. To find a specific DBus command, you can look at `dbus-monitor` or `qdbusviewer` while you invoke the component in a standard Plasma set-up.

More info on meta key handling:

-   [Reddit post describing the _Meta key - KWin_ relation](https://www.reddit.com/r/kde/comments/6wn5lk/bind_meta_key_alone_to_do_something/)

-   [Blog post describing the _DBus - Component_ relation](https://zren.github.io/2019/03/04/activate-any-widget-with-meta-key)

  
Some examples:

-   Open Application Launcher
    
    qdbus org.kde.plasmashell /PlasmaShell org.kde.PlasmaShell.activateLauncherMenu
    

-   Open Krunner
    
    qdbus org.kde.kglobalaccel /component/krunner org.kde.kglobalaccel.Component.invokeShortcut 'run command'
    

-   Open logout confirmation screen
    
    qdbus org.kde.ksmserver /KSMServer org.kde.KSMServerInterface.logout -1 -1 -1
    

For i3, you can map a keybinding with the **bindsym** command. Use `--no-startup-id` to prevent the command from generating "i3" entries in the task manager.

Example:

bindsym $mod+Shift+e exec --no-startup-id <command>

If you want to map the modifier key itself, use the **bindcode** command - you have to specify the keycode generated by the key, which you can get via the `xev` utility.

Example for the Meta key:

bindcode 133 --release exec --no-startup-id <command>

For Bspwn, you need to use the `bspc` program. See [Arch Linux wiki](https://wiki.archlinux.org/index.php/Bspwm#Keyboard) for more information.

## More Information

-   [Window managers](https://en.wikipedia.org/wiki/X_window_manager) (Wikipedia)
-   [Window managers](https://wiki.archlinux.org/index.php/Window_managers) (Arch Linux wiki)
-   [Desktop environment](https://en.wikipedia.org/wiki/Desktop_environment) (Wikipedia)
-   [Desktop environment](https://wiki.archlinux.org/index.php/Desktop_environment) (Arch Linux wiki)

## References

1.  [↑](https://userbase.kde.org/Tutorials/Using_Other_Window_Managers_with_Plasma#cite_ref-1 "Jump up") [https://drewdevault.com/2018/04/28/KDE-Sprint-retrospective.html](https://drewdevault.com/2018/04/28/KDE-Sprint-retrospective.html)

[Categories](https://userbase.kde.org/Special:Categories "Special:Categories"): 

-   [Tutorials](https://userbase.kde.org/Category:Tutorials "Category:Tutorials")
-   [Advanced Users](https://userbase.kde.org/Category:Advanced_Users "Category:Advanced Users")
-   [Plasma](https://userbase.kde.org/Category:Plasma "Category:Plasma")
-   [Desktop](https://userbase.kde.org/Category:Desktop "Category:Desktop")

  

This page was last edited on 21 November 2021, at 08:05. Content is available under [Creative Commons License SA 4.0](https://userbase.kde.org/KDE_UserBase_Wiki:Copyrights) unless otherwise noted.

### Donate to KDE [Why Donate?](https://kde.org/community/donations/index.php#money)

€Donate via PayPal

[Other ways to donate](https://kde.org/community/donations/others)

### Visit the KDE MetaStore

Show your love for KDE! Purchase books, mugs, apparel, and more to support KDE.

[Click here to browse](https://kde.org/stuff/metastore)

### About Wiki

[Privacy policy](https://userbase.kde.org/KDE_UserBase_Wiki:Privacy_policy "KDE UserBase Wiki:Privacy policy")[About KDE UserBase Wiki](https://userbase.kde.org/KDE_UserBase_Wiki:About "KDE UserBase Wiki:About")[Disclaimers](https://userbase.kde.org/KDE_UserBase_Wiki:General_disclaimer "KDE UserBase Wiki:General disclaimer")

### Products

[Plasma](https://kde.org/plasma-desktop)[KDE Applications](https://kde.org/applications/)[KDE Frameworks](https://kde.org/products/frameworks/)[Plasma Mobile](https://plasma-mobile.org/)[KDE neon](https://neon.kde.org/)[WikiToLearn](https://wikitolearn.org/)

### Develop

[TechBase Wiki](https://techbase.kde.org/)[API Documentation](https://api.kde.org/)[Qt Documentation](https://doc.qt.io/)[Inqlude Documentation](https://inqlude.org/)

### News & Press

[Announcements](https://kde.org/announcements/)[KDE.news](https://dot.kde.org/)[Planet KDE](https://planetkde.org/)[Screenshots](https://www.kde.org/screenshots)[Press Contact](https://www.kde.org/contact/)

### Resources

[Community Wiki](https://community.kde.org/Main_Page)[UserBase Wiki](https://userbase.kde.org/)[Miscellaneous Stuff](https://kde.org/stuff/)[Support](https://kde.org/support/)[International Websites](https://kde.org/support/international.php)[Download KDE Software](https://kde.org/download/)[Code of Conduct](https://kde.org/code-of-conduct/)

### Destinations

[KDE Store](https://store.kde.org/)[KDE e.V.](https://ev.kde.org/)[KDE Free Qt Foundation](https://www.kde.org/community/whatiskde/kdefreeqtfoundation.php)[KDE Timeline](https://timeline.kde.org/)

Maintained by [KDE Webmasters](mailto:kde-www@kde.org)KDE® and [the K Desktop Environment® logo](https://kde.org/media/images/trademark_kde_gear_black_logo.png) are registered trademarks of [KDE e.V.](https://ev.kde.org/ "Homepage of the KDE non-profit Organization") | [Legal](https://www.kde.org/community/whatiskde/impressum)
