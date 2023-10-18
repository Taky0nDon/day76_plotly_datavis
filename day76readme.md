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
or `pd._to_numeric(column)`

#datatype
Use `.describe()` on the dataframe or `.info()` on the dataframe or column

Achieve the same results with `.groupby()`

>If we take two of the columns, say Installs and the App name, we can count the number of entries per level of installations with `.groupby()` and `.count()`. However, because we are dealing with a non-numeric data type, the ordering is not helpful. The reason Python is not recognising our installs as numbers is because of the commas.

```python
# Exclude apps that cost over $250.
df_apps_sub250 = df_apps_num_price[df_apps_num_price.Price < 250]
# Check type of columns
df_apps_sub250.info()
# remove non-numeric characters from the Installs columns
df_apps_sub250.Installs = df_apps_sub250.Installs.str.replace(",", "")
# cast the Installs column to int64
df_apps_sub250.Installs = pd.to_numeric(df_apps_sub250.Installs)
# verify type conversion
df_apps_sub250.Installs.info()
# Count the number of apps for ever Installation level.
df_apps_sub250.groupby("Installs")["App"].count()

```
alt: `df_apps_clean[['App', 'Installs']].groupby('Installs').count()`

## Multiplying two columns
* You can use .mul(col2)  or *

```
df_apps_sub250["total revenue"] = pd.to_numeric(df_apps_clean_no_dupes["installs"].str.replace(",", "")) * df_apps_clean_no_dupes["price"]
df_apps_sub250.sort_values(by="Total Revenue", ascending=False).head(10)[df_apps_sub250.Category == "GAME"]
```

## Plotly Charts: Bar and Scatter

* Find the number of different categories:
`df_apps_clean.Category.nunique()`
* Find the 10 most common categories:
`top_10_categories = df_apps_sub250.value_counts(subset=df_apps_sub250.Category).sort_values(ascending=False)[:10]` or `df_apps_sub250.Category.value_counts()[:10]`

### [Bar Chart](https://plotly.com/python-api-reference/generated/plotly.express.bar.html?highlight=express%20bar#plotly.express.bar)

#### Vertical

`px.bar()`

```python
bar_fig = px.bar(x=top_10_categories.index, y=top_10_categories.values)
bar_fig.show()
```

#### Horizontal
```python
horizontal_bar = bar_fig.update_traces(orientation = 'h', x=top_10_categories.values, y=top_10_categories.index)
horizontal_bar.show()
```
### How often are apps downloaded in that category?


[.agg()](obsidian://open?vault=Obsidian%20Vault&file=100%20Days%20Of%20Code%2Fday74readme)
>
`.agg()` takes a dictionary as an argument. In this dictionary, we specify which operation we'd like to apply to each column. In our case, we just want to calculate the number of unique entries in the theme_id column by using our old friend, the `.nunique()` method.

```python
most_downloaded_cats = df_apps_sub250.groupby("Category").agg({"Installs": pd.Series.sum}).sort_values(by="Installs", ascending=True)


h_bar = px.bar(orientation='h', y=most_downloaded_cats.index, x=most_downloaded_cats.Installs.values)
h_bar.show()
```

### Create a Dataframe that has the number of installs in one column and the number of apps in the other


```python
df_inst_apps = df_apps_sub250.groupby("Category").agg({"Installs": pd.Series.sum, "App": pd.Series.count})
df_inst_apps.sort_values(by="Installs", ascending=False)
```

`.agg()` return a DataFrame from a groupby object.

### Show the install concentration with a scatter plot

```python
df_inst_apps = df_apps_sub250.groupby("Category").agg({"Installs": pd.Series.sum, "App": pd.Series.count})
df_inst_apps.sort_values(by="Installs", ascending=False, inplace=True)
scat = px.scatter(df_inst_apps, x="App", y="Installs", size="App",
                 color="Installs",
                 hover_name = df_inst_apps.index,
                 labels={"App": "Number of Apps (Lower=More Concentrated)"})
scat.update_layout(yaxis=dict(type='log'),
                  title="Category Concentration")

```


**Solution**
```python
cat_number = df_apps_clean.groupby('Category').agg({'App': pd.Series.count})
# Merge this with out earlier cat_instals dataframe
cat_merged_df = pd.merge(cat_number, category_installs, on='Category', how="inner")
# Sort the merged df
cat_merged_df.sort_values(by='Installs', ascending=False)
# Create the chart
scatter = px.scatter(cat_merged_df,
					x = 'App',
					y = 'Installs',
					title='Category Concentration',
					size='App',
					hover_name=cat_merged_df.index,
					color='Installs' 
					)
scatter.update_layout(xaxis_title="Number of Apps (Lower=More Concentrated)",
					  y_axis_title="Installs",
					  yaxis=dict(type='log')
					 )
scatter.show()

```

## Extracting Nested Data From a Column