[Skip to content](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#main)

[![dazeb](https://mlufdnrmzupp.i.optimole.com/w:512/h:512/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2022/12/dazebuk-tech-icon-1.webp)](https://dazeb.uk/)

[dazeb](https://dazeb.uk/)

# Proxmox VM GPU Hardware Acceleration for Jellyfin/Plex/Emby

An ultimate guide to GPU PT for hardware acceleration in virtual machines on a Proxmox host.

-   JAN 9, 2023
-   JANUARY 13, 2023
-   [JELLYFIN](https://dazeb.uk/category/jellyfin/)[PLEX](https://dazeb.uk/category/plex/)[PROXMOX](https://dazeb.uk/category/virtualization/proxmox/)[VIRTUALIZATION](https://dazeb.uk/category/virtualization/)

[HOME](https://dazeb.uk/)[VIRTUALIZATION](https://dazeb.uk/category/virtualization/)PROXMOX VM GPU HARDWARE ACCELERATION FOR JELLYFIN/PLEX/EMBY

Table of Contents

-   [Intro](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#0-intro)
-   [Enable BIOS Features](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#1-enable-bios-features)
-   [Enable IOMMU in GRUB](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#2-enable-iommu-in-grub)
-   [Proxmox Host Configuration](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#3-proxmox-host-configuration)
    -   [Getting GPU Device IDs](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#4-getting-gpu-device-ids)
    -   [Isolate GPU](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#5-isolate-gpu)
    -   [Driver Blacklisting](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#6-driver-blacklisting)
    -   [Enable VFIO Modules](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#7-enable-vfio-modules)
    -   [Dumping the GPU vBIOS](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#8-dumping-the-gpu-vbios)
        -   [A note on Windows 7/10/11](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#9-a-note-on-windows-71011)
-   [VM Creation](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#10-vm-creation)
    -   [VM Creation – General](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#11-vm-creation-general)
    -   [VM Creation – OS](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#12-vm-creation-os)
    -   [VM Creation – System](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#13-vm-creation-system)
    -   [VM Creation – Disks](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#14-vm-creation-disks)
    -   [VM Creation – CPU](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#15-vm-creation-cpu)
    -   [VM Creation – Memory](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#16-vm-creation-memory)
    -   [VM Creation – Network](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#17-vm-creation-network)
    -   [VM Creation – Confirm](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#18-vm-creation-confirm)
-   [Ubuntu Server OS Installation](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#19-ubuntu-server-os-installation)
-   [Ubuntu VM Configuration](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#20-ubuntu-vm-configuration)
    -   [Dump GPU BIOS to KVM Directory](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#21-dump-gpu-bios-to-kvm-directory)
    -   [Adding the GPU to the VM](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#22-adding-the-gpu-to-the-vm)
    -   [Booting the VM & Installing Drivers](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#23-booting-the-vm-amp-installing-drivers-)
    -   [Install Nvidia Drivers Method #1 (Manually)](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#24-install-nvidia-drivers-method-1-manually)
    -   [Driver Install Method #2 (Recommended)](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#25-driver-install-method-2-recommended)
-   [Jellyfin Install](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#26-jellyfin-install)
-   [Transcode Testing with Jellyfin](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#27-transcode-testing-with-jellyfin)
-   [Patch the Driver for More Transcode Streams](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#28-patch-the-driver-for-more-transcode-streams)
-   [Install Jellyseerr](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#29-install-jellyseerr)
    -   [Install Jellyseerr – Portainer](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#30-install-jellyseerr-portainer)
    -   [Install Jellyseer – Terminal Commands](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#31-install-jellyseer-terminal-commands)
-   [References](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#32-references-)

## Intro

GPU Passthrough in virtualized environments is nothing new but as new operating systems evolve the methods of installing and setting up these systems change. I recently had to set up hardware acceleration on an Ubuntu 22.04 Server virtual machine using an Nvidia 960M GPU and there is still so much outdated information so this guide aims to be more relevant to 2023. This guide will work with **Ubuntu 20.04/22.04 LTS** and probably Debian 11.

-   **Host OS**: Proxmox 7.1 – Debian 11 based
-   **VM OS**: Ubuntu 22.04 LTS

The guide will cover everything from the Proxmox configuration to the final virtual machine test for Hardware Acceleration using [Jellyfin Media Server](https://jellyfin.org/). We will also set up driver patches to allow us to bypass Nvidia’ transcode limits and install a movie and tv show request service for Sonarr and Radarr called [Jellyseer](https://github.com/Fallenbagel/jellyseerr).

## Enable BIOS Features

In your PC/Laptop/Server BIOS make sure the following is enabled in the BIOS:

-   Intel VT-d & VT-x – [Intel Compatible list](https://ark.intel.com/content/www/us/en/ark/search/featurefilter.html?productType=873&0_VTD=True)
-   All AMD CPUs from Bulldozer onwards should be compatible.
-   [List of IOMMU Compatible CPUs](https://en.wikipedia.org/wiki/List_of_IOMMU-supporting_hardware)
-   The GPU BIOS must be UEFI compatible – [Wiki Compatibility list](https://www.techpowerup.com/vgabios/)

## Enable IOMMU in GRUB

Boot your Proxmox host with the BIOS features enabled then we will edit our `grub` file to activate the features on every boot.

Open `/etc/default/grub` and add a new line. I prefer just to comment out what is there so that I can always go back to exactly how it was initially if I need to.

```bash
nano /etc/default/grub
```

![](https://mlufdnrmzupp.i.optimole.com/w:602/h:288/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2023/01/change-grub-allow-pci.png)

**If you have an Intel CPU add the following line**:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
```

**If you have AMD CPU add the following line**:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on"
```

Once done do a `CTRL` + `x` to exit then `y` to save.

Update grub and reboot

```bash
update-grub
reboot
```

Once booted, If you have all this set up we can use a few commands to check IOMMU is working

```bash
dmesg | grep -e DMAR -e IOMMU
```

You should see something like the below. It will likely not be the same, all you are looking for is the `IOMMU enabled` part.

![](https://mlufdnrmzupp.i.optimole.com/w:585/h:212/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2023/01/iommu-enabled-zoom-jpg.webp)

## Proxmox Host Configuration

Here is where we start to set up the Proxmox host for GPU Passthrough. First, we need our device ID.

### Getting GPU Device IDs

The command below will give you your device ID, you may have one or two devices. One will be your GPU and one will be the sound device attached to the GPU.

```bash
lspci -n -s 01:00
```

The output will look something like this

```bash
01:00.0 0300: 10de:1381 (rev a2)              # <------------ Main GPU
01:00.1 0403: 10de:0fbc (rev a1)              # <------------ GPU Sound Card
```

The part we need is `10de:1381` and `10de:0fbc`. These are our _GPU Device ID_ if you ever need to reference them again just use the `lspci -n -s 01:00` command.

### Isolate GPU

While we have our ID we can create a `vfio.conf` file that will tell Proxmox to isolate that specific device on boot. Also, because we will be using OMVF bios in our VM instead of SeaBIOS we will be able to boot using UEFI mode instead of CSM.

```bash
echo "options vfio-pci ids=10de:1381,10de:0fbc disable_vga=1" > /etc/modprobe.d/vfio.conf
```

### Driver Blacklisting

Now we have isolated the GPU we need to blacklist the drivers also so that Proxmox doesn’t try to boot any drivers as they will conflict with our passthrough.

```bash
echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf 
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf 
echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf 
```

After running these commands we need to enable the VFIO modules in Proxmox

### Enable VFIO Modules

Edit the `/etc/modules` file

```bash
nano /etc/modules
```

Add the following to the file

```bash
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

Use `CTRL` + `x` to save then `y` to exit.

Finally use `update-initramfs -u` and `reboot`

```bash
update-initramfs -u
reboot
```

After rebooted we can check to see if the VFIO drivers are running with

```bash
dmesg | grep -i vfio
```

You should get some output like

```bash
[    4.254246] VFIO - User Level meta-driver version: 0.3
[    4.275657] vfio_pci: add [10de:139b[ffffffff:ffffffff]] class 0x000000/00000000
[   62.519947] vfio-pci 0000:01:00.0: vfio_ecap_init: hiding ecap 0x1e@0x258
[   62.519959] vfio-pci 0000:01:00.0: vfio_ecap_init: hiding ecap 0x19@0x900
```

We can also check with `lspci` to see the GPU and which Kernel driver it is using currently.

```bash
lspci -nnk -d 10de:139b
```

The output should be something similar to the below

![](https://mlufdnrmzupp.i.optimole.com/w:781/h:96/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2023/01/gpu-kernel-driver.png)

### Dumping the GPU vBIOS

Dumping the GPU is pretty straightforward and gives us a couple of things. It allows us to test it and see if it is UEFI compatible and we can also use the dumped rom file to pass through to our VM to further lock the GPU to the virtual machine.

For now we will dump and test the GPU vbios file. Start by downloading the software and the dependencies

```bash
apt update
apt install gcc git build-essential -y
git clone https://github.com/awilliam/rom-parser
cd rom-parser
make
```

Now to dump the vBIOS. The vbios will be dumped to `/tmp/image.rom`.

```bash
cd /sys/bus/pci/devices/0000:01:00.0/
echo 1 > rom
cat rom > /tmp/image.rom
echo 0 > rom
```

Now we can test the vbios rom with

```bash
./rom-parser /tmp/image.rom
```

The output is something similar to this

```bash
Valid ROM signature found @0h, PCIR offset 190h
 PCIR: type 0, vendor: 10de, device: 1280, class: 030000
 PCIR: revision 0, vendor revision: 1
Valid ROM signature found @f400h, PCIR offset 1ch
 PCIR: type 3, vendor: 10de, device: 1280, class: 030000
 PCIR: revision 3, vendor revision: 0
  EFI: Signature Valid
 Last image
```

**To be UEFI compatible, you need a “type 3” in the result.**

You can log into your Proxmox machine and download the file if you want or you can save it to another location like a home folder. You may need it in future. As long as you got some output then the vbios is valid and UEFI capable.

#### A note on Windows 7/10/11

> This guide is not designed for Windows VM’s but if you absolutely must use Windows and you have a Nvidia GPU you need to use the commands below to avoid crashes with geforce experience.
> 
> https://pve.proxmox.com/wiki/Pci_passthrough#NVIDIA_Tips

```bash
echo "options kvm ignore_msrs=1" > /etc/modprobe.d/kvm.conf
```

If you get a lot of errors in `dmesg` you can add the following line instead

```bash
echo "options kvm ignore_msrs=1 report_ignored_msrs=0" > /etc/modprobe.d/kvm.conf
```

## VM Creation

I will be using Ubuntu 22.04.1 Server and we need to be quite specific about the options we enable and in which order we enable them because as soon as we attach our GPU we will not be able to log in through the Proxmox GUI.

Start with a Virtual Machine with the following settings (images below)

```bash
# VM Creation Tab 1
Name: ubuntuvm
# VM Creation Tab 2
Operating System: Ubuntu 22.04.1 Live Server
Linux Kernel: 5.x - 2.6
# VM Creation Tab 3
Graphic Card: Default
Machine: q35
BIOS: OVMF (UEFI)
Add EFI Disk: YES
Pre-Enroll keys: YES
SCSI Controller: VirtiO SCSI
# VM Creation Tab 4
SSD Emulation: YES  # Use if installing to SSD or NVME
Discard: YES        # Use if installing to SSD or NVME
# VM Creation Tab 5
Cores: 4
Type: host
# VM Creation Tab 6
Memory: 4096MB
Ballooning Device: Disable because ballooning affects performance.
# VM Creation Tab 7
Bridge: vmbr0
Network: VirtiO (paravirtualized)
# VM Creation Tab 8
Start after created: YES
```

### VM Creation – General

![](https://mlufdnrmzupp.i.optimole.com/w:717/h:508/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2023/01/tab1.png)

### VM Creation – OS

![](https://mlufdnrmzupp.i.optimole.com/w:717/h:508/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2023/01/ubuntu-intall.png)

### VM Creation – System

![](https://mlufdnrmzupp.i.optimole.com/w:716/h:507/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2023/01/tab3.png)

### VM Creation – Disks

`Discard` and `SSD Emulation` should be enabled if using an SSD or NVME

![](https://mlufdnrmzupp.i.optimole.com/w:716/h:508/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2023/01/disks.png)

### VM Creation – CPU

![](https://mlufdnrmzupp.i.optimole.com/w:719/h:510/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2023/01/cpu.png)

### VM Creation – Memory

Give as much memory as you can allow, usually, around 4GB is the minimum. Ballooning affects performance so we disable it. It allows for the ram to grow as needed but we don’t want that for stability reasons.

![](https://mlufdnrmzupp.i.optimole.com/w:719/h:510/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2023/01/memory.png)

### VM Creation – Network

![](https://mlufdnrmzupp.i.optimole.com/w:717/h:510/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2023/01/network.png)

### VM Creation – Confirm

![](https://mlufdnrmzupp.i.optimole.com/w:720/h:510/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2023/01/confirm.png)

## Ubuntu Server OS Installation

The server OS is no different from any other server install. Follow the prompts and reboot as necessary until your system is fully up and running.

## Ubuntu VM Configuration

First let’s make sure we have some basic apps, services and updates in our virtual machine. Use the following commands. Some of it may already be installed but what we need to make sure is that we can get an ssh connection because it will be our only method of accessing the virtual machine.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install git build-essential gcc openssh-server
```

In another computer, test to see if you can get an SSH connection before we start adding the GPU to the VM.

```bash
ssh user@<VM IP>
```

If you get a successful connection your good we can move on to the next step.

### Dump GPU BIOS to KVM Directory

We need to use the same method as we used earlier to dump the bios but we will put the bios in the `/usr/share/kvm/` directory

```bash
cd /sys/bus/pci/devices/0000:01:00.0/
echo 1 > rom
cat rom > /usr/share/kvm/vbios.bin
echo 0 > rom
```

We can now use the bios when we add the GPU to the VM.

### Adding the GPU to the VM

First shutdown the VM if it is online.

```bash
shutdown now
```

Edit the VM .conf file `/etc/pve/qemu-server/<VMID>.conf`

```bash
nano /etc/pve/qemu-server/<VMID>.conf        # < Replace <VMID> with the VM ID NUMBER
```

The reason why we are adding the GPU this way is because we are passing through our GPU bios which cannot be set via the Proxmox GUI.

Add the GPU by adding the line below. Add it just after the disks

```bash
hostpci0: 0000:01:00,romfile=vbios.bin,x-vga=1
```

`CTRL` + `x` then `y` to save and exit.

The GUI will look like below and you can see our vbios.bin GPU bios is loaded.

![](https://mlufdnrmzupp.i.optimole.com/w:598/h:285/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2023/01/vbios.png)

### Booting the VM & Installing Drivers

Boot the virtual machine and see if the GPU is recognised with

```bash
lspci
```

You should see your card in the list of PCI devices

![](https://mlufdnrmzupp.i.optimole.com/w:926/h:464/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2023/01/lspci.png)

### Install Nvidia Drivers Method #1 (Manually)

We can use a few ways of doing this but the final result is always pretty much the same, some methods work better than others for some and vice versa.

We can use `apt` to search for a recommended driver. We won’t be using apt to install the driver but it’s good to know what is recommended.

```bash
apt search nvidia-driver
```

We get back the available drivers as a list which tells us which driver is recommended for the system.

```bash
Full Text Search... Done
nvidia-384/jammy-updates,jammy-security 390.157-0ubuntu0.22.04.1 amd64
  Transitional package for nvidia-driver-390

nvidia-384-dev/jammy-updates,jammy-security 390.157-0ubuntu0.22.04.1 amd64
  Transitional package for nvidia-driver-390

nvidia-driver-390/jammy-updates,jammy-security 390.157-0ubuntu0.22.04.1 amd64
  NVIDIA driver metapackage

nvidia-driver-418/jammy 430.50-0ubuntu3 amd64
  Transitional package for nvidia-driver-430

nvidia-driver-418-server/jammy 418.226.00-0ubuntu4 amd64
  NVIDIA Server Driver metapackage

nvidia-driver-430/jammy 440.100-0ubuntu1 amd64
  Transitional package for nvidia-driver-440

nvidia-driver-435/jammy 455.45.01-0ubuntu1 amd64
  Transitional package for nvidia-driver-455

nvidia-driver-440/jammy 450.119.03-0ubuntu1 amd64
  Transitional package for nvidia-driver-450

nvidia-driver-440-server/jammy-updates,jammy-security 450.216.04-0ubuntu0.22.04.1 amd64
  Transitional package for nvidia-driver-450-server

nvidia-driver-450/jammy 460.91.03-0ubuntu1 amd64
  Transitional package for nvidia-driver-460

nvidia-driver-450-server/jammy-updates,jammy-security 450.216.04-0ubuntu0.22.04.1 amd64
  NVIDIA Server Driver metapackage

nvidia-driver-455/jammy 460.91.03-0ubuntu1 amd64
  Transitional package for nvidia-driver-460

nvidia-driver-460/jammy-updates,jammy-security 470.161.03-0ubuntu0.22.04.1 amd64
  Transitional package for nvidia-driver-470

nvidia-driver-460-server/jammy-updates,jammy-security 470.161.03-0ubuntu0.22.04.1 amd64
  Transitional package for nvidia-driver-470-server

nvidia-driver-465/jammy-updates,jammy-security 470.161.03-0ubuntu0.22.04.1 amd64
  Transitional package for nvidia-driver-470

nvidia-driver-470/jammy-updates,jammy-security 470.161.03-0ubuntu0.22.04.1 amd64
  NVIDIA driver metapackage

nvidia-driver-470-server/jammy-updates,jammy-security 470.161.03-0ubuntu0.22.04.1 amd64
  NVIDIA Server Driver metapackage

nvidia-driver-495/jammy-updates,jammy-security 510.108.03-0ubuntu0.22.04.1 amd64
  Transitional package for nvidia-driver-510

nvidia-driver-510/jammy-updates,jammy-security 510.108.03-0ubuntu0.22.04.1 amd64
  NVIDIA driver metapackage

nvidia-driver-510-server/jammy-updates,jammy-security 515.86.01-0ubuntu0.22.04.1 amd64
  Transitional package for nvidia-driver-515-server

nvidia-driver-515/jammy-updates,jammy-security 515.86.01-0ubuntu0.22.04.1 amd64
  NVIDIA driver metapackage

nvidia-driver-515-open/jammy-updates,jammy-security 515.86.01-0ubuntu0.22.04.1 amd64
  NVIDIA driver (open kernel) metapackage

nvidia-driver-515-server/jammy-updates,jammy-security 515.86.01-0ubuntu0.22.04.1 amd64
  NVIDIA Server Driver metapackage

nvidia-driver-520/jammy-updates,jammy-security 525.60.11-0ubuntu0.22.04.1 amd64
  Transitional package for nvidia-driver-525

nvidia-driver-520-open/jammy-updates,jammy-security 525.60.11-0ubuntu0.22.04.1 amd64
  Transitional package for nvidia-driver-525

nvidia-driver-525/jammy-updates,jammy-security 525.60.11-0ubuntu0.22.04.1 amd64
  NVIDIA driver metapackage

nvidia-driver-525-open/jammy-updates,jammy-security 525.60.11-0ubuntu0.22.04.1 amd64
  NVIDIA driver (open kernel) metapackage

nvidia-driver-525-server/jammy-updates,jammy-security 525.60.13-0ubuntu0.22.04.1 amd64
  NVIDIA Server Driver metapackage

nvidia-headless-390/jammy-updates,jammy-security 390.157-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage

nvidia-headless-418-server/jammy 418.226.00-0ubuntu4 amd64
  NVIDIA headless metapackage

nvidia-headless-450-server/jammy-updates,jammy-security 450.216.04-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage

nvidia-headless-470/jammy-updates,jammy-security 470.161.03-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage

nvidia-headless-470-server/jammy-updates,jammy-security 470.161.03-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage

nvidia-headless-510/jammy-updates,jammy-security 510.108.03-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage

nvidia-headless-515/jammy-updates,jammy-security 515.86.01-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage

nvidia-headless-515-open/jammy-updates,jammy-security 515.86.01-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage (open kernel module)

nvidia-headless-515-server/jammy-updates,jammy-security 515.86.01-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage

nvidia-headless-525/jammy-updates,jammy-security 525.60.11-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage

nvidia-headless-525-open/jammy-updates,jammy-security 525.60.11-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage (open kernel module)

nvidia-headless-525-server/jammy-updates,jammy-security 525.60.13-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage

nvidia-headless-no-dkms-390/jammy-updates,jammy-security 390.157-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage - no DKMS

nvidia-headless-no-dkms-418-server/jammy 418.226.00-0ubuntu4 amd64
  NVIDIA headless metapackage - no DKMS

nvidia-headless-no-dkms-450-server/jammy-updates,jammy-security 450.216.04-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage - no DKMS

nvidia-headless-no-dkms-470/jammy-updates,jammy-security 470.161.03-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage - no DKMS

nvidia-headless-no-dkms-470-server/jammy-updates,jammy-security 470.161.03-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage - no DKMS

nvidia-headless-no-dkms-510/jammy-updates,jammy-security 510.108.03-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage - no DKMS

nvidia-headless-no-dkms-515/jammy-updates,jammy-security 515.86.01-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage - no DKMS

nvidia-headless-no-dkms-515-open/jammy-updates,jammy-security 515.86.01-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage - no DKMS (open kernel module)

nvidia-headless-no-dkms-515-server/jammy-updates,jammy-security 515.86.01-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage - no DKMS

nvidia-headless-no-dkms-525/jammy-updates,jammy-security 525.60.11-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage - no DKMS

nvidia-headless-no-dkms-525-open/jammy-updates,jammy-security 525.60.11-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage - no DKMS (open kernel module)

nvidia-headless-no-dkms-525-server/jammy-updates,jammy-security 525.60.13-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage - no DKMS

xserver-xorg-video-nvidia-390/jammy-updates,jammy-security 390.157-0ubuntu0.22.04.1 amd64
  NVIDIA binary Xorg driver

xserver-xorg-video-nvidia-418-server/jammy 418.226.00-0ubuntu4 amd64
  NVIDIA binary Xorg driver

xserver-xorg-video-nvidia-450-server/jammy-updates,jammy-security 450.216.04-0ubuntu0.22.04.1 amd64
  NVIDIA binary Xorg driver
```

From this list it shows that the latest compatible driver is 525 and we could install it with `` apt install `nvidia-headless-no-dkms-525-server` and `nvidia-utils-525-server` `` but instead we will download the drivers off the website and install them manually so we know exactly whats installed.

Go to [https://www.nvidia.com/en-gb/drivers/unix/linux-amd64-display-archive/](https://www.nvidia.com/en-gb/drivers/unix/linux-amd64-display-archive/) and download the latest driver for your GPU to your VM. Use wget to download it to your virtual machine.

```bash
wget https://uk.download.nvidia.com/XFree86/Linux-x86_64/525.60.11/NVIDIA-Linux-x86_64-525.60.11.run
```

Run the installer with:

```bash
./NVIDIA-Linux-x86_64-525.60.11.run --no-questions --ui=none
```

Once the installer completes you can check the status of the driver with `nvidia-smi`

```bash
dazeb@jellyfin:~$ nvidia-smi
Sun Jan  8 23:31:28 2023
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 510.108.03   Driver Version: 510.108.03   CUDA Version: 11.6     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:06:10.0 Off |                  N/A |
| N/A   50C    P8    N/A /  N/A |      0MiB /  2048MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

or `nvidia-smi -L`

```bash
dazeb@jellyfin:~$ nvidia-smi -L
GPU 0: NVIDIA GeForce GTX 960M (UUID: GPU-432dca14-a3c7-c2c2-8e60-9f635b5fc2ad)
```

If nothing comes up or you get an error you may need to try installing a different driver or with a different method. I decided to use driver version 510.

### Driver Install Method #2 (Recommended)

You can install the drivers that are recommended by Ubuntu

Install the [Official PPA](https://launchpad.net/~graphics-drivers/+archive/ubuntu/ppa) (This command may not be necessary as all LTS Ubuntu versions already include the latest Linux Nvidia drivers)

```bash
sudo add-apt-repository ppa:graphics-drivers/ppa
```

See the recommended device drivers

```bash
sudo apt upgrade -y
sudo ubuntu-drivers devices
```

![](https://mlufdnrmzupp.i.optimole.com/w:571/h:295/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2023/01/ubuntu-drivers.png)

Automatically install the drivers

```bash
sudo ubuntu-drivers autoinstall
```

To remove the repo

```bash
sudo add-apt-repository --remove ppa:graphics-drivers/ppa
```

Remove Nvidia drivers completely

```bash
sudo apt purge nvidia-*
sudo apt autoremove
```

## Jellyfin Install

We can go ahead and install Jellyfin now to test our transcoding. The installation is simple enough. You can find the full instructions on the [Jellyfin docs](https://jellyfin.org/docs/)

Install some needed packages.

```bash
sudo apt install curl gnupg
```

Enable universe repository for FFMPEG.

```bash
sudo add-apt-repository universe
```

Create a keyrings folder and install the Jellyfin keyring.

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://repo.jellyfin.org/$( awk -F'=' '/^ID=/{ print $NF }' /etc/os-release )/jellyfin_team.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/jellyfin.gpg
```

Add a repository configuration at `/etc/apt/sources.list.d/jellyfin.sources` (paste as one whole command)

```shell
cat <<EOF | sudo tee /etc/apt/sources.list.d/jellyfin.sources
Types: deb
URIs: https://repo.jellyfin.org/$( awk -F'=' '/^ID=/{ print $NF }' /etc/os-release )
Suites: $( awk -F'=' '/^VERSION_CODENAME=/{ print $NF }' /etc/os-release )
Components: main
Architectures: $( dpkg --print-architecture )
Signed-By: /etc/apt/keyrings/jellyfin.gpg
EOF
```

Update the packages and install Jellyfin

```bash
sudo apt update && sudo apt install jellyfin
```

Restart the Jellyfin service

```bash
sudo systemctl restart jellyfin
```

See the service status

```bash
sudo service jellyfin status
```

## Transcode Testing with Jellyfin

The first way to test transcoding is to obviously try a video that requires transcoding but when running locally you will have to force the server into transcoding by reducing the quality while the video is playing.

First make sure you have added some media, samba and NFS work great. Open Jellyfin and go to the `Dashboard > Playback` select `Nvidia NVENC`.

![](https://mlufdnrmzupp.i.optimole.com/w:1024/h:570/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2023/01/enable-nvidia.png)

Click the cog icon ⚙️ on the video and select a very low quality.

![](https://mlufdnrmzupp.i.optimole.com/w:787/h:508/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2023/01/quality.png)

Go back to your terminal and send the `nvidia-smi` command again.

```bash
dazeb@jellyfin:~$ nvidia-smi
Sun Jan  8 23:52:51 2023
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 510.108.03   Driver Version: 510.108.03   CUDA Version: 11.6     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:06:10.0 Off |                  N/A |
| N/A   57C    P0    N/A /  N/A |     97MiB /  2048MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A      2693      C   ...ib/jellyfin-ffmpeg/ffmpeg       94MiB |# <--GPU!
+-----------------------------------------------------------------------------+
```

Now, if we did everything correctly you should have a running process! This means everything works and you have enabled Hardware Acceleration in your virtual machine.

## Patch the Driver for More Transcode Streams

We can patch the Nvidia driver to allow it to stream to more than the consumer grade devices do. The project is on GitHub [https://github.com/keylase/nvidia-patch](https://github.com/keylase/nvidia-patch)

Look up your card on this list [https://developer.nvidia.com/video-encode-and-decode-gpu-support-matrix-new#Encoder](https://developer.nvidia.com/video-encode-and-decode-gpu-support-matrix-new#Encoder) see how many streams your card supports and if you need more streams you can install the patch by downloading and running the file.

## Install Jellyseerr

Jellyseer is a fork of Overseerr for Jellyfin, Plex and Emby. Overseerr is a movie and tv show request system that you can use to add media to Sonarr and Radarr for download.

The best way to set up Jellyseer is within a Docker container in an unprivileged Proxmox LXC Container.

An easy way to add a Docker LXC is to use [Proxmox Helper Scripts](https://tteck.github.io/Proxmox/) by [tteck](https://github.com/tteck)

Below is a “oneliner” that you can paste into your Proxmox Terminal. It will take you through all the steps and options. If you create the container privileged it will have access to USB devices automatically. I will not be creating a privileged container because I do not need full access to the system as I will be using a samba share to add my media folder.

During install you will be asked if you want to install Portainer and the new Docker Compose plugin which the command will be `docker compose` not the old `docker-compose`. Id suggest selecting yes if you want a GUI to manage your Docker containers.

Docker LXC setup on Proxmox

`bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/docker-v5.sh)"`

Now we have a LXC container with Docker installed we can go ahead with the Jellyfin install. If you are running a homelab id suggest setting up a Cloudflare Tunnel or Nginx Proxy Manager. Nginx Proxy Manager can be found in the [Proxmox Helper Scripts](https://tteck.github.io/Proxmox/) also.

You can install using the Portainer GUI or via the command line. Here is both methods.

### Install Jellyseerr – Portainer

Open Portainer on the Docker LXC which will be port 9000 on the LXC IP.

Go to `Stacks > New Stack`

![](https://mlufdnrmzupp.i.optimole.com/w:1024/h:589/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2023/01/portainer-stacks.png)

Give the stack a name and paste in the following content but remember to change the info like timezone to match your own and the config directory `/path/to/appdata/config` change to match your own data folder.

```bash
version: '3'
services:
    jellyseerr:
       image: fallenbagel/jellyseerr:latest
       container_name: jellyseerr
       environment:
            - LOG_LEVEL=debug
            - TZ=Europe/London
       ports:
            - 5055:5055
       volumes:
            - /path/to/appdata/config:/app/config
       restart: unless-stopped
```

Click deploy stack and Jellyseerr will be created. Move on to the next part, [setting up Jellyseer](http://32-jellyseerr-setup/)

### Install Jellyseer – Terminal Commands

Quick commands to install Jellyseer from the command line

Create a folder and a docker compose file for your project

```bash
mkdir jellyfin
cd jellyfin
nano docker-compose.yml
```

Paste in the following, and change the values to your own like TZ and config folder.

```bash
version: '3'
services:
    jellyseerr:
       image: fallenbagel/jellyseerr:latest
       container_name: jellyseerr
       environment:
            - LOG_LEVEL=debug
            - TZ=Europe/London
       ports:
            - 5055:5055
       volumes:
            - /path/to/appdata/config:/app/config
       restart: unless-stopped
```

Type `CTRL` + `x` then `y` to save and close.

Bring up the stack

```bash
docker compose up -d
```

The stack will download and you will be able to reach it on the LXC IP port `5055`.

## References

All the guides I used to learn Nvidia PCI GPU Passthrough.

-   [https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF)
-   [https://pve.proxmox.com/wiki/Pci_passthrough](https://pve.proxmox.com/wiki/Pci_passthrough)
-   [https://www.linuxbabe.com/ubuntu/install-nvidia-driver-ubuntu](https://www.linuxbabe.com/ubuntu/install-nvidia-driver-ubuntu)
-   [https://www.reddit.com/r/homelab/comments/b5xpua/the_ultimate_beginners_guide_to_gpu_passthrough/](https://www.reddit.com/r/homelab/comments/b5xpua/the_ultimate_beginners_guide_to_gpu_passthrough/)
-   [https://theorangeone.net/posts/lxc-nvidia-gpu-passthrough/](https://theorangeone.net/posts/lxc-nvidia-gpu-passthrough/)
-   [https://jellyfin.org/docs/general/installation/linux#ubuntu](https://jellyfin.org/docs/general/installation/linux#ubuntu)
-   [https://vfio.blogspot.com/2014/08/does-my-graphics-card-rom-support-efi.html](https://vfio.blogspot.com/2014/08/does-my-graphics-card-rom-support-efi.html)[](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/)

## Affiliate Links

Some great affiliate links give you free cash to spend on VPS. If you use these links I will get a small commission if you choose to sign up.

-   [Vultr.com – $100 free to spend on VPS](https://www.vultr.com/?ref=9271404-8H)
-   [DigitalOcean.com – £200 to spend on all services including VPS](https://m.do.co/c/d42dcae75476)
-   [Atlantic.net – 1 Core 2GB RAM VPS FREE for 12 months from signup + $15](https://cloud.atlantic.net/r/nydern9z)
-   [Hetzner Cloud – $20 Free to use on all products](https://hetzner.cloud/?ref=5q9zjIQ5A7j8)
-   [GSuite – Google Workspace 10% off the first year](https://support.google.com/a/answer/2636415?hl=en)
-   Add `6AKCQG7CAFYCQWA` to your [Google Workspace Account](https://workspace.google.com/intl/en_uk/)

[# Emby](https://dazeb.uk/tag/emby/)[# GPU Acceleration](https://dazeb.uk/tag/gpu-acceleration/)[# GPU Passthrough](https://dazeb.uk/tag/gpu-passthrough/)[# Jellyfin](https://dazeb.uk/tag/jellyfin/)[# Plex](https://dazeb.uk/tag/plex/)[# Proxmox](https://dazeb.uk/tag/proxmox/)[# Ubuntu 22.04](https://dazeb.uk/tag/ubuntu-22-04/)

[

![hetzner storagebox thumbnail](https://mlufdnrmzupp.i.optimole.com/w:300/h:234/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2022/10/hetzner-dazebuk-proxmox-storagebox.webp)

PREVIOUS

](https://dazeb.uk/how-to-add-a-hetzner-storagebox-to-proxmox-with-smb-cifs/)

### Related Posts

[![proxmox over ssh](https://mlufdnrmzupp.i.optimole.com/w:940/h:788/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2023/01/proxmox-over-ssh.png)](https://dazeb.uk/how-to-access-proxmox-web-gui-remotely-using-ssh/)

#### [How to Access Proxmox Web GUI Remotely Using SSH](https://dazeb.uk/how-to-access-proxmox-web-gui-remotely-using-ssh/)

-   January 4, 2023

[![hetzner storagebox thumbnail](https://mlufdnrmzupp.i.optimole.com/w:640/h:500/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2022/10/hetzner-dazebuk-proxmox-storagebox.webp)](https://dazeb.uk/how-to-add-a-hetzner-storagebox-to-proxmox-with-smb-cifs/)

#### [How to Add a Hetzner StorageBox to Proxmox with SMB/CIFS](https://dazeb.uk/how-to-add-a-hetzner-storagebox-to-proxmox-with-smb-cifs/)

-   October 24, 2022

[![hetzner proxmox single ip](https://mlufdnrmzupp.i.optimole.com/w:640/h:500/q:mauto/f:avif/https://dazeb.uk/wp-content/uploads/2022/10/hetzner-dazebuk-proxmox-single-ip-2.webp)](https://dazeb.uk/hetzner-proxmox-multiple-vms-on-one-single-ip-address/)

#### [Hetzner & Proxmox, Multiple VM’s on One Single IP Address](https://dazeb.uk/hetzner-proxmox-multiple-vms-on-one-single-ip-address/)

-   October 14, 2022

[](https://github.com/DazEB2)[](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#)[](http://daz@dazeb.uk/)[](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#)[](http://dazeb/)[](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#)[](http://dazebuk/)[](https://dazeb.uk/proxmox-vm-gpu-hardware-acceleration-for-jellyfin-plex-emby/#)

Copyright © 2023 dazeb

## VPS Hosting

-   [Hetzner Cloud](https://hetzner.cloud/?ref=5q9zjIQ5A7j8)
-   [Atlantic.net](https://cloud.atlantic.net/r/nydern9z)
-   [Vultr.com](https://www.vultr.com/?ref=9271404-8H)
-   [DigitalOcean](https://m.do.co/c/d42dcae75476)
-   [Scaleway](https://scaleway.com/)

## Server Setup & Provisioning

-   [moss.sh](http://moss.sh/)
-   [Runcloud.io](http://runcloud.io/)
-   [Easyengine.io](https://easyengine.io/)
-   [CloudPanel v2](http://cloudpanel.io/)
-   [ServerPilot.io](http://serverpilot.io/)

## Cron & Monitoring

-   [Cronitor.io](http://cronitor.io/)
-   [UptimeRobot.com](https://uptimerobot.com/)
-   [Netumo.app](https://www.netumo.app/)
-   [Sentry.io](https://sentry.io/)
-   [BetterUptime](https://betterstack.com/better-uptime)

## Static Site Hosting

-   [Netlify.com](https://netlify.com/)
-   [Vercel.com](http://vercel.com/)
-   [Render.com](https://render.com/)
-   [Gatsbyjs.com](https://www.gatsbyjs.com/)
-   [Heroku.com](https://heroku.com/)