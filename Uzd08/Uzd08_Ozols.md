Uzd08_Ozols
================
Janis Ozols

Sagatavojam darba vidi ielasot pakotnes

``` r
if (!require("sf")) install.packages("sf");library(sf)
if (!require("arrow")) install.packages("arrow");library(arrow)
if (!require("sfarrow")) install.packages("sfarrow");library(sfarrow)
if (!require("dplyr")) install.packages("dplyr");library(dplyr)
if (!require("terra")) install.packages("terra");library(terra)
if (!require("tools")) install.packages("tools");library(tools)
```

Uzdevumu veiciet GEE interneta pārlūkā (JavaScript) vai ar R pakotnēm,
izmantojiet harmonizēto Sentinel-2 kolekciju visai Latvijas teritorijai.
Latvijas teritoriju definējiet ar projekta Zenodo repozitorijā doto 100
m vektordatu tīklu.

``` r
#uzdevumu veiksim GEE interneta pārlūkā. Gee atbalsta shape failu formātu, līdz ar to parquet failu pārveidosim par shp failu, tā kā fails pārsniedz dbf lielumu > 2gb, sadalīsim parquet failu pa 50 km lapām. 
#Mēģināju darīt ar R pakotnēm, bet problēma radās ar rgee instalāciju, rakstīja, ka nav python vides, mēģināju notīrīt un uzinstalēt, bet neizdevās, laikam jāpārliek python uz jaunāku versiju un jādefinē vidi python, tad jāielasa rgee to norādot, bet neesmu pārliecināts, ka sanāks, tāpēc mēģināšu pārlūkā sadalot failu lapās
tikls100 <- st_read_parquet("../Uzd03/tikls100_sauzeme.parquet") |> 
  st_as_sf()
if (!inherits(tikls100$geom, "sfc")) {
  st_geometry(tikls100) <- "geom"
}
tikls100 <- st_make_valid(tikls100)
unique_tiles <- unique(tikls100$tks50km)
output_dir <- "tks50km_shapefiles"
zip_dir <- "zipped_shapefiles"
dir.create(zip_dir, showWarnings = FALSE)
dir.create(output_dir, showWarnings = FALSE)
for (tile in unique_tiles) {
  tile_data <- tikls100 %>% dplyr::filter(tks50km == tile)
  fname <- paste0("tks50km_", tile)
  shapefile_path <- file.path(output_dir, fname)
  st_write(tile_data, paste0(shapefile_path, ".shp"), delete_layer = TRUE)
  files_to_zip <- list.files(output_dir, pattern = paste0("^", fname, "\\."), full.names = TRUE)
  zipfile <- file.path(zip_dir, paste0(fname, ".zip"))
  zip(zipfile = zipfile, files = files_to_zip, flags = "-j")
}
```

    ## Deleting layer `tks50km_3113' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3113' to data source 
    ##   `tks50km_shapefiles/tks50km_3113.shp' using driver `ESRI Shapefile'
    ## Writing 28526 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3131' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3131' to data source 
    ##   `tks50km_shapefiles/tks50km_3131.shp' using driver `ESRI Shapefile'
    ## Writing 26294 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3111' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3111' to data source 
    ##   `tks50km_shapefiles/tks50km_3111.shp' using driver `ESRI Shapefile'
    ## Writing 7403 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3133' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3133' to data source 
    ##   `tks50km_shapefiles/tks50km_3133.shp' using driver `ESRI Shapefile'
    ## Writing 14292 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4112' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4112' to data source 
    ##   `tks50km_shapefiles/tks50km_4112.shp' using driver `ESRI Shapefile'
    ## Writing 45695 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3114' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3114' to data source 
    ##   `tks50km_shapefiles/tks50km_3114.shp' using driver `ESRI Shapefile'
    ## Writing 39389 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3132' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3132' to data source 
    ##   `tks50km_shapefiles/tks50km_3132.shp' using driver `ESRI Shapefile'
    ## Writing 62500 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3134' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3134' to data source 
    ##   `tks50km_shapefiles/tks50km_3134.shp' using driver `ESRI Shapefile'
    ## Writing 62500 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4114' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4114' to data source 
    ##   `tks50km_shapefiles/tks50km_4114.shp' using driver `ESRI Shapefile'
    ## Writing 16113 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4132' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4132' to data source 
    ##   `tks50km_shapefiles/tks50km_4132.shp' using driver `ESRI Shapefile'
    ## Writing 3869 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3123' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3123' to data source 
    ##   `tks50km_shapefiles/tks50km_3123.shp' using driver `ESRI Shapefile'
    ## Writing 8889 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3141' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3141' to data source 
    ##   `tks50km_shapefiles/tks50km_3141.shp' using driver `ESRI Shapefile'
    ## Writing 62484 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3143' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3143' to data source 
    ##   `tks50km_shapefiles/tks50km_3143.shp' using driver `ESRI Shapefile'
    ## Writing 62500 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4121' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4121' to data source 
    ##   `tks50km_shapefiles/tks50km_4121.shp' using driver `ESRI Shapefile'
    ## Writing 62500 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4123' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4123' to data source 
    ##   `tks50km_shapefiles/tks50km_4123.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4141' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4141' to data source 
    ##   `tks50km_shapefiles/tks50km_4141.shp' using driver `ESRI Shapefile'
    ## Writing 56039 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4143' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4143' to data source 
    ##   `tks50km_shapefiles/tks50km_4143.shp' using driver `ESRI Shapefile'
    ## Writing 14004 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3142' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3142' to data source 
    ##   `tks50km_shapefiles/tks50km_3142.shp' using driver `ESRI Shapefile'
    ## Writing 53826 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3144' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3144' to data source 
    ##   `tks50km_shapefiles/tks50km_3144.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4122' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4122' to data source 
    ##   `tks50km_shapefiles/tks50km_4122.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4124' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4124' to data source 
    ##   `tks50km_shapefiles/tks50km_4124.shp' using driver `ESRI Shapefile'
    ## Writing 62499 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4142' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4142' to data source 
    ##   `tks50km_shapefiles/tks50km_4142.shp' using driver `ESRI Shapefile'
    ## Writing 63001 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4144' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4144' to data source 
    ##   `tks50km_shapefiles/tks50km_4144.shp' using driver `ESRI Shapefile'
    ## Writing 37507 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3231' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3231' to data source 
    ##   `tks50km_shapefiles/tks50km_3231.shp' using driver `ESRI Shapefile'
    ## Writing 63311 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3233' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3233' to data source 
    ##   `tks50km_shapefiles/tks50km_3233.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4211' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4211' to data source 
    ##   `tks50km_shapefiles/tks50km_4211.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4213' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4213' to data source 
    ##   `tks50km_shapefiles/tks50km_4213.shp' using driver `ESRI Shapefile'
    ## Writing 62001 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4231' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4231' to data source 
    ##   `tks50km_shapefiles/tks50km_4231.shp' using driver `ESRI Shapefile'
    ## Writing 62499 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4233' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4233' to data source 
    ##   `tks50km_shapefiles/tks50km_4233.shp' using driver `ESRI Shapefile'
    ## Writing 43195 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_5211' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_5211' to data source 
    ##   `tks50km_shapefiles/tks50km_5211.shp' using driver `ESRI Shapefile'
    ## Writing 1407 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3232' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3232' to data source 
    ##   `tks50km_shapefiles/tks50km_3232.shp' using driver `ESRI Shapefile'
    ## Writing 61758 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3234' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3234' to data source 
    ##   `tks50km_shapefiles/tks50km_3234.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4212' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4212' to data source 
    ##   `tks50km_shapefiles/tks50km_4212.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4214' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4214' to data source 
    ##   `tks50km_shapefiles/tks50km_4214.shp' using driver `ESRI Shapefile'
    ## Writing 62464 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4232' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4232' to data source 
    ##   `tks50km_shapefiles/tks50km_4232.shp' using driver `ESRI Shapefile'
    ## Writing 38115 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3214' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3214' to data source 
    ##   `tks50km_shapefiles/tks50km_3214.shp' using driver `ESRI Shapefile'
    ## Writing 10028 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3223' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3223' to data source 
    ##   `tks50km_shapefiles/tks50km_3223.shp' using driver `ESRI Shapefile'
    ## Writing 8888 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3241' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3241' to data source 
    ##   `tks50km_shapefiles/tks50km_3241.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3243' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3243' to data source 
    ##   `tks50km_shapefiles/tks50km_3243.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4221' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4221' to data source 
    ##   `tks50km_shapefiles/tks50km_4221.shp' using driver `ESRI Shapefile'
    ## Writing 54608 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4223' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4223' to data source 
    ##   `tks50km_shapefiles/tks50km_4223.shp' using driver `ESRI Shapefile'
    ## Writing 6689 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3224' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3224' to data source 
    ##   `tks50km_shapefiles/tks50km_3224.shp' using driver `ESRI Shapefile'
    ## Writing 13335 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3242' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3242' to data source 
    ##   `tks50km_shapefiles/tks50km_3242.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3244' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3244' to data source 
    ##   `tks50km_shapefiles/tks50km_3244.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4222' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4222' to data source 
    ##   `tks50km_shapefiles/tks50km_4222.shp' using driver `ESRI Shapefile'
    ## Writing 39461 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3313' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3313' to data source 
    ##   `tks50km_shapefiles/tks50km_3313.shp' using driver `ESRI Shapefile'
    ## Writing 32054 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3331' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3331' to data source 
    ##   `tks50km_shapefiles/tks50km_3331.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3333' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3333' to data source 
    ##   `tks50km_shapefiles/tks50km_3333.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4311' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4311' to data source 
    ##   `tks50km_shapefiles/tks50km_4311.shp' using driver `ESRI Shapefile'
    ## Writing 62008 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4313' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4313' to data source 
    ##   `tks50km_shapefiles/tks50km_4313.shp' using driver `ESRI Shapefile'
    ## Writing 18638 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_5311' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_5311' to data source 
    ##   `tks50km_shapefiles/tks50km_5311.shp' using driver `ESRI Shapefile'
    ## Writing 6803 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4334' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4334' to data source 
    ##   `tks50km_shapefiles/tks50km_4334.shp' using driver `ESRI Shapefile'
    ## Writing 70417 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4332' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4332' to data source 
    ##   `tks50km_shapefiles/tks50km_4332.shp' using driver `ESRI Shapefile'
    ## Writing 66178 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3314' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3314' to data source 
    ##   `tks50km_shapefiles/tks50km_3314.shp' using driver `ESRI Shapefile'
    ## Writing 17448 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3332' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3332' to data source 
    ##   `tks50km_shapefiles/tks50km_3332.shp' using driver `ESRI Shapefile'
    ## Writing 61911 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3334' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3334' to data source 
    ##   `tks50km_shapefiles/tks50km_3334.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4312' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4312' to data source 
    ##   `tks50km_shapefiles/tks50km_4312.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4314' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4314' to data source 
    ##   `tks50km_shapefiles/tks50km_4314.shp' using driver `ESRI Shapefile'
    ## Writing 62499 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_5312' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_5312' to data source 
    ##   `tks50km_shapefiles/tks50km_5312.shp' using driver `ESRI Shapefile'
    ## Writing 59174 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3341' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3341' to data source 
    ##   `tks50km_shapefiles/tks50km_3341.shp' using driver `ESRI Shapefile'
    ## Writing 59450 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3343' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3343' to data source 
    ##   `tks50km_shapefiles/tks50km_3343.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4321' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4321' to data source 
    ##   `tks50km_shapefiles/tks50km_4321.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4323' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4323' to data source 
    ##   `tks50km_shapefiles/tks50km_4323.shp' using driver `ESRI Shapefile'
    ## Writing 62001 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4341' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4341' to data source 
    ##   `tks50km_shapefiles/tks50km_4341.shp' using driver `ESRI Shapefile'
    ## Writing 62499 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4343' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4343' to data source 
    ##   `tks50km_shapefiles/tks50km_4343.shp' using driver `ESRI Shapefile'
    ## Writing 62001 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_5321' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_5321' to data source 
    ##   `tks50km_shapefiles/tks50km_5321.shp' using driver `ESRI Shapefile'
    ## Writing 62499 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_5323' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_5323' to data source 
    ##   `tks50km_shapefiles/tks50km_5323.shp' using driver `ESRI Shapefile'
    ## Writing 19666 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3323' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3323' to data source 
    ##   `tks50km_shapefiles/tks50km_3323.shp' using driver `ESRI Shapefile'
    ## Writing 28116 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3324' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3324' to data source 
    ##   `tks50km_shapefiles/tks50km_3324.shp' using driver `ESRI Shapefile'
    ## Writing 64645 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3342' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3342' to data source 
    ##   `tks50km_shapefiles/tks50km_3342.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3344' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3344' to data source 
    ##   `tks50km_shapefiles/tks50km_3344.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4322' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4322' to data source 
    ##   `tks50km_shapefiles/tks50km_4322.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4324' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4324' to data source 
    ##   `tks50km_shapefiles/tks50km_4324.shp' using driver `ESRI Shapefile'
    ## Writing 62499 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4342' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4342' to data source 
    ##   `tks50km_shapefiles/tks50km_4342.shp' using driver `ESRI Shapefile'
    ## Writing 63001 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4344' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4344' to data source 
    ##   `tks50km_shapefiles/tks50km_4344.shp' using driver `ESRI Shapefile'
    ## Writing 62499 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_5322' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_5322' to data source 
    ##   `tks50km_shapefiles/tks50km_5322.shp' using driver `ESRI Shapefile'
    ## Writing 61039 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_5324' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_5324' to data source 
    ##   `tks50km_shapefiles/tks50km_5324.shp' using driver `ESRI Shapefile'
    ## Writing 11206 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3411' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3411' to data source 
    ##   `tks50km_shapefiles/tks50km_3411.shp' using driver `ESRI Shapefile'
    ## Writing 29602 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3413' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3413' to data source 
    ##   `tks50km_shapefiles/tks50km_3413.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3431' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3431' to data source 
    ##   `tks50km_shapefiles/tks50km_3431.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3433' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3433' to data source 
    ##   `tks50km_shapefiles/tks50km_3433.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4411' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4411' to data source 
    ##   `tks50km_shapefiles/tks50km_4411.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4413' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4413' to data source 
    ##   `tks50km_shapefiles/tks50km_4413.shp' using driver `ESRI Shapefile'
    ## Writing 62001 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4431' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4431' to data source 
    ##   `tks50km_shapefiles/tks50km_4431.shp' using driver `ESRI Shapefile'
    ## Writing 62499 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4433' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4433' to data source 
    ##   `tks50km_shapefiles/tks50km_4433.shp' using driver `ESRI Shapefile'
    ## Writing 62001 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_5411' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_5411' to data source 
    ##   `tks50km_shapefiles/tks50km_5411.shp' using driver `ESRI Shapefile'
    ## Writing 36472 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3412' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3412' to data source 
    ##   `tks50km_shapefiles/tks50km_3412.shp' using driver `ESRI Shapefile'
    ## Writing 61644 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3414' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3414' to data source 
    ##   `tks50km_shapefiles/tks50km_3414.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3432' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3432' to data source 
    ##   `tks50km_shapefiles/tks50km_3432.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3434' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3434' to data source 
    ##   `tks50km_shapefiles/tks50km_3434.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4412' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4412' to data source 
    ##   `tks50km_shapefiles/tks50km_4412.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4414' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4414' to data source 
    ##   `tks50km_shapefiles/tks50km_4414.shp' using driver `ESRI Shapefile'
    ## Writing 62499 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4432' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4432' to data source 
    ##   `tks50km_shapefiles/tks50km_4432.shp' using driver `ESRI Shapefile'
    ## Writing 63001 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4434' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4434' to data source 
    ##   `tks50km_shapefiles/tks50km_4434.shp' using driver `ESRI Shapefile'
    ## Writing 37131 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_2434' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_2434' to data source 
    ##   `tks50km_shapefiles/tks50km_2434.shp' using driver `ESRI Shapefile'
    ## Writing 25380 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_2443' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_2443' to data source 
    ##   `tks50km_shapefiles/tks50km_2443.shp' using driver `ESRI Shapefile'
    ## Writing 62780 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3421' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3421' to data source 
    ##   `tks50km_shapefiles/tks50km_3421.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3423' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3423' to data source 
    ##   `tks50km_shapefiles/tks50km_3423.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3441' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3441' to data source 
    ##   `tks50km_shapefiles/tks50km_3441.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3443' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3443' to data source 
    ##   `tks50km_shapefiles/tks50km_3443.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4421' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4421' to data source 
    ##   `tks50km_shapefiles/tks50km_4421.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4423' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4423' to data source 
    ##   `tks50km_shapefiles/tks50km_4423.shp' using driver `ESRI Shapefile'
    ## Writing 62001 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4441' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4441' to data source 
    ##   `tks50km_shapefiles/tks50km_4441.shp' using driver `ESRI Shapefile'
    ## Writing 62499 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4443' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4443' to data source 
    ##   `tks50km_shapefiles/tks50km_4443.shp' using driver `ESRI Shapefile'
    ## Writing 21724 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_2444' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_2444' to data source 
    ##   `tks50km_shapefiles/tks50km_2444.shp' using driver `ESRI Shapefile'
    ## Writing 33083 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3422' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3422' to data source 
    ##   `tks50km_shapefiles/tks50km_3422.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3424' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3424' to data source 
    ##   `tks50km_shapefiles/tks50km_3424.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3442' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3442' to data source 
    ##   `tks50km_shapefiles/tks50km_3442.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3444' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3444' to data source 
    ##   `tks50km_shapefiles/tks50km_3444.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4422' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4422' to data source 
    ##   `tks50km_shapefiles/tks50km_4422.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4424' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4424' to data source 
    ##   `tks50km_shapefiles/tks50km_4424.shp' using driver `ESRI Shapefile'
    ## Writing 62499 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4442' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4442' to data source 
    ##   `tks50km_shapefiles/tks50km_4442.shp' using driver `ESRI Shapefile'
    ## Writing 63001 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4444' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4444' to data source 
    ##   `tks50km_shapefiles/tks50km_4444.shp' using driver `ESRI Shapefile'
    ## Writing 25665 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_2533' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_2533' to data source 
    ##   `tks50km_shapefiles/tks50km_2533.shp' using driver `ESRI Shapefile'
    ## Writing 31132 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3511' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3511' to data source 
    ##   `tks50km_shapefiles/tks50km_3511.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3513' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3513' to data source 
    ##   `tks50km_shapefiles/tks50km_3513.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3531' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3531' to data source 
    ##   `tks50km_shapefiles/tks50km_3531.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3533' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3533' to data source 
    ##   `tks50km_shapefiles/tks50km_3533.shp' using driver `ESRI Shapefile'
    ## Writing 62250 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4511' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4511' to data source 
    ##   `tks50km_shapefiles/tks50km_4511.shp' using driver `ESRI Shapefile'
    ## Writing 61135 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4513' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4513' to data source 
    ##   `tks50km_shapefiles/tks50km_4513.shp' using driver `ESRI Shapefile'
    ## Writing 62001 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4531' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4531' to data source 
    ##   `tks50km_shapefiles/tks50km_4531.shp' using driver `ESRI Shapefile'
    ## Writing 52566 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4533' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4533' to data source 
    ##   `tks50km_shapefiles/tks50km_4533.shp' using driver `ESRI Shapefile'
    ## Writing 9080 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3512' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3512' to data source 
    ##   `tks50km_shapefiles/tks50km_3512.shp' using driver `ESRI Shapefile'
    ## Writing 29249 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3514' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3514' to data source 
    ##   `tks50km_shapefiles/tks50km_3514.shp' using driver `ESRI Shapefile'
    ## Writing 62104 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3532' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3532' to data source 
    ##   `tks50km_shapefiles/tks50km_3532.shp' using driver `ESRI Shapefile'
    ## Writing 62750 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3534' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3534' to data source 
    ##   `tks50km_shapefiles/tks50km_3534.shp' using driver `ESRI Shapefile'
    ## Writing 49612 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4512' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4512' to data source 
    ##   `tks50km_shapefiles/tks50km_4512.shp' using driver `ESRI Shapefile'
    ## Writing 15937 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4514' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4514' to data source 
    ##   `tks50km_shapefiles/tks50km_4514.shp' using driver `ESRI Shapefile'
    ## Writing 9992 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_4532' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_4532' to data source 
    ##   `tks50km_shapefiles/tks50km_4532.shp' using driver `ESRI Shapefile'
    ## Writing 8370 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3523' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3523' to data source 
    ##   `tks50km_shapefiles/tks50km_3523.shp' using driver `ESRI Shapefile'
    ## Writing 20742 features with 6 fields and geometry type Polygon.
    ## Deleting layer `tks50km_3541' using driver `ESRI Shapefile'
    ## Writing layer `tks50km_3541' to data source 
    ##   `tks50km_shapefiles/tks50km_3541.shp' using driver `ESRI Shapefile'
    ## Writing 18110 features with 6 fields and geometry type Polygon.

