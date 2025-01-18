Trešais uzdevums: rastra dati, rasterizēšana un kodējumi
================

Visu vajadzīgo funkciju instalēšana un ielāde

``` r
if(!require(archive)){
    install.packages("archive")
    library(archive)}
if(!require(curl)){
    install.packages("curl")
    library(curl)}
if(!require(sf)){
    install.packages("sf")
    library(sf)}
if(!require(sfarrow)){ install.packages("sfarrow")
library(sfarrow) }
if(!require(httr)){
    install.packages("httr")
    library(httr)}
if(!require(ows4R)){
    install.packages("ows4R")
    library(ows4R)}
if(!require(sfarrow)){
    install.packages("sfarrow")
    library(sfarrow)}
if(!require(terra)){
    install.packages("terra")
    library(terra)}
if(!require(fasterize)){
    install.packages("fasterize")
    library(fasterize)}
if(!require(dplyr)){
    install.packages("dplyr")
    library(dplyr)}
if(!require(microbenchmark)){
    install.packages("microbenchmark")
    library(microbenchmark)}
```

\#1. uzdevums 1.1. References režģu lejuplāde - vektors un rastrs

``` r
#vektors
vektoru_url <- "https://zenodo.org/api/records/14277114/files-archive"
vektoru <- "C:/Users/vinet/Documents/VPP/3uzd/vektoru.zip"
curl_download(vektoru_url, destfile = vektoru)

vektoru_vieta <- "C:/Users/vinet/Documents/VPP/3uzd/vektoru"
archive_extract(vektoru, dir =vektoru_vieta)
```

    ## ⠙ 9 extracted | 503 MB (218 MB/s) | 2.3s⠹ 9 extracted | 520 MB (178 MB/s) |
    ## 2.9s⠸ 9 extracted | 565 MB (181 MB/s) | 3.1s⠼ 9 extracted | 604 MB (181 MB/s) |
    ## 3.3s⠴ 9 extracted | 650 MB (183 MB/s) | 3.5s⠦ 9 extracted | 696 MB (185 MB/s) |
    ## 3.8s⠧ 9 extracted | 736 MB (185 MB/s) | 4s ⠇ 9 extracted | 780 MB (187 MB/s) |
    ## 4.2s⠏ 9 extracted | 826 MB (188 MB/s) | 4.4s⠋ 9 extracted | 869 MB (189 MB/s) |
    ## 4.6s⠙ 9 extracted | 910 MB (189 MB/s) | 4.8s⠹ 9 extracted | 956 MB (190 MB/s) |
    ## 5s ⠸ 9 extracted | 990 MB (189 MB/s) | 5.2s⠼ 9 extracted | 1.0 GB (191 MB/s) |
    ## 5.4s⠴ 9 extracted | 1.1 GB (170 MB/s) | 6.4s⠦ 9 extracted | 1.1 GB (171 MB/s) |
    ## 6.5s⠧ 9 extracted | 1.1 GB (169 MB/s) | 6.7s⠇ 9 extracted | 1.2 GB (171 MB/s) |
    ## 6.9s⠏ 9 extracted | 1.2 GB (173 MB/s) | 7.1s⠋ 9 extracted | 1.3 GB (174 MB/s) |
    ## 7.3s⠙ 9 extracted | 1.3 GB (176 MB/s) | 7.5s⠹ 9 extracted | 1.4 GB (178 MB/s) |
    ## 7.7s⠸ 9 extracted | 1.4 GB (179 MB/s) | 8s ⠼ 9 extracted | 1.5 GB (181 MB/s) |
    ## 8.2s⠴ 9 extracted | 1.5 GB (182 MB/s) | 8.4s⠦ 9 extracted | 1.6 GB (183 MB/s) |
    ## 8.6s⠧ 9 extracted | 1.6 GB (184 MB/s) | 8.8s⠇ 9 extracted | 1.7 GB (186 MB/s) |
    ## 9s ⠏ 9 extracted | 1.7 GB (187 MB/s) | 9.2s⠋ 9 extracted | 1.8 GB (188 MB/s) |
    ## 9.4s⠙ 9 extracted | 1.8 GB (189 MB/s) | 9.6s⠹ 9 extracted | 1.9 GB (190 MB/s) |
    ## 9.8s⠸ 9 extracted | 1.9 GB (191 MB/s) | 10s ⠼ 9 extracted | 2.0 GB (193 MB/s) |
    ## 10.2s⠴ 9 extracted | 2.0 GB (193 MB/s) | 10.4s⠦ 9 extracted | 2.1 GB (194 MB/s)
    ## | 10.7s⠧ 9 extracted | 2.1 GB (194 MB/s) | 10.9s⠇ 9 extracted | 2.2 GB (195
    ## MB/s) | 11.1s⠏ 9 extracted | 2.2 GB (195 MB/s) | 11.3s⠋ 9 extracted | 2.3 GB
    ## (196 MB/s) | 11.5s⠙ 9 extracted | 2.3 GB (191 MB/s) | 11.8s⠹ 9 extracted | 2.3
    ## GB (191 MB/s) | 11.9s⠸ 9 extracted | 2.3 GB (192 MB/s) | 12.1s⠼ 9 extracted |
    ## 2.4 GB (192 MB/s) | 12.3s⠴ 9 extracted | 2.4 GB (192 MB/s) | 12.5s⠦ 9 extracted
    ## | 2.5 GB (193 MB/s) | 12.8s⠧ 9 extracted | 2.5 GB (193 MB/s) | 13s ⠇ 9
    ## extracted | 2.6 GB (194 MB/s) | 13.2s⠏ 9 extracted | 2.6 GB (195 MB/s) | 13.4s⠋
    ## 9 extracted | 2.7 GB (195 MB/s) | 13.6s⠙ 9 extracted | 2.7 GB (195 MB/s) |
    ## 13.8s⠹ 9 extracted | 2.7 GB (196 MB/s) | 14s ⠸ 9 extracted | 2.8 GB (197 MB/s)
    ## | 14.2s⠼ 9 extracted | 2.8 GB (197 MB/s) | 14.4s⠴ 9 extracted | 2.9 GB (198
    ## MB/s) | 14.6s⠦ 9 extracted | 2.9 GB (198 MB/s) | 14.9s⠧ 9 extracted | 3.0 GB
    ## (199 MB/s) | 15.1s⠇ 9 extracted | 3.0 GB (199 MB/s) | 15.3s⠏ 9 extracted | 3.1
    ## GB (200 MB/s) | 15.5s⠋ 9 extracted | 3.1 GB (201 MB/s) | 15.7s⠙ 9 extracted |
    ## 3.2 GB (201 MB/s) | 15.9s⠹ 9 extracted | 3.2 GB (201 MB/s) | 16.1s⠸ 9 extracted
    ## | 3.3 GB (202 MB/s) | 16.3s⠼ 9 extracted | 3.3 GB (203 MB/s) | 16.5s⠴ 9
    ## extracted | 3.4 GB (202 MB/s) | 16.7s⠦ 9 extracted | 3.4 GB (202 MB/s) | 16.9s⠧
    ## 9 extracted | 3.5 GB (202 MB/s) | 17.2s

