ref: https://android.stackexchange.com/questions/216435/how-to-build-lineageos-for-a-device-without-official-support

Firstly you need to setup the build environment. You should be running Ubuntu 18.04. LineageOS [build guide](https://wiki.lineageos.org/devices/kenzo/build) for a random device can be used as a reference. Reproducing the steps here:

Install the build packages:

```
sudo apt install bc bison build-essential ccache curl flex g++-multilib gcc-multilib git gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev liblz4-tool libncurses5-dev libsdl1.2-dev libssl-dev libwxgtk3.0-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev adb fastboot
```

Create the directories:

```
mkdir -p ~/bin
mkdir -p ~/android/lineage
```

Install the repo command:

```
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

Put the ~/bin directory in your path of execution by adding the following to your `~/.profile` if it's missing:

```
# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi
```

Then, run `source ~/.profile` to update your environment. After that, initialize the LineageOS source repository and start downloading:

```
cd ~/android/lineage
repo init -u https://github.com/LineageOS/android.git -b lineage-16.0
repo sync
```

Downloading LineageOS source will take some time. After that we will deviate from the official build guide. Create a local manifest file telling the repo tool where to pick up the kernel source, device tree and vendor files. The corresponding file for platina can be downloaded from [here](https://gist.githubusercontent.com/daltonfury42/c33fdfa7a44f261018a5d35dea7eb245/raw/5fc372ec0d36117fa3e7698d8de1952c1bac6b6a/platina.xml). If you are building for a different device, use this manifest file as a reference and edit it accordingly. Save the file as

```
.repo/local_manifests/platina.xml 
```

Rerun repo tool to fetch them:

```
repo sync
```

Initialise your build environment:

```
source build/envsetup.sh
```

Enable caching to speed up the build:

```
export USE_CCACHE=1
```

You should add the above line to your `.bashrc` file. Then set the max cache size:

```
ccache -M 50G
```

Configure jack:

```
export ANDROID_JACK_VM_ARGS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4G"
```

Edit `device/xiaomi/platina/BoardConfig.mk` and set the parameter `TARGET_KERNEL_CONFIG` to `platina_user_defconfig`:

[![enter image description here](https://i.stack.imgur.com/reFCC.png)](https://i.stack.imgur.com/reFCC.png)

`TARGET_KERNEL_SOURCE` points to the kernel code that is to be used during the build. `TARGET_KERNEL_CONFIG` specifies which configuration is to be used. By looking in `arch/arm64/configs/` in the kernel source, I set the value to `platina_user_defconfig`

Finally start the build:

```
lunch 16
mka bacon -j8
```

This should lake a few hours to complete. I usually leave it running overnight. Here is where things get challenging. It’s possible that the build can fail due to errors, in which case you will have to fix it. Once the build has completed, you will find the zip file, ready to flash:

[![enter image description here](https://i.stack.imgur.com/m3GA2.png)](https://i.stack.imgur.com/m3GA2.png)

[permalink](https://android.stackexchange.com/a/216436/379921 "Short permalink to this answer")

