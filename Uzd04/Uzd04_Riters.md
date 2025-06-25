Pakotnes, kas man bija nepieciešamas uzdevuma izpildei.

    library(sf)
    library(arrow)
    library(terra)
    library(fasterize)
    library(tidyverse)
    library(dplyr)
    library(doParallel)

## Pirmais uzdevums

Funkcijas izveide

    mana_funkcija<-function(ievades_fails, rastrs10m_vieta, rastrs100m_vieta, galamerkis){
      #Ielasu datus
      mezi <- read_parquet(ievades_fails)
      rastrs_10m <- rast(rastrs10m_vieta)
      rastrs_100m <- rast(rastrs100m_vieta)
      #Izfiltrēju un salaboju datus
      priedes <- mezi %>% filter(s10=="1")
      priedes <- st_as_sf(priedes)
      priedes <- st_set_crs(priedes, 3059)
      #Apgriežu rāmi
      rastrins <- crop(rastrs_10m, priedes)
      rastrins <- fasterize::raster(rastrins)
      #Sagatavoju datus
      priedes$s10 <- 1
      priedes <- priedes %>% select(s10)
      #Izveidoju 10m rastru
      priedes10m <- fasterize(priedes, rastrins, field = "s10", background = 0)
      priedes10m <- mask(priedes10m, rastrins)
      priedes10m <- terra::rast(priedes10m)
      #Izveidoju 100m rastru ar priežu īpatsvaru
      rastrs100m <- crop(rastrs_100m, priedes10m)
      priedes100m <- resample(priedes10m, rastrs100m, method = "average")
      # Write the output raster
      writeRaster(priedes100m, galamerkis, overwrite = TRUE)
      return(priedes100m)
    }

Funkcijas argumenti un palaišana

    ievades_fails<-"../Uzd03/centrs_geoparquet.parquet"
    rastrs10m_vieta<-"../Uzd03/rastrs/LV10m_10km.tif"
    rastrs100m_vieta<-"../Uzd03/rastrs/LV100m_10km.tif"
    galamerkis<-"./priedes100m.tif"
    mana_funkcija(ievades_fails, rastrs10m_vieta, rastrs100m_vieta, galamerkis)

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

    ## class       : SpatRaster 
    ## dimensions  : 1239, 1882, 1  (nrow, ncol, nlyr)
    ## resolution  : 100, 100  (x, y)
    ## extent      : 405200, 593400, 238500, 362400  (xmin, xmax, ymin, ymax)
    ## coord. ref. : +proj=tmerc +lat_0=0 +lon_0=24 +k=0.9996 +x_0=500000 +y_0=-6000000 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs 
    ## source(s)   : memory
    ## varname     : LV100m_10km 
    ## name        : layer 
    ## min value   :     0 
    ## max value   :     1

## Otrais uzdevums

    laiks1<- system.time(mana_funkcija(ievades_fails, rastrs10m_vieta, rastrs100m_vieta, galamerkis))

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

    print(laiks1)

    ##    user  system elapsed 
    ##   33.61   11.41   48.09

Funkcijas izpilde aizņēma 40-50 sekundes, pārsvarā aizņēma 2 kodolus,
bet mēdza arī sasniegt 3 kodolus. RAM bija pietiekams funkcijas
izpildei. Tas aizņēma aptuveni 1-1.2GB atmiņas.

## Trešais uzdevums

Vispirms iegūstu informāciju par to, cik un kādi apgabali atrodas šajā
kombinētajā failā.

    mezi<-read_parquet(ievades_fails)
    kodi<-unique(mezi$forestry_c)
    kodi<-c(kodi)
    print(kodi)

    ## [1] "2651" "2652" "2653" "2654" "2655"

    cikliska_funkcija <- function(mezi, kodi) {
      for (kods in kodi) {
        parketa_vieta <- paste0("./nodala", kods, ".parquet")
        apgabala_dati <- mezi %>% filter(forestry_c == as.character(kods))
        write_parquet(apgabala_dati, parketa_vieta)
      }
    }

    laiks2<-system.time(cikliska_funkcija(mezi, kodi))
    print(laiks2)

    ##    user  system elapsed 
    ##    2.84    0.45    3.48

Cikliskā funkcija aizņēma 2-3 sekundes un 3 CPU kodolus un 1 GB RAM.

## Ceturtais uzdevums

Nosākuma nosaku, cik man ir kodoli.

    cores <- detectCores()
    print(cores)

    ## [1] 8

Saka, ka 8, taču es labi zinu, ka manai ierīcei ir 4 kodoli. Šis laikam
atbilst procesēšanas vienībām. Tātad- katras 2 procesēšanas vienības= 1
kodols.

      cl <- makeCluster(detectCores()-6)
      registerDoParallel(cl)

    laiks3<-system.time(foreach(kods = kodi, .packages = c("dplyr", "arrow")) %dopar% {
        file_path <- paste0("./nodala", kods, ".parquet")
        subset_data <- mezi %>% filter(forestry_c == as.character(kods))
        write_parquet(subset_data, file_path)
      })

      stopCluster(cl)
    print(laiks3)

    ##    user  system elapsed 
    ##    4.12    2.95   21.56

Aizņemtais laiks ir krietni lielāks nekā ar noklusējuma uzstādījumiem.
Tika aizņemti brīžiem pat 75% CPU un 1.5GB RAM. Pietika atmiņas.

## Piektais uzdevums

    cl2 <- makeCluster(detectCores()-2)
      registerDoParallel(cl2)

    laiks4<-system.time(foreach(kods = kodi, .packages = c("dplyr", "arrow")) %dopar% {
        file_path <- paste0("./nodala", kods, ".parquet")
        subset_data <- mezi %>% filter(forestry_c == as.character(kods))
        write_parquet(subset_data, file_path)
      })

      stopCluster(cl2)
    print(laiks4)

    ##    user  system elapsed 
    ##   12.39   10.78   58.59

Tika izmantoti 2GB RAM un aizņemti 100% CPU, kā arī tika aizņemts daudz
ilgāks laiks, kas galīgi nešķiet loģiski. Nejaušības pēc nebiju
saglabājis šo uzdevumu un to nācās vēlreiz izdarīt. Pirmo reizi pildot
uzdevumu, funkcija ar 1 kodolu aizņēma daudz daudz vairāk laika nekā
funckija ar noklusējuma iestatījumiem vai 3 kodolu lietojumu. Man nav
īsti skaidrojuma šajai parādībai.
