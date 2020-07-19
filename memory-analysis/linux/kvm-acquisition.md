
### Overview

Task: Dump ELF Image with KVM
Description: Using KVM, dump a VM's memory for analysis with volatility or something else

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
```
export MEMORY_DUMP=/mnt/memory_dump/memory.img
export DOMAIN_NAME=target_host
export COPY_TO=/tmp/

# dump guest memory and copy it somewhere else
sudo virsh dump $DOMAIN_NAME $MEMORY_DUMP --bypass-cache --format elf --live --memory-only 
cp $MEMORY_DUMP $COPY_TO

```
