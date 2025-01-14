Trešais uzdevums: rastra dati, rasterizēšana un kodējumi
================
Jekaterīna Butkeviča,
2025. gads 09. janvāris

## Datu sagatavošana

Atsevišķos mainīgos saglabāsim saites uz projekta Zenodo repozitorija
rakstiem par vektoru un rastra references slāņiem.

``` r
url_rastrs_zenodo <- "https://zenodo.org/records/14497070"
url_vektors_zenodo <- "https://zenodo.org/records/14277114"
```

Iegūsim HTML saturu no katras lapas.

``` r
#Ja pakotne nav instsalēta: install.packages("httr") 
library(httr)

#rastru datiem
html_rastrs_saturs <- GET(url_rastrs_zenodo)
html_rastrs_teksts <- content(html_rastrs_saturs, "text")

#vektoru datiem
html_vektors_saturs <- GET(url_vektors_zenodo)
html_vektors_teksts <- content(html_vektors_saturs, "text")
```

No iegūtas informācijas izvilksim failu lejupielādes saites.

``` r
#Ja pakotne nav instsalēta: install.packages("stringr")
library(stringr)

##rastru datiem
saites_rastrs <- str_extract_all(html_rastrs_teksts, "https://zenodo.org/records/14497070/files/[^\\\"\\s]+")[[1]]

##vektoru datiem
saites_vektors <- str_extract_all(html_vektors_teksts, "https://zenodo.org/records/14277114/files/[^\\\"\\s]+")[[1]]
```

Izveidosim lokālo direktoriju.

``` r
#rastrs
dir_rastrs <- "C:/Users/user/Desktop/HiQBioDiv_macibas/3uzd/references_zenodo/rastrs"
dir.create(dir_rastrs, recursive = TRUE)

#vektors
dir_vektors <- "C:/Users/user/Desktop/HiQBioDiv_macibas/3uzd/references_zenodo/vektors"
dir.create(dir_vektors, recursive = TRUE)
```

Lejupielādesim failus.

``` r
#rastrs
for (katra in saites_rastrs) {
  faila_nosaukums <- basename(katra)
  download.file(katra, file.path(dir_rastrs, faila_nosaukums), mode = "wb")
  }


#vektors
for (katra in saites_vektors) {
  faila_nosaukums <- basename(katra)
  download.file(katra, file.path(dir_vektors, faila_nosaukums), mode = "wb")
  }
```

Nodzesim mainīgos, kas vairs nav nepieciešami.

``` r
rm(faila_nosaukums, katra, html_rastrs_saturs, html_vektors_saturs, 
   html_rastrs_teksts, html_vektors_teksts, url_rastrs_zenodo, 
   url_vektors_zenodo, saites_rastrs, saites_vektors, dir_vektors, dir_rastrs)
```

# Pirmais uzdevums

Lejupielādējiet Lauku atbalsta dienesta datus par teritoriju, kuru
aptver otrā uzdevuma Mežu valsts reģistra dati, no WFS, izmantojot šo
saiti
(<https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer>).

## Centra virsmežniecibas uzraugāmo teritorijas ģeometrijas sagatavošana

Caur norādīto saiti var iegūt informāciju, kas aptver visu Latvijas
teritoriju. Lai atlasītu datus tikai no teritorijas, kuru aptver 2.
uzdevumā izmantotie Mežu valsts reģistra dati, vispirms sagatavosim
slāni, kas atspoguļos vajadzīgo – Centra virsmežniecības uzraugāmo
teritoriju, ko vēlāk izmantosim Lauku atbalsta dienesta datu
filtrēšanai.

### Datu ielasīšana

*Šī apakšsadaļa izveidota, izmantojot kodu no iepriekšējā uzdevuma.*

Definēsim direktoriju, kur glabājas *.parquet* faili no otrā uzdevuma.
Ierakstīsim visus absolūtos ceļus uz *.parquet* failiem vienā vektorā.

``` r
parquet_dir_cels <- "C:/Users/user/Desktop/HiQBioDiv_macibas/2uzd/centra_virsmezn_dati/parquet"

parquet_visi <- list.files(parquet_dir_cels, pattern = "\\.parquet$", full.names = TRUE)
```

Nolasīsim un apvienosim visus *.parquet* failus kopā.

