---
title: "USA extreme weather events exploration"
author: "Marko"
date: "11/19/2020"
output: 
  html_document:
    keep_md: true
---



# Synopsis



# Data Processing

## Import raw data

Firs we need to import some libraries to execute the analysis:

```r
rm(list = ls())
graphics.off()

# Load R packages
packages <- c("dplyr",   # list of packages to load
              "tidyr",
              "ggplot2",  
              "lubridate", 
              "stringr",
              "forcats",
              "cowplot", 
              "data.table",
              "janitor",
              "kableExtra",
              "datasets") 

package.check <- lapply( # load or install & load list of packages
  packages,
  FUN = function(x) {
    if (!require(x, character.only = TRUE)) {
      install.packages(x, dependencies = TRUE)
      library(x, character.only = TRUE)
    }
  }
) 
rm(packages, package.check)
```


Raw data is downloaded here:

```r
# Download raw csv file if necessary
if(!file.exists("repdata_data_StormData.csv.bz2")){
  download.file(url = "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2", 
                destfile = "repdata_data_StormData.csv.bz2") 
  }

# Import file
df.raw <- fread(input = "repdata_data_StormData.csv.bz2")
```


## Initial data inspection

Dimension (number of observations & number of variables) of the dataset:

```r
dim(df.raw)
```

```
## [1] 902297     37
```

Structure of given data frame:

```r
str(df.raw)
```

```
## Classes 'data.table' and 'data.frame':	902297 obs. of  37 variables:
##  $ STATE__   : num  1 1 1 1 1 1 1 1 1 1 ...
##  $ BGN_DATE  : chr  "4/18/1950 0:00:00" "4/18/1950 0:00:00" "2/20/1951 0:00:00" "6/8/1951 0:00:00" ...
##  $ BGN_TIME  : chr  "0130" "0145" "1600" "0900" ...
##  $ TIME_ZONE : chr  "CST" "CST" "CST" "CST" ...
##  $ COUNTY    : num  97 3 57 89 43 77 9 123 125 57 ...
##  $ COUNTYNAME: chr  "MOBILE" "BALDWIN" "FAYETTE" "MADISON" ...
##  $ STATE     : chr  "AL" "AL" "AL" "AL" ...
##  $ EVTYPE    : chr  "TORNADO" "TORNADO" "TORNADO" "TORNADO" ...
##  $ BGN_RANGE : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ BGN_AZI   : chr  "" "" "" "" ...
##  $ BGN_LOCATI: chr  "" "" "" "" ...
##  $ END_DATE  : chr  "" "" "" "" ...
##  $ END_TIME  : chr  "" "" "" "" ...
##  $ COUNTY_END: num  0 0 0 0 0 0 0 0 0 0 ...
##  $ COUNTYENDN: logi  NA NA NA NA NA NA ...
##  $ END_RANGE : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ END_AZI   : chr  "" "" "" "" ...
##  $ END_LOCATI: chr  "" "" "" "" ...
##  $ LENGTH    : num  14 2 0.1 0 0 1.5 1.5 0 3.3 2.3 ...
##  $ WIDTH     : num  100 150 123 100 150 177 33 33 100 100 ...
##  $ F         : int  3 2 2 2 2 2 2 1 3 3 ...
##  $ MAG       : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ FATALITIES: num  0 0 0 0 0 0 0 0 1 0 ...
##  $ INJURIES  : num  15 0 2 2 2 6 1 0 14 0 ...
##  $ PROPDMG   : num  25 2.5 25 2.5 2.5 2.5 2.5 2.5 25 25 ...
##  $ PROPDMGEXP: chr  "K" "K" "K" "K" ...
##  $ CROPDMG   : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ CROPDMGEXP: chr  "" "" "" "" ...
##  $ WFO       : chr  "" "" "" "" ...
##  $ STATEOFFIC: chr  "" "" "" "" ...
##  $ ZONENAMES : chr  "" "" "" "" ...
##  $ LATITUDE  : num  3040 3042 3340 3458 3412 ...
##  $ LONGITUDE : num  8812 8755 8742 8626 8642 ...
##  $ LATITUDE_E: num  3051 0 0 0 0 ...
##  $ LONGITUDE_: num  8806 0 0 0 0 ...
##  $ REMARKS   : chr  "" "" "" "" ...
##  $ REFNUM    : num  1 2 3 4 5 6 7 8 9 10 ...
##  - attr(*, ".internal.selfref")=<externalptr>
```

Sneak peak into first few rows of the data:

