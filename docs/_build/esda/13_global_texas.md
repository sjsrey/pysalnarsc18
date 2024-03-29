---
redirect_from:
  - "/esda/13-global-texas"
interact_link: content/esda/13_global_texas.ipynb
kernel_name: python3
has_widgets: false
title: 'Global Spatial Autocorrelation'
prev_page:
  url: /esda/intro
  title: 'Exploratory Spatial Data Analysis'
next_page:
  url: /esda/12_sp_corr
  title: 'Interactive Spatial Autocorrelation'
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


# Global Spatial Autocorrelation




<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
%matplotlib inline
import pandas as pd
import numpy as np
import mapclassify as mc
import libpysal

```
</div>

</div>



A well-used functionality in PySAL is the use of PySAL to conduct exploratory spatial data analysis. This notebook will provide an overview of ways to conduct exploratory spatial analysis in Python. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import matplotlib.pyplot as plt

import geopandas as gpd
shp_link = "data/texas.shp"
tx = gpd.read_file(shp_link)
hr10 = mc.Quantiles(tx.HR90, k=10)
f, ax = plt.subplots(1, figsize=(9, 9))
tx.assign(cl=hr10.yb).plot(column='cl', categorical=True, \
        k=10, cmap='OrRd', linewidth=0.1, ax=ax, \
        edgecolor='white', legend=True)
ax.set_axis_off()
plt.title("HR90 Deciles")
plt.show()

```
</div>

</div>



## Spatial Autocorrelation

Visual inspection of the map pattern for HR90 deciles allows us to search for spatial structure. If the spatial distribution of the rates was random, then we should not see any clustering of similar values on the map. However, our visual system is drawn to the darker clusters in the south west as well as the east, and a concentration of the lighter hues (lower homicide rates) moving north to the pan handle.

Our brains are very powerful pattern recognition machines. However, sometimes they can be too powerful and lead us to detect false positives, or patterns where there are no statistical patterns. This is a particular concern when dealing with visualization of irregular polygons of differning sizes and shapes.

The concept of *spatial autocorrelation* relates to the combination of two types of similarity: spatial similarity and attribute similarity. Although there are many different measures of spatial autocorrelation, they all combine these two types of simmilarity into a summary measure.

Let's use PySAL to generate these two types of similarity measures.

### Spatial Similarity

We have already encountered spatial weights in a previous notebook. In spatial autocorrelation analysis, the spatial weights are used to formalize the notion of spatial similarity. As we have seen there are many ways to define spatial weights, here we will use queen contiguity:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
W = libpysal.weights.Queen.from_shapefile("data/texas.shp")
W.transform = 'r'

```
</div>

</div>



### Attribute Similarity

So the spatial weight between counties $i$ and $j$ indicates if the two counties are neighbors (i.e., geographically similar). What we also need is a measure of attribute similarity to pair up with this concept of spatial similarity.
The **spatial lag** is a derived variable that accomplishes this for us. For county $i$ the spatial lag is defined as:
$$HR90Lag_i = \sum_j w_{i,j} HR90_j$$





<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
HR90Lag = libpysal.weights.lag_spatial(W, data.HR90)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
HR90LagQ10 = mc.Quantiles(HR90Lag, k=10)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
f, ax = plt.subplots(1, figsize=(9, 9))
tx.assign(cl=HR90LagQ10.yb).plot(column='cl', categorical=True, \
        k=10, cmap='OrRd', linewidth=0.1, ax=ax, \
        edgecolor='white', legend=True)
ax.set_axis_off()
plt.title("HR90 Spatial Lag Deciles")

plt.show()

```
</div>

</div>



The decile map for the spatial lag tends to enhance the impression of value similarity in space. However, we still have the challenge of visually associating the value of the homicide rate in a county with the value of the spatial lag of rates for the county. The latter is a weighted average of homicide rates in the focal county's neighborhood.

To complement the geovisualization of these associations we can turn to formal statistical measures of spatial autocorrelation.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
HR90 = data.HR90
b,a = np.polyfit(HR90, HR90Lag, 1)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
f, ax = plt.subplots(1, figsize=(9, 9))

plt.plot(HR90, HR90Lag, '.', color='firebrick')

 # dashed vert at mean of the last year's PCI
plt.vlines(HR90.mean(), HR90Lag.min(), HR90Lag.max(), linestyle='--')
 # dashed horizontal at mean of lagged PCI
plt.hlines(HR90Lag.mean(), HR90.min(), HR90.max(), linestyle='--')

# red line of best fit using global I as slope
plt.plot(HR90, a + b*HR90, 'r')
plt.title('Moran Scatterplot')
plt.ylabel('Spatial Lag of HR90')
plt.xlabel('HR90')
plt.show()

```
</div>

</div>



## Global Spatial Autocorrelation



In PySAL, commonly-used analysis methods are very easy to access. For example, if we were interested in examining the spatial dependence in `HR90` we could quickly compute a Moran's $I$ statistic:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import esda
I_HR90 = esda.Moran(data.HR90.values, W)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
I_HR90.I, I_HR90.p_sim

```
</div>

</div>



Thus, the $I$ statistic is $0.859$ for this data, and has a very small $p$ value. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
b # note I is same as the slope of the line in the scatterplot

```
</div>

</div>



We can visualize the distribution of simulated $I$ statistics using the stored collection of simulated statistics:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
I_HR90.sim[0:5]

```
</div>

</div>



A simple way to visualize this distribution is to make a KDEplot (like we've done before), and add a rug showing all of the simulated points, and a vertical line denoting the observed value of the statistic:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib inline

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
sns.kdeplot(I_HR90.sim, shade=True)
plt.vlines(I_HR90.sim, 0, 0.5)
plt.vlines(I_HR90.I, 0, 10, 'r')
plt.xlim([-0.15, 0.15])

```
</div>

</div>



Instead, if our $I$ statistic were close to our expected value, `I_HR90.EI`, our plot might look like this:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
sns.kdeplot(I_HR90.sim, shade=True)
plt.vlines(I_HR90.sim, 0, 1)
plt.vlines(I_HR90.EI+.01, 0, 10, 'r')
plt.xlim([-0.15, 0.15])

```
</div>

</div>



The result of applying Moran's I is that we conclude the map pattern is not spatially random, but instead there is a signficant spatial association in homicide rates in Texas counties in 1990.

This result applies to the map as a whole, and is sometimes referred to as "global spatial autocorrelation". Later we turn to a local analysis where the attention shifts to detection of hot spots, cold spots and spatial outliers.



## Exercises

1. Repeat the global analysis for the years 1960, 70, 80 and compare the results to what we found in 1990.




<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python

















```
</div>

</div>



## Solutions



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
np.random.seed(12345)
I_pv = [esda.Moran(data[var].values, W).p_sim for var in ['HR60', 'HR70', 'HR80', 'HR90']]
I_pv

```
</div>

</div>