``` r
#Ja pakotne nav instsalēta: install.packages("dplyr")
#Ja pakotne nav instsalēta: install.packages("sfarrow")
library(dplyr)
library(sfarrow)

apvienotais_parquet <- lapply(parquet_visi, st_read_parquet) %>%
  bind_rows()

rm(parquet_dir_cels, parquet_visi) #Izdzēst mainīgos, kas vairs nav nepieciešami.
```

### Datu tirīšana

*Šī apakšsadaļa izveidota, izmantojot kodu no iepriekšējā uzdevuma.*

Pārveidosim visas ģeometrijas uz MULTIPOLYGON formātu.

``` r
#Ja pakotne nav instsalēta: install.packages("sf")
library(sf)
apvienotais_parquet <- apvienotais_parquet %>%
  mutate(geometry = st_cast(geometry, "MULTIPOLYGON"))
```

Pārbaudīsim, vai starp ģeometrijām ir nederīgas, un rezultātus izvadīsim
konsolē. Ja būs atrastas nederīgas ģeometrijas, labosim tās. Veiksim
pārbaudi vēlreiz. Rezultātam jābūt ‘0’.

``` r
# Pārbaudām, vai ir nederīgās ģeometrijas
invalid_geometrijas <- apvienotais_parquet %>% 
  filter(!st_is_valid(geometry))

# Izvadām rezultātu
print(nrow(invalid_geometrijas))
```

    ## [1] 274

``` r
# Ja rezultats > 0, tad labojam
if (nrow(invalid_geometrijas) > 0) {
  apvienotais_parquet <- st_make_valid(apvienotais_parquet)
}
```

Pārbaudīsim, vai ir tukšas ģeometrijas (rezultāts konsolē). Atbildē
saņemsim loģisko vektoru, pēc kura vajadzības gadījumā (ja rezultāts
konsolē \> 0) filtrējam datus (! = otrādi).

``` r
#Pārbaudam vai ir tukšas geometrijas
logika_tuksas_geometrijas <- st_is_empty(apvienotais_parquet)
print(sum(logika_tuksas_geometrijas))
```

    ## [1] 6

``` r
# Filtrējam ārā nederīgās ģeometrijas
apvienotais_parquet <- apvienotais_parquet[!logika_tuksas_geometrijas,]


#Izdzēst mainīgos, kas vairs nav nepieciešami.
rm(invalid_geometrijas, logika_tuksas_geometrijas)
```

### Centra virsmežniecibas uzraugāmo teritorijas poligona veidošana

Apvienosim visas ģeometrijas vienā poligonā.

``` r
apvienoti_mezi <- st_union(apvienotais_parquet)

rm(apvienotais_parquet)
```

