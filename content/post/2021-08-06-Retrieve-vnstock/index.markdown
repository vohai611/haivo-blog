---
title: An R package to retrieve Vietnam stock data
summary: Make API request to get stock data from vndirect.com.vn and cafef.vn
image: "img/vnstockr.jpg"
author: Hai Vo
date: '2021-08-06'
slug: []
categories: [Project]
tags: [r-package, httr, furrr, crul]
---

<script src="{{< blogdown/postref >}}index_files/htmlwidgets/htmlwidgets.js"></script>
<script src="{{< blogdown/postref >}}index_files/plotly-binding/plotly.js"></script>
<script src="{{< blogdown/postref >}}index_files/typedarray/typedarray.min.js"></script>
<script src="{{< blogdown/postref >}}index_files/jquery/jquery.min.js"></script>
<link href="{{< blogdown/postref >}}index_files/crosstalk/css/crosstalk.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/crosstalk/js/crosstalk.min.js"></script>
<link href="{{< blogdown/postref >}}index_files/plotly-htmlwidgets-css/plotly-htmlwidgets.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/plotly-main/plotly-latest.min.js"></script>

## Update 10-2021

I just update the function `get_cefeF2()` which using async request (from `{crul}` package). The new function seem to be way faster compare with the old `get_cafeF()`.

``` r
# New
system.time(vnstockr::get_cafeF2("ACB", "1/1/2020", "1/1/2021"))
##    user  system elapsed 
##   1.246   0.013   1.832
# Old
system.time(vnstockr::get_cafeF("ACB", "1/1/2020", "1/1/2021"))
##    user  system elapsed 
##   1.162   0.146  14.703
```

## Why

It is convenience to import stock data directly into R because there are multiple advance tools in R allow you to analyzing or run machine learning model with time series such as `{modeltime}` `{fable}`.

The`{tidyquant}` package made it simple by let user get stock data in Yahoo Finance to R in the “tidy” format. However, most of Vietnamese company does not listed in Yahoo Finance, hence user must manually download the data and import it into R before take advantage of R packages.

To automate this boring task, I write a simple package that allow user to get the Vietnam stock data from cafef.vn and vndirect.com.vn. This package is the wrap around of `{httr}` package which send request to these two data sources and parse the respond to R data frame (tibble).

