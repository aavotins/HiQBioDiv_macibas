Piektais uzdevums
================
Raitis Riters

Uzdevuma izpildei man bija nepieciešamas šādas pakotnes.

``` r
library(sf)
library(terra)
library(tidyverse)
library(arrow)
library(sfarrow)
library(fasterize)
library(tictoc)
```

Norādu funkcijai vajadzīgos argumentus

``` r
ievades_fails<-st_read_parquet("../Uzd03/centrs_geoparquet.parquet")
ievades_fails<-st_transform(ievades_fails, 3059)

rastrs_10m<-rast("../Uzd03/rastrs/LV10m_10km.tif")

rastrs_100m<-rast("../Uzd03/rastrs/LV100m_10km.tif")

regions<-st_read_parquet("../Uzd03/vektori/tks93_50km.parquet") %>% filter (NOSAUKUMS %in% c("Ogre", "Ropaži", "Mālpils", "Skrīveri"))
regions<-st_transform(regions, 3059)

regions_ogre<- regions %>% filter(NOSAUKUMS %in% "Ogre")
regions_skriveri<- regions %>% filter(NOSAUKUMS %in% "Skrīveri")
regions_ropazi<- regions %>% filter(NOSAUKUMS %in% "Ropaži")
regions_malpils<- regions %>% filter(NOSAUKUMS %in% "Mālpils")
regioni<-list(regions_ogre, regions_skriveri, regions_ropazi, regions_malpils)

kodi<-c("Ogre", "Ropaži", "Mālpils", "Skrīveri") 
```

Izlēmu darboties ar Ogres, Ropažu, Mālpils un Skrīveru kartēm.

\##1. st_join Laika mērogošanai lietošu pakotni tictoc. Var būt, ka tas
nav izcils variants, bet man patīk, ka var mērogot laiku pa vairākiem
chunks. \#1.1. Pirmais solis

``` r
tic()

skriveri<-st_join(ievades_fails, regions_skriveri, left=FALSE, join=st_intersects)
ogre<-st_join(ievades_fails, regions_ogre, left=FALSE, join=st_intersects)
ropazi<-st_join(ievades_fails, regions_ropazi, left=FALSE, join=st_intersects)
malpils<-st_join(ievades_fails, regions_malpils, left=FALSE, join=st_intersects)
kartes<-list(ogre, ropazi, malpils, skriveri)
```

Objektā *ievades_fails* ir 506828 objekti 70 kolonnās, kamēr objektos
kopā ir 116009 objekti 74 kolonnās. Objektā Ogre - 32953 objekti,
Ropaži - 34696, Mālpils - 26861, Skrīveri - 21499.

\#1.2. Otrais solis

``` r
apaksuzdevums1_izveide<-function(kartes, rastrs_10m, rastrs_100m, kodi) {
  results<-list()
  
  for (i in seq_along(kartes)) {
    karte<-kartes[[i]]
    karte<-st_transform(karte, 3059)
    priedes<-karte %>% filter(s10 == "1")
    priedes<-st_crop(priedes, karte)
    rastrins<-crop(rastrs_10m, priedes)
    rastrins<-raster(rastrins)
    
    # Sagatavoju datus
    priedes$s10<-1
    priedes<-priedes %>% select(s10)
    
    # Izveidoju 10m rastru
    priedes10m<-fasterize(priedes, rastrins, field = "s10", background = 0)
    priedes10m<-terra::rast(priedes10m)
    
    # Izveidoju 100m rastru ar priežu īpatsvaru
    rastrs100m<-crop(rastrs_100m, priedes10m)
    priedes10m<-project(priedes10m, crs(rastrs100m))
    priedes100m<-resample(priedes10m, rastrs100m, method = "average")
    
    # Ierakstu rastru uz diska
    vieta_kartei<-paste0("apaksuzd1_", kodi[i], ".tif")
    writeRaster(priedes100m, vieta_kartei, overwrite = TRUE)
    
    results[[i]]<-priedes100m
  }
  
  return(results)
}


invisible(apaksuzdevums1_izveide(kartes, rastrs_10m, rastrs_100m, kodi)) #lietoju invisible, lai fails būtu pārskatāmāks
```

