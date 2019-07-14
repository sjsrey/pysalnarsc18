---
redirect_from:
  - "/processing/07-data-processing"
interact_link: content/processing/07_data_processing.ipynb
kernel_name: python3
has_widgets: false
title: 'Geoprocessing spatial data'
prev_page:
  url: /processing/intro
  title: 'Geoprocessing and Visualization'
next_page:
  url: /processing/08_routes
  title: 'Geoprocessing Exercise'
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


# Spatial Data Processing with libpysal,  Pandas and Geopandas




<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
# By convention, we use these shorter names
import libpysal as ps
import pandas as pd
import numpy as np
import geopandas as gpd

```
</div>

</div>



## libpysal



libpysal has a command that it uses to get the paths of its example datasets. Let's work with a commonly-used dataset first. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
ps.examples.available()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
ps.examples.explain('us_income')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
csv_path = ps.examples.get_path('usjoin.csv')
csv_path

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
f = ps.io.open(csv_path)
f.header[0:10]

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
y2009 = f.by_col('2009')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
y2009[0:10]

```
</div>

</div>



### Working with shapefiles



We can also work with local files outside the built-in examples.

To read in a shapefile, we will need the path to the file.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
shp_path = 'data/texas.shp'
print(shp_path)

```
</div>

</div>



Then, we open the file using the `ps.io.open` command:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
f = ps.io.open(shp_path)

```
</div>

</div>



`f` is what we call a "file handle." That means that it only *points* to the data and provides ways to work with it. By itself, it does not read the whole dataset into memory. To see basic information about the file, we can use a few different methods. 

For instance, the header of the file, which contains most of the metadata about the file:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
f.header

```
</div>

</div>



To actually read in the shapes from memory, you can use the following commands:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
f.by_row(14) # gets the 14th shape from the file

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
all_polygons = f.read() # reads in all polygons from memory

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
len(all_polygons)

```
</div>

</div>



So, all 254 polygons have been read in from file. These are stored in libpysal shape objects, which can be used by libpysal and can be converted to other Python shape objects.

They typically have a few methods. So, since we've read in polygonal data, we can get some properties about the polygons. Let's just have a look at the first polygon:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
all_polygons[0:5]

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
all_polygons[0].centroid #the centroid of the first polygon

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
all_polygons[0].area

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
all_polygons[0].perimeter

```
</div>

</div>



While in the Jupyter Notebook, you can examine what properties an object has by using the tab key.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
polygon = all_polygons[0]

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
polygon. #press tab when the cursor is right after the dot

```
</div>

</div>



### Working with Data Tables



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
dbf_path = "data/texas.dbf"
print(dbf_path)

```
</div>

</div>



When you're working with tables of data, like a `csv` or `dbf`, you can extract your data in the following way. Let's open the dbf file we got the path for above.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
f = ps.io.open(dbf_path)

```
</div>

</div>



Just like with the shapefile, we can examine the header of the dbf file.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
f.header

```
</div>

</div>



So, the header is a list containing the names of all of the fields we can read.
If we just wanted to grab the data of interest, `HR90`, we can use either `by_col` or `by_col_array`, depending on the format we want the resulting data in:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
HR90 = f.by_col('HR90')
print(type(HR90).__name__, HR90[0:5])
HR90 = f.by_col_array('HR90')
print(type(HR90).__name__, HR90[0:5])

```
</div>

</div>



As you can see, the `by_col` function returns a list of data, with no shape. It can only return one column at a time:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
HRs = f.by_col('HR90', 'HR80')

```
</div>

</div>



This error message is called a "traceback," as you see in the top right, and it usually provides feedback on why the previous command did not execute correctly. Here, you see that one-too-many arguments was provided to `__call__`, which tells us we cannot pass as many arguments as we did to `by_col`.

If you want to read in many columns at once and store them to an array, use `by_col_array`:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
HRs = f.by_col_array('HR90', 'HR80')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
HRs[0:10]

```
</div>

</div>



It is best to use `by_col_array` on data of a single type. That is, if you read in a lot of columns, some of them numbers and some of them strings, all columns will get converted to the same datatype:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
allcolumns = f.by_col_array(['NAME', 'STATE_NAME', 'HR90', 'HR80'])

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
allcolumns

