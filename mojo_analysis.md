```python

```

# Mosquito Joe Reporting 

Project will cover the following: ---Time series analysis --- Plot showing financial advances with set backs --- Projected revenue ---Summary of data year over year and overall --- Overall financial standings --- Highest earning towns that the company covers

Table below was created in mySQL and was used to reformat the dates as actual date types rather than text. Utilized the substring command in SQL that splits up the columns on placement. After that I had to use TRIM to get rid of an slashes left in the dates for concating. I then CONCAT in the b subquery and tranform to a datetime using CAST in c. I then close off the subquery and leave it room for possible expansion. 


```python
"""--- Re formatting the dates
create table all_sprays as(
WITH seven_figures as (
select  accountnum, sitename,siteaddress,  itemnum, 
itemdescription as chemical, 
materialquantity,usagepermixuom, useunit, eventname, employee, priceperunit, totalcost, targetname, temperature, windspeed
 ,substring(completeddate,1,2) as month1, 
substring(completeddate,3,2) as Days1,
substring(completeddate,5,3) as year1,
length(completeddate) as len,
completeddate
from mojo_sprays)
, a as (
select *,trim('/' from month1) as month ,
trim('/' from year1) as year,
trim('/' from Days1) as day
from seven_figures
)
,b as (
select *,concat(20,year,'/', month, '/', day) as completeddat
from a
)
,c as (
select *,cast(completeddat as datetime) as date
from b
)
select * 
from c"""

```

This table is used to transform the resprays into negative revenue. FOr this I used a case when statement that properly assigns respray as a negative for analysis. I then multiplied the bill amount for each customers spray. Some of the data was false so I got rid of any bill amounts that were not over 1 to get rid of 0's that were repeat with accurate data. 


```python
"""with a as 
(select accountnum as account, date,siteaddress,eventname
from all_sprays)
,b as (
select account,Billname, date,billamount,eventname,siteaddress from a 
join customer_info on a.account = customer_info.accountnum
group by accountnum, date, billamount,eventname,siteaddress,Billname)
,c as (
select * , 
case when eventname = 'Re-Spray' then -1
else 1  end as respray          
from b
where billamount > 1
)
,d as(
select account,siteaddress , date,eventname, (respray * billamount) as billamount,Billname
from c
order by date)
select account,siteaddress, date,eventname,billamount,Billname
from d """
```


```python
import os
import matplotlib as plt
from pandas_profiling import ProfileReport
import matplotlib.pyplot as plt
%matplotlib inline
import pandas as pd
from matplotlib.pylab import rcParams
from datetime import datetime
import datetime
from datetime import date
from datetime import *
import numpy as np
```

# Mosquito Joe Reporting 

Project will cover the following:
---Time series analysis 
    --- Plot showing financial advances with set backs
    --- Projected revenue 
    ---Summary of data year over year and overall
--- Overall financial standings 
--- Highest earning towns that the company covers

Changing the current directory to pull the csv.


```python
os.chdir('Desktop/mojo')
```


    ---------------------------------------------------------------------------

    FileNotFoundError                         Traceback (most recent call last)

    Input In [28], in <cell line: 1>()
    ----> 1 os.chdir('Desktop/mojo')


    FileNotFoundError: [Errno 2] No such file or directory: 'Desktop/mojo'



```python
df = pd.read_csv('mojo_revenue.csv')
latitude_long = pd.read_csv('spatial_info.csv')
```


```python
#Looking at the top five entries into the data 
df.head()#.to_clipboard()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>account</th>
      <th>siteaddress</th>
      <th>Billname</th>
      <th>date</th>
      <th>eventname</th>
      <th>billamount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1018107</td>
      <td>XXXX</td>
      <td>XXXX</td>
      <td>0200-02-10 02:00:00</td>
      <td>Synthetic Barrier Spray</td>
      <td>113</td>
    </tr>
    <tr>
      <th>1</th>
      <td>891489</td>
      <td>XXX</td>
      <td>X</td>
      <td>2020-06-12 00:00:00</td>
      <td>Synthetic Barrier Spray</td>
      <td>115</td>
    </tr>
    <tr>
      <th>2</th>
      <td>896665</td>
      <td>XXX.</td>
      <td>XX</td>
      <td>2020-06-13 00:00:00</td>
      <td>Botanical Barrier 3 Week</td>
      <td>123</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891489</td>
      <td>XXX</td>
      <td>XXX</td>
      <td>2020-06-19 00:00:00</td>
      <td>Synthetic Barrier Spray</td>
      <td>115</td>
    </tr>
    <tr>
      <th>4</th>
      <td>896665</td>
      <td>XXX</td>
      <td>XXX</td>
      <td>2020-06-19 00:00:00</td>
      <td>Botanical Barrier 3 Week</td>
      <td>123</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Seeing how the tail compares to the head 
df.tail().to_clipboard()
```


