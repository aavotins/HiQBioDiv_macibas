5uzd_Rubene
================
Betija Rubene
2025-01-27

## Piektais uzdevums: procesu dalīšana un rezultātu apvienošana

Darbam nepieciešamās pakotnes

``` r
library(terra)
library(tidyverse)
library(sf)
library(arrow)
library(sfarrow)
library(leaflet)
library(stringr)
```

# Sagatavošanās

Ielasu karti, no kuras jāizvēlas četras blakus esošas karšu lapas un
meža datus

``` r
karte <- st_read_parquet("../ref_vektori/tks93_50km.parquet")
mezi <- st_read_parquet("../centra_mezi/centra_mezi_labots.parquet")
```

Tā kā vēlāk šajā uzdevumā būs interesējoši skatīties uz lietām, kas
notiek gar malām, būtu vērtīgi iemācīties izmantot pakotni leaflet. To
arī varu izmantot lapu izvēlei. Vienīgi šīs kartes nav iespējams
ievietot github dokumentā, tāpēc komandrindas ievietotas komentāros.

Nav svarīgi, kā izvēlēties lapas - paņemšu lapas kaut kur pa vidu, jo
strādājam ar centra virsmežniecības datiem

``` r
crs(karte)
```

    ## [1] "PROJCRS[\"LKS92 / Latvia TM\",\n    BASEGEOGCRS[\"LKS92\",\n        DATUM[\"Latvia 1992\",\n            ELLIPSOID[\"GRS 1980\",6378137,298.257222101,\n                LENGTHUNIT[\"metre\",1]]],\n        PRIMEM[\"Greenwich\",0,\n            ANGLEUNIT[\"degree\",0.0174532925199433]],\n        ID[\"EPSG\",4661]],\n    CONVERSION[\"Latvian Transverse Mercator\",\n        METHOD[\"Transverse Mercator\",\n            ID[\"EPSG\",9807]],\n        PARAMETER[\"Latitude of natural origin\",0,\n            ANGLEUNIT[\"degree\",0.0174532925199433],\n            ID[\"EPSG\",8801]],\n        PARAMETER[\"Longitude of natural origin\",24,\n            ANGLEUNIT[\"degree\",0.0174532925199433],\n            ID[\"EPSG\",8802]],\n        PARAMETER[\"Scale factor at natural origin\",0.9996,\n            SCALEUNIT[\"unity\",1],\n            ID[\"EPSG\",8805]],\n        PARAMETER[\"False easting\",500000,\n            LENGTHUNIT[\"metre\",1],\n            ID[\"EPSG\",8806]],\n        PARAMETER[\"False northing\",-6000000,\n            LENGTHUNIT[\"metre\",1],\n            ID[\"EPSG\",8807]]],\n    CS[Cartesian,2],\n        AXIS[\"northing (X)\",north,\n            ORDER[1],\n            LENGTHUNIT[\"metre\",1]],\n        AXIS[\"easting (Y)\",east,\n            ORDER[2],\n            LENGTHUNIT[\"metre\",1]],\n    USAGE[\n        SCOPE[\"Engineering survey, topographic mapping.\"],\n        AREA[\"Latvia - onshore and offshore.\"],\n        BBOX[55.67,19.06,58.09,28.24]],\n    ID[\"EPSG\",3059]]"

``` r
kartelatlon <- st_transform(karte, crs = 4326) # leaflet pieprasa WGS84

# leaflet(kartelatlon) %>%
#  addTiles() %>%  
#  addPolygons(
#    popup = ~paste("Attribute:", NUMURS) # gribu redzēt numuru uzklikšķinot
#  )
```

Izvēlējos lapas 3333, 3334, 3331 un 3332, ievietoju tās atsevišķā
objektā

``` r
lapas <- karte %>% filter(NUMURS %in% c(3333,3334,3331,3332))
lapas_list <- split(lapas, lapas$NUMURS)
```

Lapas saglabāšu un tad izveidošu path sarakstu uz lapu failiem. Tādējādi
būs iespējams nodrošināt failu ielasīšanu pašā funkcijā un pareizos
failu nosaukumus izvades failā

``` r
for (nr in names(lapas_list)) {
  data <- lapas_list[[nr]]  
  file_name <- paste0("lapa", nr, ".parquet") 
  st_write_parquet(data, file_name) 
}

lapas_list_paths <- list.files(pattern = "^lapa\\d{4}\\.parquet$", full.names = TRUE)
```

``` r
# Noņemu lieko
rm(data, karte, kartelatlon, file_name, nr)
```

Iepriekšējā uzdevumā izveidotajā funkcijā bija nepieciešams veikt dažas
izmaiņas. Ja pareizi esmu sapratusi uzdevuma būtību, tad references
rastrus vajadzīgs apgriezt nevis ar lapu, bet ar atlasīto meža datu
extent (crop to tā dara pēc nokusējuma) Tāds domu gājiens, jo uzdevuma
premisē bija runa par rastru pārklāšanos - ja rastru apgriezīšu uzreiz
ar pašu lapu, tad var tikt daļēji pazaudēta informācija par uz lapas
robežām esošajiem poligoniem, ja tādi būs. Tas nozīmē, ka būs
nepieciešams saglabāt arī izveidotos meža vektorfailus katrā uzdevumā.