Liekas, ka objekti pie karšu lapām tiek saglabāti, taču ir manāms tā kā
rāmis, kas iziet nedaudz ārpus reģiona teritorijas. Liekas, ka tas ir
spatial_join veids, kā saglabāt visu pieejamo informāciju.

\#1.3. Trešais solis

``` r
apaksuzdevums1_apvienosana<-function(kodi){
  kopa<-list()
  for (kods in kodi){
    dati<-terra::rast(paste0("./apaksuzd1_", kods, ".tif"))
    kopa<-append(kopa, list(dati))
  }
  kombineti<-do.call(terra::merge, kopa)
  #Ierakstu rastru uz diska
  writeRaster(kombineti, "./apaksuzdevums1_apvienots.tif", overwrite = TRUE)
  return(kopa)
}
  
invisible(apaksuzdevums1_apvienosana(kodi))

toc()
```

    ## 32.86 sec elapsed

Izmantojot spatial_join, ir redzams tāds kā “rāmis”. Objekti kas ir
iekšienē saglabājas, taču objekti pie karšu lapu malām kartes iekšienē
tiek pārklāti ar rāmi, rezultātā daļa objektu tiek apslēpti.

Viss kopā prasīja 77.5 sekundes.

\##2. clipping \#2.1. Pirmais solis

``` r
tic()

ogre<-st_intersection(ievades_fails, regions_ogre)
ropazi<- st_intersection(ievades_fails, regions_ropazi)
malpils<-st_intersection(ievades_fails, regions_malpils)
skriveri<-st_intersection(ievades_fails, regions_skriveri)
kartes<-list(ogre, ropazi, malpils, skriveri)
```

Lietojot clipping ir identisks objektu skaits karšu lapās, kas veidotas
ar spatial_join.

\#2.2. Otrais solis

``` r
apaksuzdevums2_izveide<-function(kartes, rastrs_10m, rastrs_100m, kodi) {
  results<-list()
  for (i in seq_along(kartes)) {
    karte<-kartes[[i]]
    karte<-st_transform(karte, 3059)
    priedes<-karte %>% filter(s10 == "1")
    priedes<-st_crop(priedes, karte)
    rastrins<-crop(rastrs_10m, priedes)
    rastrins<-raster(rastrins)
    
    # Sagatavoju datus
    priedes$s10<-1
    priedes<-priedes %>% select(s10)
    
    # Izveidoju 10m rastru
    priedes10m<-fasterize(priedes, rastrins, field = "s10", background = 0)
    priedes10m<-terra::rast(priedes10m)
    
    # Izveidoju 100m rastru ar priežu īpatsvaru
    rastrs100m<-crop(rastrs_100m, priedes10m)
    priedes10m<-project(priedes10m, crs(rastrs100m))
    priedes100m<-resample(priedes10m, rastrs100m, method = "average")
    
    # Ierakstu rastru uz diska
    vieta_kartei<-paste0("apaksuzd2_", kodi[i], ".tif")
    writeRaster(priedes100m, vieta_kartei, overwrite = TRUE)
    
    results[[i]]<-priedes100m
  }
  
  return(results)
}


invisible(apaksuzdevums2_izveide(kartes, rastrs_10m, rastrs_100m, kodi))
```

Izmantojot clipping, nav redzams rāmis, kā ar spatial join. Izskatās, ka
objekti iekšienē saglabājas, taču objekti pie malām netiek dzēsti, bet
tiek šķelti, t.i., objektu daļas ārpus robežām netiek saglabātas, bet
paši objekti paliek, kas izskaidro identisko objektu daudzumu ar
iepriekšējo uzdevumu.

\#2.3. Trešais solis

``` r
apaksuzdevums2_apvienosana<-function(kodi) {
  kopa<-list()
  for (kods in kodi) {
    dati<-terra::rast(paste0("./apaksuzd2_", kods, ".tif"))
    kopa<-append(kopa, list(dati))
  }
  kombineti<-do.call(terra::merge, kopa)
  writeRaster(kombineti, "./apaksuzdevums2_apvienots.tif", overwrite = TRUE)
  return(kombineti)
}

invisible(apaksuzdevums2_apvienosana(kodi)) 

toc()
```

    ## 32.48 sec elapsed

