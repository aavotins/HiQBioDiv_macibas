Piektais uzdevums: procesu dalīšana un rezultātu apvienošana
================
Jekaterīna Butkeviča
2025. gads 29. jūnijs

## Pakotnes

Izlaboju iepriekšējo kļūdu un pievienoju pārbaudi.

## Datu sagatavošana

Izlaboju kļūdu 2. uzdevumā, kad netika atlasīti tieši mežaudžu ieraksti.

``` r
centra_mezi <- st_read_parquet(here("3uzd","Output","Data","centra_apvienota.parquet"))
centra_mezi <- centra_mezi[centra_mezi$zkat == 10, ]
st_write_parquet(centra_mezi, here("3uzd","Output","Data","centra_apvienota.parquet"))

karte <- st_read_parquet(here("3uzd","Data","references_zenodo", "vektors", "tks93_50km.parquet"))
```

Kartes lapas izvēlos, izmantojot leaflet pakotni, kas ļauj atvērt
interaktīvu karti cilnē “Viewer”. Izvēlējos lapas ar numuriem 4321,
3343, 3334 un 4312.

Pārveidoju lapu datu tabulu sarakstā, sadalot pa lapām pēc to numura.
Katru lapu atsevišķi saglabāju diskā, iekļaujot lapas numuru faila
nosaukumā. Lai turpmāk šie faili būtu izmantojami un saistāmi ar MVR
datiem, atsevišķā mainīgajā ierakstīju arī ceļus uz failiem.

``` r
izveletas_lapas <- karte %>% filter(NUMURS %in% c(4321, 3343, 3334, 4312))

# Lai strādātu atsevišķi ar katru vienību, pārveidoju to par sarakstu.
izveletas_lapas_saraksts <- split(izveletas_lapas, izveletas_lapas$NUMURS)

output_dir <- here("5uzd","Output", "kartes_lapas")
dir.create(output_dir, recursive = TRUE)

# Saglabāt lapas
iwalk(izveletas_lapas_saraksts, ~ {
  file_name <- file.path(output_dir, paste0("lapa", .y, ".parquet"))
  st_write_parquet(.x, file_name)
})

# Iegūt ceļus iz visām lapām
lapas_sarakasts_celi <- list.files(output_dir, pattern = "^lapa\\d{4}\\.parquet$", full.names = TRUE)
```

Izveidoju apakšdirektoriju failu kārtībai:

``` r
output_dir <- here("5uzd","Output", "rezultati")
dir.create(output_dir, recursive = TRUE)
```

Ceturtā uzdevuma izveidoto funkciju ir nepieciešams uzlabot un pielāgot
šim uzdevumam.

``` r
mana_funkcija2 <- function(mezaudzes, uzdevums) {

    # Ielādēt MVR datus no Parquet faila
    MVR_dati <- st_read_parquet(mezaudzes)
    
    # izvilkt lapas numuru
    lapas_nr <- str_extract(mezaudzes, "(?<=lapa)\\d+")
    
    # References slāņi un to apgriešana pēc minimālā nepieciešamā taisnstūra (extent)
    reference_10m <- rast(here("3uzd","Data", "references_zenodo", "rastrs", "LV10m_10km.tif"))
    reference_10m <- crop(reference_10m, MVR_dati)
    reference_100m <- rast(here("3uzd","Data", "references_zenodo", "rastrs", "LV100m_10km.tif"))
    reference_100m <- crop(reference_100m, MVR_dati)
    
    # Veidojam faila nosaukumu un ceļus
    uzd_nr <- paste0("uzd", uzdevums)
    
    nosaukums <- paste0("lapa", lapas_nr,"_", uzd_nr, "_priezu_ipatsvars_100m.tif")
    
    uzd_dir <- here("5uzd", "Output", "rezultati", uzd_nr, "ipatsvars")
    dir.create(uzd_dir, recursive = TRUE, showWarnings = FALSE)
    
    faila_dir <- file.path(uzd_dir, nosaukums)
    
    # Atlasīt tikai audzes, kurās valdošā koku suga ir priede (sugas kods “1”)
    priede_mezaudzes <- MVR_dati %>% dplyr::filter(s10 == "1")
    
    # "Pārveidot mežu vektoru par rastra formātu ar 10 metru izšķirtspēju"
    rast_priezu_10m <- rasterize(priede_mezaudzes, reference_10m, background = 0)
    
    # Nodrošināt NA vērtības ārpus Latvijas
    rast_priezu_10m[is.na(reference_10m)] <- NA
    
    
    # Samazināt izšķirtspēju līdz 100 m, saskaitot "1" šūnas
    rast_priezu_100m <- resample(rast_priezu_10m, 
                                        reference_100m,
                                        method = "average",
                                        filename = faila_dir, 
                                        overwrite = TRUE)
    }
```

