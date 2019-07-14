---
redirect_from:
  - "/esda/14-local-texas"
interact_link: content/esda/14_local_texas.ipynb
kernel_name: python3
has_widgets: false
title: 'Local Autocorrelation Analysis'
prev_page:
  url: /esda/12_sp_corr
  title: 'Interactive Spatial Autocorrelation'
next_page:
  url: /esda/16_spatial_dynamics_analytics
  title: 'Spatial Dynamics Analytics'
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


# Local Spatial Autocorrelation Analysis





<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import matplotlib.pyplot as plt
%matplotlib inline
import libpysal as ps
import mapclassify as mc
import pandas as pd
import numpy as np
import geopandas as gpd
import esda
#from pysal.contrib.viz import mapping as maps

```
</div>

</div>



First, let's read in some data:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
data = gpd.read_file("data/texas.shp")

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
data.head()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
hr10 = mc.Quantiles(data.HR90, k=10)
f, ax = plt.subplots(1, figsize=(9, 9))
data.assign(cl=hr10.yb).plot(column='cl', categorical=True, \
        k=10, cmap='OrRd', linewidth=0.1, ax=ax, \
        edgecolor='white', legend=True)
ax.set_axis_off()
plt.title("HR90 Deciles")
plt.show()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
f, ax = plt.subplots(1, figsize=(9, 9))
data.plot(column='HR90', scheme="quantiles", \
        k=10, cmap='OrRd', linewidth=0.1, ax=ax, \
        edgecolor='white', legend=True)
ax.set_axis_off()
plt.title("HR90 Deciles")
plt.show()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
W = ps.weights.Queen.from_shapefile("data/texas.shp")
W.transform = 'r'

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
HR90Lag = ps.weights.lag_spatial(W, data.HR90)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
HR90LagQ10 = mc.Quantiles(HR90Lag, k=10)
HR90LagQ10

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



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
I_HR90 = esda.Moran(data.HR90.values, W)

```
</div>

</div>



## Local Autocorrelation Statistics



In addition to the Global autocorrelation statistics, PySAL/esda has many local autocorrelation statistics. Let's compute a local Moran statistic for the same data shown above:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
LMo_HR90 = esda.Moran_Local(data.HR90.values, W)

```
</div>

</div>



Now, instead of a single $I$ statistic, we have an *array* of local $I_i$ statistics, stored in the `.Is` attribute, and p-values from the simulation are in `p_sim`. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
LMo_HR90.Is[0:10], LMo_HR90.p_sim[0:10]

```
</div>

</div>



We can adjust the number of permutations used to derive every *pseudo*-$p$ value by passing a different `permutations` argument:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
LMo_HR90 = esda.Moran_Local(data.HR90.values, W, permutations=9999)

```
</div>

</div>



In addition to the typical clustermap, a helpful visualization for LISA statistics is a Moran scatterplot with statistically significant LISA values highlighted. 

This is very simple, if we use the same strategy we used before:

First, construct the spatial lag of the covariate:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
Lag_HR90 = ps.weights.lag_spatial(W, data.HR90.values)
HR90 = data.HR90.values

```
</div>

</div>



Then, we want to plot the statistically-significant LISA values in a different color than the others. To do this, first find all of the statistically significant LISAs. Since the $p$-values are in the same order as the $I_i$ statistics, we can do this in the following way



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
sigs = HR90[LMo_HR90.p_sim <= .001]
W_sigs = Lag_HR90[LMo_HR90.p_sim <= .001]
insigs = HR90[LMo_HR90.p_sim > .001]
W_insigs = Lag_HR90[LMo_HR90.p_sim > .001]

```
</div>

</div>



Then, since we have a lot of points, we can plot the points with a statistically insignficant LISA value lighter using the `alpha` keyword. In addition, we would like to plot the statistically significant points in a dark red color. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
b,a = np.polyfit(HR90, Lag_HR90, 1)

```
</div>

</div>



Matplotlib has a list of [named colors](http://matplotlib.org/examples/color/named_colors.html) and will interpret colors that are provided in hexadecimal strings:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
plt.plot(sigs, W_sigs, '.', color='firebrick')
plt.plot(insigs, W_insigs, '.k', alpha=.2)
 # dashed vert at mean of the last year's PCI
plt.vlines(HR90.mean(), Lag_HR90.min(), Lag_HR90.max(), linestyle='--')
 # dashed horizontal at mean of lagged PCI
plt.hlines(Lag_HR90.mean(), HR90.min(), HR90.max(), linestyle='--')

# red line of best fit using global I as slope
plt.plot(HR90, a + b*HR90, 'r')
plt.text(s='$I = %.3f$' % I_HR90.I, x=50, y=15, fontsize=18)
plt.title('Moran Scatterplot')
plt.ylabel('Spatial Lag of HR90')
plt.xlabel('HR90')

```
</div>

</div>



We can also make a LISA map of the data. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
sig = LMo_HR90.p_sim < 0.05

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
sig.sum()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
hotspots = LMo_HR90.q==1 * sig

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
hotspots.sum()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
coldspots = LMo_HR90.q==3 * sig

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
coldspots.sum()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
data.HR90[hotspots]

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
data[hotspots]

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
from matplotlib import colors
hmap = colors.ListedColormap(['grey', 'red'])
f, ax = plt.subplots(1, figsize=(9, 9))
tx.assign(cl=hotspots*1).plot(column='cl', categorical=True, \
        k=2, cmap=hmap, linewidth=0.1, ax=ax, \
        edgecolor='grey', legend=True)
ax.set_axis_off()
plt.show()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
data.HR90[coldspots]

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
cmap = colors.ListedColormap(['grey', 'blue'])
f, ax = plt.subplots(1, figsize=(9, 9))
tx.assign(cl=coldspots*1).plot(column='cl', categorical=True, \
        k=2, cmap=cmap, linewidth=0.1, ax=ax, \
        edgecolor='black', legend=True)
ax.set_axis_off()
plt.show()


```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
from matplotlib import colors
hcmap = colors.ListedColormap(['grey', 'red','blue'])
hotcold = hotspots*1 + coldspots*2
f, ax = plt.subplots(1, figsize=(9, 9))
tx.assign(cl=hotcold).plot(column='cl', categorical=True, \
        k=2, cmap=hcmap,linewidth=0.1, ax=ax, \
        edgecolor='black', legend=True)
ax.set_axis_off()
plt.show()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import seaborn as sns
sns.kdeplot(data.HR90)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
sns.kdeplot(data.HR90)
sns.kdeplot(data.HR80)
sns.kdeplot(data.HR70)
sns.kdeplot(data.HR60)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
data.HR90.mean()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
data.HR90.median()

```
</div>

</div>



## Exercises

1. Repeat the local analysis for the years 1960, 70, 80 and compare the results to what we found in 1990.
2. How many counties are hot spots in each of the periods?



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



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
LMs = [esda.Moran_Local(data[var].values, W) for var in ['HR60', 'HR70', 'HR80', 'HR90']]


```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
#sig = LMo_HR90.p_sim < 0.05
#hotspots = LMo_HR90.q==1 * sig

hotspots = np.array([ (LM.p_sim < 0.05)*(LM.q==1) for LM in LMs]).T

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
hotspots.sum(axis=0)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
hotspots.sum(axis=1)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
hotspots.sum(axis=1).max()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
data[hotspots.sum(axis=1)==2][['NAME','HR60','HR70', 'HR80','HR90']]

```
</div>

</div>

