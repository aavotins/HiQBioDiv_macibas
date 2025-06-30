Uzd08_Rubene
================
Betija Rubene
2025-06-24

# Astotais uzdevums: GEE

Uzdevuma izpildi veicu pārlūkā ar JavaScript, izmantotās komandrindas
pieejamas
[šeit](https://code.earthengine.google.com/b0d7665c37b21f326db9dfbaa1fbbaa2)

Pēc 2.3. uzdevuma rezultējošā attēla lejupielādes ar to vēl drusku
jāpastrādā:

Lejupielādējiet 2.3. rezultātu, no tā izveidojiet vienu GeoTIFF slāni,
kura aptvertā telpa un pikseļu izvietojums atbilst projekta Zenodo
repozitorijā dotajam rastra slānim ar 10m izšķirtspēju. Ar starpkvartiļu
amplitūdu raksturojiet NDVI variabilitāti ik 100 m šūnā kā rastra slāni,
kura telpiskais pārklājums un pikseļu izvietojums atbilst projekta
Zenodo repozitorijā dotajam rastra slānim ar 100m izšķirtspēju.

``` r
library(sp)
library(terra)
```

    ## terra 1.8.10

``` r
ndvi10 <- rast("NDVI_LV.tif") # 2.3. rezultāts
plot(ndvi10)
```

![](Uzd08_Rubene_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

``` r
ref100 <- rast("../ref_rastri/LV100m_10km.tif") # reference

ndvi10_iqr <- aggregate(ndvi10, fact = 10, fun = IQR, na.rm = TRUE)
ref100_crop <- crop(ref100,ndvi10_iqr) # šis būtu jāizlaiž, ja man būtu pilnais attēls 
# (vairāk par to, kāpēc tā sanāca - komentāros pārlūkā)

compareGeom(ndvi10_iqr, ref100_crop, stopOnError = FALSE) # vajag pielāgot references rastram
```

    ## [1] FALSE

``` r
ndvi1_iqr_rez <- resample(ndvi10_iqr, ref100_crop, method = "near", filename = "NDVI_IQR.tif") # pieglabāju
```
