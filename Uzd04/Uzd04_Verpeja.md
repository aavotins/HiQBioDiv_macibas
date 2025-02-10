Ceturtais uzdevums: funkcijas, cikli, vienkodola un daudzkodolu
skaitļošana
================

Vajadzīgo pakotņu lejupielāde un ielase

``` r
library(sf)
library(sfarrow)
library(dplyr)
library(terra)
library(fasterize)
library(doParallel)
library(parallel)
library(tools)
library(foreach)
```

# 1. uzdevums

``` r
mana_funkcija <- function(ievades_fails, gala_nosaukums) {
#MVR Centra virsmežniecības datu ielase
centra_gpq_vieta <- ievades_fails
MVR_dati <-st_read_parquet(centra_gpq_vieta, quiet = TRUE)

#Mežaudžu atlase pēc vadošās sugas
priezu_mezaudzes <- MVR_dati %>%
  filter(MVR_dati$zkat == 10, MVR_dati$s10 == 1)

#Refernces 10m rastra ielase
r_10m <- raster("C:/Users/vinet/Documents/VPP/3uzd/rastrs/LV10m_10km.tif")

#Apgriežu references rastru, lai datoram būtu jāstrādā ar mazāka izmēra failu, kas atbilst tikai vajadzīgajai teritorijai
mr_10m <- crop(r_10m, MVR_dati)

#MVR priezu mežaudžu datu rasterizācija
priezu_mezaudzes_10m <- fasterize(priezu_mezaudzes, mr_10m, background=0)
priezu_mezaudzes_10m <- rast(mask(priezu_mezaudzes_10m, mr_10m))
crs(priezu_mezaudzes_10m) <-"epsg:3059"

#References 100m rastra ielase
r_100m <-rast("C:/Users/vinet/Documents/VPP/3uzd/rastrs/LV100m_10km.tif")

#Rasterizētā faila pārveide uz 100m izšķirtspēju
priezu_mezaudzes_100m <- resample(priezu_mezaudzes_10m, r_100m, method="sum")

#Gala faila saglabāšana
writeRaster(priezu_mezaudzes_100m, gala_nosaukums , overwrite = TRUE)}
```

# 2. uzdevums

``` r
izpildes_laiks1 <- system.time(mana_funkcija("C:/Users/vinet/Documents/VPP/2uzd/apv_centra.parquet", "C:/Users/vinet/Documents/VPP/4uzd/priezu_mezaudzes_100m.tif"))
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
izpildes_laiks1
```

    ##    user  system elapsed 
    ##  140.53   29.90  171.98

``` r
#Pārbaudu vai sanāca uztaisīt vajadzīgo rastru
gala_rezultats <- rast("C:/Users/vinet/Documents/VPP/4uzd/priezu_mezaudzes_100m.tif")
plot(gala_rezultats)
```

![](Uzd04_Verpeja_files/figure-gfm/Jaunās%20funkcijas%20notestēšana%20un%20tās%20patērētā%20laika%20noteikšana-1.png)<!-- -->
Atbilde: Izmantojot neapgrieztu 10x10m references režģa objektu
rasterizēšanas procesā, nebija iespējams veikt funkciju uz mana datora.
Taču, apgriežot refernces slāni ar centra MVR datiem, tas izdevās ļoti
veiksmīgi. Apgriežot r_10m, funkcijas izpilde prasīja 215 s jeb aptuveni
3,5 min. Šī darbība vidēji patērēja aptuveni 900MB (max 1400MB) no RAM
atmiņas, kā arī 8% no CPU.

# 3. uzdevums

``` r
#Izveidoju funkciju, kas ielasīs shapefile, pārveidos un saglabās kā parquet
mana_funkcija2 <- function(ievades_shp, parquet_mape) {
#Ielasu shp failu
shp_fails <- st_read(ievades_shp, quiet=TRUE)
  
#iegūstu nodaļu nosaukumus
nodalas_nosaukums <- file_path_sans_ext(basename(ievades_shp))

#Uztaisu gala parquet failu nosaukumus
parquet_nosaukums <- file.path(parquet_mape, paste0(nodalas_nosaukums, ".parquet"))

#Izveidoju un saglabāju jauno parquet failu
st_write_parquet(shp_fails,parquet_nosaukums)
}


#Uztaisi gala failu mapu kā objektu
parquet_mape <- "C:/Users/vinet/Documents/VPP/4uzd/nodalas"
#Uztaisu sarakstu ar MVR nodaļu shp failu nosaukumiem (shp_failu_nosaukumi būs x)
shp_failu_nosaukumi <- list.files(path = "C:/Users/vinet/Documents/VPP/2uzd/centra", pattern = "\\.shp$", full.names = TRUE)
```

