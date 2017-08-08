Introduction to **`tidyverse`** ecosystem <br>
================
Georgios Kaiafas, PhD student <br> mail: <georgios.kaiafas@uni.lu>
<br> EarthBiAs2017, Rhodes Island, Greece

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
    -   [Quick geospatial view](#quick-geospatial-view)
    -   [Reshape into wide layout](#reshape-into-wide-layout)
    -   [Missing value summaries](#missing-value-summaries)

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
appoach2 <- system.time(envData <- dplyr::mutate(envData, 
                                                 Year = lubridate::year(DATE), 
                                                 Month = lubridate::month(DATE), 
                                                 Day = lubridate::day(DATE)))
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

    ## [1] "1st approach time: 19.96 vs 2nd approach time: 12"

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

In what extend *NA* values affect our analysis? <br>
We calculate the missing days per Year per month per Station with decreasing order and return a subset of columns using **`select`**

``` r
Missingdaysofmonths <- envData %>% 
  group_by(Year, Month, STAID) %>% 
  count(is.na(RR)) %>% 
  arrange(-n) %>% 
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
MissingYeardays <- Missingdaysofmonths %>% filter(n > 20) %>% group_by(STAID, Year) %>% tally()
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
MissingYeardays1 <- Missingdaysofmonths %>% filter(n > 20) %>% group_by(STAID, Year) %>% count()
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

### Quick geospatial view

-   Where are located the stations that per year contain *`NAs > 70%`* ? <br> We are going to use the `MissingYeardays` dataset and `filter` by the above condition. We use the `distinct` verb to get the stations which have at least one year that satisfies our condition. `inner_join` with the `stationsData` will give us the *lattitude* and *longitude* variables. **`leaflet`** package will plot for us the geolocation of the result.

``` r
stations <- MissingYeardays %>% filter (nn > 0.7 * 365) %>% distinct(STAID)
geostations <- inner_join(stations, select(stationsData, STAID, STANAME, CN, latUpd, longUpd), by = "STAID")
options(tibble.width = Inf)
geostations
```

    ## # A tibble: 200 x 5
    ## # Groups:   STAID [?]
    ##    STAID                                  STANAME    CN   latUpd    longUpd
    ##    <int>                                    <chr> <chr>    <dbl>      <dbl>
    ##  1   229 BADAJOZ/TALAVERA LA REAL                    ES 38.88306 -5.1708333
    ##  2   230 MADRID - RETIRO                             ES 40.41167 -2.3219444
    ##  3   231 MALAGA AEROPUERTO                           ES 36.66667 -3.5119444
    ##  4   232 NAVACERRADA                                 ES 40.78056 -3.9897222
    ##  5   233 SALAMANCA AEROPUERTO                        ES 40.95917 -4.5019444
    ##  6   234 SAN SEBASTIAN - IGUELDO                     ES 43.30750 -1.9608333
    ##  7   235 TORREVIEJA                                  ES 37.97694  0.7105556
    ##  8   236 TORTOSA - OBSERVATORIO DEL EBRO             ES 40.82056  0.4913889
    ##  9   237 VALENCIA                                    ES 39.48056  0.3663889
    ## 10   238 ZARAGOZA AEROPUERTO                         ES 41.66167 -0.9919444
    ## # ... with 190 more rows

plot with leaflet

``` r
leaflet(geostations) %>% 
  addTiles() %>% 
  addAwesomeMarkers(lng = ~longUpd, lat= ~latUpd, popup = ~as.character(STANAME), label = ~as.character(STANAME))
```

![](EarthTidyverse_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-19-1.png)

### Reshape into wide layout

-   We reshape `MissingYeardays` dataset using **spread** verb of *`tidyr`* package.

``` r
### Turn values of Year variable into new columns
spreadtib <- tidyr::spread(MissingYeardays, Year, nn)
options(tibble.width = Inf)
spreadtib
```

    ## # A tibble: 200 x 123
    ## # Groups:   STAID [200]
    ##    STAID `1896` `1897` `1898` `1899` `1900` `1901` `1902` `1903` `1904` `1905` `1906` `1907` `1908` `1909` `1910` `1911` `1912` `1913` `1914` `1915` `1916` `1917` `1918` `1919` `1920` `1921` `1922` `1923` `1924` `1925` `1926` `1927` `1928` `1929` `1930` `1931` `1932` `1933` `1934` `1935` `1936` `1937` `1938` `1939` `1940` `1941` `1942` `1943` `1944` `1945` `1946` `1947` `1948` `1949` `1950` `1951` `1952` `1953` `1954` `1955` `1956` `1957` `1958` `1959` `1960` `1961` `1962` `1963` `1964` `1965` `1966` `1967` `1968` `1969` `1970` `1971` `1972` `1973` `1974` `1975` `1976` `1977` `1978` `1979` `1980` `1981` `1982` `1983` `1984` `1985` `1986` `1987` `1988` `1989` `1990` `1991` `1992` `1993` `1994` `1995` `1996` `1997` `1998` `1999` `2000` `2001` `2002` `2003` `2004` `2005` `2006` `2007` `2008` `2009` `2010` `2011` `2012` `2013` `2014` `2015` `2016` `2017`
    ##  * <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>
    ##  1   229     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366     90
    ##  2   230     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366     90
    ##  3   231     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366     90
    ##  4   232     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366     90
    ##  5   233     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366     90
    ##  6   234     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366     90
    ##  7   235     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    364    364    366    365    365    365    366    365    365    365    366    365    365    365    366     90
    ##  8   236     NA     NA     NA     NA     NA     NA     NA     NA     NA    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    331    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366     90
    ##  9   237     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     61    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    364    365    366    365    365    365    366    365    365    365    366     90
    ## 10   238     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366    365    365    365    366     90
    ## # ... with 190 more rows

### Missing value summaries

Before we dive deeper into the missing values pool we have to shape the status of *NAs* in our dataset.

-   We calculate the ratio of *NAs* per station per month

``` r
# Divide with correct number of days
Missingdaysofmonths31 <- filter(Missingdaysofmonths, Month %in% c(1, 3, 5, 7, 8, 10, 12))
Missingdaysofmonths31 <- Missingdaysofmonths31 %>% mutate(ratio = n/31)
Missingdaysofmonths30 <- filter(Missingdaysofmonths, Month  %in% c(4, 6, 9, 11))
Missingdaysofmonths30 <- Missingdaysofmonths30 %>% mutate(ratio = n/30)

Missingdaysofmonths28 <- Missingdaysofmonths %>% filter(Month == 2)
Missingdaysofmonths28 <- Missingdaysofmonths28  %>% mutate(ratio = n/27)
# bind_rows can bind more than 2 tibbles
Missingdaysofmonths <- dplyr::bind_rows(Missingdaysofmonths30, 
                                        Missingdaysofmonths31, 
                                        Missingdaysofmonths28) %>% arrange(Month)
Missingdaysofmonths <- Missingdaysofmonths %>% mutate(ratio = ifelse(ratio > 1, 1, ratio))
```

-   We can give as input the monthly ratio of *`NAs`* and/or the year ratio

``` r
thresdata <- Missingdaysofmonths %>% 
  filter(ratio < 0.5) %>% 
  group_by(STAID, Year) %>% 
  summarise(yearRatio = sum(ratio)/12) %>% 
  filter(yearRatio < 0.4)
thresdata
```

    ## # A tibble: 453 x 3
    ## # Groups:   STAID [97]
    ##    STAID  Year   yearRatio
    ##    <int> <dbl>       <dbl>
    ##  1   235  2002 0.002688172
    ##  2   235  2003 0.002688172
    ##  3   236  1938 0.037903226
    ##  4   237  2006 0.002688172
    ##  5   309  2002 0.010752688
    ##  6   335  1913 0.013440860
    ##  7   337  1986 0.035444046
    ##  8   412  1938 0.038888889
    ##  9   412  1941 0.030555556
    ## 10   415  1989 0.029569892
    ## # ... with 443 more rows

-   Also, we can subset by a station and get the results for every year

``` r
Missingdaysofmonths %>% 
  filter(ratio < 0.5) %>% 
  group_by(STAID, Year) %>% 
  summarise(yearRatio = sum(ratio)/12) %>% 
  filter(yearRatio < 0.4 & STAID == "235")
```

    ## # A tibble: 2 x 3
    ## # Groups:   STAID [1]
    ##   STAID  Year   yearRatio
    ##   <int> <dbl>       <dbl>
    ## 1   235  2002 0.002688172
    ## 2   235  2003 0.002688172

-   Why do not we enrich our dataset using the information about the country of each station?

``` r
countryinfo <- inner_join(distinct(thresdata, STAID), 
                          select(stationsData, STAID, CN, STANAME, latUpd, longUpd), by = "STAID")
countryinfo
```

    ## # A tibble: 97 x 5
    ## # Groups:   STAID [?]
    ##    STAID    CN                                  STANAME   latUpd    longUpd
    ##    <int> <chr>                                    <chr>    <dbl>      <dbl>
    ##  1   235    ES TORREVIEJA                               37.97694  0.7105556
    ##  2   236    ES TORTOSA - OBSERVATORIO DEL EBRO          40.82056  0.4913889
    ##  3   237    ES VALENCIA                                 39.48056  0.3663889
    ##  4   309    ES ALICANTE EL ALTET                        38.28278  0.5705556
    ##  5   335    ES BARCELONA -FABRA OBSERVATORY             41.41806  2.1238889
    ##  6   337    ES CORDOBA AEROPUERTO                       37.84417 -3.1541667
    ##  7   412    ES ALICANTE                                 38.37250  0.4941667
    ##  8   415    ES CADIZ                                    36.50083 -5.7433333
    ##  9   416    ES CIUDAD REAL                              38.98917 -2.0805556
    ## 10   418    ES HUELVA (RONDA DEL ESTE)                  37.28000 -5.0905556
    ## # ... with 87 more rows

plot with leaflet

``` r
leaflet(as.data.frame(countryinfo)) %>% 
  addTiles() %>% 
  addAwesomeMarkers(lng = ~longUpd, lat= ~latUpd, popup = ~as.character(STANAME), label = ~as.character(STAID))
```

![](EarthTidyverse_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-25-1.png)

-   We calculate the Consecutive years for a station which contain `NAs` above a given threshold

``` r
library(data.table)
```

    ## 
    ## Attaching package: 'data.table'

    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     between, first, last

    ## The following object is masked from 'package:purrr':
    ## 
    ##     transpose

First step:

``` r
# Total missing days per year for all stations
MissingdaysYear <- Missingdaysofmonths %>% filter(ratio>0.2) %>% 
  group_by(STAID, Year) %>% 
  summarise(Totaldays = sum(n))
# new variable with 1 year lag
laged <- dplyr::mutate(MissingdaysYear, Yearlag1 = lag(x = Year, n=1L))
### First and Last observation for each group, just for illustration purpose
laged %>%  slice(c(1,n()))
```

    ## # A tibble: 400 x 4
    ## # Groups:   STAID [200]
    ##    STAID  Year Totaldays Yearlag1
    ##    <int> <dbl>     <int>    <dbl>
    ##  1   229  1955       365       NA
    ##  2   229  2017        90     2016
    ##  3   230  1920       366       NA
    ##  4   230  2017        90     2016
    ##  5   231  1942       365       NA
    ##  6   231  2017        90     2016
    ##  7   232  1946       365       NA
    ##  8   232  2017        90     2016
    ##  9   233  1945       365       NA
    ## 10   233  2017        90     2016
    ## # ... with 390 more rows

Second step: Until now it is clear from which *Year* until which *Year* the condition of *`NAs`* is satisfied

``` r
# update the column Yearlag1 Replace NA with 0
lagged <- laged %>% 
  mutate(Yearlag1 = replace(Yearlag1 , is.na(Yearlag1), 0)) %>%
  filter(Year<2017)
### First and Last observation for each group, just for illustration purpose
lagged %>%  slice(c(1,n()))
```

    ## # A tibble: 400 x 4
    ## # Groups:   STAID [200]
    ##    STAID  Year Totaldays Yearlag1
    ##    <int> <dbl>     <int>    <dbl>
    ##  1   229  1955       365        0
    ##  2   229  2016       366     2015
    ##  3   230  1920       366        0
    ##  4   230  2016       366     2015
    ##  5   231  1942       365        0
    ##  6   231  2016       366     2015
    ##  7   232  1946       365        0
    ##  8   232  2016       366     2015
    ##  9   233  1945       365        0
    ## 10   233  2016       366     2015
    ## # ... with 390 more rows

Third step:

``` r
## Crteate a new column as a middle step for consecutive calculation
lagged <- lagged %>% 
  mutate(diffYearlag = Year - Yearlag1)
Station <- "412"
# Any difference greater than 1 is does not mean duration
tibstation <- filter(lagged, STAID == Station & diffYearlag != 1)
tibstation <- dplyr::mutate(tibstation, Yearlead1 = shift(x = Yearlag1, type = "lead"))
# Find the ending year that fulfil the condition
untilwhichYear <- lagged %>% summarise(max(Year)) %>% filter()
# Replace the above value to the dataset
tibstation <- tibstation %>% 
  mutate(Yearlead1 = replace(Yearlead1 , is.na(Yearlead1), as.numeric(filter(untilwhichYear, STAID == Station)[2])))
# Calculate the duration
tibstation <- tibstation %>% 
  mutate(durationYears = Yearlead1 - Year +1)

tibstation %>% select(STAID, Year, Yearlead1, durationYears)
```

    ## # A tibble: 1 x 4
    ## # Groups:   STAID [1]
    ##   STAID  Year Yearlead1 durationYears
    ##   <int> <dbl>     <dbl>         <dbl>
    ## 1   412  1938      2016            79

-   Can we dive deeper? Yes, of course.<br>

We calculate the *`percentage of Nas`* for the duration of the previous dataset using [purrr](https://cran.r-project.org/web/packages/purrr/index.html) a *`core tidyverse`* package So, for each year in the interval *`[Year, Yearlead1]`* we get the NAs from the *`MissingdaysYear`* dataset

``` r
templist <- list()
for (i in 1:length(tibstation$Year)){
  templist[[i]] <- MissingdaysYear %>% filter(STAID == Station & Year >= tibstation$Year[i] & Year <= tibstation$Yearlead1[i])
}
#### calculate the percentage of Missing days for the calculated duration
percentage <- map_dbl(templist, ~ 100 * sum(.x[["Totaldays"]]) / (length(.x[["Totaldays"]]) * 365))

finalresult <- tibstation %>% mutate(PercentageMissing = percentage)
finalresult %>% select(STAID,  Year, Yearlead1, durationYears, PercentageMissing)
```

    ## # A tibble: 1 x 5
    ## # Groups:   STAID [1]
    ##   STAID  Year Yearlead1 durationYears PercentageMissing
    ##   <int> <dbl>     <dbl>         <dbl>             <dbl>
    ## 1   412  1938      2016            79          99.22663
