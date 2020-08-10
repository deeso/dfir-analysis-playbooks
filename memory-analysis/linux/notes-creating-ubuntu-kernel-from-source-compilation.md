
### Overview

Task: Alternative ways to capture symbols from Linux
Description: info on how to capture symbols

### Requirements
1. Linux system with enough space to dump the guest's memory
2. Root access to dump memory and adjust access rights

### Tasks
1. Dump KVM guest memory

### Products
1. Memory dump for analysis

### Post-activities
1. Copy the memory to new location

### Commands

1. Projects to extract Linux symbols from the memory dump
    1. __https://github.com/pagabuc/kallsyms-extractor__
    2. __https://github.com/marin-m/vmlinux-to-elf__
    
2. Prepping to compile for a specific kernel
```
#during acquisition if possible, grab a copy of all the kernels and info in /boot

# ubuntu in this case lsb_release would be the specific release candidate we want to target
# could be done for other targets
git clone git://kernel.ubuntu.com/ubuntu/ubuntu-$(lsb_release -cs).git
cd ubuntu-$(lsb_release -cs)
cp BASE_KERNEL_CONFIG .config
chmod a+x debian/rules
chmod a+x debian/scripts/*
chmod a+x debian/scripts/misc/*
LANG=C fakeroot debian/rules clean
# consult the differnet builds build-* and binary-*
LANG=C fakeroot debian/rules build-generic binary-headers binary-generic skipdbg=false
# once kernel binary is created, extract the symbols

```



```
export MEMORY_DUMP=/mnt/memory_dump/memory.img
export DOMAIN_NAME=target_host
export COPY_TO=/tmp/

# dump guest memory and copy it somewhere else
sudo virsh dump $DOMAIN_NAME $MEMORY_DUMP --bypass-cache --format elf --live --memory-only 
cp $MEMORY_DUMP $COPY_TO

```