```r
head(df.raw) %>% 
  kbl() %>% 
  kable_paper() %>%
  scroll_box(width = "100%", height = "250px")
```

<div style="border: 1px solid #ddd; padding: 0px; overflow-y: scroll; height:250px; overflow-x: scroll; width:100%; "><table class=" lightable-paper" style='font-family: "Arial Narrow", arial, helvetica, sans-serif; margin-left: auto; margin-right: auto;'>
 <thead>
  <tr>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> STATE__ </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> BGN_DATE </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> BGN_TIME </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> TIME_ZONE </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> COUNTY </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> COUNTYNAME </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> STATE </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> EVTYPE </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> BGN_RANGE </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> BGN_AZI </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> BGN_LOCATI </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> END_DATE </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> END_TIME </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> COUNTY_END </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> COUNTYENDN </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> END_RANGE </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> END_AZI </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> END_LOCATI </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> LENGTH </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> WIDTH </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> F </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> MAG </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> FATALITIES </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> INJURIES </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> PROPDMG </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> PROPDMGEXP </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> CROPDMG </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> CROPDMGEXP </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> WFO </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> STATEOFFIC </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> ZONENAMES </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> LATITUDE </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> LONGITUDE </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> LATITUDE_E </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> LONGITUDE_ </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> REMARKS </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> REFNUM </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:left;"> 4/18/1950 0:00:00 </td>
   <td style="text-align:left;"> 0130 </td>
   <td style="text-align:left;"> CST </td>
   <td style="text-align:right;"> 97 </td>
   <td style="text-align:left;"> MOBILE </td>
   <td style="text-align:left;"> AL </td>
   <td style="text-align:left;"> TORNADO </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 14.0 </td>
   <td style="text-align:right;"> 100 </td>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 15 </td>
   <td style="text-align:right;"> 25.0 </td>
   <td style="text-align:left;"> K </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 3040 </td>
   <td style="text-align:right;"> 8812 </td>
   <td style="text-align:right;"> 3051 </td>
   <td style="text-align:right;"> 8806 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:left;"> 4/18/1950 0:00:00 </td>
   <td style="text-align:left;"> 0145 </td>
   <td style="text-align:left;"> CST </td>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:left;"> BALDWIN </td>
   <td style="text-align:left;"> AL </td>
   <td style="text-align:left;"> TORNADO </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 2.0 </td>
   <td style="text-align:right;"> 150 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 2.5 </td>
   <td style="text-align:left;"> K </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 3042 </td>
   <td style="text-align:right;"> 8755 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:left;"> 2/20/1951 0:00:00 </td>
   <td style="text-align:left;"> 1600 </td>
   <td style="text-align:left;"> CST </td>
   <td style="text-align:right;"> 57 </td>
   <td style="text-align:left;"> FAYETTE </td>
   <td style="text-align:left;"> AL </td>
   <td style="text-align:left;"> TORNADO </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 0.1 </td>
   <td style="text-align:right;"> 123 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 25.0 </td>
   <td style="text-align:left;"> K </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 3340 </td>
   <td style="text-align:right;"> 8742 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 3 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:left;"> 6/8/1951 0:00:00 </td>
   <td style="text-align:left;"> 0900 </td>
   <td style="text-align:left;"> CST </td>
   <td style="text-align:right;"> 89 </td>
   <td style="text-align:left;"> MADISON </td>
   <td style="text-align:left;"> AL </td>
   <td style="text-align:left;"> TORNADO </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 0.0 </td>
   <td style="text-align:right;"> 100 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 2.5 </td>
   <td style="text-align:left;"> K </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 3458 </td>
   <td style="text-align:right;"> 8626 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 4 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:left;"> 11/15/1951 0:00:00 </td>
   <td style="text-align:left;"> 1500 </td>
   <td style="text-align:left;"> CST </td>
   <td style="text-align:right;"> 43 </td>
   <td style="text-align:left;"> CULLMAN </td>
   <td style="text-align:left;"> AL </td>
   <td style="text-align:left;"> TORNADO </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 0.0 </td>
   <td style="text-align:right;"> 150 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 2.5 </td>
   <td style="text-align:left;"> K </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 3412 </td>
   <td style="text-align:right;"> 8642 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 5 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:left;"> 11/15/1951 0:00:00 </td>
   <td style="text-align:left;"> 2000 </td>
   <td style="text-align:left;"> CST </td>
   <td style="text-align:right;"> 77 </td>
   <td style="text-align:left;"> LAUDERDALE </td>
   <td style="text-align:left;"> AL </td>
   <td style="text-align:left;"> TORNADO </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 1.5 </td>
   <td style="text-align:right;"> 177 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 6 </td>
   <td style="text-align:right;"> 2.5 </td>
   <td style="text-align:left;"> K </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 3450 </td>
   <td style="text-align:right;"> 8748 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 6 </td>
  </tr>
