





Exploring the U.S. National Oceanic and Atmospheric Administration's (NOAA) Storm Database
========================================================

## Synopsis
The NOAA database tracks characteristics of major storms and weather events in the United States, including when and where they occur, as well as estimates of any fatalities, injuries, and property damage. In this project we explore the NOAA Storm Database and answer a few basic questions about severe weather events. In particular, we are interested in the following: which types of events are most harmful with respect to population health across the United States, and which types of events have the greatest economic consequences across the United States. We find that, over the period from 1993 through 2011, extreme temperature events resulted in the most fatalities, Illinois being the state that suffered the most (998 fatalities). Texas suffered from the greatest number of flood events, resulting in the most number of injuries (6,951). The state of California suffered the most property damages, mainly from flood events ($117.4 billion). 

## Organization
This document is organized as follows
- *Data Preprocessing.* This involves downloading the NOAA data and loading it into `R`, and can be time consuming. (The zipped database is about 49 Mb and unzips to about one-half Gb `csv` file).
- *Analysis*
  - *Categorization of Storm Events.* The NOAA data contains a vast number of event categories (variable `EVTYPE`) - 985 categories. We group these events into 5 major categories: `Convection`, `Extreme Temperatures`, `Flood`, `Winter`, and ` Other`. The rationale for choosing these categories is drawn from [this document] (http://www.ncdc.noaa.gov/oa/climate/sd/annsum2009.pdf).
  - *Property and Crop Damages*
  - *Injuries and Fatalities*
- *Results*

## Data Processing
### Getting Data
The data (in `bz2` zipped format) were downloaded from [this url] (https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2) and stored locally as a `csv` (comma-separated values) file. The following piece of `R` code accomplishes this:


```r
if (!file.exists("data")) dir.create("data")

if (!file.exists("./data/stormData.csv")) {
    temp <- tempfile()
    fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2"
    download.file(fileUrl, temp)
    data <- read.csv(bzfile(temp))
    write.csv(data, file = "./data/stormData.csv", row.names = FALSE)
    unlink(temp)
}
```


We read the file into `R` and check the first few rows (of `902,297` total rows).
Of the total of `37` variables only a handful will be of interest to us.


```
## [1] 902297     37
```

```
##   STATE__           BGN_DATE BGN_TIME TIME_ZONE COUNTY COUNTYNAME STATE
## 1       1  4/18/1950 0:00:00     0130       CST     97     MOBILE    AL
## 2       1  4/18/1950 0:00:00     0145       CST      3    BALDWIN    AL
## 3       1  2/20/1951 0:00:00     1600       CST     57    FAYETTE    AL
## 4       1   6/8/1951 0:00:00     0900       CST     89    MADISON    AL
## 5       1 11/15/1951 0:00:00     1500       CST     43    CULLMAN    AL
## 6       1 11/15/1951 0:00:00     2000       CST     77 LAUDERDALE    AL
```



### Preprocessing

We begin with the `csv` file that was created by running the previous step. We 
convert the data to a `data.table` (using package `data.table`) for faster processing. 
We then extract those observations that contain entries for fatalities, injuries,
crop damage, or property damage:

A few things we do at the beginning:
- Convert the data to a `data.table` (package `data.table`) for faster processing
- Extract those observations that contain entries for fatalities, injuries,
crop damage, or property damage
- Further restrict the observations to the 50 US states only 
- We will need to clean up the `EVTYPE` (`event type`) column of this data. The
previous step saved us from a lot of unnecessary cleanup by removing non-essential data,






After the preprocessing step, the dataset has been reduced to 253,210 observations - 
a size a bit more manageable than that of the original dataset.

We next create the 5 different categories for the `Event Types`, and find the events 
that fall into these 5 categories. Also, create *intidactor variables* for each 
of the categories. For any event, its indicator variable takes on the value of 
`TRUE` if that occurred and `FALSE` otherwise. For example `FloodI` indicator shows 
whether the event in question was categorizes as a `Flood` event or not.










For example, the following events fall into the `Extreme Temperatures` category:

```r
uniqueEventTypes <- unique(dt[indicator, evtype])
show(uniqueEventTypes[order(uniqueEventTypes)])
```

```
##  [1] Cold                      COLD                     
##  [3] COLD AIR TORNADO          COLD AND SNOW            
##  [5] COLD AND WET CONDITIONS   Cold Temperature         
##  [7] COLD WAVE                 COLD WEATHER             
##  [9] COLD/WIND CHILL           COLD/WINDS               
## [11] DROUGHT/EXCESSIVE HEAT    EXCESSIVE HEAT           
## [13] Extended Cold             Extreme Cold             
## [15] EXTREME COLD              EXTREME COLD/WIND CHILL  
## [17] EXTREME HEAT              FOG AND COLD TEMPERATURES
## [19] HEAT                      Heat Wave                
## [21] HEAT WAVE                 HEAT WAVE DROUGHT        
## [23] HEAT WAVES                HIGH WINDS/COLD          
## [25] RECORD COLD               RECORD HEAT              
## [27] RECORD/EXCESSIVE HEAT     SNOW/ BITTER COLD        
## [29] SNOW/COLD                 Unseasonable Cold        
## [31] UNSEASONABLY COLD        
## 471 Levels:  FLASH FLOOD  TSTM WIND  TSTM WIND (G45) ... WINTRY MIX
```













And here are the results cross-tabulated:


```
##     ConvectionI ExtremeTempI FloodI WinterI OtherI      N
##  1:        TRUE         TRUE  FALSE   FALSE  FALSE    208
##  2:        TRUE        FALSE   TRUE    TRUE  FALSE      1
##  3:        TRUE        FALSE   TRUE   FALSE  FALSE     19
##  4:        TRUE        FALSE  FALSE    TRUE  FALSE     11
##  5:        TRUE        FALSE  FALSE   FALSE  FALSE 208626
##  6:       FALSE         TRUE  FALSE    TRUE  FALSE      4
##  7:       FALSE         TRUE  FALSE   FALSE  FALSE   1227
##  8:       FALSE        FALSE   TRUE    TRUE  FALSE     96
##  9:       FALSE        FALSE   TRUE   FALSE  FALSE  33038
## 10:       FALSE        FALSE  FALSE    TRUE  FALSE   4845
## 11:       FALSE        FALSE  FALSE   FALSE   TRUE   5135
```



```
##               Category      N
## 1:          Convection 208865
## 2:               Flood  33134
## 3:              Winter   4845
## 4:               Other   5135
## 5: Extreme Temperature   1231
```



This reveals that a disproportionate number of events fall under the `Convection` 
category. Perhaps there were more observations taken for this category.

Looking at the dates more carefully, we see that:


```
## Earliest observation for Event Convection:  1950
```

```
## Earliest observation for Event that is NOT Convection:  1993
```


Thus we will be considering data from 1993 to 2011. This further reduces the 
number of observations:


```r
dim(dt)
```

```
## [1] 225836     43
```

## Results 
### Damage Assessment
The property and crop damages inflicted by the storms during the years of 1993-2011 were 
as follows (`B` stands for `billion`, `M`- for `million`, `K` - for `thousand`, all in USD): 


```
##     propdmgexp      N
##  1:             10076
##  2:          B     39
##  3:          K 206990
##  4:          M   8478
##  5:          +      5
##  6:          0    210
##  7:          5     18
##  8:          6      3
##  9:          4      4
## 10:          H      7
## 11:          2      1
## 12:          7      3
## 13:          3      1
## 14:          -      1
```

```
##     Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
## 0.00e+00 2.00e+03 8.00e+03 1.74e+06 3.00e+04 1.15e+11
```

```
##    cropdmgexp      N
## 1:            124392
## 2:          M   1961
## 3:          K  99453
## 4:          B      7
## 5:          ?      6
## 6:          0     17
```

```
##     Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
## 0.00e+00 0.00e+00 0.00e+00 2.14e+05 0.00e+00 5.00e+09
```



### Fatalities and Injuries







#### Fatalities
The fatalities resulting from extreme weather over this period are shown in the 
following table. For example, the state of Illinois suffered the most 
fatalities, 998, due to extreme temperatures:


```
##    state value            Category
## 1:    IL   998 Extreme Temperature
## 2:    IL   181          Convection
## 3:    IL    24               Flood
## 4:    IL    20              Winter
## 5:    IL     4               Other
```


#### Injuries
The injuries resulting from extreme weather are also shown. For example, the 
state of Texas suffered 6,951 injuries as a result of floods over this period:


```
##    state value            Category
## 1:    TX 6,951               Flood
## 2:    TX 1,997          Convection
## 3:    TX   787 Extreme Temperature
## 4:    TX   211              Winter
## 5:    TX   186               Other
```


### Plots
<img src="figure/plotResults.png" title="plot of chunk plotResults" alt="plot of chunk plotResults" style="display: block; margin: auto;" />




### Acknowledgments:
* The rationale for selecting these 5 categories was drawn [from here] (http://www.ncdc.noaa.gov/oa/climate/sd/annsum2009.pdf). And the ideas and methods on how to do this using regular expressions were gleaned from various sources, including the discussion forums at `stackoverflow` and coursera's discussion forums ([e.g.this](https://class.coursera.org/repdata-002/forum/thread?thread_id=32) or [this](https://class.coursera.org/repdata-002/forum/thread?thread_id=46)).
