### Overview

Task: Create Volatility3 Symbols
Description: Create a set of debugging symbols for an existing Ubuntu System using `apt` package manager

### Requirements
1. Ubuntu system with the same release name
2. Root access to install packages

### Tasks
1. Update `apt` sources
2. Install supporting packages
3. Clone `dwarf2json` repo and build

### Products
1. Symbols for Volatility Linux Memory Analysis

### Post-activities
1. Copy the symbols to analysis host.

### Commands
```
# install dependencies
sudo apt -y install libncurses-dev flex bison openssl libssl-dev dkms libelf-dev libudev-dev libpci-dev libiberty-dev autoconf git pkg-config golang zip git


# from a package
echo "deb http://ddebs.ubuntu.com $(lsb_release -cs) main restricted universe multiverse"  | sudo tee -a /etc/apt/sources.list.d/ddebs.list
echo "deb http://ddebs.ubuntu.com $(lsb_release -cs)-updates main restricted universe multiverse" | sudo tee -a /etc/apt/sources.list.d/ddebs.list
sudo apt-get update
sudo apt-get install linux-image-$(uname -r)-dbgsym

# get the dwarf2json
git clone https://github.com/volatilityfoundation/dwarf2json
cd dwarf2json
go build

# create the symbols
dwarf2json linux --elf /usr/lib/debug/boot/vmlinux-$(uname -r) > linux-$(uname -r).json
zip linux-$(uname -r).zip linux-$(uname -r).json 

```
