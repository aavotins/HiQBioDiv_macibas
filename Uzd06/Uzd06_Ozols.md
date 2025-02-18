Uzd06_Ozols
================
Janis Ozols
2025-02-04

Pirms uzdevuma veikšanas kā parasti sagatavojamies darbam ielādējot
nepieciešamās pakotnes ar “library()”, ja pakotnes ir nepieciešams
uzinstalēt izmantojam “install.packages”

``` r
library(sf)
library(arrow)
library(sfarrow)
library(concaveman)
library(ows4R)
library(httr)
library(tidyverse)
library(raster)
library(terra)
library(fasterize)
library(dplyr)
```

Uzdevuma ietvaros veidojamo rastru pārklājumam un šūnu novietojumam ir
jāsakrīt ar projekta Zenodo repozitorijā ievietoto references slāni
LV10m_10km.tif. Rasterizēšanas nosacījumam izmantojiet šūnas centru. 1.
Rasterizējiet Lauku atbalsta dienestā reģistrētos laukus, izveidojot
klasi “100” (vai izmantojiet trešajā uzdevumā sagatavoto slāni, ja tas
ir korekts); Atkārtosim 3. uzdevuma darbības, lai iegūtu rasterizējamos
datus un izveidosim terra rastra objektu ar klasi 100

``` r
shpcentrs <- st_read("../Uzd02/shpcentrs.shp")
```

    ## Reading layer `shpcentrs' from data source 
    ##   `C:\Users\Ozols\Documents\HiQBioDiv_macibas\Uzd02\shpcentrs.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 505654 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 405229.7 ymin: 234180.7 xmax: 594509.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM

``` r
teritorija <- st_union(shpcentrs)
teritorija_robeza <- concaveman(st_coordinates(teritorija))
teritorija_poligons <- st_polygon(list(teritorija_robeza[, 1:2]))
teritorija_poligons <- st_sfc(teritorija_poligons)
st_crs(teritorija_poligons) <- st_crs(shpcentrs)
cetrsturis <- st_bbox(teritorija_poligons)
cetrsturis <- st_transform(cetrsturis,crs = 4326)
teritorija_poligons
```

    ## Geometry set for 1 feature 
    ## Geometry type: POLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 405229.7 ymin: 234180.7 xmax: 594509.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM

    ## POLYGON ((405230.9 310878.7, 405229.7 310894.4,...

``` r
cetrsturis
```

    ##     xmin     ymin     xmax     ymax 
    ## 22.42279 56.24303 25.57287 57.41234

``` r
wfs_lad <- "https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer"
client <- WFSClient$new(wfs_lad, serviceVersion = "2.0.0")
client$getFeatureTypes(pretty = TRUE)
```

    ##        name title
    ## 1 LAD:Lauki Lauki

``` r
url <- parse_url(wfs_lad)
url$query <- list(service = "WFS",
                  request = "GetFeature",
                  typename = "Lauki",
                  srsName = "EPSG:4326",
                  bbox = paste(cetrsturis, collapse = ",")
                  )
request <- build_url(url)
dati <- read_sf(request)
dati <- st_transform(dati, crs = 3059 )
centrs <- st_intersection(dati, teritorija_poligons)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout
    ## all geometries

``` r
LV10m_10km <- rast("../Uzd03/LV10m_10km.tif")
st_write_parquet(centrs,"centra_lauku_dati.parquet")
```

    ## Warning: This is an initial implementation of Parquet/Feather file support and
    ## geo metadata. This is tracking version 0.1.0 of the metadata
    ## (https://github.com/geopandas/geo-arrow-spec). This metadata
    ## specification may change and does not yet make stability promises.  We
    ## do not yet recommend using this in a production setting unless you are
    ## able to rewrite your Parquet/Feather files.

``` r
centrs <- st_read_parquet("./centra_lauku_dati.parquet")
centrs_kaste <- st_bbox(centrs)
LV10m_10kmcrop <- crop(LV10m_10km, centrs_kaste)
lauki_rastrs <- rasterize(centrs, LV10m_10kmcrop)
lauki_100klase <- subst(lauki_rastrs,1,100)
plot(lauki_100klase)
```

![](Uzd06_Ozols_files/figure-gfm/1uzdevums-1.png)<!-- --> 2.Izveidojiet
rastra slāņus ar skujkoku (klase “204”), šaurlapju (klase “203”),
platlapju (klase “202”) un jauktu koku mežiem (klase “201”) no sevis
ierosinātās klasifikācijas otrajā uzdevumā. Papildus ieviešu klasi
1000 - nav mežs.

``` r
mezaudze <- st_read_parquet("../Uzd04/mezicentrs.parquet")
mezaudze <- mezaudze %>%
  filter(zkat == 10)
