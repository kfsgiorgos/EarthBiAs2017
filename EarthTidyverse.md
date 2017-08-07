Introduction to **`tidyverse`** ecosystem <br>
================
Georgios Kaiafas, PhD student <br> mail: <georgios.kaiafas@uni.lu>
EarthBiAs2017, Rhodes Island, Greece

-   [Introduction](#introduction)
    -   [Load the *core* `tidyverse` packages](#load-the-core-tidyverse-packages)
    -   [`tibble`](#tibble)
    -   [`dplyr`, `magrittr` & `lubridate`](#dplyr-magrittr-lubridate)
    -   [Update columns](#update-columns)
    -   [Create new columns](#create-new-columns)
    -   [Update column conditionally](#update-column-conditionally)
    -   [Grouped summaries](#grouped-summaries)
-   [Explore missing values](#explore-missing-values)
    -   [Group by *NA* values](#group-by-na-values)
    -   [`filter`, `group_by` & count with: **`tally()`** and **`count()`**](#filter-group_by-count-with-tally-and-count)

Introduction
============

### Load the *core* `tidyverse` packages

``` r
library(tidyverse)
```

    ## Loading tidyverse: ggplot2
    ## Loading tidyverse: tibble
    ## Loading tidyverse: tidyr
    ## Loading tidyverse: readr
    ## Loading tidyverse: purrr
    ## Loading tidyverse: dplyr

    ## Conflicts with tidy packages ----------------------------------------------

    ## filter(): dplyr, stats
    ## lag():    dplyr, stats

### `tibble`

R convert columns that are detected to be character/strings to be factor variables. **`tibble`** has as default mode the `stringsAsFactors = FALSE`

``` r
envData <- as_tibble(readRDS("/home/kfsgiorgos/EarthBias2017/spanishPrecipRecords.RDS"))
stationsData <- as_tibble(readRDS("/home/kfsgiorgos/EarthBias2017/precipStations.RDS"))
dplyr::glimpse(envData)
```

    ## Observations: 3,351,644
    ## Variables: 5
    ## $ STAID <int> 229, 229, 229, 229, 229, 229, 229, 229, 229, 229, 229, 2...
    ## $ SOUID <int> 709, 709, 709, 709, 709, 709, 709, 709, 709, 709, 709, 7...
    ## $ DATE  <int> 19550101, 19550102, 19550103, 19550104, 19550105, 195501...
    ## $ RR    <int> 35, 194, 0, 168, 61, 0, 0, 82, 72, 0, 97, 0, 0, 51, 5, 1...
    ## $ Q_RR  <int> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,...

-   Tibbles have a refined print method that shows only the first 10 rows, and all the columns that fit on screen.
-   Tibbles are one of the unifying features of the `tidyverse`.
-   `tibble()` in contrast to `data.frame()` does much less: it *never changes the type of the inputs* (e.g. it never converts strings to factors!), it *never changes the names of variables, and it never creates row names.*
-   Tibble subsetting **\[** always returns another tibble. Data frame subsetting **\[** sometimes returns a data frame and sometimes it just returns a vector.

``` r
class(envData[, 1:2])
```

    ## [1] "tbl_df"     "tbl"        "data.frame"

``` r
class(envData[, 1])
```

    ## [1] "tbl_df"     "tbl"        "data.frame"

### `dplyr`, `magrittr` & `lubridate`

**`dplyr`** package works with structured data and is built around 5 verbs which make up the majority of the data manipulation.

-   **`Select`** certain columns.
-   **`Filter`** to select specific rows.
-   **`Arrange`** the rows into an order.
-   **`Mutate`** to create new columns.
-   **`Summarise`** many values down to a single summary.

**`magrittr`** comes with the pipe **`%>%`** and it is a powerful tool for chaining multiple operations. The point of the pipe is to help you write code in a way that easier to read and understand.

**`lubridate`** simplifies parsing dates and times by handling a wide variety of formats and seperators. It is not part of core tidyverse so we have to load it manually.

Conditional subset using **`dplyr`**

``` r
dplyr::filter(envData, STAID == "229" & RR > 100)
```

    ## # A tibble: 956 x 5
    ##    STAID SOUID     DATE    RR  Q_RR
    ##    <int> <int>    <int> <int> <int>
    ##  1   229   709 19550102   194     0
    ##  2   229   709 19550104   168     0
    ##  3   229   709 19550129   150     0
    ##  4   229   709 19550215   150     0
    ##  5   229   709 19550216   138     0
    ##  6   229   709 19550222   124     0
    ##  7   229   709 19550225   120     0
    ##  8   229   709 19550314   153     0
    ##  9   229   709 19550319   120     0
    ## 10   229   709 19550523   113     0
    ## # ... with 946 more rows

Conditional subset using **`dplyr`** & **`magrittr`**

``` r
envData %>% dplyr::filter(STAID %in% c("236", "230") & RR > 100)
```

    ## # A tibble: 2,877 x 5
    ##    STAID SOUID     DATE    RR  Q_RR
    ##    <int> <int>    <int> <int> <int>
    ##  1   230   710 19200218   245     0
    ##  2   230   710 19200219   262     0
    ##  3   230   710 19200220   144     0
    ##  4   230   710 19200513   384     0
    ##  5   230   710 19200528   229     0
    ##  6   230   710 19200627   139     0
    ##  7   230   710 19201005   133     0
    ##  8   230   710 19201006   153     0
    ##  9   230   710 19201013   114     0
    ## 10   230   710 19201024   159     0
    ## # ... with 2,867 more rows

### Update columns

We can easily parse dates using **`lubridate`** functions and reassign the result with **`mutate`** verb

``` r
# load lubridate
library(lubridate)
```

``` r
envData <- dplyr::mutate(envData, DATE = lubridate::ymd(DATE))
glimpse(envData)
```

    ## Observations: 3,351,644
    ## Variables: 5
    ## $ STAID <int> 229, 229, 229, 229, 229, 229, 229, 229, 229, 229, 229, 2...
    ## $ SOUID <int> 709, 709, 709, 709, 709, 709, 709, 709, 709, 709, 709, 7...
    ## $ DATE  <date> 1955-01-01, 1955-01-02, 1955-01-03, 1955-01-04, 1955-01...
    ## $ RR    <int> 35, 194, 0, 168, 61, 0, 0, 82, 72, 0, 97, 0, 0, 51, 5, 1...
    ## $ Q_RR  <int> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,...

### Create new columns

When we are dealing with dates, there are two appoaches on how to create new columns.

-   First approach: use **`tidyr`** package to **separate** the DATE column and each part is a new column.

``` r
# Track the computational time for seperate operation
approach1 <- system.time(envData1 <- tidyr::separate(envData, DATE, c("Year", "Month", "Day")))
dplyr::glimpse(envData1)
```

    ## Observations: 3,351,644
    ## Variables: 7
    ## $ STAID <int> 229, 229, 229, 229, 229, 229, 229, 229, 229, 229, 229, 2...
    ## $ SOUID <int> 709, 709, 709, 709, 709, 709, 709, 709, 709, 709, 709, 7...
    ## $ Year  <chr> "1955", "1955", "1955", "1955", "1955", "1955", "1955", ...
    ## $ Month <chr> "01", "01", "01", "01", "01", "01", "01", "01", "01", "0...
    ## $ Day   <chr> "01", "02", "03", "04", "05", "06", "07", "08", "09", "1...
    ## $ RR    <int> 35, 194, 0, 168, 61, 0, 0, 82, 72, 0, 97, 0, 0, 51, 5, 1...
    ## $ Q_RR  <int> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,...

-   Second approach: use **`mutate`** verb to create the new columns manually.

``` r
# Track the computational time for each operation
appoach2 <- system.time(envData <- dplyr::mutate(envData, Year = lubridate::year(DATE), Month = lubridate::month(DATE), Day = lubridate::day(DATE)))
dplyr::glimpse(envData)
```

    ## Observations: 3,351,644
    ## Variables: 8
    ## $ STAID <int> 229, 229, 229, 229, 229, 229, 229, 229, 229, 229, 229, 2...
    ## $ SOUID <int> 709, 709, 709, 709, 709, 709, 709, 709, 709, 709, 709, 7...
    ## $ DATE  <date> 1955-01-01, 1955-01-02, 1955-01-03, 1955-01-04, 1955-01...
    ## $ RR    <int> 35, 194, 0, 168, 61, 0, 0, 82, 72, 0, 97, 0, 0, 51, 5, 1...
    ## $ Q_RR  <int> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,...
    ## $ Year  <dbl> 1955, 1955, 1955, 1955, 1955, 1955, 1955, 1955, 1955, 19...
    ## $ Month <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,...
    ## $ Day   <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 1...

-   The comparison between the two approaches shows that the second is by far more efficient.

<!-- -->

    ## [1] "1st approach time: 18.76 vs 2nd approach time: 12"

### Update column conditionally

We update only the values of column RR which are equal to -9999 with NA in order to take advantage **`dplyr`** functions that work with NA. Also, we make a double check if the operation worked as it was supposed to work.

``` r
print(dim(filter(envData, RR == -9999)))
```

    ## [1] 260169      8

``` r
# For the rows where RR variable is equal to -9999 we update their values with NA
envData <- envData %>% mutate(RR = ifelse(RR == -9999, NA, RR))
print(dim(filter(envData, is.na(RR))))
```

    ## [1] 260169      8

### Grouped summaries

-   Find *Rainfall quantiles* for each Station and for each month of the year by excluding *NA* values and print the result in desceding order

``` r
RRquantiles <- envData %>% group_by(Year, Month, STAID) %>% summarise(quantiles = quantile(RR, na.rm = T, probs = 0.9)) %>% arrange(-quantiles)
RRquantiles
```

    ## # A tibble: 110,121 x 4
    ## # Groups:   Year, Month [1,445]
    ##     Year Month STAID quantiles
    ##    <dbl> <dbl> <int>     <dbl>
    ##  1  1979     1  1388    1140.0
    ##  2  2015     3 11064     851.0
    ##  3  1965    10  3920     804.0
    ##  4  1965    10  3930     804.0
    ##  5  1965    10 11040     804.0
    ##  6  1978    12  1394     726.0
    ##  7  1962     3  1395     716.0
    ##  8  1989    11   231     672.3
    ##  9  1947     3  3955     662.0
    ## 10  1978    12  1395     650.0
    ## # ... with 110,111 more rows

Explore missing values
======================

### Group by *NA* values

In what extend *NA* values affect our analysis? We calculate the missing days per Year per month per Station with decreasing order and return a subset of columns using **`select`**

``` r
Missingdaysofmonths <- envData %>% group_by(Year, Month, STAID) %>% 
  count(is.na(RR)) %>% arrange(-n) %>% 
  select(Year, Month, STAID, n)
Missingdaysofmonths
```

    ## # A tibble: 111,521 x 4
    ## # Groups:   Year, Month, STAID [110,121]
    ##     Year Month STAID     n
    ##    <dbl> <dbl> <int> <int>
    ##  1  1896    12  3920    31
    ##  2  1896    12  3930    31
    ##  3  1896    12 11040    31
    ##  4  1897     1  3920    31
    ##  5  1897     1  3930    31
    ##  6  1897     1 11040    31
    ##  7  1897     3  3920    31
    ##  8  1897     3  3930    31
    ##  9  1897     3 11040    31
    ## 10  1897     5  3920    31
    ## # ... with 111,511 more rows

### `filter`, `group_by` & count with: **`tally()`** and **`count()`**

`filter` by given threshold of NA values (20 days per month) that are included in each observation of the previous dataset.

-   `tally` calculates the total number of NA values for every Station every Year

``` r
MissingYeardays <- Missingdaysofmonths %>% filter(n > 20) %>% group_by(STAID, Year) %>% tally() #%>% arrange(-n)
```

    ## Using `n` as weighting variable

``` r
MissingYeardays
```

    ## # A tibble: 9,371 x 3
    ## # Groups:   STAID [?]
    ##    STAID  Year    nn
    ##    <int> <dbl> <int>
    ##  1   229  1955   365
    ##  2   229  1956   366
    ##  3   229  1957   365
    ##  4   229  1958   365
    ##  5   229  1959   365
    ##  6   229  1960   366
    ##  7   229  1961   365
    ##  8   229  1962   365
    ##  9   229  1963   365
    ## 10   229  1964   366
    ## # ... with 9,361 more rows

-   `count` calculates the number of months for every Station every Year that fulfil the condition `"n > 20"`

``` r
MissingYeardays1 <- Missingdaysofmonths %>% filter(n > 20) %>% group_by(STAID, Year) %>% count() #%>% arrange(-n)
MissingYeardays1
```

    ## # A tibble: 9,371 x 3
    ## # Groups:   STAID, Year [9,371]
    ##    STAID  Year    nn
    ##    <int> <dbl> <int>
    ##  1   229  1955    12
    ##  2   229  1956    12
    ##  3   229  1957    12
    ##  4   229  1958    12
    ##  5   229  1959    12
    ##  6   229  1960    12
    ##  7   229  1961    12
    ##  8   229  1962    12
    ##  9   229  1963    12
    ## 10   229  1964    12
    ## # ... with 9,361 more rows

``` r
# dplyr::
```
