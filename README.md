# Spatial Data Analysis with PySAL @NARSC2018

### Instructors

- [Sergio Rey](http://sergerey.org) - University of California, Riverside
- [Wei Kang](http://spatial.ucr.edu/peopleKang.html) - University of California, Riverside

![journeys2016-18](figs/readmefigs/quad.png)
---

This repository contains the materials and instructions for the PySAL workshop at [NARSC 2018](http://www.narsc.org/newsite/conference/workshops-and-tutorials/).


## Schedule


* 8:00-10:00
  * Overview of PySAL and workshop
  * Introductions
  * Installation
  * Jupyter notebooks
  * Python primer
* 10:00-10:30
  * Coffee Break
* 10:30-12:30
  * Spatial data processing
  * Choropleth mapping and geovisualization
  * Spatial weights
* 12:30-1:30
  * Lunch
* 1:30-3:00
  * Global spatial autocorrelation
  * Local spatial autocorrelation
  * Spatial inequality analysis
* 3:00-3:30
  * Coffee Break  
* 3:30-5:00
  * Geodemographics and regionalization
  * Spatial dynamics
  * Spatial regression
  
## Obtaining Workshop Materials

If you are familiar with GitHub, you should clone or fork this GitHub repository to a specific directory. Cloning can be done by:

```bash
git clone https://github.com/sjsrey/pysalnarsc18.git
```

If you are not using git, you can grab the workshop materials as a zip file by pointing your browser to (https://github.com/sjsrey/pysalnarsc18.git) and clicking on the green *Clone or download* button in the upper right.

![download](figs/readmefigs/download.png)

Extract the downloaded zip file to a working directory.

## Installation

We will be using a number of Python packages for geospatial analysis.


An easy way to install all of these packages is to use a Python distribution such as [Anaconda](https://www.anaconda.com/download/#macos). In this workshop we will be using **Python 3.7** so please download that version of Anaconda.

![anaconda](figs/readmefigs/anaconda.png)

Once you have downloaded Anaconda, start a terminal and navigate to the directory of the downloaded/ cloned materials. For example, if the materials now live in the directory ```/Users/weikang/Downloads/pysalnarsc18-master```, you need to navigate to that directory from the terminal (using command ```cd```):

![directory](figs/readmefigs/directory.png)

Once we have done that, run:

```bash
conda-env create -f workshop.yml
```

This will build a conda environment that sandboxes the installation of the required packages for this workshop so we don't break anything in your computer's system Python (if it has one).

This may take 10-15 minutes to complete depending on the speed of your network connection.

Once this completes, you can activate the workshop environment with:

* on Mac, Linux
```bash
conda activate workshop18
```
* on Windows:
```bash
activate workshop18
```

Next, you will want to test your installation with:
```bash
 jupyter-nbconvert --execute --ExecutePreprocessor.timeout=120 check_workshop.ipynb
```

You should see something like:
```bash
[NbConvertApp] Converting notebook check_workshop.ipynb to html
[NbConvertApp] Executing notebook with kernel: python3
[NbConvertApp] Writing 386650 bytes to check_workshop.html
```

Open check_workshop.html in a browser, and scroll all the way down, you should see something like:

![htmlout](figs/readmefigs/htmlout.png)

You should also see a new file in the current directory called `inc.png` that contains a map looking something line:

![incmap](figs/readmefigs/inc.png)

If you do see the above, you are ready for the tutorial. If not, please contact either of us for help.
