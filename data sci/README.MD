# NICE COMMAND AND SYSTAX

```python
import pandas as pd
using df as DataFrame
usinf se as Series
```

## should know

### attr:values

return n-array of values of from [datatype] 

### attr:value_counts()

return Series of set of values and number of occurence

### attr:any()

Returns False if there's not at least one element within a series or along a Dataframe axis that is True

## การสร้าง DataFrame

```python
df = pd.DataFrame([list with the same shape], [index=[list with the same shape]])  # 1 column, many rows
df = pd.DataFrame([[list n size], [list n size] ... x list], [columns=[column_name_lists n size]])  # x rows with n column
df = pd.DataFrame([{'column1_name': value, 'column2_name': value, a items}, {the same}, ... n dicts])  # n rows with a column
```

note: dataframe ประกอบด้วยหลายๆ series หรืออันเดียวมารวมกัน

## การสร้าง DataFrame จาก list ของแต่ละ row

```python
rows_lists = df['MESSAGE'].tolist()  # get the list for each MESSAGE of email
words_column_df = pd.DataFrame.from_records(rows_lists)  # create new dataframe from given row of lists
```

## การต่อ DataFrame

```python
df = df.append(df1)
```

## การเข้าถึง 1 column ของ df

```python
df['[column_name]']  # return pandas's Series
# is equal to
df.[column_name]  # return pandas's Series
```

## check missing values

```python
df.isnull().values.any()  # DataFrame >> dataframe(bool) >> n-array >> boolean
df['[column_name]'].isnull().values.any()  # Series >> series >> n-array >> boolean
(df['[column_name]'].str.len() == 0).values.any()  # Series >> pd.StringMethods >> Series >> (==0) n-array(boolean) >> n-array >> boolean
```

เราจะเน้นไปที่การตรวจสอบที่เราสนใจ เช่น isnull หรือความยาว = 0 จะได้ True(มีค่าหายไป), False(ค่ายังอยู่)

แล้วใช้ any เพื่อทำการ detect หา True ใน df, Series, n-array นั้นๆ

## locate dataframe with the specific value

```python
df_wanted = df[n-array(boolean)]
#df['[column_name]'].str.len() == 0   # this will return n-array(boolean)
index_wanted = df[df['[column_name]'].str.len() == 0].index  # df[n-array(boolean)condition] return df
df.index.get_loc('[index]')  # DataFrame >> Index >> n-array(boolean)
df[df.index.get_loc('[index]')]  # df[n-array(boolean)condition] return get df
```

## slice the DataFrame

note: สามารถใช้กับ Series ได้ด้วยนะะ

```python
df[start_index:endposition]  # retrun DataFrame
df.iloc[start_index:endposition]  # retrun DataFrame
df.loc[[Index object, index:str, index:str:list]]  # retrun DataFrame
```

การใช้งาน

```python
a = df[df['MESSAGE'].str.len()==0].index
df.loc[a]
df.loc['index']
df.loc[['index1', 'index2']]
```

## get cell's values

```python
at[[row_number], [column number]]
at[[row_number], '[column_name]']
```

## convert Series to List in python

```python
x = df['[column_name]'].tolist()
```

## hererererererereerererer
Convert structured or record ndarray to DataFrame.

## apply all column with function

```python
df['[column_name]'] = df['[column_name]'].apply([function name with out ()])
# df['new'] = df['old].apply(change_to_new)
```

## set or get index name

```python
df.index.name 
```

## Set the value to column

```python
df.loc[df['[column_name]'] == 0, '[column_name]'] = 1
```

if loc[condition] จะไม่สามารถกำหนดค่าได้เพราะเราลือกมาทั้งแถว แต่ถ้าเป็น [condition, column] มันจะอารมณ์เหมือนกับเลือก cell นั้นเพื่อกำหนดค่า
