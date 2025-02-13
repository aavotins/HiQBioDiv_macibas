6uzd_Rubene
================
Betija Rubene
2025-02-10

## Sestais uzdevums: dažādu slāņu savienošana vienotai ainavas aprakstīšanai

Darbam nepieciešamās pakotnes

``` r
library(terra)
library(sfarrow)
library(tidyverse)
library(fasterize)
library(sf)
```

Uzdevuma ietvaros veidojamo rastru pārklājumam un šūnu novietojumam ir
jāsakrīt ar projekta Zenodo repozitorijā ievietoto references slāni
LV10m_10km.tif. Rasterizēšanas nosacījumam izmantojiet šūnas centru.

# 1) LAD datu rasterizēšana

Rasterizējiet Lauku atbalsta dienestā reģistrētos laukus, izveidojot
klasi “100” (vai izmantojiet trešajā uzdevumā sagatavoto slāni, ja tas
ir korekts).

Trešajā uzdevumā izveidotais rastra slānim vajadzētu būt korektam
(veidots ar noklusējuma argumentu `touches = FALSE`, lai rasterizēšanas
nosacījumam tiktu izmantots šūnas centrs), vienīgi lauki atzīmēti ar
vērtību “1” nevis “100” un pārējā analīzes teritorija, kas ietilpst
Latvijas sauszemes teritorijā, aizpildīta ar vērtībām “0”. To nomainu ar
funkciju subst.

``` r
LAD10m <- rast("../LAD_dati/LAD_lauki_10m.tif")
crs(LAD10m) # LKS-92
```

    ## [1] "PROJCRS[\"LKS-92 / Latvia TM\",\n    BASEGEOGCRS[\"LKS-92\",\n        DATUM[\"Latvian geodetic coordinate system 1992\",\n            ELLIPSOID[\"GRS 1980\",6378137,298.257222101,\n                LENGTHUNIT[\"metre\",1]]],\n        PRIMEM[\"Greenwich\",0,\n            ANGLEUNIT[\"degree\",0.0174532925199433]],\n        ID[\"EPSG\",4661]],\n    CONVERSION[\"Latvian Transverse Mercator\",\n        METHOD[\"Transverse Mercator\",\n            ID[\"EPSG\",9807]],\n        PARAMETER[\"Latitude of natural origin\",0,\n            ANGLEUNIT[\"degree\",0.0174532925199433],\n            ID[\"EPSG\",8801]],\n        PARAMETER[\"Longitude of natural origin\",24,\n            ANGLEUNIT[\"degree\",0.0174532925199433],\n            ID[\"EPSG\",8802]],\n        PARAMETER[\"Scale factor at natural origin\",0.9996,\n            SCALEUNIT[\"unity\",1],\n            ID[\"EPSG\",8805]],\n        PARAMETER[\"False easting\",500000,\n            LENGTHUNIT[\"metre\",1],\n            ID[\"EPSG\",8806]],\n        PARAMETER[\"False northing\",-6000000,\n            LENGTHUNIT[\"metre\",1],\n            ID[\"EPSG\",8807]]],\n    CS[Cartesian,2],\n        AXIS[\"northing (X)\",north,\n            ORDER[1],\n            LENGTHUNIT[\"metre\",1]],\n        AXIS[\"easting (Y)\",east,\n            ORDER[2],\n            LENGTHUNIT[\"metre\",1]],\n    USAGE[\n        SCOPE[\"Engineering survey, topographic mapping.\"],\n        AREA[\"Latvia - onshore and offshore.\"],\n        BBOX[55.67,19.06,58.09,28.24]],\n    ID[\"EPSG\",3059]]"

``` r
LAD10m <- subst(LAD10m, 1, 100) # aizstāj vērtības 1 ar 100
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
LAD10m <- subst(LAD10m, 0, NA) # aizstāj vērtības 0 ar NA
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
plot(LAD10m)
```