```
</div>

</div>



Note that the numerical columns, `HR90` & `HR80` are now considered strings, since they show up with the single tickmarks around them, like `'0.0'`.



These methods work similarly for `.csv` files as well.



## Geopandas & pandas



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
shp_path = ps.examples.get_path('NAT.shp')
data_table = gpd.read_file(shp_path)

```
</div>

</div>



This reads in *the entire database table* and adds a column to the end, called `geometry`, that stores the geometries read in from the shapefile. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
data_table.head()

```
</div>

</div>



The `read_file` function only works on shapefile/dbf pairs. If you need to read in data using CSVs, use pandas directly:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
csv_path = ps.examples.get_path('usjoin.csv')
usjoin = pd.read_csv(csv_path)
#usjoin = ps.pdio.read_files(csv_path) #will not work, not a shp/dbf pair

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
gpd.read_file(csv_path)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
usjoin.head()

```
</div>

</div>



The nice thing about working with geopandas dataframes is that they have very powerful baked-in support for relational-style queries. By this, I mean that it is very easy to find things like:

The number of counties in each state:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
data_table.groupby("STATE_NAME").size()

```
</div>

</div>



Or, to get the rows of the table that are in Arizona, we can use the `query` function of the dataframe:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
data_table.query('STATE_NAME == "Arizona"')

```
</div>

</div>



Behind the scenes, this uses a fast vectorized library, `numexpr`, to essentially do the following. 

First, compare each row's `STATE_NAME` column to `'Arizona'` and return `True` if the row matches:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
data_table.STATE_NAME == 'Arizona'

```
</div>

</div>



Then, use that to filter out rows where the condition is true:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
data_table[data_table.STATE_NAME == 'Arizona']

```
</div>

</div>



We might need this behind the scenes knowledge when we want to chain together conditions, or when we need to do spatial queries. 

This is because spatial queries are somewhat more complex. Let's say, for example, we want all of the counties in the US to the West of `-121` longitude. We need a way to express that question. Ideally, we want something like:

```
SELECT
        *
FROM
        data_table
WHERE
        x_centroid < -121
```



So, let's refer to an arbitrary polygon in the the dataframe's geometry column as `poly`. We can acquire the centroid of the polygon as a shapely Point. Then we can acquire the coordinates of the point. The longitude is the first element of the pair of the coordinates. 

Then, applying this condition to each geometry, we get the same kind of filter we used above to grab only counties in Arizona:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
type(data_table.geometry[0].centroid)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
list(data_table.geometry[0].centroid.coords)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
data_table.geometry.apply(lambda x: x.centroid.coords[0][0] < -121)\
                   .head()

```
</div>

</div>



If we use this as a filter on the table, we can get only the rows that match that condition, just like we did for the `STATE_NAME` query:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
data_table[data_table.geometry.apply(lambda x: x.centroid.coords[0][0] < -119)].head()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
len(data_table[data_table.geometry.apply(lambda x: x.centroid.coords[0][0] < -119)]) #how many west of -119?

```
</div>

</div>



## Other types of spatial queries

Everybody knows the following statements are true:

1. If you head directly west from Reno, Nevada, you will shortly enter California.
2. San Diego is in California.

But what does this tell us about the location of San Diego relative to Reno?

Or for that matter, how many counties in California are to the east of Reno?







<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
geom = data_table.query('(NAME == "Washoe") & (STATE_NAME == "Nevada")').geometry

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
geom.values[0]

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
geom.values[0].centroid

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
lon,lat = geom.values[0].centroid.coords[0]

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
lon

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
cal_counties = data_table.query('(STATE_NAME=="California")')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
cal_counties[cal_counties.geometry.apply(lambda x: x.centroid.coords[0][0] > lon)]

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
len(cal_counties)

