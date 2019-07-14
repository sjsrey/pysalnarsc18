---
redirect_from:
  - "/esda/17-spatial-dynamics-visualization"
interact_link: content/esda/17_spatial_dynamics_visualization.ipynb
kernel_name: python3
has_widgets: false
title: 'Spatial Dynamics Visualization'
prev_page:
  url: /esda/16_spatial_dynamics_analytics
  title: 'Spatial Dynamics Analytics'
next_page:
  url: 
  title: ''
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


# Exploratory Spatial and Temporal Data Analysis (ESTDA) - Visualization
   



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import matplotlib
import numpy as np
import libpysal
import pandas as pd
import matplotlib.pyplot as plt
%matplotlib inline
import seaborn as sns

```
</div>

</div>



Load example dataset in **pysal**: nominal per capita incomes observed annually from 1929 to 2009 for the lower 48 US states. Downloaded from [US Bureau of Economic Analysis](https://www.bea.gov).



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
pdUS = pd.read_csv(libpysal.examples.get_path('usjoin.csv'))
pdUS.head()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
pdUS.info()

```
</div>

</div>



## Visualization

* Temporal dynamics



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
names = pdUS["Name"].values
names

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
years = range(1929,2010)
pd_pci = pdUS[list(map(str,years))]
pd_pci.head()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
pd_pci.index = names
pd_pci.head()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
pd_pci = pd_pci.T
pd_pci.head()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
pd_pci.plot(legend=None)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
order1929 = np.argsort(pdUS["1929"])
order2009 = np.argsort(pdUS["2009"])
names1929 = names[order1929[::-1]]
names2009 = names[order2009[::-1]]
first_last = np.vstack((names[order1929[::-1]],names[order2009[::-1]]))
from pylab import rcParams
sns.set_palette(sns.color_palette("Set1", 2010-1929))
rcParams['figure.figsize'] = 15,10
plt.plot(years,pd_pci.as_matrix())
#pd_pci.plot(legend=None)
for i in range(48):
    plt.text(1915,pd_pci.values.max()-500-(i*1159), names1929[i],fontsize=12)
    plt.text(2010.5,pd_pci.values.max()-500-(i*1159), names2009[i],fontsize=12)
plt.xlim((years[0], years[-1]))
plt.ylim((0, pd_pci.values.max()))
plt.ylabel("Per capita income (Nominal dollar)",fontsize=14,color="r")
plt.xlabel('Year',fontsize=12)
plt.title('Absolute Dynamics',fontsize=18)

```
</div>

</div>



* Distribution dynamics



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import seaborn as sns
plt.figure(figsize=(8,7))
sns.kdeplot(pdUS["1929"], color="b") 
sns.kdeplot(pdUS["2009"], color="R")
plt.legend()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
sns.set_palette(sns.color_palette("coolwarm", 2010-1929))
plt.figure(figsize=(10,8))
for i in range(2010-1929):
    sns.kdeplot(pd_pci.T[str(i+1929)],legend=False)
plt.xlabel("Per capita income (Nominal dollar)",fontsize=14,color="r")

```
</div>

</div>



Another way to visualize the dynamics of cross-sectional distributions - Joyplot