```python
#Data types were working with 
#Notice we do not have any nulls in the data as we got rid of them in sql
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 3729 entries, 0 to 3728
    Data columns (total 6 columns):
     #   Column       Non-Null Count  Dtype 
    ---  ------       --------------  ----- 
     0   account      3729 non-null   int64 
     1   siteaddress  3729 non-null   object
     2   Billname     3729 non-null   object
     3   date         3729 non-null   object
     4   eventname    3729 non-null   object
     5   billamount   3729 non-null   int64 
    dtypes: int64(2), object(4)
    memory usage: 174.9+ KB



```python
#Data takes on a (3729, 6) shape
df.shape
```




    (3729, 6)




```python
#Figuring out the sum amount of revenue over the years
sum(df.billamount)
```




    372329




```python
#Average bill amount is $99 company wide 
# Notice a very concerning -183.00 that may be inaccurate 
#Mean $5 lower than the median 
df.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>account</th>
      <th>billamount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>3.729000e+03</td>
      <td>3729.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>1.000696e+06</td>
      <td>99.846876</td>
    </tr>
    <tr>
      <th>std</th>
      <td>5.019606e+04</td>
      <td>42.437235</td>
    </tr>
    <tr>
      <th>min</th>
      <td>8.914890e+05</td>
      <td>-183.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>9.678610e+05</td>
      <td>93.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>9.883590e+05</td>
      <td>103.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>1.052009e+06</td>
      <td>113.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>1.096973e+06</td>
      <td>606.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Creating a function to apply to the data set to get high and low billing amounts 
def amount_paid(x):
    if x > 99:
        return 1
    else:
        return 0
```


```python
#Splitting the columns to get the zip code to see what is the highest paid town
df['Zip'] = df.siteaddress.str.split('VT | NH | Vermont', expand = True)[1]
```


```python
#Looking at each town in pandas profiling
#profile = ProfileReport(df, title = 'Mojo Profiling')
profile
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    Input In [38], in <cell line: 3>()
          1 #Looking at each town in pandas profiling
          2 #profile = ProfileReport(df, title = 'Mojo Profiling')
    ----> 3 profile


    NameError: name 'profile' is not defined



```python
#First entry in the dataset had an out of range 
df["datetime"] = pd.to_datetime(df.date, errors = 'coerce')
#Dropping the na values that were created in the datetime column 
#1st entry dropped showed 0020-02...
df = df.dropna()
```


```python
#Indexing the dataset
Indexed = df.set_index(['datetime'])

```


```python
#Creating a subset for the time series analysis 
billamount = Indexed[['billamount']]
```


```python
# Creating an input variable for desired range of dates 
#2020-1-15
start_date = input('Put start date in this format YYYY-MM-DD:')
end_date = input('Put end date in this format YYYY-MM-DD:')
```

    Put start date in this format YYYY-MM-DD:2020-01-01
    Put end date in this format YYYY-MM-DD:2023-01-01



```python
#Creating a filter on the dataframe from the inputs above
mask = (df['datetime'] > start_date) & (df['datetime'] <= end_date)
selected_range = df.loc[mask]
```


```python
#indexing the selected ranges 
selected_range_indexed = selected_range.set_index(['datetime'])
```


```python
#Subsetting the selected ranges cumulative summary 
selected_range_indexed = selected_range_indexed[['billamount']].cumsum()
```


```python
#Changing the Figure size
rcParams['figure.figsize'] = 16,6
```


```python
#Creating a cumulative subset to put into matplotlib
cumulative_revenue = Indexed.billamount.cumsum()
```


```python
#Plotting the cumulative revenue of the whole business 
plt.ylabel('Revenue')
plt.xlabel('Days')
plt.title('Overall Cumulative Revenue')
plt.plot(selected_range_indexed)
```




    [<matplotlib.lines.Line2D at 0x7fdd612975b0>]




    
![png](output_31_1.png)
    


![image-2.png](attachment:image-2.png)

![image.png](attachment:image.png)

![image.png](attachment:image.png)

![image.png](attachment:image.png)


```python
#Storing the orgininal ddataframe 
Index_2=Indexed
```


```python
from datetime import *
```


```python
Indexed.date = pd.to_datetime(Indexed.date)
```


```python
#Creating the current date time and converting the date column to a datetime 
Indexed['today'] = datetime.now()
```


```python
#Calculating the difference between current date and spray to get the difference 
Indexed['Difference'] = Indexed['today'].sub(Indexed['date'], axis=0)
```


```python
#Subsetting the two columns I am interested in being accountnum and days 
difference = Indexed[['account', 'Difference']]
```


```python
#Creating a difference string to split and extract the days
difference['days'] = difference['Difference'].astype(str)
#Splitting the string we just made to get the index 0 = days
difference['days']= difference.days.str.split(' ', expand = True)[0]
```

    /var/folders/d5/yv3yty4s3y33ty4r_pc546j80000gn/T/ipykernel_26378/2624201271.py:2: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      difference['days'] = difference['Difference'].astype(str)
    /var/folders/d5/yv3yty4s3y33ty4r_pc546j80000gn/T/ipykernel_26378/2624201271.py:4: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      difference['days']= difference.days.str.split(' ', expand = True)[0]



```python
#Transforming the days to a float to use math operators on 
difference['days'] = difference.days.astype(float)

```

    /var/folders/d5/yv3yty4s3y33ty4r_pc546j80000gn/T/ipykernel_26378/372786923.py:2: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      difference['days'] = difference.days.astype(float)