![](Uzd06_Rubene_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

# 2) MVR datu rastra slāņu izveidošana

Izveidojiet rastra slāņus ar skujkoku (klase “204”), šaurlapju (klase
“203”), platlapju (klase “202”) un jauktu koku mežiem (klase “201”) no
sevis ierosinātās klasifikācijas otrajā uzdevumā.

Tātad būs jāstrādā ar centra virsmežniecības vektordatiem:

``` r
MVRdati <- st_read_parquet("../centra_mezi/centra_mezi_labots.parquet")
```

Iepriekš nebiju pamanījusi, ka šajos datos ne viss ir mežaudzes, tas
uzrādās laukā zkat - ar kodu 10 atzīmētas mežaudzes. Atlasīšu tikai tās:

``` r
MVRdati <- MVRdati %>%
  filter(zkat == 10)
# filtrējot tika atmesti ~ 40 000 lauki
```

Atkal jānodefinē, kuras koku sugas ierindoju kurā kategorijā (pieeju
nokopēju no 2. uzdevuma):

``` r
skujkoki <- c(1,3,13,14,15,22,23,28,29)
platlapji <- c(8,9,10,11,12,16,17,18,21,24,25,26,27,32,35,50,61,62,63,64,65,66,67,69)
saurlapji <- c(4,6,19,20,68)
```

Izmantošu komandrindas no 2. uzdevuma, lai ieviestu kolonnas ar meža
tipiem, kuros uzrādās procentuālais šķērslaukums katrā koku grupā. Gan
jau to kaut kā kompaktāk var uzrakstīt, bet ir tikai trīs grupas, tāpēc
es tam šobrīd laiku netērēšu.

Turpinot pildīt uzdevumu, pamanīju, ka ir rindas, kurās ir norādīta
valdošā suga un tās vecums, tomēr šķērslaukums ir 0. Pētot datus
sapratu, ka vietās, kur šķērslaukuma kolonna ir tukša, ir aizpildīta
kolonna n10 - tā atspoguļo koku skaitu (bez mērvienības - tātad visā
teritorijā?). Man šis šķita dīvaini - katrā ziņā es negribēju pavisam
pazaudēt datus par šiem mežiem, jo šāda īpatnība datos sastopama gana
bieži, tapēc pielietoju to pašu principu kā šķērslaukumam (doma -
skaitam arī var aprēķināt proporciju, ja mērķis ir tikai noteikt meža
tipu. Saskatu šajā dažas potenciālās problēmas, tomēr tas šobrīd ir
labākais, kas ir pieejams)

``` r
sum(MVRdati$n10 > 1)
```

    ## [1] 106353

``` r
sum(MVRdati$n11 > 1)
```

    ## [1] 43165

``` r
# kopējā šķērslaukuma vai koku skaita aprēķins
# pmax() atgriež lielāko summu no abiem variantiem
MVRdati <- MVRdati %>%
  mutate(kopa_koki = pmax(g10 + g11 + g12 + g13 + g14, 
                              n10 + n11 + n12 + n13 + n14))


# šķērslaukuma aprēķins skujkokiem, platlapjiem un šaurlapjiem
MVRdati <- MVRdati %>% 
  mutate (skujkoki = 
ifelse(s10 %in% skujkoki, ifelse(g10 > 0, g10, n10), 0) +
ifelse(s11 %in% skujkoki, ifelse(g11 > 0, g11, n11), 0) +
ifelse(s12 %in% skujkoki, ifelse(g12 > 0, g12, n12), 0) +
ifelse(s13 %in% skujkoki, ifelse(g13 > 0, g13, n13), 0) +
ifelse(s14 %in% skujkoki, ifelse(g14 > 0, g14, n14), 0))

MVRdati <- MVRdati %>% 
  mutate (platlapji = 
ifelse(s10 %in% platlapji, ifelse(g10 > 0, g10, n10), 0) +
ifelse(s11 %in% platlapji, ifelse(g11 > 0, g11, n11), 0) +
ifelse(s12 %in% platlapji, ifelse(g12 > 0, g12, n12), 0) +
ifelse(s13 %in% platlapji, ifelse(g13 > 0, g13, n13), 0) +
ifelse(s14 %in% platlapji, ifelse(g14 > 0, g14, n14), 0))

MVRdati <- MVRdati %>% 
  mutate (saurlapji = 
ifelse(s10 %in% saurlapji, ifelse(g10 > 0, g10, n10), 0) +
ifelse(s11 %in% saurlapji, ifelse(g11 > 0, g11, n11), 0) +
ifelse(s12 %in% saurlapji, ifelse(g12 > 0, g12, n12), 0) +
ifelse(s13 %in% saurlapji, ifelse(g13 > 0, g13, n13), 0) +
ifelse(s14 %in% saurlapji, ifelse(g14 > 0, g14, n14), 0))
```

Tad aprēķinu katras grupas īpatsvaru mežaudzē:

``` r
MVRdati <- MVRdati %>% 
  mutate (prop_skujkoki = skujkoki/kopa_koki*100)

MVRdati <- MVRdati %>% 
  mutate (prop_saurlapji = saurlapji/kopa_koki*100)

MVRdati <- MVRdati %>% 
  mutate (prop_platlapji = platlapji/kopa_koki*100)
```

Visbeidzot ieviešu kolonnu, kurā nodefinēts meža tips, izmantojot 75%
slieksni (pieder konkrētajai kategorijai, ja šķērslaukums sasniedz
vismaz 75%, ja nevienā grupā slieksnis netiek sasniegts, tad tas
klasificēts kā jauktu koku mežs).

Kategoriju kodējums atbilstoši aprakstā prasītajam:  
Skujkoku meži - 204  
Šaurlapju meži - 203  
Platlapju meži - 202  
Jauktu koku meži - 201

(Ja nav datu par kokiem, funkcija atgriezīs vērtību `NA`. Var jau būt,
ka citās situācijās būtu vajadzīgs šai grupai iedot vērtību, tomēr
šoreiz es tam neredzu jēgu un arī uzdevuma nosacījumos par to nekas nav
teikts - vajadzīgs tikai kodēt pēc meža tipa.)

``` r
MVRdati <- MVRdati %>%
  mutate(
    meza_grupa = case_when(
      !is.na(prop_skujkoki) & prop_skujkoki >= 75 ~ 204,  
      !is.na(prop_saurlapji) & prop_saurlapji >= 75 ~ 203,
      !is.na(prop_platlapji) & prop_platlapji >= 75 ~ 202,
      (!is.na(prop_skujkoki) & prop_skujkoki > 0) |
      (!is.na(prop_saurlapji) & prop_saurlapji > 0) |
      (!is.na(prop_platlapji) & prop_platlapji > 0) ~ 201
    )
  )

summary(MVRdati$meza_grupa) # Nav nullīšu :)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   201.0   202.0   203.0   202.8   204.0   204.0

Katram meža tipam filtrēšu tabulas pēc klases un tad rasterizēšu.
Sākotnēji visu rasterizēju vienā slānī, bet tad pārskatīju uzdevuma
nosacījumus, kur teikts, ka katrai klasei jābut savam slānim. Saprotu
domu par slāņu secības noteikšanu, tomēr neredzu, kāpēc tas šajā
gadījumā ir nepieciešams, ja jau iepriekš tika pārbaudīts, ka meža datos
ģeometrijas nepārklājas. Saredzu problēmas, ja mēs izmantotu argumentu
`touches = TRUE`, tomēr šoreiz kā atskaites punkts jāizmanto pikseļa
centrs, kurš reizē var atrasties tikai vienā MVR poligonā. (protams,
citādi ir ar LAD datiem). Tomēr pieturos pie nosacījumiem un veidoju
katrai klasei savu rastru.

Man no 3. uzdevuma prātā bija palicis, ka ar rasterize viss notika
ātrāk, bet pamēģināšu arī fasterize. Pārbaudīju un tā nav (2 min pret
1,5 min) - turpmāk apsvēršu izmantot fasterize, tomēr šoreiz palikšu pie
ar rasterize izveidotā, jo rezultāts ir terra objekts. Ieviesīšu ciklu,
jo ir četras klases.

``` r
crs(MVRdati) # ir LKS-92
```

    ## [1] "PROJCRS[\"LKS-92 / Latvia TM\",\n    BASEGEOGCRS[\"LKS-92\",\n        DATUM[\"Latvian geodetic coordinate system 1992\",\n            ELLIPSOID[\"GRS 1980\",6378137,298.257222101,\n                LENGTHUNIT[\"metre\",1]]],\n        PRIMEM[\"Greenwich\",0,\n            ANGLEUNIT[\"degree\",0.0174532925199433]],\n        ID[\"EPSG\",4661]],\n    CONVERSION[\"Latvian Transverse Mercator\",\n        METHOD[\"Transverse Mercator\",\n            ID[\"EPSG\",9807]],\n        PARAMETER[\"Latitude of natural origin\",0,\n            ANGLEUNIT[\"degree\",0.0174532925199433],\n            ID[\"EPSG\",8801]],\n        PARAMETER[\"Longitude of natural origin\",24,\n            ANGLEUNIT[\"degree\",0.0174532925199433],\n            ID[\"EPSG\",8802]],\n        PARAMETER[\"Scale factor at natural origin\",0.9996,\n            SCALEUNIT[\"unity\",1],\n            ID[\"EPSG\",8805]],\n        PARAMETER[\"False easting\",500000,\n            LENGTHUNIT[\"metre\",1],\n            ID[\"EPSG\",8806]],\n        PARAMETER[\"False northing\",-6000000,\n            LENGTHUNIT[\"metre\",1],\n            ID[\"EPSG\",8807]]],\n    CS[Cartesian,2],\n        AXIS[\"northing (X)\",north,\n            ORDER[1],\n            LENGTHUNIT[\"metre\",1]],\n        AXIS[\"easting (Y)\",east,\n            ORDER[2],\n            LENGTHUNIT[\"metre\",1]],\n    USAGE[\n        SCOPE[\"Engineering survey, topographic mapping.\"],\n        AREA[\"Latvia - onshore and offshore.\"],\n        BBOX[55.67,19.06,58.09,28.24]],\n    ID[\"EPSG\",3059]]"

``` r
ref10m <- rast("../ref_rastri/LV10m_10km.tif") # 10m references rastrs
ref10m_cropped <- crop(ref10m,MVRdati) # izgriežu analīzes teritoriju
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
klases <- unique(MVRdati$meza_grupa)

rastri2uzd <- list() # saglabāšu rastrus sarakstā

# cikls
for(i in klases){
MVRdati_filtered <- MVRdati %>%  
  filter(meza_grupa == i)
rastri2uzd[[as.character(i)]] <- rasterize(MVRdati_filtered,ref10m_cropped, field = "meza_grupa")
}
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

``` r
rm(platlapji, saurlapji, skujkoki)
```

# 3) Papildināšana ar vecuma grupu

Papildiniet otrā punkta rastrus, izveidojot jaunus slāņus ar informāciju
par mežaudzes vecuma grupu kā klases vērtības otro ciparu. Dalījumam
mežaudžu vecuma grupās, izmantojiet vecumgrupas un vecumklases un,
aprēķinot galvenās cirtes vecumu, pieņemiet “I un augstāka” bonitāti.

DBF spefizikācijas failā ir informācija par mežaudžu vecuma grupu
klasifikatoru, tomēr MVR datos es šādu kolonnu neatrodu…

``` r
"vgr" %in% colnames(MVRdati) # :(
```

    ## [1] FALSE

Tāpēc vecumgrupas jāaprēķina pašai, ņemot vērā ieteiktajos avotos
aprakstīto pieeju.

Vispirms jāņem vērā, ka vecumgrupu nosaka
[vecumklase](https://www.letonika.lv/groups/default.aspx?cid=36193&r=7&lid=36193&q=&h=1166),
kas ir atkarīga no koku sugas augšanas ātruma:

Skujkoku un [cieto lapkoku](https://tezaurs.lv/mwe:395317) audzēm - 20
gadi;  
Mīkstajiem lapu kokiem — 10 gadi;  
Īpaši ātraudzīgajiem kokiem (baltalkšņiem, blīgznām un vītoliem) — 5
gadi.

Visas tās koku sugas, kuras neietilpa pirmajā vai pēdējā kategorijā, es
ieliku otrajā.

``` r
unique(MVRdati$s10) 
```

    ##  [1] "16" "9"  "8"  "1"  "4"  "6"  "10" "3"  "11" "20" "24" "21" "32" "19" "12"
    ## [16] "25" "13" "64" "23" "17" "15" "63" "14" "35" "61" "50" "68" "67" "22"

``` r
klase20 <- c(1,3,10,11,13,14,15,16,22,23,28,29,61,64)
klase10 <- c(4,6,8,12,17,19,24,25,32,35,50,63,67,68)
klase5 <- c(9,20,21)
```

Lai vienkāršotu domu gājienu, uzreiz piešķiršu katrai mežaudzei
vecumklasi:

``` r
MVRdati <- MVRdati %>%
  mutate(
    vecumklase = case_when(
      s10 %in% klase20 ~ 20,
      s10 %in% klase10 ~ 10,
      s10 %in% klase5  ~ 5,
      TRUE ~ NA_real_  # visi, kas neatbilst, būs NA
    ))
```

Lai noteiktu
[vecumgrupas](https://www.letonika.lv/groups/default.aspx?cid=36192&r=7&lid=36192&q=&h=1144),
vajadzīga informācija par ciršanas vecumu. Galvenās cirtes vecums
atrodams [meža likumā](https://likumi.lv/ta/id/2825#p9). Uzdevuma
nosacījumos norādīts, ka jāizmanto “I un augstāka” bonitāte, jo MVR
datos nav šādas kolonnas nav.

Galvenās cirtes vecumi kategorijai “I un augstāka” bonitāte:  
Ozols, priede un lapegle - 101;  
Egle, osis, liepa, goba, vīksna un kļava - 81 gadi;  
Bērzs, melnalksnis - 71;  
Apse - 41

Te arī jāpieņem lēmumi… `klase20` atstāšu, kā ir, atbilstoši cirtes
vecumam 101. Vecumos 71 un 41 neko nepievienošu - viss, kas nav minēts
meža likumā, būs otrajā kategorijā (81)

``` r
cirte101 <- c(1,3,10,11,13,14,15,16,22,23,28,29,61,64)
cirte81 <- c(4,6,8,9,12,17,19,20,21,24,25,32,35,50,63,67,68)
cirte71 <- c(4,6)
cirte41 <- c(8)
```

Arī uzreiz piešķiršu katrai mežaudzei cirtes vecumu:

``` r
MVRdati <- MVRdati %>%
  mutate(
    g_cirtes_vecums = case_when(
      s10 %in% cirte101 ~ 101,
      s10 %in% cirte81  ~ 81,
      s10 %in% cirte71  ~ 71,
      s10 %in% cirte41  ~ 41,
      TRUE ~ NA_real_  # visi, kas neatbilst, būs NA
    ))
```

Tagad beidzot varu ķerties pie vecumgrupas piešķiršanas.

Atkarībā no valdošās sugas koku vecuma un tai noteiktā ciršanas vecuma,
kokaudzi ieskaita vienā no 5 vecumgrupām:

1)  jaunaudzē (1.-2. vecumklase),  
2)  vidēja vecuma audzē (viss starp jaunaudzi un briestaudzi),
3)  briestaudzē (1 vecumklase pirms ciršanas vecuma),
4)  pieaugušā audzē (divas vecumklases pēc ciršanas vecuma) vai
5)  pāraugušā audzē (viss virs pieaugušas audzes).

