Otrais uzdevums
================
Betija Rubene
2025-01-08

Darbam nepieciešamo bibliotēku ielāde

``` r
library(sf)
library(sfarrow)
library(microbenchmark)
library(archive)
library(curl)
library(tidyverse)
```

# Pirmā daļa - lejupielāde, atarhivēšana, failu izmēri un ielasīšanas ātrums

Centra virsmežniecības faila lejupielāde

``` r
url <- "https://data.gov.lv/dati/lv/dataset/40014c0a-90f5-42be-afb2-fe3c4b8adf92/resource/392dfb67-eeeb-43c2-b082-35f9cf986128/download/centra.7z"
dest <- "C:/Users/ruben/Documents/HiQBioDiv_macibas_fork/centra_mezi.7z"

curl_download(url, destfile = dest)
file.exists(dest) # fails ir atrodams direktorijā
```

    ## [1] TRUE

Atarhivēšana

``` r
archive_extract(dest, dir = "C:/Users/ruben/Documents/HiQBioDiv_macibas_fork/centra_mezi")
```

Failu apskate atarhivētajā mapē

``` r
list.files("../centra_mezi")
```

    ##  [1] "centra_mezi.parquet"        "centra_mezi_labots.parquet"
    ##  [3] "nodala2651.cpg"             "nodala2651.dbf"            
    ##  [5] "nodala2651.gpkg"            "nodala2651.parquet"        
    ##  [7] "nodala2651.prj"             "nodala2651.sbn"            
    ##  [9] "nodala2651.sbx"             "nodala2651.shp"            
    ## [11] "nodala2651.shp.xml"         "nodala2651.shx"            
    ## [13] "nodala2652.cpg"             "nodala2652.dbf"            
    ## [15] "nodala2652.prj"             "nodala2652.sbn"            
    ## [17] "nodala2652.sbx"             "nodala2652.shp"            
    ## [19] "nodala2652.shp.xml"         "nodala2652.shx"            
    ## [21] "nodala2653.cpg"             "nodala2653.dbf"            
    ## [23] "nodala2653.prj"             "nodala2653.sbn"            
    ## [25] "nodala2653.sbx"             "nodala2653.shp"            
    ## [27] "nodala2653.shp.xml"         "nodala2653.shx"            
    ## [29] "nodala2654.cpg"             "nodala2654.dbf"            
    ## [31] "nodala2654.prj"             "nodala2654.sbn"            
    ## [33] "nodala2654.sbx"             "nodala2654.shp"            
    ## [35] "nodala2654.shp.xml"         "nodala2654.shx"            
    ## [37] "nodala2655.cpg"             "nodala2655.dbf"            
    ## [39] "nodala2655.prj"             "nodala2655.sbn"            
    ## [41] "nodala2655.sbx"             "nodala2655.shp"            
    ## [43] "nodala2655.shp.xml"         "nodala2655.shx"

Cik daudz vietas aizņem ESRI shapefile?

``` r
file.info("../centra_mezi/nodala2651.shp")$size
```

    ## [1] 27092404

Izveidoju GeoPackage failu. Cik daudz vietas tas aizņem?

``` r
nod2651 <- st_read("../centra_mezi/nodala2651.shp", quiet = TRUE)
st_write(nod2651, "../centra_mezi/nodala2651.gpkg",
         quiet = TRUE, ,
         delete_layer = TRUE)
file.info("../centra_mezi/nodala2651.gpkg")$size
```

    ## [1] 57794560

Izveidoju GeoParquet failu. Cik daudz vietas tas aizņem?

``` r
st_write_parquet(nod2651, "../centra_mezi/nodala2651.parquet")
file.info("../centra_mezi/nodala2651.parquet")$size
```

    ## [1] 22118733

Tātad visvairāk vietas aizņem GeoPackage, shapefile aptuveni divreiz
mazāk, geoparquet fails aizņem vēl mazliet mazāk vietas.

Kā ir ar failu ielasīšanas ātrumiem? Ar microbenchmark ielasu katru
failu 10 reizes

``` r
atrumsshp <- microbenchmark(st_read  ("../centra_mezi/nodala2651.shp", quiet = TRUE),
                         times = 10 )
atrumsgpkg <- microbenchmark(st_read  ("../centra_mezi/nodala2651.gpkg", quiet = TRUE),
                            times = 10)
atrumspar <- microbenchmark(st_read_parquet ("../centra_mezi/nodala2651.parquet", quiet = TRUE),
                            times = 10)
```

