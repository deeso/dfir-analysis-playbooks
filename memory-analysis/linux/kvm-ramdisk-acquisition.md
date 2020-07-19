
### Overview

Task: Create RAM Disk and Dump ELF Image with KVM
Description: Using KVM, dump a VM's memory for analysis with volatility or something else

### Requirements
1. Linux system with enough space to dump the guest's memory
2. Root access to create RAM disk, dump memory and adjust access rights

### Tasks
1. Create RAM disk for memory
2. Dump KVM guest memory
3. copy memory dump out of RAM disk

### Products
1. Memory dump for analysis

### Post-activities
1. Copy the memory to new location

### Commands
```
# Note RAMDISK_SZ needs to be able to accommodate the memory image

export RAMDISK_SZ=16G
export RAMDISK_LOC=/mnt/ramdisk_backend
export RAMDISK_FS=/mnt/ramdisk_backend/ramdisk_data
export MOUNT_POINT=/mnt/memory_dump
export MEMORY_DUMP=/mnt/memory_dump/memory.img
export DOMAIN_NAME=target_host
export COPY_TO=/tmp/

# create the RAM disk
sudo mkdir -p $MOUNT_POINT $RAMDISK_LOC
sudo mount -t ramfs ramfs $RAMDISK_LOC
sudo truncate -s $RAMDISK_SZ $RAMDISK_FS
mkfs.ext4 $RAMDISK_FS

# mount the new ramdisk
sudo mount $RAMDISK_FS $MOUNT_POINT
sudo chmod a+rw -R $MOUNT_POINT

# dump guest memory and copy it somewhere else
sudo virsh dump $DOMAIN_NAME $MEMORY_DUMP --bypass-cache --format elf --live --memory-only 
cp $MEMORY_DUMP $COPY_TO

# unmount ramdisk components
umount $MOUNT_POINT $RAMDISK_LOC

```