You can browse the package source code [here](https://github.com/vohai611/vnstockr)

## Install

To install this package, use:

``` r
devtools::install_github("https://github.com/vohai611/vnstockr")
```

## Usage

This package only contain two function `get_cafeF()` and `get_vndirect()`. User just need to specify stock code (symbol) and time frame. For example:

``` r
library(vnstockr)
acb <- get_vndirect('ACB', '1/5/2020', '1/5/2021')
acb |>
  head(5) 
```

    ## # A tibble: 5 × 25
    ##   code  date       time     floor type  basicPrice ceilingPrice floorPrice  open
    ##   <chr> <date>     <chr>    <chr> <chr>      <dbl>        <dbl>      <dbl> <dbl>
    ## 1 ACB   2021-04-29 15:04:05 HOSE  STOCK       33.8         36.2       31.4  33.8
    ## 2 ACB   2021-04-28 15:04:02 HOSE  STOCK       34           36.4       31.6  33.5
    ## 3 ACB   2021-04-27 15:04:02 HOSE  STOCK       33.3         35.6       31    33  
    ## 4 ACB   2021-04-26 15:04:04 HOSE  STOCK       33.4         35.7       31.1  33.4
    ## 5 ACB   2021-04-23 15:04:04 HOSE  STOCK       32.5         34.8       30.2  32.4
    ## # … with 16 more variables: high <dbl>, low <dbl>, close <dbl>, average <dbl>,
    ## #   adOpen <dbl>, adHigh <dbl>, adLow <dbl>, adClose <dbl>, adAverage <dbl>,
    ## #   nmVolume <dbl>, nmValue <dbl>, ptVolume <dbl>, ptValue <dbl>, change <dbl>,
    ## #   adChange <dbl>, pctChange <dbl>

Data retrieve are in the same “tidy” format as `{tidyquant}` package and could easily integrate with other R package quickly. For example, data could be visualized with `ggplot2`:

``` r
library(ggplot2)
library(plotly)
acb_plot <- acb |>
  ggplot(aes(date, close))+
  geom_line()+
  scale_x_date(date_breaks = "4 weeks")+
  theme_light()+
  theme(axis.text.x = element_text(angle = 45, hjust =1))+
  expand_limits(y = 0)+
  labs(title = "ACB stock price from 1/5/2020 to 1/5/2021\n", x= NULL)

ggplotly(acb_plot) |>
  config(displayModeBar = FALSE)
```

<div id="htmlwidget-1" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-1">{"x":{"data":[{"x":[18386,18387,18388,18389,18390,18393,18394,18395,18396,18397,18400,18401,18402,18403,18404,18407,18408,18409,18410,18411,18414,18415,18416,18417,18418,18421,18422,18423,18424,18425,18428,18429,18430,18431,18432,18435,18436,18437,18438,18439,18442,18443,18444,18445,18446,18449,18450,18451,18452,18453,18456,18457,18458,18459,18460,18463,18464,18465,18466,18467,18470,18471,18472,18473,18474,18477,18478,18479,18480,18481,18484,18485,18486,18487,18488,18491,18492,18493,18494,18495,18498,18499,18500,18501,18502,18505,18506,18508,18509,18512,18513,18514,18515,18516,18519,18520,18521,18522,18523,18526,18527,18528,18529,18530,18533,18534,18535,18536,18537,18540,18541,18542,18543,18544,18547,18548,18549,18550,18551,18554,18555,18556,18557,18558,18561,18562,18563,18564,18565,18568,18569,18570,18571,18572,18575,18576,18577,18578,18579,18582,18583,18584,18585,18586,18589,18590,18591,18592,18593,18596,18597,18605,18606,18607,18610,18611,18612,18613,18614,18617,18618,18619,18620,18621,18624,18625,18626,18627,18631,18632,18633,18634,18635,18638,18639,18640,18641,18642,18645,18646,18647,18648,18649,18652,18653,18654,18655,18656,18659,18660,18661,18662,18663,18666,18667,18675,18676,18677,18680,18681,18682,18683,18684,18687,18688,18689,18690,18691,18694,18695,18696,18697,18698,18701,18702,18703,18704,18705,18708,18709,18710,18711,18712,18715,18716,18717,18718,18719,18722,18723,18724,18725,18726,18729,18730,18731,18732,18733,18736,18737,18739,18740,18743,18744,18745,18746],"y":[20.3,20.3,20.8,21.2,21.4,21.7,21.8,21.7,21.4,21.7,21.8,22.2,22.2,22.4,22.4,22.6,23.3,22.8,22.9,22.9,25.1,24.8,25.5,25.3,25.2,25.4,25.5,25.6,24.5,24.5,23.3,23.8,23.7,23.8,24.4,24.1,24,23.8,23.7,23.6,22.9,22.8,23.2,23.1,23,23.6,23.7,23.9,24.3,24,23.9,24,24,24,24.8,24.5,24.6,24.4,24.2,23.2,21.8,22.9,22.2,22.6,22.5,23.2,23.7,24,23.8,23.7,23.8,25.4,25.4,25.5,25.3,25.8,25.6,25.6,20.8,21.2,21,21.1,21.2,21.5,21.4,21.2,21.1,21.2,21.1,20.8,20.6,20.9,20.9,20.8,21.2,21.3,21.2,21.5,21.7,22,22.3,22.5,22.2,22.2,22.6,22.4,22.5,22.5,23,23.6,24,23.4,23.2,23.5,23.3,23.4,24,24.5,24.7,25.3,25.3,25.2,25.6,25.6,24.8,24.4,23.8,23.9,24.1,24.5,25,25.5,24.9,25.1,25.4,25.3,25.4,26.4,26.5,26.2,27.2,27.2,27.2,27.3,27.3,27.3,27.3,27.4,27.2,27.2,27.3,28.55,27.95,28.2,28.2,28,28.25,28.3,28.4,28.95,28.75,28.1,27.3,27.95,27.8,27.7,27.8,28.1,28.65,29.35,29.9,29.9,30.2,30.35,30.1,29.85,29.85,30.25,29.95,27.9,28.3,28.95,28.65,28.1,27.8,27.5,25.6,26.8,26.65,27.35,28.1,28.2,28.5,27.5,28.55,29.4,29.15,31.1,31.6,31.7,31.2,32.4,33.05,33.25,33,33.25,32.4,32.45,31.8,32.05,32.75,33,32.95,33.5,33.45,33.5,33.95,33.8,33.4,33,32.45,32.05,32.45,33,33.3,33.3,33.85,34.65,34.75,34.75,34.4,34.4,34.4,35.15,34.6,34.6,33.85,33.1,33.7,33.6,32.5,33.4,33.3,34,33.8,34.65],"text":["date: 2020-05-04<br />close: 20.30","date: 2020-05-05<br />close: 20.30","date: 2020-05-06<br />close: 20.80","date: 2020-05-07<br />close: 21.20","date: 2020-05-08<br />close: 21.40","date: 2020-05-11<br />close: 21.70","date: 2020-05-12<br />close: 21.80","date: 2020-05-13<br />close: 21.70","date: 2020-05-14<br />close: 21.40","date: 2020-05-15<br />close: 21.70","date: 2020-05-18<br />close: 21.80","date: 2020-05-19<br />close: 22.20","date: 2020-05-20<br />close: 22.20","date: 2020-05-21<br />close: 22.40","date: 2020-05-22<br />close: 22.40","date: 2020-05-25<br />close: 22.60","date: 2020-05-26<br />close: 23.30","date: 2020-05-27<br />close: 22.80","date: 2020-05-28<br />close: 22.90","date: 2020-05-29<br />close: 22.90","date: 2020-06-01<br />close: 25.10","date: 2020-06-02<br />close: 24.80","date: 2020-06-03<br />close: 25.50","date: 2020-06-04<br />close: 25.30","date: 2020-06-05<br />close: 25.20","date: 2020-06-08<br />close: 25.40","date: 2020-06-09<br />close: 25.50","date: 2020-06-10<br />close: 25.60","date: 2020-06-11<br />close: 24.50","date: 2020-06-12<br />close: 24.50","date: 2020-06-15<br />close: 23.30","date: 2020-06-16<br />close: 23.80","date: 2020-06-17<br />close: 23.70","date: 2020-06-18<br />close: 23.80","date: 2020-06-19<br />close: 24.40","date: 2020-06-22<br />close: 24.10","date: 2020-06-23<br />close: 24.00","date: 2020-06-24<br />close: 23.80","date: 2020-06-25<br />close: 23.70","date: 2020-06-26<br />close: 23.60","date: 2020-06-29<br />close: 22.90","date: 2020-06-30<br />close: 22.80","date: 2020-07-01<br />close: 23.20","date: 2020-07-02<br />close: 23.10","date: 2020-07-03<br />close: 23.00","date: 2020-07-06<br />close: 23.60","date: 2020-07-07<br />close: 23.70","date: 2020-07-08<br />close: 23.90","date: 2020-07-09<br />close: 24.30","date: 2020-07-10<br />close: 24.00","date: 2020-07-13<br />close: 23.90","date: 2020-07-14<br />close: 24.00","date: 2020-07-15<br />close: 24.00","date: 2020-07-16<br />close: 24.00","date: 2020-07-17<br />close: 24.80","date: 2020-07-20<br />close: 24.50","date: 2020-07-21<br />close: 24.60","date: 2020-07-22<br />close: 24.40","date: 2020-07-23<br />close: 24.20","date: 2020-07-24<br />close: 23.20","date: 2020-07-27<br />close: 21.80","date: 2020-07-28<br />close: 22.90","date: 2020-07-29<br />close: 22.20","date: 2020-07-30<br />close: 22.60","date: 2020-07-31<br />close: 22.50","date: 2020-08-03<br />close: 23.20","date: 2020-08-04<br />close: 23.70","date: 2020-08-05<br />close: 24.00","date: 2020-08-06<br />close: 23.80","date: 2020-08-07<br />close: 23.70","date: 2020-08-10<br />close: 23.80","date: 2020-08-11<br />close: 25.40","date: 2020-08-12<br />close: 25.40","date: 2020-08-13<br />close: 25.50","date: 2020-08-14<br />close: 25.30","date: 2020-08-17<br />close: 25.80","date: 2020-08-18<br />close: 25.60","date: 2020-08-19<br />close: 25.60","date: 2020-08-20<br />close: 20.80","date: 2020-08-21<br />close: 21.20","date: 2020-08-24<br />close: 21.00","date: 2020-08-25<br />close: 21.10","date: 2020-08-26<br />close: 21.20","date: 2020-08-27<br />close: 21.50","date: 2020-08-28<br />close: 21.40","date: 2020-08-31<br />close: 21.20","date: 2020-09-01<br />close: 21.10","date: 2020-09-03<br />close: 21.20","date: 2020-09-04<br />close: 21.10","date: 2020-09-07<br />close: 20.80","date: 2020-09-08<br />close: 20.60","date: 2020-09-09<br />close: 20.90","date: 2020-09-10<br />close: 20.90","date: 2020-09-11<br />close: 20.80","date: 2020-09-14<br />close: 21.20","date: 2020-09-15<br />close: 21.30","date: 2020-09-16<br />close: 21.20","date: 2020-09-17<br />close: 21.50","date: 2020-09-18<br />close: 21.70","date: 2020-09-21<br />close: 22.00","date: 2020-09-22<br />close: 22.30","date: 2020-09-23<br />close: 22.50","date: 2020-09-24<br />close: 22.20","date: 2020-09-25<br />close: 22.20","date: 2020-09-28<br />close: 22.60","date: 2020-09-29<br />close: 22.40","date: 2020-09-30<br />close: 22.50","date: 2020-10-01<br />close: 22.50","date: 2020-10-02<br />close: 23.00","date: 2020-10-05<br />close: 23.60","date: 2020-10-06<br />close: 24.00","date: 2020-10-07<br />close: 23.40","date: 2020-10-08<br />close: 23.20","date: 2020-10-09<br />close: 23.50","date: 2020-10-12<br />close: 23.30","date: 2020-10-13<br />close: 23.40","date: 2020-10-14<br />close: 24.00","date: 2020-10-15<br />close: 24.50","date: 2020-10-16<br />close: 24.70","date: 2020-10-19<br />close: 25.30","date: 2020-10-20<br />close: 25.30","date: 2020-10-21<br />close: 25.20","date: 2020-10-22<br />close: 25.60","date: 2020-10-23<br />close: 25.60","date: 2020-10-26<br />close: 24.80","date: 2020-10-27<br />close: 24.40","date: 2020-10-28<br />close: 23.80","date: 2020-10-29<br />close: 23.90","date: 2020-10-30<br />close: 24.10","date: 2020-11-02<br />close: 24.50","date: 2020-11-03<br />close: 25.00","date: 2020-11-04<br />close: 25.50","date: 2020-11-05<br />close: 24.90","date: 2020-11-06<br />close: 25.10","date: 2020-11-09<br />close: 25.40","date: 2020-11-10<br />close: 25.30","date: 2020-11-11<br />close: 25.40","date: 2020-11-12<br />close: 26.40","date: 2020-11-13<br />close: 26.50","date: 2020-11-16<br />close: 26.20","date: 2020-11-17<br />close: 27.20","date: 2020-11-18<br />close: 27.20","date: 2020-11-19<br />close: 27.20","date: 2020-11-20<br />close: 27.30","date: 2020-11-23<br />close: 27.30","date: 2020-11-24<br />close: 27.30","date: 2020-11-25<br />close: 27.30","date: 2020-11-26<br />close: 27.40","date: 2020-11-27<br />close: 27.20","date: 2020-11-30<br />close: 27.20","date: 2020-12-01<br />close: 27.30","date: 2020-12-09<br />close: 28.55","date: 2020-12-10<br />close: 27.95","date: 2020-12-11<br />close: 28.20","date: 2020-12-14<br />close: 28.20","date: 2020-12-15<br />close: 28.00","date: 2020-12-16<br />close: 28.25","date: 2020-12-17<br />close: 28.30","date: 2020-12-18<br />close: 28.40","date: 2020-12-21<br />close: 28.95","date: 2020-12-22<br />close: 28.75","date: 2020-12-23<br />close: 28.10","date: 2020-12-24<br />close: 27.30","date: 2020-12-25<br />close: 27.95","date: 2020-12-28<br />close: 27.80","date: 2020-12-29<br />close: 27.70","date: 2020-12-30<br />close: 27.80","date: 2020-12-31<br />close: 28.10","date: 2021-01-04<br />close: 28.65","date: 2021-01-05<br />close: 29.35","date: 2021-01-06<br />close: 29.90","date: 2021-01-07<br />close: 29.90","date: 2021-01-08<br />close: 30.20","date: 2021-01-11<br />close: 30.35","date: 2021-01-12<br />close: 30.10","date: 2021-01-13<br />close: 29.85","date: 2021-01-14<br />close: 29.85","date: 2021-01-15<br />close: 30.25","date: 2021-01-18<br />close: 29.95","date: 2021-01-19<br />close: 27.90","date: 2021-01-20<br />close: 28.30","date: 2021-01-21<br />close: 28.95","date: 2021-01-22<br />close: 28.65","date: 2021-01-25<br />close: 28.10","date: 2021-01-26<br />close: 27.80","date: 2021-01-27<br />close: 27.50","date: 2021-01-28<br />close: 25.60","date: 2021-01-29<br />close: 26.80","date: 2021-02-01<br />close: 26.65","date: 2021-02-02<br />close: 27.35","date: 2021-02-03<br />close: 28.10","date: 2021-02-04<br />close: 28.20","date: 2021-02-05<br />close: 28.50","date: 2021-02-08<br />close: 27.50","date: 2021-02-09<br />close: 28.55","date: 2021-02-17<br />close: 29.40","date: 2021-02-18<br />close: 29.15","date: 2021-02-19<br />close: 31.10","date: 2021-02-22<br />close: 31.60","date: 2021-02-23<br />close: 31.70","date: 2021-02-24<br />close: 31.20","date: 2021-02-25<br />close: 32.40","date: 2021-02-26<br />close: 33.05","date: 2021-03-01<br />close: 33.25","date: 2021-03-02<br />close: 33.00","date: 2021-03-03<br />close: 33.25","date: 2021-03-04<br />close: 32.40","date: 2021-03-05<br />close: 32.45","date: 2021-03-08<br />close: 31.80","date: 2021-03-09<br />close: 32.05","date: 2021-03-10<br />close: 32.75","date: 2021-03-11<br />close: 33.00","date: 2021-03-12<br />close: 32.95","date: 2021-03-15<br />close: 33.50","date: 2021-03-16<br />close: 33.45","date: 2021-03-17<br />close: 33.50","date: 2021-03-18<br />close: 33.95","date: 2021-03-19<br />close: 33.80","date: 2021-03-22<br />close: 33.40","date: 2021-03-23<br />close: 33.00","date: 2021-03-24<br />close: 32.45","date: 2021-03-25<br />close: 32.05","date: 2021-03-26<br />close: 32.45","date: 2021-03-29<br />close: 33.00","date: 2021-03-30<br />close: 33.30","date: 2021-03-31<br />close: 33.30","date: 2021-04-01<br />close: 33.85","date: 2021-04-02<br />close: 34.65","date: 2021-04-05<br />close: 34.75","date: 2021-04-06<br />close: 34.75","date: 2021-04-07<br />close: 34.40","date: 2021-04-08<br />close: 34.40","date: 2021-04-09<br />close: 34.40","date: 2021-04-12<br />close: 35.15","date: 2021-04-13<br />close: 34.60","date: 2021-04-14<br />close: 34.60","date: 2021-04-15<br />close: 33.85","date: 2021-04-16<br />close: 33.10","date: 2021-04-19<br />close: 33.70","date: 2021-04-20<br />close: 33.60","date: 2021-04-22<br />close: 32.50","date: 2021-04-23<br />close: 33.40","date: 2021-04-26<br />close: 33.30","date: 2021-04-27<br />close: 34.00","date: 2021-04-28<br />close: 33.80","date: 2021-04-29<br />close: 34.65"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(0,0,0,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"visible":false,"showlegend":false,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null}],"layout":{"margin":{"t":43.7625570776256,"r":7.30593607305936,"b":45.1930835802123,"l":37.2602739726027},"plot_bgcolor":"rgba(255,255,255,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"title":{"text":"ACB stock price from 1/5/2020 to 1/5/2021<br />","font":{"color":"rgba(0,0,0,1)","family":"","size":17.5342465753425},"x":0,"xref":"paper"},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[18368,18764],"tickmode":"array","ticktext":["2020-05-11","2020-06-08","2020-07-06","2020-08-03","2020-08-31","2020-09-28","2020-10-26","2020-11-23","2020-12-21","2021-01-18","2021-02-15","2021-03-15","2021-04-12","2021-05-10"],"tickvals":[18393,18421,18449,18477,18505,18533,18561,18589,18617,18645,18673,18701,18729,18757],"categoryorder":"array","categoryarray":["2020-05-11","2020-06-08","2020-07-06","2020-08-03","2020-08-31","2020-09-28","2020-10-26","2020-11-23","2020-12-21","2021-01-18","2021-02-15","2021-03-15","2021-04-12","2021-05-10"],"nticks":null,"ticks":"outside","tickcolor":"rgba(179,179,179,1)","ticklen":3.65296803652968,"tickwidth":0.33208800332088,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-45,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(222,222,222,1)","gridwidth":0.33208800332088,"zeroline":false,"anchor":"y","title":{"text":"","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-1.7575,36.9075],"tickmode":"array","ticktext":["0","10","20","30"],"tickvals":[0,10,20,30],"categoryorder":"array","categoryarray":["0","10","20","30"],"nticks":null,"ticks":"outside","tickcolor":"rgba(179,179,179,1)","ticklen":3.65296803652968,"tickwidth":0.33208800332088,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(222,222,222,1)","gridwidth":0.33208800332088,"zeroline":false,"anchor":"x","title":{"text":"close","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":"transparent","line":{"color":"rgba(179,179,179,1)","width":0.66417600664176,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":false,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895}},"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","showSendToCloud":false,"displayModeBar":false},"source":"A","attrs":{"15061238b6b99":{"x":{},"y":{},"type":"scatter"},"150612f5c300b":{"y":{}}},"cur_data":"15061238b6b99","visdat":{"15061238b6b99":["function (y) ","x"],"150612f5c300b":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot","plotly_sunburstclick"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script>

## Note

`get_cafeF()` have to send multiple requests to cafef.vn, depend on the time frame user specified. Although I use `{furrr}` to send request in parallel but the the speed is much slower compare with `get_vndirect()` which is only send one request.
