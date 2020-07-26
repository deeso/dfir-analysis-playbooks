
### Overview

Task: Search a core file for potential ELF files
Description: Using grep search a binary file for the location of "ELF" magic number. Source: https://stackoverflow.com/questions/6319878/using-grep-to-search-for-hex-strings-in-a-file#17168847

### Requirements
1. Linux process core file
2. Access to `grep`

### Tasks
1. Find boundaries that might be an ELF file

### Products
1. List of locations where ELF appears

### Post-activities
1. Normalize the offsets to hexadecimal
2. Check and extract the ELF info from the core file

### Commands
```
# Soure https://stackoverflow.com/questions/6319878/using-grep-to-search-for-hex-strings-in-a-file

export CORE_FILE=filename
export OUTPUT > ELF_locations.txt
# validate this works on ELFs besides x64
grep -obUaP grep -obUaP "\x7f\x45\x4c\x46[\x01-\x02][\x01-\x02][\x00-\x12]" $CORE_FILE > $OUTPUT

```