Mezaudzu_tips <- list(skujkoki = c(1, 3), platlapju_koki = c(10, 11, 12, 16, 17, 18, 20, 24), saurlapju_koki =c(4, 6, 8, 9))
mezaudze <- mezaudze %>% mutate (skujkoku_skerslaukums = 
ifelse(s10 %in% Mezaudzu_tips$skujkoki, g10, 0) +
ifelse(s11 %in% Mezaudzu_tips$skujkoki, g11, 0) +
ifelse(s12 %in% Mezaudzu_tips$skujkoki, g12, 0) +
ifelse(s13 %in% Mezaudzu_tips$skujkoki, g13, 0) +
ifelse(s14 %in% Mezaudzu_tips$skujkoki, g14, 0),
platlapju_skerslaukums = 
ifelse(s10 %in% Mezaudzu_tips$platlapju_koki, g10, 0) +
ifelse(s11 %in% Mezaudzu_tips$platlapju_koki, g11, 0) +
ifelse(s12 %in% Mezaudzu_tips$platlapju_koki, g12, 0) +
ifelse(s13 %in% Mezaudzu_tips$platlapju_koki, g13, 0) +
ifelse(s14 %in% Mezaudzu_tips$platlapju_koki, g14, 0),
saurlapju_skerslaukums = 
ifelse(s10 %in% Mezaudzu_tips$saurlapju_koki, g10, 0) +
ifelse(s11 %in% Mezaudzu_tips$saurlapju_koki, g11, 0) +
ifelse(s12 %in% Mezaudzu_tips$saurlapju_koki, g12, 0) +
ifelse(s13 %in% Mezaudzu_tips$saurlapju_koki, g13, 0) +
ifelse(s14 %in% Mezaudzu_tips$saurlapju_koki, g14, 0))
mezaudze <- mezaudze %>% mutate (kop_skerslaukums = g10+g11+g12+g13+g14)
mezaudze <- mezaudze %>% mutate (klase = case_when(                    kop_skerslaukums == 0 ~ 1000,
skujkoku_skerslaukums/kop_skerslaukums >= 0.90 ~ 204, saurlapju_skerslaukums/kop_skerslaukums >= 0.90 ~ 203, platlapju_skerslaukums/kop_skerslaukums >= 0.90 ~ 202,                             TRUE ~ 201))
klase204 <- mezaudze %>%
  filter(klase == 204)
klase203 <- mezaudze %>%
  filter(klase == 203)
klase202 <- mezaudze %>%
  filter(klase == 202)
klase201 <- mezaudze %>%
  filter(klase == 201)