``` r
# vidējais ātrums un konvertācija uz milisekundēm
mean_shp <- mean(atrumsshp$time) / 1e6  
mean_gpkg <- mean(atrumsgpkg$time) / 1e6
mean_parquet <- mean(atrumspar$time) / 1e6

print(mean_shp)
```

    ## [1] 6430.097

``` r
print(mean_gpkg)
```

    ## [1] 3914.038

``` r
print(mean_parquet)
```

    ## [1] 1276.276

Geoparquet fails ielasās krietni ātrāk nekā pārējie formāti

# Otrais uzdevums - nodaļu datu apvienošana

Tā kā geoparquet faili aizņem vismazāk vietas un ielasās visātrāk,
turpmāk darbošos ar tiem. Sākotnēji nepieciešams visu nodaļu failus
transformēt geoparquet formātā.

``` r
list.files("../centra_mezi", pattern = "\\.shp$") # ir pieci shapefiles
```

    ## [1] "nodala2651.shp" "nodala2652.shp" "nodala2653.shp" "nodala2654.shp"
    ## [5] "nodala2655.shp"

``` r
# failu ielasīšana R vidē
nod2651 <- st_read("../centra_mezi/nodala2651.shp", quiet = TRUE)
nod2652 <- st_read("../centra_mezi/nodala2652.shp", quiet = TRUE)
nod2653 <- st_read("../centra_mezi/nodala2653.shp", quiet = TRUE)
nod2654 <- st_read("../centra_mezi/nodala2654.shp", quiet = TRUE)
nod2655 <- st_read("../centra_mezi/nodala2655.shp", quiet = TRUE)

# transformēšana. Vieglāk apvienot shapefiles un tad gatavo apvienoto failu transformēt parquet formātā.
centra_mezi <- rbind(nod2651,nod2652,nod2653,nod2654,nod2655)
st_write_parquet(centra_mezi, "../centra_mezi/centra_mezi.parquet")
centra_mezi <- st_read_parquet("../centra_mezi/centra_mezi.parquet")
```

Jānoskaidro, vai visas ģeometrijas ir multipolygon

``` r
summary(centra_mezi$geometry) # visas tiešām ir multipolygon
```

    ##  MULTIPOLYGON     epsg:3059 +proj=tmer... 
    ##        505660             0             0

Vai ir kādas tukšas vai nekorektas ģeometrijas?

``` r
tuksas <- st_is_empty(centra_mezi)
summary(tuksas) # ir sešas tukšas ģeometrijas
```

    ##    Mode   FALSE    TRUE 
    ## logical  505654       6

``` r
nekorektas <- st_is_valid(centra_mezi)
summary(nekorektas) # ir 274 nekorektas ģeometrijas
```

    ##    Mode   FALSE    TRUE 
    ## logical     274  505386

Ar tukšajām ģeometrijām īsti neko iesākt nevar, tāpēc no tām atbrīvojos.
Nederīgās ģeometrijas var padarīt par derīgām ar funkciju
‘st_make_valid’

``` r
centra_mezi <- centra_mezi[!tuksas, ]
st_make_valid(centra_mezi)
```

Saglabāju jauno laboto parquet failu, lai ir skaidrs, kurš ir labotais
fails, bet arī oriģinālais vajadzības gadījumā paliek pieejams

``` r
st_write_parquet(centra_mezi, "../centra_mezi/centra_mezi_labots.parquet")

centra_mezi <- st_read_parquet("../centra_mezi/centra_mezi_labots.parquet")
```

# Trešais uzdevums - priežu mežaudzes

Sākumā jāaprēķina priežu pirmā stāva šķērslaukuma īpatsvars, ar vērtību
“1” atzīmējot tās mežaudzes, kurās priedes šķērslaukuma īpatsvars
pirmajā stāvā ir vismaz 75% un ar “0” tās mežaudzes, kurās īpatsvars ir
mazāks, pārējos ierakstus atstājot bez vērtībām.

Katrā stāvā var būt līdz piecām sugām, 1. stāva 1. suga atrodama kolonnā
‘s10’ (attiecīgi otrā suga būs kolonnā ‘s11’, utt.). Datubāzē priedēm ir
trīs kodi - priede (1), citas priedes (14) un ciedru priede (22). Tā pā
prasīts aprēķināt kumulatīvo dažādām sugām, ņemšu vērā visas.

Šķērslaukums atrodams kolonnā ‘g10’ (attiecīgi otrās sugas šķērslaukums
būs kolonnā ‘g11’, utt.)

Sākumā jāaprēķina kopējais šķērslaukums un priežu šķērslaukums, lai
varētu aprēķināt priežu īpatsvaru.

