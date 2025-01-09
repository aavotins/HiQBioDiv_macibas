Otrais uzdevums
================
Betija Rubene
2025-01-08

Darbam nepieciešamo bibliotēku ielāde

``` r
library(sf)
library(sfarrow)
library(microbenchmark)
library(archive)
library(curl)
library(fs)
```

Centra virsmežniecības faila lejupielāde

``` r
url <- "https://data.gov.lv/dati/lv/dataset/40014c0a-90f5-42be-afb2-fe3c4b8adf92/resource/392dfb67-eeeb-43c2-b082-35f9cf986128/download/centra.7z"
dest <- "C:/Users/ruben/Documents/HiQBioDiv_macibas_fork/centra_mezi.7z"

curl_download(url, destfile = dest)
file.exists(dest) # fails ir atrodams direktorijā
```

    ## [1] TRUE

Atarhivēšana

``` r
archive_extract(dest, dir = "C:/Users/ruben/Documents/HiQBioDiv_macibas_fork/centra_mezi")
```

Failu apskate atarhivētajā mapē

``` r
getwd()
```

    ## [1] "C:/Users/ruben/Documents/HiQBioDiv_macibas_fork/Uzd02"

``` r
list.files("../centra_mezi")
```

    ##  [1] "nodala2651.cpg"     "nodala2651.dbf"     "nodala2651.prj"    
    ##  [4] "nodala2651.sbn"     "nodala2651.sbx"     "nodala2651.shp"    
    ##  [7] "nodala2651.shp.xml" "nodala2651.shx"     "nodala2652.cpg"    
    ## [10] "nodala2652.dbf"     "nodala2652.prj"     "nodala2652.sbn"    
    ## [13] "nodala2652.sbx"     "nodala2652.shp"     "nodala2652.shp.xml"
    ## [16] "nodala2652.shx"     "nodala2653.cpg"     "nodala2653.dbf"    
    ## [19] "nodala2653.prj"     "nodala2653.sbn"     "nodala2653.sbx"    
    ## [22] "nodala2653.shp"     "nodala2653.shp.xml" "nodala2653.shx"    
    ## [25] "nodala2654.cpg"     "nodala2654.dbf"     "nodala2654.prj"    
    ## [28] "nodala2654.sbn"     "nodala2654.sbx"     "nodala2654.shp"    
    ## [31] "nodala2654.shp.xml" "nodala2654.shx"     "nodala2655.cpg"    
    ## [34] "nodala2655.dbf"     "nodala2655.prj"     "nodala2655.sbn"    
    ## [37] "nodala2655.sbx"     "nodala2655.shp"     "nodala2655.shp.xml"
    ## [40] "nodala2655.shx"

Cik daudz vietas aizņem ESRI shapefile, GeoPackage un geoparquet? Sākumā
ielasu 2651. nodaļas datus

``` r
file_size("../centra_mezi/nodala2651.shp")
```

    ## 25.8M

Izveidoju GeoPackage failu. Cik daudz vietas tas aizņem?

``` r
nod2651 <- st_read("../centra_mezi/nodala2651.shp")
```

    ## Reading layer `nodala2651' from data source 
    ##   `C:\Users\ruben\Documents\HiQBioDiv_macibas_fork\centra_mezi\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM

``` r
st_write(nod2651, "C:/Users/ruben/Documents/HiQBioDiv_macibas_fork/centra_mezi/nodala2651.gpkg")
```

    ## Writing layer `nodala2651' to data source 
    ##   `C:/Users/ruben/Documents/HiQBioDiv_macibas_fork/centra_mezi/nodala2651.gpkg' using driver `GPKG'
    ## Writing 91369 features with 70 fields and geometry type Multi Polygon.

``` r
file_size("../centra_mezi/nodala2651.gpkg")
```

    ## 55.1M

Cik daudz vietas aizņem geoparquet?

``` r
st_write_parquet(nod2651, "C:/Users/ruben/Documents/HiQBioDiv_macibas_fork/centra_mezi/nodala2651.parquet")
file_size("../centra_mezi/nodala2651.parquet")
```

    ## 21.1M
