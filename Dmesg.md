# How to use the dmesg Command on Linux

-   Last Updated : 09 May, 2022

-   Read
-   Discuss

**dmesg** is a display message command that display kernel-related messages on Unix-like systems. It used to control the kernel ring buffer. The output contains messages produced by the device drivers.

### **Usage of dmesg command:**

All the messages received from the kernel ring buffer is displayed when we execute the command **“dmesg”**, here only the latest messages will be shown. This is used to check messages stored in the kernel ring buffer. In the case of non-root user use **“sudo”** to run root-level commands without being root.

$ sudo dmesg

![](https://media.geeksforgeeks.org/wp-content/uploads/20210621153817/Screenshot1398.jpg)

While we use the dmesg command it gives large output, we can use tail, head or less command to view the logs page wise. To search for a specific log or term of your choice with the help of forward slash **“/”** to search within _**less**_ 

$ sudo dmesg | less

![](https://media.geeksforgeeks.org/wp-content/uploads/20210621160148/Screenshot1399.jpg)

dmesg gives coloured output by default as shown above but if you want to colourize it for proper understanding of the messages then you can colourize them using the **“L” command.** 

$ sudo dmesg -L

![](https://media.geeksforgeeks.org/wp-content/uploads/20210621164959/Screenshot1402.jpg)

dmesg uses timestamps in seconds and nanoseconds, for human-friendly format use the **“H” option** for timestamps. The same is displayed in less. In the timestamp, it shows the date and time. Messages taking place every minute are marked as **seconds and nanoseconds.**

$ sudo dmesg -H

![](https://media.geeksforgeeks.org/wp-content/uploads/20210621164959/Screenshot1402.jpg)

We use **-T** (human-readable) option as they display with standard date and time. In this option, the resolution is lowered by a minute

$ sudo dmesg -T

![](https://media.geeksforgeeks.org/wp-content/uploads/20210622175923/Screenshot1403.jpg)

To monitor real-time logs **–follow** option is used with dmesg, and it displays the recent messages at the bottom of the terminal

$ sudo dmesg --follow

![](https://media.geeksforgeeks.org/wp-content/uploads/20210623173016/Screenshot1405.jpg)

As we see that dmesg gives out large output, we can use the tail or head option to list out a specific number of messages and view them. Here we check for the first 10 messages using the **head option**

$ sudo dmesg | head -10

![](https://media.geeksforgeeks.org/wp-content/uploads/20210623173444/Screenshot1417.jpg)

The last 10 messages are displayed using the **tail option**

$ sudo dmesg | tail -10

![](https://media.geeksforgeeks.org/wp-content/uploads/20210623173458/Screenshot1406.png)

Here we’ll use the **-i (ignore) option** to search for a specific string or patterns or message by scanning through the dmesg output but this option will ignore the case of the strings and will focus only on the string we search for. In the results, it will display messages for both **“USB” and “usb”  combinations**

$ sudo dmesg | grep -i usb

![](https://media.geeksforgeeks.org/wp-content/uploads/20210623174040/Screenshot1407.jpg)

Searching messaged related to Memory, RAM, Hard Disk or USB Drive using grep command with dmesg. Here grep is used with “sda” to check which hard disks have been detected by the kernel. **sda command** is used to check for hard disk and will display the messages wherever sda is listed

$ sudo dmesg | grep -i sda

![](https://media.geeksforgeeks.org/wp-content/uploads/20210623174256/Screenshot1408.png)

A level is assigned to each message logged to the kernel ring buffer. The level represents the significance of the information in the communication. The levels are as follows:

-   **emerg:** The system is unusable.
-   **alert:** Action must be taken immediately.
-   **crit:** Critical conditions.
-   **err:** Error conditions.
-   **warn:** Warning conditions.
-   **notice:** Normal but significant condition.
-   **info:** Informational.
-   **debug:** Debug-level messages.

Extracting messages using the **-l (level) option** with dmesg followed by the name of the level. Here we list all the informational messages using level **“info”** with the dmesg command they will display the notifications which are needed and important.

$ sudo dmesg -l info

![](https://media.geeksforgeeks.org/wp-content/uploads/20210623174645/Screenshot14091.jpg)

To extract messages with multiple log levels we have to combine two or more log levels. Here we are using **“debug and notice”** log levels to extract messages.

$ sudo dmesg -l debug,notice

![](https://media.geeksforgeeks.org/wp-content/uploads/20210623175621/Screenshot1410.jpg)

Another example of combining more than one log level is **“err and warn”** where it will display **error logs and warning logs**

$ sudo dmesg --level=err,warn

![](https://media.geeksforgeeks.org/wp-content/uploads/20210623180940/Screenshot1414.jpg)

Displaying dmesg messages for **eth0 user interface** use grep option followed by **eth0**

$ sudo dmesg | grep -i eth0

![](https://media.geeksforgeeks.org/wp-content/uploads/20210623181344/Screenshot1415.png)

Filtering dmesg messages using **facility option -f** where it will display messages of a particular facility **“daemon”.** 

$ sudo dmesg -f daemon

![](https://media.geeksforgeeks.org/wp-content/uploads/20210623180054/Screenshot1412min.jpg)

**-x (decode) option** is used to display the facility and level as prefixes of every line in a human-readable format.

$ sudo dmesg -x


Related Articles

1.[dmesg command in Linux for driver messages](https://www.geeksforgeeks.org/dmesg-command-linux-driver-messages/?ref=rp "Permalink to dmesg command in Linux for driver messages")

2.[Ccat – Colorize Cat Command Output command in Linux with Examples](https://www.geeksforgeeks.org/ccat-colorize-cat-command-output-command-in-linux-with-examples/?ref=rp "Permalink to Ccat – Colorize Cat Command Output command in Linux with Examples")

3.[How to use aplay and spd-say command in Linux?](https://www.geeksforgeeks.org/how-to-use-aplay-and-spd-say-command-in-linux/?ref=rp "Permalink to How to use aplay and spd-say command in Linux?")

4.[How to Install and Use Sysdig from Linux Command Line?](https://www.geeksforgeeks.org/how-to-install-and-use-sysdig-from-linux-command-line/?ref=rp "Permalink to How to Install and Use Sysdig from Linux Command Line?")

5.[Bash Script - How to use Command Line Arguments](https://www.geeksforgeeks.org/bash-script-how-to-use-command-line-arguments/?ref=rp "Permalink to Bash Script - How to use Command Line Arguments")

6.[How to find time taken by a command/program on Linux Shell?](https://www.geeksforgeeks.org/how-to-find-time-taken-by-a-program-on-linux-shell/?ref=rp "Permalink to How to find time taken by a command/program on Linux Shell?")

7.[mindepth and maxdepth in Linux find() command for limiting search to a specific directory.](https://www.geeksforgeeks.org/mindepth-maxdepth-linux-find-command-limiting-search-specific-directory/?ref=rp "Permalink to mindepth and maxdepth in Linux find() command for limiting search to a specific directory.")

8.[Netstat command in Linux](https://www.geeksforgeeks.org/netstat-command-linux/?ref=rp "Permalink to Netstat command in Linux")

9.[Practical applications of 'ls' command in Linux](https://www.geeksforgeeks.org/practical-applications-ls-command-linux/?ref=rp "Permalink to Practical applications of 'ls' command in Linux")

10.['dd' command in Linux](https://www.geeksforgeeks.org/dd-command-linux/?ref=rp "Permalink to 'dd' command in Linux")
