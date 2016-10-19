
#### General

[![Build Status](https://travis-ci.org/pat-s/oddsratio.svg?branch=master)](https://travis-ci.org/pat-s/oddsratio) [![Project Status: Active – The project has reached a stable, usable state and is being actively developed.](http://www.repostatus.org/badges/latest/active.svg)](http://www.repostatus.org/#active) [![codecov](https://codecov.io/gh/pat-s/oddsratio/branch/master/graph/badge.svg)](https://codecov.io/gh/pat-s/oddsratio) [![minimal R version](https://img.shields.io/badge/R%3E%3D-3.0.0-6666ff.svg)](https://cran.r-project.org/) [![Last-changedate](https://img.shields.io/badge/last%20change-2016--10--19-yellowgreen.svg)](/commits/master)

#### CRAN

[![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/oddsratio)](http://cran.r-project.org/package=oddsratio) [![Downloads](http://cranlogs.r-pkg.org/badges/oddsratio?color=brightgreen)](http://www.r-pkg.org/pkg/oddsratio) [![Rdoc](http://www.rdocumentation.org/badges/version/oddsratio)](http://www.rdocumentation.org/packages/oddsratio)

#### Github

[![Github Releases](https://img.shields.io/github/downloads/pat-s/oddsratio/latest/total.svg)](https://github.com/pat-s/oddsratio) [![packageversion](https://img.shields.io/badge/Package%20version-0.2.0.9000-orange.svg?style=flat-square)](commits/master)

oddsratio
=========

Functions for calculation and plotting of odds ratios of Generalized Additive (Mixed) Models and Generalized Linear (Mixed) Models with a binomial response variable (i.e. logistic regression models).

Examples
--------

### GLM

Odds ratio calculation of predictors `gre` & `gpa` of a fitted model `fit.glm` with increment steps of 380 and 5, respectively.
For factor variables (here: `rank` with 4 levels), automatically all odds ratios corresponding to the base level (here: `rank1`) are returned including their respective confident intervals. The default level is 95%. However, other levels can be specified with the param `CI`. Data source: <http://www.ats.ucla.edu/stat/r/dae/logit.htm>

``` r
library(oddsratio)
dat <- read.csv("http://www.ats.ucla.edu/stat/data/binary.csv")
dat$rank <- factor(dat$rank)
fit.glm <- glm(admit ~ gre + gpa + rank, data = dat, family = "binomial")

calc.oddsratio.glm(data = dat, model = fit.glm, 
                   incr = list(gre = 380, gpa = 5, CI = 0.95))
```

    ##   predictor oddsratio CI.low (2.5 %) CI.high (97.5 %)          increment
    ## 1       gre     2.364          1.054            5.396                380
    ## 2       gpa    55.712          2.229         1511.282                  5
    ## 3     rank2     0.509          0.272            0.945 Indicator variable
    ## 4     rank3     0.262          0.132            0.512 Indicator variable
    ## 5     rank4     0.212          0.091            0.471 Indicator variable

### GAM

For GAMs, the calculation of odds ratio is different. Due to its non-linear definition, odds ratios do only apply to specific value changes and are not constant throughout the whole value range of the predictor as for GLMs. Hence, odds ratios of GAMs can only be computed for one predictor at a time by holding all other predictors at a fixed value while changing the value of the specific predictor. Confident intervals are currently fixed to the 95% level for GAMs. Data source: `?mgcv::predict.gam()`

Here, the usage of `calc.oddsratio.gam()` is shown by calculating odds ratios of pred `x2` for a 20% steps across the whole value range of the predictor.

``` r
suppressPackageStartupMessages(library(mgcv))
set.seed(1234)
n <- 200
sig <- 2
dat <- gamSim(1, n = n,scale = sig, verbose = FALSE)
dat$x4 <- as.factor(c(rep("A", 50), rep("B", 50), rep("C", 50), rep("D", 50)))
fit.gam <- mgcv::gam(y ~ s(x0) + s(I(x1^2)) + s(x2) + offset(x3) + x4, data = dat)

calc.oddsratio.gam(data = dat, model = fit.gam, pred = "x2", 
                   percentage = 20, slice = TRUE)
```

    ##   predictor value1 value2 perc1 perc2    oddsratio CI.low (2.5%)
    ## 1        x2  0.001  0.200     0    20 2.510768e+03  1.091683e+03
    ## 2        x2  0.200  0.400    20    40 2.870699e-02  2.621879e-02
    ## 3        x2  0.400  0.599    40    60 5.761210e-01  5.556941e-01
    ## 4        x2  0.599  0.799    60    80 6.032289e-02  5.789875e-02
    ## 5        x2  0.799  0.998    80   100 4.063187e-01  7.469151e-01
    ##   CI.high (97.5%)
    ## 1    5.774533e+03
    ## 2    3.143133e-02
    ## 3    5.972988e-01
    ## 4    6.284853e-02
    ## 5    2.210357e-01

If you want to compute a single odds ratio for specific values, simply set param `slice = FALSE`:

``` r
calc.oddsratio.gam(data = dat, model = fit.gam, 
                   pred = "x2", values = c(0.099, 0.198))
```

    ##   predictor value1 value2 oddsratio CI.low (2.5%) CI.high (97.5%)
    ## 1        x2  0.099  0.198  23.32353      23.30424        23.34283

From version 0.3.0 on (Github version 0.2.0.9000) you can now do the following things:

-   Plot smoothing functions of GAM(M)s

``` r
plot_smooth.gam(fit.gam, pred = "x2", title = "Predictor 'x2'")
```

![](README_files/figure-markdown_github/unnamed-chunk-6-1.png)

-   Insert the calculated odds ratios into the smoothing function

``` r
plot.object <- plot_smooth.gam(fit.gam, pred = "x2", title = "Predictor 'x2'")
or.object <- calc.oddsratio.gam(data = dat, model = fit.gam, 
                   pred = "x2", values = c(0.099, 0.198))

plot <- add.oddsratio.into.plot(plot.object, or.object, height.or = 5, x.shift = 0.04)
plot
```

![](README_files/figure-markdown_github/unnamed-chunk-7-1.png)

If you want to insert multiple odds ratios, just go on!

``` r
or.object2 <- calc.oddsratio.gam(data = dat, model = fit.gam, 
                   pred = "x2", values = c(0.4, 0.6))

add.oddsratio.into.plot(plot, or.object2, height.or = 2.5, x.shift = 0.024, 
               col.line = "green4", col.text = "green4")
```

![](README_files/figure-markdown_github/unnamed-chunk-8-1.png)

Installation
------------

Install from CRAN:

``` r
install.packages("oddsratio")
```

Get the development version from Github:

``` r
devtools::install_github("pat-s/oddsratio", build_vignettes = TRUE)
```

Examples
--------

To see full examples, please see the examples in the respective help pages `?calc.oddsratio.gam` and `?calc.oddsratio.gam` or take a look at the package vignette.