Izveidoju atsevišķus parquet failus katrai MVR nodaļai

``` r
#Uztaisu ciklu, kas mana_funkcija2 izpilda visiem MVR nodaļu shapefiliem
for (x in shp_failu_nosaukumi){
mana_funkcija2(x,parquet_mape)}

#Izveidoju sarakstu ar jaunajiem parquet failiem
parquet_failu_nosaukumi <- list.files(path = "C:/Users/vinet/Documents/VPP/4uzd/nodalas", pattern = "\\.parquet$", full.names = TRUE)
```

Izpildu iepriekš izveidoto mana_funkcija katrai MVR nodaļai atsevišķi ar
for cikla palīdzību

``` r
izpildes_laiks2 <- system.time(for (x in parquet_failu_nosaukumi){
  gala_nosaukums <- paste0(file_path_sans_ext(x), ".tif")
  mana_funkcija(x,gala_nosaukums)})
izpildes_laiks2
```

    ##    user  system elapsed 
    ##  129.91   38.89  170.62

Atbilde: Cikla izpilde prasīja 335s jeb nedaudz virs 5,5 min. Šī cikla
izpildei vidēji tika patērti 1200-1400MB no RAM atmiņas (max 1900MB) un
8% no CPU. Salīdznot ar darbības izpildi visām nodaļām kopā, šis
variants prasīja aptuveni 2 min vairāk laika, kā arī patērēja nedaudz
vairāk RAM atmiņas.

# 4. uzdevums

Darbības izpilde ar foreach ciklu, izmantojot CPU 1 kodolu.

``` r
klast_1k <- makeCluster(1) 
registerDoParallel(klast_1k)

foreach_izpildes_laiks <- system.time(
  foreach(x = parquet_failu_nosaukumi, .packages = c("sf", "sfarrow", "dplyr", "terra", "fasterize", "tools")) %dopar% {
    gala_nosaukums <- paste0(file_path_sans_ext(x), ".tif")
    mana_funkcija(x, gala_nosaukums)})

stopCluster(klast_1k) 
print(foreach_izpildes_laiks)
```

    ##    user  system elapsed 
    ##    3.25    1.47  181.16

Atbilde: Cikla izpilde prasīja 209 s jeb nedaudz zem 3,5 min. Šī cikla
izpildei vidēji tika patērti 900-1200MB no RAM atmiņas (max 1900MB). Šis
variants bija aptuveni tik pat ātrs un līdzīgu RAM daudzumu patērēja kā
izpildot mana_funkcija kopīgajam MVR parquet failam, kurā ir visas
nodaļas kopā un noteikti ātrāks nekā variants ar for ciklu.

# 5. uzdevums

Darbības izpilde ar foreach ciklu, izmantojot CPU 2 kodolus.

``` r
gc()
```

    ##           used  (Mb) gc trigger  (Mb) max used  (Mb)
    ## Ncells 2657586 142.0    4729874 252.7  4729874 252.7
    ## Vcells 3504240  26.8    8388608  64.0  6179263  47.2

``` r
#klast_2k <- makeCluster(2) 
#registerDoParallel(klast_2k)

#foreach_izpildes_laiks <- system.time(
#  foreach(x = parquet_failu_nosaukumi, .packages = c("sf", "sfarrow", "dplyr", "terra", #"fasterize", "tools")) %dopar% {
#    gala_nosaukums <- paste0(file_path_sans_ext(x), ".tif")
#    mana_funkcija(x, gala_nosaukums)})

#stopCluster(klast_2k) 
#print(foreach_izpildes_laiks)
```

Diemžēl man neizdevās procesu atkārtot, izmantojot vairāk kā 1 kodolu.
To mēģinot, meta ārā paziņojumu, ka nepietika datora RAM. Šo mēģināju
vairākkārt, izslēdzot citas datora programmas, ielādējot tikai
vajadzīgos failus R vidē un iztīrot R kešsatmiņu pirms cikla izpildes,
bet tas diemžēl nepalīdzēja. Bet, ja būtu pietikusi datora RAM, tad nu
teorētiski tam vajadzētu būt ātrākam risinājumam.