skujkoku_klase <- rasterize(klase204,LV10m_10kmcrop, field = "klase")
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
saurlapju_klase <- rasterize(klase203,LV10m_10kmcrop, field = "klase")
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
platlapju_klase <- rasterize(klase202,LV10m_10kmcrop, field = "klase")
jauktu_klase <- rasterize(klase201,LV10m_10kmcrop, field = "klase")
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
plot(skujkoku_klase)
```

![](Uzd06_Ozols_files/figure-gfm/2uzdevums-1.png)<!-- -->

``` r
plot(saurlapju_klase)
```

![](Uzd06_Ozols_files/figure-gfm/2uzdevums-2.png)<!-- -->

``` r
plot(platlapju_klase)
```

![](Uzd06_Ozols_files/figure-gfm/2uzdevums-3.png)<!-- -->

``` r
plot(jauktu_klase)
```

![](Uzd06_Ozols_files/figure-gfm/2uzdevums-4.png)<!-- --> 3.
Papildieniet otrā punkta rastrus, izveidojot jaunus slāņus, ar
informāciju par mežaudzes vecuma grupu kā klases vērtības otro ciparu.
Dalījumam mežaudžu vecuma grupās, izmantojiet vecumgrupas un vecumklases
un, aprēķinot galvenās cirtes vecumu, pieņemiet “I un augstāka” bonitāti

Mežaudzes vecumu grupas iedalās: Jaunaudze = 1, 2 vecuma klase vidēja
vecuma audze = jaunaudze-briestaudze briestaudze = 1 klase pirms
ciršanas-ciršanas vecumam pieaugusi audze = ciršanas vecums-2klases virs
ciršanas pāraugusi audze = viss, kas vecāks par pieaugušu audzi

Vecuma klases iedalās atkarībā no sugas ātraudzības: skujkoku un cieto
lapkoku audzēm - 20 gadi mīkstajiem lapu kokiem 10 gadi ātraudzīgajiem
kokiem - vītols, baltalksnis, blīgzna 5 gadi

Ciršanas vecums I un augstākai bonitātei iedalās: priede, ozols, lapegle
\> 100 gadiem Egle, osis, liepa, goba, vīksna un kļava \> 80 gadiem;  
Bērzs, melnalksnis \> 70 gadiem;  
Apse \> 40 gadiem

Izveidosim katrai sugai lauku ar vecumu klasi un ciršanas vecumu, tad
izmantojot 1 stāva valdošās sugas vecumu noteiksim, kurā vecuma grupā ir
mežaudze

``` r
Vecuma_klases <- list(gadi20 = c(1, 3, 10, 11, 16), gadi10  = c(4, 6, 8, 12, 17, 18, 24), gadi5 =c(9, 20, 21))
Cirsanas_vecumi <- list(gadi100 = c(1, 10), gadi80 = c(3,11,12,16,17,18,24), gadi70 =c(4,6), gadi40 = c(8,9,20,21))
mezaudze <- mezaudze %>% mutate (Vecuma_klase = case_when(s10 %in% Vecuma_klases$gadi20 ~ 20, s10 %in% Vecuma_klases$gadi10 ~ 10, s10 %in% Vecuma_klases$gadi5 ~ 5, TRUE ~ NA_real_))
mezaudze <- mezaudze %>% mutate (Cirsanasvecums = case_when(s10 %in% Cirsanas_vecumi$gadi100 ~ 101, s10 %in% Cirsanas_vecumi$gadi80 ~ 81, s10 %in% Cirsanas_vecumi$gadi70 ~ 71, s10 %in% Cirsanas_vecumi$gadi40 ~ 41, TRUE ~ NA_real_))
#tā kā vecuma grupai ir jāatbilst klases otrajam ciparam, nosakam, ka jaunaudzes = 10, vidēja vecuma audzes = 20, briestaudzes = 30, pieaugušas audzes = 40, pāraugušas audzes = 50
mezaudze <- mezaudze %>% mutate (mezaudzes_vecuma_grupa = case_when(a10 <= Vecuma_klase * 2 ~ 10, a10 > Vecuma_klase * 2 & a10 < (Cirsanasvecums - Vecuma_klase) ~ 20, a10 >= (Cirsanasvecums - Vecuma_klase) & a10 < Cirsanasvecums ~ 30, a10 >= Cirsanasvecums & a10 < (Cirsanasvecums + Vecuma_klase*2) ~ 40, a10 >= (Cirsanasvecums + Vecuma_klase*2) ~ 50, TRUE ~ NA_real_))
mezaudze$klasearvecumu <- mezaudze$klase + mezaudze$mezaudzes_vecuma_grupa
klases <- unique(mezaudze$klasearvecumu)
print(klases)
```

    ##  [1]  231  243 1010 1020  224  214  223  213  253  221  234  233  244  241  251
    ## [16]  232  222  252  211  242   NA  254  212 1030 1040

Interesanti, ka izveidojas klases ar šķērslaukumu nulle pieaugušās
audzēs, kas nozīmē, ka ir mežaudzes ar pirmo stāvu, kurām nav noteikts
šķērslaukums. Varētu skatīties, kas tad šajos nogabalos atrodas un kā to
izmantot, bet šobrīd nenodarbosimies ar datu bāzes kārtošanu, tā vietā
atfiltrēsim tikai nogabalus, kur šķērslaukums ir lielāks par nulli.
Papildus apskatīsimies kādas koku sugas mums ir jauktu koku mežā, jo es
gribu tikai Latvijas pirmā stāva meža sugas, nevis stādījumu sugas,
atfiltrēsim mežaudzes tikai ar koku sugām, kuras ir izmantotas
klasifikācijā.

``` r
kokusugas <- unique(klase201$s10)
print(kokusugas)
```

    ##  [1] "16" "1"  "10" "8"  "9"  "4"  "3"  "24" "20" "11" "6"  "21" "19" "12" "25"
    ## [16] "13" "32" "23" "15" "63" "14" "61" "50" "17" "35" "67" "22"

``` r
mezaudze <- mezaudze %>%
  filter(kop_skerslaukums > 0)
