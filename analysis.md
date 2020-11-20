---
title: "USA extreme weather events exploration"
author: "M.I."
date: "11/19/2020"
output: 
  html_document:
    keep_md: true
    number_sections: yes
    toc: yes
    toc_float: yes
---



## Synopsis

In this report we would like to analyze **severe weather events** in the **USA** for the past decades. For data analysis we are using the **[NOAA](https://www.noaa.gov/) Storm Database** . With provided analysis we would like to provide answers for given questions below:

1. Across the United States, which types of events are most harmful with respect to population health?
2. Across the United States, which types of events have the greatest economic consequences?

Main tools used for providing answers is **exploratory data analysis** (**EDA**). Main focus were variables indicating harm done to people (fatalities and injures) and damage done to property or crops. We have identified top 10 nature events that have the highest death rate (**Top nature killers**) in the USA. Also top 10 nature events, that caused the greatest economic consequences as for damage done (**Top nature destroyers**), were identified. 

Data is provided for a time span between years 1950 and 2011 and data is gathered for each state and event. Therefore additional summaries were calculated in order to provide adequate insights.



## Data Processing

In this part we will describe all the transformations applied to raw data, including summaries calculated.

### Import raw data

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
              "datasets",
              "scales") 

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


### Initial data inspection

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


How many different event types are in the data set:

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



### Data wrangling

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



### Calculate summaries

In order to find most devastating weather events in observed time period we will first calculate total of:

* injuries and/or fatalities
* damage done

For the entire USA and complete time span. Based on this summary we will be able to find "Top Killers" and "Top Destroyers", that have caused most trouble for the USA in the past decades (we add rank based on number of fatalities and another rank for damage done):


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


Now lets re-formulate our total tables in order to show:

* totals for weather event that is top ten killer or destroyer
* events that are not among top ten are being under event called **"OTHER WEATHER EVENTS"**

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


### Data for the plots

We will create data for first plot:


```r
plot.killers.data <- df.total.killers %>% 
  select(evtype_, fatality_rank, injuries, fatalities) %>% 
  rename(`Weather event` = evtype_) %>% 
  arrange(fatality_rank) %>% 
  mutate(`Weather event` = fct_inorder(`Weather event`)) %>% 
  pivot_longer(cols = c("injuries", "fatalities"), 
               names_to = "Harm type", 
               values_to = "Number of people") %>% 
  mutate(`Harm type` = factor(`Harm type`, 
                              levels = c("fatalities", "injuries"), 
                              labels = c("killed", "injured"))) 
```


Data for second plot:

```r
plot.destroyers.data <- df.total.destroyers %>% 
  select(evtype_, damage_rank, damage_total_usd) %>% 
  rename(`Weather event` = evtype_) %>% 
  arrange(damage_rank) %>% 
  mutate(`Weather event` = fct_inorder(`Weather event`),
         damage_total_usd_billions = damage_total_usd / 1000000000) # convert damage to billion USD
```

Data for third plot:


```r
plot.scatter <- df.total %>% 
  # create new variable weather group event (keep event name if event is top killer or top destroyer)
  mutate(`Weather event` = case_when(fatality_rank <= 5 | damage_rank <= 5 ~ evtype,
                                     T ~ "OTHER WEATHER EVENTS"),
         damage_total_usd_billions = damage_total_usd / 1000000000) %>%  # convert damage to billion USD
  # Factor variable (fatality rank considered)
  mutate(rank_ = case_when(`Weather event` == "OTHER WEATHER EVENTS" ~ df.total %>% pull(fatality_rank) %>% max(.),
                          `Weather event` != "OTHER WEATHER EVENTS" ~ fatality_rank)) %>% 
  arrange(rank_) %>% 
  mutate(`Weather event` = fct_inorder(`Weather event`))

colors.pal <- viridis_pal(option = "A")(plot.scatter %>% 
                                          filter(`Weather event` != "OTHER WEATHER EVENTS") %>% 
                                          nrow()) %>% 
  c(., "gray")
```




## Results

In this section the results are shown. First we will highlight top nature's killers, then we will show top nature destroyers and finally all events are drawn on a single scatter plot showing all relevant counts.

### Nature's Top Killers