``` r
dir.create("5uzd_faili") # mape visiem failiem lai mazinātu haosu

mana_funkcija <- function(mezs_dati, koku_suga, uzd_nr) {
numurs <- str_extract(mezs_dati, "(?<=lapa)\\d+")
mezs_dati <- st_read_parquet(mezs_dati)
ref10m <- rast(("../ref_rastri/LV10m_10km.tif"))
ref10m <- crop(ref10m, mezs_dati)
ref100m <- rast(("../ref_rastri/LV100m_10km.tif"))
ref100m <- crop(ref100m, mezs_dati)
faila_nos <- paste0("lapa", numurs,"_",uzd_nr, "_priedes100m")
fails <- file.path("5uzd_faili", paste0(faila_nos, ".tif"))
priezu_mezi <- mezs_dati %>% filter(s10 == koku_suga)
priezu_mezi_10m <- rasterize(priezu_mezi,ref10m, background = 0)
priezu_mezi_10m[is.na(ref10m)] <- NA
priezu_mezi_100m <-
  terra::resample(priezu_mezi_10m,ref100m,method="average",
                  filename = fails, overwrite=TRUE)
}
```

# 1) Spatial join

1.1. solis: izmantojot spatial join, saistiet MVR datus ar izvēlētajām
karšu lapām (šis ir sākums laika mērogošanai). Kāds ir objektu skaits
pirms un pēc savienošanas? Apskatiet katrai kartes lapai piederīgos
objektus - vai tie atrodas tikai kartes lapas iekšienē, vai ir iekļauti
visi objekti, kas ir uz kartes lapas robežām?

Izmantoju spatial join.

``` r
for (i in lapas_list_paths) {
  lapa <- st_read_parquet(i)
  mezi <-st_read_parquet("../centra_mezi/centra_mezi_labots.parquet")
  mezi <- st_transform(mezi, st_crs(lapa)) # crs it kā ir ir vienādas, bet cikls bez šī tāpat meta error par nesakritībām...
  fails <- st_join(mezi, lapa, left = FALSE)
  basename <- tools::file_path_sans_ext(basename(i))
  nosaukums <- file.path("5uzd_faili", paste0(basename,"mezi_1uzd.parquet"))
  st_write_parquet(fails, nosaukums)
}
```

    ## Warning: This is an initial implementation of Parquet/Feather file support and
    ## geo metadata. This is tracking version 0.1.0 of the metadata
    ## (https://github.com/geopandas/geo-arrow-spec). This metadata
    ## specification may change and does not yet make stability promises.  We
    ## do not yet recommend using this in a production setting unless you are
    ## able to rewrite your Parquet/Feather files.
    ## Warning: This is an initial implementation of Parquet/Feather file support and
    ## geo metadata. This is tracking version 0.1.0 of the metadata
    ## (https://github.com/geopandas/geo-arrow-spec). This metadata
    ## specification may change and does not yet make stability promises.  We
    ## do not yet recommend using this in a production setting unless you are
    ## able to rewrite your Parquet/Feather files.
    ## Warning: This is an initial implementation of Parquet/Feather file support and
    ## geo metadata. This is tracking version 0.1.0 of the metadata
    ## (https://github.com/geopandas/geo-arrow-spec). This metadata
    ## specification may change and does not yet make stability promises.  We
    ## do not yet recommend using this in a production setting unless you are
    ## able to rewrite your Parquet/Feather files.
    ## Warning: This is an initial implementation of Parquet/Feather file support and
    ## geo metadata. This is tracking version 0.1.0 of the metadata
    ## (https://github.com/geopandas/geo-arrow-spec). This metadata
    ## specification may change and does not yet make stability promises.  We
    ## do not yet recommend using this in a production setting unless you are
    ## able to rewrite your Parquet/Feather files.

``` r
# vienkāršā veidā paskatīšos vienu karti, vai izskatās ok
parbaude <- st_read_parquet("5uzd_faili/lapa3331mezi_1uzd.parquet")
plot(st_geometry(parbaude))
```

