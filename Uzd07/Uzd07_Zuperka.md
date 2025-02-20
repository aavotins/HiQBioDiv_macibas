07Uzd
================
Marks
2025-02-16

<br><br> 1) Izmantojiet {exactextractr}, lai 500 m buferzonās ap
projekta Zenodo repozitorijā pieejamo 100 m šūnu centriem
(pts100_sauszeme.parquet), kuri atrodas piektajā uzdevumā izvēlētajās
kartes lapās, aprēķinātu sestā uzdevuma ceturtajā apakšpunktā izveidotā
rastra katras klases platības īpatsvaru. <br> 1.1.) 5.uzd karšu lapas
nebiju saglabājis, tās uztaisu pa jaunam.

``` r
tks93_50km <- st_as_sf(arrow::read_parquet("../Uzd03/HiQBioDiv_vector_reference_grids/tks93_50km.parquet"))
tks_crs <- st_crs(tks93_50km) #Izvelku, lietošu visur turpmāk
Combined_centrs <- st_as_sf(arrow::read_parquet("../Uzd02/Combined_centrs.parquet"))

tks_centrs_i <- st_filter(tks93_50km, Combined_centrs)

saraksts <- c("Ķemeri", "Tukums", "Jaunpils", "Dobele")
tks_filtrs <- tks_centrs_i[tks_centrs_i$NOSAUKUMS %in% saraksts, ]

rm(tks_centrs_i, tks93_50km, Combined_centrs, saraksts)
```

<br> 1.2.) Aprēķināšu katras vecuma un mežu tipa klases īpatsvaru 500
metru buferzonā ap references centroīdiem.

``` r
Latvia_ref_vector_100x100 <- read_parquet("../Uzd03/HiQBioDiv_vector_reference_grids/pts100_sauzeme.parquet")

#Pasmags, lai visu ielasītu kā sf, izvelku tikai vajadzīgām lapām atbilstošās šūnas
saraksts <- as.list(tks_filtrs$NUMURS)
Latvia_ref_vector_100x100 <- Latvia_ref_vector_100x100[Latvia_ref_vector_100x100$tks50km %in% saraksts, ]
Latvia_ref_vector_100x100 <- st_as_sf(Latvia_ref_vector_100x100)  

MVR_LAD_rastrs <- rast("../Uzd06/MVR_LAD_centrs.tif")

st_crs(Latvia_ref_vector_100x100) <- crs(tks_crs)
rm(saraksts)
```

<br> Izpildu paralēli uz 4 kodoliem.

``` r
tks_filtrs <- tks_filtrs %>%
  group_by(NOSAUKUMS) %>%
  group_split()

tks_bbox_saraksts <- lapply(tks_filtrs, function(bbox) {st_bbox(bbox)})

ref_vector_100x100_saraksts <- lapply(tks_bbox_saraksts, function(crop) {
  crop <- st_crop(Latvia_ref_vector_100x100, crop)
  st_set_crs(crop, st_crs(tks_crs))
})

bufferu_saraksts <- lapply(ref_vector_100x100_saraksts, function(buffer)
  st_buffer(buffer, dist=500))

MVR_LAD_crop_saraksts <- lapply(tks_bbox_saraksts, function(rastercrop) {
  ext <- ext(rastercrop)
  raster::raster(terra::crop(MVR_LAD_rastrs, ext)) #foreach vajag rasterlayer
})

extr_funkcija <- function(extr, MVR_LAD_crop_saraksts, bufferu_saraksts) {
  rez <- exactextractr::exact_extract(MVR_LAD_crop_saraksts[[extr]], bufferu_saraksts[[extr]], fun = "frac", append_cols = "id")
}

cluster <- makeCluster(4)
clusterExport(cluster, varlist = c("MVR_LAD_crop_saraksts", "bufferu_saraksts", "extr_funkcija"))
registerDoParallel(cluster)

results <- foreach(extr = 1:4) %dopar% {
  extr_funkcija(extr, MVR_LAD_crop_saraksts, bufferu_saraksts)
}

stopCluster(cluster)

for (i in seq_along(bufferu_saraksts)) {
  bufferu_saraksts[[i]] <- left_join(bufferu_saraksts[[i]], results[[i]], by="id")
}

comb <- do.call(rbind, bufferu_saraksts)

Latvia_ref_vector_100x100 <- left_join(Latvia_ref_vector_100x100, as.data.frame(comb), by="id")
Latvia_ref_vector_100x100$geom.y <- NULL #Lai nesaglabā bufera ģeom

Latvia_ref_vector_100x100 <- Latvia_ref_vector_100x100 %>%
  filter(!is.na(frac_0), frac_0 < 1) #Ja ir 1, tad nav datu par 500 metru rādiusā esošām klasēm - nav jēgas saglabāt tukšas vērtības, un NA ir 500 punkti pašos rietumos (kaut kas ar bbox?)

st_write_parquet(Latvia_ref_vector_100x100, "../Uzd07/lapas_klasu_ipatsvars.parquet")
```

