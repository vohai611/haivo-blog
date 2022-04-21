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
---



## Get the data

Data came from [Vietnam general statistic organization](https://gso.gov.vn)
To be able to easily get the data from R. I build a simple package [`gsovn`](https://github.com/vohai611/gsovn) to scrape data from their website.



```r
library(tidyverse)
data = gsovn::gso_read(gsovn::avail_dataset$link[30])
```


## clean


```r
df = data$data
df = df %>% rename(region =1) %>% 
  mutate(region= str_squish(region))
```



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

df_long = df %>% 
  pivot_longer(cols = c(-region, -group),names_to = 'year', values_to = "pop_growth_rate")
```



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


How to use fct_reorder2(.f, .x, .y, .fun)

```r
df_long %>% 
  filter(!region %in% .env$region[-1]) %>% 
  group_by(group, year) %>% 
  summarise(mean = mean(pop_growth_rate, na.rm =TRUE), n = n()) %>% 
  ungroup() %>% 
  mutate(group = paste0(group, " (", n, ")")) %>% 
  mutate(group = fct_reorder2(group, year, mean)) %>% 
  ggplot(aes(year, mean, color = group, group = group))+
  geom_point()+
  geom_line()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="1152" />




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

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="1152" />

Distribution of population growth rate


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




