---
title: "Height"
author: "David Yang"
date: "April 13, 2021"
output:
  html_document:  
    keep_md: true
    toc: true
    toc_float: true
    code_folding: hide
    fig_height: 6
    fig_width: 12
    fig_align: 'center'
---





```r
ex <- tempfile()
download('https://byuistats.github.io/M335/data/heights/Height.xlsx', ex, mode = 'wb')
excel <- read_xlsx(ex,skip = 2)
GermanM <- read_dta('https://byuistats.github.io/M335/data/heights/germanconscr.dta')
BavarianM <- read_dta('https://byuistats.github.io/M335/data/heights/germanprison.dta')
NSH <- read_sav('http://www.ssc.wisc.edu/nsfh/wave3/NSFH3%20Apr%202005%20release/main05022005.sav')
BLS <- read_csv('https://github.com/hadley/r4ds/raw/master/data/heights.csv')
Drag <- "https://byuistats.github.io/M335/data/heights/Heights_south-east.zip"
RDrag <- tempfile()
VDrag <- tempfile()
downloader::download(Drag , RDrag, mode = "wb")
unzip(zipfile = RDrag, exdir = VDrag)
QT <- list.files(VDrag, pattern = "DBF")
UQT <- read.dbf(file = file.path(VDrag, QT)) %>%
  as_tibble()
```

## Background

The Scientific American argues that humans have been getting taller over the years. As the data scientists that we are becoming, we would like to find data that validates this concept. Our challenge is to show different male heights across the centuries.

This project is not as severe as the two quotes below, but it will give you a taste of pulling various data and file formats together into “tidy” data for visualization and analysis. You will not need to search for data as all the files are listed here

“Classroom data are like teddy bears and real data are like a grizzly bear with salmon blood dripping out its mouth.” - Jenny Bryan
“Up to 80% of data analysis is spent on the process of cleaning and preparing data” - Hadley Wickham

## Data Wrangling #1


```r
CS5 <- excel %>% 
  pivot_longer(cols = `1800`:`2011`, names_to = "year", values_to = "value") %>% 
  mutate(year_decade = year) %>% 
  separate(year, into = c("century","decade"),sep = 2) %>% 
  separate(decade, into = c("decade","year"),sep = 1) %>% 
  rename("height.cm" = "value") %>%
  mutate(height.in = (conv_unit(height.cm, "cm", "inch")))
```

## Data Wrangling #2


```r
UQT<- UQT %>% 
  mutate(Study_id = "UQT1") %>%
  rename("height.cm" = "CMETER") %>% 
  rename("birth_year" = "GEBJ") %>% 
  select(birth_year,height.cm,Study_id) %>% 
  mutate(height.in = round((conv_unit(height.cm, "cm", "inch")),0)) %>% 
  mutate(height.cm = round(height.cm)) %>% 
  select(birth_year,height.cm,height.in,Study_id)
BLS<- BLS %>% 
  mutate(Study_id = "BLS1") %>% 
  mutate(birth_year = "1950") %>% 
  select(birth_year,height,Study_id) %>% 
  rename("height.in" = "height") %>% 
  mutate(height.cm = round(conv_unit(height.in, "inch", "cm"),0),
         birth_year = as.numeric(birth_year)) %>%
  mutate(height.in = round(height.in)) %>% 
  select(birth_year,height.cm,height.in,Study_id)
NSH<- NSH %>% 
  mutate(Study_id = "NSH1") %>%
  mutate(centry = 19, ) %>% 
  unite("birth_year", centry, DOBY, sep = "") %>% 
  mutate(height.in = round((conv_unit(RT216F, "feet", "inch")),0)) %>% 
  mutate(height.in = height.in + RT216I) %>% 
  mutate(height.cm = round(conv_unit(height.in, "inch", "cm"), 0),
         birth_year = as.numeric(birth_year)) %>%
  filter(birth_year>1724) %>% 
  select(birth_year,height.cm,height.in,Study_id)
BavarianM<- BavarianM %>% 
  mutate(Study_id = "BAV1") %>% 
  select(bdec,height,Study_id) %>% 
  rename("height.cm" = "height") %>% 
  mutate(height.in = round((conv_unit(height.cm, "cm", "inch")),0)) %>% 
  rename("birth_year" = "bdec") %>%
  mutate(height.cm = round(height.cm)) %>% 
  select(birth_year,height.cm,height.in,Study_id)
GermanM<- GermanM %>% 
  mutate(Study_id = "GER1") %>% 
  select(bdec,height,Study_id) %>% 
  rename("height.cm" = "height") %>% 
  mutate(height.in = round((conv_unit(height.cm, "cm", "inch"))),0) %>% 
  rename("birth_year" = "bdec") %>% 
  mutate(height.cm = round(height.cm)) %>% 
  select(birth_year,height.cm,height.in,Study_id)
 
CS51<- bind_rows(GermanM, BavarianM, NSH, BLS, UQT)
```

## Data Visualization


```r
CS5%>% 
  ggplot(aes(x = decade, y = height.in)) +
  geom_point() +
  geom_point(CS5 %>% filter(`Continent, Region, Country` == "Germany"), 
             mapping = aes(x = decade, y = height.in), 
             color = "red") +
  labs(title = "Height for Two Centuries", 
       x = "Decades", y = "Height(Inches)")+
  theme_bw()
```

![](Height_files/figure-html/plot_data-1.png)<!-- -->

## Conclusions
People are still generally have similar height.