mezaudze <- mezaudze %>% filter (s10 == 1 | s10 == 3 | s10 == 10 | s10 == 11 | s10 == 12 | s10 == 16 | s10 == 17 | s10 == 18 | s10 == 20 | s10 == 24 | s10 == 4 | s10 == 6 | s10 == 8 | s10 == 9)
kokusugasmezaudze <- unique(mezaudze$s10)
print(kokusugasmezaudze)
```

    ##  [1] "16" "9"  "1"  "4"  "6"  "10" "8"  "3"  "11" "20" "24" "12" "17"

``` r
Mezaudzu_tips <- list(skujkoki = c(1, 3), platlapju_koki = c(10, 11, 12, 16, 17, 18, 20, 24), saurlapju_koki =c(4, 6, 8, 9))
mezaudze <- mezaudze %>% mutate (skujkoku_skerslaukums = 
ifelse(s10 %in% Mezaudzu_tips$skujkoki, g10, 0) +
ifelse(s11 %in% Mezaudzu_tips$skujkoki, g11, 0) +
ifelse(s12 %in% Mezaudzu_tips$skujkoki, g12, 0) +
ifelse(s13 %in% Mezaudzu_tips$skujkoki, g13, 0) +
ifelse(s14 %in% Mezaudzu_tips$skujkoki, g14, 0),
platlapju_skerslaukums = 
ifelse(s10 %in% Mezaudzu_tips$platlapju_koki, g10, 0) +
ifelse(s11 %in% Mezaudzu_tips$platlapju_koki, g11, 0) +
ifelse(s12 %in% Mezaudzu_tips$platlapju_koki, g12, 0) +
ifelse(s13 %in% Mezaudzu_tips$platlapju_koki, g13, 0) +
ifelse(s14 %in% Mezaudzu_tips$platlapju_koki, g14, 0),
saurlapju_skerslaukums = 
ifelse(s10 %in% Mezaudzu_tips$saurlapju_koki, g10, 0) +
ifelse(s11 %in% Mezaudzu_tips$saurlapju_koki, g11, 0) +
ifelse(s12 %in% Mezaudzu_tips$saurlapju_koki, g12, 0) +
ifelse(s13 %in% Mezaudzu_tips$saurlapju_koki, g13, 0) +
ifelse(s14 %in% Mezaudzu_tips$saurlapju_koki, g14, 0))
mezaudze <- mezaudze %>% mutate (kop_skerslaukums = g10+g11+g12+g13+g14)
mezaudze <- mezaudze %>% mutate (klase = case_when(
skujkoku_skerslaukums/kop_skerslaukums >= 0.90 ~ 204, saurlapju_skerslaukums/kop_skerslaukums >= 0.90 ~ 203, platlapju_skerslaukums/kop_skerslaukums >= 0.90 ~ 202,                             TRUE ~ 201))
klase204 <- mezaudze %>%
  filter(klase == 204)
klase203 <- mezaudze %>%
  filter(klase == 203)
klase202 <- mezaudze %>%
  filter(klase == 202)
klase201 <- mezaudze %>%
  filter(klase == 201)
