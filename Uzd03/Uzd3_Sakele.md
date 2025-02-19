Uzd3_Sakele
================
Vita
2025-01-27

\#1.

``` r
#Vajadzīgās pakotnes
if (!require("pacman")) install.packages("pacman")
pacman::p_load(sf, httr, tidyverse, ows4R, terra, RCurl, archive, sfarrow, raster)
```

\##Mežniecības dati

``` r
if (!dir.exists("data/download")) {
  dir.create("data/download", recursive=TRUE)
  }

url= "https://data.gov.lv/dati/lv/dataset/40014c0a-90f5-42be-afb2-fe3c4b8adf92/resource/392dfb67-eeeb-43c2-b082-35f9cf986128/download/centra.7z"

destfile = "data/download/centra.7z"

download.file(url=url, destfile = destfile, mode='wb')



if (!dir.exists("data/VMD")) {
  dir.create("data/VMD")
}
archive_extract(destfile, dir = "data/VMD")
```

    ## ⠙ 1 extracted | 629 MB (283 MB/s) | 2.2s⠹ 1 extracted | 641 MB (269 MB/s) |
    ## 2.4s⠸ 1 extracted | 676 MB (262 MB/s) | 2.6s⠼ 1 extracted | 734 MB (263 MB/s) |
    ## 2.8s⠴ 1 extracted | 777 MB (259 MB/s) | 3s ⠦ 3 extracted | 786 MB (245 MB/s) |
    ## 3.2s⠧ 5 extracted | 789 MB (230 MB/s) | 3.4s⠇ 5 extracted | 793 MB (219 MB/s) |
    ## 3.6s⠏ 5 extracted | 797 MB (208 MB/s) | 3.8s⠋ 5 extracted | 803 MB (199 MB/s) |
    ## 4s ⠙ 5 extracted | 808 MB (190 MB/s) | 4.2s⠹ 5 extracted | 811 MB (183 MB/s) |
    ## 4.4s⠸ 9 extracted | 826 MB (177 MB/s) | 4.7s⠼ 9 extracted | 879 MB (181 MB/s) |
    ## 4.9s⠴ 9 extracted | 917 MB (181 MB/s) | 5.1s⠦ 9 extracted | 948 MB (179 MB/s) |
    ## 5.3s⠧ 9 extracted | 966 MB (176 MB/s) | 5.5s⠇ 9 extracted | 995 MB (174 MB/s) |
    ## 5.7s⠏ 9 extracted | 1.0 GB (172 MB/s) | 5.9s⠋ 9 extracted | 1.1 GB (172 MB/s) |
    ## 6.1s⠙ 9 extracted | 1.1 GB (170 MB/s) | 6.3s⠹ 9 extracted | 1.1 GB (169 MB/s) |
    ## 6.5s⠸ 9 extracted | 1.1 GB (167 MB/s) | 6.7s⠼ 9 extracted | 1.2 GB (166 MB/s) |
    ## 6.9s⠴ 9 extracted | 1.2 GB (164 MB/s) | 7.2s⠦ 9 extracted | 1.2 GB (164 MB/s) |
    ## 7.4s⠧ 9 extracted | 1.2 GB (164 MB/s) | 7.6s⠇ 9 extracted | 1.3 GB (164 MB/s) |
    ## 7.8s⠏ 9 extracted | 1.3 GB (168 MB/s) | 8s ⠋ 9 extracted | 1.4 GB (169 MB/s) |
    ## 8.2s⠙ 9 extracted | 1.4 GB (172 MB/s) | 8.4s⠹ 10 extracted | 1.5 GB (173 MB/s)
    ## | 8.6s⠸ 13 extracted | 1.5 GB (170 MB/s) | 8.8s⠼ 13 extracted | 1.5 GB (166
    ## MB/s) | 9s ⠴ 13 extracted | 1.5 GB (163 MB/s) | 9.2s⠦ 13 extracted | 1.5 GB
    ## (160 MB/s) | 9.4s⠧ 13 extracted | 1.5 GB (157 MB/s) | 9.6s⠇ 13 extracted | 1.5
    ## GB (154 MB/s) | 9.8s⠏ 15 extracted | 1.5 GB (151 MB/s) | 10s ⠋ 17 extracted |
    ## 1.5 GB (151 MB/s) | 10.3s⠙ 17 extracted | 1.6 GB (151 MB/s) | 10.5s⠹ 17
    ## extracted | 1.6 GB (151 MB/s) | 10.7s⠸ 17 extracted | 1.7 GB (152 MB/s) |
    ## 10.9s⠼ 17 extracted | 1.7 GB (152 MB/s) | 11.1s⠴ 17 extracted | 1.7 GB (152
    ## MB/s) | 11.3s⠦ 17 extracted | 1.8 GB (154 MB/s) | 11.5s⠧ 17 extracted | 1.8 GB
    ## (154 MB/s) | 11.7s⠇ 17 extracted | 1.8 GB (154 MB/s) | 11.9s⠏ 17 extracted |
    ## 1.9 GB (154 MB/s) | 12.1s⠋ 17 extracted | 1.9 GB (154 MB/s) | 12.3s⠙ 17
    ## extracted | 1.9 GB (154 MB/s) | 12.5s⠹ 17 extracted | 2.0 GB (156 MB/s) |
    ## 12.7s⠸ 17 extracted | 2.1 GB (160 MB/s) | 12.9s⠼ 17 extracted | 2.1 GB (163
    ## MB/s) | 13.1s⠴ 17 extracted | 2.2 GB (167 MB/s) | 13.3s⠦ 17 extracted | 2.3 GB
    ## (172 MB/s) | 13.6s⠧ 17 extracted | 2.4 GB (174 MB/s) | 13.8s⠇ 17 extracted |
    ## 2.4 GB (175 MB/s) | 14s ⠏ 18 extracted | 2.5 GB (175 MB/s) | 14.2s⠋ 21
    ## extracted | 2.5 GB (173 MB/s) | 14.4s⠙ 21 extracted | 2.5 GB (171 MB/s) |
    ## 14.6s⠹ 21 extracted | 2.5 GB (169 MB/s) | 14.8s⠸ 21 extracted | 2.5 GB (167
    ## MB/s) | 15s ⠼ 21 extracted | 2.5 GB (165 MB/s) | 15.2s⠴ 21 extracted | 2.5 GB
    ## (163 MB/s) | 15.4s⠦ 21 extracted | 2.5 GB (161 MB/s) | 15.6s⠧ 21 extracted |
    ## 2.5 GB (159 MB/s) | 15.8s⠇ 21 extracted | 2.5 GB (157 MB/s) | 16.1s⠏ 21
    ## extracted | 2.5 GB (155 MB/s) | 16.3s⠋ 25 extracted | 2.5 GB (153 MB/s) |
    ## 16.5s⠙ 25 extracted | 2.6 GB (153 MB/s) | 16.7s⠹ 25 extracted | 2.6 GB (154
    ## MB/s) | 16.9s⠸ 25 extracted | 2.7 GB (155 MB/s) | 17.1s⠼ 25 extracted | 2.7 GB
    ## (155 MB/s) | 17.3s⠴ 25 extracted | 2.7 GB (155 MB/s) | 17.5s⠦ 25 extracted |
    ## 2.7 GB (154 MB/s) | 17.7s⠧ 25 extracted | 2.8 GB (154 MB/s) | 17.9s⠇ 25
    ## extracted | 2.8 GB (153 MB/s) | 18.1s⠏ 25 extracted | 2.8 GB (154 MB/s) |
    ## 18.3s⠋ 25 extracted | 2.9 GB (154 MB/s) | 18.5s⠙ 25 extracted | 2.9 GB (154
    ## MB/s) | 18.7s⠹ 25 extracted | 2.9 GB (153 MB/s) | 18.9s⠸ 25 extracted | 2.9 GB
    ## (153 MB/s) | 19.2s⠼ 25 extracted | 3.0 GB (154 MB/s) | 19.4s⠴ 25 extracted |
    ## 3.0 GB (153 MB/s) | 19.6s⠦ 25 extracted | 3.0 GB (152 MB/s) | 19.8s⠧ 25
    ## extracted | 3.0 GB (151 MB/s) | 20s ⠇ 25 extracted | 3.0 GB (151 MB/s) | 20.2s⠏
    ## 25 extracted | 3.1 GB (151 MB/s) | 20.4s⠋ 25 extracted | 3.1 GB (151 MB/s) |
    ## 20.6s⠙ 25 extracted | 3.2 GB (153 MB/s) | 20.8s⠹ 25 extracted | 3.2 GB (154
    ## MB/s) | 21s ⠸ 25 extracted | 3.3 GB (156 MB/s) | 21.2s⠼ 25 extracted | 3.3 GB
    ## (156 MB/s) | 21.4s⠴ 25 extracted | 3.4 GB (156 MB/s) | 21.6s⠦ 25 extracted |
    ## 3.4 GB (157 MB/s) | 21.8s⠧ 25 extracted | 3.5 GB (158 MB/s) | 22.1s⠇ 27
    ## extracted | 3.5 GB (157 MB/s) | 22.3s⠏ 29 extracted | 3.5 GB (156 MB/s) |
    ## 22.5s⠋ 29 extracted | 3.5 GB (155 MB/s) | 22.7s⠙ 29 extracted | 3.5 GB (154
    ## MB/s) | 22.9s⠹ 29 extracted | 3.5 GB (153 MB/s) | 23.1s⠸ 29 extracted | 3.5 GB
    ## (151 MB/s) | 23.3s⠼ 29 extracted | 3.5 GB (150 MB/s) | 23.5s⠴ 29 extracted |
    ## 3.5 GB (149 MB/s) | 23.7s⠦ 29 extracted | 3.5 GB (148 MB/s) | 23.9s⠧ 31
    ## extracted | 3.5 GB (147 MB/s) | 24.1s⠇ 33 extracted | 3.6 GB (147 MB/s) |
    ## 24.3s⠏ 33 extracted | 3.6 GB (146 MB/s) | 24.5s⠋ 33 extracted | 3.6 GB (146
    ## MB/s) | 24.7s⠙ 33 extracted | 3.6 GB (145 MB/s) | 25s ⠹ 33 extracted | 3.6 GB
    ## (145 MB/s) | 25.2s⠸ 33 extracted | 3.7 GB (145 MB/s) | 25.4s⠼ 33 extracted |
    ## 3.7 GB (145 MB/s) | 25.6s⠴ 33 extracted | 3.7 GB (145 MB/s) | 25.8s⠦ 33
    ## extracted | 3.8 GB (145 MB/s) | 26s ⠧ 33 extracted | 3.8 GB (145 MB/s) | 26.2s⠇
    ## 33 extracted | 3.8 GB (145 MB/s) | 26.4s⠏ 33 extracted | 3.9 GB (146 MB/s) |
    ## 26.6s⠋ 33 extracted | 3.9 GB (146 MB/s) | 26.8s⠙ 33 extracted | 3.9 GB (146
    ## MB/s) | 27s ⠹ 33 extracted | 4.0 GB (146 MB/s) | 27.2s⠸ 33 extracted | 4.0 GB
    ## (145 MB/s) | 27.4s⠼ 33 extracted | 4.0 GB (145 MB/s) | 27.6s⠴ 33 extracted |
    ## 4.0 GB (146 MB/s) | 27.8s⠦ 33 extracted | 4.1 GB (146 MB/s) | 28s ⠧ 33
    ## extracted | 4.1 GB (146 MB/s) | 28.2s⠇ 33 extracted | 4.2 GB (146 MB/s) |
    ## 28.4s⠏ 33 extracted | 4.2 GB (146 MB/s) | 28.7s⠋ 33 extracted | 4.2 GB (147
    ## MB/s) | 28.9s⠙ 33 extracted | 4.3 GB (147 MB/s) | 29.1s⠹ 33 extracted | 4.3 GB
    ## (147 MB/s) | 29.3s⠸ 33 extracted | 4.3 GB (147 MB/s) | 29.5s⠼ 33 extracted |
    ## 4.4 GB (148 MB/s) | 29.7s⠴ 33 extracted | 4.4 GB (148 MB/s) | 29.9s⠦ 33
    ## extracted | 4.5 GB (148 MB/s) | 30.1s⠧ 35 extracted | 4.5 GB (148 MB/s) |
    ## 30.3s⠇ 37 extracted | 4.5 GB (147 MB/s) | 30.5s⠏ 37 extracted | 4.5 GB (146
    ## MB/s) | 30.7s⠋ 37 extracted | 4.5 GB (145 MB/s) | 30.9s⠙ 37 extracted | 4.5 GB
    ## (144 MB/s) | 31.1s⠹ 37 extracted | 4.5 GB (143 MB/s) | 31.3s⠸ 37 extracted |
    ## 4.5 GB (143 MB/s) | 31.5s⠼ 37 extracted | 4.5 GB (142 MB/s) | 31.8s⠴ 37
    ## extracted | 4.5 GB (141 MB/s) | 32s ⠦ 37 extracted | 4.5 GB (140 MB/s) | 32.2s⠧
    ## 37 extracted | 4.5 GB (139 MB/s) | 32.4s⠇ 37 extracted | 4.5 GB (138 MB/s) |
    ## 32.6s⠏ 37 extracted | 4.5 GB (138 MB/s) | 32.8s⠋ 37 extracted | 4.5 GB (137
    ## MB/s) | 33s ⠙ 37 extracted | 4.5 GB (136 MB/s) | 33.2s⠹ 37 extracted | 4.5 GB
    ## (135 MB/s) | 33.4s⠸ 39 extracted | 4.5 GB (134 MB/s) | 33.6s

