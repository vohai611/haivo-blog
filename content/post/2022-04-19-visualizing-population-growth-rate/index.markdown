---
title: Visualizing population growth rate
author: Hai Vo
date: '2022-04-19'
slug: []
categories: [Viz]
tags:
subtitle: ''
description: 'Visulizing population growth rate using half-eye plot'
image: ''
draft: true
---



## Get the data

Data came from [Vietnam general statistic organization](https://gso.gov.vn)
To be able to easily get the data from R. I build a simple package [`gsovn`](https://github.com/vohai611/gsovn) to scrape data from their website.


```r
library(tidyverse)
data = gsovn::gso_read(gsovn::avail_dataset$link[30])
data$data %>%
  head(10) %>% 
  knitr::kable()
```



|                | 2005| 2007|2008 |2009 |2010 |2011 |2012 |2013 |2014 |2015 |2016 |2017 |2018 |2019 |Prel. 2020 |
|:---------------|----:|----:|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----------|
|WHOLE COUNTRY   | 1.17| 1.09|1.07 |1.06 |1.07 |1.05 |1.08 |1.07 |1.08 |1.12 |1.11 |1.11 |1.17 |1.15 |1.14       |
|Red River Delta | 0.90| 0.63|1.28 |0.74 |1.19 |1.08 |1.04 |1.02 |1.09 |1.37 |1.41 |1.38 |1.47 |1.48 |1.33       |
|Ha Noi          | 2.02| 1.37|NA   |1.41 |2.50 |1.93 |1.54 |1.63 |1.70 |2.03 |2.11 |1.99 |2.23 |2.27 |1.89       |
|Ha Tay          | 2.03| 1.14|NA   |NA   |NA   |NA   |NA   |NA   |NA   |NA   |NA   |NA   |NA   |NA   |NA         |
|Vinh Phuc       | 1.03| 0.69|NA   |0.66 |0.72 |0.38 |1.09 |0.68 |1.22 |1.26 |1.45 |1.54 |1.36 |1.45 |1.42       |
|Bac Ninh        | 0.80| 0.95|0.87 |0.82 |1.73 |1.84 |2.10 |2.06 |2.08 |3.52 |3.23 |3.17 |3.05 |3.08 |2.94       |
|Quang Ninh      | 1.33| 1.19|1.12 |0.97 |0.97 |0.93 |0.83 |0.82 |1.01 |1.13 |1.58 |1.90 |1.46 |1.61 |0.96       |
|Hai Duong       | 0.30| 0.30|0.36 |0.35 |0.56 |0.78 |0.69 |0.58 |0.65 |0.95 |1.11 |0.97 |1.46 |1.02 |1.05       |
|Hai Phong       | 0.89| 0.98|0.98 |0.89 |0.94 |1.19 |1.29 |1.11 |1.08 |0.96 |0.80 |0.81 |0.75 |0.83 |1.00       |
|Hung Yen        | 0.60| 0.43|0.44 |0.21 |0.33 |0.54 |0.50 |0.66 |0.56 |1.12 |1.12 |1.10 |1.10 |1.08 |1.06       |

This dataset contain data of population growth (by percent) in Vietnam. The data is quite in a good a shape, but still need some cleaning

## Cleaning

First I need to remove white space from province/city name.


```r
df = data$data
df = df %>% rename(region =1) %>% 
  mutate(region = str_squish(region))
```

Next I split up province/city by region it belong to.


```r
region = c(
  "WHOLE COUNTRY",
  "Northern midlands and mountain areas",
  "Red River Delta",
  "Northern Central area and Central coastal area",
  "Central Highlands",
  "South East",
  "Mekong River Delta" 
)


region_regex = paste0("(",str_c(region, collapse = "|"),")")

df = df %>% 
  mutate(across(-region, as.numeric)) %>% 
  mutate(group = cumsum(str_detect(region, region_regex))) %>% 
  group_by(group) %>% 
  mutate(group = first(region)) %>% 
  ungroup()
```

Then pivoting data to long format


```r
df_long = df %>% 
  pivot_longer(cols = c(-region, -group),names_to = 'year', values_to = "pop_growth_rate")
  
head(df_long,5) %>% 
  knitr::kable()
```



|region        |group         |year | pop_growth_rate|
|:-------------|:-------------|:----|---------------:|
|WHOLE COUNTRY |WHOLE COUNTRY |2005 |            1.17|
|WHOLE COUNTRY |WHOLE COUNTRY |2007 |            1.09|
|WHOLE COUNTRY |WHOLE COUNTRY |2008 |            1.07|
|WHOLE COUNTRY |WHOLE COUNTRY |2009 |            1.06|
|WHOLE COUNTRY |WHOLE COUNTRY |2010 |            1.07|

## Prepare data for visualization

I want the audience know which province/city belong to which region, so first, I need the map data of Vietnam split by regions. This is quite common task for me, so I saved the map to my personal pacakge [`{haitools}`](https://github.com/vohai611/haitools), and load from it when I need.


```r
library(sf)
library(ggrepel)
theme_set(theme_light())
vn_sf = haitools::small_vnsf

# fix thua thien hue, tp ho chi minh name
df_long = df_long %>% 
  mutate(region = case_when(region == "Thua Thien-Hue" ~ "Thua Thien Hue",
                            region == "Ho Chi Minh city" ~ "Ho Chi Minh",
                            TRUE ~ region))

a = map(vn_sf$geometry, ~ colMeans(chuck(.x,1,1))) 
coord_x = map_dbl(a, 1)
coord_y = map_dbl(a,2)

vn_sf = vn_sf %>% mutate(coord_x = coord_x, coord_y = coord_y)
```

Viet nam map by region 


```r
p1 = df_long %>%  
  distinct(region,group) %>% 
  mutate(group = fct_relevel(group, .env$region[-1] )) %>% 
  inner_join(vn_sf, by = c(region = "province")) %>% 
  ggplot(aes(geometry = geometry, fill = group))+
  geom_sf(color= "white", alpha = .5)+
  guides(fill = guide_legend(ncol = 1))+
  theme_void()+
  labs(fill = "")+
  #geom_text_repel(aes(x = coord_x, y = coord_y, label = region),max.overlaps = 20, size = 3)
  theme(legend.position =c(0.4,0.35), text = element_text(size =16))
p1
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="1152" />

Next, I visualizing the distribution of growth rate for each region. There are a interesting package `{ggdist}` which draw half-eye plot for distribution, compare with the typical boxplot or violin-plot. 


```r
df_count_group = df_long %>% 
  filter(!region %in% .env$region) %>% 
  filter(year == 2019) %>% 
  count(group)

df_by_group = df_long %>% 
  filter(!region %in% .env$region) %>% 
  left_join(df_count_group, by = c("group"= "group")) %>% 
  mutate(group = fct_relevel(group, .env$region[-1])) 

df_by_group_n = df_by_group %>% distinct(group, n) %>% 
  mutate(new_name = paste0(group, " (n=", n, ")"))
new_name = df_by_group_n$new_name
names(new_name) = df_by_group_n$group

df_by_group = df_by_group %>% 
  mutate(group = fct_relabel(group, ~new_name[.] ))


df_whole = df_long %>% 
  filter(region == "WHOLE COUNTRY")

df_whole2 = df_whole %>% 
  mutate(group = map(region, ~unique(df_by_group$group))) %>% 
  unnest(group)
```


```r
p2 = df_by_group %>% 
  ggplot(aes(year, pop_growth_rate, group = group, fill = group))+
  ggdist::stat_halfeye(alpha = .7)+
  geom_line(data = df_whole2,
            aes(year, pop_growth_rate, group = region),
            linetype =2, 
            inherit.aes = FALSE) +
  facet_wrap( ~ group, scales = "free",ncol = 2) +
  guides(fill = "none") +
  scale_y_continuous(labels = scales::percent_format(scale = 1)) +
  theme(panel.grid.major.x = element_blank(),
        panel.grid.minor.x = element_blank(),
        strip.background = element_rect(fill = "#9eba89"),
        strip.text = element_text(colour = "#2d3443", size = 18),
        #plot.background = element_rect(fill = "grey80"),
        #panel.background = element_rect(fill ="grey80"),
        text = element_text(size= 18)) +
  labs(
    y = NULL,
    x = NULL
  )

p2
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-8-1.png" width="1152" />

At last, I combine two plot with `{patchwork}`


```r
library(patchwork)
library(ggtext)
p = p1 + p2 +
  plot_annotation(
    title = "Distribution of population growth rate group by area, from 2005 to 2020",
    subtitle = "**(n)** is number of provinces in each group <br/>
                  *Dash-line* is the average of whole country",
    theme = theme(
      plot.title = element_markdown(size = 22),
      plot.subtitle = element_markdown(size = 18),
      plot.background = element_rect(fill = "grey80"),
    )
  )
```


To save out images, use `ggsave()`




```r
knitr::include_graphics("Dist-of-pop-growth.png")
```

<img src="Dist-of-pop-growth.png" width="4400" />




