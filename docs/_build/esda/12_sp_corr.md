---
redirect_from:
  - "/esda/12-sp-corr"
interact_link: content/esda/12_sp_corr.ipynb
kernel_name: python3
has_widgets: false
title: 'Interactive Spatial Autocorrelation'
prev_page:
  url: /esda/13_global_texas
  title: 'Global Spatial Autocorrelation'
next_page:
  url: /esda/14_local_texas
  title: 'Local Autocorrelation Analysis'
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


## Note
Interact widget does not currently work in jupyter-lab.

Need to run this notebook after starting `jupyter notebook`.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import pysal as ps
import numpy as np
import matplotlib.pyplot as plt
from scipy.linalg import inv

```
</div>

</div>



# Interactive spatial autocorrelation



This notebook illustrates the concept of spatial autocorrelation using the new interactivity in IPython. The data generating process (DGP) considered here is the following:



1. $u = \lambda Wu + \epsilon$

1. $u - \lambda Wu = \epsilon$

1. $u (I - \lambda W) = \epsilon$

1. $u = (I - \lambda W)^{-1} \epsilon$



Where `u` is a vector spatially autocorrelated, `W` is a spatial weights matrix as you could created with `PySAL`, and $\epsilon$ is an i.i.d. random vector.



To implement the previous DGP, the simple method `draw_map` (actual code pasted at the bottom of the notebook, so make sure to run that cell beforehand) creates a random vector with degree of spatial autocorrelation $\lambda$ and allocates it to a lattice geography, where every pixel is assumed to be an area with a value. Right next to it, the function also displays the Moran's scatter plot. Both map and plot depend on the $\lambda$ parameter that controls the degree of spatial autocorrelation.

Here's a static version of the function:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
draw_map(0.95)

```
</div>

</div>



Now, we can make the $\lambda$ value change in an interactive way:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
from ipywidgets import interact, interactive, fixed, interact_manual
import ipywidgets as widgets
interact(draw_map, lamb=(-0.9, 0.9));

```
</div>

</div>



# Actual plotting function



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
def draw_map(lamb):
    s = 20
    n = s**2
    w = ps.lat2W(s, s, rook=False)
    w.transform = 'R'
    e = np.random.random((n, 1))
    u = inv(np.eye(n) - lamb * w.full()[0])
    u = np.dot(u, e)
    ul = ps.lag_spatial(w, u)
    u = (u - u.mean()) / np.std(u)
    ul = (ul - ul.mean()) / np.std(ul)
    gu = u.reshape((s, s))
    # Figure
    f = plt.figure(figsize=(9, 4))
    ax1 = f.add_subplot(121)
    ax1.matshow(gu, cmap=plt.cm.YlGn)
    ax1.set_frame_on(False)
    ax1.axes.get_xaxis().set_visible(False)
    ax1.axes.get_yaxis().set_visible(False)
    #---
    ax2 = f.add_subplot(122)
    sc = ax2.scatter(u, ul, linewidth=0)
    ols = ps.spreg.OLS(ul, u)
    tag = "b = %.3f"%ols.betas[1][0]
    ax2.plot(u, ols.predy, c='red', label=tag)
    ax2.axvline(0, c='0.5')
    ax2.axhline(0, c='0.5')
    ax2.legend()
    plt.xlabel('u')
    plt.ylabel('Wu')
    plt.suptitle("$\lambda$ = %.2f"%lamb)
    plt.show()

```
</div>

</div>

