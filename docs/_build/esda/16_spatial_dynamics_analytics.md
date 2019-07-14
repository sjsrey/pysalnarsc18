---
redirect_from:
  - "/esda/16-spatial-dynamics-analytics"
interact_link: content/esda/16_spatial_dynamics_analytics.ipynb
kernel_name: python3
has_widgets: false
title: 'Spatial Dynamics Analytics'
prev_page:
  url: /esda/14_local_texas
  title: 'Local Autocorrelation Analysis'
next_page:
  url: /esda/17_spatial_dynamics_visualization
  title: 'Spatial Dynamics Visualization'
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


# Spatial dynamics analytics
* Dynamics of spatial autocorrelation 
* Markov-based methods
    * Classic Markov
    * Spatial Markov
    * LISA Markov



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import matplotlib
import numpy as np
import geopandas as gpd
import pandas as pd
import libpysal
import matplotlib.pyplot as plt
%matplotlib inline
import seaborn as sns

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
pdUS_real = pd.read_csv("data/US_state_pci_constant09_1929_2009.csv")
data_table = gpd.read_file(libpysal.examples.get_path('us48.shp'))
complete_table = data_table.merge(pdUS_real,left_on='STATE_NAME',right_on='Name')
complete_table.head()

```
</div>

</div>



## Dynamics of Spatial Dependence



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
names = pdUS_real["Name"].values
years = range(1929,2010)
pd_pci_real = pdUS_real[list(map(str,years))]
pd_pci_real.index = names
pd_pci_real.head()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
pci_real = pd_pci_real.values.T
pci_real.shape

```
</div>

</div>



Prepare the spatial weight matrix - queen contiguity



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
w = libpysal.io.open(libpysal.examples.get_path("states48.gal")).read()
w.transform = 'R'

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import esda
mits = [esda.moran.Moran(cs, w) for cs in pci_real]
res = np.array([(mi.I, mi.EI, mi.seI_norm, mi.sim[974]) for mi in mits])
fig, ax = plt.subplots(nrows=1, ncols=1,figsize = (10,5) )
ax.plot(years, res[:,0], label='Moran\'s I')
#plot(years, res[:,1], label='E[I]')
ax.plot(years, res[:,1]+1.96*res[:,2], label='Upper bound',linestyle='dashed')
ax.plot(years, res[:,1]-1.96*res[:,2], label='Lower bound',linestyle='dashed')
ax.set_title("Global spatial autocorrelation for annual US per capita incomes",fontdict={'fontsize':15})
ax.set_xlim([1929,2009])
ax.legend()

```
</div>

</div>



## Markov-based methods
* Role of space in shaping per capita income dynamics



Spatial Markov - consider the impacts of regions' income levels on their neighbors in the following time period



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
mean = pci_real.mean(axis=1)
mean.shape = (81,1)
rpci_real = pci_real / mean

```
</div>

</div>



Discretization



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
pooled_rpci_real = rpci_real.flatten()
sns.kdeplot(pooled_rpci_real,shade=True)


```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
pooled_n = len(pooled_rpci_real)
pooled_rpci_real.sort()
plt.axvline(pooled_rpci_real[int(pooled_n * 0.2)],color="r")

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
sns.kdeplot(pooled_rpci_real,shade=True)
plt.axvline(pooled_rpci_real[int(pooled_n * 0.2)],color="r")
plt.axvline(pooled_rpci_real[int(pooled_n * 0.4)],color="r")
plt.axvline(pooled_rpci_real[int(pooled_n * 0.6)],color="r")
plt.axvline(pooled_rpci_real[int(pooled_n * 0.8)],color="r")

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import giddy
smarkov = giddy.markov.Spatial_Markov(rpci_real.T, w, fixed = True, k = 5)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
giddy.markov.Spatial_Markov?

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
smarkov.summary()

```
</div>

</div>



Steady state distributions



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
smarkov.s

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
smarkov.S

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
smarkov.F

```
</div>

</div>



LISA Markov - consider the joint transitions of regions' and neighbors' income levels

* Markov state space={1(HH), 2(LH), 3(LL), 4(HL)}



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
giddy.markov.LISA_Markov?

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
lm = giddy.markov.LISA_Markov(pci_real.T, w)
lm.classes

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
lm.p

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
lm.steady_state

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
giddy.ergodic.fmpt(lm.p)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
lm.chi_2

```
</div>

</div>

