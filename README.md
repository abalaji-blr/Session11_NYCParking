# Session11_NYCParking
**Context**: The New York County (NYC) collects the data about the parking ticket violation. The dataset can be obtained from [kaggle](https://www.kaggle.com/new-york-city/nyc-parking-tickets).

**Objective**

 * **Goal 1:**

   Create a **lazy iterator** that will return a namedtuple of the data in each row. 

   The data type should be appropriate - if the column is date, you should be storing dates in the named tuple. If the field is integer, then it should be stored as integer. etc.

 * **Goal-2:**

   Calculate the number of violations by car make.

   Note: Try to use lazy evaluation as much as possible - it may not be always possible though. That's OK, as long as it's kept to a minimum.

   

   ### Goal 1: Create a Lazy iterator for the data in each row

   #### Read the csv file and yield each row.

   ```python
   def my_csv_reader(file):
     with open(file) as f:
       # process the header
       hdr = next(f)
       hdr.strip('\n').split(',')
       new_hdr =[ item.replace(' ', '_') for item in hdr.strip('\n').split(',')]
       yield new_hdr
       #print(new_hdr)
   
       # get the remaining lines
       for row in f:
         yield row
   ```

   ```python
   csv_gen  = my_csv_reader(file)
   hdr = next(csv_gen)
   print(hdr)
   
   # sample rows
   for _ in range(4):
     row = next(csv_gen)
     row_values = row.strip('\n').split(',')
     print(row_values)
   ```

   

   ```
   ['Summons_Number', 'Plate_ID', 'Registration_State', 'Plate_Type', 'Issue_Date', 'Violation_Code', 'Vehicle_Body_Type', 'Vehicle_Make', 'Violation_Description']
   ['4006478550', 'VAD7274', 'VA', 'PAS', '10/5/2016', '5', '4D', 'BMW', 'BUS LANE VIOLATION']
   ['4006462396', '22834JK', 'NY', 'COM', '9/30/2016', '5', 'VAN', 'CHEVR', 'BUS LANE VIOLATION']
   ['4007117810', '21791MG', 'NY', 'COM', '4/10/2017', '5', 'VAN', 'DODGE', 'BUS LANE VIOLATION']
   ['4006265037', 'FZX9232', 'NY', 'PAS', '8/23/2016', '5', 'SUBN', 'FORD', 'BUS LANE VIOLATION']
   ```

### Make sure the row values are of appropriate data type

```python
from datetime import datetime

def parse_int(value):
  try:
    return int(value)
  except ValueError:
    return None

def parse_string(value):
  try:
    return str(value)
  except ValueError:
    return None

def parse_date(value):
  try:
    date = datetime.strptime(value, '%m/%d/%Y').date()
    #print(date)
    return(date)
  except ValueError:
    return None
```

### Plug in Row Parser

```python
row_value_parser = (parse_int,
                    parse_string,
                    parse_string,
                    parse_string,
                    parse_date,
                    parse_int,
                    parse_string,
                    parse_string,
                    parse_string
                    )

def my_csv_reader(file):
  with open(file) as f:
    # process the header
    hdr = next(f)
    hdr.strip('\n').split(',')
    new_hdr =[ item.replace(' ', '_') for item in hdr.strip('\n').split(',')]
    yield new_hdr
    #print(new_hdr)

    # get the remaining lines
    for row in f:
      row_values = row.strip('\n').split(',')
      parsed_row = [ func(value) for func, value in zip(row_value_parser, row_values)]
      #print(parsed_row)
      yield parsed_row
```

```python
csv_gen  = my_csv_reader(file)
hdr = next(csv_gen)
print(hdr)

# sample rows
for _ in range(4):
  row = next(csv_gen)
  print(row)
```

### Create namedtuple

```python
Data = namedtuple('Data', hdr)

for _ in range(10):
  row = next(csv_gen)
  data = Data._make(row)
  print(data)

```

```python
Data(Summons_Number=4006535600, Plate_ID='N203399C', Registration_State='NY', Plate_Type='OMT', Issue_Date=datetime.date(2016, 10, 19), Violation_Code=5, Vehicle_Body_Type='SUBN', Vehicle_Make='FORD', Violation_Description='BUS LANE VIOLATION')
Data(Summons_Number=4007156700, Plate_ID='92163MG', Registration_State='NY', Plate_Type='COM', Issue_Date=datetime.date(2017, 4, 13), Violation_Code=5, Vehicle_Body_Type='VAN', Vehicle_Make='FRUEH', Violation_Description='BUS LANE VIOLATION')
Data(Summons_Number=4006687989, Plate_ID='MIQ600', Registration_State='SC', Plate_Type='PAS', Issue_Date=datetime.date(2016, 11, 21), Violation_Code=5, Vehicle_Body_Type='VN', Vehicle_Make='HONDA', Violation_Description='BUS LANE VIOLATION')
```



## Goal 2: Calculate the number of violations based on car make.

### Change the generator to yield both header and value for each row.

```python
def my_csv_reader2(file):
  '''
  CSV reader to read nyc parking ticket file.
  '''
  with open(file) as f:
    # process the header
    hdr = next(f)
    hdr.strip('\n').split(',')
    new_hdr =[ item.replace(' ', '_') for item in hdr.strip('\n').split(',')]

    # get the remaining lines
    for row in f:
      row_values = row.strip('\n').split(',')
      parsed_row = [ func(value) for func, value in zip(row_value_parser, row_values)]
      yield zip(new_hdr, parsed_row)
```

```python
csv_gen = my_csv_reader2(file)
for _ in range(4):
  print(list(next(csv_gen)))
```

```python
[('Summons_Number', 4006478550), ('Plate_ID', 'VAD7274'), ('Registration_State', 'VA'), ('Plate_Type', 'PAS'), ('Issue_Date', datetime.date(2016, 10, 5)), ('Violation_Code', 5), ('Vehicle_Body_Type', '4D'), ('Vehicle_Make', 'BMW'), ('Violation_Description', 'BUS LANE VIOLATION')]
[('Summons_Number', 4006462396), ('Plate_ID', '22834JK'), ('Registration_State', 'NY'), ('Plate_Type', 'COM'), ('Issue_Date', datetime.date(2016, 9, 30)), ('Violation_Code', 5), ('Vehicle_Body_Type', 'VAN'), ('Vehicle_Make', 'CHEVR'), ('Violation_Description', 'BUS LANE VIOLATION')]
```

### Build a dictionary of flags based on car make.

```python
violations_cnt = {}

csv_gen = my_csv_reader2(file)
for _ in csv_gen:
  row = list(next(csv_gen))
  car_make = row[7][1]
  #print(row[7])
  if car_make in violations_cnt:
    violations_cnt[car_make] += 1
  else:
    violations_cnt[car_make] = 1
```

```python
'ACURA': 7,
 'AM/T': 1,
 'AUDI': 7,
 'BMW': 15,
 'BUICK': 3,
 'CADIL': 4,
 'CHEVR': 40,
 'CHRYS': 7,
 'DODGE': 21,
 'FIR': 1,
 'FORD': 51,
 'FRUEH': 21,
 'GMC': 18,
 'HIN': 5,
 'HINO': 1,
 'HONDA': 51,
 'HYUND': 18,
 'INFIN': 6,
 'INTER': 11,
 'ISUZU': 4,
 'JAGUA': 3,
 'JEEP': 11,
 'KENWO': 3,
 'KIA': 3,
```