Figure below shows how many people were killed and injured by a given severe weather event (all years given - total count). In order to have a meaningful plot, we are showing top ten events and all other events are grouped in a single event called "OTHER WEATHER EVENTS" (Total number of different events is 952). We can see that the most deadliest weather event in the US history are tornadoes that have killed 5633 people and injured 91.346 thousands of people. Other devastating weather events are heat, floods, lightning, avalanches and others. Top ten weather events have killed 11.977 thousands people, which corresponds to 80.7% of all people, that were killed by a severe weather event.  


```r
plot.killers.data %>%  
  ggplot(aes(x = `Weather event`, y = `Number of people`, fill = `Weather event`)) +
  geom_col(color = "black") +
  scale_fill_viridis_d(option = "D") +
  facet_grid(`Harm type` ~ ., scales = "free") +
  xlab("Type of weather event") +
  ylab("Number of people") +
  ggtitle("Top nature killers (total fatalities & injuries count)") +
  theme(axis.text.x = element_text(size = 12, angle = 20),
        axis.text.y = element_text(size = 14),
        axis.title = element_text(size = 16),
        strip.text = element_text(size = 18),
        title = element_text(size = 18, face = "bold"))
```

![](analysis_files/figure-html/naturetopkillers-1.png)<!-- -->


### Nature's Top Destroyers

Figure below shows the total damage done (in USD dollars) to property or crops by a given severe weather event (all years given - total count). In order to have a meaningful plot, we are showing top ten events and all other events are grouped in a single event called "OTHER WEATHER EVENTS" (Total number of different events is 952). We can see that the most destructive weather event in the US history are floods that have done damage measured in 150.1452873 billions of USD dollars. Other top most destructive weather events are hurricanes, tornadoes, storms , drought, and so on. Top ten weather events have made a total damage of 404.7062859 billions of USD dollars, which corresponds to 85.7% of all damage done (measured in USD dollars), that were caused by a severe weather event.  




```r
plot.destroyers.data %>%  
  ggplot(aes(x = `Weather event`, y = damage_total_usd_billions, fill = `Weather event`)) +
  geom_col(color = "black") +
  scale_y_continuous(breaks = seq(0,1000,10)) +
  scale_fill_viridis_d(option = "D") +
  xlab("Type of weather event") +
  ylab("Damage done in billions USD dollars") +
  ggtitle("Top nature destroyers (total property & crop damage done)") +
  theme(axis.text.x = element_text(size = 12, angle = 20),
        axis.text.y = element_text(size = 14),
        axis.title = element_text(size = 16),
        strip.text = element_text(size = 18),
        title = element_text(size = 18, face = "bold"))
```

![](analysis_files/figure-html/naturetopdestroyers-1.png)<!-- -->

### Nature's deadliest tools

On final the plot shown below we try to highlight 3 aspects of severe weather events:

* total people injured (x-axis)
* total people killed (y-axis)
* total damage done in USD dollars (size of the points)

In order to do so we have drawn a scatter plot, where each weather event is shown with a certain point. From top killers and top destroyers we have selected the top 5 most ranking events. These events have a unique color and unique name, all other events are colored with gray color and are grouped in a event called "OTHER WEATHER EVENTS". Interesting thing is the fact, when looking at the top 5 in both groups returns 9 different events. Meaning that rankings are different and if a certain event is a top killer does not mean it is also a top destroyer. In order to have a meaningful plot we had to apply logarithm with base 10 transformation on both axis.


```r
plot.scatter %>% 
  ggplot(aes(x = injuries + 1, 
             y = fatalities + 1, 
             size = damage_total_usd_billions, 
             color = `Weather event`)) +
  geom_point(alpha = 1/2) +
  geom_text(data = plot.scatter %>% 
              filter(`Weather event` != "OTHER WEATHER EVENTS"), 
            aes(label = `Weather event`), size = 7, show.legend = F) +
  scale_x_log10(limits = c(1,1e7)) +
  scale_y_log10(limits = c(1,1.5e4)) +
  scale_size(range = c(1, 80)) +
  scale_color_manual(values = colors.pal) +
  labs(x = "Number of people injured (log 10 scale)", 
       y = "Number of people killed (log 10 scale)",
       title = "Deadliest nature weather events",
       subtitle = "Point size ~ total damage done in USD") +
  guides(size=F) +
  theme(axis.text.x = element_text(size = 12, angle = 20),
        legend.text = element_text(size = 14),
        legend.title = element_text(size = 16),
        axis.text.y = element_text(size = 14),
        axis.title = element_text(size = 16),
        plot.title = element_text(size = 18, face = "bold"))
```

![](analysis_files/figure-html/scatteplot-1.png)<!-- -->
