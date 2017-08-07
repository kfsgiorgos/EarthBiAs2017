Untitled
================

``` r
# Load sparklyr
library(sparklyr)
# Connect to Local Remote Apache Spark cluster
sc      <- spark_connect(master = "local")
```

    ## * Using Spark: 2.2.0

``` r
# Load environmental data
envData <- readRDS("/home/kfsgiorgos/EarthBias2017/spanishPrecipRecords.RDS")
# Copy R object into spark
tbl <- sparklyr::sdf_copy_to(sc          = sc,
                             x           = envData,
                             name        = "envSparlTbl",
                             memory      = TRUE,
                             repartition = 0L,
                             overwrite   = TRUE)
tbl
```

    ## # Source:   table<envSparlTbl> [?? x 5]
    ## # Database: spark_connection
    ##    STAID SOUID     DATE    RR  Q_RR
    ##    <int> <int>    <int> <int> <int>
    ##  1   229   709 19550101    35     0
    ##  2   229   709 19550102   194     0
    ##  3   229   709 19550103     0     0
    ##  4   229   709 19550104   168     0
    ##  5   229   709 19550105    61     0
    ##  6   229   709 19550106     0     0
    ##  7   229   709 19550107     0     0
    ##  8   229   709 19550108    82     0
    ##  9   229   709 19550109    72     0
    ## 10   229   709 19550110     0     0
    ## # ... with 3.352e+06 more rows