``` r
#rastrs
rastru_url <- "https://zenodo.org/api/records/14497070/files-archive"
rastrs <- "C:/Users/vinet/Documents/VPP/3uzd/rastrs.zip"
curl_download(rastru_url, destfile = rastrs)

rastrs_vieta <- "C:/Users/vinet/Documents/VPP/3uzd/rastrs"
archive_extract(rastrs, dir =rastrs_vieta)
```

1.2. Mežu valsts reģistra centra datu geoparquet ielasīšana

``` r
centra_gpq_vieta <-"C:/Users/vinet/Documents/VPP/2uzd/centra.parquet_apv"
centra_gpq <-st_read_parquet(centra_gpq_vieta, quiet = TRUE)
```

1.3. WFS slāņu informācijas iegūšana

``` r
wfs_link1 <- "https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer"
wfs_url1 <- parse_url(wfs_link1)
wfs_url1$query <- list(service = "wfs",
                  request = "GetCapabilities")
request <- build_url(wfs_url1)
request
```

    ## [1] "https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer?service=wfs&request=GetCapabilities"

``` r
#Pārbaudu koordinātu sistēmu, kādu output faila tipu un serviceversion atbalsta šis WFS. Pārliecinājos, ka atbalsta gml, kas ir standarts. Izmantojamā koordinātu sistēma šim WFS ir wgs

bwk_client <- WFSClient$new(wfs_link1, 
                            serviceVersion = "2.0.0")
bwk_client$getFeatureTypes(pretty = TRUE)
```

    ##        name title
    ## 1 LAD:Lauki Lauki