<br> Brīvi izvēlaties desmit blakus esošus 1km kvadrātus, kas atrodas
trešajā uzdevumā ar Lauku atbalsta dienesta datiem aptvertajā
teritorijā. Izmantojiet projekta Zenodo repozitorijā dotos 300 m un 100
m tīklu centrus, kas atrodas izvēlētajos kvadrātos. Aprēķiniet 3 km
buferzonas ap ik centru

``` r
rm(list = ls()) #Vieglāk sākt pa jaunam visu
gc()
```

    ##            used  (Mb) gc trigger   (Mb)  max used   (Mb)
    ## Ncells  3231434 172.6   14590968  779.3  33861745 1808.5
    ## Vcells 13886519 106.0  139005008 1060.6 173642355 1324.8

``` r
tks1km_sauszeme <- st_as_sf(arrow::read_parquet("../Uzd03/HiQBioDiv_vector_reference_grids/tikls1km_sauzeme.parquet"))
LAD_raster_10m <- rast("../Uzd03/LAD_raster.tif")
LAD_raster_100m <- rast("../Uzd03/LAD_raster_100m_sum.tif")

#Atļaušos izvēlēties QGIS'ā 10 karšu lapas un ielasīt tā eksportu. 
izveletas_1km_lapas <- st_read_parquet("../Uzd07/izveletas_1km_lapas.parquet")

pts_crs <- st_crs(st_read_parquet("../Uzd03/HiQBioDiv_vector_reference_grids/pts300_sauzeme.parquet"), izveletas_1km_lapas)
st_crs(izveletas_1km_lapas) <- st_crs(pts_crs)

pts100_vajadzigie <- st_intersection(st_read_parquet("../Uzd03/HiQBioDiv_vector_reference_grids/pts100_sauzeme.parquet"), izveletas_1km_lapas)
pts300_vajadzigie <- st_intersection(st_read_parquet("../Uzd03/HiQBioDiv_vector_reference_grids/pts300_sauzeme.parquet"), izveletas_1km_lapas)

pts100_buffer <- st_buffer(pts100_vajadzigie, dist = 3000)
pts300_buffer <- st_buffer(pts300_vajadzigie, dist = 3000)

#Nākamajā uzdevumā vajag arī 100x100 tīķlu, pievienoju vajadzīgās
ref_vector_100x100_vajadzigie <- read_parquet("../Uzd03/HiQBioDiv_vector_reference_grids/tikls100_sauzeme.parquet")
ref_vector_100x100_vajadzigie <- ref_vector_100x100_vajadzigie %>%
                                      filter(ref_vector_100x100_vajadzigie$ID1km %in% izveletas_1km_lapas$ID1km)

ref_vector_100x100_vajadzigie <- st_as_sf(ref_vector_100x100_vajadzigie)
st_crs(ref_vector_100x100_vajadzigie) <- st_crs(pts_crs)
st_crs(pts100_buffer) <- st_crs(pts_crs)
st_crs(pts300_buffer) <- st_crs(pts_crs)
crs(LAD_raster_10m) <- st_crs(pts_crs)$wkt
crs(LAD_raster_100m) <- st_crs(pts_crs)$wkt
```

<br> Definēšu 4 funkcijas ar atšķirīgiem ievades parametriem un mērogiem

