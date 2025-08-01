Uzd07_Sakele
================
Vita
2025-07-29

``` r
if (!require("sf")) install.packages("sf")
if (!require("arrow")) install.packages("arrow")
if (!require("sfarrow")) install.packages("sfarrow")
if (!require("terra")) install.packages("terra")
if (!require("dplyr")) install.packages("dplyr")
if (!require("raster")) install.packages("raster")
if (!require("fasterize")) install.packages("fasterize")
if (!require("httr")) install.packages("httr")
if (!require("ggplot2")) install.packages("ggplot2")
if (!require("exactextractr")) install.packages("exactextractr") 
library(exactextractr)
if (!require("microbenchmark")) install.packages("microbenchmark")
```

# Datu savākšana

``` r
# no 5. uzdevuma - kartes lapas

sheets_list_paths <- list.files(
  path = "../Uzd05/data/sheet_prqs",
  pattern = "^sheet_\\d{4}\\.parquet$", 
  full.names = TRUE)

sheets_list <- lapply(sheets_list_paths, st_read_parquet)
all_sheets <- do.call(rbind, sheets_list)

rm(sheets_list_paths, sheets_list)
```

``` r
# Sauszeme

if (!dir.exists("data")) {
  dir.create("data")
}

url= "https://zenodo.org/records/14277114/files/pts100_sauzeme.parquet?download=1"
destfile = "data/pts100_sauszeme.parquet"
download.file(url=url, destfile = destfile, mode='wb')

pts100_sauszeme<-st_read_parquet(destfile)
pts100_sauszeme <- st_transform(pts100_sauszeme, st_crs(all_sheets))

# piegriežu 
pts100_cropped <- st_filter(pts100_sauszeme, all_sheets, .predicate = st_within)

rm(url, destfile, pts100_sauszeme)
```

``` r
# mežu klašu rastrs
merged_forest_rasters<-raster("../Uzd06/data/forest_classes/forest_classes_merged.tif")

#references rastrs
#raster_100m_10km <- rast("../Uzd04/data/reference_raster/LV100m_10km.tif")
raster_100m_10km <- raster("../Uzd04/data/reference_raster/LV100m_10km.tif")

raster_100m_10km_cropped <- crop(raster_100m_10km, all_sheets) #darba teritorijas reference

rm(raster_100m_10km)
```

## 1. Klašu īpatsvars 500 m buferzonās

``` r
pts100_buffer <- st_buffer(pts100_cropped, dist = 500)
```

``` r
#ar plot() nemaz neizdevās uzzīmēt

ggplot() +
  geom_sf(data = pts100_buffer, fill = "lightblue", color = NA) +
  geom_sf(data = pts100_cropped, color = "red", size = 0.5) +
  theme_minimal()
```

