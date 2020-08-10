
### Overview

Task: Create symbol table for Vol3 Memory Analysis
Description: The symbol table needs to be created in order to properly analyze the memory.  

### Requirements
1. Linux system with the same exact kernel banner and system map

### Tasks
1. Grab the Kernel Symbols (or debug kernel with symbols) and System Map
2. Create volatility symbol table.

### Products
1. Symbol table 

### Post-activities
1. Analyze Linux memory image

### Commands
```
export TARGET_KERNEL_VERSION=

# maybe kernel image has symbols and is part of a distro, get kernel image extract and copy kernel to /tmp
mkdir /tmp/extract_kernel && cd /tmp/extract_kernel
cp /boot/vmlinuz-$TARGET_KERNEL_VERSION . && cp /boot/System.map-$TARGET_KERNEL_VERSION .
sudo apt-get install linux-headers-$TARGET_KERNEL_VERSION

sudo /usr/src/linux-headers-$TARGET_KERNEL_VERSION/scripts/extract-vmlinux vmlinuz-$TARGET_KERNEL_VERSION > vmlinux
./extract-vmlinux vmlinuz-$TARGET_KERNEL_VERSION 

sudo ./dwarf2json linux --elf /tmp/kernel-extract/vmlinux --system-map /boot/System.map-TARGET_KERNEL_VERSION

# build acquire a similar kernel from the distro
# from a package
echo "deb http://ddebs.ubuntu.com $(lsb_release -cs) main restricted universe multiverse"  | sudo tee -a /etc/apt/sources.list.d/ddebs.list
echo "deb http://ddebs.ubuntu.com $(lsb_release -cs)-updates main restricted universe multiverse" | sudo tee -a /etc/apt/sources.list.d/ddebs.list
sudo apt-get update

sudo apt update
sudo apt search linux-image | grep dbgsym

sudo apt install TARGET_KERNEL_VERSION
./dwarf2json linux --elf/ usr/lib/debug/boot/vmlinux-$TARGET_KERNEL_VERSION


# build symbols from a similar kernel based on the config and system map

# acquiring the kernel src
sudo chown -Rv _apt:root /var/cache/apt/archives/partial/
sudo chmod -Rv 700 /var/cache/apt/archives/partial/
echo "deb-src http://archive.ubuntu.com/ubuntu $(lsb_release -cs) main restricted universe multiverse"  | sudo tee -a /etc/apt/sources.list.d/deb-src.list
echo "deb-src http://archive.ubuntu.com/ubuntu $(lsb_release -cs)-updates main restricted universe multiverse"  | sudo tee -a /etc/apt/sources.list.d/deb-src.list
sudo apt-get update
sudo apt -y source linux-image-$(uname -r) 

# packages for building kernel
sudo apt install install git build-essential kernel-package fakeroot libncurses5-dev libssl-dev ccache

# or clone kernel source
git clone git://kernel.ubuntu.com/ubuntu/ubuntu-$(lsb_release -cs).git $KERNEL_SRC

cp TARGET_CONFIG ubuntu-$(lsb_release -cs)/.config
yes '' | make oldconfig


cd ubuntu-$(lsb_release -cs)
chmod a+x debian/rules
chmod a+x debian/scripts/*
chmod a+x debian/scripts/misc/*
LANG=C fakeroot debian/rules clean
LANG=C fakeroot debian/rules build-generic binary-perarch binary-headers binary-generic skipdbg=false


# build the kernel based on the configuration captured from the machine
# on ubuntu
git clone git://kernel.ubuntu.com/ubuntu/ubuntu-$(lsb_release -cs).git
cd ubuntu-$(lsb_release -cs)
chmod a+x debian/rules
chmod a+x debian/scripts/*
chmod a+x debian/scripts/misc/*
LANG=C fakeroot debian/rules clean
make oldconfig
LANG=C fakeroot debian/rules build-generic binary-headers binary-generic skipdbg=false


# unchecked needs validation
cd $KERNEL_SRC
cp config-$TARGET_KERNEL_VERSION .config
yes '' | make oldconfig
make -j `getconf _NPROCESSORS_ONLN` deb-pkg LOCALVERSION=-$TARGET_KERNEL_VERSION

# attempt to extract the symbols from the memory image


```