Šeit ir links uz radīto projektu ar visām komandrindām:
<https://code.earthengine.google.com/?accept_repo=users/janisozols/uzdevums8>
Projektā ir 6 skripti, skripts rezgis_100m - apvieno visas tks50km lapas
vienā failā, skripts apaksuzdevums1 - izpilda visu 1.-1.3 uzdevumā
prasīto, skripts apaksuzdevums2 - izpildu visu 2-2.3 uzdevumā prasīto,
skripts lejupielāde - izpilda vēlreiz 2.3 uzdevumu vienai lapai un
sadala to daļās lejupielādei, skripts tiek manuāli palaists katrai lapai
nomainot lapas nr pie var assetId, skripts uzdevums3 aprēķina IQR,
skripts IQR_karte apvieno IQR failus un attēlo kartē, tālāk apvienosim
šeit iegūtos rastrus:

``` r
tif_files <- list.files(path = "../Uzd08/GEE_Exports", pattern = "\\.tif$", full.names = TRUE)
rasters <- lapply(tif_files, rast)
rasters
```

    ## [[1]]
    ## class       : SpatRaster 
    ## dimensions  : 2431, 2101, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 629000, 650010, 175690, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_2434.tif 
    ## name        : NDVI 
    ## 
    ## [[2]]
    ## class       : SpatRaster 
    ## dimensions  : 2070, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 641990, 648010, 179300, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_2434_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[3]]
    ## class       : SpatRaster 
    ## dimensions  : 780, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 694990, 7e+05, 192200, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_2444_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[4]]
    ## class       : SpatRaster 
    ## dimensions  : 1000, 191, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 726000, 727910, 190000, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_2533_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[5]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 2502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 324890, 349910, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3132_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[6]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 192, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 372990, 374910, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3141_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[7]]
    ## class       : SpatRaster 
    ## dimensions  : 912, 2501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 349900, 374910, 290890, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3143_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[8]]
    ## class       : SpatRaster 
    ## dimensions  : 60, 102, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 448990, 450010, 249400, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3214_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[9]]
    ## class       : SpatRaster 
    ## dimensions  : 731, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 494990, 500010, 242690, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3224_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[10]]
    ## class       : SpatRaster 
    ## dimensions  : 602, 522, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 425990, 431210, 249990, 256010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3232_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[11]]
    ## class       : SpatRaster 
    ## dimensions  : 1502, 2491, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 399990, 424900, 284990, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3233_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[12]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 91, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 473990, 474900, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3241_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[13]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 201, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 498000, 500010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3242_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[14]]
    ## class       : SpatRaster 
    ## dimensions  : 461, 1241, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 477000, 489410, 275000, 279610  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3244_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[15]]
    ## class       : SpatRaster 
    ## dimensions  : 1350, 191, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 523000, 524910, 236500, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3313_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[16]]
    ## class       : SpatRaster 
    ## dimensions  : 2791, 202, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 597990, 600010, 222090, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3324_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[17]]
    ## class       : SpatRaster 
    ## dimensions  : 1361, 2491, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 5e+05, 524910, 261400, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3331_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[18]]
    ## class       : SpatRaster 
    ## dimensions  : 1802, 360, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 546400, 550000, 256990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3332_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[19]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 503990, 508000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3333_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[20]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 91, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 524000, 524910, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3333_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[21]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 528000, 533010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3334_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[22]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 531990, 537010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3334_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[23]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 535990, 542010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3334_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[24]]
    ## class       : SpatRaster 
    ## dimensions  : 2321, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 540990, 546010, 275000, 298210  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3334_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[25]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 1320, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 536800, 550000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3334_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[26]]
    ## class       : SpatRaster 
    ## dimensions  : 512, 800, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 542000, 550000, 294890, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3334_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[27]]
    ## class       : SpatRaster 
    ## dimensions  : 2301, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 549990, 555000, 252000, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3341_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[28]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 553990, 560010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3341_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[29]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 558990, 564000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3341_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[30]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 563000, 568000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3341_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[31]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 567000, 572010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3341_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[32]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 392, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 570990, 574910, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3341_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[33]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 411, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 574900, 579010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3342_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[34]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 577990, 583010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3342_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[35]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 581990, 587000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3342_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[36]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 586000, 591000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3342_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[37]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 590000, 595010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3342_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[38]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 593990, 600010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3342_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[39]]
    ## class       : SpatRaster 
    ## dimensions  : 1982, 391, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 596100, 600010, 255190, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3342_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[40]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 549990, 554000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3343_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[41]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 553990, 558010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3343_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[42]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 558000, 562000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3343_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[43]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 561990, 566000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3343_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[44]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 565990, 570010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3343_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[45]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 570000, 574010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3343_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[46]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 91, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 574000, 574910, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3343_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[47]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 411, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 574900, 579010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3344_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[48]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 577990, 583010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3344_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[49]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 581990, 587000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3344_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[50]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 586000, 591000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3344_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[51]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 590000, 595010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3344_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[52]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 593990, 599010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3344_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[53]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 202, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 597990, 600010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3344_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[54]]
    ## class       : SpatRaster 
    ## dimensions  : 1321, 1400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 6e+05, 614000, 211790, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3411_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[55]]
    ## class       : SpatRaster 
    ## dimensions  : 1821, 701, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 613000, 620010, 206790, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3411_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[56]]
    ## class       : SpatRaster 
    ## dimensions  : 2131, 591, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 618990, 624900, 203690, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3411_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[57]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 512, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 624890, 630010, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3412_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[58]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 629000, 634010, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3412_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[59]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 632990, 638000, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3412_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[60]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 636990, 642000, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3412_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[61]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 641000, 646010, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3412_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[62]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 644990, 650010, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3412_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[63]]
    ## class       : SpatRaster 
    ## dimensions  : 1660, 102, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 648990, 650010, 208400, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3412_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[64]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 6e+05, 604010, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3413_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[65]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 604000, 608000, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3413_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[66]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 607990, 612000, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3413_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[67]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 611990, 617000, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3413_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[68]]
    ## class       : SpatRaster 
    ## dimensions  : 2411, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 616000, 621000, 224990, 249100  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3413_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[69]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 1121, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 613690, 624900, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3413_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[70]]
    ## class       : SpatRaster 
    ## dimensions  : 421, 890, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 616000, 624900, 245790, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3413_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[71]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 412, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 624890, 629010, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3414_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[72]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 627990, 633000, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3414_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[73]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 632000, 637000, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3414_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[74]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 636000, 641010, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3414_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[75]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 639990, 645010, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3414_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[76]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 643990, 650010, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3414_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[77]]
    ## class       : SpatRaster 
    ## dimensions  : 2021, 582, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 644190, 650010, 229790, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3414_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[78]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 650000, 654000, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3421_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[79]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 653990, 658000, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3421_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[80]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 657990, 662010, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3421_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[81]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 662000, 666010, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3421_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[82]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 666000, 671010, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3421_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[83]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 611, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 668800, 674910, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3421_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[84]]
    ## class       : SpatRaster 
    ## dimensions  : 1092, 292, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 671990, 674910, 199990, 210910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3421_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[85]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 410, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 674900, 679000, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3422_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[86]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 678000, 683000, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3422_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[87]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 682000, 687010, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3422_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[88]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 685990, 691000, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3422_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[89]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 689990, 695000, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3422_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[90]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 694000, 699010, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3422_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[91]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 200, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 698000, 7e+05, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3422_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[92]]
    ## class       : SpatRaster 
    ## dimensions  : 2321, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 650000, 655010, 224990, 248200  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3423_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[93]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 653990, 659010, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3423_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[94]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 657990, 663000, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3423_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[95]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 662000, 667000, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3423_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[96]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 666000, 671010, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3423_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[97]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 492, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 669990, 674910, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3423_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[98]]
    ## class       : SpatRaster 
    ## dimensions  : 890, 2491, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 650000, 674910, 241100, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3423_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[99]]
    ## class       : SpatRaster 
    ## dimensions  : 2450, 511, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 674900, 680010, 225500, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3424_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[100]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 678990, 684000, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3424_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[101]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 682990, 688000, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3424_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[102]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 687000, 692010, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3424_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[103]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 690990, 696010, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3424_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[104]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 694990, 7e+05, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3424_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[105]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 2510, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 674900, 7e+05, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3424_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[106]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 6e+05, 604010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3431_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[107]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 604000, 608000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3431_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[108]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 607990, 612000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3431_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[109]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 611990, 616010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3431_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[110]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 616000, 620010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3431_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[111]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 620000, 624000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3431_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[112]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 91, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 623990, 624900, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3431_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[113]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 412, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 624890, 629010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3432_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[114]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 627990, 633000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3432_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[115]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 632000, 637000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3432_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[116]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 636000, 641010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3432_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[117]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 639990, 645010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3432_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[118]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 643990, 649000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3432_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[119]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 201, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 648000, 650010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3432_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[120]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 6e+05, 604010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3433_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[121]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 604000, 608000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3433_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[122]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 607990, 612000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3433_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[123]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 611990, 616010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3433_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[124]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 616000, 620010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3433_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[125]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 620000, 624000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3433_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[126]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 91, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 623990, 624900, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3433_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[127]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 412, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 624890, 629010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3434_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[128]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 627990, 633000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3434_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[129]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 632000, 637000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3434_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[130]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 636000, 641010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3434_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[131]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 639990, 645010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3434_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[132]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 643990, 650010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3434_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[133]]
    ## class       : SpatRaster 
    ## dimensions  : 1911, 382, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 646190, 650010, 280900, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3434_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[134]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 650000, 654000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3441_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[135]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 653990, 658000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3441_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[136]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 657990, 663000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3441_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[137]]
    ## class       : SpatRaster 
    ## dimensions  : 2431, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 662000, 667000, 249990, 274300  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3441_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[138]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 666000, 671010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3441_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[139]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 492, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 669990, 674910, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3441_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[140]]
    ## class       : SpatRaster 
    ## dimensions  : 1361, 1452, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 660390, 674910, 261400, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3441_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[141]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 410, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 674900, 679000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3442_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[142]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 678000, 683000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3442_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[143]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 682000, 687010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3442_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[144]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 685990, 691000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3442_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[145]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 689990, 695000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3442_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[146]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 694000, 699010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3442_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[147]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 200, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 698000, 7e+05, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3442_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[148]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 650000, 654000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3443_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[149]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 653990, 658000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3443_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[150]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 657990, 662010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3443_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[151]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 662000, 666010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3443_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[152]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 666000, 670000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3443_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[153]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 669990, 674000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3443_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[154]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 92, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 673990, 674910, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3443_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[155]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 410, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 674900, 679000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3444_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[156]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 678000, 683000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3444_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[157]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 682000, 687010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3444_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[158]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 685990, 691000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3444_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[159]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 689990, 695000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3444_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[160]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 694000, 699010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3444_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[161]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 200, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 698000, 7e+05, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3444_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[162]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 699990, 704000, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3511_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[163]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 703990, 708010, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3511_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[164]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 708000, 712010, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3511_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[165]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 712000, 716000, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3511_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[166]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 715990, 720000, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3511_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[167]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 719990, 724010, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3511_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[168]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 91, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 724000, 724910, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3511_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[169]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 510, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 724900, 730000, 199990, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3512_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[170]]
    ## class       : SpatRaster 
    ## dimensions  : 1911, 601, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 729000, 735010, 205890, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3512_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[171]]
    ## class       : SpatRaster 
    ## dimensions  : 1531, 1251, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 733990, 746500, 209690, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3512_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[172]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 699990, 704000, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3513_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[173]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 703990, 708010, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3513_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[174]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 708000, 712010, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3513_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[175]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 712000, 716000, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3513_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[176]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 715990, 720000, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3513_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[177]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 719990, 724010, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3513_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[178]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 91, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 724000, 724910, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3513_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[179]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 411, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 724900, 729010, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3514_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[180]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 728000, 733010, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3514_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[181]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 731990, 737000, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3514_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[182]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 735990, 741000, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3514_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[183]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 740000, 745010, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3514_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[184]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 601, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 743990, 750000, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3514_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[185]]
    ## class       : SpatRaster 
    ## dimensions  : 2191, 100, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 749000, 750000, 228090, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3514_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[186]]
    ## class       : SpatRaster 
    ## dimensions  : 2340, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 749990, 755000, 226600, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3523_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[187]]
    ## class       : SpatRaster 
    ## dimensions  : 1911, 701, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 754000, 761010, 230890, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3523_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[188]]
    ## class       : SpatRaster 
    ## dimensions  : 850, 262, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 759990, 762610, 241500, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3523_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[189]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 699990, 705010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3531_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[190]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 703990, 709000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3531_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[191]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 708000, 713000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3531_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[192]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 712000, 717010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3531_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[193]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 715990, 721010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3531_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[194]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 492, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 719990, 724910, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3531_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[195]]
    ## class       : SpatRaster 
    ## dimensions  : 2061, 2492, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 699990, 724910, 254400, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3531_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[196]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 411, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 724900, 729010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3532_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[197]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 728000, 733010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3532_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[198]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 731990, 737000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3532_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[199]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 735990, 741000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3532_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[200]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 740000, 745010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3532_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[201]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 743990, 749010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3532_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[202]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 201, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 747990, 750000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3532_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[203]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 699990, 704000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3533_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[204]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 703990, 708010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3533_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[205]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 708000, 713000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3533_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[206]]
    ## class       : SpatRaster 
    ## dimensions  : 2481, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 712000, 717010, 275200, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3533_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[207]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 715990, 721010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3533_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[208]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 492, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 719990, 724910, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3533_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[209]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 1421, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 710700, 724910, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3533_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[210]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 411, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 724900, 729010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3534_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[211]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 728000, 733010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3534_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[212]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 731990, 737000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3534_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[213]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 735990, 742010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3534_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[214]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 901, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 740990, 750000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3534_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[215]]
    ## class       : SpatRaster 
    ## dimensions  : 2882, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 749990, 754010, 249990, 278810  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3541_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[216]]
    ## class       : SpatRaster 
    ## dimensions  : 2861, 712, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 752990, 760110, 249990, 278600  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_3541_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[217]]
    ## class       : SpatRaster 
    ## dimensions  : 1150, 1301, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 319990, 333000, 3e+05, 311500  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4112_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[218]]
    ## class       : SpatRaster 
    ## dimensions  : 1870, 800, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 332000, 340000, 3e+05, 318700  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4112_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[219]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 339000, 344010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4112_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[220]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 342990, 348010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4112_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[221]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 292, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 346990, 349910, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4112_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[222]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 562, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 342390, 348010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4114_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[223]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 292, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 346990, 349910, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4114_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[224]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 410, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 349900, 354000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4121_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[225]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 353000, 358010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4121_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[226]]
    ## class       : SpatRaster 
    ## dimensions  : 2400, 591, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 357090, 363000, 3e+05, 324000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4121_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[227]]
    ## class       : SpatRaster 
    ## dimensions  : 2240, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 362000, 367010, 3e+05, 322400  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4121_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[228]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 491, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 366100, 371010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4121_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[229]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 2041, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 354500, 374910, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4121_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[230]]
    ## class       : SpatRaster 
    ## dimensions  : 412, 1042, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 356990, 367410, 320890, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4121_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[231]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 410, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 374900, 379000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4122_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[232]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 378000, 383010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4122_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[233]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 381990, 387010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4122_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[234]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 385990, 391000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4122_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[235]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 390000, 395010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4122_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[236]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 394000, 399010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4122_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[237]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 201, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 397990, 4e+05, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4122_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[238]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 410, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 349900, 354000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4123_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[239]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 353000, 358010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4123_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[240]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 356990, 362010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4123_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[241]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 360990, 366000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4123_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[242]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 365000, 370000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4123_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[243]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 369000, 374010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4123_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[244]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 111, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 373800, 374910, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4123_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[245]]
    ## class       : SpatRaster 
    ## dimensions  : 2381, 611, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 374900, 381010, 325000, 348810  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4124_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[246]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 379990, 385010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4124_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[247]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 383990, 389000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4124_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[248]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 388000, 393000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4124_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[249]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 392000, 397010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4124_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[250]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 2510, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 374900, 4e+05, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4124_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[251]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 551, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 374990, 380500, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4124_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[252]]
    ## class       : SpatRaster 
    ## dimensions  : 1081, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 343890, 349910, 349900, 360710  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4132_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[253]]
    ## class       : SpatRaster 
    ## dimensions  : 1920, 710, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 349900, 357000, 349900, 369100  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4141_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[254]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 355990, 361000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4141_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[255]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 360000, 365010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4141_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[256]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 364000, 369010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4141_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[257]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 367990, 373000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4141_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[258]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 291, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 372000, 374910, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4141_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[259]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 410, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 374900, 379000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4142_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[260]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 378000, 383010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4142_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[261]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 381990, 387010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4142_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[262]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 385990, 391000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4142_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[263]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 390000, 395010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4142_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[264]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 394000, 399010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4142_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[265]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 740, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 392600, 4e+05, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4142_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[266]]
    ## class       : SpatRaster 
    ## dimensions  : 1112, 1592, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 358990, 374910, 374990, 386110  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4143_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[267]]
    ## class       : SpatRaster 
    ## dimensions  : 1101, 391, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 371000, 374910, 374990, 386000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4143_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[268]]
    ## class       : SpatRaster 
    ## dimensions  : 1251, 910, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 374900, 384000, 374990, 387500  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4144_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[269]]
    ## class       : SpatRaster 
    ## dimensions  : 1601, 800, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 383000, 391000, 374990, 391000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4144_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[270]]
    ## class       : SpatRaster 
    ## dimensions  : 1872, 701, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 390000, 397010, 374990, 393710  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4144_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[271]]
    ## class       : SpatRaster 
    ## dimensions  : 2072, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 395990, 4e+05, 374990, 395710  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4144_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[272]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 399990, 404010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4211_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[273]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 404000, 408010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4211_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[274]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408000, 412000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4211_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[275]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 411990, 416000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4211_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[276]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 415990, 420010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4211_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[277]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 420000, 424010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4211_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[278]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 90, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 424000, 424900, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4211_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[279]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 511, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 424890, 430000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4212_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[280]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 429000, 434010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4212_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[281]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 432990, 438010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4212_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[282]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 436990, 442000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4212_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[283]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 441000, 446000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4212_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[284]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 445000, 450010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4212_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[285]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 2512, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 424890, 450010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4212_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[286]]
    ## class       : SpatRaster 
    ## dimensions  : 2222, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 399990, 405000, 327690, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4213_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[287]]
    ## class       : SpatRaster 
    ## dimensions  : 2411, 600, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 404000, 410000, 325800, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4213_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[288]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408990, 414000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4213_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[289]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 413000, 418010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4213_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[290]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 417000, 422010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4213_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[291]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 2491, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 399990, 424900, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4213_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[292]]
    ## class       : SpatRaster 
    ## dimensions  : 411, 1100, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 401000, 412000, 325000, 329110  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4213_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[293]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 412, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 424890, 429010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4214_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[294]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 427990, 433000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4214_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[295]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 431990, 437000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4214_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[296]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 436000, 441010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4214_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[297]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 440000, 445010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4214_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[298]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 443990, 450010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4214_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[299]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 191, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 448100, 450010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4214_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[300]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 450000, 454010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4221_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[301]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 431, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 454000, 458310, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4221_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[302]]
    ## class       : SpatRaster 
    ## dimensions  : 2471, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 457990, 463010, 3e+05, 324710  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4221_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[303]]
    ## class       : SpatRaster 
    ## dimensions  : 2250, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 461990, 467000, 3e+05, 322500  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4221_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[304]]
    ## class       : SpatRaster 
    ## dimensions  : 2160, 600, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 466000, 472000, 3e+05, 321600  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4221_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[305]]
    ## class       : SpatRaster 
    ## dimensions  : 1720, 390, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 471000, 474900, 3e+05, 317200  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4221_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[306]]
    ## class       : SpatRaster 
    ## dimensions  : 1460, 811, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 474890, 483000, 3e+05, 314600  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4222_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[307]]
    ## class       : SpatRaster 
    ## dimensions  : 1531, 800, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 482000, 490000, 3e+05, 315310  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4222_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[308]]
    ## class       : SpatRaster 
    ## dimensions  : 1790, 701, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 489000, 496010, 3e+05, 317900  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4222_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[309]]
    ## class       : SpatRaster 
    ## dimensions  : 2191, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 494990, 500010, 3e+05, 321910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4222_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[310]]
    ## class       : SpatRaster 
    ## dimensions  : 2290, 760, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 450000, 457600, 325000, 347900  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4223_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[311]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 399990, 404010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4231_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[312]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 402990, 408010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4231_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[313]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 406990, 412000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4231_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[314]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 411000, 416000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4231_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[315]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 415000, 420010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4231_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[316]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 591, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 418990, 424900, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4231_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[317]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 420, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 420700, 424900, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4231_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[318]]
    ## class       : SpatRaster 
    ## dimensions  : 2710, 412, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 424890, 429010, 349900, 377000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4232_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[319]]
    ## class       : SpatRaster 
    ## dimensions  : 2521, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 427990, 434010, 349900, 375110  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4232_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[320]]
    ## class       : SpatRaster 
    ## dimensions  : 1951, 702, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 432990, 440010, 349900, 369410  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4232_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[321]]
    ## class       : SpatRaster 
    ## dimensions  : 1260, 1071, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 438990, 449700, 349900, 362500  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4232_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[322]]
    ## class       : SpatRaster 
    ## dimensions  : 2281, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 399990, 405000, 374990, 397800  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4233_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[323]]
    ## class       : SpatRaster 
    ## dimensions  : 2472, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 404000, 409000, 374990, 399710  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4233_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[324]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408000, 413010, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4233_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[325]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 701, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 411990, 419000, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4233_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[326]]
    ## class       : SpatRaster 
    ## dimensions  : 912, 700, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 418000, 425000, 374990, 384110  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4233_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[327]]
    ## class       : SpatRaster 
    ## dimensions  : 2402, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 5e+05, 505010, 300990, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4311_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[328]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 503990, 509000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4311_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[329]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 507990, 513000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4311_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[330]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 512000, 517010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4311_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[331]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 515990, 521010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4311_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[332]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 2491, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 5e+05, 524910, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4311_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[333]]
    ## class       : SpatRaster 
    ## dimensions  : 1350, 750, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 5e+05, 507500, 3e+05, 313500  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4311_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[334]]
    ## class       : SpatRaster 
    ## dimensions  : 2290, 511, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 524900, 530010, 3e+05, 322900  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4312_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[335]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 528990, 534000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4312_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[336]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 533000, 538000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4312_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[337]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 537000, 542010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4312_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[338]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 540990, 546010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4312_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[339]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 544990, 550000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4312_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[340]]
    ## class       : SpatRaster 
    ## dimensions  : 962, 2510, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 524900, 550000, 315390, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4312_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[341]]
    ## class       : SpatRaster 
    ## dimensions  : 1271, 1920, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 500800, 520000, 325000, 337710  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4313_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[342]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 591, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 519000, 524910, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4313_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[343]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 410, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 524900, 529000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4314_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[344]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 528000, 533010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4314_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[345]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 531990, 537010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4314_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[346]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 511, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 536300, 541410, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4314_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[347]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 540990, 546010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4314_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[348]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 544990, 550000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4314_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[349]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 2510, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 524900, 550000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4314_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[350]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 549990, 554000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4321_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[351]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 553990, 558010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4321_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[352]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 558000, 562000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4321_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[353]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 561990, 566000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4321_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[354]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 565990, 570010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4321_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[355]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 570000, 574010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4321_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[356]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 91, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 574000, 574910, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4321_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[357]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 411, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 574900, 579010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4322_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[358]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 577990, 583010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4322_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[359]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 581990, 587000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4322_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[360]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 586000, 591000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4322_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[361]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 590000, 595010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4322_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[362]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 593990, 599010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4322_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[363]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 202, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 597990, 600010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4322_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[364]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 549990, 555000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4323_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[365]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 553990, 559000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4323_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[366]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 558000, 563010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4323_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[367]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 561990, 567010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4323_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[368]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 565990, 571000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4323_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[369]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 491, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 570000, 574910, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4323_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[370]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 381, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 571100, 574910, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4323_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[371]]
    ## class       : SpatRaster 
    ## dimensions  : 2451, 510, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 574900, 580000, 325000, 349510  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4324_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[372]]
    ## class       : SpatRaster 
    ## dimensions  : 2301, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 579000, 584000, 325000, 348010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4324_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[373]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 583000, 588010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4324_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[374]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 586990, 592010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4324_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[375]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 590990, 596000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4324_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[376]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 595000, 600010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4324_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[377]]
    ## class       : SpatRaster 
    ## dimensions  : 752, 2511, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 574900, 600010, 342390, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4324_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[378]]
    ## class       : SpatRaster 
    ## dimensions  : 2261, 641, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 522590, 529000, 352390, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4332_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[379]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 600, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 528000, 534000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4332_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[380]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 533000, 538000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4332_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[381]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 537000, 542010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4332_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[382]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 540990, 546010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4332_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[383]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 544990, 550000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4332_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[384]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 2581, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 524190, 550000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4332_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[385]]
    ## class       : SpatRaster 
    ## dimensions  : 2081, 731, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 521690, 529000, 377500, 398310  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4334_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[386]]
    ## class       : SpatRaster 
    ## dimensions  : 2431, 600, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 528000, 534000, 375600, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4334_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[387]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 533000, 538000, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4334_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[388]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 537000, 542010, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4334_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[389]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 540990, 546010, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4334_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[390]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 544990, 550000, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4334_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[391]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 2961, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 520390, 550000, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4334_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[392]]
    ## class       : SpatRaster 
    ## dimensions  : 212, 550, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 530000, 535500, 374990, 377110  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4334_part_7.tif 
    ## name        : NDVI 
    ## 
    ## [[393]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 549990, 554000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4341_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[394]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 553000, 558010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4341_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[395]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 556990, 562000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4341_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[396]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 560990, 566000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4341_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[397]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 565000, 570010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4341_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[398]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 569000, 574010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4341_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[399]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 192, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 572990, 574910, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4341_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[400]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 510, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 574900, 580000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4342_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[401]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 579000, 584000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4342_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[402]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 583000, 588010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4342_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[403]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 586990, 592010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4342_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[404]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 590990, 596000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4342_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[405]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 595000, 600010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4342_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[406]]
    ## class       : SpatRaster 
    ## dimensions  : 2400, 2511, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 574900, 600010, 351000, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4342_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[407]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 549990, 555000, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4343_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[408]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 553990, 559000, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4343_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[409]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 558000, 563010, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4343_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[410]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 561990, 567010, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4343_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[411]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 565990, 571000, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4343_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[412]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 491, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 570000, 574910, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4343_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[413]]
    ## class       : SpatRaster 
    ## dimensions  : 2252, 91, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 574000, 574910, 377390, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4343_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[414]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 411, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 574900, 579010, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4344_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[415]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 577990, 583010, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4344_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[416]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 581990, 587000, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4344_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[417]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 586000, 591000, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4344_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[418]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 590000, 595010, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4344_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[419]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 593990, 599010, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4344_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[420]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 122, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 598790, 600010, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4344_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[421]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 6e+05, 604010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4411_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[422]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 604000, 608000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4411_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[423]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 607990, 612000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4411_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[424]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 611990, 616010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4411_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[425]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 616000, 620010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4411_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[426]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 620000, 624000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4411_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[427]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 91, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 623990, 624900, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4411_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[428]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 412, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 624890, 629010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4412_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[429]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 627990, 633000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4412_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[430]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 632000, 637000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4412_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[431]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 636000, 641010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4412_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[432]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 639990, 646010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4412_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[433]]
    ## class       : SpatRaster 
    ## dimensions  : 2330, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 644990, 650010, 3e+05, 323300  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4412_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[434]]
    ## class       : SpatRaster 
    ## dimensions  : 1381, 952, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 640490, 650010, 311200, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4412_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[435]]
    ## class       : SpatRaster 
    ## dimensions  : 2400, 601, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 6e+05, 606010, 325000, 349000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4413_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[436]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 604990, 610000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4413_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[437]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 609000, 614000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4413_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[438]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 613000, 618010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4413_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[439]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 616990, 622010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4413_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[440]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 2490, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 6e+05, 624900, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4413_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[441]]
    ## class       : SpatRaster 
    ## dimensions  : 811, 540, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 6e+05, 605400, 341800, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4413_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[442]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 412, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 624890, 629010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4414_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[443]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 627990, 633000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4414_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[444]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 512, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 632790, 637910, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4414_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[445]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 636990, 642000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4414_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[446]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 641000, 646010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4414_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[447]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 2512, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 624890, 650010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4414_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[448]]
    ## class       : SpatRaster 
    ## dimensions  : 712, 452, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 645490, 650010, 342790, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4414_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[449]]
    ## class       : SpatRaster 
    ## dimensions  : 2420, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 650000, 655010, 3e+05, 324200  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4421_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[450]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 653990, 659010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4421_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[451]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 657990, 663000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4421_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[452]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 662000, 667000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4421_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[453]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 666000, 671010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4421_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[454]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 492, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 669990, 674910, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4421_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[455]]
    ## class       : SpatRaster 
    ## dimensions  : 1101, 2491, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 650000, 674910, 314000, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4421_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[456]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 410, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 674900, 679000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4422_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[457]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 678000, 683000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4422_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[458]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 682000, 687010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4422_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[459]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 685990, 691000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4422_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[460]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 689990, 695000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4422_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[461]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 694000, 699010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4422_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[462]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 2510, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 674900, 7e+05, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4422_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[463]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 650000, 655010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4423_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[464]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 653990, 659010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4423_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[465]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 657990, 663000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4423_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[466]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 662000, 667000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4423_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[467]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 666000, 671010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4423_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[468]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 492, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 669990, 674910, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4423_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[469]]
    ## class       : SpatRaster 
    ## dimensions  : 2251, 92, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 673990, 674910, 327400, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4423_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[470]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 410, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 674900, 679000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4424_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[471]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 678000, 683000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4424_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[472]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 682000, 687010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4424_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[473]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 685990, 692010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4424_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[474]]
    ## class       : SpatRaster 
    ## dimensions  : 2370, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 690990, 696010, 325000, 348700  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4424_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[475]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 1210, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 687900, 7e+05, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4424_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[476]]
    ## class       : SpatRaster 
    ## dimensions  : 461, 831, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 691690, 7e+05, 345300, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4424_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[477]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 6e+05, 604010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4431_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[478]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 602990, 608000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4431_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[479]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 606990, 612000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4431_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[480]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 611000, 616010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4431_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[481]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 614990, 620010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4431_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[482]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 618990, 624000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4431_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[483]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 190, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 623000, 624900, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4431_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[484]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 512, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 624890, 630010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4432_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[485]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 629000, 634010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4432_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[486]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 632990, 637000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4432_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[487]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 636000, 641010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4432_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[488]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 639990, 645010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4432_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[489]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 643990, 649000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4432_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[490]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 2512, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 624890, 650010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4432_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[491]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 6e+05, 605000, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4433_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[492]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 604000, 609010, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4433_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[493]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 607990, 613010, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4433_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[494]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 611990, 617000, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4433_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[495]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 616000, 621000, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4433_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[496]]
    ## class       : SpatRaster 
    ## dimensions  : 2492, 490, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 620000, 624900, 374990, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4433_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[497]]
    ## class       : SpatRaster 
    ## dimensions  : 2122, 2490, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 6e+05, 624900, 378690, 399910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4433_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[498]]
    ## class       : SpatRaster 
    ## dimensions  : 2891, 412, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 624890, 629010, 374990, 403900  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4434_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[499]]
    ## class       : SpatRaster 
    ## dimensions  : 2602, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 627990, 633000, 374990, 401010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4434_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[500]]
    ## class       : SpatRaster 
    ## dimensions  : 1922, 1000, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 632000, 642000, 374990, 394210  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4434_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[501]]
    ## class       : SpatRaster 
    ## dimensions  : 961, 901, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 641000, 650010, 374990, 384600  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4434_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[502]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 650000, 654000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4441_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[503]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 653000, 658000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4441_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[504]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 657000, 662010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4441_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[505]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 660990, 666010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4441_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[506]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 664990, 670000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4441_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[507]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 591, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 669000, 674910, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4441_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[508]]
    ## class       : SpatRaster 
    ## dimensions  : 2011, 362, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 671290, 674910, 354890, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4441_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[509]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 410, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 674900, 679000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4442_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[510]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 678000, 683000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4442_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[511]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 682000, 687010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4442_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[512]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 685990, 691000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4442_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[513]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 689990, 695000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4442_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[514]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 694000, 699010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4442_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[515]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 200, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 698000, 7e+05, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4442_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[516]]
    ## class       : SpatRaster 
    ## dimensions  : 1121, 1601, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 650000, 666010, 374990, 386200  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4443_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[517]]
    ## class       : SpatRaster 
    ## dimensions  : 1592, 901, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 664990, 674000, 374990, 390910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4443_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[518]]
    ## class       : SpatRaster 
    ## dimensions  : 1671, 191, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 673000, 674910, 374990, 391700  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4443_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[519]]
    ## class       : SpatRaster 
    ## dimensions  : 1641, 740, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 674900, 682300, 374990, 391400  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4444_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[520]]
    ## class       : SpatRaster 
    ## dimensions  : 1141, 1100, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 682000, 693000, 374990, 386400  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4444_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[521]]
    ## class       : SpatRaster 
    ## dimensions  : 881, 800, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 692000, 7e+05, 374990, 383800  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4444_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[522]]
    ## class       : SpatRaster 
    ## dimensions  : 2411, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 699990, 705010, 300900, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4511_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[523]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 703990, 709000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4511_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[524]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 708000, 713000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4511_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[525]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 712000, 717010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4511_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[526]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 715990, 721010, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4511_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[527]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 492, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 719990, 724910, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4511_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[528]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 2492, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 699990, 724910, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4511_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[529]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 910, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 724900, 734000, 3e+05, 325010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4512_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[530]]
    ## class       : SpatRaster 
    ## dimensions  : 1060, 981, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 733000, 742810, 3e+05, 310600  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4512_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[531]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 699990, 705010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4513_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[532]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 703990, 709000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4513_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[533]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 708000, 713000, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4513_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[534]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 712000, 717010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4513_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[535]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 715990, 721010, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4513_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[536]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 492, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 719990, 724910, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4513_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[537]]
    ## class       : SpatRaster 
    ## dimensions  : 2251, 91, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 724000, 724910, 327400, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4513_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[538]]
    ## class       : SpatRaster 
    ## dimensions  : 2491, 741, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 724900, 732310, 325000, 349910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4514_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[539]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 699990, 704000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4531_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[540]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 703000, 708010, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4531_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[541]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 601, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 706990, 713000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4531_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[542]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 600, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 712000, 718000, 349900, 375000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4531_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[543]]
    ## class       : SpatRaster 
    ## dimensions  : 1880, 701, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 717000, 724010, 349900, 368700  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4531_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[544]]
    ## class       : SpatRaster 
    ## dimensions  : 1431, 192, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 722990, 724910, 349900, 364210  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4531_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[545]]
    ## class       : SpatRaster 
    ## dimensions  : 1370, 860, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 724900, 733500, 349900, 363600  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4532_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[546]]
    ## class       : SpatRaster 
    ## dimensions  : 802, 1372, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 699990, 713710, 374990, 383010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_4533_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[547]]
    ## class       : SpatRaster 
    ## dimensions  : 290, 870, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408400, 417100, 399900, 402800  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5211_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[548]]
    ## class       : SpatRaster 
    ## dimensions  : 1520, 521, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 519700, 524910, 399900, 415100  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5311_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[549]]
    ## class       : SpatRaster 
    ## dimensions  : 2121, 610, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 524900, 531000, 399900, 421110  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5312_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[550]]
    ## class       : SpatRaster 
    ## dimensions  : 2400, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 530000, 535010, 399900, 423900  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5312_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[551]]
    ## class       : SpatRaster 
    ## dimensions  : 2480, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 533990, 539010, 399900, 424700  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5312_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[552]]
    ## class       : SpatRaster 
    ## dimensions  : 2451, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 537990, 543000, 399900, 424410  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5312_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[553]]
    ## class       : SpatRaster 
    ## dimensions  : 2761, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 542000, 547010, 399900, 427510  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5312_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[554]]
    ## class       : SpatRaster 
    ## dimensions  : 2810, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 546000, 550000, 399900, 428000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5312_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[555]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 549990, 554000, 399900, 425000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5321_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[556]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 553000, 558010, 399900, 425000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5321_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[557]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 556990, 562000, 399900, 425000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5321_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[558]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 560990, 566000, 399900, 425000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5321_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[559]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 565000, 570010, 399900, 425000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5321_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[560]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 569000, 574010, 399900, 425000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5321_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[561]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 192, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 572990, 574910, 399900, 425000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5321_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[562]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 411, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 574900, 579010, 399900, 425000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5322_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[563]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 577990, 583010, 399900, 425000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5322_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[564]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 581990, 587000, 399900, 425000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5322_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[565]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 586000, 591000, 399900, 425000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5322_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[566]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 590000, 595010, 399900, 425000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5322_part_4.tif 
    ## name        : NDVI 
    ## 
    ## [[567]]
    ## class       : SpatRaster 
    ## dimensions  : 2510, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 593990, 600010, 399900, 425000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5322_part_5.tif 
    ## name        : NDVI 
    ## 
    ## [[568]]
    ## class       : SpatRaster 
    ## dimensions  : 1101, 101, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 599000, 600010, 409390, 420400  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5322_part_6.tif 
    ## name        : NDVI 
    ## 
    ## [[569]]
    ## class       : SpatRaster 
    ## dimensions  : 1172, 1601, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 549990, 566000, 424990, 436710  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5323_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[570]]
    ## class       : SpatRaster 
    ## dimensions  : 1392, 991, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 565000, 574910, 424990, 438910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5323_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[571]]
    ## class       : SpatRaster 
    ## dimensions  : 1362, 1311, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 574900, 588010, 424990, 438610  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5324_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[572]]
    ## class       : SpatRaster 
    ## dimensions  : 312, 742, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 586990, 594410, 424990, 428110  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5324_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[573]]
    ## class       : SpatRaster 
    ## dimensions  : 2171, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 6e+05, 605000, 399900, 421610  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5411_part_0.tif 
    ## name        : NDVI 
    ## 
    ## [[574]]
    ## class       : SpatRaster 
    ## dimensions  : 2101, 701, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 604000, 611010, 399900, 420910  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5411_part_1.tif 
    ## name        : NDVI 
    ## 
    ## [[575]]
    ## class       : SpatRaster 
    ## dimensions  : 1540, 802, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 609990, 618010, 399900, 415300  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5411_part_2.tif 
    ## name        : NDVI 
    ## 
    ## [[576]]
    ## class       : SpatRaster 
    ## dimensions  : 1491, 712, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 617890, 625010, 399900, 414810  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI_5411_part_3.tif 
    ## name        : NDVI 
    ## 
    ## [[577]]
    ## class       : SpatRaster 
    ## dimensions  : 1800, 1401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 629000, 643010, 182000, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI2434part0.tif 
    ## name        : NDVI 
    ## 
    ## [[578]]
    ## class       : SpatRaster 
    ## dimensions  : 2070, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 641990, 648010, 179300, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI2434part1.tif 
    ## name        : NDVI 
    ## 
    ## [[579]]
    ## class       : SpatRaster 
    ## dimensions  : 2431, 302, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 646990, 650010, 175690, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI2434part2.tif 
    ## name        : NDVI 
    ## 
    ## [[580]]
    ## class       : SpatRaster 
    ## dimensions  : 2420, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 650000, 655010, 175800, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI2443part0.tif 
    ## name        : NDVI 
    ## 
    ## [[581]]
    ## class       : SpatRaster 
    ## dimensions  : 2691, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 653990, 659010, 173090, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI2443part1.tif 
    ## name        : NDVI 
    ## 
    ## [[582]]
    ## class       : SpatRaster 
    ## dimensions  : 2700, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 657990, 662010, 173000, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI2443part2.tif 
    ## name        : NDVI 
    ## 
    ## [[583]]
    ## class       : SpatRaster 
    ## dimensions  : 2711, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 660990, 666010, 172890, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI2443part3.tif 
    ## name        : NDVI 
    ## 
    ## [[584]]
    ## class       : SpatRaster 
    ## dimensions  : 2480, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 664990, 670000, 175200, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI2443part4.tif 
    ## name        : NDVI 
    ## 
    ## [[585]]
    ## class       : SpatRaster 
    ## dimensions  : 2620, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 669000, 674000, 173800, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI2443part5.tif 
    ## name        : NDVI 
    ## 
    ## [[586]]
    ## class       : SpatRaster 
    ## dimensions  : 2631, 191, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 673000, 674910, 173690, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI2443part6.tif 
    ## name        : NDVI 
    ## 
    ## [[587]]
    ## class       : SpatRaster 
    ## dimensions  : 2631, 511, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 674900, 680010, 173690, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI2444part0.tif 
    ## name        : NDVI 
    ## 
    ## [[588]]
    ## class       : SpatRaster 
    ## dimensions  : 2220, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 678990, 685010, 177800, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI2444part1.tif 
    ## name        : NDVI 
    ## 
    ## [[589]]
    ## class       : SpatRaster 
    ## dimensions  : 1291, 1202, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 683990, 696010, 187090, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI2444part2.tif 
    ## name        : NDVI 
    ## 
    ## [[590]]
    ## class       : SpatRaster 
    ## dimensions  : 780, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 694990, 7e+05, 192200, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI2444part3.tif 
    ## name        : NDVI 
    ## 
    ## [[591]]
    ## class       : SpatRaster 
    ## dimensions  : 1300, 901, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 699990, 709000, 187000, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI2533part0.tif 
    ## name        : NDVI 
    ## 
    ## [[592]]
    ## class       : SpatRaster 
    ## dimensions  : 1190, 1101, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 708000, 719010, 188100, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI2533part1.tif 
    ## name        : NDVI 
    ## 
    ## [[593]]
    ## class       : SpatRaster 
    ## dimensions  : 1231, 901, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 717990, 727000, 187690, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI2533part2.tif 
    ## name        : NDVI 
    ## 
    ## [[594]]
    ## class       : SpatRaster 
    ## dimensions  : 1000, 191, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 726000, 727910, 190000, 2e+05  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI2533part3.tif 
    ## name        : NDVI 
    ## 
    ## [[595]]
    ## class       : SpatRaster 
    ## dimensions  : 740, 1282, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 315690, 328510, 217600, 225000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3111part0.tif 
    ## name        : NDVI 
    ## 
    ## [[596]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 522, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 312790, 318010, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3113part0.tif 
    ## name        : NDVI 
    ## 
    ## [[597]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 316990, 322000, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3113part1.tif 
    ## name        : NDVI 
    ## 
    ## [[598]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 390, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 321000, 324900, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3113part2.tif 
    ## name        : NDVI 
    ## 
    ## [[599]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 512, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 324890, 330010, 224990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3114part0.tif 
    ## name        : NDVI 
    ## 
    ## [[600]]
    ## class       : SpatRaster 
    ## dimensions  : 2201, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 328990, 334010, 227990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3114part1.tif 
    ## name        : NDVI 
    ## 
    ## [[601]]
    ## class       : SpatRaster 
    ## dimensions  : 1680, 802, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 332990, 341010, 233200, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3114part2.tif 
    ## name        : NDVI 
    ## 
    ## [[602]]
    ## class       : SpatRaster 
    ## dimensions  : 1400, 992, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 339990, 349910, 236000, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3114part3.tif 
    ## name        : NDVI 
    ## 
    ## [[603]]
    ## class       : SpatRaster 
    ## dimensions  : 690, 2501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 349900, 374910, 243100, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3123part0.tif 
    ## name        : NDVI 
    ## 
    ## [[604]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 611, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 312900, 319010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3131part0.tif 
    ## name        : NDVI 
    ## 
    ## [[605]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 318000, 323010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3131part1.tif 
    ## name        : NDVI 
    ## 
    ## [[606]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 291, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 321990, 324900, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3131part2.tif 
    ## name        : NDVI 
    ## 
    ## [[607]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 411, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 324890, 329000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3132part0.tif 
    ## name        : NDVI 
    ## 
    ## [[608]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 328000, 333000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3132part1.tif 
    ## name        : NDVI 
    ## 
    ## [[609]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 332000, 337010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3132part2.tif 
    ## name        : NDVI 
    ## 
    ## [[610]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 335990, 341010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3132part3.tif 
    ## name        : NDVI 
    ## 
    ## [[611]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 339990, 345000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3132part4.tif 
    ## name        : NDVI 
    ## 
    ## [[612]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 344000, 349000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3132part5.tif 
    ## name        : NDVI 
    ## 
    ## [[613]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 2502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 324890, 349910, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3132part6.tif 
    ## name        : NDVI 
    ## 
    ## [[614]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 760, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 316400, 324000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3133part0.tif 
    ## name        : NDVI 
    ## 
    ## [[615]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 190, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 323000, 324900, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3133part1.tif 
    ## name        : NDVI 
    ## 
    ## [[616]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 411, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 324890, 329000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3134part0.tif 
    ## name        : NDVI 
    ## 
    ## [[617]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 328000, 333000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3134part1.tif 
    ## name        : NDVI 
    ## 
    ## [[618]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 332000, 337010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3134part2.tif 
    ## name        : NDVI 
    ## 
    ## [[619]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 335990, 341010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3134part3.tif 
    ## name        : NDVI 
    ## 
    ## [[620]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 339990, 346010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3134part4.tif 
    ## name        : NDVI 
    ## 
    ## [[621]]
    ## class       : SpatRaster 
    ## dimensions  : 2160, 492, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 344990, 349910, 275000, 296600  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3134part5.tif 
    ## name        : NDVI 
    ## 
    ## [[622]]
    ## class       : SpatRaster 
    ## dimensions  : 602, 662, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 343290, 349910, 293990, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3134part6.tif 
    ## name        : NDVI 
    ## 
    ## [[623]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 410, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 349900, 354000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3141part0.tif 
    ## name        : NDVI 
    ## 
    ## [[624]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 353000, 358010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3141part1.tif 
    ## name        : NDVI 
    ## 
    ## [[625]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 356990, 362010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3141part2.tif 
    ## name        : NDVI 
    ## 
    ## [[626]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 360990, 366000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3141part3.tif 
    ## name        : NDVI 
    ## 
    ## [[627]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 365000, 370000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3141part4.tif 
    ## name        : NDVI 
    ## 
    ## [[628]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 369000, 374010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3141part5.tif 
    ## name        : NDVI 
    ## 
    ## [[629]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 192, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 372990, 374910, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3141part6.tif 
    ## name        : NDVI 
    ## 
    ## [[630]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 510, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 374900, 380000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3142part0.tif 
    ## name        : NDVI 
    ## 
    ## [[631]]
    ## class       : SpatRaster 
    ## dimensions  : 2061, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 378990, 385010, 254400, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3142part1.tif 
    ## name        : NDVI 
    ## 
    ## [[632]]
    ## class       : SpatRaster 
    ## dimensions  : 2171, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 383990, 390010, 253300, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3142part2.tif 
    ## name        : NDVI 
    ## 
    ## [[633]]
    ## class       : SpatRaster 
    ## dimensions  : 2312, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 388990, 394010, 251890, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3142part3.tif 
    ## name        : NDVI 
    ## 
    ## [[634]]
    ## class       : SpatRaster 
    ## dimensions  : 2392, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 392990, 399010, 251090, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3142part4.tif 
    ## name        : NDVI 
    ## 
    ## [[635]]
    ## class       : SpatRaster 
    ## dimensions  : 2321, 201, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 397990, 4e+05, 251800, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3142part5.tif 
    ## name        : NDVI 
    ## 
    ## [[636]]
    ## class       : SpatRaster 
    ## dimensions  : 2360, 511, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 349900, 355010, 275000, 298600  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3143part0.tif 
    ## name        : NDVI 
    ## 
    ## [[637]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 353990, 359000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3143part1.tif 
    ## name        : NDVI 
    ## 
    ## [[638]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 358000, 363000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3143part2.tif 
    ## name        : NDVI 
    ## 
    ## [[639]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 362000, 367010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3143part3.tif 
    ## name        : NDVI 
    ## 
    ## [[640]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 365990, 371010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3143part4.tif 
    ## name        : NDVI 
    ## 
    ## [[641]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 492, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 369990, 374910, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3143part5.tif 
    ## name        : NDVI 
    ## 
    ## [[642]]
    ## class       : SpatRaster 
    ## dimensions  : 912, 2501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 349900, 374910, 290890, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3143part6.tif 
    ## name        : NDVI 
    ## 
    ## [[643]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 510, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 374900, 380000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3144part0.tif 
    ## name        : NDVI 
    ## 
    ## [[644]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 378990, 384000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3144part1.tif 
    ## name        : NDVI 
    ## 
    ## [[645]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 383000, 388010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3144part2.tif 
    ## name        : NDVI 
    ## 
    ## [[646]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 386990, 392010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3144part3.tif 
    ## name        : NDVI 
    ## 
    ## [[647]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 390990, 396000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3144part4.tif 
    ## name        : NDVI 
    ## 
    ## [[648]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 395000, 4e+05, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3144part5.tif 
    ## name        : NDVI 
    ## 
    ## [[649]]
    ## class       : SpatRaster 
    ## dimensions  : 2402, 2510, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 374900, 4e+05, 275990, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3144part6.tif 
    ## name        : NDVI 
    ## 
    ## [[650]]
    ## class       : SpatRaster 
    ## dimensions  : 971, 2502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 424990, 450010, 240290, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3214part0.tif 
    ## name        : NDVI 
    ## 
    ## [[651]]
    ## class       : SpatRaster 
    ## dimensions  : 60, 102, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 448990, 450010, 249400, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3214part1.tif 
    ## name        : NDVI 
    ## 
    ## [[652]]
    ## class       : SpatRaster 
    ## dimensions  : 690, 2490, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 450000, 474900, 243100, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3223part0.tif 
    ## name        : NDVI 
    ## 
    ## [[653]]
    ## class       : SpatRaster 
    ## dimensions  : 830, 2112, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 474890, 496010, 241700, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3224part0.tif 
    ## name        : NDVI 
    ## 
    ## [[654]]
    ## class       : SpatRaster 
    ## dimensions  : 731, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 494990, 500010, 242690, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3224part1.tif 
    ## name        : NDVI 
    ## 
    ## [[655]]
    ## class       : SpatRaster 
    ## dimensions  : 2321, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 399990, 405000, 251800, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3231part0.tif 
    ## name        : NDVI 
    ## 
    ## [[656]]
    ## class       : SpatRaster 
    ## dimensions  : 2431, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 404000, 409000, 250700, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3231part1.tif 
    ## name        : NDVI 
    ## 
    ## [[657]]
    ## class       : SpatRaster 
    ## dimensions  : 2532, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408000, 413010, 249690, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3231part2.tif 
    ## name        : NDVI 
    ## 
    ## [[658]]
    ## class       : SpatRaster 
    ## dimensions  : 2551, 602, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 411990, 418010, 246300, 271810  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3231part3.tif 
    ## name        : NDVI 
    ## 
    ## [[659]]
    ## class       : SpatRaster 
    ## dimensions  : 2730, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 417000, 422010, 245900, 273200  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3231part4.tif 
    ## name        : NDVI 
    ## 
    ## [[660]]
    ## class       : SpatRaster 
    ## dimensions  : 2752, 1440, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 410600, 425000, 247490, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3231part5.tif 
    ## name        : NDVI 
    ## 
    ## [[661]]
    ## class       : SpatRaster 
    ## dimensions  : 571, 890, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 413000, 421900, 269300, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3231part6.tif 
    ## name        : NDVI 
    ## 
    ## [[662]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 511, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 424890, 430000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3232part0.tif 
    ## name        : NDVI 
    ## 
    ## [[663]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 600, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 429000, 435000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3232part1.tif 
    ## name        : NDVI 
    ## 
    ## [[664]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 434000, 439000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3232part2.tif 
    ## name        : NDVI 
    ## 
    ## [[665]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 438000, 443010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3232part3.tif 
    ## name        : NDVI 
    ## 
    ## [[666]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 441990, 447010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3232part4.tif 
    ## name        : NDVI 
    ## 
    ## [[667]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 2512, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 424890, 450010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3232part5.tif 
    ## name        : NDVI 
    ## 
    ## [[668]]
    ## class       : SpatRaster 
    ## dimensions  : 602, 522, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 425990, 431210, 249990, 256010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3232part6.tif 
    ## name        : NDVI 
    ## 
    ## [[669]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 399990, 405000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3233part0.tif 
    ## name        : NDVI 
    ## 
    ## [[670]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 404000, 409000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3233part1.tif 
    ## name        : NDVI 
    ## 
    ## [[671]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 408000, 413010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3233part2.tif 
    ## name        : NDVI 
    ## 
    ## [[672]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 411990, 417010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3233part3.tif 
    ## name        : NDVI 
    ## 
    ## [[673]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 415990, 421000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3233part4.tif 
    ## name        : NDVI 
    ## 
    ## [[674]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 490, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 420000, 424900, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3233part5.tif 
    ## name        : NDVI 
    ## 
    ## [[675]]
    ## class       : SpatRaster 
    ## dimensions  : 1502, 2491, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 399990, 424900, 284990, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3233part6.tif 
    ## name        : NDVI 
    ## 
    ## [[676]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 412, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 424890, 429010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3234part0.tif 
    ## name        : NDVI 
    ## 
    ## [[677]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 427990, 433000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3234part1.tif 
    ## name        : NDVI 
    ## 
    ## [[678]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 431990, 437000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3234part2.tif 
    ## name        : NDVI 
    ## 
    ## [[679]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 436000, 441010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3234part3.tif 
    ## name        : NDVI 
    ## 
    ## [[680]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 440000, 445010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3234part4.tif 
    ## name        : NDVI 
    ## 
    ## [[681]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 443990, 449000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3234part5.tif 
    ## name        : NDVI 
    ## 
    ## [[682]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 201, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 448000, 450010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3234part6.tif 
    ## name        : NDVI 
    ## 
    ## [[683]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 450000, 454010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3241part0.tif 
    ## name        : NDVI 
    ## 
    ## [[684]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 454000, 458000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3241part1.tif 
    ## name        : NDVI 
    ## 
    ## [[685]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 457990, 462000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3241part2.tif 
    ## name        : NDVI 
    ## 
    ## [[686]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 461990, 466010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3241part3.tif 
    ## name        : NDVI 
    ## 
    ## [[687]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 466000, 470010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3241part4.tif 
    ## name        : NDVI 
    ## 
    ## [[688]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 470000, 474000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3241part5.tif 
    ## name        : NDVI 
    ## 
    ## [[689]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 91, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 473990, 474900, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3241part6.tif 
    ## name        : NDVI 
    ## 
    ## [[690]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 411, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 474890, 479000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3242part0.tif 
    ## name        : NDVI 
    ## 
    ## [[691]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 477990, 483000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3242part1.tif 
    ## name        : NDVI 
    ## 
    ## [[692]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 482000, 487010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3242part2.tif 
    ## name        : NDVI 
    ## 
    ## [[693]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 485990, 491010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3242part3.tif 
    ## name        : NDVI 
    ## 
    ## [[694]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 489990, 495000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3242part4.tif 
    ## name        : NDVI 
    ## 
    ## [[695]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 494000, 499000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3242part5.tif 
    ## name        : NDVI 
    ## 
    ## [[696]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 450000, 454010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3243part0.tif 
    ## name        : NDVI 
    ## 
    ## [[697]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 454000, 458000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3243part1.tif 
    ## name        : NDVI 
    ## 
    ## [[698]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 457990, 462000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3243part2.tif 
    ## name        : NDVI 
    ## 
    ## [[699]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 461990, 466010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3243part3.tif 
    ## name        : NDVI 
    ## 
    ## [[700]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 466000, 470010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3243part4.tif 
    ## name        : NDVI 
    ## 
    ## [[701]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 470000, 474000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3243part5.tif 
    ## name        : NDVI 
    ## 
    ## [[702]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 91, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 473990, 474900, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3243part6.tif 
    ## name        : NDVI 
    ## 
    ## [[703]]
    ## class       : SpatRaster 
    ## dimensions  : 2151, 512, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 474890, 480010, 278500, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3244part0.tif 
    ## name        : NDVI 
    ## 
    ## [[704]]
    ## class       : SpatRaster 
    ## dimensions  : 2332, 601, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 478990, 485000, 276690, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3244part1.tif 
    ## name        : NDVI 
    ## 
    ## [[705]]
    ## class       : SpatRaster 
    ## dimensions  : 2481, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 484000, 489010, 275200, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3244part2.tif 
    ## name        : NDVI 
    ## 
    ## [[706]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 487990, 493010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3244part3.tif 
    ## name        : NDVI 
    ## 
    ## [[707]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 491990, 497000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3244part4.tif 
    ## name        : NDVI 
    ## 
    ## [[708]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 2512, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 474890, 500010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3244part5.tif 
    ## name        : NDVI 
    ## 
    ## [[709]]
    ## class       : SpatRaster 
    ## dimensions  : 1631, 900, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 5e+05, 509000, 233690, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3313part0.tif 
    ## name        : NDVI 
    ## 
    ## [[710]]
    ## class       : SpatRaster 
    ## dimensions  : 1601, 701, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 507990, 515000, 233990, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3313part1.tif 
    ## name        : NDVI 
    ## 
    ## [[711]]
    ## class       : SpatRaster 
    ## dimensions  : 1361, 1001, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 514000, 524010, 236390, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3313part2.tif 
    ## name        : NDVI 
    ## 
    ## [[712]]
    ## class       : SpatRaster 
    ## dimensions  : 1510, 710, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 524900, 532000, 234900, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3314part0.tif 
    ## name        : NDVI 
    ## 
    ## [[713]]
    ## class       : SpatRaster 
    ## dimensions  : 1260, 1261, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 530990, 543600, 237400, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3314part1.tif 
    ## name        : NDVI 
    ## 
    ## [[714]]
    ## class       : SpatRaster 
    ## dimensions  : 2281, 1040, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 557600, 568000, 227190, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3323part0.tif 
    ## name        : NDVI 
    ## 
    ## [[715]]
    ## class       : SpatRaster 
    ## dimensions  : 2400, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 567000, 572010, 226000, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3323part1.tif 
    ## name        : NDVI 
    ## 
    ## [[716]]
    ## class       : SpatRaster 
    ## dimensions  : 2400, 392, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 570990, 574910, 226000, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3323part2.tif 
    ## name        : NDVI 
    ## 
    ## [[717]]
    ## class       : SpatRaster 
    ## dimensions  : 2591, 510, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 574900, 580000, 224090, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3324part0.tif 
    ## name        : NDVI 
    ## 
    ## [[718]]
    ## class       : SpatRaster 
    ## dimensions  : 2710, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 579000, 584000, 222900, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3324part1.tif 
    ## name        : NDVI 
    ## 
    ## [[719]]
    ## class       : SpatRaster 
    ## dimensions  : 2611, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 583000, 587000, 223890, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3324part2.tif 
    ## name        : NDVI 
    ## 
    ## [[720]]
    ## class       : SpatRaster 
    ## dimensions  : 2690, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 586000, 591000, 223100, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3324part3.tif 
    ## name        : NDVI 
    ## 
    ## [[721]]
    ## class       : SpatRaster 
    ## dimensions  : 2710, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 590000, 595010, 222900, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3324part4.tif 
    ## name        : NDVI 
    ## 
    ## [[722]]
    ## class       : SpatRaster 
    ## dimensions  : 2770, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 593990, 599010, 222300, 250000  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3324part5.tif 
    ## name        : NDVI 
    ## 
    ## [[723]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 5e+05, 505010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3331part0.tif 
    ## name        : NDVI 
    ## 
    ## [[724]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 503990, 509000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3331part1.tif 
    ## name        : NDVI 
    ## 
    ## [[725]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 507990, 513000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3331part2.tif 
    ## name        : NDVI 
    ## 
    ## [[726]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 512000, 517010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3331part3.tif 
    ## name        : NDVI 
    ## 
    ## [[727]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 515990, 521010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3331part4.tif 
    ## name        : NDVI 
    ## 
    ## [[728]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 492, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 519990, 524910, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3331part5.tif 
    ## name        : NDVI 
    ## 
    ## [[729]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 410, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 524900, 529000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3332part0.tif 
    ## name        : NDVI 
    ## 
    ## [[730]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 528000, 533010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3332part1.tif 
    ## name        : NDVI 
    ## 
    ## [[731]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 502, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 531990, 537010, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3332part2.tif 
    ## name        : NDVI 
    ## 
    ## [[732]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 501, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 535990, 541000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3332part3.tif 
    ## name        : NDVI 
    ## 
    ## [[733]]
    ## class       : SpatRaster 
    ## dimensions  : 2502, 500, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 540000, 545000, 249990, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3332part4.tif 
    ## name        : NDVI 
    ## 
    ## [[734]]
    ## class       : SpatRaster 
    ## dimensions  : 2422, 600, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 544000, 550000, 250790, 275010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3332part5.tif 
    ## name        : NDVI 
    ## 
    ## [[735]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 400, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 5e+05, 504000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3333part0.tif 
    ## name        : NDVI 
    ## 
    ## [[736]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 507990, 512010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3333part2.tif 
    ## name        : NDVI 
    ## 
    ## [[737]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 512000, 516010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3333part3.tif 
    ## name        : NDVI 
    ## 
    ## [[738]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 401, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 515990, 520000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3333part4.tif 
    ## name        : NDVI 
    ## 
    ## [[739]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 402, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 519990, 524010, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3333part5.tif 
    ## name        : NDVI 
    ## 
    ## [[740]]
    ## class       : SpatRaster 
    ## dimensions  : 2501, 410, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 524900, 529000, 275000, 300010  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : NDVI3334part0.tif 
    ## name        : NDVI

``` r
NDVI_LV <- do.call(merge, rasters)
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
writeRaster(NDVI_LV, "NDVI_LV.tif", overwrite = TRUE)
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
plot(NDVI_LV)
```

![](Uzd08_Ozols_files/figure-gfm/uzdevums3-1.png)<!-- -->
