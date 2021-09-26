## Android Log Parser
- Convert a android log into csv/xslx 
- Load the csv/xlsx data in Pandas
- Process the data
- Plot the graph for pid's (applications) vs # of lines logged
- to-do many more

```python
# tons of libraries 
import regex as re 
import pprint
import csv
import os
import pandas as pd
from docx import Document
from openpyxl import Workbook
import pickle as p
from matplotlib import pyplot as plt
```

### 0. Reading Files


```python
"""
Read the file in binary mode and 
Print the number of log lines
"""

filename = 'android-sample.log'
output_file = filename.split('.')[0]

with open(filename, 'rb') as log:
    lines = log.readlines()
    
print(f"Number of log lines # {len(lines)}")

```

    Number of log lines # 174094
    

### 1. Parsing and Slicing Data


```python
"""
# Android log hdfs format
    <date> <time> <pid> <tid> <level> <component>: <content>
"""
columns=["date", "time",  "pid", "level", "component", "content"]
```

### 2. csv, openpyxl libraries


```python
# Writing into CSV 
# find the longest log line length
# failed logs - logs which are not in hdfs format
# failed logs is a dict with key as lines number and value as line content

with open(output_file + ".csv", 'w', encoding='utf-8') as f:
    write = csv.writer(f)
    write.writerow(columns)    

    failed_messages = {}
    long_line_length = 0
    for cnt, line in enumerate(lines):
        long_line_length = max(long_line_length, len(line))
        log_messages = []
        line = line.strip()
        if not line:
            continue
        try:
            line = re.sub(r'\s+',' ', line.decode("utf-8"))
            date, time, pid, _, log_level, component, *content  = line.split()
            log_messages.extend([date, time, pid, log_level, component, ' '.join([str(elem) for elem in content])])
        except Exception as e:
            failed_messages[cnt] = line
            pass
        write.writerow(log_messages)

```


```python
print(f"Generated a output csv file # {output_file}.csv")
print("Failed Logs : \n #line: content ")
pprint.pprint(failed_messages)
```

    Generated a output Excel file # android-sample.csv
    Failed Logs : 
    #line: content 
    {9976: '--------- ',
     9977: '---------  radio',
     9978: '---------  events',
     9979: '---------  system',
     9980: '---------  crash'}
    


```python
# Writing into xslx
wb = Workbook()
ws = wb.active

ws.append(columns)
failed_messages = {}
long_line_length = 0

for cnt, line in enumerate(lines):
    long_line_length = max(long_line_length, len(line))
    line = line.strip()
    log_messages = []
    try:
        line = re.sub(r'\s+',' ', line.decode("utf-8"))
        date, time, pid, _, log_level, component, *content  = line.split()
        log_messages.extend([date, time, pid, log_level, component, ' '.join([str(elem) for elem in content])])
        ws.append(log_messages)
    except Exception as e:
        failed_messages[cnt] = line
        pass

wb.save(output_file + ".xlsx")

```


```python
print(f"Generated a output Excel file # {output_file}.xlsx")
print("Failed Logs : \n #line: content ")
pprint.pprint(failed_messages)
```

    Generated a output Excel file # android-sample.xlsx
    Failed Logs : 
    #line: content 
    {9976: '--------- ',
     9977: '---------  radio',
     9978: '---------  events',
     9979: '---------  system',
     9980: '---------  crash'}
    

### 3. Data Frames - Pandas


```python
# Time to Panda Magic 
df = pd.read_excel(output_file.split('.')[0] + ".xlsx")
df.head(5)
```


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>time</th>
      <th>pid</th>
      <th>level</th>
      <th>component</th>
      <th>content</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>09-20</td>
      <td>08:37:00.912</td>
      <td>5402</td>
      <td>D</td>
      <td>ABCD</td>
      <td>:XYZ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>09-20</td>
      <td>08:37:00.912</td>
      <td>5402</td>
      <td>D</td>
      <td>ABCD</td>
      <td>: qwerdft, -...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>09-20</td>
      <td>08:37:00.912</td>
      <td>5402</td>
      <td>D</td>
      <td>ABCD</td>
      <td>: -asxdc,...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>09-20</td>
      <td>08:37:00.913</td>
      <td>5402</td>
      <td>D</td>
      <td>ABCD</td>
      <td>: , qwer...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>09-20</td>
      <td>08:37:00.913</td>
      <td>5402</td>
      <td>D</td>
      <td>ABCD</td>
      <td>: 123456...</td>
    </tr>
  </tbody>
</table>
</div>



### 4. Matplotlib Graphs


```python
# time to plot for fun
# 
# Bar-Chart for top 10 apps with number of lines logged
# 

num = df.pid.value_counts().head(10)
print(num.head(5))
num.plot(xlabel = "PID", ylabel='Number of Lines', title='PID vs #Lines', kind='bar')
```
![plot](https://user-images.githubusercontent.com/3856415/134793641-1e5a9455-e560-432b-9d2b-fba92a6efcf4.PNG)

```python

```

## To-Do

### 5. Pickle 

### 6. Numpy