![](Uzd07_Sakele_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

``` r
rm(pts100_cropped)
```

``` r
#īpatsvaru aprēķins
fractions <- exact_extract(merged_forest_rasters, pts100_buffer, fun = "frac", append_cols = "id")
fractions <- left_join(pts100_buffer, fractions, by = "id")
```

``` r
forest_classses <- grep("^frac_", names(fractions), value = TRUE)

dir.create("fract_rasters_1")

for (fc in forest_classses) {
  rastrs <- fasterize(fractions, raster_100m_10km_cropped, field = fc, fun="max")
  # ja izmantotu rasterize, tad lietotu fun = 'mean'. Bet rasterize ir lēnāks.
  writeRaster(rastrs, paste0("fract_rasters_1/", fc, "_fract.tif"), overwrite = TRUE)
}
```

``` r
rm(rastrs, fc, forest_classses, merged_forest_rasters, all_sheets, pts100_buffer, raster_100m_10km_cropped, fractions)
```

## 2.

``` r
#Datu savākšana

# 1km kvadrāti

url= "https://zenodo.org/records/14277114/files/tikls1km_sauzeme.parquet?download=1"
destfile = "data/tikls1km_sauszeme.parquet"
download.file(url=url, destfile = destfile, mode='wb')

grid_1km<-st_read_parquet(destfile)

sq_id <- c("1kmX501Y263","1kmX502Y263", "1kmX503Y263", "1kmX504Y263", "1kmX505Y263",
           "1kmX501Y262","1kmX502Y262", "1kmX503Y262", "1kmX504Y262", "1kmX505Y262")

squares <- grid_1km %>% filter(ID1km %in% sq_id)

raster_100m_10km <- raster("../Uzd04/data/reference_raster/LV100m_10km.tif")
raster_100m_10km_sq <- crop(raster_100m_10km, squares)

rm (destfile, url, sq_id, grid_1km, raster_100_10km)



#Buferi
# 100m tīkla centri

pts100_sauszeme <- st_read_parquet("data/pts100_sauszeme.parquet")
pts100_sauszeme <- st_transform(pts100_sauszeme, st_crs(squares))
pts100m_sauszeme_sq <- st_filter(pts100_sauszeme, squares, .predicate = st_within)
pts100m_sauszeme_sq_buffers <- st_buffer(pts100m_sauszeme_sq, dist = 3000)


# 300m šūnu centri

url= "https://zenodo.org/records/14277114/files/pts300_sauzeme.parquet?download=1"
destfile= "data/pts300_sauzeme.parquet"
download.file(url=url, destfile = destfile, mode='wb')

pts300_sauszeme <- st_read_parquet(destfile)
pts300_sauszeme <- st_transform(pts300_sauszeme, st_crs(squares))
pts300m_sauszeme_sq <- st_filter(pts300_sauszeme, squares, .predicate = st_within)
pts300m_sauszeme_sq_buffers <- st_buffer(pts300m_sauszeme_sq, dist = 3000)


#LAD dati no 3. uzdevuma

LAD10m <- raster("../Uzd03/data/LAD_raster_10m.tif")
LAD100m <- raster("../Uzd03/data/LAD_raster_100m_agr_bin.tif")

rm(url, destfile, pts100_sauszeme, pts100m_sauszeme_sq, pts300_sauszeme, pts300m_sauszeme_sq,
   raster_100m_10km, squares)
```

``` r
plot(LAD10m, main="LAD 10m")
```

![](Uzd07_Sakele_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

``` r
plot(LAD100m, main="LAD 100m")
```

![](Uzd07_Sakele_files/figure-gfm/unnamed-chunk-11-2.png)<!-- -->

## 2.1

Ik 100 m tīkla centram, kā aprakstāmo rastru izmantojot iepriekšējos
uzdevumos sagatavoto lauku klātbūtni 10 m šūnā;

``` r
time_2.1 <- microbenchmark(exact_extract(LAD10m, pts100m_sauszeme_sq_buffers, fun = "frac"), 
                           times = 10)
mean_time_2.1s <- round(mean(time_2.1$time) / 1e9, 2) # konvertācija uz sekundēm
```

``` r
fract_2.1 <- exact_extract(LAD10m, pts100m_sauszeme_sq_buffers, fun = "frac", append_cols = "id")
fract_2.1 <- left_join(pts100m_sauszeme_sq_buffers, fract_2.1, by = "id")

raster_2.1 <- fasterize(fract_2.1, raster_100m_10km_sq, field = "frac_1", fun = "max")

dir.create("fract_rasters_2/")
writeRaster(raster_2.1, "fract_rasters_2/frac_2.1.tif", overwrite = TRUE)
```

## 2.2

Ik 100 m tīkla centram, kā aprakstāmo rastru izmantojot iepriekšējos
uzdevumos sagatavoto lauku īpatsvaru 100 m šūnā

``` r
time_2.2 <- microbenchmark(exact_extract(LAD100m, pts100m_sauszeme_sq_buffers, fun = "frac"), 
                           times = 10)
mean_time_2.2s <- round(mean(time_2.2$time) / 1e9, 2) # konvertācija uz sekundēm
```

``` r
fract_2.2 <- exact_extract(LAD100m, pts100m_sauszeme_sq_buffers, fun = "frac", append_cols = "id")
fract_2.2 <- left_join(pts100m_sauszeme_sq_buffers, fract_2.2, by = "id")

raster_2.2 <- fasterize(fract_2.2, raster_100m_10km_sq, field = "frac_1", fun = "max")

writeRaster(raster_2.2, "fract_rasters_2/frac_2.2.tif", overwrite = TRUE)
```

## 2.3.

Ik 300 m tīkla centram, kā aprakstāmo rastru izmantojot iepriekšējos
uzdevumos sagatavoto lauku klātbūtni 10 m šūnā.

``` r
time_2.3 <- microbenchmark(exact_extract(LAD10m, pts300m_sauszeme_sq_buffers, fun = "frac"), 
                           times = 10)
mean_time_2.3s <- round(mean(time_2.3$time) / 1e9, 2) # konvertācija uz sekundēm
```

``` r
fract_2.3 <- exact_extract(LAD10m, pts300m_sauszeme_sq_buffers, fun = "frac", append_cols = "rinda300")
fract_2.3 <- left_join(pts300m_sauszeme_sq_buffers, fract_2.3, by = "rinda300")

raster_2.3 <- fasterize(fract_2.3, raster_100m_10km_sq, field = "frac_1", fun = "max")

writeRaster(raster_2.3, "fract_rasters_2/frac_2.3.tif", overwrite = TRUE)
```

## 2.4.

Ik 300 m tīkla centram, kā aprakstāmo rastru izmantojot iepriekšējos
uzdevumos sagatavoto lauku klātbūtni 100 m šūnā.

``` r
time_2.4 <- microbenchmark(exact_extract(LAD100m, pts300m_sauszeme_sq_buffers, fun = "frac"), 
                           times = 10)
mean_time_2.4s <- round(mean(time_2.4$time) / 1e9, 2) # konvertācija uz sekundēm
```

``` r
fract_2.4 <- exact_extract(LAD100m, pts300m_sauszeme_sq_buffers, fun = "frac", 
                           append_cols = "rinda300")
fract_2.4 <- left_join(pts300m_sauszeme_sq_buffers, fract_2.4, by = "rinda300")

raster_2.4 <- fasterize(fract_2.4, raster_100m_10km_sq, field = "frac_1", fun = "max")

writeRaster(raster_2.4, "fract_rasters_2/frac_2.4.tif", overwrite = TRUE)
```

``` r
#rezultāts
res <- c("100 - 10", "100 - 100", "300- 10", "300 - 100")
t <- c(mean_time_2.1s, mean_time_2.2s, mean_time_2.3s, mean_time_2.4s)
fr <- c(mean(fract_2.1$frac_1), mean(fract_2.2$frac_1), mean(fract_2.3$frac_1),
      mean(fract_2.4$frac_1))

result<-data.frame(
  resolution = res,
  time = t,
  fraction = fr)

result
```

    ##   resolution  time   fraction
    ## 1   100 - 10 30.09 0.05944666
    ## 2  100 - 100  1.09 0.11237840
    ## 3    300- 10  2.81 0.05817662
    ## 4  300 - 100  0.33 0.10995769

Kā sagaidāms, vislielākais laika patēriņš ir pirmajā variantā (100 m
tīkls ar 10 m šūnu), ja šūna ir 100m, tad patēriņš samazinās ~10 reizes.
Līdzīgs 10 reižu laika samazinājums ir arī starp 300 m tīklu ar 10 m un
100 m šūnām.

Lauku īpatsvari vienādas izmēra šūnās praktiski sakrīt. Neesmu
pārliecināta, ka spēju novērtēt, cik būtisks ir precizitāts zaudējums,
samazinot izšķirtspēju. Tomēr ir redzams, ka tāds notiek.

``` r
par(mfrow = c(2, 2))  # 2 rindas, 2 kolonnas

plot(raster_2.1, main = "100 - 10")
plot(raster_2.2, main = "100 - 100")
plot(raster_2.3, main = "300 - 10")
plot(raster_2.4, main = "300 - 100")
```

![](Uzd07_Sakele_files/figure-gfm/unnamed-chunk-21-1.png)<!-- -->

``` r
par(mfrow = c(1, 1))  # atjauno noklusēto
```

``` r
rm(list = ls())
```
