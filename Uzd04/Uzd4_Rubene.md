Uzd4_Rubene
================
Betija Rubene
2025-01-18

## Ceturtais uzdevums - funkcijas, cikli, vienkodola un daudzkodolu skaitļošana

Darbam nepieciešamās pakotnes

``` r
library(tidyverse)
library(terra)
library(sf)
library(sfarrow)
library(foreach)
library(doParallel)
```

Uzdevuma veikšanai nepieciešami Centra virsmežniecības dati, kā arī
references rastri 10 un 100 m izšķirtspējā

``` r
centra_mezi <- st_read_parquet("../centra_mezi/centra_mezi_labots.parquet")
ref10m <- rast(("../ref_rastri/LV10m_10km.tif"))
ref100m <- rast(("../ref_rastri/LV100m_10km.tif"))
```

# 1) Funkcijas izveide

Jāizveido tāda funkcija, kura secīgi:

- no piedāvātā MVR faila atlasa mežaudzes, kurās valdošā koku suga ir
  priede (sugas kods ir “1”);

- sagatavo tādu rastru 10 m izšķirtspējā visai valsts teritorijai, kurā
  priežu mežaudzes no iepriekšējā punkta ir apzīmētas ar 1, pārējās
  Latvijas sauzsemes teritorijā esošās šūnas apzīmētas ar 0 un pārējās
  šūnas ir NA, un tas atbilst projekta Zenodo repozitorijā ievietotajam
  references slānim LV10m_10km.tif;

- iepriekšējā punkta rastru pārveido uz 100 m šūnu, aprēķinot priežu
  mežaudžu platības īpatsvaru (no kopējās platības) ik 100 m šūnā,
  nodrošinot atbilstību projekta Zenodo repozitorijā ievietotajam
  references slānim LV100m_10km.tif;

- saglabā iepriekšējā punktā izveidoto slāni (ar 100 m izšķirtspēju) kā
  GeoTIFF failu ar relatīvo ceļu norādītā vietā cietajā diskā, pieņemot
  ievades faila nosaukumu.

Ieteiktajos avotos lasīju, ka vieglāk ir sākumā uzrakstīt komandrindas
un pēc tam tās savietot strādājošā funkcijā nevis uzreiz rakstīt
funkciju.

Atlasu, kurās mežaudzēs valdošā suga ir priedes (pēc pirmā stāva pirmās
sugas, kolonna s10 = “1”)

``` r
priezu_mezi <- centra_mezi %>% filter(s10 == 1)
```

10 m rastra sagatavošana, kurā priežu mežaudzes no iepriekšējā punkta ir
apzīmētas ar 1, pārējās Latvijas sauzsemes teritorijā esošās šūnas
apzīmētas ar 0 un pārējās šūnas ir NA atbilstoši references slānim

``` r
# ref10m_cropped <- crop(ref10m, centra_mezi)
priezu_mezi_10m <- rasterize(priezu_mezi,ref10m, background = 0)
priezu_mezi_10m[is.na(ref10m)] <- NA
```

Iepriekšējā punkta rastru pārveido uz 100 m šūnu, aprēķinot priežu
mežaudžu platības īpatsvaru (no kopējās platības) ik 100 m šūnā un
uzreiz saglabāju rezultātu

``` r
priezu_mezi_100m <- priezu_mezi_10m %>%
    aggregate(fact = 10, fun = sum) %>%
    resample(ref100m, method = "near", filename = "../centra_mezi/priezu_mezi_100m.tif", overwrite = TRUE)
```

Tagad tas viss jāsavieto funkcijā

``` r
#bet vispirms noņemu visus objektus, lai vairu sākt ar tīru darba vidi
rm(centra_mezi, ref10m, ref100m, priezu_mezi, priezu_mezi_10m, priezu_mezi_100m) 
```

Funkcija:

``` r
mana_funkcija <- function(mezs_dati,koku_suga) {
ref10m <- rast(("../ref_rastri/LV10m_10km.tif"))
ref100m <- rast(("../ref_rastri/LV100m_10km.tif"))
vieta <- "../centra_mezi"
mezi <- st_read_parquet(mezs_dati)
faila_nos <- tools::file_path_sans_ext(basename(mezs_dati))
fails <- file.path(vieta, paste0(faila_nos, ".tif"))
priezu_mezi <- mezi %>% filter(s10 == koku_suga)
priezu_mezi_10m <- rasterize(priezu_mezi,ref10m, background = 0)
priezu_mezi_10m[is.na(ref10m)] <- NA
priezu_mezi_100m <-
  terra::resample(priezu_mezi_10m,ref100m,method="average",
                  filename = fails, overwrite=TRUE)
}
```

Izmēģinu ar centra mežu datiem, mērot patērēto laiku:

``` r
uzd2 <- system.time(mana_funkcija(
  mezs_dati = "../centra_mezi/centra_mezi_labots.parquet",
  koku_suga = 1
))
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

``` r
print(uzd2)
```

    ##    user  system elapsed 
    ##  121.42   20.83  342.67

Šī darbība aizņēma aptuveni 3,5 minūtes.

Operatīvā atmiņa pietika, maksimāli tika izmantoti aptuveni 6,3 GB
pastāvīgi ap 5,8 GB, pirms procesa uzsākšanas ap 5,4 GB. Skatoties
resource monitor, maksimāli novēroju 10 CPU (šķiet, ka vēl neesmu
sapratusi kādas terminoloģijas nianses), lielākoties tas svārstījās no
6-9

Vai funkcija dara to, kas bija paredzēts? Paskatīšos vizuāli uz
rezultātu

``` r
rezultats <- rast("../centra_mezi/centra_mezi_labots.tif")
plot(rezultats) # šķiet, ka viss labi
```

![](Uzd4_Rubene_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

# 3) Funkcjas pielietošana ciklā

Vispirms jāpārveido nodaļu faili geoparquet formātā, jo iepriekš
izveidoju tikai parquet failu visām nodaļām kopā. To arī darīšu ar ciklu

Sākumā izveidošu sarakstu ar shapefile nosaukumiem, kas atrodami
lejupielādētajā mapē

``` r
MVRnodalas_shp <- list.files("../centra_mezi", pattern = "\\.shp$", full.names = TRUE)
```

Izveidoju funkciju, kura pārvērš visus shapefile parquet formātā un to
pielietoju ciklā

``` r
fun_parquet <- function(mezs_dati){
meza_fails <- st_read(mezs_dati, quiet = TRUE)
faila_nos <- tools::file_path_sans_ext(basename(mezs_dati))
fails <- file.path("../centra_mezi", paste0(faila_nos, ".parquet"))
st_write_parquet(meza_fails,fails)
}

for (file in MVRnodalas_shp) {
  fun_parquet(file)
}
```

Pielietoju iepriekš izveidoto funkciju (mana_funkcija) ciklā

``` r
MVRnodalas_par <- list.files("../centra_mezi", pattern = "^nodala\\d{4}\\.parquet$", full.names = TRUE)
print(MVRnodalas_par)
```

    ## [1] "../centra_mezi/nodala2651.parquet" "../centra_mezi/nodala2652.parquet"
    ## [3] "../centra_mezi/nodala2653.parquet" "../centra_mezi/nodala2654.parquet"
    ## [5] "../centra_mezi/nodala2655.parquet"

``` r
uzd3 <- system.time(
  for (file in MVRnodalas_par) {
  result <- mana_funkcija(mezs_dati = file, koku_suga = 1)
  })
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

``` r
print(uzd3)
```

    ##    user  system elapsed 
    ##  653.81  121.19  990.28

Darbība aizņēma krietni ilgāku laiku - aptuveni 15 minūtes. Operatīvā
atmiņa pietika, viss līdzīgi kā iepriekš - maksimāli tika izmantoti
aptuveni 6,3 GB, bet lielākoties ap 5,8 GB . Maksimāli novēroju 10 CPU,
lielākoties svārstījās no 6-9

# 4) Iepriekšējā uzdevuma atkārtošana, izmantojot {doParallel} un {foreach}

Vispirms jānosaka klāsteris kā tieši vienu CPU kodolu liels

``` r
cluster <- makeCluster(1)
registerDoParallel(cluster)
```

Izmantošu {foreach} ciklu

``` r
# sākotnējais variants, kurš nenostrādāja

uzd4 <- system.time(
  foreach(mezi = MVRnodalas_par, 
          .packages = c("tidyverse","sfarrow","sf","terra")) %dopar% {
            mana_funkcija(mezs_dati = mezi, koku_suga = 1)})

stopCluster(cluster)
```

Darbība aizņēma ilgāk - aptuveni 26 minūtes. Operatīvā atmiņa pietika,
maksimāli tika izmantoti aptuveni 7 GB, bet lielākoties ap 6,5 GB.
Maksimāli novēroju 8 CPU, lielākoties svārstījās no 3-6.

# 5) Uzdevuma atkārtošana ar diviem CPU kodoliem

``` r
cluster2 <- makeCluster(2)
registerDoParallel(cluster2)

uzd5 <- system.time({
  foreach(mezi = MVRnodalas_par, 
          .packages = c("tidyverse","sfarrow","sf","terra")) %dopar% {
            mana_funkcija(mezs_dati = mezi, koku_suga = 1)}})
print(uzd5)
```

    ##    user  system elapsed 
    ##    1.20    0.08  681.23

``` r
stopCluster(cluster2)
```

Darbība aizņēma aptuveni 18 minūtes - tātad krietni mazāk nekā
iepriekšējā. Operatīvā atmiņa pietika, līdzīgi kā iepriekš, maksimāli
tika izmantoti aptuveni 7 GB, bet lielākoties ap 6,5 GB. Maksimāli
novēroju 8 CPU, lielākoties svārstījās no 4-7