``` r
centra_mezi <- centra_mezi %>% 
  mutate (skerslaukums = g10 + g11 + g12 + g13 + g14)

centra_mezi <- centra_mezi %>% 
  mutate (pinus_skersl = 
ifelse(s10 %in% c("1", "14", "22"), g10, 0) +
ifelse(s11 %in% c("1", "14", "22"), g11, 0) +
ifelse(s12 %in% c("1", "14", "22"), g12, 0) +
ifelse(s13 %in% c("1", "14", "22"), g13, 0) +
ifelse(s14 %in% c("1", "14", "22"), g14, 0))
  
centra_mezi <- centra_mezi %>% 
  mutate (prop_priedes = pinus_skersl/skerslaukums*100)

# noņemu liekās kolonnas, kas bija vajadzīgas aprēķinam, kopējais šķērslaukums vēl noderēs 4. uzdevumā
centra_mezi <- centra_mezi %>% select(-pinus_skersl,)
```

Tagad jāizveido kolonna, kurā ar “1” atzīmēt mežaudzes, kurās priežu
šķērslaukuma īpatsvars pārsniedz 75%, ar “0” ,kur tas ir zemāks. Bez
vērtības paliks visi lauki bez vērtības kolonnā ‘prop_priedes’ (tās ir
mežaudzes, kurās koku šķērslaukums ir bijis 0)

``` r
centra_mezi <- centra_mezi %>%
  mutate(
    PriezuMezi = case_when(
      !is.na(prop_priedes) & prop_priedes >= 75 ~ 1,  
      !is.na(prop_priedes) & prop_priedes < 75 ~ 0,  
    )
  )
```

Kāds ir priežu mežaudžu īpatsvars no visām mežaudzēm? To var vienkārši
aprēķināt, saskaitot vērtības rindās ar vērtību “1” un to dalot ar
kopējo rindu skaitu un reizinot ar 100

``` r
pinus_mezu_ipatsvars <- sum(centra_mezi$PriezuMezi == 1, na.rm = TRUE) / nrow(centra_mezi) * 100
print(pinus_mezu_ipatsvars) 
```

    ## [1] 17.83927

Tātad priežu meži sastāda 17,83927% no visām mežaudzēm. Bet jāņem vērā,
ka tie ir % no visām mežaudzēm, pat ja pirmajā stāvā nav nevienas koku
sugas - tātad mežaudzes tikai teorijā.

Aprēķinu arī īpatsvaru no mežaudzēm, kurās reāli ir koki

``` r
pinus_mezu_ipatsvars2 <- sum(centra_mezi$PriezuMezi == 1, na.rm = TRUE) / sum(centra_mezi$PriezuMezi %in% c(1, 0), na.rm = TRUE) * 100
print(pinus_mezu_ipatsvars2)
```

    ## [1] 25.29578

Tātad priežu meži sastāda 25.29578% no visiem mežiem.

# Ceturtais uzdevums - mežaudžu klasifikācija

Sākumā jāpieņem lēmumi par koku sugu klasifikāciju, Ar skujkokiem
neskaidrību nav, problēmas sagādā platlapju un šaurlapju iedalījums -
par dažām sugām uzreiz skaidrs, bet citas sagādā problēmas - nevarēju
atrast informācijas avotu, kurā visas koku sugas būtu skaidri iedalītas
šajās divās kategorijās.

Ja tiešām nav avotu, uz kuriem balstīties, šādā gadījumā projekta
ietvaros būtu jāpieņem kopīgi lēmumi un visiem pie tiem jāpieturas, lai
vismaz pieeja būtu vienota. Zemāk redzama mana pielietotā pieeja
dalījumam:

``` r
skujkoki <- c(1,3,13,14,15,22,23,28,29)
platlapji <- c(8,9,10,11,12,16,17,18,21,24,25,26,27,32,35,50,61,62,63,64,65,66,67,69)
saurlapji <- c(4,6,8,9,19,20,68)
```

Par iedalījumu skujkoku, šaurlapju, platlapju un jauktos mežos - arī
šajā gadījumā man nav zināms vispārpielietojams skaidrs slieksnis, tomēr
būtu jānosaka vienota pieeja. Lai būtu konsekvence ar iepriekš
pielietoto slieksni priežu mežu klasifikācijā, kā slieksni izmantošu
75%. (Ja slieksni tomēr būtu nepieciešams mainīt, to ar komandrindām
varētu viegli izdarīt). Ja neviena sugu grupa nesasniedz šo slieksni,
klasificēšu to kā jauktu koku mežu. Klasifikāciju veikšu, balstoties uz
pirmajā stāvā sastopamajām koku sugām.

