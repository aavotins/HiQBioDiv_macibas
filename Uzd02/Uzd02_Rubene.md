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
```

Centra virsmežniecības faila lejupielāde

``` r
url <- "https://data.gov.lv/dati/lv/dataset/40014c0a-90f5-42be-afb2-fe3c4b8adf92/resource/392dfb67-eeeb-43c2-b082-35f9cf986128/download/centra.7z"
dest <- "C:/Users/ruben/Documents/HiQBioDiv_macibas_fork/centra_mezi.7z"

curl_download(url, destfile = dest)
file.exists(dest)
```

    ## [1] TRUE

Atarhivēšana

``` r
archive_extract(dest, dir = "C:/Users/ruben/Documents/HiQBioDiv_macibas_fork/centra_mezi")
```

Failu apskate

``` r
list.files("./centra_mezi")
```

    ## character(0)