1.4. Geoparquet failā esošo ģeometrijas datu robežu noteikšana un robežu
pārveidošana uz WGS84 sistēmu. Tā kā mežu valsts reģistra centra dati
bija LKS92 sistēmā, tad tos vajadzēja pārveidot, lai pareizi norādītu
robežas tālākajās darbībās.

``` r
centra_robezas <- st_bbox(centra_gpq)
centra_robezas_wgs <- st_transform(centra_robezas, crs = 4326)
print(centra_robezas_wgs)
```

    ##     xmin     ymin     xmax     ymax 
    ## 22.42279 56.24303 25.57287 57.41234

1.5. WFS datu lejupielāde un ielasīšana Tā kā nav sagaidāms, ka tuvākajā
laikā tiks aktīvi rediģēti WFS pieejamie dati, tad WFS vispirms
lejuplādēju uz datora. Tas aizņem vairāk atmiņu, taču nodrošina ātrāku
datu apstrādi turpmāko darbību veikšanā.

``` r
wfs_link2 <- "https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer"
wfs_url2 <- parse_url(wfs_link2)
wfs_url2$query <- list(service = "WFS",
                  
                  request = "GetFeature",
                  typename = "LAD:Lauki",
                  bbox = "22.42279, 56.24303, 25.57287, 57.41234")
request2 <- build_url(wfs_url2)


centra_gml_vieta <- ("C:/Users/vinet/Documents/VPP/3uzd/centra.gml")
invisible(GET(url = request2, 
    write_disk(centra_gml_vieta, overwrite = TRUE)))
centra_gml <- read_sf(centra_gml_vieta, quiet = TRUE)
plot(centra_gml)
```