<!-- -->

1.  stāva 1. sugas vecums pieejams kolonnā a10.

``` r
MVRdati <- MVRdati %>%
  mutate(
    vecumgrupa = case_when(
      a10 <= 2 * vecumklase ~ 1,  
      a10 > 2 * vecumklase & a10 < (g_cirtes_vecums - vecumklase) ~ 2,
      a10 >= (g_cirtes_vecums - vecumklase) & a10 < g_cirtes_vecums ~ 3,
      a10 >= g_cirtes_vecums & a10 < (g_cirtes_vecums + 2 * vecumklase) ~ 4,
      a10 >= (g_cirtes_vecums + 2 * vecumklase) ~ 5,
      TRUE ~ NA_real_ # visi, kas neatbilst, būs NA
    ))
```

Tagad jāpapildina rastrs ar jauniegūtajām vecuma grupām.

Varbūt te bija domāts darīt kaut ko sarežģītāku, bet es nesaskatu
iemeslu, kāpēc kolonnas ar meža veida klasēm un vecuma grupām nevarētu
vienkārši saskaitīt jaunā kolonnā (vecuma grupas reizinot ar 10)…

``` r
MVRdati$klase <- MVRdati$meza_grupa + MVRdati$vecumgrupa *10
```

Katram meža tipam ciklā filtrēšu tabulas pēc tipa un tad rasterizēšu. Ja
pareizi saprotu, katrai unikālajai klasei savs rastrs (jo prasīts
izveidot jaunus slāņus).