Izmantojot clipping, nav redzams rāmis, tādēļ arī apvienojot karšu lapas
nav redzams rāmis un netiek aizsegti objekti. Nav arī manāmas robežas.

\##3. st_filter \#3.1. Pirmais solis

``` r
tic()

apaksuzdevums3_objekti <- function(ievades_fails, regioni, kodi) {
  for (i in seq_along(regioni)) {
    filtreti <- st_filter(ievades_fails, regioni[[i]])
    assign(paste0(kodi[i], "_filtrets"), filtreti, envir = .GlobalEnv)
  }
}
apaksuzdevums3_objekti(ievades_fails, regioni, kodi)
```

Izmantojot filtrēšanu, lapās ir 32953, 34696, 21499, 26861 objektu.
Skaits atkal sakrīt ar iepriekšējiem uzdevumiem.

\#3.2. Otrais solis

``` r
kartes<-list(Ogre_filtrets, Ropaži_filtrets, Mālpils_filtrets, Skrīveri_filtrets)
apaksuzdevums3_izveide<-function(kartes, rastrs_10m, rastrs_100m, kodi) {
  
  results<-list()
  for (i in seq_along(kartes)) {
    karte<-kartes[[i]]
    karte<-st_transform(karte, 3059)
    priedes<-karte %>% filter(s10 == "1")
    priedes<-st_crop(priedes, karte)
    rastrins<-crop(rastrs_10m, priedes)
    rastrins<-raster(rastrins)
    
    # Sagatavoju datus
    priedes$s10<-1
    priedes<-priedes %>% select(s10)
    
    # Izveidoju 10m rastru
    priedes10m<-fasterize(priedes, rastrins, field = "s10", background = 0)
    priedes10m<-terra::rast(priedes10m)
    
    # Izveidoju 100m rastru ar priežu īpatsvaru
    rastrs100m<-crop(rastrs_100m, priedes10m)
    priedes10m<-project(priedes10m, crs(rastrs100m))
    priedes100m<-resample(priedes10m, rastrs100m, method = "average")
    
    # Ierakstu rastru uz diska
    vieta_kartei<-paste0("apaksuzd3_", kodi[i], ".tif")
    writeRaster(priedes100m, vieta_kartei, overwrite = TRUE)
    
    results[[i]]<-priedes100m
  }
  
  return(results)
}

invisible(apaksuzdevums3_izveide(kartes, rastrs_10m, rastrs_100m, kodi))
```

Lietojot filtrēšanu, rodas identisks rezultāts kā lietojot st_join. Ir
redzams rāmis, kas iziet ārpus reģiona teritorijas un tiek saglabāti
objekti, kuru vismaz kaut kāda daļa ir reģiona iekšienē. Šī metode
aizņēma 98 sekundes.

\#3.3. Trešais solis Trešais solis strādā kā iecerēts.

``` r
apaksuzdevums3_apvienosana<-function(kodi) {
  kopa<-list()
  for (kods in kodi) {
    dati<-terra::rast(paste0("./apaksuzd3_", kods, ".tif"))
    kopa<-append(kopa, list(dati))
  }
  kombineti<-do.call(terra::merge, kopa)
  writeRaster(kombineti, "./apaksuzdevums3_apvienots.tif", overwrite = TRUE)
  return(kombineti)
}

  
invisible(apaksuzdevums3_apvienosana(kodi))
toc()
```

    ## 35.29 sec elapsed

Kopumā ir novērojami tieši tādi paši rezultāti kā ar st_join - ir rāmis
starp kartes lapām, kas aizsedz objektus. Kopumā šī metode aizņēma 131.5
sekundes.

\##4. st_within \#4.1. Pirmais solis Visas iepriekšējās pieejas kaut
kādā veidā atgriež vairāk vai mazāk vienādus objektus ar vienādu objektu
skaitu. Taču man interesē, kas notiktu, ja varētu pielikt argumentu, kas
iekļauj tikai un vienīgi objektus, kas atrodas pilnīgi lapas iekšienē.
Par laimi pastāv arguments *st_within*. Var būt pārprotu, bet liekas, ka
šis apakšuzdevums veicams tikai ar vienu lapu. Izmēģināšu argumentu uz
Ogres lapu, jo tajā ir daudz objektu uz karšu malām.