</tbody>
</table></div>

Check how many different states occur in the data set (we would like to focus only on existing US states):

```r
df.raw %>% pull(STATE) %>% unique(.) %>% length(.)
```

```
## [1] 72
```
We will need to keep only state names abbreviations that actually exists. Measurements with other state name abbr. will be removed from the data set. We would like to analyze data on the USA level.


How many different event types are in the dataset:

```r
df.raw %>% count(EVTYPE) %>% nrow()
```

```
## [1] 985
```
There are a lot of different event types, so we will focus only on most extreme ones (ones that are most harmful for the people or ones that have the greatest economic consequences).


What is the time span of data:

```r
df.raw %>% 
  summarise(`date min` = min(BGN_DATE),
            `date max` = max(BGN_DATE))
```

```
##           date min         date max
## 1 1/1/1966 0:00:00 9/9/2011 0:00:00
```

Summary of variables (number of injuries and fatalities):

```r
df.raw %>% 
  select(INJURIES, FATALITIES) %>% 
  summary()
```

```
##     INJURIES           FATALITIES      
##  Min.   :   0.0000   Min.   :  0.0000  
##  1st Qu.:   0.0000   1st Qu.:  0.0000  
##  Median :   0.0000   Median :  0.0000  
##  Mean   :   0.1557   Mean   :  0.0168  
##  3rd Qu.:   0.0000   3rd Qu.:  0.0000  
##  Max.   :1700.0000   Max.   :583.0000
```

Summary of variables related to damage:

```r
df.raw %>% 
  select(PROPDMG, CROPDMG, CROPDMGEXP) %>% 
  summary()
```

```
##     PROPDMG           CROPDMG         CROPDMGEXP       
##  Min.   :   0.00   Min.   :  0.000   Length:902297     
##  1st Qu.:   0.00   1st Qu.:  0.000   Class :character  
##  Median :   0.00   Median :  0.000   Mode  :character  
##  Mean   :  12.06   Mean   :  1.527                     
##  3rd Qu.:   0.50   3rd Qu.:  0.000                     
##  Max.   :5000.00   Max.   :990.000
```

```r
df.raw %>% count(PROPDMGEXP) %>% arrange(desc(n))
```

```
##     PROPDMGEXP      n
##  1:            465934
##  2:          K 424665
##  3:          M  11330
##  4:          0    216
##  5:          B     40
##  6:          5     28
##  7:          1     25
##  8:          2     13
##  9:          ?      8
## 10:          m      7
## 11:          H      6
## 12:          +      5
## 13:          7      5
## 14:          3      4
## 15:          4      4
## 16:          6      4
## 17:          -      1
## 18:          8      1
## 19:          h      1
```

```r
df.raw %>% count(CROPDMGEXP) %>% arrange(desc(n))
```

```
##    CROPDMGEXP      n
## 1:            618413
## 2:          K 281832
## 3:          M   1994
## 4:          k     21
## 5:          0     19
## 6:          B      9
## 7:          ?      7
## 8:          2      1
## 9:          m      1
```



## Data wrangling

We need to clean the initial data set a bit in order to answer our questions.

First we will clean columns names, set proper variable types, and create additional variables;


```r
# clean column names
df.clean <- df.raw %>% 
  clean_names  

# column type conversion & create new columns
df.clean <- df.clean %>% 
  mutate(bgn_date = mdy_hms(bgn_date), # convert to date
         bgn_datetime = paste0(as.character(bgn_date), " ",  # build date + time column
                               str_sub(bgn_time, start = 1, end = 2), ":",
                               str_sub(bgn_time, start = 3, end = 4)),
         bgn_datetime = ymd_hm(bgn_datetime), # convert to date time object
         year = lubridate::year(bgn_date), # add year
         # generate new variables
         fatalities_and_injuries = fatalities + injuries, # count deaths and injuries together
         propdmg_usd = case_when(propdmgexp == "K" ~ propdmg * 1000, # calculate actual property damage in USD
                                 propdmgexp == "M" ~ propdmg * 1000000,
                                 propdmgexp == "B" ~ propdmg * 1000000000,
                                 T ~ propdmg),
         cropdmg_usd = case_when(cropdmgexp == "K" ~ cropdmg * 1000, # calculate actual crop damage in USD
                                 cropdmgexp == "M" ~ cropdmg * 1000000,
                                 cropdmgexp == "B" ~ cropdmg * 1000000000,
                                 T ~ cropdmg),
         dmgtot_usd = propdmg_usd + cropdmg_usd) # total damage in USD (property + crop)
```

