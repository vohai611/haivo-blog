---
title: Scraping data from Vietnam general statistic organization
author: Hai Vo
date: '2022-04-21'
slug: scraping-data-from-vietnam-general-statistic-organization
categories: [Project]
tags: [rvest, R-package]
subtitle: ''
description: "Build a simple package that enable user to access data from gso.gov.vn from R"
image: 'img/star-light.jpg'
---



The goal of gsovn is to scrape dataset from gso.gov.vn ([Vietnam General statistic organization](https://gso.gov.vn))

## Installation

This package available through [github](https://github.com/vohai611/gsovn). You can install it via

```r
# install.packages("devtools")
devtools::install_github("vohai611/gsovn")
```

## Usage

You can check what dataset is now on gso.gov.vn using `{gso_avail()}`. There are two options for data retrieved: data in Vietnamese (vi) or English (en). You can also search for the term in title with `search_term=`


```r
df = gsovn::gso_avail(lang = "vi", search_term = "giao duc")
df
```

```
## # A tibble: 4 × 3
##   title                                                        link  title_clean
##   <chr>                                                        <chr> <chr>      
## 1 Giáo dục đại học và cao đẳng(*)                              http… Giao duc d…
## 2 Giáo dục nghề nghiệp(*)                                      http… Giao duc n…
## 3 Số giáo viên giáo dục nghề nghiệp phân theo trình độ chuyên… http… So giao vi…
## 4 Số giáo viên, học sinh, sinh viên giáo dục nghề nghiệp năm … http… So giao vi…
```

To access the data. Use `gso_read()` on the corresponding the link from `gso_avail()`


```r
education = df$link[1] |>
  gsovn::gso_read()

education$data |>
  head(10) |>
  knitr::kable()
```



|                         |1995  |1996 |1997  |1998  |1999  |2000  |2001  |2002    |2003    |2004    |2005    |2006    |2007    |2008    |2009    |2010    |2011    |2012    |2013    |2014    |2015    |2016    |2017    |2018    |2019    |
|:------------------------|:-----|:----|:-----|:-----|:-----|:-----|:-----|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|
|Tổng số                  |      |     |      |      |      |      |      |        |        |        |        |        |        |        |        |        |        |        |        |        |        |        |        |        |        |
|Trường học               |NA    |NA   |NA    |NA    |NA    |178,0 |191,0 |202,0   |214,0   |230,0   |277,0   |322,0   |369,0   |393,0   |403,0   |414,0   |419,0   |421,0   |428,0   |436,0   |223,0   |235,0   |236,0   |237,0   |237,0   |
|Trường công lập          |109,0 |96,0 |110,0 |123,0 |131,0 |148,0 |168,0 |179,0   |187,0   |201,0   |243,0   |275,0   |305,0   |322,0   |326,0   |334,0   |337,0   |340,0   |343,0   |347,0   |163,0   |170,0   |171,0   |172,0   |172,0   |
|Trường ngoài công lập    |NA    |NA   |NA    |NA    |NA    |30,0  |23,0  |23,0    |27,0    |29,0    |34,0    |47,0    |64,0    |71,0    |77,0    |80,0    |82,0    |81,0    |85,0    |89,0    |60,0    |65,0    |65,0    |65,0    |65,0    |
|Giảng viên               |NA    |NA   |NA    |NA    |NA    |32,3  |35,9  |38,7    |40,0    |47,6    |48,6    |53,4    |56,1    |60,7    |69,6    |74,6    |84,1    |87,7    |91,6    |91,4    |69,6    |72,8    |75,0    |73,3    |73,1    |
|Giáo viên công lập       |22,8  |23,5 |24,1  |26,1  |27,1  |27,9  |31,4  |33,4    |34,9    |40,0    |42,0    |45,7    |51,3    |54,8    |60,3    |63,3    |70,4    |73,9    |75,2    |74,1    |55,4    |57,6    |59,2    |57,0    |57,0    |
|Giáo viên ngoài công lập |NA    |NA   |NA    |NA    |NA    |4,5   |4,5   |5,3     |5,1     |7,6     |6,6     |7,7     |4,8     |5,9     |9,3     |11,3    |13,7    |13,8    |16,4    |17,3    |14,2    |15,2    |15,8    |16,3    |16,1    |
|Giáo viên Nam            |NA    |NA   |NA    |NA    |NA    |NA    |NA    |NA      |NA      |NA      |28,1    |NA      |30,8    |32,4    |36,8    |39,2    |43,0    |44,9    |46,7    |42,3    |36,9    |37,7    |38,4    |36,5    |36,4    |
|Giáo viên Nữ             |NA    |NA   |NA    |NA    |NA    |NA    |NA    |NA      |NA      |NA      |20,5    |NA      |25,3    |28,3    |32,8    |35,4    |41,1    |42,8    |44,9    |49,1    |32,7    |35,1    |36,6    |36,8    |36,7    |
|Sinh viên                |NA    |NA   |NA    |NA    |NA    |899,5 |974,1 |1.020,7 |1.131,0 |1.319,8 |1.387,1 |1.666,2 |1.603,5 |1.719,5 |1.956,2 |2.162,1 |2.208,1 |2.178,6 |2.061,6 |2.363,9 |1.753,2 |1.767,9 |1.707,0 |1.526,1 |1.672,9 |

## TODO

This version of `gso_read()` use `rvest::html_table()` under the hood, this function work fine but I might be slow if you queries a large data set, this function work fine but I might be slow if you queries a large data set. I will implement function to download csv data in the future.