``` r
tic()
mezi<-st_filter(ievades_fails, regions_ogre, .predicate = st_within)
```

Šajā gadījumā tika iegūti 32165 objekti, kas ir par 788 objektiem mazāk
nekā visos iepriekšēojos mēginājumos.

\#4.2. Otrais solis

``` r
apaksuzdevums4_rakstisana<-function(mezi, rastrs_10m, rastrs_100m){
  priedes<-mezi %>% filter(s10=="1")
  rastrins<-crop(rastrs_10m, priedes)
  rastrins<-raster(rastrins)
  #Sagatavoju datus
  priedes$s10<-1
  priedes<-priedes %>% select(s10)
  #Izveidoju 10m rastru
  priedes10m<-fasterize(priedes, rastrins, field = "s10", background = 0)
  priedes10m<-terra::rast(priedes10m)
  #Izveidoju 100m rastru ar priežu īpatsvaru
  rastrs100m<-crop(rastrs_100m, priedes10m)
  priedes10m<-project(priedes10m, crs(rastrs100m))
  priedes100m<-resample(priedes10m, rastrs100m, method = "average")
  #Ierakstu rastru uz diska
  writeRaster(priedes100m, "./apaksuzdevums4.tif", overwrite = TRUE)
  return(priedes100m)
}
  
invisible(apaksuzdevums4_rakstisana(mezi, rastrs_10m, rastrs_100m))
toc()
```

    ## 6.06 sec elapsed

Rastram nav rāmja vai kaut kā tamlīdzīga. Kopā tika aizņemtas 63.5
sekundes.

## 5. crop

``` r
tic()
apaksuzdevums5_apvienosana<-function(ievades_fails, rastrs_10m, rastrs_100m, regioni){
  regioni_apvienots <- do.call(rbind, regioni)
  bbox <- st_bbox(regioni_apvienots)
  rastrs_10m <- terra::crop(rastrs_10m, ext(bbox))
  rastrs_100m <- terra::crop(rastrs_100m, ext(bbox))
  priedes<-ievades_fails %>% filter(s10=="1")
  rastrins<-crop(rastrs_10m, priedes)
  rastrins<-raster(rastrins)
  #Sagatavoju datus
  priedes$s10<-1
  priedes<-priedes %>% select(s10)
  #Izveidoju 10m rastru
  priedes10m<-fasterize(priedes, rastrins, field = "s10", background = 0)
  priedes10m<-terra::rast(priedes10m)
  #Izveidoju 100m rastru ar priežu īpatsvaru
  rastrs100m<-crop(rastrs_100m, priedes10m)
  priedes10m<-project(priedes10m, crs(rastrs100m))
  priedes100m<-resample(priedes10m, rastrs100m, method = "average")
  #Ierakstu rastru uz diska
  writeRaster(priedes100m, "./apaksuzdevums5.tif", overwrite = TRUE)
  return(priedes100m)
}
  
invisible(apaksuzdevums5_apvienosana(ievades_fails, rastrs_10m, rastrs_100m, regioni))
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
toc()
```

    ## 17.64 sec elapsed

Kopā šī metode aizņēma 44.2 sekundes. Nav redzams rāmis vai kaut kas
tamlīdzīgs, taču nav īsti informācijas par objektu skaitu. Tomēr,
izskatās pēc būtībā tā paša rezultāta kā ar clipping, taču šī metode ir
ātrāka.

6.  apakšuzdevums - mans dators tomēr nav uzskatāms par jaudīgu, tādēļ
    šo uzdevumus labāk atlikšu.

Vismazāk laika aizņēma croppošana, kamēr visvairāk - filtrēšana.

Kopumā, manuprāt, visefektīvākais veids ir izmantot clipping vai
vienkārši croppot. Tomēr, bieži vien ir nepieciešams saglabāt visu
pieejamo informāciju, tādēļ spatial_join var būt noderīgs risinājums
atsevišķos gadījumos, kad nav tik svarīgi iekļauties kādās robežās.