``` r
if (!dir.exists("data/VMD/merged")) {
  dir.create("data/VMD/merged")
}
 
file_list <- list.files("data/VMD", pattern = "*shp$", full.names = TRUE)
shapefile_list <- lapply(file_list, read_sf) #ielasa visus failus
centra_shp <- do.call(rbind, shapefile_list) # apvieno vienā failā
st_write_parquet(centra_shp, "data/VMD/merged/centra.parquet") # ieraksta apvienoto failu, kā parquet uz diska

rm(centra_shp, shapefile_list) #iztīrīt atmiņu

centra_parq<- st_read_parquet("data/VMD/merged/centra.parquet")


# Pārbaudīt, vai visas ģeometrijas ir MULTIPOLYGON
if(all(st_geometry_type(centra_parq) == "MULTIPOLYGON")) {
  print("Visas ģeometrijas ir MULTIPOLYGON")
} else {
  print("Dažas ģeometrijas nav MULTIPOLYGON")
  }
```

    ## [1] "Visas ģeometrijas ir MULTIPOLYGON"

``` r
# Pārbaudīt, vai nav tukšu ģeometriju
if(any(st_is_empty(centra_parq))) {
  print("Slānī ir tukšas ģeometrijas")
  # Izdzēst tukšās ģeometrijas
  centra_parq <- centra_parq[!st_is_empty(centra_parq), ] 
}
```

    ## [1] "Slānī ir tukšas ģeometrijas"