![](Uzd03_Verpeja_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

``` r
rm(centra_gml_vieta, vektoru_url, vektoru, vektoru_vieta, rastrs, rastrs_vieta, rastru_url, centra_gpq_vieta, centra_gpq, wfs_link1, wfs_url1, wfs_link1, wfs_url1, request, request2, bwk_client, centra_robezas, centra_robezas_wgs)
#Noņemu liekos objektus, lai dators vieglāk tiktu galā ar nākošajām darbībām
```

\#2. uzdevums

2.1. Gml faila pārveidošana atpakaļ par geoparquet failu, lai ātrāk
veiktu nākošās darbības

``` r
LAD_centra_vieta <- "C:/Users/vinet/Documents/VPP/3uzd/LAD_centra.parquet"
st_write_parquet(centra_gml, LAD_centra_vieta)
LAD_centra <-st_read_parquet(LAD_centra_vieta, quiet = TRUE)
```

2.2. Rasterizācija

``` r
rastrs10m <- rast("C:/Users/vinet/Documents/VPP/3uzd/rastrs/LV10m_10km.tif")

st_crs(LAD_centra)==st_crs(rastrs10m)
```

    ## [1] TRUE

``` r
#Pārliecinos, ka abiem failiem ir vienāda koordinātu sistēma izmantota

mazs_rastrs10m <-terra::crop(rastrs10m,LAD_centra)
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
r_10m <-raster::raster(mazs_rastrs10m)
#Apgriežu rastru, lai mans dators spētu ar to darboties un pārvēršu par korekto rastra faila formātu

LAD_centra$yes=1
LAD_yes <-LAD_centra %>% 
  dplyr::select(yes)
centra10m <- fasterize(LAD_yes,r_10m, field="yes", background=0)
centra10m <- mask(centra10m, r_10m)
plot(centra10m)
```

![](Uzd03_Verpeja_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

``` r
centra10m_vieta <- "C:/Users/vinet/Documents/VPP/3uzd/centra10m.tif"
writeRaster(centra10m, filename=centra10m_vieta, format = "GTiff", overwrite = TRUE)

rm(LAD_centra_vieta, LAD_centra, rastrs10m, mazs_rastrs10m, r_10m, centra10m, centra10m_vieta)
#Noņemu liekos objektus, lai dators vieglāk tiktu galā ar nākošajām darbībām
```

\#3.uzdevums

3.1. Rastra 100m režģa sagatavošana

``` r
rastrs100m <- rast("C:/Users/vinet/Documents/VPP/3uzd/rastrs/LV100m_10km.tif")
centra10m <- rast("C:/Users/vinet/Documents/VPP/3uzd/centra10m.tif")
st_crs(rastrs100m)==st_crs(centra10m)
```

    ## [1] FALSE

``` r
st_crs(rastrs100m)
```

    ## Coordinate Reference System:
    ##   User input: LKS-92 / Latvia TM 
    ##   wkt:
    ## PROJCRS["LKS-92 / Latvia TM",
    ##     BASEGEOGCRS["LKS-92",
    ##         DATUM["Latvian geodetic coordinate system 1992",
    ##             ELLIPSOID["GRS 1980",6378137,298.257222101,
    ##                 LENGTHUNIT["metre",1]]],
    ##         PRIMEM["Greenwich",0,
    ##             ANGLEUNIT["degree",0.0174532925199433]],
    ##         ID["EPSG",4661]],
    ##     CONVERSION["Latvian Transverse Mercator",
    ##         METHOD["Transverse Mercator",
    ##             ID["EPSG",9807]],
    ##         PARAMETER["Latitude of natural origin",0,
    ##             ANGLEUNIT["degree",0.0174532925199433],
    ##             ID["EPSG",8801]],
    ##         PARAMETER["Longitude of natural origin",24,
    ##             ANGLEUNIT["degree",0.0174532925199433],
    ##             ID["EPSG",8802]],
    ##         PARAMETER["Scale factor at natural origin",0.9996,
    ##             SCALEUNIT["unity",1],
    ##             ID["EPSG",8805]],
    ##         PARAMETER["False easting",500000,
    ##             LENGTHUNIT["metre",1],
    ##             ID["EPSG",8806]],
    ##         PARAMETER["False northing",-6000000,
    ##             LENGTHUNIT["metre",1],
    ##             ID["EPSG",8807]]],
    ##     CS[Cartesian,2],
    ##         AXIS["northing (X)",north,
    ##             ORDER[1],
    ##             LENGTHUNIT["metre",1]],
    ##         AXIS["easting (Y)",east,
    ##             ORDER[2],
    ##             LENGTHUNIT["metre",1]],
    ##     USAGE[
    ##         SCOPE["Engineering survey, topographic mapping."],
    ##         AREA["Latvia - onshore and offshore."],
    ##         BBOX[55.67,19.06,58.09,28.24]],
    ##     ID["EPSG",3059]]

``` r
st_crs(centra10m)
```

    ## Coordinate Reference System:
    ##   User input: PROJCRS["unknown",
    ##     BASEGEOGCRS["unknown",
    ##         DATUM["Unknown based on GRS 1980 ellipsoid using towgs84=0,0,0,0,0,0,0",
    ##             ELLIPSOID["GRS 1980",6378137,298.257222101004,
    ##                 LENGTHUNIT["metre",1],
    ##                 ID["EPSG",7019]]],
    ##         PRIMEM["Greenwich",0,
    ##             ANGLEUNIT["degree",0.0174532925199433,
    ##                 ID["EPSG",9122]]]],
    ##     CONVERSION["Transverse Mercator",
    ##         METHOD["Transverse Mercator",
    ##             ID["EPSG",9807]],
    ##         PARAMETER["Latitude of natural origin",0,
    ##             ANGLEUNIT["degree",0.0174532925199433],
    ##             ID["EPSG",8801]],
    ##         PARAMETER["Longitude of natural origin",24,
    ##             ANGLEUNIT["degree",0.0174532925199433],
    ##             ID["EPSG",8802]],
    ##         PARAMETER["Scale factor at natural origin",0.9996,
    ##             SCALEUNIT["unity",1],
    ##             ID["EPSG",8805]],
    ##         PARAMETER["False easting",500000,
    ##             LENGTHUNIT["metre",1],
    ##             ID["EPSG",8806]],
    ##         PARAMETER["False northing",-6000000,
    ##             LENGTHUNIT["metre",1],
    ##             ID["EPSG",8807]]],
    ##     CS[Cartesian,2],
    ##         AXIS["easting",east,
    ##             ORDER[1],
    ##             LENGTHUNIT["metre",1,
    ##                 ID["EPSG",9001]]],
    ##         AXIS["northing",north,
    ##             ORDER[2],
    ##             LENGTHUNIT["metre",1,
    ##                 ID["EPSG",9001]]]] 
    ##   wkt:
    ## PROJCRS["unknown",
    ##     BASEGEOGCRS["unknown",
    ##         DATUM["Unknown based on GRS 1980 ellipsoid using towgs84=0,0,0,0,0,0,0",
    ##             ELLIPSOID["GRS 1980",6378137,298.257222101004,
    ##                 LENGTHUNIT["metre",1],
    ##                 ID["EPSG",7019]]],
    ##         PRIMEM["Greenwich",0,
    ##             ANGLEUNIT["degree",0.0174532925199433,
    ##                 ID["EPSG",9122]]]],
    ##     CONVERSION["Transverse Mercator",
    ##         METHOD["Transverse Mercator",
    ##             ID["EPSG",9807]],
    ##         PARAMETER["Latitude of natural origin",0,
    ##             ANGLEUNIT["degree",0.0174532925199433],
    ##             ID["EPSG",8801]],
    ##         PARAMETER["Longitude of natural origin",24,
    ##             ANGLEUNIT["degree",0.0174532925199433],
    ##             ID["EPSG",8802]],
    ##         PARAMETER["Scale factor at natural origin",0.9996,
    ##             SCALEUNIT["unity",1],
    ##             ID["EPSG",8805]],
    ##         PARAMETER["False easting",500000,
    ##             LENGTHUNIT["metre",1],
    ##             ID["EPSG",8806]],
    ##         PARAMETER["False northing",-6000000,
    ##             LENGTHUNIT["metre",1],
    ##             ID["EPSG",8807]]],
    ##     CS[Cartesian,2],
    ##         AXIS["easting",east,
    ##             ORDER[1],
    ##             LENGTHUNIT["metre",1,
    ##                 ID["EPSG",9001]]],
    ##         AXIS["northing",north,
    ##             ORDER[2],
    ##             LENGTHUNIT["metre",1,
    ##                 ID["EPSG",9001]]]]

``` r
#Pārbaudu vai ir vienāda kooridnātu sistēma. Režģa rastram ir LKS92, taču centra datu rastram ir atšķirīga.
```

3.2.Jaunā rastra izveide 3 variantos un ātruma noteikšana katrai
funkcijai

Resample - pārnes datus no viena rastra uz otru, kuriem nesakrīt
izšķirtspēja vai pakotne, ar ko tā izveidota (raster un spatraster
faili). Aggregate - uztaisa jaunu rastru ar zemāku izšķirtspēju nekā
oriģinālajam failam. Jānorāda cik šūnas sapludināt kopā ar funkciju
fact. Project - nomaina koordinātu sistēmu SpatRaster failam. Cik
sapratu, ja kā references koordinātu sistēmu ieliek citu SpatRaster
failu, nevis norāda konkrētu koordinātu sistēmu, tad tas nomaina arī
izšķirtspēju (šajā gadījumā 10m -\> 100m).

Kā jauno šūnu vērtības iegūšanas metodi izvēlos “sum”, kas tad arī
norādīs LAD lauku īpatsvaru.

``` r
atrums_koord_maina <- microbenchmark( centra10m_lks <- project(centra10m,"EPSG:3059"),times = 3)
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

``` r
atruma_salidz <- microbenchmark(
  centra100m_r <- resample(centra10m_lks, rastrs100m, method="sum"),
  centra100m_a <- aggregate(centra10m, fact=10, fun="sum"),
  centra100m_p <- project(centra10m, rastrs100m, method="sum"), times = 3)
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

``` r
summary(atrums_koord_maina)
```

    ##                                               expr     min       lq     mean
    ## 1 centra10m_lks <- project(centra10m, "EPSG:3059") 22.5938 22.64332 22.68762
    ##     median       uq      max neval
    ## 1 22.69283 22.73453 22.77623     3

``` r
summary(atruma_salidz)
```

    ##                                                                  expr
    ## 1 centra100m_r <- resample(centra10m_lks, rastrs100m, method = "sum")
    ## 2        centra100m_a <- aggregate(centra10m, fact = 10, fun = "sum")
    ## 3      centra100m_p <- project(centra10m, rastrs100m, method = "sum")
    ##          min         lq       mean     median         uq        max neval
    ## 1  91.029084  91.488205  92.670633  91.947326  93.491407  95.035489     3
    ## 2   1.785889   1.824387   1.915184   1.862884   1.979832   2.096779     3
    ## 3 162.551271 162.722300 163.965715 162.893328 164.672937 166.452545     3

``` r
plot(centra100m_a)
```

![](Uzd03_Verpeja_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

``` r
#pārbaudu kā izskatās
```

Visātrāk jauno rastru izdevās izveidot ar aggregate funkciju. Otra
ātrākā metode bija resample. Ja abi rastra faili ir ar vienādu
koordinātu sistēmu, tad ātrumu atspoguļo centra100m_r, taču, ja
atšķirīgas un vispirms jāveic koordinātu sistēmas maiņa, tad
centra100m_lks+centra100m_r. Funkcija project bija vislēnākā.

\#4. uzdevums

1)  Iepriekš uztaisītais centra100m_a jau ir aprēķinājis LAD lauku
    īpatsvaru. Tā kā tika pāriets no 10x10 uz 100x100 izšķirtsspēju, tad
    esošās vērtības jau būs tie paši procenti un jaunu failu taisīt nav
    vajadzība.

4.1. Jau 3.uzd uztaisītā faila saglabāšana uz datora

``` r
writeRaster(centra100m_a, "C:/Users/vinet/Documents/VPP/3uzd/centra100m_proc.tif", overwrite = TRUE)
```

4.2. Jauna rastra izveide 100m izšķirtspējā, kur tikai vērtības 1 vai 0
šūnās, un tā saglabāšana uz datora

``` r
centra100m_bin <- aggregate(centra10m, fact=10, fun="max")
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
plot(centra100m_bin)
```

![](Uzd03_Verpeja_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

``` r
#Pārbaudu kā izskatās, vai viss veiksmīgi izdevās

writeRaster(centra100m_bin, "C:/Users/vinet/Documents/VPP/3uzd/centra100m_bin.tif", overwrite = TRUE)
```

4.3.Kā mainās faila izmērs uz cietā diska, atkarībā no rastra šūnas
izmēra?

``` r
centra10m_info <- file.info("C:/Users/vinet/Documents/VPP/3uzd/centra10m.tif")
centra10m_izmers <-centra10m_info$size / (1024^2)
centra100m_info <- file.info("C:/Users/vinet/Documents/VPP/3uzd/centra100m_proc.tif")
centra100m_izmers <-centra100m_info$size / (1024^2)
cat("10x10m izšķirtspējas faila vieta MB:", centra10m_izmers, "\n")
```

    ## 10x10m izšķirtspējas faila vieta MB: 12.70663

``` r
cat("100x100m izšķirtspējas faila vieta MB:", centra100m_izmers)
```

    ## 100x100m izšķirtspējas faila vieta MB: 0.3929825

Fails ar lielākām šūnām (100x100m) aizņem daudz mazāk vietas datorā. Tas
diezgan loģiski, jo uz to pašu teritoriju ir mazāk šūnas vajadzīgas, ja
ir mazāka izškirtspēja.

4.4. Kā mainās faila izmērs uz cietā diska, atkarībā no tajā ietvertās
informācijas?

``` r
centra100m_bin_info <- file.info("C:/Users/vinet/Documents/VPP/3uzd/centra100m_bin.tif")
centra100m_bin_izmers <-centra100m_bin_info$size / (1024^2)

cat("Bināras informācijas faila vieta MB:", centra100m_bin_izmers, "\n")
```

    ## Bināras informācijas faila vieta MB: 0.3411713

``` r
cat("Procentuālas informācijas faila vieta MB:", centra100m_izmers)
```

    ## Procentuālas informācijas faila vieta MB: 0.3929825

Ja informācija tiek uzglabāta kā bināras vērtības, tad šāds fails aizņem
nedaudz mazāk vietas, nekā saglabājot informāciju izteiktu procentos.
Izmēra atšķirības gan ir ļoti minimālas - 5MB.

4.5. Kāds kodējums (joslas tips) aizņem vismazāko apjomu uz cietā diska,
saglabājot informāciju? Ir dažādi informācijas kodējumi. Tie parasti
sastāv no 5 rakstzīmēm, piemēram, LOG1S, INT1S, INT1U, INT2S, INT2U u.c.
Pirmie 3 burti norāda vērtības tipu, 4. rakstīme norāda kodēto baitu
skaitu un vai skaitļi ir tikai pozitīvi (U), vai var būt arī negatīvi
(S). Mana hipotēze būtu, ka ar lielāku bitu skaitu, fails aizņems vairāk
vietas. Tāpat līdzīgi varētu būt - ja ir kodētas gan negatīvas, gan
pozitīvas vērtības vai ja tiek pieņemti arī daļskaitļi, tad arī fails
aizņems vairāk vietas.

``` r
#Baitu skaita ietekmes pārbaude
writeRaster(centra10m, "C:/Users/vinet/Documents/VPP/3uzd/centra10_INT1U.tif", datatype="INT1U", overwrite=TRUE)
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
writeRaster(centra10m, "C:/Users/vinet/Documents/VPP/3uzd/centra10_INT2U.tif", datatype="INT2U", overwrite=TRUE)
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
centra10_INT1U_info <- file.info("C:/Users/vinet/Documents/VPP/3uzd/centra10_INT1U.tif")
centra10_INT1U_izmers <-centra10_INT1U_info$size / (1024^2)
centra10_INT2U_info <- file.info("C:/Users/vinet/Documents/VPP/3uzd/centra10_INT2U.tif")
centra10_INT2U_izmers <-centra10_INT2U_info$size / (1024^2)

cat("Ar INT1U kodējumu faila izmērs MB:", centra10_INT1U_izmers, "\n")
```

    ## Ar INT1U kodējumu faila izmērs MB: 4.058842

``` r
cat("Ar INT2U kodējumu faila izmērs MB MB:", centra10_INT2U_izmers)
```

    ## Ar INT2U kodējumu faila izmērs MB MB: 6.034531

Ar lielāku baitu skaitu tiešām fails aizņem vairāk vietu. Failu izmēra
atšķirības gan ir ļoti mazas (2MB), vismaz apskatītajā gadījumā.

``` r
#S vai U kodējuma ietekmes pārbaude
writeRaster(centra10m, "C:/Users/vinet/Documents/VPP/3uzd/centra10_INT1S.tif", datatype="INT1S", overwrite=TRUE)
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
centra10_INT1S_info <- file.info("C:/Users/vinet/Documents/VPP/3uzd/centra10_INT1S.tif")
centra10_INT1S_izmers <-centra10_INT1S_info$size / (1024^2)

cat("Ar INT1U kodējumu faila izmērs MB:", centra10_INT1U_izmers, "\n")
```

    ## Ar INT1U kodējumu faila izmērs MB: 4.058842

``` r
cat("Ar INT1S kodējumu faila izmērs MB MB:", centra10_INT1S_izmers)
```

    ## Ar INT1S kodējumu faila izmērs MB MB: 4.058852

Ka ir kodētas gan negatīvas, gan pozitīvas vērtības (S) fails aizņem
gandrīz identiski daudz vietas datorā nekā U, taču U aizņem minimāli
mazāk vietas.

``` r
#Veselu skaitļi vai decimālskaitļu kodējuma ietekmes pārbaude
writeRaster(centra10m, "C:/Users/vinet/Documents/VPP/3uzd/centra10_FLT4S.tif", datatype="FLT4S", overwrite=TRUE)
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
writeRaster(centra10m, "C:/Users/vinet/Documents/VPP/3uzd/centra10_INT4S.tif", datatype="INT4S", overwrite=TRUE)
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
centra10_FLT4S_info <- file.info("C:/Users/vinet/Documents/VPP/3uzd/centra10_FLT4S.tif")
centra10_FLT4S_izmers <-centra10_FLT4S_info$size / (1024^2)
centra10_INT4S_info <- file.info("C:/Users/vinet/Documents/VPP/3uzd/centra10_INT4S.tif")
centra10_INT4S_izmers <-centra10_INT4S_info$size / (1024^2)

cat("Ar FLT4S kodējumu faila izmērs MB:", centra10_FLT4S_izmers, "\n")
```

    ## Ar FLT4S kodējumu faila izmērs MB: 12.69079

``` r
cat("Ar INT4S kodējumu faila izmērs MB MB:", centra10_INT4S_izmers)
```

    ## Ar INT4S kodējumu faila izmērs MB MB: 12.59775

Saglabājot failu INT kodējumā (tikai veseli skaitļi) aizņem gandrīz
tikpat daudz vietas kā FLT (kodē arī decimālskatiļus), taču INT aizņem
minimāli mazāk vietas.

4.uzd.secinājums: ar INT1U no testētajiem kodējumiem fails aizņēma
vismazāk vietas.