``` r
fun_pts100_10m <- function(pts100_buffer, LAD_raster_10m, ref_vector_100x100_vajadzigie) {
  pts100_buffer$pts100_r10m <- exact_extract(LAD_raster_10m, pts100_buffer, fun = "mean")
  ref_vector_100x100_vajadzigie <- left_join(
    ref_vector_100x100_vajadzigie, 
    select(as.data.frame(pts100_buffer), id, pts100_r10m), 
    by = "id"
  )
} 

fun_pts100_100m <- function(pts100_buffer, LAD_raster_100m, ref_vector_100x100_vajadzigie) {
  pts100_buffer$pts100_r100m <- exact_extract(LAD_raster_100m, pts100_buffer, fun = "mean")
  ref_vector_100x100_vajadzigie <- left_join(
    ref_vector_100x100_vajadzigie, 
    select(as.data.frame(pts100_buffer), id, pts100_r100m),
    by = "id"
  )
}

fun_pts300_10m <- function(pts300_buffer, LAD_raster_10m, ref_vector_100x100_vajadzigie) {
  pts300_buffer$pts300_r10m <- exact_extract(LAD_raster_10m, pts300_buffer, fun = "mean")
  ref_vector_100x100_vajadzigie <- left_join(
    ref_vector_100x100_vajadzigie, 
    select(as.data.frame(pts300_buffer), rinda300, pts300_r10m),
    by = "rinda300"
  )
} 

fun_pts300_100m <- function(pts300_buffer, LAD_raster_100m, ref_vector_100x100_vajadzigie) {
  pts300_buffer$pts300_r100m <- exact_extract(LAD_raster_100m, pts300_buffer, fun = "mean")
  ref_vector_100x100_vajadzigie <- left_join(
    ref_vector_100x100_vajadzigie, 
    select(as.data.frame(pts300_buffer), rinda300, pts300_r100m), 
    by = "rinda300"
  )
}
```

<br> `microbenchmark` man met nesaprotamas kļūdas ārā, ja norādu vairāk
nekā 1 atkārtošanas reizes(ar 1 viss izpildās bez problēmām). Mēģināju
visādi risināt, nekādīgi nesanāca. Intereses pēc saviem spēkiem
uzrakstīšu laika pārbaudes komandrindas 10 atkārtojumiem katrai
funkcijai, izmantojot stabilu Sys.time().

``` r
laika_tabula <- data.frame(atkartojums = integer(40),izpildes_laiks_sec = numeric(40), fun_nr = integer(40))

# Predefine columns to avoid dynamic creation
ref_vector_100x100_vajadzigie$pts100_r10m <- NA
ref_vector_100x100_vajadzigie$pts100_r100m <- NA
ref_vector_100x100_vajadzigie$pts300_r10m <- NA
ref_vector_100x100_vajadzigie$pts300_r100m <- NA

for (i in 1:10) {
  sak <- Sys.time()
  
  ref_vector_100x100_vajadzigie$pts100_r10m <- fun_pts100_10m(pts100_buffer, LAD_raster_10m, ref_vector_100x100_vajadzigie)$pts100_r10m
  
  end <- Sys.time()
  starp <- as.numeric(difftime(end, sak, units = "secs"))
  
  laika_tabula$atkartojums[i] <- i
  laika_tabula$izpildes_laiks_sec[i] <- starp
  laika_tabula$fun_nr[i] <- 1
}

for (i in 11:20) {
  sak <- Sys.time()
  
  ref_vector_100x100_vajadzigie$pts100_r100m <- fun_pts100_100m(pts100_buffer, LAD_raster_100m, ref_vector_100x100_vajadzigie)$pts100_r100m
  
  end <- Sys.time()
  starp <- as.numeric(difftime(end, sak, units = "secs"))
  
  laika_tabula$atkartojums[i] <- i
  laika_tabula$izpildes_laiks_sec[i] <- starp
  laika_tabula$fun_nr[i] <- 2
}

for (i in 21:30) {
  sak <- Sys.time()
  
  ref_vector_100x100_vajadzigie$pts300_r10m <- fun_pts300_10m(pts300_buffer, LAD_raster_10m, ref_vector_100x100_vajadzigie)$pts300_r10m
  
  end <- Sys.time()
  starp <- as.numeric(difftime(end, sak, units = "secs"))
  
  laika_tabula$atkartojums[i] <- i
  laika_tabula$izpildes_laiks_sec[i] <- starp
  laika_tabula$fun_nr[i] <- 3
}

for (i in 31:40) {
  sak <- Sys.time()
  
  ref_vector_100x100_vajadzigie$pts300_r100m <- fun_pts300_100m(pts300_buffer, LAD_raster_100m, ref_vector_100x100_vajadzigie)$pts300_r100m
  
  end <- Sys.time()
  starp <- as.numeric(difftime(end, sak, units = "secs"))
  
  laika_tabula$atkartojums[i] <- i
  laika_tabula$izpildes_laiks_sec[i] <- starp
  laika_tabula$fun_nr[i] <- 4
}

#Reizinu ar 100 salīdzināmām vērtībām
ref_vector_100x100_vajadzigie$pts300_r10m <- ref_vector_100x100_vajadzigie$pts300_r10m * 100
ref_vector_100x100_vajadzigie$pts100_r10m <- ref_vector_100x100_vajadzigie$pts100_r10m * 100
```

