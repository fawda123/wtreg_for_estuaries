# WRTDStidal
Marcus W. Beck, beck.marcus@epa.gov, James D. Hagy III, hagy.jim@epa.gov  

[![Travis-CI Build Status](https://travis-ci.org/fawda123/wtreg_for_estuaries.png?branch=master)](https://travis-ci.org/fawda123/wtreg_for_estuaries)

This is the development repository for the WRTDStidal package.  Functions within this package can be used to model chlorophyll time series from tidal datasets.  The approach follows on previous methods described in the [EGRET](https://github.com/USGS-R/EGRET) package developed by USGS for non-tidal waters.

The development version of this package can be installed as follows:


```r
install.packages('devtools')
library(devtools)
install_github('fawda123/WRTDStidal')
library(WRTDStidal)
```

### Using the functions

The adapation of WRTDS in tidal waters is designed to predict or normalize chlorophyll concentrations as a function of time, season, and salinity.  The functions have been developed following [S3 documentation](http://adv-r.had.co.nz/OO-essentials.html#s3), with specific methods for `tidal` objects.  The input data are typically a `data.frame` with rows as monthly observations (ascending time) and four columns as date, chlorophyll, salinity, and lower detection limit for chlorophyll.  The `chldat` dataset that accompanies this package shows the proper format for using the functions.


```r
# import data
data(chldat)

# data format
str(chldat)
```

```
## 'data.frame':	156 obs. of  4 variables:
##  $ date : Date, format: "2000-01-01" "2000-02-01" ...
##  $ chla : num  2.46 1.09 1.78 1.65 1.77 ...
##  $ salff: num  0.192 0.157 0.181 0.166 0.146 ...
##  $ lim  : num  0.875 0.875 0.875 0.875 0.875 ...
```

The quickest implementation of WRTDS with an input data frame is to use the `modfit` function which is a wrapoer for several other functions that complete specific tasks.  Executing the function will create a `tidal` object with multiple attributes.  The following text will also be printed in the console that describes current actions and progress. 


```
## Loading WRTDStidal
```

```r
# get wrtds results
res <- modfit(chldat)
```

```
## 
## Estimating interpolation grids for tau = 0.5, % complete...
## 
## 10 	20 	30 	40 	50 	60 	70 	80 	90 	100 	
## 
## Interpolating chlorophyll predictions
## 
## Normalizing chlorophyll predictions
```

The results include the original `data.frame` with additional columns for parameters used to fit the model, model predictions for specified conditional quantiles, and the respective normalized predictions.  The `modfit` function implements four individual functions which can be used separately to create the model. 


```r
# this is equivalent to running modfit
# modfit is a wrapper for tidal, wrtds, chlpred, and chlnorm functions

# pipes from the dplyr package are used for simplicity
library(dplyr)

res <- tidal(chldat) %>%  # creates a tidal object
  wrtds %>% # creates wrtds interpolation grids
  chlpred %>% # get predictions from grids
  chlnorm # get normalized predictions from grids
```

```
## 
## Estimating interpolation grids for tau = 0.5, % complete...
## 
## 10 	20 	30 	40 	50 	60 	70 	80 	90 	100 	
## 
## Interpolating chlorophyll predictions
## 
## Normalizing chlorophyll predictions
```

All arguments that apply to each of the four functions in the previous chunk can be passed to the `modfit` function to control parameters used to fit the WRTDS model.  Examples in the help file for `modfit` illustrate some of the more important arguments a user may consider.  Note that the `wins` argument applies to the `getwts` function that is implemented within the `wrtds` function.


```r
## fit the model and get predicted/normalized chlorophyll data
# default median fit
# grids predicted across salinity range with ten values
res <- modfit(chldat)

## fit different quantiles and smaller interpolation grid
res <- modfit(chldat, tau = c(0.2, 0.8), sal_div = 5)

## fit with different window widths
# half-window widths of one day, five years, and 0.3 salff
res <- modfit(chldat, wins = list(1, 5, 0.3))

## suppress console output
res <- modfit(chldat, trace = FALSE)
```

An additional dataset is included in this package that is used to illustrates examples after running `modfit`.  The `tidfit` dataset was created to predict and normalize chlorophyll concentrations for the tenth,  median, and ninetieth conditional quantile distributions.  It can be recreated from `chldat` using the following code.


```r
tidfit <- modfit(chldat, tau = c(0.1, 0.5, 0.9))
```