``` r
klases <- unique(MVRdati$klase) # saraksts ar unikālajām vērtībām

rastri3uzd <- list() # saglabāšu rastrus sarakstā

# cikls
for(i in klases){
MVRdati_filtered <- MVRdati %>%  
  filter(klase == i)
rastri3uzd[[as.character(i)]] <- rasterize(MVRdati_filtered,ref10m_cropped, field = "klase")
}
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

``` r
# un tos arī saglabāšu ar klases nosaukumu jaunā direktorijā
dir.create("6uzd_rastri")

for (i in names(rastri3uzd)) {
  writeRaster(rastri3uzd[[i]], filename = paste0(i, ".tif"), overwrite = TRUE)
}
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

``` r
parbaude <- rast("211.tif")
plot(parbaude) # ir ok
```

![](Uzd06_Rubene_files/figure-gfm/unnamed-chunk-17-1.png)<!-- -->

``` r
rm(meza_klases, MVRdati_filtered, MVRrastrs1, parbaude, rastri2uzd, ref10m, ref10m_rast, cirte101, cirte41, cirte71, cirte81, i, klase5, klase10, klase20)
```

Tomēr uztaisīšu arī kopīgo rastru (visas meža klases vienā), manam
datoram ar merge savienot 21 rastru bija drusku par grūtu, bet rezultāts
no tā nemainās.