skujkoku_klase <- rasterize(klase204,LV10m_10kmcrop, field = "klase")
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
saurlapju_klase <- rasterize(klase203,LV10m_10kmcrop, field = "klase")
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
platlapju_klase <- rasterize(klase202,LV10m_10kmcrop, field = "klase")
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
jauktu_klase <- rasterize(klase201,LV10m_10kmcrop, field = "klase")
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
kokusugasklase201 <- unique(klase201$s10)
print(kokusugasklase201)
```

    ##  [1] "16" "1"  "10" "8"  "9"  "4"  "3"  "24" "20" "11" "6"  "12" "17"

``` r
plot(skujkoku_klase)
```

![](Uzd06_Ozols_files/figure-gfm/3uzdevums_labojumi-1.png)<!-- -->

``` r
plot(saurlapju_klase)
```

![](Uzd06_Ozols_files/figure-gfm/3uzdevums_labojumi-2.png)<!-- -->

``` r
plot(platlapju_klase)
```

![](Uzd06_Ozols_files/figure-gfm/3uzdevums_labojumi-3.png)<!-- -->

``` r
plot(jauktu_klase)
```

![](Uzd06_Ozols_files/figure-gfm/3uzdevums_labojumi-4.png)<!-- -->

``` r
mezaudze <- mezaudze %>% mutate (Vecuma_klase = case_when(s10 %in% Vecuma_klases$gadi20 ~ 20, s10 %in% Vecuma_klases$gadi10 ~ 10, s10 %in% Vecuma_klases$gadi5 ~ 5))
mezaudze <- mezaudze %>% mutate (Cirsanasvecums = case_when(s10 %in% Cirsanas_vecumi$gadi100 ~ 101, s10 %in% Cirsanas_vecumi$gadi80 ~ 81, s10 %in% Cirsanas_vecumi$gadi70 ~ 71, s10 %in% Cirsanas_vecumi$gadi40 ~ 41))
mezaudze <- mezaudze %>% mutate (mezaudzes_vecuma_grupa = case_when(a10 <= Vecuma_klase * 2 ~ 10, a10 > Vecuma_klase * 2 & a10 < (Cirsanasvecums - Vecuma_klase) ~ 20, a10 >= (Cirsanasvecums - Vecuma_klase) & a10 < Cirsanasvecums ~ 30, a10 >= Cirsanasvecums & a10 < (Cirsanasvecums + Vecuma_klase*2) ~ 40, a10 >= (Cirsanasvecums + Vecuma_klase*2) ~ 50))
mezaudze$klasearvecumu <- mezaudze$klase + mezaudze$mezaudzes_vecuma_grupa
klases <- unique(mezaudze$klasearvecumu)
print(klases)
```

    ##  [1] 231 243 224 214 223 213 253 221 234 233 244 241 251 232 222 252 211 242 254
    ## [20] 212

``` r
mezaudzes_rastri <- list()
for(klas in klases){
Mezaudzesklase <- mezaudze %>%  
  filter(klasearvecumu == klas)
mezaudzes_rastri[[as.character(klas)]] <- rasterize(Mezaudzesklase,LV10m_10kmcrop, field = "klasearvecumu")
}
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

``` r
for (n in names(mezaudzes_rastri)) {
  writeRaster(mezaudzes_rastri[[n]], filename = paste0(n, ".tif"), overwrite = TRUE)
}
r244 <- rast("244.tif")
plot(r244)
```

![](Uzd06_Ozols_files/figure-gfm/3uzdturpinajums-1.png)<!-- -->

4.  Savienojiet pirmajā un trešajā punktos izveidotos slāņus, tā, lai
    mazāka skaitliskā vērtība nozīmē augstāku prioritāti/svaru/dominanci
    rastra šūnas vērtības piešķiršanai

``` r
?merge #merge savieno rastrus pēc secības 
```

    ## starting httpd help server ... done

``` r
mezaudzes_rastri_seciba <- mezaudzes_rastri[order(names(mezaudzes_rastri))]
print(mezaudzes_rastri_seciba)
```

    ## $`211`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e472bb5678_4836_J0P2uAoeGj8eURN.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           211 
    ## max value   :           211 
    ## 
    ## $`212`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e479703d86_4836_I6VOVroMnudXN8o.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           212 
    ## max value   :           212 
    ## 
    ## $`213`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e42d1912c2_4836_4P3LWamMvx89ZOT.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           213 
    ## max value   :           213 
    ## 
    ## $`214`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e44a393057_4836_LuCqFxcmTzL4pna.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           214 
    ## max value   :           214 
    ## 
    ## $`221`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e430c13f5f_4836_bTGLeWgikzRi5uE.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           221 
    ## max value   :           221 
    ## 
    ## $`222`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e47d8d6ea_4836_LM26AmsODEGPU5D.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           222 
    ## max value   :           222 
    ## 
    ## $`223`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e446903e17_4836_aYjuhHgkOkYNCZW.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           223 
    ## max value   :           223 
    ## 
    ## $`224`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e41a1f4807_4836_l9EFoqzFKoIF0vD.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           224 
    ## max value   :           224 
    ## 
    ## $`231`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e4721655ed_4836_iAd7SUlxZLsaoD1.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           231 
    ## max value   :           231 
    ## 
    ## $`232`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e454537bb_4836_Py3ptIw2UFUXKtt.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           232 
    ## max value   :           232 
    ## 
    ## $`233`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e461c04b4d_4836_6Uxy02myoMrg5LO.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           233 
    ## max value   :           233 
    ## 
    ## $`234`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e4181a7a18_4836_VuY9VpSXXzR4qRb.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           234 
    ## max value   :           234 
    ## 
    ## $`241`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e46a0d547e_4836_8nr4ZcY08gqXcRL.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           241 
    ## max value   :           241 
    ## 
    ## $`242`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e42cd15a8a_4836_XiIHkcBogJBNMLc.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           242 
    ## max value   :           242 
    ## 
    ## $`243`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e42be030f_4836_koFoVwhPtQxaX98.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           243 
    ## max value   :           243 
    ## 
    ## $`244`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e410c74ee7_4836_rG2neQku8ByG59H.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           244 
    ## max value   :           244 
    ## 
    ## $`251`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e46c313ff1_4836_CV8ON4lE27FBVEp.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           251 
    ## max value   :           251 
    ## 
    ## $`252`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e4198c47e8_4836_8UwxxeZk3QEYL3o.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           252 
    ## max value   :           252 
    ## 
    ## $`253`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e46f6e7f64_4836_O0jKOAvnRJrWcA7.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           253 
    ## max value   :           253 
    ## 
    ## $`254`
    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e45ec3adc_4836_omf5NvSmIUuRURk.tif 
    ## varname     : LV10m_10km 
    ## name        : klasearvecumu 
    ## min value   :           254 
    ## max value   :           254