![](5uzd_Rubene_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

``` r
rm(parbaude,basename,i,nosaukums, fails, lapa, lapas)

# saglabāto failu saraksts, ko vēlāk pielietot ciklā
uzd1_mezi_file_paths <- list.files("5uzd_faili", pattern = "^lapa\\d{4}mezi_1uzd\\.parquet$", full.names = TRUE)

# cik objektu katrā lapā?
uzd1_mezi <- map(uzd1_mezi_file_paths, st_read_parquet)
```

Pirms savienošanas objektā mezi ir 505654 objekti, katrā lapā piederīgi
ir 15729, 29042, 33449 un 32904 objekti.

Paskatīšu, kas notiek kartes lapas malās ar leaflet interaktīvo karti
pirmajai kartes lapai:

``` r
lapa3331 <- st_transform(lapas_list[["3331"]], crs = 4326)
#bet vēlāk noderēs arī pārējās
lapa3332 <- st_transform(lapas_list[["3332"]], crs = 4326)
lapa3333 <- st_transform(lapas_list[["3333"]], crs = 4326)
lapa3334 <- st_transform(lapas_list[["3334"]], crs = 4326)

lapa3331uzd1 <- st_read_parquet("5uzd_faili/lapa3331mezi_1uzd.parquet")
lapa3331uzd1 <- st_transform(lapa3331uzd1, crs = 4326)

#leaflet() %>%
#  addTiles() %>%
#  addPolygons(data = lapa3331, color = "blue") %>%
#  addPolygons(data = lapa3331uzd1, color = "green") 
```

Kartes lapai piederīgie objekti, kā jau bija sagaidāms no funkcijas
st_join, ir visi, kuri kaut vai daļēji ietilpst kartes lapā. Tātad
iekļauti arī tie objekti, kas atrodas uz lapas malām.

``` r
rm(lapa3331uzd1, uzd1_mezi)
```

1.2. solis - iteratīvā ciklā izmantojiet pārskatīto funkciju no uzdevuma
sākuma, lai saglabātu katras karšu lapas rezultātu savā GeoTIFF failā,
kura nosaukums ir saistāms ar šo apakšuzdevumu un individuālo lapu. Kā
izskatās šūnu aizpildījums pie karšu lapu malām? Vai ir saglabājušies
objekti ārpus kartes lapām?

Pielietoju funkciju:

``` r
for (i in seq_along(uzd1_mezi_file_paths)) {
  mana_funkcija(mezs_dati = uzd1_mezi_file_paths[i], koku_suga = 1, uzd_nr = "1uzd")
}

rezultats <- rast("5uzd_faili/lapa3331_1uzd_priedes100m.tif")
plot(rezultats) # Ir ok, tikai var redzēt, ka gar malām informācija ir pazudusi (gar malām tikai zemas vērtības)
```

![](5uzd_Rubene_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

``` r
rm(rezultats, i)
```

1.3 solis apvienot izveidotos rastrus vienā slānī

Sākumā gribu paskatīties, kā izskatās divas no lapām, jo sagaidāma
pārklāšanās

``` r
# pirmais rastrs
lapa3331_priedes_1uzd <- rast("5uzd_faili/lapa3331_1uzd_priedes100m.tif")

#otrais rastrs
lapa3332_priedes_1uzd <- rast("5uzd_faili/lapa3332_1uzd_priedes100m.tif")

#leaflet() %>%
#  addTiles() %>%
#  addRasterImage(lapa3331_priedes_1uzd, group = "rastrs3331") %>%
#  addRasterImage(lapa3332_priedes_1uzd, group = "rastrs3332") %>%
#  addPolygons(data = lapa3331, color = "blue", group = "lapas") %>%
#  addPolygons(data = lapa3332, color = "blue", group = "lapas") %>%
#  addLayersControl(
#    overlayGroups = c("rastrs3331","rastrs3332","lapas"),
#    options = layersControlOptions(collapsed = FALSE))
```

Rastru malām aiz lapu robežām ir dažādi platumi. Bet redzams, ka rastrs
ir arī ārpus lapas, kas bija sagaidāms, kā arī tas, ka vietās, kur
rastri pārklājas, abos rastros vērtības nesakrīt. Pie malām vērtības
vienmēr ir mazas, jo poligoni, kas ir bijuši tieši ārpus atlasītajiem
mežiem, nav ņemti vērā (šīs malu vērtības teorētiski daļā gadījumu
varēja ietekmēt, kā redzams vietās, kur malas pārklājas).

Platās malas laikam ir tādēļ, jo tika atlasīti arī poligoni ārpus lapas,
attiecīgi tad rastrs arī tika izgriezts tikpat tālu. Leaflet karte
pieprasa WGS84 - cik daudz un kādas kļūdas var rasties, transformējot
crs? Labprāt to apmācību laikā apspriestu. Negribēju visu laiku
darboties WGS84 koordinātu sistēmā, jo premisē minēts, ka projektā
pastāvīgi strādāsim ar LKS-92.

Izveidoju sarakstu ar ceļiem uz visiem radītajiem failiem:

``` r
uzd1_priedes_file_paths <- list.files("5uzd_faili", pattern = "^lapa\\d{4}_1uzd_priedes100m\\.tif$", full.names = TRUE)
uzd1_priedes <- lapply(uzd1_priedes_file_paths, rast)
```

Tagad rastri jāapvieno. Jādomā par to, ko darīt ar malām, kuras
pārklājas. Negribu, lai viena rastra vērtības ir svarīgākas par otrām,
ko dara funkcija merge, jo, kā redzams augstāk kartē, rastri krietni
iestiepjas arī blakus lapā. Tāpēc izmantošu terra:mosaic ar fun = max,
lai ņemtu vērā maksimālo vērtību no pārklāšanās vietām (centīšos
paskaidrot domu, bet kaut kā nesanāk veikli - ja jau vienā lapā mežu
īpatsvars tika aprēķināts, tad tas noteikti ir vismaz tik liels
konkrētajā šūnā, kas ir precīzākais, ko šobrīd varu iegūt).

``` r
uzd1_priedes_merged <- do.call(mosaic, c(uzd1_priedes, fun = max))
plot(uzd1_priedes_merged)
```

![](5uzd_Rubene_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->

Acīmredzot tajā platājā malā bijuši meža dati, kuri izfiltrēti, atlasot
tikai priežu mežus.

``` r
writeRaster(uzd1_priedes_merged, "Uzd1_priedes100m_merged.tif", overwrite = TRUE)
```

``` r
#leaflet() %>%
#  addTiles() %>%
#  addRasterImage(uzd1_priedes_merged) %>%
#  addPolygons(data = lapa3331, color = "blue", group = "lapas") %>%
#  addPolygons(data = lapa3332, color = "blue", group = "lapas") %>%
#  addPolygons(data = lapa3333, color = "blue", group = "lapas") %>%
#  addPolygons(data = lapa3334, color = "blue", group = "lapas") %>%
#  addLayersControl(
#    overlayGroups = "lapas",
#    options = layersControlOptions(collapsed = FALSE))
```

Bet tā kopumā pēc savienošanas karšu lapu robežas izteikti nav redzamas,
lai gan dažviet ir mazākas vērtības vietās, kur savstarpēji robežojas
lapu malas. Man šķiet, ka strādājot būtu vērtīgi ieviest buferi, lai
malās nekas nepazustu - jo šobrīd poligoni, kas ir bijuši tieši ārpus
atlasītajiem mežiem, nav ņemti vērā, līdz ar to pie pašas malas kaut
kāda informācija var būt zaudēta.

Lai mērītu laiku, es visas uz apakšuzdevumiem tieši attiecināmās
komandrindas sarakstīju kompakti (zemāk), vēlos atstāt visu savu domu
gaitu, pārbaudes un bildītes (augstāk)

``` r
SakumsUzd1 <- Sys.time()

# 1.1. solis
for (i in lapas_list_paths) {
  lapa <- st_read_parquet(i)
  mezi <-st_read_parquet("../centra_mezi/centra_mezi_labots.parquet")
  mezi <- st_transform(mezi, st_crs(lapa)) 
  fails <- st_join(mezi, lapa, left = FALSE)
  basename <- tools::file_path_sans_ext(basename(i))
  nosaukums <- file.path("5uzd_faili", paste0(basename,"mezi_1uzd.parquet"))
  st_write_parquet(fails, nosaukums)
}

# 1.2. solis
for (i in seq_along(uzd1_mezi_file_paths)) {
  mana_funkcija(mezs_dati = uzd1_mezi_file_paths[i], koku_suga = 1, uzd_nr = "1uzd")
}

# 1.3. solis
uzd1_priedes_file_paths <- list.files("5uzd_faili", pattern = "^lapa\\d{4}_1uzd_priedes100m\\.tif$", full.names = TRUE)
uzd1_priedes <- lapply(uzd1_priedes_file_paths, rast)

uzd1_priedes_merged <- do.call(mosaic, uzd1_priedes)
writeRaster(uzd1_priedes_merged, "Uzd1_priedes100m_merged.tif", overwrite = TRUE)

BeigasUzd1 <- Sys.time()
print(BeigasUzd1 - SakumsUzd1)
```

    ## Time difference of 1.839719 mins

Process aizņema aptuveni minūti.

``` r
rm(fails,lapa,lapa3331_priedes_1uzd, lapa3332_priedes_1uzd, uzd1_priedes, BeigasUzd1, SakumsUzd1, uzd1_mezi_file_paths, uzd1_priedes_file_paths)
```

# 2) Clipping

2.1. solis: sāciet iteratīvo ciklu, kurā, izmantojot clipping iegūstiet
MVR datus ar apstrādājamo kartes lapu (šis ir sākums laika mērogošanai).
Kāds ir objektu skaits katrā kartes lapā, kā tas saistās ar iepriekšējo
apakšuzdevumu? Ārpus cikla apskatiet katrai kartes lapai piederīgos
objektus - vai tie atrodas tikai kartes lapas iekšienē, vai ir iekļauti
visi objekti, kas ir uz kartes lapas robežām?

Tagad tas pats, kas iepriekš, jāatkārto ar clipping, izmantojot funkciju
st_intersection. Šī funkcija atgriež tikai tādas ģeometrijas, kurās x un
y objekti savstarpēji pārklājas, līdz ar to sagaidāms, ka ģeometrijas
tiks apgrieztas līdz ar lapas malām.

``` r
for (i in lapas_list_paths) {
  lapa <- st_read_parquet(i)
  mezi <-st_read_parquet("../centra_mezi/centra_mezi_labots.parquet")
  mezi <- st_transform(mezi, st_crs(lapa)) 
  fails <- st_intersection(mezi, lapa)
  basename <- tools::file_path_sans_ext(basename(i))
  nosaukums <- file.path("5uzd_faili", paste0(basename,"mezi_2uzd.parquet"))
  st_write_parquet(fails, nosaukums)
}

# saglabāto failu saraksts, ko vēlāk pielietot ciklā
uzd2_mezi_file_paths <- list.files("5uzd_faili", pattern = "^lapa\\d{4}mezi_2uzd\\.parquet$", full.names = TRUE)

uzd2_mezi <- map(uzd2_mezi_file_paths, st_read_parquet)
```

Katrā lapā piederīgi ir 15729, 29042, 33449 un 32904 objekti - tieši
tāpat, kā ar funkciju st_join. Tas bija sagaidāms, jo, pat ja funkcija
apgriež daļu objektu, to skaits paliek nemainīgs.

Paskatīšos, kas notiek gar lapas malām:

``` r
lapa3331uzd2 <- st_read_parquet("5uzd_faili/lapa3331mezi_2uzd.parquet")
lapa3331uzd2 <- st_transform(lapa3331uzd2, crs = 4326)

#leaflet() %>%
#  addTiles() %>%
#  addPolygons(data = lapa3331, color = "blue") %>%
#  addPolygons(data = lapa3331uzd2, color = "green") 
```

Kā jau bija gaidāms, funkcija ir apgriezusi visu, kas atrodas ārpus
lapas.

2.2. solis: izmantojiet pārskatīto funkciju no uzdevuma sākuma, lai
saglabātu katras karšu lapas rezultātu savā GeoTIFF failā, kura
nosaukums ir saistāms ar šo apakšuzdevumu un individuālo lapu. Kā
izskatās šūnu aizpildījums pie karšu lapu malām? Vai ir saglabājušies
objekti ārpus kartes lapām?

Pielietoju funkciju:

``` r
for (i in seq_along(uzd2_mezi_file_paths)) {
  mana_funkcija(mezs_dati = uzd2_mezi_file_paths[i], koku_suga = 1, uzd_nr = "2uzd")
}
```

Atkal uztaisīšu interaktīvo karti, lai paskatītos uz malām:

``` r
# pirmais rastrs
lapa3331_priedes_2uzd <- rast("5uzd_faili/lapa3331_2uzd_priedes100m.tif")
lapa3331_priedes_2uzd <- project(lapa3331_priedes_2uzd, st_crs(lapa3331)$wkt)

#otrais rastrs
lapa3332_priedes_2uzd <- rast("5uzd_faili/lapa3332_2uzd_priedes100m.tif")
lapa3332_priedes_2uzd <- project(lapa3332_priedes_2uzd, st_crs(lapa3331)$wkt)

#leaflet() %>%
#  addTiles() %>%
#  addRasterImage(lapa3331_priedes_2uzd, group = "rastrs3331") %>%
#  addRasterImage(lapa3332_priedes_2uzd, group = "rastrs3332") %>%
#  addPolygons(data = lapa3331, color = "blue", group = "lapas") %>%
#  addPolygons(data = lapa3332, color = "blue", group = "lapas") %>%
#  addLayersControl(
#    overlayGroups = c("rastrs3331","rastrs3332","lapas"),
#    options = layersControlOptions(collapsed = FALSE))
```

Kā jau sagaidāms, rastri apgriezti līdz ar lapas malām. Tomēr mani
mulsina tas, ka starp lapām ir neaizpildīta robeža, kas ir šaurāka nekā
100 m pikselis - kā tā var būt, ja izmantoju vienu references rastru?
Atkal kaut kas ar crs transformēšanu saistīts? Pa vidu dažās vietās ir
nobīdes, kas rezultējas pikseļu iztrūkumos.

Mēģināšu apvienot rastrus un tad jau redzēs.

2.3. solis: apvienojiet iteratīvajā ciklā radītos GeoTIFF failus vienā
slānī, kura nosaukums ir saistāms ar apakšuzdevumu (laika mērogošanas
beigas). Vai apvienotajā slānī ir redzamas karšu lapu robežas? Kā
vērtības sakrīt ar citiem apakšuzdevumiem?

Tā kā rastri šoreiz nepārklājas, mēģināšu pielietot merge funkciju, kam
jābūt ātrākai.

``` r
uzd2_priedes_file_paths <- list.files("5uzd_faili", pattern = "^lapa\\d{4}_2uzd_priedes100m\\.tif$", full.names = TRUE)
uzd2_priedes <- lapply(uzd2_priedes_file_paths, rast)

uzd2_priedes_merged <- do.call(merge, uzd2_priedes)
writeRaster(uzd2_priedes_merged, "Uzd2_priedes100m_merged.tif", overwrite = TRUE)
```

Skatos, kas sanācis

``` r
#leaflet() %>%
#  addTiles() %>%
#  addRasterImage(uzd2_priedes_merged) %>%
#  addPolygons(data = lapa3331, color = "blue", group = "lapas") %>%
#  addPolygons(data = lapa3332, color = "blue", group = "lapas") %>%
#  addPolygons(data = lapa3333, color = "blue", group = "lapas") %>%
#  addPolygons(data = lapa3334, color = "blue", group = "lapas") %>%
#  addLayersControl(
#    overlayGroups = "lapas",
#    options = layersControlOptions(collapsed = FALSE))
```

Robežlīnijas vairs nav redzamas un rastrs ir vienlaidus. Acīmredzot
tiešām problēma ir crs transformēšanā…

Laika mērīšanai:

``` r
SakumsUzd2 <- Sys.time()

# 2.1. solis
for (i in lapas_list_paths) {
  lapa <- st_read_parquet(i)
  mezi <-st_read_parquet("../centra_mezi/centra_mezi_labots.parquet")
  mezi <- st_transform(mezi, st_crs(lapa)) 
  fails <- st_intersection(mezi, lapa)
  basename <- tools::file_path_sans_ext(basename(i))
  nosaukums <- file.path("5uzd_faili", paste0(basename,"mezi_2uzd.parquet"))
  st_write_parquet(fails, nosaukums)
}

# 2.2. solis
for (i in seq_along(uzd2_mezi_file_paths)) {
  mana_funkcija(mezs_dati = uzd2_mezi_file_paths[i], koku_suga = 1, uzd_nr = "2uzd")
}

# 2.3. solis
uzd2_priedes_file_paths <- list.files("5uzd_faili", pattern = "^lapa\\d{4}_2uzd_priedes100m\\.tif$", full.names = TRUE)
uzd2_priedes <- lapply(uzd2_priedes_file_paths, rast)

uzd2_priedes_merged <- do.call(mosaic, uzd2_priedes)
writeRaster(uzd2_priedes_merged, "Uzd2_priedes100m_merged.tif", overwrite = TRUE)

BeigasUzd2 <- Sys.time()
print(BeigasUzd2 - SakumsUzd2)
```

    ## Time difference of 1.945556 mins

Viss kopā aizņēma aptuveni 2,5 minūtes.

``` r
rm(fails,lapa,lapa3331_priedes_2uzd, lapa3332_priedes_2uzd, lapa3331uzd2, uzd2_mezi, uzd2_priedes, BeigasUzd2, SakumsUzd2, uzd2_mezi_file_paths, uzd2_priedes_file_paths)
```

# 3) Spatial filtering

3.1. solis: sāciet iteratīvo ciklu, kurā, izmantojot spatial filtering
iegūstiet MVR datus ar apstrādājamo kartes lapu (šis ir sākums laika
mērogošanai). Kāds ir objektu skaits katrā kartes lapā, kā tas saistās
ar iepriekšējo apakšuzdevumu? Ārpus cikla apskatiet katrai kartes lapai
piederīgos objektus - vai tie atrodas tikai kartes lapas iekšienē, vai
ir iekļauti visi objekti, kas ir uz kartes lapas robežām?

Spatial filtering (izmanto objektu savstarpējo pārklāšanos) mērķiem
pieejamas dažādas funkcijas ar dažādiem attiecību nosacījumiem,
izvēlējos izmantot funkciju st_within, kas atgriež poligonus, kuri
pilnībā ietilpst lapā.

``` r
for (i in lapas_list_paths) {
  lapa <- st_read_parquet(i)
  mezi <-st_read_parquet("../centra_mezi/centra_mezi_labots.parquet")
  mezi <- st_transform(mezi, st_crs(lapa)) 
  fails <- st_filter(mezi, lapa, .predicate = st_within)
  basename <- tools::file_path_sans_ext(basename(i))
  nosaukums <- file.path("5uzd_faili", paste0(basename,"mezi_3uzd.parquet"))
  st_write_parquet(fails, nosaukums)
}

# saglabāto failu saraksts, ko vēlāk pielietot ciklā
uzd3_mezi_file_paths <- list.files("5uzd_faili", pattern = "^lapa\\d{4}mezi_3uzd\\.parquet$", full.names = TRUE)

uzd3_mezi <- map(uzd3_mezi_file_paths, st_read_parquet)
```

Katrā lapā piederīgi ir 15271, 28223, 32753 un 32117 objekti - tas ir
mazāk, nekā ar iepriekšējām divām funkcijām (par ~ 500 - 800 objektiem
mazāk). Tas bija sagaidāms, jo st_within atlasa tikai objektus, kas
pilnībā ietilpst lapā - tātad uz malām esošie objekti tiek atmesti.

``` r
lapa3331uzd3 <- st_read_parquet("5uzd_faili/lapa3331mezi_3uzd.parquet")
lapa3331uzd3 <- st_transform(lapa3331uzd3, crs = 4326)

#leaflet() %>%
#  addTiles() %>%
#  addPolygons(data = lapa3331, color = "blue") %>%
#  addPolygons(data = lapa3331uzd3, color = "green") 
```

Redzams, ka tiešām atlasīti tikai lapā pilnībā ietilstošie objekti.

3.2. solis: izmantojiet pārskatīto funkciju no uzdevuma sākuma, lai
saglabātu katras karšu lapas rezultātu savā GeoTIFF failā, kura
nosaukums ir saistāms ar šo apakšuzdevumu un individuālo lapu. Kā
izskatās šūnu aizpildījums pie karšu lapu malām? Vai ir saglabājušies
objekti ārpus kartes lapām?

Pielietoju funkciju:

``` r
for (i in seq_along(uzd3_mezi_file_paths)) {
  mana_funkcija(mezs_dati = uzd3_mezi_file_paths[i], koku_suga = 1, uzd_nr = "3uzd")
}
```

Kārtējā interaktīvā karte:

``` r
# pirmais rastrs
lapa3331_priedes_3uzd <- rast("5uzd_faili/lapa3331_3uzd_priedes100m.tif")
lapa3331_priedes_2uzd <- project(lapa3331_priedes_3uzd, st_crs(lapa3331)$wkt)

# otrais rastrs
lapa3332_priedes_3uzd <- rast("5uzd_faili/lapa3332_3uzd_priedes100m.tif")
lapa3332_priedes_3uzd <- project(lapa3332_priedes_3uzd, st_crs(lapa3331)$wkt)

#leaflet() %>%
#  addTiles() %>%
#  addRasterImage(lapa3331_priedes_3uzd, group = "rastrs3331") %>%
#  addRasterImage(lapa3332_priedes_3uzd, group = "rastrs3332") %>%
#  addPolygons(data = lapa3331, color = "blue", group = "lapas") %>%
#  addPolygons(data = lapa3332, color = "blue", group = "lapas") %>%
#  addLayersControl(
#    overlayGroups = c("rastrs3331","rastrs3332","lapas"),
#    options = layersControlOptions(collapsed = FALSE))
```

Man bija aizdomas, ka kādā no malām poligoni neaizsniegsies līdz pašām
lapas malām, attiecīgi radot pārrāvumus starp atsevišķiem rastriem,
tomēr izskatās, ka tas tā nav noticis. Tomēr saredzu šajā problēmu, ka
ne vienmēr var tā paveikties.

3.3. solis: apvienojiet iteratīvajā ciklā radītos GeoTIFF failus vienā
slānī, kura nosaukums ir saistāms ar apakšuzdevumu (laika mērogošanas
beigas). Vai apvienotajā slānī ir redzamas karšu lapu robežas? Kā
vērtības sakrīt ar citiem apakšuzdevumiem?

Apvienoju failus vienā slānī ar merge:

``` r
uzd3_priedes_file_paths <- list.files("5uzd_faili", pattern = "^lapa\\d{4}_3uzd_priedes100m\\.tif$", full.names = TRUE)
uzd3_priedes <- lapply(uzd3_priedes_file_paths, rast)

uzd3_priedes_merged <- do.call(merge, uzd3_priedes)
writeRaster(uzd3_priedes_merged, "Uzd3_priedes100m_merged.tif", overwrite = TRUE)
```

Skatos, kas sanācis:

``` r
#leaflet() %>%
#  addTiles() %>%
#  addRasterImage(uzd3_priedes_merged) %>%
#  addPolygons(data = lapa3331, color = "blue", group = "lapas") %>%
#  addPolygons(data = lapa3332, color = "blue", group = "lapas") %>%
#  addPolygons(data = lapa3333, color = "blue", group = "lapas") %>%
#  addPolygons(data = lapa3334, color = "blue", group = "lapas") %>%
#  addLayersControl(
#    overlayGroups = "lapas",
#    options = layersControlOptions(collapsed = FALSE))
```

Pārrāvumu nav, tomēr redzams, ka tieši uz lapu robežām vērtības ir
zemākas - tas tāpēc, ka nav iekļauti uz robežām vai tuvu tām ārpusē
esošie poligoni, kas tad attiecīgi nav pierēķināti priežu mežu
īpatsvaram un rada kļūdas rastrā.

Laika mērīšanai:

``` r
SakumsUzd3 <- Sys.time()

# 3.1. solis
for (i in lapas_list_paths) {
  lapa <- st_read_parquet(i)
  mezi <-st_read_parquet("../centra_mezi/centra_mezi_labots.parquet")
  mezi <- st_transform(mezi, st_crs(lapa)) 
  fails <- st_filter(mezi, lapa, .predicate = st_within)
  basename <- tools::file_path_sans_ext(basename(i))
  nosaukums <- file.path("5uzd_faili", paste0(basename,"mezi_3uzd.parquet"))
  st_write_parquet(fails, nosaukums)
}

# 3.2. solis
for (i in seq_along(uzd3_mezi_file_paths)) {
  mana_funkcija(mezs_dati = uzd3_mezi_file_paths[i], koku_suga = 1, uzd_nr = "3uzd")
}

# 3.3. solis
uzd3_priedes_file_paths <- list.files("5uzd_faili", pattern = "^lapa\\d{4}_3uzd_priedes100m\\.tif$", full.names = TRUE)
uzd3_priedes <- lapply(uzd3_priedes_file_paths, rast)

uzd3_priedes_merged <- do.call(mosaic, uzd3_priedes)
writeRaster(uzd3_priedes_merged, "Uzd2_priedes100m_merged.tif", overwrite = TRUE)

BeigasUzd3 <- Sys.time()
print(BeigasUzd3 - SakumsUzd3)
```

    ## Time difference of 1.8413 mins

Viss kopā aizņēma 2,5 minūtes.

``` r
rm(fails,lapa,lapa3331_priedes_3uzd, lapa3332_priedes_3uzd, lapa3331uzd3, uzd3_mezi, uzd3_priedes, BeigasUzd3, SakumsUzd3, uzd3_mezi_file_paths, uzd3_priedes_file_paths)
```

# 4) Funkcija bez iteratīvā procesa

4.1. solis: ar sevis izvēlētu pieeju atlasiet mežaudzes, kuras pieder
kādai no izvēlētajām karšu lapām (sākums laika mērogošanai).

4.2. solis: ieviesiet funkciju atlasītajām mežaudzēm bez iteratīvā
procesa, bet nodrošinot rezultējošā rastra aptveri tikai izvēlētajām
kartes lapām (beigas laika mērogošanai). Kā vērtības sakrīt ar citiem
apakšuzdevumiem?

Tā kā beigu rezultāts tāpat ir kopīgs rastrs, apvienošu visas četras
lapas vienā slānī. Tās aizvien glabājas objektā lapas_list, tikai
jāsavieno atkal kopā.

``` r
lapas <- bind_rows(lapas_list)
```

Izmantošu pieeju ar st_intersection, jo pagaidām tā šķita precīzākā. Te
uzreiz varu mērīt laiku, jo pa vidu neko neapskatīšu - funkciju darbība
ir skaidra. Vērtības salīdzināšu pašās beigās visiem apakšuzdevumiem
kopā.

``` r
SakumsUzd4 <- Sys.time()

uzd4_mezi <- st_intersection(mezi, lapas)
st_write_parquet(uzd4_mezi, "5uzd_faili/mezi_4uzd.parquet")

# modificētā funkcija šim uzdevumam
mana_funkcija_4uzd <- function(mezs_dati, koku_suga) {
mezs_dati <- st_read_parquet(mezs_dati)
ref10m <- rast(("../ref_rastri/LV10m_10km.tif"))
ref10m <- crop(ref10m, mezs_dati)
ref100m <- rast(("../ref_rastri/LV100m_10km.tif"))
ref100m <- crop(ref100m, mezs_dati)
priezu_mezi <- mezs_dati %>% filter(s10 == koku_suga)
priezu_mezi_10m <- rasterize(priezu_mezi,ref10m, background = 0)
priezu_mezi_10m[is.na(ref10m)] <- NA
priezu_mezi_100m <-
  terra::resample(priezu_mezi_10m,ref100m,method="average",
                  filename = "4uzd_priedes100m.tif", overwrite=TRUE)
}

# pielietoju funkciju
uzd4_priedes <- mana_funkcija_4uzd(mezs_dati = "5uzd_faili/mezi_4uzd.parquet", koku_suga = 1)

BeigasUzd4 <- Sys.time()
print(BeigasUzd4 - SakumsUzd4)
```

    ## Time difference of 29.79132 secs

Process aizņēma aptuveni pus minūti

``` r
rm(uzd4_mezi, BeigasUzd4, SakumsUzd4, mana_funkcija_4uzd)
```

# 5) Funkcija visai MVR informācijai

5.1. solis: ieviesiet funkciju visai MVR informācijai (sākums laika
mērogošanai)

Pieļauju, ka ar “visu MVR informāciju” domāta tikai centra
virsmežniecība?

Ieviešu un pielietoju funkciju:

``` r
SakumsUzd5 <- Sys.time()

mana_funkcija_5uzd <- function(mezs_dati, koku_suga) {
mezs_dati <- st_read_parquet(mezs_dati)
ref10m <- rast(("../ref_rastri/LV10m_10km.tif"))
ref10m <- crop(ref10m, mezs_dati)
ref100m <- rast(("../ref_rastri/LV100m_10km.tif"))
ref100m <- crop(ref100m, mezs_dati)
priezu_mezi <- mezs_dati %>% filter(s10 == koku_suga)
priezu_mezi_10m <- rasterize(priezu_mezi,ref10m, background = 0)
priezu_mezi_10m[is.na(ref10m)] <- NA
priezu_mezi_100m <-
  terra::resample(priezu_mezi_10m,ref100m,method="average", 
                  filename = "5uzd_priedes100m.tif", overwrite=TRUE)
uzd5_priedes <- crop(rast("5uzd_priedes100m.tif"), lapas,)
writeRaster(uzd5_priedes,"5uzd_priedes100m.tif", overwrite = TRUE)
}

mana_funkcija_5uzd(mezs_dati = "../centra_mezi/centra_mezi_labots.parquet", koku_suga = 1)
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

``` r
BeigasUzd5 <- Sys.time()
print(BeigasUzd5 - SakumsUzd5)
```

    ## Time difference of 1.326294 mins

Process aizņēma aptuveni 2 minūtes

``` r
rm(basename, BeigasUzd5 ,i ,nosaukums, SakumsUzd5)
```

Tā kā iepriekš nesalīdzināju iegūtos rezultātus, darīšu to tagad:

``` r
# ielasu failus
uzd1_priedes_merged <- rast("Uzd1_priedes100m_merged.tif")
uzd2_priedes_merged <- rast("Uzd2_priedes100m_merged.tif")
uzd3_priedes_merged <- rast("Uzd3_priedes100m_merged.tif")
uzd4_priedes <- rast("4uzd_priedes100m.tif")
uzd5_priedes <- rast("5uzd_priedes100m.tif")

#leaflet() %>%
#  addTiles() %>%
#  addRasterImage(uzd1_priedes_merged, group = "1. uzd - st_join") %>%
#  addRasterImage(uzd2_priedes_merged, group = "2. uzd - st_intersection") %>%
#  addRasterImage(uzd3_priedes_merged, group = "3. uzd - st_filter(.predicate = st_within)") %>%
#  addRasterImage(uzd4_priedes, group = "4. uzd - bez iterācijām") %>%
#  addRasterImage(uzd5_priedes, group = "5. uzd - izgriežot") %>%
#  addPolygons(data = lapa3331, color = "blue", group = "lapas") %>%
#  addPolygons(data = lapa3332, color = "blue", group = "lapas") %>%
#  addPolygons(data = lapa3333, color = "blue", group = "lapas") %>%
#  addPolygons(data = lapa3334, color = "blue", group = "lapas") %>%
#  addLayersControl(
#    overlayGroups = c("lapas","1. uzd - st_join", "2. uzd - st_intersection", "3. uzd - #st_filter(.predicate = st_within)", "4. uzd - bez iterācijām", "5. uzd - izgriežot"),
#    options = layersControlOptions(collapsed = FALSE))
```

No rastriem visprecīzākā informācija sagaidāma no 5. uzdevuma - tur
teorētiski nevar rasties problēmas rastra malās, tāpēc to varētu
uzskatīt par precīzāko reālās situācijas atspoguļojumu un primāri ar to
salīdzināšu visu pārējo. 4. uzdevuma rezultāts sagaidāms tāds pats
vismaz lapu savienojuma vietās, jo aprēķins veikts visai teritorijai
reizē. Pārējos rastros visticamāk sagaidāmas vismaz kaut kādas
problēmas, līmējot tos kopā.

Skatos, ka rezultāti 2., 4. un 5. uzdevumā ir identiski, tātad
st_intersection šajā gadījumā ir bijusi efektīva pieeja ar precīzām malu
vērtībām.

Robežlīnijas salīmētajos rastros ir redzamas 1. un 3. uzdevuma
rastriem - pie robežam vērtības ir zemākas, jo tur tiek zaudēta
informācija par malām. Bet, ja pareizi saprotu, kā notiek šis process,
tad tas, cik daudz informācijas gar malām tiek pazaudēts, ir atkarīgs no
tā, cik tālu 100m rastrā iestiepjas 10m rastra šūnas (ja tukšas, tad
pārrēķinā laikam tiek uztvertas kā šūna ar vērtību 0, jo šūnas gar malām
savu izmēru nav mainījušas). Man netapa skaidrs, kāpēc ar šo ir
problēmas 1. apakšuzdevumā - tika taču saglabāti visi poligoni arī ārpus
lapas. Teorētiski, tā kā bija pārklāšanās, pēc salīmēšanas (ņemot vērā
maksimālās vērtības, nevis vidējo vai ko citu) nevajadzēja būt
problēmām?

Kartes lapām droši vien ir “apaļi” malu izmēri, kuros references rastri
smuki ietilpst, tāpēc, precīzi apgriežot mežu datus ar st_intersects,
info gar malām netiek zaudēts. To apliecina arī tas, ka 2. un 4.
uzdevuma vektors ir rezultējies identiskā rastrā kā 5. - tomēr pretēji
intuīcijai 2. uzdevums aizņēma mazliet vairāk laika (laiks gan mēdza
krietni variēt starp izpildes reizēm, tāpēc to grūti salīdzināt). Tātad
ilgāk ir iterēt pa kartas lapām, nekā visu vajadzīgo veikt visai
teritorijai uzreiz?

Ņemot vērā iepriekšminēto, efektīvākā versija šķita 4. uzdevuma
versija - tas bija atrāk un komandrindas bija īsākas un vienkāršākas,
nekas nebija jālīmē kopā. Ņemot vērā premisi un aprakstus tajā par
efektīvu telpas dalīšanu, mani mulsina, ka telpas dalīšana aizņēma
vairāk laika. Tomēr strādāju ar salīdzinoši mazu telpu - nezinu, cik
ilgs būtu šis process lielākā telpā.

Šajā gadījumā nebija būtiski pievērst uzmanību atribūtiem datu tabulās,
jo jāpielieto bija tikai viena kolonna no MVR datiem (1. stāva 1. suga)
tāpēc es tiem īpaši nepievērsu uzmanību, bet citos gadījumos tas varētu
būt svarīgi.

Domājot par citiem EGV šajā projektā - ir jāapreķina, piemēram, attālumi
no konkrētiem objektiem. Tādos gadījumos būtu vēl svarīgāk apdomāt, kā
nodrošināt pareizus datus gar telpas malām, ja tās tiek līmētas kopā.

# 6) Jaudīgiem datoriem un dažādu pieeju izmēģināšanai

Saistiet atbilstošo mežaudžu informāciju ar 100 m tīklu no projekta
repozitorija un izveidojiet projekta repozitorijā esošajam 100 m rastra
tīklam atbilstošu rastru pēc šūnu novietojuma un vērtībām. Kad iegūstat
salīdzināmu rezultātu, mērogojiet tā izpildei nepieciešamo laiku. Kā
vērtības sakrīt ar citiem apakšuzdevumiem?

Mans dators nav jaudīgs :’) un šis izklausās pēc ārkārtīgi ilga un smaga
procesa, un es nesaredzu, kā šī pieeja varētu radīt ievērojami labāku
rezultātu.

Ņemot vērā visu iepriekšminēto, atļaujos šo uzdevumu neveikt, lai
virzītos tālāk apmācībās.