``` r
meza_rastrs <- rasterize(MVRdati,ref10m_cropped, field = "klase")
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
plot(meza_rastrs)
```

![](Uzd06_Rubene_files/figure-gfm/unnamed-chunk-18-1.png)<!-- -->

``` r
writeRaster(meza_rastrs, filename = "mezi_rastrs_kopa_10m.tif", overwrite = TRUE)
```

    ## |---------|---------|---------|---------|=========================================                                          

# 4) Rastru savienošana

Savienojiet pirmajā un trešajā punktos izveidotos slāņus, tā, lai mazāka
skaitliskā vērtība nozīmē augstāku prioritāti/svaru/dominanci rastra
šūnas vērtības piešķiršanai.

Reāli šajā gadījumā tas būtiski ir tikai gadījumos, kad pārklāsies meži
ar laukiem, bet pieturēšos pie pieejas par zemākās vērtības dominanci.

Manam datoram bija pārāk smagi ar merge savienot 21 rastru (mēģināju
divreiz), šeit atstāju komandrindas, kuras mēģināju:

``` r
# rastri3uzd_seciba <- rastri3uzd[order(names(rastri3uzd))] # sakārtoju augošā secībā
# rastri3uzd_seciba <- sprc(rastri3uzd_seciba) # pārtaisu par SpatRasterCollection, lai vieglāk strādāt
# LAD10m <- lapply(LAD10m, function(r) extend(r, ref10m_cropped)) #lai visam sakrīt extent, jo kaut kas drusku nesakrita

# funkcija merge ņem vērā secību, ja sakārtoju augošā secībā, tad noteiktā prioritāte tiek ievērota
# uzd4_rastrs <- merge(LAD10m,rastri3uzd_seciba)
```

