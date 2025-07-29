KlimataEGV_Rubene
================
Betija Rubene
2025-07-29

# Klimata EGV

Man piešķirts klimata mainīgais surface_net_thermal_radiation

Skripts datu lejupielādei no GEE pieejams
[šeit](https://code.earthengine.google.com/cc8241d659ba04fb420232da50591d37)

Fails pēc saglabāšanas drive lejupielādēts atbilstošajā direktorijā
(Uzd11)

# Eksportētā faila apstrāde

``` r
# pakotnes
library(terra)
```

    ## terra 1.8.60

``` r
library(whitebox)
```

## Ekstrapolācija un interpolācija

Faili un priekšdarbi

``` r
# 100m reference
ref100m <- rast("../ref_rastri/LV100m_10km.tif")

# thermal radiation
ievade <- rast("thermal_rad_2016-2024.tif")
plot(ievade)
```

![](KlimataEGV_Rubene_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

``` r
# man nav līdz galam saprotams, kādēļ vērtības ir negatīvas, bet lai nu būtu, tādi tie dati ir.

th_rad <- resample(ievade, ref100m, method = "bilinear", filename = "thermal_rad.tif", overwrite = TRUE) # sakritībai
plot(th_rad)
```

![](KlimataEGV_Rubene_files/figure-gfm/unnamed-chunk-2-2.png)<!-- -->

Nepieciešams aizpildīt gar jūru esošos caurumus, kuros sauszemes
teritorijā iztrūkst dati. Tam izmantošu wbt_fill_missing_data no
pakotnes whitebox Vispirms veikšu tukšumu inventarizāciju, lai zinātu,
cik tālu nepieciešams ekstrapolēt

``` r
tuksumi <- is.na(th_rad) & !is.na(ref100m)
tuksumi_count <- global(tuksumi, fun="sum", na.rm=TRUE)
tuksumi_count # 70118
```

    ##                                     sum
    ## surface_net_thermal_radiation_sum 70118

``` r
plot(tuksumi)
```

![](KlimataEGV_Rubene_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

``` r
# cik plata ir platākā vieta?
aizpilditie <- ifel(!tuksumi, 1, NA)
platums <- distance(aizpilditie)
plot(platums, main = "Attālums līdz tuvākajai ne-NA vērtībai")
```

![](KlimataEGV_Rubene_files/figure-gfm/unnamed-chunk-3-2.png)<!-- -->

``` r
max_attalums <- terra::global(platums, fun = "max", na.rm = TRUE)
print(max_attalums) # lielākais platums ir 3551
```

    ##                                        max
    ## surface_net_thermal_radiation_sum 3551.056

``` r
# Bet šis ir attālums (metri) - izvēlētā funkcija prasa pikseļu (100 x 100 m) skaitu
# tātad (apaļojot uz augšu) 36 * 2 = 72 (x2 jo max vērtība ir viduspunkts)

# tukšumu aizpildīšana
wbt_fill_missing_data(
  i = "./thermal_rad.tif",
  output = "./thermal_rad_filled.tif",
  filter = 72, # platumu aprēķina rezultāts    
  weight = 1,
  no_edges = FALSE # lai notiktu ekstrapolācija gar malām
)

result <- rast("./thermal_rad_filled.tif")
plot(result)
```

![](KlimataEGV_Rubene_files/figure-gfm/unnamed-chunk-3-3.png)<!-- -->

Otrs uzdevums ir panākt, lai oriģinālās pikseļu malas nebūtu redzamas.
Tam izmantošu funkciju focal no pakotnes terra

``` r
smooth_th_rad <- focal(result, w=99, fun = "mean") # window izvēlējos daļēji arbitrāri, paturot prātā, ka 
# oriģinālo datu izšķirtspēja ir  ~ 9 km
plot(smooth_th_rad)
```

![](KlimataEGV_Rubene_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

``` r
# izmēģinot uz pusi mazāku w, pikseļu malas, protams, vairs nav redzamas, tomēr labi varēja redzēt, kur tās bijušas
# bet principā šis ir diskutējams, kādu w vērtību izvēlēties. Vai vispār izmantot exact extract, bet tas gan jau būtu
# ilgāk un nez vai tik būtiski ietekmētu rezultātu?

# lieko malu apgriešana (kas neietilpst 100m references rastrā)
dir.create("./EGV")
```

    ## Warning in dir.create("./EGV"): '.\EGV' already exists

``` r
th_rad_raw <- mask(smooth_th_rad, ref100m, filename = "./EGV/Climate_ThermalRadiation_cell_RAW.tif", overwrite=TRUE)
plot(th_rad_raw)
```

![](KlimataEGV_Rubene_files/figure-gfm/unnamed-chunk-4-2.png)<!-- -->

## Centrēšana un mērogošana

``` r
videjais <- global(th_rad_raw,fun = "mean",na.rm = TRUE)
centrets <- th_rad_raw-videjais[,1]
standartnovirze <- terra::global(centrets,fun = "rms",na.rm = TRUE)
merogots <- centrets/standartnovirze[,1]
videjais # -158061153
```

    ##                  mean
    ## focal_mean -158061153

``` r
standartnovirze # 5986845   
```

    ##                rms
    ## focal_mean 5986845

``` r
writeRaster(merogots, filename = "./EGV/Climate_ThermalRadiation_cell.tif", overwrite=TRUE)
```
