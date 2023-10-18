#plotly

## Replicating app store analytics like those provided by App Annie or SensorTower

### Gain Insight:

1. How competitive different app categories (e.g., Games, Lifestyle, Weather) are
2. Which app category offers compelling opportunities based on its popularity
3. How many downloads you would give up by making your app paid vs. free
4. How much you can reasonably charge for a paid app
5. Which paid apps have had the highest revenue
6. How many paid apps will recoup their development costs based on their sales revenue

### Today We'll Learn

1. How to quickly remove duplicates
2. How to remove unwanted symbols and convert data into a numeric format
3. How to wrangle columns containing nested data with Pandas
4. How to create compelling data visualisations with the plotly library
5. Create vertical, horizontal and grouped bar charts
6. Create pie and donut charts for categorical data
7. Use colour scales to make beautiful scatter plots

## Data Exploration

rows: 10841
columns: 12
```
Index(['App', 'Category', 'Rating', 'Reviews', 'Size_MBs', 'Installs', 'Type',
       'Price', 'Content_Rating', 'Genres', 'Last_Updated', 'Android_Ver'],
      dtype='object')
```

use `df.sample(n)` to see a sample of n different rows

### Removing columns from a dataframe

https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.drop.html

`df_apps.drop(['Last_Updated', 'Android_Ver'], axis=1)`

### Counting NaN values


`df_apps.isna().sum().sum()`

the first sum gives the total NaN values in each column, the second gives the total NaN values in the whole dataframe
or create a subset of the dataframe based on where `.isna()` evaluates to True

```python
nan_rows = df_apps[df_apps.Rating.isna()]
```
### Remove all rows with NaN in the Rating column

`df_apps_clean = df_apps.dropna(axis=0, subset=['Rating'])`

### Detecting duplicates

https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.duplicated.html


```python
all_dupes = df_apps_clean[df_apps_clean.duplicated(keep=False)]
df_apps_clean[df_apps_clean.App == "Instagram"]
df_apps_clean_no_dupes = df_apps_clean.drop_duplicates(subset=['App', 'Type', 'Price'])
df_apps_clean_no_dupes[df_apps_clean_no_dupes.App == "Instagram"]

df_apps_clean.drop_duplicates(subset=["App"]).sort_values(by="Reviews", ascending=False).head(50).query("Type == 'Free'")
```

## Data Visualization with Plotly

Looking at the distribution of content ratings. use `.value_counts()` to count the occurrence of each rating.

```
df_apps_nd = df_apps_clean.drop_duplicates(subset=['App', 'Type', 'Price'])
df_apps_nd.Content_Rating.value_counts()
```

>Everyone           6621
>Teen                912
>Mature 17+          357
>Everyone 10+        305
>Adults only 18+       3
>Unrated               1
>Name: Content_Rating, dtype: int64

#### Import Plotly
`import plotly.express as px`
show the above as a [pie chart](https://plotly.com/python-api-reference/generated/plotly.express.pie.html):
```
px.pie(values=None, names=None)

fig = px.pie(df_apps_nd, values=df_apps_nd.Content_Rating.value_counts(), names=df_apps_nd.Content_Rating.unique(), title="Content Rating Distribution").show()
```

Use `update_traces()` to modify chart style
```python
fig.update_traces(textposition="outside",
textinfo="percent+label")
fig.show()
```

## Converting a string column to int

1. remove non numeric characters with the string attribute:
`app_installs_no_comma = app_installs.str.replace(",", "")
`
2. cast as int
`app_installs_int = app_installs_no_comma.astype(int)`

#datatype
Use `.describe()` on the dataframe or `.info()` on the dataframe or column

Achieve the same results with `.groupby()`

## Multiplying two columns
* You can use .mul(col2)  or *

```
df_apps_sub250["Total Revenue"] = pd.to_numeric(df_apps_clean_no_dupes["Installs"].str.replace(",", "")) * df_apps_clean_no_dupes["Price"]
df_apps_sub250.sort_values(by="Total Revenue", ascending=False).head(10)[df_apps_sub250.Category == "GAME"]
```

