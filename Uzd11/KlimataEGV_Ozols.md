KlimataEGV_Ozols
================
Janis Ozols

# Klimata EGV

11. uzdevumam sagatavoju klimata egv ar nosaukumu:
    Climate_WinterMinTemp_cell.tif, sagatavots GEE, sagatavošanas
    apraksts: iegūta vidējā ikdienas minimālā temperatūra ziemas
    mēnešos, no decembra līdz februārim, katra gada ziemas sezonai, pēc
    tam aprēķināta vidējā minimālā temperatūra temperatūra no katra gada
    ziemas sezonas perioda vidējās minimālās temperatūras no 2016 līdz
    2024 gadam. Skripts datu lejupielādei no GEE pieejams
    [šeit](https://code.earthengine.google.co.in/cfe636713bb982ee89232aee52fd20ce?asset=projects%2Fee-rr11031%2Fassets%2Ftikls100_UNIONpolygon)
    Kad fails ir lejupielādēts, pārejam pie tā apstrādes, sagatavojam r
    vidi darbam

``` r
if (!require("terra")) install.packages("terra");library(terra)
```

    ## Loading required package: terra

    ## terra 1.8.24

``` r
if (!require("whitebox")) install.packages("whitebox");library(whitebox)
```

    ## Loading required package: whitebox

    ## Warning: package 'whitebox' was built under R version 4.4.3

``` r
whitebox::install_whitebox() #ar install packages meta ārā kļūdu, ka nav uzlikts
```

    ## Performing one-time download of WhiteboxTools binary from
    ##   https://www.whiteboxgeo.com/WBT_Windows/WhiteboxTools_win_amd64.zip 
    ## (This could take a few minutes, please be patient...)
    ## WhiteboxTools binary is located here:  C:/Users/Ozols/AppData/Roaming/R/data/R/whitebox/WBT/whitebox_tools.exe 
    ## You can now start using whitebox
    ##     library(whitebox)
    ##     wbt_version()

Ielādējam lejupielādēto failu un references rastru un resamplojam

``` r
ref100 <- rast("../Uzd03/LV100m_10km.tif")
klimats <- rast("ERA5L_Tmin_WinterMean_2016_2024.tif")
klimats <- project(klimats, "EPSG:3059")
plot(klimats)
```

![](KlimataEGV_Ozols_files/figure-gfm/resample-1.png)<!-- -->

``` r
klimats_resample <- resample(klimats, ref100, filename = "vid_aukst_temp.tif", overwrite = TRUE)
plot(klimats_resample)
```

![](KlimataEGV_Ozols_files/figure-gfm/resample-2.png)<!-- --> Rastrs ir
resamplots, problēma, kas palikusi, ERA5_LAND nesatur datus par jūru un
tā, kā flīzes ir 9km, tad rastrā ir lieli robi, kurus nepieciešams
aizpildīt.

``` r
robi <- is.na(klimats_resample) & !is.na(ref100)
pilns <- ifel(!robi, 1, NA)
platums <- distance(pilns)
max_attalums <- terra::global(platums, fun = "max", na.rm = TRUE)
print(max_attalums) 
```

    ##                            max
    ## Tmin_Winter_2016_2024 4808.326

``` r
#maksimālā vērtība ir 4808, tie ir metri, pārejot uz 100m2, tie ir 48,1m2, tātad 49m2.
wbt_fill_missing_data(
  i = "vid_aukst_temp.tif",
  output = "klimats_aukstums.tif",
  filter = 98,  
  weight = 1,
  no_edges = FALSE
)
aukstums <- rast("klimats_aukstums.tif")
plot(aukstums)
```

![](KlimataEGV_Ozols_files/figure-gfm/aizpildam-1.png)<!-- --> Tālāk
jāpanāk, lai pikseļu malas nav redzamas, izmantošu funkciju focal no
pakotnes terra. Tā kā oriģinālais datu šūnu izmērs ir 9km, kas
resamplots uz 100m, tas nozīmē, ka oriģinālā šūna satur 90 100m šūnas,
līdz ar to funkcijā w - izmantosim kā 91.

``` r
smooth_aukstums <- focal(aukstums, w=91, fun = "mean")
plot(smooth_aukstums)
```

![](KlimataEGV_Ozols_files/figure-gfm/focal-1.png)<!-- -->

``` r
raw_aukstums <- mask(smooth_aukstums, ref100, filename = "Climate_WinterMinTemp_cell_RAW.tif", overwrite=TRUE)
plot(raw_aukstums)
```

![](KlimataEGV_Ozols_files/figure-gfm/focal-2.png)<!-- -->

``` r
videjais <- global(raw_aukstums,fun = "mean",na.rm = TRUE)
centrets <- aukstums-videjais[,1]
standartnovirze <- terra::global(centrets,fun = "rms",na.rm = TRUE)
merogots <- centrets/standartnovirze[,1]
writeRaster(merogots, filename = "Climate_WinterMinTemp_cell_scaled.tif", overwrite=TRUE)
```
