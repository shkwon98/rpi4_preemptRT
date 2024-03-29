# Raspberry Pi 4 Preempt-RT Patch
These steps will help you through the installation of preempt-rt on the **Raspberry Pi 4**. The kernel that is used is the 4.19.86 linux kernel.

* It contains additional support as an option to use W5500 Ethernet network driver for users who need extra ethernet port on raspberry pi.
* You can find out much more simplified method using pre-built kernel on the [link](https://github.com/shkwon98/rpi4-preemptrt-prebuilt)
<br>



## Requirements:
* Raspberry Pi 4
* 16gb Micro-SD card + reader
* Computer with Ubuntu
* (Option) WIZnet W5500 Ethernet chip with SPI interface

> Create a new raspbian image on the micro-SDcard with the Pi imager. Use the file on the link below.<br>
> [2020-02-13-raspbian-buster.zip](http://downloads.raspberrypi.org/raspbian/images/raspbian-2020-02-14/2020-02-13-raspbian-buster.zip)<br>
<br>



## A. Directory initialization

1. Create directories on the host computer
```console
host@ubuntu:~$ mkdir rpi-kernel
host@ubuntu:~$ cd rpi-kernel
host@ubuntu:~/rpi-kernel$ mkdir rt-kernel
```

2. Start by pulling the linux repository
```console
host@ubuntu:~/rpi-kernel$ git clone -b rpi-4.19.y-rt https://github.com/raspberrypi/linux.git
host@ubuntu:~/rpi-kernel$ git clone https://github.com/raspberrypi/tools.git
```

<br>

## B. Building linux

1. Environment variable setting
```console
host@ubuntu:~/rpi-kernel$ export ARCH=arm
host@ubuntu:~/rpi-kernel$ export CROSS_COMPILE=~/rpi-kernel/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin/arm-linux-gnueabihf-
host@ubuntu:~/rpi-kernel$ export INSTALL_MOD_PATH=~/rpi-kernel/rt-kernel
host@ubuntu:~/rpi-kernel$ export INSTALL_DTBS_PATH=~/rpi-kernel/rt-kernel
```

2. Setting for Raspberry pi 4
```console
host@ubuntu:~/rpi-kernel$ export KERNEL=kernel7l
host@ubuntu:~/rpi-kernel$ cd linux/
host@ubuntu:~/rpi-kernel/linux$ make bcm2711_defconfig
```

3. Configurate Kernel Compile options
```console
host@ubuntu:~/rpi-kernel/linux$ sudo apt-get install libncurses-dev libssl-dev flex bison
host@ubuntu:~/rpi-kernel/linux$ make menuconfig
```

4. Edit the following variables:
```
[Kernel hacking] → [Debug preemptible kernel] Off
(Option) [Device Drivers] → [Network device support] → [Ethernet driver support] → [WIZnet W5100 Ethernet support], [WIZnetW5100/W5200/W5500 Ethernet support for SPI mode] On
```

5. Build the linux kernel (This can take some time, so get coffee or tea...)
```console
host@ubuntu:~/rpi-kernel/linux$ make -j4 zImage modules dtbs
host@ubuntu:~/rpi-kernel/linux$ make –j4 modules_install dtbs_install
```

<br>

## C. Installing the linux kernel on the Micro-SD card

1. Create .img Kernel file
```console
host@ubuntu:~/rpi-kernel/linux$ mkdir $INSTALL_MOD_PATH/boot
host@ubuntu:~/rpi-kernel/linux$ ./scripts/mkknlimg ./arch/arm/boot/zImage $INSTALL_MOD_PATH/boot/$KERNEL.img
```

2. Replace the Kernel to Raspberry Pi
```console
host@ubuntu:~/rpi-kernel/linux$ cd $INSTALL_MOD_PATH
host@ubuntu:~/rpi-kernel/rt-kernel$ tar czf ../rt-kernel.tgz *
host@ubuntu:~/rpi-kernel/rt-kernel$ cd ..
host@ubuntu:~/rpi-kernel$ scp rt-kernel.tgz pi@<raspberry pi IP address>:/tmp
```

<br>

## D. Installing the preempt-rt on the Raspberry pi

1. Inside the raspberry pi, unpack the .tgz and copy it
```console
pi@raspberrypi:~$ cd /tmp
pi@raspberrypi:/tmp$ tar xzf rt-kernel.tgz
pi@raspberrypi:/tmp$ cd boot
pi@raspberrypi:/tmp/boot$ sudo cp -rd * /boot/
pi@raspberrypi:/tmp/boot$ cd ../lib
pi@raspberrypi:/tmp/lib$ sudo cp -rd * /lib/
pi@raspberrypi:/tmp/lib$ cd ../overlays
pi@raspberrypi:/tmp/overlays$ sudo cp -d * /boot/overlays
pi@raspberrypi:/tmp/overlays$ cd ..
pi@raspberrypi:/tmp$ sudo cp -d bcm* /boot/
```

2. Edit the boot configutarion
```console
pi@raspberrypi:/tmp$ sudo nano /boot/config.txt
```
> Add in the beginning:
```
kernel=kernel7l.img
```
> (Option) Type in the last line:
```
dtoverlay=w5500
```

3. Reboot and check the patched kernel
```console
pi@raspberrypi:/tmp$ sudo reboot
pi@raspberrypi:~$ uname -r
```

<br>

## BENCHMARK:
* [cyclictest benchmark](https://github.com/shkwon98/cyclictest)