Tāpēc iepriekšminēto iemeslu dēļ vienkārši savienošu kopīgo meža rastru
(ar visām klasēm) ar LAD rastru, pieņemot LAD datus par prioritāriem
(jeb secībā norādot tos pirmos):

``` r
ext(LAD10m)
```

    ## SpatExtent : 401770, 597090, 232910, 363630 (xmin, xmax, ymin, ymax)

``` r
ext(meza_rastrs) # nesakritības, tāpēc to labošu
```

    ## SpatExtent : 405230, 594510, 234180, 363290 (xmin, xmax, ymin, ymax)

``` r
ext(ref10m_cropped)
```

    ## SpatExtent : 405230, 594510, 234180, 363290 (xmin, xmax, ymin, ymax)

``` r
LAD10m <- crop(LAD10m, meza_rastrs) # LAD datus vajag tikai meža datu teritorijā
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
LAD10m <- extend(LAD10m, meza_rastrs) # palielinu iztrūkstošās malas 
uzd4_rastrs <- merge(LAD10m,meza_rastrs)
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
plot(uzd4_rastrs)
```

![](Uzd06_Rubene_files/figure-gfm/unnamed-chunk-20-1.png)<!-- -->

``` r
writeRaster(uzd4_rastrs, filename = "4uzd.tif", overwrite = TRUE)
```

    ## |---------|---------|---------|---------|=========================================                                          