## Pirmais apakšuzdevums

1.1. solis: izmantojot spatial join saistiet MVR datus ar izvēlētajām
karšu lapām (šis ir sākums laika mērogošanai). Kāds ir objektu skaits
pirms un pēc savienošanas? Apskatiet katrai kartes lapai piederīgos
objektus - vai tie atrodas tikai kartes lapas iekšienē, vai ir iekļauti
visi objekti, kas ir uz kartes lapas robežām?

1.2. solis: iteratīvā ciklā izmantojiet pārskatīto funkciju no uzdevuma
sākuma, lai saglabātu katras karšu lapas rezultātu savā GeoTIFF failā,
kura nosaukums ir saistāms ar šo apakšuzdevumu un individuālo lapu. Kā
izskatās šūnu aizpildījums pie karšu lapu malām? Vai ir saglabājušies
objekti ārpus kartes lapām?

Izskātas, kā ērtak būs darīt visu pakapeniski un ar komentāriem un tad
apvienot un vēlreiz ielaist lai reģistrēt izpildes laiku.

### 1.1. solis

Apakšdirektorija

``` r
output_dir <- here("5uzd","Output", "rezultati", "uzd1", "lapas")
dir.create(output_dir, recursive = TRUE)
```

Sanāca pielāgot koordinātu sistēmu, jo tās atšķīrās. Pēc cikla izveidoju
sarakstu ar ceļiem uz failiem. Funkcija *st_join()* atgriež visus
elementus, kas vismaz daļēji atrodas kartes lapā.

``` r
for (katrs in lapas_sarakasts_celi) {
  lapa <- st_read_parquet(katrs)
  lapa <- st_transform(lapa, st_crs(centra_mezi))
  apvienots <- st_join(centra_mezi, lapa, left = FALSE)
  lapas_nosaukums <- file_path_sans_ext(basename(katrs))
  fails_nosaukums  <- file.path(output_dir, paste0(lapas_nosaukums,"mezaudzes_1uzd.parquet"))
  st_write_parquet(apvienots, fails_nosaukums )
}

# failu saraksts
mezu_lapas_celi_1uzd <- list.files(output_dir, 
                                   pattern = "^lapa\\d{4}mezaudzes_1uzd\\.parquet$", 
                                   full.names = TRUE)
```

Pārbaudu objektu skaitu. Sākumā Centrālajai mežniecībai ir 462 440
ieraksti. Atsevišķām lapām ir 30 279, 19 510, 32 319 un 24 449 ieraksti.

``` r
# Objektu skaits
mezu_lapas <- map(mezu_lapas_celi_1uzd, st_read_parquet)
nrow(centra_mezi) # Sākumā 462440
```

    ## [1] 462440

``` r
map_int(mezu_lapas, nrow) # lapām ir 30279, 19510, 32319, 24449
```

    ## [1] 30279 19510 32319 24449

Atradu dažādus veidus vizualizācijai: funkcija plot() ir ātrs
risinājums, taču tajā ir grūti novērtēt, cik labi pieguļ lapas. Pakotne
leaflet ļauj apskatīt interaktīvu karti, tomēr pārvietošana ir ļoti
lēna, jo tā prasa daudz datorresursu. Visērtāk man bija paralēli
skatīties QGIS. Šeit ievietošu ātrākas vizualizācijas.

``` r
#Vizualizācija katrai lapai:

#walk(mezu_lapas_celi_1uzd, function(cels) {
#lapa <- st_read_parquet(cels)
#  plot(
#    st_geometry(lapa),
#    main = basename(cels)
#  )
#})

# Pārklāt lapu ar atlasītajām mežaudzēm.

ref_lapa3334 <- st_transform(izveletas_lapas_saraksts[["3334"]], crs = 4326)

lapa3334_mazaudzes_uzd1 <- st_read_parquet(here("5uzd","Output", "rezultati", "uzd1", "lapas", 
                                     "lapa3334mezaudzes_1uzd.parquet"))
lapa3334_mazaudzes_uzd1 <- st_transform(lapa3334_mazaudzes_uzd1, crs = 4326)

# atrāks
plot(st_geometry(ref_lapa3334), col = "lightblue")
plot(st_geometry(lapa3334_mazaudzes_uzd1), col = "darkgreen", add = TRUE)
```