Izmantojot [Concave
Hull](https://www.researchgate.net/publication/312373158_A_Concave_Hull_Based_Algorithm_for_Object_Shape_Reconstruction/figures?lo=1)
metodi, aprēķināsim ierobežojošo daudzstūri, kas ietvers visus meža
nogabalus.

``` r
#Ja pakotne nav instsalēta: install.packages("concaveman")
library(concaveman)

Centra_virsmeznieciba <- concaveman(st_coordinates(apvienoti_mezi))
```

Izveidosim daudzstūri no iegūtās koordinātu kopas.

``` r
Centra_virsmeznieciba <- st_polygon(list(Centra_virsmeznieciba[, 1:2]))
```

Lai turpinātu darbu, jāparveido mainīgais uz *sf* objektu.

``` r
Centra_virsmeznieciba <- st_sfc(Centra_virsmeznieciba)
```

Vizualizēsim, lai apskatītu rezultātu:

``` r
plot(st_geometry(Centra_virsmeznieciba), main = "Centra virsmežniecibas \nuzraugāma teritorija")
```

![](Uzd03_Butkevica_files/figure-gfm/vizualizet%20teritorijas%20poligonu-1.png)<!-- -->

Uz šo brīdi ģeometrija nav piesaistīta koordinātu sistēmai. Lai
turpinātu darbu, atgriežam sākotnējo koordinātu sistēmu.

``` r
#pievienot koordināšu sistēmu atpakaļ 
st_crs(Centra_virsmeznieciba) <- st_crs(apvienoti_mezi)

rm(apvienoti_mezi)#Izdzēst mainīgos, kas vairs nav nepieciešami.
```

MVR dati izmanto *LKS-92*, bet WFS serveris uztur datus *WGS 84*
koordinātu sistēmā. Pielāgosim to mūsu datiem.

``` r
Centra_virsmeznieciba <- st_transform(Centra_virsmeznieciba, crs = 4326)
```

## Lauku atbalsta dienesta datu iegūšana

Cik man izdevās saprast, šis WFS serveris neļauj pieprasīt sarežģīto
ģeometriju. Lai vienkāršotu vaicājumu, izveidosim taisnstūri apkārt mūsu
MVR datiem.

``` r
bbox_Centra <- st_bbox(Centra_virsmeznieciba)
```

Pieprasīsim no servera tikai datus, kas atrodas šī taisnstūra iekšpusē.

``` r
#Ja pakotne nav instsalēta: install.packages("ows4R")
library(ows4R)

wfs_LAD <- "https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer"
LAD_client <- WFSClient$new(wfs_LAD, serviceVersion = "2.0.0")
LAD_client$getFeatureTypes(pretty = TRUE)
url_LAD <- parse_url(wfs_LAD)
url_LAD$query <- list(
  service = "wfs",
  request = "GetFeature",
  typename = "Lauki",
  srsName = "EPSG:4326",
  bbox_Centra = paste(bbox_Centra, collapse = ",")
)
request <- build_url(url_LAD)

LAD_dati <- st_read(request)

#Izdzēst mainīgos, kas vairs nav nepieciešami.
rm(url_LAD, LAD_client, bbox_Centra, request)
```

## Lauku atbalsta dienesta datu filtrēšana

Dati no Lauku atbalsta dienesta ir iegūti, un tagad atgriežamies pie
Latvijas ģeodēziskās koordinātu sistēmas (LKS-92).

``` r
LAD_dati <- st_transform(LAD_dati, crs = 3059 )
Centra_virsmeznieciba <- st_transform(Centra_virsmeznieciba, crs = 3059 )
```

Atstāsim tikai tos LAD datus, kas ietilpst Centra virsmežniecības
uzraugāmajā teritorijā. Šie dati būs ievades vektors otrajā uzdevumā.

``` r
LAD_centra <- st_intersection(LAD_dati, Centra_virsmeznieciba)
```

Vizualizēsim, lai pārbaudītu rezultātus:

``` r
plot(LAD_centra["gml_id"], main = "Filtrētie pļavu nogabali")
```

![](Uzd03_Butkevica_files/figure-gfm/vizualizet%20plavas-1.png)<!-- -->

``` r
#Izdzēst mainīgos, kas vairs nav nepieciešami.
rm(Centra_virsmeznieciba, LAD_dati)
```

# Otrais uzdevums

Atbilstoši referencei (10 metru izšķirtspējā), rasterizējiet iepriekšējā
punktā lejupielādētos vektordatus, sagatavojot GeoTIFF slāni ar
informāciju par vietām, kurās ir lauku bloki (kodētas ar 1) vai tās
atrodas Latvijā, bet tajās nav lauku bloki vai par tiem nav informācijas
(kodētas ar 0). Vietām ārpus Latvijas saglabājiet šūnas bez vērtībām.

Saglabāsim ceļu uz failu atsevišķā mainīgajā. Ielādēsim references
rastru (10 m izšķirtspējā). Pārbaudīsim, kurai koordinātu sistēmai
piesaistīts rastrs.

``` r
#Ja pakotne nav instsalēta: install.packages("raster")
library(raster)

cels_ref_rastrs_10m <- "C:/Users/user/Desktop/HiQBioDiv_macibas/3uzd/references_zenodo/rastrs/LV10m_10km.tif"
ref_rastrs_10m <- raster(cels_ref_rastrs_10m)

st_crs(ref_rastrs_10m) #koordinātes
```

Rastrs bija saistīts ar LKS-92 koordinātu sistēmu, kas atbilst arī
vektorslāņa koordinātu sistēmai. Veiksim vektorslāņa rastrizēšanu pēc
references ar 10 metru izšķirtspēju. Ar masku nodrošināsim, lai šūnām
ārpus Latvijas teritorijas būtu vērtība *NA*.

``` r
#Ja pakotne nav instsalēta: install.packages("fasterize")
library(fasterize)

LAD_centra_rastrs_10m <- fasterize(LAD_centra, ref_rastrs_10m, background = 0)
LAD_centra_rastrs_10m_maskets <- mask(LAD_centra_rastrs_10m, ref_rastrs_10m)
```

Drošs paliek drošs. Saglabāsim starp rezultātu.

``` r
writeRaster(LAD_centra_rastrs_10m_maskets, "C:/Users/user/Desktop/HiQBioDiv_macibas/3uzd/LAD_centra_10m.tif", format = "GTiff", overwrite = TRUE)

#Izdzēst mainīgos, kas vairs nav nepieciešami.
rm(ref_rastrs_10m, LAD_centra_rastrs_10m, LAD_centra, cels_ref_rastrs_10m)
```

# Trešais uzdevums

Izmēģiniet {terra} funkcijas resample(), aggregate() un project(), lai
no iepirkšējā punktā sagatavotā rastra izveidotu jaunu: - ar 100m
pikseļa malas garumu un atbilstību references slānim; - informāciju par
platības, kurā ir lauka bloki, īpatsvaru.

Kura funkcija vai to kombinācija piedāvā ātrāko risinājumu?

Saglabāsim ceļu uz failu atsevišķā mainīgajā. Ielādēsim references
rastru (100 metru izšķirtspējā). Pārbaudīsim, kurai koordinātu sistēmai
ir piesaistīts rastrs.

``` r
cels_ref_rastrs_100m <- "C:/Users/user/Desktop/HiQBioDiv_macibas/3uzd/references_zenodo/rastrs/LV100m_10km.tif"
ref_rastrs_100m <- raster(cels_ref_rastrs_100m)
st_crs(ref_rastrs_100m) #LKS-92
```

Pārveidosim rastra izšķirtspēju līdz 100 metru šūnām, grupējot mazākas
šūnas lielākās un aprēķinot summu.

``` r
#Ja pakotne nav instsalēta: install.packages("terra")
library(terra)

LAD_centra_rastrs_100m <- aggregate(LAD_centra_rastrs_10m_maskets, fact = 10, fun = sum)
```

Saskaņosim iegūto 100 metru rastru ar referenci.

``` r
LAD_centra_rastrs_100m_valid <- resample(LAD_centra_rastrs_100m, ref_rastrs_100m, method = "bilinear")
```

Aprēķināsim lauku īpatsvaru katrā 100 metru šūnā un saglabāsim
rezultātu.

``` r
LAD_centra_rastrs_100m_plavu_ipatsvars <- LAD_centra_rastrs_100m_valid / 100

writeRaster(LAD_centra_rastrs_100m_plavu_ipatsvars, "C:/Users/user/Desktop/HiQBioDiv_macibas/3uzd/LAD_centra_100m_plavu_ipatsvars.tif", format = "GTiff")

#Izdzēst mainīgos, kas vairs nav nepieciešami.
rm(ref_rastrs_100m, LAD_centra_rastrs_100m_plavu_ipatsvars, LAD_centra_rastrs_100m, cels_ref_rastrs_100m)
```

Diemžēl, es nevaru atbildēt uz jautājumu “Kura funkcija vai to
kombinācija piedāvā ātrāko risinājumu?”, jo es neatradu alternatīvu
risinājumu.

# Ceturtais uzdevums

Izmantojot iepriekšējā punktā radīto 100m šūnas izmēra slāni,
sagatavojiet divus jaunus, tā, lai: 1. slānis satur informāciju par
lauka bloku platību noapaļotu līdz procentam un izteiktu kā veselu
skaitli, tāpat kā iepriekš saglabājot šūnas vietām ārpus Latvijas; 1.
slānis satur bināri kodētas vērtības: Latvijā esošā šūnā ir lauku bloki
(vērtība 1) vai to nav (vērtība 0), tāpat kā iepriekš saglabājot šūnas
vietām ārpus Latvijas.

Salīdziniet izveidotos slāņus, lai raksturotu: - kā mainās faila izmērs
uz cietā diska, atkarībā no rastra šūnas izmēra; - kā mainās faila
izmērs uz cietā diska, atkarībā no tajā ietvertās informācijas un tās
kodējuma. Kāds kodējums (joslas tips) aizņem vismazāko apjomu uz cietā
diska, saglabājot informāciju?

## Procentuālais rastrs

Rastrs, kas raksturo, cik procentus no šūnas platības sastāda pļava, jau
bija izveidots, veicot rastrizēšanu. Tad vienkārši saglabāsim to
atsevišķā failā.

``` r
writeRaster(LAD_centra_rastrs_100m_valid, "C:/Users/user/Desktop/HiQBioDiv_macibas/3uzd/LAD_centra_100m_plavu_procents.tif", format = "GTiff")
```

## Binārais rastrs

Lai izmantotu {ifelse()} funkcijas analogu no pakotnes {terra}, mēs
mainīsim rastra formātu uz tādu, ko atbalsta {terra}. Pēc funkcijas
izpildīšanas atgriezīsim iepriekšējo formātu un saglabāsim rezultātu.“

``` r
LAD_centra_rastrs_100m_valid_terra <- rast(LAD_centra_rastrs_100m_valid)

LAD_centra_rastrs_100m_binars <- ifel(LAD_centra_rastrs_100m_valid_terra > 0, 1, 0)

LAD_centra_rastrs_100m_binars <- raster(LAD_centra_rastrs_100m_binars)

writeRaster(LAD_centra_rastrs_100m_binars, "C:/Users/user/Desktop/HiQBioDiv_macibas/3uzd/LAD_centra_100m_plavu_binars.tif", format = "GTiff")
```

## Salīdzinājums pēc diskā aizņemtās vietas

Salīdzinājumam izveidosim jaunu apakšdirektoriju.

``` r
dir_salidzinajums <- "C:/Users/user/Desktop/HiQBioDiv_macibas/3uzd/salidzinajums"
dir.create(dir_salidzinajums, recursive = TRUE)
```

Pārsaglabāsim tajā 10 un 100 metru rastrus, kā arī saglabāsim 100 metru
rastrus ar dažādiem joslas tipiem.

``` r
# Dažāds šunu izmērs
writeRaster(LAD_centra_rastrs_10m_maskets, "C:/Users/user/Desktop/HiQBioDiv_macibas/3uzd/salidzinajums/rasts_10m_FLT4S.tif", datatype = "FLT4S", overwrite = TRUE) # 10m rastrs, 32 bitu peldošais punkts
writeRaster(LAD_centra_rastrs_100m_valid, "C:/Users/user/Desktop/HiQBioDiv_macibas/3uzd/salidzinajums/rasts_100m_FLT4S.tif", datatype = "FLT4S", overwrite = TRUE) # 100m rastrs, 32 bitu peldošais punkts

# Dažādi formāti
writeRaster(LAD_centra_rastrs_100m_valid, "C:/Users/user/Desktop/HiQBioDiv_macibas/3uzd/salidzinajums/rasts_100m_BYTE.tif", datatype = "INT1U", overwrite = TRUE) # 8 bitu (0-255)
writeRaster(LAD_centra_rastrs_100m_valid, "C:/Users/user/Desktop/HiQBioDiv_macibas/3uzd/salidzinajums/rasts_100m_INT2U.tif", datatype = "INT2U", overwrite = TRUE) # 16 bitu (0-65535)
```

Izveidosim datu rāmi, kas satur informāciju par interesējošajiem
failiem. Pārvērsīsim baitus megabaitos un iegūsim pašu failu nosaukumus
(nevis absolūtos ceļus).

``` r
failu_info <- file.info(list.files(dir_salidzinajums, full.names = TRUE))
failu_info$izmers_MB <- failu_info$size / (1024 * 1024) # Baiti uz megabaitiem
failu_info$nosaukums <- basename(rownames(failu_info)) #iegūst faila nosaukumuno pilnā ceļa
```

Izvadīsim rezultātus konsolē:

``` r
cat(sprintf("Fails: %s\nIzmērs: %.2f MB\n\n", 
            failu_info$nosaukums, failu_info$izmers_MB))
```

    ## Fails: rasts_100m_BYTE.tif
    ## Izmērs: 0.45 MB
    ## 
    ##  Fails: rasts_100m_FLT4S.tif
    ## Izmērs: 1.30 MB
    ## 
    ##  Fails: rasts_100m_INT2U.tif
    ## Izmērs: 0.62 MB
    ## 
    ##  Fails: rasts_10m_FLT4S.tif
    ## Izmērs: 49.25 MB

*Atbilde*: Samazinoties šūnu izmēriem, izteikti palielinās aizņemtās
vietas apjoms. Vismazāk vietas diskā aizņem fails ar kodējumu **INT1U**,
taču tas nevar glabāt sevī daļskaitļus, kas nav piemēroti, ja
nepieciešams, piemēram, reģistrēt vērtības diapazonā no 0 līdz 1.

``` r
rm(LAD_centra_rastrs_100m_valid_terra, failu_info, LAD_centra_rastrs_100m_binars, 
   LAD_centra_rastrs_100m_valid,LAD_centra_rastrs_10m_maskets,dir_salidzinajums)
```