```
</div>

</div>



This works on any type of spatial query. 

For instance, if we wanted to find all of the counties that are within a threshold distance from an observation's centroid, we can do it in the following way. 

But first, we need to handle distance calculations on the earth's surface.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
from math import radians, sin, cos, sqrt, asin

def gcd(loc1, loc2, R=3961):
    """Great circle distance via Haversine formula
    
    Parameters
    ----------
    
    loc1: tuple (long, lat in decimal degrees)
    
    loc2: tuple (long, lat in decimal degrees)
    
    R: Radius of the earth (3961 miles, 6367 km)
    
    Returns
    -------
    great circle distance between loc1 and loc2 in units of R
    
    
    Notes
    ------
    Does not take into account non-spheroidal shape of the Earth
    
    
    
    >>> san_diego = -117.1611, 32.7157
    >>> austin = -97.7431, 30.2672
    >>> gcd(san_diego, austin)
    1155.474644164695
  
    
    """
    lon1, lat1 = loc1
    lon2, lat2 = loc2
    dLat = radians(lat2 - lat1)
    dLon = radians(lon2 - lon1)
    lat1 = radians(lat1)
    lat2 = radians(lat2)
 
    a = sin(dLat/2)**2 + cos(lat1)*cos(lat2)*sin(dLon/2)**2
    c = 2*asin(sqrt(a))

    return R * c
 
def gcdm(loc1, loc2):
    return gcd(loc1, loc2)

def gcdk(loc1, loc2):
    return gcd(loc1, loc2, 6367 )

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
san_diego = -117.1611, 32.7157
austin = -97.7431, 30.2672
gcd(san_diego, austin)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
gcdk(san_diego, austin)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
loc1 = (-117.1611, 0.0)
loc2 = (-118.1611, 0.0)
gcd(loc1, loc2)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
loc1 = (-117.1611, 45.0)
loc2 = (-118.1611, 45.0)
gcd(loc1, loc2)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
loc1 = (-117.1611, 89.0)
loc2 = (-118.1611, 89.0)
gcd(loc1, loc2)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
lats = range(0, 91)
onedeglon = [ gcd((-117.1611,lat),(-118.1611,lat)) for lat in lats]

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import matplotlib.pyplot as plt
%matplotlib inline
plt.plot(lats, onedeglon)
plt.ylabel('miles')
plt.xlabel('degree of latitude')
plt.title('Length of a degree of longitude')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
san_diego = -117.1611, 32.7157
austin = -97.7431, 30.2672
gcd(san_diego, austin)

```
</div>

</div>



Now we can use our distance function to pose distance-related queries on our data table.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
# Find all the counties with centroids within 50 miles of Austin
def near_target_point(polygon, target=austin, threshold=50):
    return gcd(polygon.centroid.coords[0], target) < threshold 

data_table[data_table.geometry.apply(near_target_point)]

```
</div>

</div>



### Moving in and out of the dataframe



Most things in PySAL will be explicit about what type their input should be. Most of the time, PySAL functions require either lists or arrays. This is why the file-handler methods are the default IO method in PySAL: the rest of the computational tools are built around their datatypes. 

However, it is very easy to get the correct datatype from Pandas or Geopandas using the `values` and `tolist` commands. 

`tolist()` will convert its entries to a list. But, it can only be called on individual columns (called `Series` in `pandas` documentation).

So, to turn the `NAME` column into a list:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
data_table.NAME.tolist()[0:10]

```
</div>

</div>



To extract many columns, you must select the columns you want and call their `.values` attribute. 

If we were interested in grabbing all of the `HR` variables in the dataframe, we could first select those column names:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
HRs = [col for col in data_table.columns if col.startswith('HR')]
HRs

```
</div>

</div>



We can use this to focus only on the columns we want:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
data_table[HRs].head()

```
</div>

</div>



With this, calling `.values` gives an array containing all of the entries in this subset of the table:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
data_table[['HR90', 'HR80']].values

```
</div>

</div>



## Exercises

1. Find the county with the western most centroid that is within 1000 miles of Austin.
2. Find the distance between Austin and that centroid.



## Solutions



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
austin_lon = austin[0]
west = data_table[data_table.geometry.apply(lambda x: x.centroid.coords[0][0] < austin_lon)]


```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
def near_target_point(polygon, target=austin, threshold=1000):
    return gcd(polygon.centroid.coords[0], target) <= threshold 

west_lt_1000 = west[west.geometry.apply(near_target_point)]

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
maxd = 0.
county = None
for i,row in west_lt_1000.iterrows():
    d = gcd(row['geometry'].centroid.coords[0], austin)
    if d > maxd:
        county = row['NAME']
        state = row['STATE_NAME']
        maxd = d

print('Centroid of %s, %s is %f miles west of Austin'%(county, state, maxd))

```
</div>

</div>

