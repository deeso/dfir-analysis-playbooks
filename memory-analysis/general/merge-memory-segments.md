
### Overview

Task: Create a merged memory segment containing a memory address range
Description: Create a merged memory file, filling empty space between segments.

### Requirements
1. System with enough disk space to dump the merged memory segment
2. System with enough RAM to load memory (or modify script to be serial)
3. Perform **Splitting Memory Maps** task (assumes files are named based on location)

### Tasks
1. Merge contiguous memory segments into a single binary blob

### Products
1. Contiguous file containing the specified memory segments from the process

### Post-activities
1. None

### Commands

Python script to merge the memory segements into a single file filled with the specfied bytes
```
import os

MEMORY_SEGEMENTS = "./memory_segments" # directory with the memory segments
START = 0x0         # START Address
END =   0xffffffff  # END Address
FILLER = b'\x00'    # Filler byte for the pages

OUTPUT = '0x{:016x}-0x{:016x}-merged_segment.bin'

def get_segments(start, end, ms_dir):
    files = [i for i in os.listdir(ms_dir)]
    results = []
    for i in sorted(files):
        x = i.split('.')[0]
        _start = x.split('-')[0]
        _end = x.split('-')[1]
        _s = int(_start, 16)
        _e = int(_end, 16)

        valid = True
        if _e <= start: 
            valid = False
        elif end < _s: 
            valid = False
        
        if valid:
            # print((start, _start, _s),(end, _end, _e))
            results.append([_s, _e, os.path.join(ms_dir, i)])    
    return results

def build_filler_data(start, end, filler = b'\x00'):
    return filler * (end - start)



segment_files = get_segments(START, END, MEMORY_SEGEMENTS) 
data = {}
end_last = None

for s, e, f in segment_files:
    data[s] = open(f, 'rb').read()
    if end_last is None:
        end_last = e
        continue
    elif end_last == s:
        end_last = e
        continue

    next_data = build_filler_data(end_last, s, filler=FILLER)
    data[end_last] = next_data
    end_last = e

addrs = sorted(data.keys())
output = open(OUTPUT.format(addrs[0], end_last), 'wb')
for addr in sorted(data.keys()):
    d = data[addr]
    output.write(d)
output.close()
```