![](Uzd05_Butkevica_files/figure-gfm/vizualizacija1-1.png)<!-- -->

Attēlā redzams, ka ir iekļauti arī tie poligoni, kas atrodas lapas
robežās. Šis rezultāts atbilst izmantotajai funkcijai.

### 1.2. solis

Iteratīvi izmantoju manu funkciju uz katras lapas atsevišķi.

``` r
walk(mezu_lapas_celi_1uzd, 
     ~ mana_funkcija2(mezaudzes = .x, uzdevums = "1"))
```

Apskatīju QGISā. Pie malas vērtības visur ir zemas. Tā kā poligonu
platība ārpus karšu lapas bija atšķirīga, tagad rastriem ir izveidojušās
dažāda izmēra malas. Radās sajūta, ka pikseļi, kas attiecas uz divām
lapām, ir ar atšķirīgām vērtībām, tātad pie malas vērtības atšķiras
starp karšu lapām.

### 1.3. solis

Veidoju sarakstu ar visiem failu ceļiem, ielasu datus un sagatavoju lapu
sarakstu.

``` r
# Saraksts ar visu izveidoto failu ceļiem, lai vēlāk ielikt ciklā laika mērīšanai
uzd1_priedzu_ipatsvars_celi <- list.files(here("5uzd", "Output", "rezultati", "uzd1", "ipatsvars" ), 
                                          pattern = "^lapa\\d{4}_uzd1_priezu_ipatsvars_100m\\.tif$", full.names = TRUE)
uzd1_ipatsvara_lapas <- map(uzd1_priedzu_ipatsvars_celi, rast)
```

Es izmantoju funkciju *reduce()*, jo tā secīgi pielieto norādīto
funkciju pāriem saraksta elementu. Šajā gadījumā *reduce()* kopā ar
*mosaic()* (un argumentu fun = max) ļauj pakāpeniski apvienot vairākus
rastrus, vienlaikus taupot operatīvo atmiņu, jo vienlaikus tiek
apstrādāti tikai divi rastri. Arguments fun = max nodrošina, ka
apvienojot rastrus, tiek izvēlēta lielākā vērtība katrā šūnā, kas,
manuprāt, ir objektīvāks risinājums situācijā, kad pārklājas vairākas
“māliņas” (piemēram, rastri). Pretējā gadījumā viena rastra vērtība
varētu tikt aizvietota ar citu, kas, manuprāt, būtu sliktāks risinājums.

``` r
uzd1_ipatsvara_lapas_apvienotas <- reduce(uzd1_ipatsvara_lapas, mosaic, fun = max)
par(mfrow = c(1, 1))

plot(uzd1_ipatsvara_lapas_apvienotas)
```

![](Uzd05_Butkevica_files/figure-gfm/reduce_mosaic-1.png)<!-- -->

Vizuālizācija redzama dažādu platību māla kāršu lapu ārējā robežā.

``` r
writeRaster(uzd1_ipatsvara_lapas_apvienotas, 
            filename = here("5uzd", "Output", "rezultati", "uzd1", "Uzd1_apvienots.tif"),
            overwrite = TRUE)
```

Apskatot QGISā apvienoto rastru, saduras vietas vairs neizceļas ar zemām
vērtībām.

## Laika mērīšana pirmā uzdevumā