``` r
print(lauki_100klase)
```

    ## class       : SpatRaster 
    ## dimensions  : 12587, 16833, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408070, 576400, 234350, 360220  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source(s)   : memory
    ## varname     : LV10m_10km 
    ## name        : layer 
    ## min value   :   100 
    ## max value   :   100

``` r
#meginaju ar rastru listu apvienot, met ārā kļūdu, kad izmēģināju apvienot atsevišķus rastrus viss izdodas
r211 <- rast("211.tif")
r212 <- rast("212.tif")
r213 <- rast("213.tif")
r214 <- rast("214.tif")
r221 <- rast("221.tif")
r222 <- rast("222.tif")
r223 <- rast("223.tif")
r224 <- rast("224.tif")
r231 <- rast("231.tif")
r232 <- rast("232.tif")
r233 <- rast("233.tif")
r234 <- rast("234.tif")
r241 <- rast("241.tif")
r242 <- rast("242.tif")
r243 <- rast("243.tif")
r244 <- rast("244.tif")
r251 <- rast("251.tif")
r252 <- rast("252.tif")
r253 <- rast("253.tif")
r254 <- rast("254.tif")
apvienoti_rastri <- merge(lauki_100klase,r211,r212,r213,r214,r221,r222,r223,r224,r231,r232,r233,r234,r241,r242,r243,r244,r251,r252,r253,r254)
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
plot(apvienoti_rastri)
```

![](Uzd06_Ozols_files/figure-gfm/4uzdevums-1.png)<!-- --> 5. Cik šūnās
ir gan mežaudžu, gan lauku informācija? Apvienosim mežaudžu rastrus
vienā meža rastrā

``` r
meza_rastrs <- merge(r211,r212,r213,r214,r221,r222,r223,r224,r231,r232,r233,r234,r241,r242,r243,r244,r251,r252,r253,r254)
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
parklajas <- ifel(!is.na(lauki_100klase) & !is.na(meza_rastrs), 1, NA)
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

``` r
sunu_skaits <- freq(parklajas)
str(sunu_skaits) 
```

    ## 'data.frame':    1 obs. of  3 variables:
    ##  $ layer: num 1
    ##  $ value: num 1
    ##  $ count: num 1399

1399 šūnās ir gan mežaudžu, gan lauku informācija

6.  Cik šūnas atrodas Latvijas sauszemes teritorijā, bet nav raksturotas
    šī uzdevuma iepriekšējos punktos?

``` r
visa_latvija <- merge(apvienoti_rastri,LV10m_10km)
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
plot(visa_latvija)
```

![](Uzd06_Ozols_files/figure-gfm/6uzdevums-1.png)<!-- -->

``` r
visa_latvija #mazākā vērtība ir viens, tās ir šūnas, kuras ir sauszeme, bet nav definētas iepriekšējos uzdevumos
```

    ## class       : SpatRaster 
    ## dimensions  : 28600, 47000, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 302800, 772800, 162900, 448900  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : spat_12e460eb5f85_4836_6Ve0U0mHi6uNtMK.tif 
    ## varname     : LV10m_10km 
    ## name        : layer 
    ## min value   :     1 
    ## max value   :   254

``` r
tuksas_sunas <- global(visa_latvija == 1, "sum", na.rm=TRUE)
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
print(tuksas_sunas)
```

    ##             sum
    ## layer 601844320

601844320 šūnas atrodas Latvijas sauszemes teritorijā, bet nav
raksturotas šī uzdevuma iepriekšējos punktos