```python
#Grouping the customer in a new final dataset to keep a the original
customers = df.groupby('account').min()
```


```python
#This will be used as the final data set we will create categories off of 
customers[["min_days","days"]] = difference.groupby('account').min()
```


```python
#creating a function to show customer chirn 
def churn_prediction(b):
    if b > 200:
        return 1 # 1 is used for people who have churned
    else:
        return 0
```


```python
customers['churn'] = customers.days.apply(churn_prediction)
```


```python
customers
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>siteaddress</th>
      <th>Billname</th>
      <th>date</th>
      <th>eventname</th>
      <th>billamount</th>
      <th>Zip</th>
      <th>datetime</th>
      <th>min_days</th>
      <th>year</th>
      <th>days</th>
      <th>churn</th>
    </tr>
    <tr>
      <th>account</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>891489</th>
      <td>X</td>
      <td>X</td>
      <td>2020-06-12 00:00:00</td>
      <td>Synthetic Barrier Spray</td>
      <td>115</td>
      <td>05359</td>
      <td>2020-06-12</td>
      <td>64 days 15:27:09.441171</td>
      <td>64.0</td>
      <td>64.0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>896665</th>
      <td>X</td>
      <td>X</td>
      <td>2020-06-13 00:00:00</td>
      <td>Botanical Barrier 3 Week</td>
      <td>123</td>
      <td>05672-05672</td>
      <td>2020-06-13</td>
      <td>50 days 15:27:09.441171</td>
      <td>50.0</td>
      <td>50.0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>905074</th>
      <td>X</td>
      <td>X</td>
      <td>2020-06-19 00:00:00</td>
      <td>Accelerated Service 1</td>
      <td>123</td>
      <td>05445</td>
      <td>2020-06-19</td>
      <td>34 days 15:27:09.441171</td>
      <td>34.0</td>
      <td>34.0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>905637</th>
      <td>X</td>
      <td>X</td>
      <td>2020-06-26 00:00:00</td>
      <td>Accelerated Service 1</td>
      <td>173</td>
      <td>05452</td>
      <td>2020-06-26</td>
      <td>49 days 15:27:09.441171</td>
      <td>49.0</td>
      <td>49.0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>906190</th>
      <td>X</td>
      <td>X</td>
      <td>2020-06-26 00:00:00</td>
      <td>Accelerated Service 1</td>
      <td>113</td>
      <td>05672</td>
      <td>2020-06-26</td>
      <td>63 days 15:27:09.441171</td>
      <td>63.0</td>
      <td>63.0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1094609</th>
      <td>X</td>
      <td>X</td>
      <td>2022-08-03 00:00:00</td>
      <td>Synthetic Barrier Spray</td>
      <td>83</td>
      <td>05452</td>
      <td>2022-08-03</td>
      <td>64 days 15:27:09.441171</td>
      <td>64.0</td>
      <td>64.0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1095582</th>
      <td>X</td>
      <td>X</td>
      <td>2022-08-12 00:00:00</td>
      <td>Synthetic Barrier Spray</td>
      <td>93</td>
      <td>03281</td>
      <td>2022-08-12</td>
      <td>52 days 15:27:09.441171</td>
      <td>52.0</td>
      <td>52.0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1095648</th>
      <td>X</td>
      <td>X</td>
      <td>2022-08-09 00:00:00</td>
      <td>Special Event Spray</td>
      <td>188</td>
      <td>03583</td>
      <td>2022-08-09</td>
      <td>83 days 15:27:09.441171</td>
      <td>83.0</td>
      <td>83.0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1096639</th>
      <td>X</td>
      <td>X</td>
      <td>2022-08-19 00:00:00</td>
      <td>Synthetic Barrier Spray</td>
      <td>93</td>
      <td>03303</td>
      <td>2022-08-19</td>
      <td>49 days 15:27:09.441171</td>
      <td>49.0</td>
      <td>49.0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1096973</th>
      <td>X</td>
      <td>X</td>
      <td>2022-08-16 00:00:00</td>
      <td>Special Event Spray</td>
      <td>200</td>
      <td>05482</td>
      <td>2022-08-16</td>
      <td>76 days 15:27:09.441171</td>
      <td>76.0</td>
      <td>76.0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>408 rows × 11 columns</p>
</div>




```python
customers[customers['billamount']]
```


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    Input In [102], in <cell line: 1>()
    ----> 1 customers[customers['billamount'] < 93 & customers['billamount'] > 0]


    File ~/opt/anaconda3/lib/python3.9/site-packages/pandas/core/generic.py:1527, in NDFrame.__nonzero__(self)
       1525 @final
       1526 def __nonzero__(self):
    -> 1527     raise ValueError(
       1528         f"The truth value of a {type(self).__name__} is ambiguous. "
       1529         "Use a.empty, a.bool(), a.item(), a.any() or a.all()."
       1530     )


    ValueError: The truth value of a Series is ambiguous. Use a.empty, a.bool(), a.item(), a.any() or a.all().



```python

```