```
## Warning: 5 failed to parse.
```

Keep only US state names abbr. that actually exists. We will use **datasets** package and function **state.abb** to get state names abbr. and will join main table with US abbr. names:


```r
df.states <- data.frame(state = state.abb, # US state names 
                        US_state_name = T) # flag for actual names  

# Join tables
df.clean <- df.clean %>% 
  left_join(x = ., 
            y = df.states,
            by = c("state_2" = "state")) %>% 
  mutate(US_state_name = replace_na(US_state_name, FALSE)) %>%  # NAs replace with FALSE
  filter(US_state_name == T) # keep only actual US state names 
```



## Calculate summaries

In order to find most devastating weather events in observed time period we will first calculate total of:

* injuries and/or fatalities
* damage done

For the entire USA and complete time span. Based on this summary we will be able to find "Top Killers" and "Top Destroyers", that have caused most trouble for the USA in the past decades (we add rank based on number of fatalities and another rank for damage dome):


```r
df.total <- df.clean %>% 
  group_by(evtype) %>% # calculate total for each event
  summarise(injuries = sum(injuries),
            fatalities = sum(fatalities),
            fatalities_and_injuries = sum(fatalities_and_injuries),
            propdmg_usd = sum(propdmg_usd),
            cropdmg_usd = sum(cropdmg_usd),
            damage_total_usd = sum(dmgtot_usd)) %>% 
  ungroup() %>% 
  # add rank based on fatalities
  arrange(desc(fatalities)) %>% 
  mutate(fatality_rank = row_number()) %>% 
  # add rank based on damage done
  arrange(desc(damage_total_usd)) %>% 
  mutate(damage_rank = row_number())
```

Now lets calculate totals for each weather event and year:

```r
df.total.year <- df.clean %>% 
  group_by(evtype, year) %>% # calculate total for each event and year
  summarise(injuries = sum(injuries),
            fatalities = sum(fatalities),
            fatalities_and_injuries = sum(fatalities_and_injuries),
            propdmg_usd = sum(propdmg_usd),
            cropdmg_usd = sum(cropdmg_usd),
            damage_total_usd = sum(dmgtot_usd)) %>% 
  # bring ranks from previous total table
  left_join(x = .,
            y = df.total %>% select(evtype, fatality_rank, damage_rank),
            by = "evtype")
```


Now lets re-formulate our total tables in order to show:

* totals for weather event that is top ten killer or destroyer
* events that are not among top ten are being under event called "OTHER" 

Total table for "killers":

```r
df.total.killers <- df.total %>% 
  mutate(evtype_ = case_when(fatality_rank <= 10 ~ evtype,  # recode event type
                             T ~ "OTHER WEATHER EVENTS")) %>% 
  group_by(evtype_) %>%  # calculate the totals
  summarise(injuries = sum(injuries),
            fatalities = sum(fatalities),
            fatalities_and_injuries = sum(fatalities_and_injuries),
            propdmg_usd = sum(propdmg_usd),
            cropdmg_usd = sum(cropdmg_usd),
            damage_total_usd = sum(damage_total_usd)) %>% 
  # bring ranks
  left_join(x = .,
            y = df.total %>% select(evtype, fatality_rank),
            by = c("evtype_" = "evtype")) %>% 
  mutate(fatality_rank = replace_na(fatality_rank, 11)) %>%  # other weather events have rank 11
  # calculate percentage for fatalities
  mutate(`fatalities percentage` = fatalities / sum(fatalities) * 100)
```

Total table for "destroyers":