``` r
start_time <- Sys.time()


# 1.1. solis:
for (katrs in lapas_sarakasts_celi) {
  lapa <- st_read_parquet(katrs)
  lapa <- st_transform(lapa, st_crs(centra_mezi))
  apvienots <- st_join(centra_mezi, lapa, left = FALSE)
  lapas_nosaukums <- file_path_sans_ext(basename(katrs))
  fails_nosaukums  <- file.path(output_dir, paste0(lapas_nosaukums,"mezaudzes_1uzd.parquet"))
  st_write_parquet(apvienots, fails_nosaukums )
}


mezu_lapas_celi_1uzd <- list.files(output_dir, 
                                   pattern = "^lapa\\d{4}mezaudzes_1uzd\\.parquet$", 
                                   full.names = TRUE)

# 1.2. solis:
walk(mezu_lapas_celi_1uzd, 
     ~ mana_funkcija2(mezaudzes = .x, uzdevums = "1"))


# 1.3. solis:
uzd1_priedzu_ipatsvars_celi <- list.files(here("5uzd", "Output", "rezultati", "uzd1", "ipatsvars"), 
                                          pattern = "^lapa\\d{4}_uzd1_priezu_ipatsvars_100m\\.tif$", full.names = TRUE)
uzd1_ipatsvara_lapas <- map(uzd1_priedzu_ipatsvars_celi, rast)


uzd1_ipatsvara_lapas_apvienotas <- reduce(uzd1_ipatsvara_lapas, mosaic, fun = max)
#Saglabāt

writeRaster(uzd1_ipatsvara_lapas_apvienotas, 
            filename = here("5uzd", "Output", "rezultati", "uzd1", "Uzd1_apvienots.tif"),
            overwrite = TRUE)



# Beigu laiks un ilgums
end_time <- Sys.time()
cat("Visu soļu izpildes ilgums:", round(difftime(end_time, start_time, units = "secs"), 2), "sekundes\n")
```

    ## Visu soļu izpildes ilgums: 21.67 sekundes

## Otrais apakšuzdevums

### 2.1. solis

Apakšdirektorija

``` r
output_dir2 <- here("5uzd","Output", "rezultati", "uzd2", "lapas")
dir.create(output_dir2, recursive = TRUE)
```

Funkcija *st_intersection()* nodrošina griešanas (clipping) pieeju. Ar
izmantotajiem iestatījumiem funkcija atgriež tikai to ģeometrijas daļu,
kas pārklājas ar attiecīgo malu, un poligoni, kas atrodas uz robežas,
tiks apgriezti pēc lapas malas.

``` r
for (katrs in lapas_sarakasts_celi) {
  lapa <- st_read_parquet(katrs)
  lapa <- st_transform(lapa, st_crs(centra_mezi))
  apvienots <- st_intersection(centra_mezi, lapa, left = FALSE)
  lapas_nosaukums <- file_path_sans_ext(basename(katrs))
  fails_nosaukums  <- file.path(output_dir2, paste0(lapas_nosaukums,"mezaudzes_2uzd.parquet"))
  st_write_parquet(apvienots, fails_nosaukums )
}



mezu_lapas_celi_2uzd <- list.files(output_dir2, 
                                   pattern = "^lapa\\d{4}mezaudzes_2uzd\\.parquet$", 
                                   full.names = TRUE)
```

Pārbaudu objektu skaitu.

``` r
mezu_lapas <- map(mezu_lapas_celi_2uzd, st_read_parquet)
map_int(mezu_lapas, nrow) # lapām ir 30279, 19510, 32319, 24449
```

    ## [1] 30279 19510 32319 24449

### 2.2. solis

Iteratīvi izmantoju funkciju manu_funkciju() uz MVR lapām.

``` r
walk(mezu_lapas_celi_2uzd, 
     ~ mana_funkcija2(mezaudzes = .x, uzdevums = "2"))
```

Apskatīju rezultātus QGISā. Pie malām vērtības ir normālas — neveidojas
zemu vērtību josla, kas atbilst funkcijas darbībai, jo ģeometrijas tika
apgrieztas līdz lapas malai. Tā kā ģeometrijas tika apgrieztas,
neveidojas atšķirīga platuma josla kā iepriekšējā apakšuzdevumā, un
tāpēc lapas ārējā mala izskatās vienmērīgāka.

### 2.3. solis

Veidoju sarakstu ar ceļiem uz visiem failiem, ielasu datus un izveidoju
lapu sarakstu.

``` r
# Saraksts ar visu izveidoto failu ceļiem, lai vēlāk ievietotu ciklā laika mērīšanai.
uzd2_priedzu_ipatsvars_celi <- list.files(here("5uzd", "Output", "rezultati", "uzd2", "ipatsvars"), 
                                          pattern = "^lapa\\d{4}_uzd2_priezu_ipatsvars_100m\\.tif$", full.names = TRUE)
uzd2_ipatsvara_lapas <- map(uzd2_priedzu_ipatsvars_celi, rast)
```

Es izmantoju funkciju *reduce()*, jo tā secīgi pielieto norādīto
funkciju pāriem no saraksta elementiem. Savukārt izmantoju vienkāršu
*merge()* funkciju, jo nav vajadzības izvēlēties starp dažādām lapu
vērtībām. Tādēļ *merge()* arī būtu ātrāka.