``` r
# Pārbaudīt, vai nav kļūdainu ģeometriju
if(any(!st_is_valid(centra_parq))) {
  print("Slānī ir kļūdainas ģeometrijas")
  # Salabot un izdzēst kļūdainās ģeometrijas
  centra_parq <- st_make_valid(centra_parq)
  centra_parq <- centra_parq[st_is_valid(centra_parq), ] 
}
```

    ## [1] "Slānī ir kļūdainas ģeometrijas"

``` r
bbox_centra <- st_bbox(centra_parq)
bbox_query <- paste(bbox_centra["xmin"],bbox_centra["ymin"], 
                    bbox_centra["xmax"], bbox_centra["ymax"], "EPSG:3059", sep = ",")

rm(centra_parq)
```

\##LAD dati

``` r
wfs_lad<-"https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer"

url <- parse_url(wfs_lad)

url$query <- list(
  service = "WFS",
  #version = "2.0.0",
  request = "GetFeature",
  typename = "LAD:Lauki",
  #srsName = "EPSG:4326",
  srsName = "EPSG:3059", 
    bbox = bbox_query  
  )
request <- build_url(url)
LAD_data <- read_sf(request)


if (!dir.exists("data/LAD")) {
  dir.create("data/LAD")
}

st_write_parquet(LAD_data, "data/LAD/LAD.parquet")
LAD_parq <- st_read_parquet("data/LAD/LAD.parquet", quiet = TRUE)

rm(LAD_data)
```

