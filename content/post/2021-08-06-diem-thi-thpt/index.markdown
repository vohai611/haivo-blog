---
title: Scrape university entrance exam score
summary: Using {rvest} and {furrr} to scrape exam score in parallel
cover: 
  image: "cover.png"
  caption: Density plot by subject score (2021)
author: Hai Vo
date: '2021-08-06'
slug: []
categories: []
tags: [API, httr]
---




## Why

University entrance exam is one of the most important exam of every Vietnamese student. In the year of 2020, by analyzing exam score, people discover abnormal data point and then detect a serious cheating. 
The full data is normally not public by the Education Department. However, anyone could get the exam result through multiple website by fill in correct candidate ID.
This post present how to use R to send API request to retrieve full exam score in the whole country in 2021. The full script and data could be found at my github [repository](https://github.com/vohai611/diemthi-thpt-2021). 

## How does it work
There are several website provide web interface to get the score. In my script, I use two sources of data:
https://diemthi.vnanet.vn and https://tienphong.vn/tra-cuu-diem-thi.tpo.

Under the hood, these two website use API to retrieve the data, and user can access this API request directly via web browser devtools (network tab). The API from vnanet.vn is quite slow and only allow to retrieve 1 result per request. On the other hand, tienphong.vn allows user to get a maximum of 300 result per request, hence I decide to use the latter options to get the whole data.

These two API take the student ID as input and provide full result of that student as output. Input are in the form {province_code}{student_id}. Province_code vary from 01 to 64, while the student_id are from 1 to the max number of student attend in the exam. For example, input = '01000001' is for student 1 from the province that had code 01 (Ha Noi).


## Demonstration
Option 1:

```r
library(tidyverse)
library(glue)
library(httr)
library(here)
get_score <- function(sbd) {
  url <- glue("https://diemthi.vnanet.vn/Home/SearchBySobaodanh?code={ sbd }&nam=2021")
  GET(url) %>% 
    content(type = 'text', encoding = 'UTF-8') %>% 
    jsonlite::fromJSON() %>% 
    .[['result']] %>% 
    as_tibble()
}

get_score('01000001') %>% 
  knitr::kable()
```



|CityCode |CityArea |Code     |Toan |NguVan |NgoaiNgu |VatLi |HoaHoc |SinhHoc |KHTN |DiaLi |LichSu |GDCD |KHXH |ResultGroup                                                                              |Result |
|:--------|:--------|:--------|:----|:------|:--------|:-----|:------|:-------|:----|:-----|:------|:----|:----|:----------------------------------------------------------------------------------------|:------|
|01       |NA       |01000001 |2.20 |3.50   |         |      |       |        |     |5.50  |2.50   |     |     |[{"g":"A07","p":10.20},{"g":"C00","p":11.50},{"g":"C03","p":8.20},{"g":"C04","p":11.20}] |       |

Option 2:
In this API options, I can use sbd = '0100001' to get the result of 10 attendance in one request from 01000011 to 01000019

```r
get_score2 <- function(sbd){
  
  # prepare URL
  url <- glue('https://tienphong.vn/api/diemthi/get/result?type=0&keyword={ sbd }&kythi=THPT&nam=2021&cumthi=0')
  ## send request
  a <- GET(url,
           add_headers('referer'= 'https://tienphong.vn/tra-cuu-diem-thi.tpo')
  ) %>% 
    content(as = 'text') %>% 
    jsonlite::fromJSON()
  
  # parse request to text format
  a$data$results %>% 
    rvest::read_html() %>% 
    rvest::html_text2() %>% 
    return()
}

# data received in the form of TSV:
get_score2('0100001') %>% 
  data.table::fread() %>% 
  knitr::kable()
```



| V1|      V2|  V3|   V4|  V5|   V6|   V7|   V8|   V9|  V10|  V11|V12 |
|--:|-------:|---:|----:|---:|----:|----:|----:|----:|----:|----:|:---|
|  1| 1000019| 7.0| 8.50| 8.8|   NA|   NA|   NA| 4.00| 5.25| 6.75|NA  |
|  2| 1000018| 8.8| 8.25|  NA| 8.00| 5.00| 6.75|   NA|   NA|   NA|NA  |
|  3| 1000017| 7.8| 8.00| 9.6| 7.50| 8.00| 7.25|   NA|   NA|   NA|NA  |
|  4| 1000016| 7.8| 8.50| 9.4|   NA|   NA|   NA| 6.50| 7.50| 8.00|NA  |
|  5| 1000015| 6.0| 7.75| 9.0|   NA|   NA|   NA| 4.00| 7.75| 7.00|NA  |
|  6| 1000014| 7.4| 8.00| 8.6|   NA|   NA|   NA| 6.00| 6.25| 7.50|NA  |
|  7| 1000013| 7.4| 6.75| 9.0|   NA|   NA|   NA| 3.75| 8.50| 6.50|NA  |
|  8| 1000012| 6.4| 6.75| 7.8|   NA|   NA|   NA| 5.50| 7.00| 7.50|NA  |
|  9| 1000011| 6.0| 7.75| 8.2|   NA|   NA|   NA| 3.00| 7.25| 8.50|NA  |
| 10| 1000010| 8.8| 6.25| 9.2| 8.75| 8.75| 3.00|   NA|   NA|   NA|NA  |

In the script, I also exploit multi-core in my machine (4-cores) to send multiple request simultaneously by using `{furrr}` package (front-end to the `{future}` package). The use of parallel code yield around 3 times faster result (60 mins for nearly 1 millions result) compare with normal use.

Because the API sent me the province code instead of province name, I will need an extra step to get the province.


```r
library(rvest)
province_code <- read_html("https://diemthi.vnanet.vn/Home/") %>% 
  html_elements("#listCity") %>% 
  html_text2() %>% 
  str_replace_all(pattern = "(\\d{2})", "\n\\1, ") %>% 
  read_csv(skip = 1, col_names = c('province_code', 'province')) %>% 
  mutate(province = stringi::stri_trans_general(province, "Latin-ASCII")) %>% 
  mutate(province = str_remove(province, 'So GDDT '),
         province = str_remove(province, "So GD KHCN "))

province_code
```

```
## # A tibble: 64 × 2
##    province_code province       
##    <chr>         <chr>          
##  1 01            Ha Noi         
##  2 02            TP. Ho Chi Minh
##  3 03            Hai Phong      
##  4 04            Da Nang        
##  5 05            Ha Giang       
##  6 06            Cao Bang       
##  7 07            Lai Chau       
##  8 08            Lao Cai        
##  9 09            Tuyen Quang    
## 10 10            Lang Son       
## # … with 54 more rows
```

Finally I just need to join to data sets, this is what final result look like:


```r
here::i_am("content/project/2021-08-06-diem-thi-thpt/index.markdown")
path <- "content/project/2021-08-06-diem-thi-thpt"

data <- read_rds(here(path, "data/output.rds"))

data %>% 
  left_join(province_code) %>% 
  head(10) %>% 
  knitr::kable()
```



|province_code |      V2| toan|  van|  nn|   li| hoa| sinh|   su|  dia| gdcd|province |
|:-------------|-------:|----:|----:|---:|----:|---:|----:|----:|----:|----:|:--------|
|01            | 1000099|  7.8| 7.75| 7.2|   NA|  NA|   NA| 2.75| 6.50| 7.50|Ha Noi   |
|01            | 1000098|  9.4| 7.50| 8.4| 8.75| 8.0| 4.75|   NA|   NA|   NA|Ha Noi   |
|01            | 1000097|  3.8| 8.25| 6.8|   NA|  NA|   NA| 4.75| 6.75| 9.00|Ha Noi   |
|01            | 1000096|  5.8| 8.00| 9.2|   NA|  NA|   NA| 4.25| 7.00| 7.25|Ha Noi   |
|01            | 1000095|  5.2| 6.00| 2.6|   NA|  NA|   NA| 7.50| 8.00| 8.50|Ha Noi   |
|01            | 1000094|  3.8| 7.00| 4.2|   NA|  NA|   NA| 3.75| 7.25| 7.50|Ha Noi   |
|01            | 1000093|  9.4| 7.50|  NA| 8.50| 5.5| 6.75|   NA|   NA|   NA|Ha Noi   |
|01            | 1000092|  6.2| 7.75| 4.0|   NA|  NA|   NA| 3.75| 8.00| 8.25|Ha Noi   |
|01            | 1000091|  6.0| 6.50| 3.8|   NA|  NA|   NA| 4.50| 6.00| 8.00|Ha Noi   |
|01            | 1000090|  8.4| 8.00| 9.2|   NA|  NA|   NA| 6.25| 8.50| 8.50|Ha Noi   |


<style type="text/css">
h2 {
  color: #9EBA89;
}
</style>