``` r
uzd2_ipatsvara_lapas_apvienotas <- reduce(uzd2_ipatsvara_lapas, 
                                          merge)
```

Saglabāju rezultātu.

``` r
writeRaster(uzd2_ipatsvara_lapas_apvienotas, 
            filename = here("5uzd", "Output", "rezultati", "uzd2", "Uzd2_apvienots.tif"),
            overwrite = TRUE)
```

## Laika mērīšana otrā uzdevumā

``` r
# Starta laiks
start_time <- Sys.time()

# 2.1. solis:
for (katrs in lapas_sarakasts_celi) {
  lapa <- st_read_parquet(katrs)
  lapa <- st_transform(lapa, st_crs(centra_mezi))
  apvienots <- st_intersection(centra_mezi, lapa, left = FALSE)
  lapas_nosaukums <- file_path_sans_ext(basename(katrs))
  fails_nosaukums  <- file.path(output_dir2, paste0(lapas_nosaukums,"mezaudzes_2uzd.parquet"))
  st_write_parquet(apvienots, fails_nosaukums )
  }



mezu_lapas_celi_2uzd <- list.files(output_dir2, 
                                   pattern = "^lapa\\d{4}mezaudzes_2uzd\\.parquet$", 
                                   full.names = TRUE)



# 2.2. solis:
walk(mezu_lapas_celi_2uzd, 
     ~ mana_funkcija2(mezaudzes = .x, uzdevums = "2"))



# 2.3. solis:
uzd2_priedzu_ipatsvars_celi <- list.files(here("5uzd", "Output", "rezultati", "uzd2", "ipatsvars"), 
                                         pattern = "^lapa\\d{4}_uzd2_priezu_ipatsvars_100m\\.tif$", full.names = TRUE)
uzd2_ipatsvara_lapas <- map(uzd2_priedzu_ipatsvars_celi, rast)


uzd2_ipatsvara_lapas_apvienotas <- reduce(uzd2_ipatsvara_lapas, 
                                          merge)


#Saglabāt
writeRaster(uzd2_ipatsvara_lapas_apvienotas, 
            filename = here("5uzd", "Output", "rezultati", "uzd2", "Uzd2_apvienots.tif"),
            overwrite = TRUE)

end_time <- Sys.time()
cat("Visu soļu izpildes ilgums:", round(difftime(end_time, start_time, units = "secs"), 2), "sekundes\n")
```

    ## Visu soļu izpildes ilgums: 22.81 sekundes

## Trešais apakšuzdevums

Apakšdirektorija

``` r
output_dir3 <- here("5uzd","Output", "rezultati", "uzd3", "lapas")
dir.create(output_dir3, recursive = TRUE)
```

### 3.1. solis

Funkcijai *st_filter()* ir pieejami dažādi attiecību nosacījumi, kas
ļauj strādāt ar ģeometriju savstarpējās attiecībām. Arguments st_within
nodrošina, ka tiek atlasītas tikai tās ģeometrijas, kas pilnībā ietilpst
kartes lapā. Izvēlējos šo variantu, jo nevēlējos redzēt nevienmērīgu
josliņu apkārt lapai, kā tas bija pirmajā apakšuzdevumā. Tomēr sagaidu,
ka ap lapas malām var veidoties vērtību caurumi, jo poligoni, kas
atrodas uz malas, netiks pieskaitīti nevienai no lapām.

``` r
for (katrs in lapas_sarakasts_celi) {
  lapa <- st_read_parquet(katrs)
  lapa <- st_transform(lapa, st_crs(centra_mezi))
  apvienots <- st_filter(centra_mezi, lapa, 
                         .predicate = st_within)
  lapas_nosaukums <- file_path_sans_ext(basename(katrs))
  fails_nosaukums  <- file.path(output_dir3, paste0(lapas_nosaukums,"mezaudzes_3uzd.parquet"))
  st_write_parquet(apvienots, fails_nosaukums )
}


mezu_lapas_celi_3uzd <- list.files(output_dir3, 
                                   pattern = "^lapa\\d{4}mezaudzes_3uzd\\.parquet$", 
                                   full.names = TRUE)
```

Pārbaudu objektu skaitu.