\##References dati

``` r
if (!dir.exists("data/reference_raster")) {
  dir.create("data/reference_raster")
}

url= "https://zenodo.org/records/14497070/files/LV100m_10km.tif?download=1"
destfile = "data/reference_raster/LV10m_10km.tif"

download.file(url=url, destfile = destfile, mode='wb')

raster_10m_10km = raster(destfile)

rm(url, destfile)
```

\#2.

``` r
#samazināt references rastru
raster_10m_10km_adjusted<-crop(raster_10m_10km, LAD_parq)


if(!require("fasterize")) install.packages("fasterize")
```

    ## Loading required package: fasterize

    ## 
    ## Attaching package: 'fasterize'

    ## The following object is masked from 'package:graphics':
    ## 
    ##     plot

    ## The following object is masked from 'package:base':
    ## 
    ##     plot

``` r
#viss, kas nav LAD, būs 0
LAD_raster_10m <- fasterize(LAD_parq, raster_10m_10km_adjusted, background = 0)

#Ārpus LV būs NA
LAD_raster_10m_masked <- mask(LAD_raster_10m, raster_10m_10km_adjusted)

plot(LAD_raster_10m_masked)
```

![](Uzd3_Sakele_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

``` r
writeRaster(LAD_raster_10m_masked, "data/LAD_raster_10m.tif", format = "GTiff", overwrite = TRUE)
```

\#3. \##Reference 100m

``` r
if (!dir.exists("data/reference_raster")) {
  dir.create("data/reference_raster")
}

url= "https://zenodo.org/records/14497070/files/LV100m_10km.tif?download=1"
destfile = "data/reference_raster/LV100m_10km.tif"

download.file(url=url, destfile = destfile, mode='wb')

raster_100m_10km = raster(destfile)

rm(url, destfile)
```

## pārveidošanas ātrums

``` r
raster_100m_10km_adjusted<-crop(raster_100m_10km, LAD_parq)

if (!require("microbenchmark")) install.packages("microbenchmark")
```

    ## Loading required package: microbenchmark

``` r
#lauku īpatsvars 100x100m laukumos

speed<-microbenchmark(
  LAD_raster_100m_agr <- aggregate(LAD_raster_10m_masked, fact = 10, fun = 'mean'),
  LAD_raster_100m_resampled<-resample(LAD_raster_10m_masked, raster_100m_10km_adjusted,
                                    method="bilinear"), #nesapratu, kāpēc "average" neņēma
  times =5,
  unit="s"
)

print(speed) #aggregate() ir ātrāks
```

    ## Unit: seconds
    ##                                                                                                               expr
    ##                              LAD_raster_100m_agr <- aggregate(LAD_raster_10m_masked, fact = 10,      fun = "mean")
    ##  LAD_raster_100m_resampled <- resample(LAD_raster_10m_masked,      raster_100m_10km_adjusted, method = "bilinear")
    ##       min       lq     mean   median       uq      max neval cld
    ##  1.546002 1.563182 1.591143 1.604756 1.610563 1.631212     5  a 
    ##  3.643140 3.725293 4.088929 3.756187 4.162203 5.157823     5   b

``` r
plot(LAD_raster_100m_agr)
```

![](Uzd3_Sakele_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

``` r
plot(LAD_raster_100m_resampled)
```

![](Uzd3_Sakele_files/figure-gfm/unnamed-chunk-8-2.png)<!-- -->

``` r
rm(LAD_raster_100m_resampled)
```

\#4.

``` r
#lai būtu procenti
LAD_raster_100m_agr_prc<-as.integer(LAD_raster_100m_agr*100)

#Veselos skaitļu glabāšana

LAD_INT1S <- writeRaster(LAD_raster_100m_agr_prc,"data/LAD_INT1S.tif", datatype = "INT1S", overwrite = TRUE)
print(file.size("data/LAD_INT1S.tif"))
```

    ## [1] 5479

``` r
LAD_INT1U <- writeRaster(LAD_raster_100m_agr_prc,"data/LAD_INT1U.tif", datatype = "INT1U", overwrite = TRUE)
print(file.size("data/LAD_INT1U.tif"))
```

    ## [1] 5469

``` r
#Dažādi rastri

LAD_raster_100m_agr_bin <- aggregate(LAD_raster_10m_masked, fact = 10, fun = 'max')


writeRaster(LAD_raster_10m_masked, "data/LAD_raster_10m_masked.tif", format = "GTiff")
print(file.size('data/LAD_raster_10m_masked.tif'))
```

    ## [1] 335770

``` r
writeRaster(LAD_raster_100m_agr_bin, "data/LAD_raster_100m_agr_bin.tif", format = "GTiff")
print(file.size('data/LAD_raster_100m_agr_bin.tif'))
```

    ## [1] 6922

Izvēlētie datu tipi minimāli ietekmē faila izmēru. Rastra izšķirtspēja
starp 10 un 100m faila izmērā veido atšķirību gandrīz 50 reizes.

``` r
unlink("data", recursive = TRUE)

rm(list = ls(all.names = TRUE)) #will clear all objects includes hidden objects.
gc()
```

    ##           used  (Mb) gc trigger  (Mb)  max used   (Mb)
    ## Ncells 4888950 261.1   14025405 749.1  21914695 1170.4
    ## Vcells 7845692  59.9   97730776 745.7 237889828 1815.0
