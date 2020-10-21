
### Overview (use at your own risk)

Task: Extract and dump potential files to disk up to a fixed amount
Description: Need to be able to look at some PE Files, need to fix-up the headers

### Requirements
1. System with enough disk space to dump individual memory files
2. System with enough RAM to load memory (or modify script to be serial)
4. Target directory for the segments

### Tasks
1. Extract potential PE files found in memory, may need to be loaded and rebased

### Products
1. Files containing data that might represent the file on disk

### Post-activities
1. None

### Commands

Python script to extract PE files.
```
import binascii
import os

OUTPUT_DIR = 'potential_bins'

output_file_fmt = os.path.join(OUTPUT_DIR, "potential_bin-{:08x}.bin")

data = b"!This program cannot be run in DOS mode."
filename = 'pid.2080.dmp'
binfile = open(filename, 'rb').read()

end = len(binfile)
pos = 0
offset = 0x4c # 76 
file_cnt = 0
default_size = 80000000 # 40 MB

my_db = binfile
abs_pos = 0
while True:
    my_db = binfile[abs_pos:]
    next_loc = my_db.find(data)

    if next_loc == -1:
       break

    abs_pos += next_loc
    print("Found potential_bin @ {:08x}".format(abs_pos))
    db = binfile[abs_pos-offset-1:pos+abs_pos+default_size]
    open(output_file_fmt.format(abs_pos-offset-1), 'wb').write(db)
    abs_pos += 1
```