```r
df.total.destroyers <- df.total %>% 
  mutate(evtype_ = case_when(damage_rank <= 10 ~ evtype,  # recode event type
                             T ~ "OTHER WEATHER EVENTS")) %>% 
  group_by(evtype_) %>%  # calculate the totals
  summarise(injuries = sum(injuries),
            fatalities = sum(fatalities),
            fatalities_and_injuries = sum(fatalities_and_injuries),
            propdmg_usd = sum(propdmg_usd),
            cropdmg_usd = sum(cropdmg_usd),
            damage_total_usd = sum(damage_total_usd)) %>% 
  # bring ranks
  left_join(x = .,
            y = df.total %>% select(evtype, damage_rank),
            by = c("evtype_" = "evtype")) %>% 
  mutate(damage_rank = replace_na(damage_rank, 11)) %>%  # other weather events have rank 11
  # calculate percentage for damage done (in USD)
  mutate(`damage USD percentage` = damage_total_usd / sum(damage_total_usd) * 100)
```


Total year table for "killers":

```r
df.total.killers.year <- df.total.year %>% 
  mutate(evtype_ = case_when(fatality_rank <= 10 ~ evtype,  # recode event type
                             T ~ "OTHER WEATHER EVENTS")) %>% 
  group_by(evtype_, year) %>%  # calculate the totals
  summarise(injuries = sum(injuries),
            fatalities = sum(fatalities),
            fatalities_and_injuries = sum(fatalities_and_injuries),
            propdmg_usd = sum(propdmg_usd),
            cropdmg_usd = sum(cropdmg_usd),
            damage_total_usd = sum(damage_total_usd)) %>% 
  # bring ranks
  left_join(x = .,
            y = df.total %>% select(evtype, fatality_rank),
            by = c("evtype_" = "evtype")) %>% 
  mutate(fatality_rank = replace_na(fatality_rank, 11)) # other weather events have rank 11
```


Total table for "destroyers":

```r
df.total.destroyers.year <- df.total.year %>% 
  mutate(evtype_ = case_when(damage_rank <= 10 ~ evtype,  # recode event type
                             T ~ "OTHER WEATHER EVENTS")) %>% 
  group_by(evtype_, year) %>%  # calculate the totals
  summarise(injuries = sum(injuries),
            fatalities = sum(fatalities),
            fatalities_and_injuries = sum(fatalities_and_injuries),
            propdmg_usd = sum(propdmg_usd),
            cropdmg_usd = sum(cropdmg_usd),
            damage_total_usd = sum(damage_total_usd)) %>% 
  # bring ranks
  left_join(x = .,
            y = df.total %>% select(evtype, damage_rank),
            by = c("evtype_" = "evtype")) %>% 
  mutate(damage_rank = replace_na(damage_rank, 11)) # other weather events have rank 11
```




# Results


### Nature Top Killers


```r
plot1.killers <- df.total.killers %>% 
  select(evtype_, fatality_rank, fatalities) %>% 
  rename(`Weather event` = evtype_) %>% 
  arrange(fatality_rank) %>% 
  mutate(`Weather event` = fct_inorder(`Weather event`)) %>% 
  ggplot(aes(x = `Weather event`, y = fatalities, fill = `Weather event`)) +
  geom_col(color = "black") +
  scale_y_continuous(breaks = seq(0,10000,500)) +
  scale_fill_viridis_d(option = "D") +
  xlab("Type of weather event") +
  ylab("Number of people killed") +
  ggtitle("Top nature killers (Total fatalities count)") +
  theme(axis.text.x = element_text(size = 12, angle = 30),
        axis.text.y = element_text(size = 14),
        axis.title = element_text(size = 16),
        title = element_text(size = 20, face = "bold"))

levels.killers <- df.total.killers %>% arrange(fatality_rank) %>% pull(evtype_)
  
plot2.killers <- df.total.killers.year %>% 
  select(evtype_, fatality_rank, fatalities, year) %>% 
  rename(`Weather event` = evtype_) %>% 
  mutate(`Weather event` = factor(`Weather event`, levels = levels.killers)) %>% 
  ggplot(aes(x = year, y = fatalities, fill = `Weather event`)) +
  geom_area(color = "black") +
  scale_y_continuous(breaks = seq(0,10000,250)) +
  scale_x_continuous(breaks = seq(1950,2020,5)) +
  scale_fill_viridis_d(option = "D") +
  xlab("Year of occurence") +
  ylab("Number of people killed") +
  ggtitle("Top nature killers (Total fatalities count over the years)") +
  theme(axis.text.x = element_text(size = 12, angle = 0),
        axis.text.y = element_text(size = 14),
        axis.title = element_text(size = 16),
        title = element_text(size = 20, face = "bold"),
        legend.position = "none")

  
plot_grid(plot1.killers, plot2.killers, ncol = 2)
```

![](analysis_files/figure-html/naturetopkillers-1.png)<!-- -->


### Nature Top Destroyers

### Killers and Destroyers
