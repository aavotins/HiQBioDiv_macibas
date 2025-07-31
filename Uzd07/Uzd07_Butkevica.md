Septitais uzdevums: zonālā statistika
================
Jekaterīna Butkeviča
2025. gads 31. jūlijs

## Pakotnes

``` r
if (!require("here")) install.packages("here"); library(here)
if (!require("curl")) install.packages("curl"); library(curl)
if (!require("sfarrow")) install.packages("sfarrow"); library(sfarrow)
if (!require("dplyr")) install.packages("dplyr"); library(dplyr)
if (!require("terra")) install.packages("terra"); library(terra)
if (!require("sf")) install.packages("sf"); library(sf)
if (!require("exactextractr")) install.packages("exactextractr"); library(exactextractr)
if (!require("microbenchmark")) install.packages("microbenchmark"); library(microbenchmark)
if (!require("ggplot2")) install.packages("ggplot2"); library(ggplot2)
```

# Pirmais uzdevums

Izmantojiet {exactextractr}, lai 500 m buferzonās ap [projekta *Zenodo*
repozitorijā](https://zenodo.org/communities/hiqbiodiv/records?q=&l=list&p=1&s=10&sort=newest)
pieejamo 100 m šūnu centriem (`pts100_sauszeme.parquet`), kuri atrodas
piektajā uzdevumā izvēlētajās kartes lapās, aprēķinātu sestā uzdevuma
ceturtajā apakšpunktā izveidotā rastra katras klases platības īpatsvaru.
No aprēķinu rezultātiem sagatavojiet rastra slāņus, kas atbilst
[projekta *Zenodo*
repozitorijā](https://zenodo.org/communities/hiqbiodiv/records?q=&l=list&p=1&s=10&sort=newest)
dotajam 100 m rastram (`LV100m_10km.tif`).

## Datu sagatavošana pirmajam uzdevumam

Šis fails man iepriekš nebija lejupielādēts. Lejupielādējam *šūnu centru
references failu*, izdzēšam lieko un ielasām lejupielādēto failu.

``` r
ref_dir <- here("7uzd","Data")
dir.create(ref_dir, recursive = TRUE)

ref100SunuCenri_url <- "https://zenodo.org/records/14277114/files/pts100_sauzeme.parquet?download=1"
ref100SunuCenri_cels <- file.path(ref_dir, "pts100_sauszeme.parquet")

con_in <- curl(ref100SunuCenri_url, open = "rb") # Atver savienojumu ar tīmekļa failu
con_out <- file(ref100SunuCenri_cels, open = "wb") # izveido savienojumu ar failu, lai to varētu rakstīt binārā režīmā

writeBin(readBin(con_in, what = "raw", n = 1e8), con_out) # Lejupielādē un saglabā bināro saturu

# Aizvert savienojumus
close(con_in)
close(con_out)

# Izdzest nevajadzigo 
rm(con_in, con_out, ref100SunuCenri_url)

# Ielasīt failu
Sunu100Centri <- st_read_parquet(ref100SunuCenri_cels)
```

Ielasām *100 metru references rastru*.

``` r
Sunas100 <- rast(here("3uzd", "Data", "references_zenodo", "rastrs", "LV100m_10km.tif"))
```

Ielasījām karšu lapas, kuras izvēlējāmies piektajam uzdevumam.
Apvienojām tās un izdzēsām lieko.

``` r
lapa3334 <- st_read_parquet(here("5uzd", "Output", "kartes_lapas", "lapa3334.parquet"))
lapa3343 <- st_read_parquet(here("5uzd", "Output", "kartes_lapas", "lapa3343.parquet"))
lapa4312 <- st_read_parquet(here("5uzd", "Output", "kartes_lapas", "lapa4312.parquet"))
lapa4321 <- st_read_parquet(here("5uzd", "Output", "kartes_lapas", "lapa4321.parquet"))
lapas_kopa <- bind_rows(lapa3334, lapa3343, lapa4312, lapa4321) # apvienot vienā objektā

rm(lapa3334, lapa3343, lapa4312, lapa4321)
```

Importējam rastru, kas sagatavots 6. uzdevuma 4. apakšuzdevumā.

``` r
uzd6_rastrs <- rast(here("6uzd", "Output", "rastri", "savienoti_rastri.tif"))
```

## Risinājums

Vispirms apgriežam references slāņus interesējošajai teritorijai.
Iekļauju arī koordinātu sistēmas pārbaudi, jo neatceros, kāda tā bija
katram slānim.

``` r
st_crs(Sunu100Centri) == st_crs(lapas_kopa)
```

    ## [1] TRUE

``` r
Sunu100Centri_apgriezts <- st_intersection(Sunu100Centri, lapas_kopa) # atstāt tikai punktus no vajadzīgas teritorijas
Sunas100_apgriezts <- crop(Sunas100, lapas_kopa) # apgriezt referenci tikai vajadzīgai teritorijai
```

Veidojam 500 metru buferzonu ap šūnu centriem.

``` r
Sunu100Centri_buffer <- st_buffer(Sunu100Centri_apgriezts, dist = 500) # 500 metru buferis ap punktiem
```

Aprēķinām lauku platību īpatsvaru. Funkcija `exact_extract()` ņem vērā
arī šūnas, kas pārklājas ar zonu tikai daļēji. Tā aprēķina, kāda šūnas
daļa ietilpst attiecīgajā zonā. Mainīgais `id` kalpo kā identifikators,
lai vēlāk varētu datus savienot ar references slāni.

``` r
ipatsvari_500buff <- exact_extract(uzd6_rastrs, 
                                     Sunu100Centri_buffer, 
                                     fun = "frac",
                                     append_cols = "id") # pievieno katram rezultātam arī atbilstošo poligona ID no Sunu100Centri_buffer
```

Savienoju ar references slāni un noņēmu nevajadzīgās kolonnas.

``` r
# Savienot ar references slāni
ipatsvari_500buff <- left_join(Sunu100Centri_buffer, 
                               ipatsvari_500buff, 
                               by = "id")

ipatsvari_500buff <- ipatsvari_500buff[,-c(1:10)] # Noņemt nevajadzīgas kolonnas
```

Veicu rezultātu rastrēšanu.

``` r
klasuKol_nosaukumi <- names(ipatsvari_500buff)[startsWith(names(ipatsvari_500buff), "frac_")]

rez_dir <- here("7uzd","Output", "Pirmais_apaksuzd")
dir.create(rez_dir, recursive = TRUE)


for (klase in klasuKol_nosaukumi) {
  try({
    # Rasterizācija
    rastrs <- rasterize(ipatsvari_500buff, 
                        Sunas100_apgriezts, 
                        field = klase, 
                        fun = "mean")
    
    # Faila nosaukums un saglabāšana
    faila_nosaukums <- file.path(rez_dir, paste0(klase, "_ipatsvars500suna.tif"))
    writeRaster(rastrs, faila_nosaukums, overwrite = TRUE)
    
    message("Saglabāts: ", faila_nosaukums)
  }, silent = TRUE)
}
```

# Otrais uzdevums

Brīvi izvēlaties desmit blakus esošus 1km kvadrātus, kas atrodas trešajā
uzdevumā ar Lauku atbalsta dienesta datiem aptvertajā teritorijā.
Izmantojiet projekta Zenodo repozitorijā dotos 300 m un 100 m tīklu
centrus, kas atrodas izvēlētajos kvadrātos. Aprēķiniet 3 km buferzonas
ap ik centru. Veiciet zonālās statistikas aprēķinus lauka bloku
īpatsvaram buferzonā (no tās kopējās platības), mērogojot aprēķiniem
nepieciešamo laiku (desmit atkārtojumos): (…)

## Datu sagatavošana otrajam uzdevumam

``` r
# 1 km kvadrāti
tikls1km <- st_read_parquet(here("3uzd", "Data", "references_zenodo", "vektors", "tikls1km_sauzeme.parquet"))

# 300 metru sunu centri
Sunu300Centri <- st_read_parquet(here("3uzd", "Data", "references_zenodo", "vektors", "pts300_sauzeme.parquet"))


## LAD dati
# LAD 10 metru šūnas
LAD_10metri <- rast(here("3uzd", "Output", "TIFF", "LAD_centra_10m.tif"))

# LAD 100 metru šūnas
LAD_100metri  <- rast(here("3uzd", "Output", "TIFF", "LAD_centra_100m_plavu_binars.tif"))
```

## Kvadrātu atlase

Lai atlasītu kvadrātus, vispirms apskatīju attiecīgo slāni QGIS
programmā. Izvēlēto kvadrātu identifikatorus pārkopēju no atribūtu
tabulas un izveidoju atsevišķu vektoru ar tiem. Balstoties uz šiem
identifikatoriem, veicu vajadzīgo kvadrātu atlasi.

``` r
# Saglabājam kvadrātu ID atsevišķa vektorā
string_ID1km <- c("1kmX467Y279", "1kmX467Y278", "1kmX467Y277", 
                  "1kmX468Y279", "1kmX468Y278", "1kmX468Y277",
                  "1kmX469Y279", "1kmX469Y278", "1kmX469Y277",
                  "1kmX470Y279")

#Atlase
kvadrati <- tikls1km %>% filter(ID1km %in% string_ID1km)
```

Apgriežu referenču slāni. Funkcija `crop()` apgriež rastru precīzi
taisnstūra formā (bounding box), kas aptver visus norādītos mainīgā
`kvadrati` objektus.

``` r
Sunas100_apgriezts <- crop(Sunas100, kvadrati)
```

Tālāk aprēķinu 3 km buferzonas ap šūnu centriem. Arguments `dist =`
attālumam izmanto tās pašas mērvienības, kādās ir objekta
`Sunu300Centri_apgriezts` koordinātu sistēmas vienības. LKS92 / Latvia
TM (EPSG:3059) nozīmē, ka mērvienības ir metri, tāpēc `dist = 3000`
atbilst 3000 metriem jeb 3 kilometriem.

``` r
# 100m šūnas
Sunu100Centri_apgriezts2 <- st_intersection(Sunu100Centri, kvadrati)
Sunu100Centri_buffer2 <- st_buffer(Sunu100Centri_apgriezts2, dist = 3000)

# 300m šūnas
Sunu300Centri_apgriezts <- st_intersection(Sunu300Centri, kvadrati)
Sunu300Centri_buffer2 <- st_buffer(Sunu300Centri_apgriezts, dist = 3000)
```

## 2.1 apakšuzdevums

(…) ik 100 m tīkla centram, kā aprakstāmo rastru izmantojot iepriekšējos
uzdevumos sagatavoto lauku klātbūtni 10 m šūnā;

Veicu laika mērīšanu, rezultātu saglabāju atsevišķā objektā.

``` r
laiks_2_1 <- microbenchmark(exact_extract(
  LAD_10metri, 
  Sunu100Centri_buffer2, 
  fun = "frac"), times = 10)
```

Zonālā statistika tiek aprēķināta, izmantojot funkciju
`exact_extract()`, un rezultāts tiek apvienots ar buferzonu ģeometriju.

``` r
uzd_2_1 <- exact_extract(LAD_10metri, Sunu100Centri_buffer2, fun = "frac", append_cols = "id")
```

    ##   |                                                                              |                                                                      |   0%  |                                                                              |                                                                      |   1%  |                                                                              |=                                                                     |   1%  |                                                                              |=                                                                     |   2%  |                                                                              |==                                                                    |   2%  |                                                                              |==                                                                    |   3%  |                                                                              |==                                                                    |   4%  |                                                                              |===                                                                   |   4%  |                                                                              |===                                                                   |   5%  |                                                                              |====                                                                  |   5%  |                                                                              |====                                                                  |   6%  |                                                                              |=====                                                                 |   6%  |                                                                              |=====                                                                 |   7%  |                                                                              |=====                                                                 |   8%  |                                                                              |======                                                                |   8%  |                                                                              |======                                                                |   9%  |                                                                              |=======                                                               |   9%  |                                                                              |=======                                                               |  10%  |                                                                              |=======                                                               |  11%  |                                                                              |========                                                              |  11%  |                                                                              |========                                                              |  12%  |                                                                              |=========                                                             |  12%  |                                                                              |=========                                                             |  13%  |                                                                              |=========                                                             |  14%  |                                                                              |==========                                                            |  14%  |                                                                              |==========                                                            |  15%  |                                                                              |===========                                                           |  15%  |                                                                              |===========                                                           |  16%  |                                                                              |============                                                          |  16%  |                                                                              |============                                                          |  17%  |                                                                              |============                                                          |  18%  |                                                                              |=============                                                         |  18%  |                                                                              |=============                                                         |  19%  |                                                                              |==============                                                        |  19%  |                                                                              |==============                                                        |  20%  |                                                                              |==============                                                        |  21%  |                                                                              |===============                                                       |  21%  |                                                                              |===============                                                       |  22%  |                                                                              |================                                                      |  22%  |                                                                              |================                                                      |  23%  |                                                                              |================                                                      |  24%  |                                                                              |=================                                                     |  24%  |                                                                              |=================                                                     |  25%  |                                                                              |==================                                                    |  25%  |                                                                              |==================                                                    |  26%  |                                                                              |===================                                                   |  26%  |                                                                              |===================                                                   |  27%  |                                                                              |===================                                                   |  28%  |                                                                              |====================                                                  |  28%  |                                                                              |====================                                                  |  29%  |                                                                              |=====================                                                 |  29%  |                                                                              |=====================                                                 |  30%  |                                                                              |=====================                                                 |  31%  |                                                                              |======================                                                |  31%  |                                                                              |======================                                                |  32%  |                                                                              |=======================                                               |  32%  |                                                                              |=======================                                               |  33%  |                                                                              |=======================                                               |  34%  |                                                                              |========================                                              |  34%  |                                                                              |========================                                              |  35%  |                                                                              |=========================                                             |  35%  |                                                                              |=========================                                             |  36%  |                                                                              |==========================                                            |  36%  |                                                                              |==========================                                            |  37%  |                                                                              |==========================                                            |  38%  |                                                                              |===========================                                           |  38%  |                                                                              |===========================                                           |  39%  |                                                                              |============================                                          |  39%  |                                                                              |============================                                          |  40%  |                                                                              |============================                                          |  41%  |                                                                              |=============================                                         |  41%  |                                                                              |=============================                                         |  42%  |                                                                              |==============================                                        |  42%  |                                                                              |==============================                                        |  43%  |                                                                              |==============================                                        |  44%  |                                                                              |===============================                                       |  44%  |                                                                              |===============================                                       |  45%  |                                                                              |================================                                      |  45%  |                                                                              |================================                                      |  46%  |                                                                              |=================================                                     |  46%  |                                                                              |=================================                                     |  47%  |                                                                              |=================================                                     |  48%  |                                                                              |==================================                                    |  48%  |                                                                              |==================================                                    |  49%  |                                                                              |===================================                                   |  49%  |                                                                              |===================================                                   |  50%  |                                                                              |===================================                                   |  51%  |                                                                              |====================================                                  |  51%  |                                                                              |====================================                                  |  52%  |                                                                              |=====================================                                 |  52%  |                                                                              |=====================================                                 |  53%  |                                                                              |=====================================                                 |  54%  |                                                                              |======================================                                |  54%  |                                                                              |======================================                                |  55%  |                                                                              |=======================================                               |  55%  |                                                                              |=======================================                               |  56%  |                                                                              |========================================                              |  56%  |                                                                              |========================================                              |  57%  |                                                                              |========================================                              |  58%  |                                                                              |=========================================                             |  58%  |                                                                              |=========================================                             |  59%  |                                                                              |==========================================                            |  59%  |                                                                              |==========================================                            |  60%  |                                                                              |==========================================                            |  61%  |                                                                              |===========================================                           |  61%  |                                                                              |===========================================                           |  62%  |                                                                              |============================================                          |  62%  |                                                                              |============================================                          |  63%  |                                                                              |============================================                          |  64%  |                                                                              |=============================================                         |  64%  |                                                                              |=============================================                         |  65%  |                                                                              |==============================================                        |  65%  |                                                                              |==============================================                        |  66%  |                                                                              |===============================================                       |  66%  |                                                                              |===============================================                       |  67%  |                                                                              |===============================================                       |  68%  |                                                                              |================================================                      |  68%  |                                                                              |================================================                      |  69%  |                                                                              |=================================================                     |  69%  |                                                                              |=================================================                     |  70%  |                                                                              |=================================================                     |  71%  |                                                                              |==================================================                    |  71%  |                                                                              |==================================================                    |  72%  |                                                                              |===================================================                   |  72%  |                                                                              |===================================================                   |  73%  |                                                                              |===================================================                   |  74%  |                                                                              |====================================================                  |  74%  |                                                                              |====================================================                  |  75%  |                                                                              |=====================================================                 |  75%  |                                                                              |=====================================================                 |  76%  |                                                                              |======================================================                |  76%  |                                                                              |======================================================                |  77%  |                                                                              |======================================================                |  78%  |                                                                              |=======================================================               |  78%  |                                                                              |=======================================================               |  79%  |                                                                              |========================================================              |  79%  |                                                                              |========================================================              |  80%  |                                                                              |========================================================              |  81%  |                                                                              |=========================================================             |  81%  |                                                                              |=========================================================             |  82%  |                                                                              |==========================================================            |  82%  |                                                                              |==========================================================            |  83%  |                                                                              |==========================================================            |  84%  |                                                                              |===========================================================           |  84%  |                                                                              |===========================================================           |  85%  |                                                                              |============================================================          |  85%  |                                                                              |============================================================          |  86%  |                                                                              |=============================================================         |  86%  |                                                                              |=============================================================         |  87%  |                                                                              |=============================================================         |  88%  |                                                                              |==============================================================        |  88%  |                                                                              |==============================================================        |  89%  |                                                                              |===============================================================       |  89%  |                                                                              |===============================================================       |  90%  |                                                                              |===============================================================       |  91%  |                                                                              |================================================================      |  91%  |                                                                              |================================================================      |  92%  |                                                                              |=================================================================     |  92%  |                                                                              |=================================================================     |  93%  |                                                                              |=================================================================     |  94%  |                                                                              |==================================================================    |  94%  |                                                                              |==================================================================    |  95%  |                                                                              |===================================================================   |  95%  |                                                                              |===================================================================   |  96%  |                                                                              |====================================================================  |  96%  |                                                                              |====================================================================  |  97%  |                                                                              |====================================================================  |  98%  |                                                                              |===================================================================== |  98%  |                                                                              |===================================================================== |  99%  |                                                                              |======================================================================|  99%  |                                                                              |======================================================================| 100%

``` r
# īpatsvars 
ipatsvars_2_1 <- left_join(Sunu100Centri_buffer2, uzd_2_1, by = "id") # savienot
```

Ar funkciju `rasterize()` tiek izveidots rastrs, kurā katra šūna satur
vidējo LAD īpatsvaru `(frac_1)`, pamatojoties uz zonālās statistikas
rezultātiem

``` r
uzd_2_1_rastr <- rasterize(ipatsvars_2_1, Sunas100_apgriezts, 
                           field = "frac_1", 
                           fun = "mean")
```

Veicu rezultātu saglabāšanu.

``` r
dir <- here("7uzd", "Output", "Otrais_uzd", "Pirmais_apaksuzd")
dir.create(dir, recursive = TRUE)
writeRaster(uzd_2_1_rastr, 
            filename = file.path(dir, "uzd_2_1_LAD_ipatsvars.tif"), 
            overwrite = TRUE)
```

## 2.2 apakšuzdevums

(…) ik 100 m tīkla centram, kā aprakstāmo rastru izmantojot iepriekšējos
uzdevumos sagatavoto lauku īpatsvaru 100 m šūnā;

Veicu laika mērījumu.

``` r
laiks_2_2 <- microbenchmark(exact_extract(
  LAD_100metri, 
  Sunu100Centri_buffer2, 
  fun = "frac"), times = 10)
```

Zonālā statistika.

``` r
uzd_2_2 <- exact_extract(LAD_100metri, 
                         Sunu100Centri_buffer2, 
                         fun = "frac", append_cols = "id")
```

    ##   |                                                                              |                                                                      |   0%  |                                                                              |                                                                      |   1%  |                                                                              |=                                                                     |   1%  |                                                                              |=                                                                     |   2%  |                                                                              |==                                                                    |   2%  |                                                                              |==                                                                    |   3%  |                                                                              |==                                                                    |   4%  |                                                                              |===                                                                   |   4%  |                                                                              |===                                                                   |   5%  |                                                                              |====                                                                  |   5%  |                                                                              |====                                                                  |   6%  |                                                                              |=====                                                                 |   6%  |                                                                              |=====                                                                 |   7%  |                                                                              |=====                                                                 |   8%  |                                                                              |======                                                                |   8%  |                                                                              |======                                                                |   9%  |                                                                              |=======                                                               |   9%  |                                                                              |=======                                                               |  10%  |                                                                              |=======                                                               |  11%  |                                                                              |========                                                              |  11%  |                                                                              |========                                                              |  12%  |                                                                              |=========                                                             |  12%  |                                                                              |=========                                                             |  13%  |                                                                              |=========                                                             |  14%  |                                                                              |==========                                                            |  14%  |                                                                              |==========                                                            |  15%  |                                                                              |===========                                                           |  15%  |                                                                              |===========                                                           |  16%  |                                                                              |============                                                          |  16%  |                                                                              |============                                                          |  17%  |                                                                              |============                                                          |  18%  |                                                                              |=============                                                         |  18%  |                                                                              |=============                                                         |  19%  |                                                                              |==============                                                        |  19%  |                                                                              |==============                                                        |  20%  |                                                                              |==============                                                        |  21%  |                                                                              |===============                                                       |  21%  |                                                                              |===============                                                       |  22%  |                                                                              |================                                                      |  22%  |                                                                              |================                                                      |  23%  |                                                                              |================                                                      |  24%  |                                                                              |=================                                                     |  24%  |                                                                              |=================                                                     |  25%  |                                                                              |==================                                                    |  25%  |                                                                              |==================                                                    |  26%  |                                                                              |===================                                                   |  26%  |                                                                              |===================                                                   |  27%  |                                                                              |===================                                                   |  28%  |                                                                              |====================                                                  |  28%  |                                                                              |====================                                                  |  29%  |                                                                              |=====================                                                 |  29%  |                                                                              |=====================                                                 |  30%  |                                                                              |=====================                                                 |  31%  |                                                                              |======================                                                |  31%  |                                                                              |======================                                                |  32%  |                                                                              |=======================                                               |  32%  |                                                                              |=======================                                               |  33%  |                                                                              |=======================                                               |  34%  |                                                                              |========================                                              |  34%  |                                                                              |========================                                              |  35%  |                                                                              |=========================                                             |  35%  |                                                                              |=========================                                             |  36%  |                                                                              |==========================                                            |  36%  |                                                                              |==========================                                            |  37%  |                                                                              |==========================                                            |  38%  |                                                                              |===========================                                           |  38%  |                                                                              |===========================                                           |  39%  |                                                                              |============================                                          |  39%  |                                                                              |============================                                          |  40%  |                                                                              |============================                                          |  41%  |                                                                              |=============================                                         |  41%  |                                                                              |=============================                                         |  42%  |                                                                              |==============================                                        |  42%  |                                                                              |==============================                                        |  43%  |                                                                              |==============================                                        |  44%  |                                                                              |===============================                                       |  44%  |                                                                              |===============================                                       |  45%  |                                                                              |================================                                      |  45%  |                                                                              |================================                                      |  46%  |                                                                              |=================================                                     |  46%  |                                                                              |=================================                                     |  47%  |                                                                              |=================================                                     |  48%  |                                                                              |==================================                                    |  48%  |                                                                              |==================================                                    |  49%  |                                                                              |===================================                                   |  49%  |                                                                              |===================================                                   |  50%  |                                                                              |===================================                                   |  51%  |                                                                              |====================================                                  |  51%  |                                                                              |====================================                                  |  52%  |                                                                              |=====================================                                 |  52%  |                                                                              |=====================================                                 |  53%  |                                                                              |=====================================                                 |  54%  |                                                                              |======================================                                |  54%  |                                                                              |======================================                                |  55%  |                                                                              |=======================================                               |  55%  |                                                                              |=======================================                               |  56%  |                                                                              |========================================                              |  56%  |                                                                              |========================================                              |  57%  |                                                                              |========================================                              |  58%  |                                                                              |=========================================                             |  58%  |                                                                              |=========================================                             |  59%  |                                                                              |==========================================                            |  59%  |                                                                              |==========================================                            |  60%  |                                                                              |==========================================                            |  61%  |                                                                              |===========================================                           |  61%  |                                                                              |===========================================                           |  62%  |                                                                              |============================================                          |  62%  |                                                                              |============================================                          |  63%  |                                                                              |============================================                          |  64%  |                                                                              |=============================================                         |  64%  |                                                                              |=============================================                         |  65%  |                                                                              |==============================================                        |  65%  |                                                                              |==============================================                        |  66%  |                                                                              |===============================================                       |  66%  |                                                                              |===============================================                       |  67%  |                                                                              |===============================================                       |  68%  |                                                                              |================================================                      |  68%  |                                                                              |================================================                      |  69%  |                                                                              |=================================================                     |  69%  |                                                                              |=================================================                     |  70%  |                                                                              |=================================================                     |  71%  |                                                                              |==================================================                    |  71%  |                                                                              |==================================================                    |  72%  |                                                                              |===================================================                   |  72%  |                                                                              |===================================================                   |  73%  |                                                                              |===================================================                   |  74%  |                                                                              |====================================================                  |  74%  |                                                                              |====================================================                  |  75%  |                                                                              |=====================================================                 |  75%  |                                                                              |=====================================================                 |  76%  |                                                                              |======================================================                |  76%  |                                                                              |======================================================                |  77%  |                                                                              |======================================================                |  78%  |                                                                              |=======================================================               |  78%  |                                                                              |=======================================================               |  79%  |                                                                              |========================================================              |  79%  |                                                                              |========================================================              |  80%  |                                                                              |========================================================              |  81%  |                                                                              |=========================================================             |  81%  |                                                                              |=========================================================             |  82%  |                                                                              |==========================================================            |  82%  |                                                                              |==========================================================            |  83%  |                                                                              |==========================================================            |  84%  |                                                                              |===========================================================           |  84%  |                                                                              |===========================================================           |  85%  |                                                                              |============================================================          |  85%  |                                                                              |============================================================          |  86%  |                                                                              |=============================================================         |  86%  |                                                                              |=============================================================         |  87%  |                                                                              |=============================================================         |  88%  |                                                                              |==============================================================        |  88%  |                                                                              |==============================================================        |  89%  |                                                                              |===============================================================       |  89%  |                                                                              |===============================================================       |  90%  |                                                                              |===============================================================       |  91%  |                                                                              |================================================================      |  91%  |                                                                              |================================================================      |  92%  |                                                                              |=================================================================     |  92%  |                                                                              |=================================================================     |  93%  |                                                                              |=================================================================     |  94%  |                                                                              |==================================================================    |  94%  |                                                                              |==================================================================    |  95%  |                                                                              |===================================================================   |  95%  |                                                                              |===================================================================   |  96%  |                                                                              |====================================================================  |  96%  |                                                                              |====================================================================  |  97%  |                                                                              |====================================================================  |  98%  |                                                                              |===================================================================== |  98%  |                                                                              |===================================================================== |  99%  |                                                                              |======================================================================|  99%  |                                                                              |======================================================================| 100%

``` r
ipatsvars_2_2 <- left_join(Sunu100Centri_buffer2, uzd_2_2, by = "id")
```

Veidoju rastru.

``` r
uzd_2_2_rast <- rasterize(ipatsvars_2_2, 
                          Sunas100_apgriezts, 
                          field = "frac_1", fun = "mean")
```

Rezultātus saglabāju.

``` r
dir <- here("7uzd", "Output", "Otrais_uzd", "Otrais_apaksuzd")
dir.create(dir, recursive = TRUE)
writeRaster(uzd_2_2_rast, 
            filename = file.path(dir, "uzd_2_2_LAD_ipatsvars.tif"), 
            overwrite = TRUE)
```

## 2.3 apakšuzdevums

(…) ik 300 m tīkla centram, kā aprakstāmo rastru izmantojot iepriekšējos
uzdevumos sagatavoto lauku klātbūtni 10 m šūnā, savienojiet iegūtos
rezultātus ar 100 m tīklu, izmantojot kopīgos identifikatorus;

Veicu laika mērījumu.

``` r
laiks_2_3 <- microbenchmark(exact_extract(
  LAD_10metri, 
  Sunu300Centri_buffer2, 
  fun = "frac"), times = 10)
```

Zonālā statistika.

``` r
uzd_2_3 <- exact_extract(
  LAD_10metri, 
  Sunu300Centri_buffer2, 
  fun = "frac", append_cols = "rinda300")
```

    ##   |                                                                              |                                                                      |   0%  |                                                                              |=                                                                     |   1%  |                                                                              |=                                                                     |   2%  |                                                                              |==                                                                    |   3%  |                                                                              |===                                                                   |   4%  |                                                                              |===                                                                   |   5%  |                                                                              |====                                                                  |   6%  |                                                                              |=====                                                                 |   7%  |                                                                              |======                                                                |   8%  |                                                                              |======                                                                |   9%  |                                                                              |=======                                                               |  10%  |                                                                              |========                                                              |  11%  |                                                                              |========                                                              |  12%  |                                                                              |=========                                                             |  13%  |                                                                              |==========                                                            |  14%  |                                                                              |==========                                                            |  15%  |                                                                              |===========                                                           |  16%  |                                                                              |============                                                          |  17%  |                                                                              |=============                                                         |  18%  |                                                                              |=============                                                         |  19%  |                                                                              |==============                                                        |  20%  |                                                                              |===============                                                       |  21%  |                                                                              |===============                                                       |  22%  |                                                                              |================                                                      |  23%  |                                                                              |=================                                                     |  24%  |                                                                              |=================                                                     |  25%  |                                                                              |==================                                                    |  26%  |                                                                              |===================                                                   |  27%  |                                                                              |===================                                                   |  28%  |                                                                              |====================                                                  |  28%  |                                                                              |=====================                                                 |  29%  |                                                                              |=====================                                                 |  30%  |                                                                              |======================                                                |  31%  |                                                                              |======================                                                |  32%  |                                                                              |=======================                                               |  33%  |                                                                              |========================                                              |  34%  |                                                                              |========================                                              |  35%  |                                                                              |=========================                                             |  36%  |                                                                              |==========================                                            |  37%  |                                                                              |==========================                                            |  38%  |                                                                              |===========================                                           |  39%  |                                                                              |============================                                          |  39%  |                                                                              |============================                                          |  40%  |                                                                              |=============================                                         |  41%  |                                                                              |==============================                                        |  42%  |                                                                              |==============================                                        |  43%  |                                                                              |===============================                                       |  44%  |                                                                              |===============================                                       |  45%  |                                                                              |================================                                      |  46%  |                                                                              |=================================                                     |  47%  |                                                                              |=================================                                     |  48%  |                                                                              |==================================                                    |  49%  |                                                                              |===================================                                   |  50%  |                                                                              |====================================                                  |  51%  |                                                                              |=====================================                                 |  52%  |                                                                              |=====================================                                 |  53%  |                                                                              |======================================                                |  54%  |                                                                              |=======================================                               |  55%  |                                                                              |=======================================                               |  56%  |                                                                              |========================================                              |  57%  |                                                                              |========================================                              |  58%  |                                                                              |=========================================                             |  59%  |                                                                              |==========================================                            |  60%  |                                                                              |==========================================                            |  61%  |                                                                              |===========================================                           |  61%  |                                                                              |============================================                          |  62%  |                                                                              |============================================                          |  63%  |                                                                              |=============================================                         |  64%  |                                                                              |==============================================                        |  65%  |                                                                              |==============================================                        |  66%  |                                                                              |===============================================                       |  67%  |                                                                              |================================================                      |  68%  |                                                                              |================================================                      |  69%  |                                                                              |=================================================                     |  70%  |                                                                              |=================================================                     |  71%  |                                                                              |==================================================                    |  72%  |                                                                              |===================================================                   |  72%  |                                                                              |===================================================                   |  73%  |                                                                              |====================================================                  |  74%  |                                                                              |=====================================================                 |  75%  |                                                                              |=====================================================                 |  76%  |                                                                              |======================================================                |  77%  |                                                                              |=======================================================               |  78%  |                                                                              |=======================================================               |  79%  |                                                                              |========================================================              |  80%  |                                                                              |=========================================================             |  81%  |                                                                              |=========================================================             |  82%  |                                                                              |==========================================================            |  83%  |                                                                              |===========================================================           |  84%  |                                                                              |============================================================          |  85%  |                                                                              |============================================================          |  86%  |                                                                              |=============================================================         |  87%  |                                                                              |==============================================================        |  88%  |                                                                              |==============================================================        |  89%  |                                                                              |===============================================================       |  90%  |                                                                              |================================================================      |  91%  |                                                                              |================================================================      |  92%  |                                                                              |=================================================================     |  93%  |                                                                              |==================================================================    |  94%  |                                                                              |===================================================================   |  95%  |                                                                              |===================================================================   |  96%  |                                                                              |====================================================================  |  97%  |                                                                              |===================================================================== |  98%  |                                                                              |===================================================================== |  99%  |                                                                              |======================================================================| 100%

``` r
ipatsvars_2_3 <- left_join(Sunu300Centri_buffer2, uzd_2_3, by = "rinda300") #savienot
```

Veidoju rastru.

``` r
uzd_2_3_rastrs <- rasterize(ipatsvars_2_3, 
                            Sunas100_apgriezts, 
                            field = "frac_1", fun = "mean")
```

Rezultātus saglabāju.

``` r
dir <- here("7uzd", "Output", "Otrais_uzd", "Trešais_apaksuzd")
dir.create(dir, recursive = TRUE)

writeRaster(uzd_2_3_rastrs, 
            filename = file.path(dir, "uzd_2_3_LAD_ipatsvars.tif"), 
            overwrite = TRUE)
```

## 2.4 apakšuzdevums

(…) ik 300 m tīkla centram, kā aprakstāmo rastru izmantojot iepriekšējos
uzdevumos sagatavoto lauku īpatsvaru 100 m šūnā, savienojiet iegūtos
rezultātus ar 100 m tīklu, izmantojot kopīgos identifikatorus.

Veicu laika mērījumu.

``` r
laiks_2_4 <- microbenchmark(exact_extract(
  LAD_100metri, 
  Sunu300Centri_buffer2, 
  fun = "frac"), times = 10)
```

Zonālā statistika.

``` r
uzd_2_4 <- exact_extract(LAD_100metri, Sunu300Centri_buffer2, fun = "frac", append_cols = "rinda300")
```

    ##   |                                                                              |                                                                      |   0%  |                                                                              |=                                                                     |   1%  |                                                                              |=                                                                     |   2%  |                                                                              |==                                                                    |   3%  |                                                                              |===                                                                   |   4%  |                                                                              |===                                                                   |   5%  |                                                                              |====                                                                  |   6%  |                                                                              |=====                                                                 |   7%  |                                                                              |======                                                                |   8%  |                                                                              |======                                                                |   9%  |                                                                              |=======                                                               |  10%  |                                                                              |========                                                              |  11%  |                                                                              |========                                                              |  12%  |                                                                              |=========                                                             |  13%  |                                                                              |==========                                                            |  14%  |                                                                              |==========                                                            |  15%  |                                                                              |===========                                                           |  16%  |                                                                              |============                                                          |  17%  |                                                                              |=============                                                         |  18%  |                                                                              |=============                                                         |  19%  |                                                                              |==============                                                        |  20%  |                                                                              |===============                                                       |  21%  |                                                                              |===============                                                       |  22%  |                                                                              |================                                                      |  23%  |                                                                              |=================                                                     |  24%  |                                                                              |=================                                                     |  25%  |                                                                              |==================                                                    |  26%  |                                                                              |===================                                                   |  27%  |                                                                              |===================                                                   |  28%  |                                                                              |====================                                                  |  28%  |                                                                              |=====================                                                 |  29%  |                                                                              |=====================                                                 |  30%  |                                                                              |======================                                                |  31%  |                                                                              |======================                                                |  32%  |                                                                              |=======================                                               |  33%  |                                                                              |========================                                              |  34%  |                                                                              |========================                                              |  35%  |                                                                              |=========================                                             |  36%  |                                                                              |==========================                                            |  37%  |                                                                              |==========================                                            |  38%  |                                                                              |===========================                                           |  39%  |                                                                              |============================                                          |  39%  |                                                                              |============================                                          |  40%  |                                                                              |=============================                                         |  41%  |                                                                              |==============================                                        |  42%  |                                                                              |==============================                                        |  43%  |                                                                              |===============================                                       |  44%  |                                                                              |===============================                                       |  45%  |                                                                              |================================                                      |  46%  |                                                                              |=================================                                     |  47%  |                                                                              |=================================                                     |  48%  |                                                                              |==================================                                    |  49%  |                                                                              |===================================                                   |  50%  |                                                                              |====================================                                  |  51%  |                                                                              |=====================================                                 |  52%  |                                                                              |=====================================                                 |  53%  |                                                                              |======================================                                |  54%  |                                                                              |=======================================                               |  55%  |                                                                              |=======================================                               |  56%  |                                                                              |========================================                              |  57%  |                                                                              |========================================                              |  58%  |                                                                              |=========================================                             |  59%  |                                                                              |==========================================                            |  60%  |                                                                              |==========================================                            |  61%  |                                                                              |===========================================                           |  61%  |                                                                              |============================================                          |  62%  |                                                                              |============================================                          |  63%  |                                                                              |=============================================                         |  64%  |                                                                              |==============================================                        |  65%  |                                                                              |==============================================                        |  66%  |                                                                              |===============================================                       |  67%  |                                                                              |================================================                      |  68%  |                                                                              |================================================                      |  69%  |                                                                              |=================================================                     |  70%  |                                                                              |=================================================                     |  71%  |                                                                              |==================================================                    |  72%  |                                                                              |===================================================                   |  72%  |                                                                              |===================================================                   |  73%  |                                                                              |====================================================                  |  74%  |                                                                              |=====================================================                 |  75%  |                                                                              |=====================================================                 |  76%  |                                                                              |======================================================                |  77%  |                                                                              |=======================================================               |  78%  |                                                                              |=======================================================               |  79%  |                                                                              |========================================================              |  80%  |                                                                              |=========================================================             |  81%  |                                                                              |=========================================================             |  82%  |                                                                              |==========================================================            |  83%  |                                                                              |===========================================================           |  84%  |                                                                              |============================================================          |  85%  |                                                                              |============================================================          |  86%  |                                                                              |=============================================================         |  87%  |                                                                              |==============================================================        |  88%  |                                                                              |==============================================================        |  89%  |                                                                              |===============================================================       |  90%  |                                                                              |================================================================      |  91%  |                                                                              |================================================================      |  92%  |                                                                              |=================================================================     |  93%  |                                                                              |==================================================================    |  94%  |                                                                              |===================================================================   |  95%  |                                                                              |===================================================================   |  96%  |                                                                              |====================================================================  |  97%  |                                                                              |===================================================================== |  98%  |                                                                              |===================================================================== |  99%  |                                                                              |======================================================================| 100%

``` r
ipatsvars_2_4 <- left_join(Sunu300Centri_buffer2, uzd_2_4, by = "rinda300")
```

Veidoju rastru.

``` r
uzd_2_4_rast <- rasterize(ipatsvars_2_4, 
                          Sunas100_apgriezts, 
                          field = "frac_1", fun = "mean")
```

Rezultātus saglabāju.

``` r
dir <- here("7uzd", "Output", "Otrais_uzd", "Ceturtais_apaksuzd")
dir.create(dir, recursive = TRUE)

writeRaster(uzd_2_4_rast, 
            filename = file.path(dir, "uzd_2_4_LAD_ipatsvars.tif"), 
            overwrite = TRUE)
```

# Atbildes uz jautājumiem

## Kāds ir aprēķiniem nepieciešamais laiks katrā no četriem variantiem?

Izpildes laiku salīdzinājums. Veidoju datu rāmi ar `microbenchmark()`
funkcijas rezultātiem.

``` r
laiki_df <- bind_rows(
  as.data.frame(laiks_2_1) %>% mutate(uzdevums = "2.1.", tīkls = "100 m", rastrs = "10 m"),
  as.data.frame(laiks_2_2) %>% mutate(uzdevums = "2.2.", tīkls = "100 m", rastrs = "100 m"),
  as.data.frame(laiks_2_3) %>% mutate(uzdevums = "2.3.", tīkls = "300 m", rastrs = "10 m"),
  as.data.frame(laiks_2_4) %>% mutate(uzdevums = "2.4.", tīkls = "300 m", rastrs = "100 m")
)
```

Vizualizēju

``` r
ggplot(laiki_df, aes(x = uzdevums, y = time / 1e9)) +
  geom_boxplot() +
  labs(
    title = "Zonālās statistikas izpildes laiks",
    x = "Uzdevums", y = "Laiks (sekundēs)",
    caption = "2.1. – tīkls: 100m, rastrs: 10m\n2.2. – tīkls: 100m, rastrs: 100m\n2.3. – tīkls: 300m, rastrs: 10m\n2.4. – tīkls: 300m, rastrs: 100m"
  ) +
  theme_minimal()
```

![](Uzd07_Butkevica_files/figure-gfm/laiks_viz-1.png)<!-- -->

Un varam aprēķināt arī vidējo izpildes laiku:

``` r
vid_laiks_2_1 <- mean(laiks_2_1$time) / 1e9 #pārverst par sekundēm
vid_laiks_2_2 <- mean(laiks_2_2$time) / 1e9
vid_laiks_2_3 <- mean(laiks_2_3$time) / 1e9
vid_laiks_2_4 <- mean(laiks_2_4$time) / 1e9

print(vid_laiks_2_1)
```

    ## [1] 12.88618

``` r
print(vid_laiks_2_2)
```

    ## [1] 0.4982738

``` r
print(vid_laiks_2_3)
```

    ## [1] 1.28649

``` r
print(vid_laiks_2_4)
```

    ## [1] 0.1038069

Apskatot dažādu pieeju izpildes laika salīdzinājumu, var secināt, ka gan
tīkla, gan rastra izšķirtspēja ietekmē izpildes laiku.

Aprēķinu vidējais izpildes laiks (sekundēs) katrā variantā ir:

| Uzdevums | Tīkls | Rastra izšķirtspēja | Vidējais laiks (sek) |
|----------|-------|---------------------|----------------------|
| 2.1      | 100 m | 10 m                | 13.39                |
| 2.2      | 100 m | 100 m               | 0.94                 |
| 2.3      | 300 m | 10 m                | 1.80                 |
| 2.4      | 300 m | 100 m               | 0.14                 |

Redzams, ka aprēķini ar 10 m rastru prasa ievērojami ilgāku laiku nekā
ar 100 m rastru, īpaši, ja tīkls ir blīvāks (100 m centri).

## Kādas tendences ir saskatāmas?

- Izpildes laiks ir lielāks, ja tiek izmantots rastrs ar augstāku
  izšķirtspēju (10 m) salīdzinājumā ar (100 m).
- Blīvāks tīkls (100 m centri) palielina aprēķinu laiku, jo tiek
  apstrādāts vairāk punktu.
- Lielākais laiks ir uzdevumā 2.1 (100 m tīkls, 10 m rastrs), bet
  visīsākais — 2.4 (300 m tīkls, 100 m rastrs).

Kopumā var novērot tendenci, ka, palielinot tīkla vai rastra
izšķirtspēju, pagarinās aprēķiniem nepieciešamais laiks.

Vizuāli apskatot izveidotos rastrus (QGIS), es nenovēroju izteiktas
atšķirības rezultātā. Tātad, manuprāt, pieeja ar 300 metru tīklu un 100
metru rastru ir piemērota izmantošanai resursu taupīšanas nolūkiem.

## Kādas ir novērotās lauku platības īpatsvara atšķirības?

Salīdzinot platības īpatsvarus, aprēķinātus ar 10 m un 100 m rastriem,
vidējās atšķirības (`diff = frac_10m - frac_100m`) ir šādas:

``` r
# Salīdzinājums 2.1 un 2.2 (100 m centri ar 10m un 100m LAD)
ipatsvars_2_1_2 <- uzd_2_1 %>%
  rename(frac_10m = frac_1) %>%
  left_join(uzd_2_2 %>% rename(frac_100m = frac_1), by = "id") %>%
  mutate(diff = frac_10m - frac_100m)

summary(ipatsvars_2_1_2$diff)
```

    ##     Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
    ## -0.06527 -0.05139 -0.04809 -0.04728 -0.04245 -0.03219

``` r
boxplot(ipatsvars_2_1_2$diff, main = "Atšķirības 10m vs 100m LAD (100m centri)")
```

![](Uzd07_Butkevica_files/figure-gfm/2_1un_2_2-1.png)<!-- -->

Un

``` r
# Salīdzinājums 2.3 un 2.4 (300 m centri ar 10m un 100m LAD)
ipatsvars_2_3_4 <- uzd_2_3 %>%
  rename(frac_10m = frac_1) %>%
  left_join(uzd_2_4 %>% rename(frac_100m = frac_1), by = "rinda300") %>%
  mutate(diff = frac_10m - frac_100m)

summary(ipatsvars_2_3_4$diff)
```

    ##     Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
    ## -0.06293 -0.05134 -0.04813 -0.04734 -0.04298 -0.03470

``` r
boxplot(ipatsvars_2_3_4$diff, main = "Atšķirības 10m vs 100m LAD (300m centri)")
```

![](Uzd07_Butkevica_files/figure-gfm/2_3un_2_4-1.png)<!-- --> Atšķirības
ir nelielas, nedaudz zem nulles, kas nozīmē, ka 10 m rastrā īpatsvars
mēdz būt mazliet lielāks nekā 100 m rastrā.

## Kādas ir maksimālās teorētiski sagaidāmās atšķirības?

Es nezinu kā novērtēt maksimālas *teoretiski* sagaidāmas atšķirības.

Maksimālās absolūtās atšķirības starp platības īpatsvariem:

``` r
max(abs(ipatsvars_2_1_2$diff), na.rm = TRUE)  # 100 m centri
```

    ## [1] 0.06526746

``` r
max(abs(ipatsvars_2_3_4$diff), na.rm = TRUE)  # 300 m centri
```

    ## [1] 0.06293391

Tas norāda, ka maksimālā atšķirība starp 10 m un 100 m rastra aprēķiniem
nepārsniedz aptuveni 6.5% no teritorijas platības.