``` r
mezu_lapas <- map(mezu_lapas_celi_2uzd, st_read_parquet)
map_int(mezu_lapas, nrow) # lapām ir 29548, 18972, 31561, 23854
```

    ## [1] 30279 19510 32319 24449

### 3.2. solis

Iteratīvi izmantoju funkciju *manu_funkciju()* MVR lapām.

``` r
walk(mezu_lapas_celi_3uzd, 
     ~ mana_funkcija2(mezaudzes = .x, uzdevums = "3"))
```

Apskatot rezultātus QGIS, jā – kā arī bija gaidāms, pie karšu malu malām
veidojas zemas vērtības josla, jo ģeometrijas tika nomestas.

### 3.3. solis

Veidoju sarakstu ar ceļiem uz visiem failiem, ielasu datus un izveidoju
lapu sarakstu.”

``` r
uzd3_priedzu_ipatsvars_celi <- list.files(here("5uzd", "Output", "rezultati", "uzd3", "ipatsvars"), 
                                          pattern = "^lapa\\d{4}_uzd3_priezu_ipatsvars_100m\\.tif$", full.names = TRUE)
uzd3_ipatsvara_lapas <- map(uzd3_priedzu_ipatsvars_celi, rast)
```

Šeit bez izmaiņām.

``` r
uzd3_ipatsvara_lapas_apvienotas <- reduce(uzd3_ipatsvara_lapas, 
                                          merge)
```

Un saglabāšana.

``` r
writeRaster(uzd3_ipatsvara_lapas_apvienotas, 
            filename = here("5uzd", "Output", "rezultati", "uzd3", "Uzd3_apvienots.tif"),
            overwrite = TRUE)
```

Apskatīju QGIS, un kā sagaidāms, zemo vērtību josla pie lapas robežām
saglabājās.

## Laika mērīšana trešā uzdevumā

``` r
start_time <- Sys.time()

for (katrs in lapas_sarakasts_celi) {
  lapa <- st_read_parquet(katrs)
  lapa <- st_transform(lapa, st_crs(centra_mezi))
  apvienots <- st_filter(centra_mezi, lapa, 
                         .predicate = st_within)
  lapas_nosaukums <- file_path_sans_ext(basename(katrs))
  fails_nosaukums  <- file.path(output_dir3, paste0(lapas_nosaukums,"mezaudzes_3uzd.parquet"))
  st_write_parquet(apvienots, fails_nosaukums )
}


mezu_lapas_celi_3uzd <- list.files(output_dir3, 
                                   pattern = "^lapa\\d{4}mezaudzes_3uzd\\.parquet$", 
                                   full.names = TRUE)



# 3.2. solis:
walk(mezu_lapas_celi_3uzd, 
     ~ mana_funkcija2(mezaudzes = .x, uzdevums = "3"))



# 3.3. solis:
uzd3_priedzu_ipatsvars_celi <- list.files(here("5uzd", "Output", "rezultati", "uzd3", "ipatsvars"), 
                                          pattern = "^lapa\\d{4}_uzd3_priezu_ipatsvars_100m\\.tif$", full.names = TRUE)
uzd3_ipatsvara_lapas <- map(uzd3_priedzu_ipatsvars_celi, rast)


uzd3_ipatsvara_lapas_apvienotas <- reduce(uzd3_ipatsvara_lapas, 
                                          merge)


#Saglabāt
writeRaster(uzd3_ipatsvara_lapas_apvienotas, 
            filename = here("5uzd", "Output", "rezultati", "uzd3", "Uzd3_apvienots.tif"),
            overwrite = TRUE)



end_time <- Sys.time()
cat("Visu soļu izpildes ilgums:", round(difftime(end_time, start_time, units = "secs"), 2), "sekundes\n")
```

    ## Visu soļu izpildes ilgums: 18.74 sekundes

## Ceturtais apakšuzdevums

Apakšdirektorija

``` r
output_dir4 <- here("5uzd","Output", "rezultati", "uzd4", "lapas")
dir.create(output_dir4, recursive = TRUE)

ip_dir4 <- here("5uzd","Output", "rezultati", "uzd4", "ipatsvars")
dir.create(ip_dir4, recursive = TRUE)
```

### 4.1 solis

Drošības pēc atkārtoju lapu izvēli, pielāgoju koordinātu sistēmu, un
karšu lapu savienošanai ar MVR datiem izmantoju *st_intersection()*, jo
šis risinājums deva vislabākos rezultātus.