Bet intereses pēc - cik daudzām mežaudzēm vispār ir minēts vairāk par
vienu sugu, kur būtu jādomā par slieksni?

``` r
viena_suga <- centra_mezi %>%
  filter(g11 == 0) %>%
  nrow()

print(viena_suga) # aptuveni pusei
```

    ## [1] 254958

Sāku ar aprēķinu katras koku grupas kopējam šķērslaukumam

``` r
centra_mezi <- centra_mezi %>% 
  mutate (skujkoki = 
ifelse(s10 %in% skujkoki, g10, 0) +
ifelse(s11 %in% skujkoki, g11, 0) +
ifelse(s12 %in% skujkoki, g12, 0) +
ifelse(s13 %in% skujkoki, g13, 0) +
ifelse(s14 %in% skujkoki, g14, 0))

centra_mezi <- centra_mezi %>% 
  mutate (platlapji = 
ifelse(s10 %in% platlapji, g10, 0) +
ifelse(s11 %in% platlapji, g11, 0) +
ifelse(s12 %in% platlapji, g12, 0) +
ifelse(s13 %in% platlapji, g13, 0) +
ifelse(s14 %in% platlapji, g14, 0))

centra_mezi <- centra_mezi %>% 
  mutate (saurlapji = 
ifelse(s10 %in% saurlapji, g10, 0) +
ifelse(s11 %in% saurlapji, g11, 0) +
ifelse(s12 %in% saurlapji, g12, 0) +
ifelse(s13 %in% saurlapji, g13, 0) +
ifelse(s14 %in% saurlapji, g14, 0))
```

Tad aprēķinu katras grupas īpatsvaru mežaudzē

``` r
centra_mezi <- centra_mezi %>% 
  mutate (prop_skujkoki = skujkoki/skerslaukums*100)

centra_mezi <- centra_mezi %>% 
  mutate (prop_saurlapji = saurlapji/skerslaukums*100)

centra_mezi <- centra_mezi %>% 
  mutate (prop_platlapji = platlapji/skerslaukums*100)
```

Visbeidzot iedalu mežaudzes četrās kategorijās. Kodu atšifrējums kolonnā
meza_veids: 1 - skujkoku meži 2 - šaurlapju meži 3 - platlapju meži 4 -
jauktu koku meži

Vērtība “4” jāatiecina uz mežaudzēm, kurās neviens koku veids
nepārsniedz 75%, tomēr nevar būt tā, ka koku nav vispār. Mežaudzes bez
kokiem paliek bez vērtībām.

``` r
centra_mezi <- centra_mezi %>%
  mutate(
    meza_veids = case_when(
      !is.na(prop_skujkoki) & prop_skujkoki >= 75 ~ 1,  
      !is.na(prop_saurlapji) & prop_saurlapji >= 75 ~ 2,
      !is.na(prop_platlapji) & prop_platlapji >= 75 ~ 3,
      (!is.na(prop_skujkoki) & prop_skujkoki > 0) |
      (!is.na(prop_saurlapji) & prop_saurlapji > 0) |
      (!is.na(prop_platlapji) & prop_platlapji > 0) ~ 4
    )
  )
```

Kāds ir katra veida mežu īpatsvars no visiem ierakstiem?

``` r
mezu_ipatsvars <- centra_mezi %>%
  group_by(meza_veids) %>%
  summarise(count = n()) %>%     
  mutate(percentage = (count / sum(count)) * 100)
print(mezu_ipatsvars$percentage) 
```

    ## [1] 29.278123 27.712626  1.727861 11.804119 29.477271

Skujkoku meži - 29.27%, šaurlapju meži - 27.71%, platlapju meži - 1,73%,
jauktu koku meži - 11,80%, meži bez kokiem - 29,48%

Bet šie, tāpat kā 3. uzdevumā, ir % no visām mežaudzēm.

Ja interesējoši ir % no mežiem (ir koki), tad sākumā filtrēju vērtības,
kurās meza_veids nav NA:

``` r
mezu_ipatsvars2 <- centra_mezi %>%
  filter(!is.na(meza_veids)) %>% 
  group_by(meza_veids) %>%
  summarise(count = n()) %>%     
  mutate(percentage = (count / sum(count)) * 100)
print(mezu_ipatsvars2$percentage) 
```

    ## [1] 41.515868 39.296020  2.450077 16.738035

Tātad skujkoku meži - 41,52%, šaurlapju meži - 39,30%, platlapju meži -
2,45%, jauktu koku meži - 16,74%