Redzams, kur beidzas mežu dati un ka lauki atrodas arī ārpus centra
virsmežniecības teritorijas (jo apgriezts tika, balstoties uz extent kā
kastīti, nevis precīzām robežām), bet tas šoreiz nav tik svarīgi.

# 5) Cik šūnās ir gan mežaudžu, gan lauku informācija?

Šo var pārbaudīt dažados veidos. Izmantošu ifel, kas atgriezīs rastru ar
vērtībām “1”, ja konkrētajā pikselī abos rastra slāņos vērtības nav NA

``` r
parklajas <- ifel(!is.na(LAD10m) & !is.na(meza_rastrs), 1, NA)
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

``` r
plot(parklajas)
```

![](Uzd06_Rubene_files/figure-gfm/unnamed-chunk-21-1.png)<!-- -->

``` r
vertibas <- freq(parklajas) # frekvenču aprēķins
str(vertibas) 
```

    ## 'data.frame':    1 obs. of  3 variables:
    ##  $ layer: num 1
    ##  $ value: num 1
    ##  $ count: num 1765

Tātad meža un lauku dati pārklājas 1765 pikseļos.

# 6) Cik šūnas atrodas Latvijas sauszemes teritorijā, bet nav raksturotas šī uzdevuma iepriekšējos punktos?

Atbildēšu uz šo jautājumu, balstoties tikai uz analīzes teritoriju.
Vispirms savienošu 4. uzdevumā radīto rastru ar references rastru, lai
aizpildītu šūnas, kuras ietilpst Latvijas teritorijā. Tas nozīmē, ka
visi pikseļi, kuros nav informācijas ne no LAD datiem, ne no meža
datiem, būs aizpildīti ar vērtību “1”

``` r
uzd6rastrs <- merge(uzd4_rastrs,ref10m_cropped)
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
plot(uzd6rastrs)
```

![](Uzd06_Rubene_files/figure-gfm/unnamed-chunk-22-1.png)<!-- -->

``` r
# un cik ir pikseļu ar vērtību "1"?
vieninieki <- global(uzd6rastrs == 1, "sum", na.rm=TRUE)
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
print(vieninieki)
```

    ##             sum
    ## layer 138463687

Tātad šajā teritorijā tie ir 138463687 pikseļi.