``` r
izveletas_lapas <- karte %>% filter(NUMURS %in% c(4321, 3343, 3334, 4312))

izveletas_lapas <- st_transform(izveletas_lapas, st_crs(centra_mezi))
apvienots_uzd4 <- st_intersection(centra_mezi, izveletas_lapas)


output_file4 <- file.path(output_dir4, "apvienots_uzd4.parquet")
st_write_parquet(apvienots_uzd4, output_file4)
```

Šīm uzdevumam ir nepieciešams pielāgot *manu_funkciju()*.

``` r
mana_funkcija_4uzd <- function(mezaudzes) {
  
  # Ielasīt MVR datus no GeoParquet faila
  MVR_dati <- st_read_parquet(mezaudzes)
  

  
  # references slāņi un to apgriešana pēc minimāla nepieciešama taisnstūra (extent)
  reference_10m <- rast(here("3uzd","Data", "references_zenodo", "rastrs", "LV10m_10km.tif"))
  reference_10m <- crop(reference_10m, MVR_dati)
  reference_100m <- rast(here("3uzd","Data", "references_zenodo", "rastrs", "LV100m_10km.tif"))
  reference_100m <- crop(reference_100m, MVR_dati)
  
 
  # Atlasīt tikai audzes, kur valdošā koku suga ir priede (sugas kods ir “1”)
  priede_mezaudzes <- MVR_dati %>% dplyr::filter(s10 == "1")
  
  # Rastrēt mežu vektoru uz 10 metru izšķirtspēju
  rast_priezu_10m <- rasterize(priede_mezaudzes, reference_10m, background = 0)
  
  # Nodrošināt NA ārpus LAtvijas
  rast_priezu_10m[is.na(reference_10m)] <- NA
  
  
  # 6. Samazināt izšķirtspēju uz 100m, summējot "1" šūnas
  rast_priezu_100m <- resample(rast_priezu_10m, 
                               reference_100m,
                               method = "average",
                               filename = here("5uzd","Output", "rezultati", "uzd4", "ipatsvars", "uzd4_priezu_ipatsvars_100m.tif"), 
                               overwrite = TRUE)
}
```

### 4.2 solis

Pielietoju manu *funkciju()*.

``` r
mana_funkcija_4uzd(output_file4)
```

Apskatot rezultātus QGIS, redzams, ka pie lapu malām nav nekādu
problēmu, jo mežaudzes netika sadalītas, un īpatsvars tiek aprēķināts
kopā visu lapu ietvaros.

## Laika mērīšana ceturtā uzdevumā

``` r
start_time <- Sys.time()


# 4.1. solis:
apvienots_uzd4 <- st_intersection(centra_mezi, izveletas_lapas)
nrow(apvienots_uzd4) #106557
```

    ## [1] 106557

``` r
output_file4 <- file.path(output_dir4, "apvienots_uzd4.parquet")
st_write_parquet(apvienots_uzd4, output_file4)

# 4.2. solis:
mana_funkcija_4uzd(output_file4)

end_time <- Sys.time()
cat("Visu soļu izpildes ilgums:", round(difftime(end_time, start_time, units = "secs"), 2), "sekundes\n")
```

    ## Visu soļu izpildes ilgums: 17.13 sekundes

## Piektais apakšuzdevums

Mēģinot izpildīt šo uzdevumu, sastapos ar kļūdu: Error: \[rasterize\]
too many values for writing: 233099035 \> 156879470. Pašai to neizdevās
atrisināt. Esmu uzrakstījusi mūsu diskusijas formā, gaidu gaidu Jūsu
ieteikumus.

Apakšdirektorija

``` r
# Es nesaprotu vai visa MVR ir domāti tikai Centra virsmežniecīabs dati? Laika taupīšanai izmantošu tos
ip_dir5 <- here("5uzd","Output", "rezultati", "uzd5", "ipatsvars")
dir.create(ip_dir5, recursive = TRUE)
```

Arī šim uzdevumam jāmodificē mana_funkcija:

``` r
mana_funkcija_5uzd <- function(mezaudzes) {
  
  # Ielasīt MVR datus no GeoParquet faila
  MVR_dati <- st_read_parquet(mezaudzes)
  
  
  # references slāņi un to apgriešana pēc minimāla nepieciešama taisnstūra (extent)
  reference_10m <- rast(here("3uzd","Data", "references_zenodo", "rastrs", "LV10m_10km.tif"))
  reference_10m <- crop(reference_10m, MVR_dati)
  reference_100m <- rast(here("3uzd","Data", "references_zenodo", "rastrs", "LV100m_10km.tif"))
  reference_100m <- crop(reference_100m, MVR_dati)  
  
    # Atlasīt tikai audzes, kur valdošā koku suga ir priede (sugas kods ir “1”)
  priede_mezaudzes <- MVR_dati %>% dplyr::filter(s10 == "1")
  

  
  # Rastrēt mežu vektoru uz 10 metru izšķirtspēju
  rast_priezu_10m <- rasterize(priede_mezaudzes, 
                               reference_10m, 
                               background = 0)
  
  # Nodrošināt NA ārpus LAtvijas
  rast_priezu_10m[is.na(reference_10m)] <- NA
  
  
  # 6. Samazināt izšķirtspēju uz 100m, summējot "1" šūnas
  rast_priezu_100m <- resample(rast_priezu_10m, 
                               reference_100m,
                               method = "average",
                               filename = here("5uzd","Output", "rezultati", "uzd5", "ipatsvars", "uzd5_priezu_ipatsvars_100m.tif"), 
                               overwrite = TRUE)
  
  priede_mezaudzes_grieztas <- crop(rast(here("5uzd","Output", "rezultati", "uzd5", "ipatsvars", "uzd5_priezu_ipatsvars_100m.tif")), izveletas_lapas,)
  writeRaster(priede_mezaudzes_grieztas, here("5uzd","Output", "rezultati", "uzd5", "ipatsvars", "uzd5_priezu_ipatsvars_100m.tif"), overwrite = TRUE)
}
```

### 5.1. un 5.2. soļi kopā

„Pielietoju funkciju `mana_funkcija()` un uzreiz mēru tās izpildes
laiku.

``` r
start_time <- Sys.time()
mana_funkcija_5uzd(here("3uzd","Output","Data","centra_apvienota.parquet"))
end_time <- Sys.time()
cat("Visu soļu izpildes ilgums:", round(difftime(end_time, start_time, units = "secs"), 2), "sekundes\n")
```

    ## Visu soļu izpildes ilgums: 36.55 sekundes

## Kopsavilkums

Kopā MVR ierkastu: 462440

Manuprāt, visveiksmīgākais rezultāts bija, izmantojot funkciju
`st_intersection`, jo atšķirībā no pārējām pieejām tā nodrošināja
precīzas vērtības arī malās, nevis radīja tukšas joslas vai joslas ar
samazinātām vērtībām. Gan `spatial join`, gan `st_filter` gadījumā ir
redzamas karšu lapu malu problēmas.

### 1. apakšuzdevums

izmantojot `spatial join` lapām ir: 30279, 19510, 32319, 24449

Visu soļu izpildes ilgums: 21,46 sekundes. Malu problēmas ir redzamas.

### 2. apakšuzdevums

izmantojot `st_intersection` lapām ir: 30279, 19510, 32319, 24449 — tas
sakrīt ar iepriekšējā apakšuzdevuma rezultātiem.

Visu soļu izpildes ilgums: 23,68 sekundes. Malu vērtības saglabājas
precīzi.

### 3. apakšuzdevums

izmantojot `st_filter` ar argumentu st_within lapām ir: 29548, 18972,
31561, 23854. Tas ir mazāk nekā citos variantos, jo objekti, kas atrodas
uz lapu robežām, netika pieskaitīti nevienai no lapām. Visu soļu
izpildes ilgums: 23,83 sekundes

### 4.apakšuzdevums

Visu soļu izpildes ilgums: 19,12 sekundes. Šis variants bija visātrākais
un bez malu vērību zudumiem. Sanāk, ka veikt darbības visai teritorijai
uzreiz ir efektīvāk nekā iterēt pa karšu lapām. Turklāt šajā variantā
neveidojas problēmas ar vērtībām gar lapu malām.

### 5. apakšuzdevums

Visu soļu izpildes ilgums: 43.79 sekundes. Šajā gadījumā arī neveidojas
problēmas, kas saistītas ar malām.

Kopumā, salīdzinot pieejas, `st_intersection` nodrošina precīzas
vērtības gar malām, kamēr `spatial join` un `st_filter` malās zaudē
informāciju. Iterācija pa karšu lapām ir lēnāka un var radīt malu
problēmas, salīdzinot ar apstrādi visai teritorijai vienlaikus.