``` r
aggregate(izpildes_laiks_sec ~ fun_nr, data = laika_tabula, summary)
```

    ##   fun_nr izpildes_laiks_sec.Min. izpildes_laiks_sec.1st Qu.
    ## 1      1              9.28632593                 9.54521346
    ## 2      2              0.34383583                 0.35084593
    ## 3      3              1.06520700                 1.06785744
    ## 4      4              0.06158781                 0.06405008
    ##   izpildes_laiks_sec.Median izpildes_laiks_sec.Mean izpildes_laiks_sec.3rd Qu.
    ## 1                9.57132661              9.57398818                 9.67436391
    ## 2                0.36204791              0.39283240                 0.43796378
    ## 3                1.08339942              1.09569480                 1.11383873
    ## 4                0.06511557              0.06511259                 0.06667352
    ##   izpildes_laiks_sec.Max.
    ## 1              9.74601793
    ## 2              0.49432015
    ## 3              1.15964794
    ## 4              0.06788087

``` r
cat("Maksimālā starpība starp 300 metru punktu aprēķiniem:", max(abs(ref_vector_100x100_vajadzigie$pts300_r100m - ref_vector_100x100_vajadzigie$pts300_r10m), na.rm = TRUE))
```

    ## Maksimālā starpība starp 300 metru punktu aprēķiniem: 0.04408038

``` r
cat("Maksimālā starpība starp 100 metru punktu aprēķiniem:", max(abs(ref_vector_100x100_vajadzigie$pts100_r100m - ref_vector_100x100_vajadzigie$pts100_r10m), na.rm = TRUE))
```

    ## Maksimālā starpība starp 100 metru punktu aprēķiniem: 0.0562883

``` r
cat("Maksimālā starpība starp 300 un 100 metru punktu aprēķiniem 10 metru izšķirtspējā:", max(abs(ref_vector_100x100_vajadzigie$pts100_r10m - ref_vector_100x100_vajadzigie$pts300_r10m), na.rm = TRUE))
```

    ## Maksimālā starpība starp 300 un 100 metru punktu aprēķiniem 10 metru izšķirtspējā: 1.029445

``` r
cat("Maksimālā starpība starp 300 un 100 metru punktu aprēķiniem 100 metru izšķirtspējā:", max(abs(ref_vector_100x100_vajadzigie$pts100_r100m - ref_vector_100x100_vajadzigie$pts300_r100m), na.rm = TRUE))
```

    ## Maksimālā starpība starp 300 un 100 metru punktu aprēķiniem 100 metru izšķirtspējā: 1.026475

<br> Visātrāk bija rēķināt 300 metru punktu buferiem un 100 metru rastra
izšķirtspējā. 300 metros punktiem un 10 metru rastram rezultāts arī ir
diezgan ātri, bet problēma ka ir daudzas NA vērtības.  
<br> Maksimālā starpība visos gadījumos sanāk ir max 1%, neatkarīgi no
punktu attāluma un rastru izšķirtspējas. Lai nebūtu NA tad rēķina 100
metru punktiem un 100 metru rastram (protams ja mērķis ir raksturot
īpatsvaru 100 metru vektorā). <br> Tendences īsti neredzu - ir poligoni
kuros 300 metru aprēķini rāda mazākas vērtības, ir, kur lielākas, ir arī
kur sakrīt…
