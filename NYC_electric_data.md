# NYC Electric Data


## Day 4


```python
import pandas as pd
data = pd.read_csv('Value_of_Energy_Cost_Savings_Program_Savings_for_Businesses.csv')
```

## Inspect Data


```python
print(f"Shape of data is: {data.shape}")
print('\n================================================================\n')
print(f"Columns in data:\n {list(data.columns)}")
```

    Shape of data is: (2363, 31)
    
    ================================================================
    
    Columns in data:
     ['Period', 'Company Name', 'company contact', 'company email', 'company phone', 'Address', 'City', 'State', 'Postcode', 'Industry', 'Industry descr', 'Company Type', 'Current fulltime', 'Job created', 'Job retain', 'Effective Date', 'Total Savings', 'Savings from beginning receiving benefits', 'Gas Savings', 'Cogen savings', 'Electric Savings', 'Borough', 'Latitude', 'Longitude', 'Community Board', 'Council District', 'BIN', 'BBL', 'Census Tract (2020)', 'Neighborhood Tabulation Area (NTA) (2020)', 'month']
    

### 1. How many different companies are represented in the data set?


```python
print(f"There are {data['Company Name'].nunique()} different companies are represented in the data set.")
```

    There are 787 different companies are represented in the data set.
    

### 2. What is the total number of jobs created for businesses in Queens?


```python
boroughs_grouped_jobs = data.groupby(['Borough'])['Job created'].count()

print(f"There have been a total of {boroughs_grouped_jobs['QUEENS']} jobs created in Queens.")
```

    There have been a total of 83 jobs created in Queens.
    

### 3. How many different unique email domains names are there in the data set? 


```python
print(f"There are {data['company email'].describe()['unique']} unique email domains in the data set.")
```

    There are 760 unique email domains in the data set.
    

### 4. Considering only NTAs with at least 5 listed businesses, what is the average total savings and the total jobs created for   each NTA? 


```python
#find counts for all NTAs 
nta_val_counts = data['Neighborhood Tabulation Area (NTA) (2020)'].value_counts()

#create list of all NTAs with at least five businesses
nta_to_keep = [nta for nta,count in dict(nta_val_counts).items() if count >= 5]

#filter data to only keep rows with NTAs found above
data_nta_5 = data[data['Neighborhood Tabulation Area (NTA) (2020)'].isin(nta_to_keep)] 

#group filtered df by NTA
nta_grouped_df = data_nta_5.groupby(['Neighborhood Tabulation Area (NTA) (2020)'])

#generate avg total savings and total jobs for each NTA in groupby object
savings_jobs_by_NTA = nta_grouped_df.agg({'Total Savings': lambda x: round(x.mean(),2),
                   'Job created': 'count'})


```

### 5. Save your result for the previous question as a CSV file.


```python
display(savings_jobs_by_NTA)

#savings_jobs_by_NTA.to_csv('Savings&JobsCreated_by_NTA.csv', sep = ',')
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
      <th>Total Savings</th>
      <th>Job created</th>
    </tr>
    <tr>
      <th>Neighborhood Tabulation Area (NTA) (2020)</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>BK0101</th>
      <td>10367.96</td>
      <td>3</td>
    </tr>
    <tr>
      <th>BK0102</th>
      <td>12599.75</td>
      <td>2</td>
    </tr>
    <tr>
      <th>BK0103</th>
      <td>19150.92</td>
      <td>0</td>
    </tr>
    <tr>
      <th>BK0104</th>
      <td>21158.25</td>
      <td>18</td>
    </tr>
    <tr>
      <th>BK0201</th>
      <td>15102.04</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>QN1305</th>
      <td>166379.35</td>
      <td>0</td>
    </tr>
    <tr>
      <th>QN1306</th>
      <td>21160.51</td>
      <td>0</td>
    </tr>
    <tr>
      <th>SI0106</th>
      <td>6338.25</td>
      <td>2</td>
    </tr>
    <tr>
      <th>SI0107</th>
      <td>113610.16</td>
      <td>2</td>
    </tr>
    <tr>
      <th>SI0204</th>
      <td>605464.84</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>72 rows × 2 columns</p>
</div>


# Visualizations

## Day 5


```python
#import visualization packages
import matplotlib.pyplot as plt
import seaborn as sns
```

### 1. Create scatter plot of jobs created versus average savings. Use both a standard and a logarithmic scale for the average savings.


```python
#Standard scale in scatter plot , name of dataset is savings_jobs_by_NTA

sns.scatterplot(data = savings_jobs_by_NTA, x = 'Job created', y = 'Total Savings')
plt.xlabel('Jobs Created')
plt.ylabel('Average Total Savings (USD)')
plt.title('Average Total Savings vs Jobs Created')
plt.show()
```


    
![png](output_17_0.png)
    



```python
#Log scale
sns.scatterplot(data = savings_jobs_by_NTA, x = 'Job created', y = 'Total Savings', color = 'g')
plt.yscale('log')
plt.xlabel('Jobs Created')
plt.ylabel('Average Savings (USD)')
plt.title('Logarithmic Average Total Savings vs Jobs Created')
plt.show()
```


    
![png](output_18_0.png)
    


### 2. Create histogram of the log of the average total savings.


```python
#drop all vals where Total Savings == 0
savings_without_0 = savings_jobs_by_NTA[savings_jobs_by_NTA['Total Savings'] > 0]


sns.histplot(data = savings_without_0, x = 'Total Savings', log_scale = True, color = 'c')
plt.title('Count of Log Average Total Savings')
plt.ylabel('Number of Businesses')
plt.xlabel('Average Total Savings (USD)')

plt.show()
```


    
![png](output_20_0.png)
    


### 3. Create line plot of the total jobs created for each month.


```python
##use original dataset

#create month column
#print(data['Effective Date'].describe())

data['month'] = data['Effective Date'].apply(lambda x: int(str(x)[:2]))

#group job counts by month and sum counts
monthly_job_counts = data.groupby(['month'])['Job created'].sum()

#construct plot using groupby object
sns.lineplot(data = monthly_job_counts)

plt.xticks(ticks= range(1,13,1), labels =['Jan', 'Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'],
           rotation = 45)
plt.xlabel('Month')
plt.ylabel('Number of Jobs Created')
plt.title('Jobs Created Per Month')
plt.show()
```


    
![png](output_22_0.png)
    