> **NOTE**: This material has been ported and adapted from [Joyplot](https://gist.github.com/sjsrey/f98977d78cdcacddc2f6a8891539cb80) by Serge Rey.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
from seaborn.distributions import _statsmodels_univariate_kde
# Source http://nbviewer.jupyter.org/gist/ljwolf/37c89ddb704c013debb9d205c3acb25b
def joyplot(data, ax=None, 
            flatten = .1, #rescale the height of each distribution to avoid overlap. If large, will flatten out each of the KDEs
            linecolor='k', 
            shadecolor='w',
            shade=True, 
            line_kws = None,
            kde_kws=None,
            fig_kws=None,
            shade_kws=None):
    line_kws = dict() if line_kws is None else line_kws
    kde_kws = (dict(kernel='gau', bw='scott',
                         gridsize=100, cut=3,
                         clip=None) if kde_kws is None else kde_kws)
    fig_kws = dict(figsize=(5,5)) if fig_kws is None else fig_kws
    shade_kws = (dict(alpha=.75, 
                          clip_on=True, 
                          zorder=1, 
                          color=None) if shade_kws is None else shade_kws)
    if kde_kws.get('clip',None) is None:
        kde_kws['clip'] = (-np.inf, np.inf)
    if ax is None:
        f,ax = plt.subplots(1,1, **fig_kws)
    T,N = data.shape
    dsupport = np.array([])
    for i, row in enumerate(data):
        x,y = _statsmodels_univariate_kde(row, **kde_kws)
        y = np.max(np.c_[np.zeros_like(y), y], axis=1)
        y = y/(flatten*y.max()) + i
        ax.plot(x,y,color=linecolor,**line_kws)
        #print(y)
        if shade:
            if shade_kws.get('color', None) is None:
                shade_kws['color'] = shadecolor
            ax.fill_between(x, i, y, 
                             **shade_kws)
        dsupport = np.concatenate((dsupport, x))
    ax.set_xlim(np.min(dsupport)*.75, np.max(dsupport)*1.25)
    return f,ax

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
array_pci = pd_pci.as_matrix().astype(float)
xmin = array_pci.min()
xmax = array_pci.max()
f,ax = joyplot(array_pci, shade=True, shadecolor='lightgrey', linecolor='k', fig_kws=dict(figsize=(15,10))) 
ax.set_xlim(0, xmax)
ax.legend(ncol=2, fontsize=16)
ax.set_ylabel("Year",fontsize=14)
ax.set_xlabel("Per capita income (Nominal Dollar)",fontsize=14,color="r")
ax.set_yticks(range(0,9*10,10))
ax.set_yticklabels(range(1929,2010,10))
plt.show()

```
</div>

</div>



### Per capita income dynamics in constant dollar 2009 - structural mobility

We need to adjust for price change over years for a valid temporal comparison. First we acquire [Historical Consumer Price Index for All Urban Consumers (CPI-U)](https://www.bls.gov/cpi/tables/historical-cpi-u-201709.pdf) from [US Bureau of labor Statistics](https://www.bls.gov/home.htm). 




<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
pd_cpi = pd.read_csv("data/CPI1913-2016.csv")
pd_cpi.head()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
pd_cpi.index = pd_cpi["year"].as_matrix()
pd_cpi = pd_cpi.drop(["year"],axis=1)
pd_cpi.head()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
pd_cpi2909 = pd_cpi.loc[years]
pd_cpi2909

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
deflator = (pd_cpi2909.loc[2009]/pd_cpi2909).T.values[0]
deflator

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
real_2909 = np.dot(np.diag(deflator),pd_pci.values)
real_2909

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
pd_real_2909 = pdUS.copy()
for year in years:
    pd_real_2909[str(year)] = real_2909[year-1929,:]
pd_real_2909.head()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
pd_real_2909.to_csv("data/US_state_pci_constant09_1929_2009.csv")

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
from pylab import rcParams
sns.set_palette(sns.color_palette("Set1", 2010-1929))
rcParams['figure.figsize'] = 15,10
plt.plot(years,real_2909)
#pd_pci.plot(legend=None)
for i in range(48):
    plt.text(1915,real_2909.max()-700-(i*1189), names1929[i],fontsize=12)
    plt.text(2010.5,real_2909.max()-700-(i*1189), names2009[i],fontsize=12)
plt.xlim((years[0], years[-1]))
plt.ylim((0, real_2909.max()))
plt.ylabel(r"$y_{i,t}$",fontsize=14)
plt.ylabel("Per capita income (Constant Dollar 2009)",fontsize=14,color="r")
plt.xlabel('Year',fontsize=12)


```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
sns.set_palette(sns.color_palette("coolwarm", 2010-1929))
plt.figure(figsize=(10,8))
plt.xlabel("Per capita income (Constant Dollar 2009)",fontsize=14,color="r")
for i in range(2010-1929):
    sns.kdeplot(real_2909[i],legend=False)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
xmin = real_2909.min()
xmax = real_2909.max()
f,ax = joyplot(real_2909, shade=True, shadecolor='lightgrey', linecolor='k', fig_kws=dict(figsize=(15,10))) 
ax.set_xlim(0, xmax)
ax.legend(ncol=2, fontsize=16)
ax.set_ylabel("Year",fontsize=14)
ax.set_xlabel("Per capita income (Constant Dollar 2009)",fontsize=14,color="r")
ax.set_yticks(range(0,9*10,10))
ax.set_yticklabels(range(1929,2010,10))
plt.show()

```
</div>

</div>



### Relative per capita income dynamics - exchange mobility



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
mean = pd_pci.values.mean(axis=1)
mean.shape = (len(mean),1)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
rpci = pd_pci.values/mean
rpci

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
from pylab import rcParams
sns.set_palette(sns.color_palette("Set1", 2010-1929))
rcParams['figure.figsize'] = 15,10
plt.plot(years,rpci)
#pd_pci.plot(legend=None)
for i in range(48):
    plt.text(1915,rpci.max()-(i*0.042), names1929[i],fontsize=12)
    plt.text(2010.5,rpci.max()-(i*0.042), names2009[i],fontsize=12)
plt.xlim((years[0], years[-1]))
plt.ylim((0, rpci.max()))
plt.ylabel("Relative Per capita income (mean-normalized)",fontsize=14,color="r")
plt.xlabel('Year',fontsize=12)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
sns.set_palette(sns.color_palette("coolwarm", 2010-1929))
plt.figure(figsize=(10,8))
for i in range(2010-1929):
    sns.kdeplot(rpci[i],legend=False)
plt.xlabel("Relative Per capita income (mean-normalized)",fontsize=14,color="r")

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
xmin = rpci.min()
xmax = rpci.max()
f,ax = joyplot(rpci, shade=True, shadecolor='lightgrey', linecolor='k', fig_kws=dict(figsize=(15,10))) 
ax.set_xlim(0, xmax)
ax.legend(ncol=2, fontsize=16)
ax.set_ylabel("Year",fontsize=14)
ax.set_xlabel("Relative Per capita income (mean-normalized)",fontsize=14,color="r")
ax.set_yticks(range(0,9*10,10))
ax.set_yticklabels(range(1929,2010,10))
plt.show()

```
</div>

</div>



### Spatial-temporal dynamics visualization



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import pysal
from pysal.contrib.viz import mapping as maps
data_table = pysal.pdio.read_files(libpysal.examples.get_path('us48.shp'))
#income_table = pd.read_csv(ps.examples.get_path("usjoin.csv"))
complete_table = data_table.merge(pdUS,left_on='STATE_NAME',right_on='Name')
complete_table.head()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
index_year = range(1929,2010,15)
fig, axes = plt.subplots(nrows=2, ncols=3,figsize = (15,7))
for i in range(2):
    for j in range(3):
        ax = axes[i,j]
        maps.geoplot(complete_table, col=str(index_year[i*3+j]),ax=ax,classi="Quantiles")
        ax.set_title('Per Capita Income %s Quintiles'%str(index_year[i*3+j]))
plt.tight_layout()

```
</div>

</div>

