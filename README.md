# Introduction
This page aims to provide a quick _"Cook Book"_-like Set of instructions to set up [BeagleLogic](http://beaglelogic.net) on a [BeagleBone Black](https://beagleboard.org/black).

When I recently did this, I found much of the information available online quite dated, which often meant that it did not apply to or work with the most recent Distribution.

1. Pre-requisites
2. Basic Setup
   1.  _Optional:_ Set up SSH
   2.  _Optional:_ `sudo` without Password
   3.  _Optional:_ Grow File System
3. Check PRU Configuration


# Pre-requisites
For completeness, here's the list of things you'll need as well as a few links to instructions on how to get your Board running with the plain distribution.

Things needed:
- Micro SD Card w/ at least 8 GBytes Capacity.
- BeagleBone Black Rev. C Board.

The rest of this page assumes that your Board is powered by USB. Also, it should be connected by cable to your local Ethernet network. Your network should have a workin DHCP and mDNS configuration. I guess mDNS is optional, but it will let you access your board via the default `beaglebone` host name instead of using its IP address.

## Download and Flash Image
Start by downloading the most recent Debian Image for the BeagleBone Black.

At the time of writing, this is the [AM3358 Debian 10.3 2020-04-06 4GB SD IoT](https://debian.beagleboard.org/images/bone-debian-10.3-iot-armhf-2020-04-06-4gb.img.xz) linked from the official [BeagleBoard Site](https://beagleboard.org/latest-images).

Generally, you can follow the official [Getting Started](https://beagleboard.org/getting-started#update) instructions.

# Basic Setup

## Set up SSH
This step is optional and can be skipped. I just don't like re-typing passwords over and over again while I'm working on projects like this one.

### First Log-in
SSH into your board with the default User `debian`. The initial Password is `temppwd`.

```
$ ssh debian@beaglebone
(...)
Debian GNU/Linux 10

BeagleBoard.org Debian Buster IoT Image 2020-04-06

Support: http://elinux.org/Beagleboard:BeagleBoneBlack_Debian

default username:password is [debian:temppwd]


The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
(...)
debian@beaglebone:~$
```

### Generate SSH Keys
On the BeagleBone, run:
```
debian@beaglebone:~$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/debian/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/debian/.ssh/id_rsa.
Your public key has been saved in /home/debian/.ssh/id_rsa.pub.
(...)
```

### Copy authorized SSH Keys
From your Desktop machine, copy its SSH Public Key to the BeagleBoard:

```
$ ssh-copy-id -i ~/.ssh/id_rsa debian@beaglebone
(...)
```

## sudo without Password
Open the `/etc/sudoers.d/admin` file and add the `NOPASSWD` keyword:

```
debian@beaglebone:~$ sudo vim /etc/sudoers.d/admin
(...)
```
Change the file to read this:

```
Defaults	env_keep += "NODE_PATH"
%admin ALL=(ALL:ALL) NOPASSWD: ALL
```

## Grow File System
After installation, grow the file system to use all of your SD Card's Capacity:

```
sudo /opt/scripts/tools/grow_partition.sh
```
# Build BeagleLogic

## Clone the git Repository
Clone the BeagleLogic git repository:
```
debian@beaglebone:~$ git clone https://github.com/abhishek-kakkar/BeagleLogic.git
Cloning into 'BeagleLogic'...
(...)
```

## Install the Kernel Headers
```
debian@beaglebone:~$ sudo apt-get update
(...)                                                                                             
Reading package lists... Done
debian@beaglebone:~$ sudo apt-get update
debian@beaglebone:~$ sudo apt-get install linux-headers-$(uname -r)
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following NEW packages will be installed:
  linux-headers-4.19.94-ti-r42
(...)
```

# Check PRU Configuration
I found the below options / settings were already enabled by default when I initially flashed the image. However, the information may come in handy when trying to figure out why certain things don't work.

## Check the u-Boot Flattened Device Tree (FTD) Overlays
Open `/boot/uEnv.txt` and make sure the following lines are active, i.e. not commented out:

```
uboot_overlay_pru=/lib/firmware/AM335X-PRU-RPROC-4-19-TI-00A0.dtbo
```

To check whether the overlay was in fact loaded, do this:

```
debian@beaglebone:~$ sudo /opt/scripts/tools/version.sh | grep UBOOT
UBOOT: Booted Device-Tree:[am335x-boneblack-uboot-univ.dts]
UBOOT: Loaded Overlay:[AM335X-PRU-RPROC-4-19-TI-00A0]
UBOOT: Loaded Overlay:[BB-ADC-00A0]
UBOOT: Loaded Overlay:[BB-BONE-eMMC1-01-00A0]
```

## Check the PRU remoteproc drivers
Check whether the PRU remoteproc driver attached properly:

```
debian@beaglebone:~$ rc_test_drivers | grep rproc
PASSED: pru-rproc
```

# Assorted Notes
## Set-up uEnv.txt
```
disable_uboot_overlay_video=1
uboot_overlay_pru=/lib/firmware/AM335X-PRU-RPROC-4-19-TI-00A0.dtbo
dtb_overlay=/lib/firmware/beaglelogic-00A0.dtbo
sudo /opt/scripts/tools/update_kernel.sh
sudo apt-get install seeed-modules ???
sudo modprobe pruss ???
```
