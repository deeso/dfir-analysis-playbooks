
### Overview

**Task**: Perform Timeline Event Analysis with MSTIC using Splunk Logs

**Description**: Use Microsoft Threat Intelligence Python (msticpy) Library to create an interactive timeline, see https://msticpy.readthedocs.io/en/latest/visualization/EventTimeline.html#plotting-a-simple-timeline

### Requirements
1. Install msticpy, Jupyter Lab and IPython
2. Audit and understand which fields from logs will be used
3. Know the sourcetype, fields of interest, input JSON File

### Tasks
1. Create JSON logs from Splunk

### Products
1. Bokeh timeline of the events

### Post-activities
1. None

### Commands

##### install `mstic` and `ipython`
```
pip3 install msticpy ipython
ipython
```

##### Open ipython run the code (optionally jupyter-lab)
```
import json
from datetime import datetime, timedelta
from bokeh.io import output_file, show

from IPython import get_ipython
from IPython.display import display, HTML, Markdown
import ipywidgets as widgets

from msticpy.nbtools import *
from msticpy.sectools import *

import pandas as pd
pd.set_option('display.max_rows', 100)
pd.set_option('display.max_columns', 50)
pd.set_option('display.max_colwidth', 100)


WIDGET_DEFAULTS = {'layout': widgets.Layout(width='95%'),
                   'style': {'description_width': 'initial'}}

# Set pre-requisite values (input, output, columns, sourcetype)
input_file = "SET_INPUT_FILE"
sourcetype = "SET_SOURCE_TYPE"
output_html = "SET_OUTPUT_HTML"
columns_of_interest = ["SET_COLUMNS_OF_INTEREST"]

# load the data
data = [json.loads(i) for i in open(input_file).readlines()]

# set the time field to TimeGenerated in the input dictionary
items = []
for d in data:
    r = d.get('result', None)
    if r:
        r = r.copy()
        r['TimeGenerated'] = pd.to_datetime(r['_time'])
        r['date'] = r['_time']
        del[r['_time']]
        items.append(r)

df = pd.DataFrame([i for i in items if i['sourcetype'] in sourcetype])

# large datasets with duplication will consume RAM, ease system contention here
del data
del items

events_df = df[df.sourcetype.isin([sourcetype])].dropna(axis=1, how='all')
fig = nbdisplay.display_timeline(data=events_df, time_column="TimeGenerated", source_columns=columns_of_interest) 
output_file(output_html)
show(fig)

```
