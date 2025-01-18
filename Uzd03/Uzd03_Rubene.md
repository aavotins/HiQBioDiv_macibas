Uzd03_Rubene
================
Betija Rubene
2025-01-15

## Trešais uzdevums - rastra dati, rasterizēšana un kodējumi

Darbam nepieciešamās pakotnes

``` r
library(terra)
library(curl)
library(tidyverse)
library(archive)
library(arrow)
library(sfarrow)
library(sf)
library(httr)
library(ows4R)
library(terra)
library(fasterize)
library(microbenchmark)
```

Vispirms jālejuplādē abas mapes ar references slāņiem

``` r
url_vekt <- "https://zenodo.org/api/records/14277114/files-archive"
url_rast <- "https://zenodo.org/api/records/14497070/files-archive"
dest_vekt <- "C:/Users/ruben/Documents/HiQBioDiv_macibas_fork/ref_vektori.zip"
dest_rast <- "C:/Users/ruben/Documents/HiQBioDiv_macibas_fork/ref_rastri.zip"
curl_download(url_vekt, destfile = dest_vekt)
curl_download(url_rast, destfile = dest_rast)
file.exists(dest_vekt) # izdevās
```

    ## [1] TRUE

``` r
file.exists(dest_rast) # izdevās
```

    ## [1] TRUE

Atarhivēšana

``` r
archive_extract(dest_vekt, dir = "C:/Users/ruben/Documents/HiQBioDiv_macibas_fork/ref_vektori")
archive_extract(dest_rast, dir = "C:/Users/ruben/Documents/HiQBioDiv_macibas_fork/ref_rastri")

rm(url_vekt, url_rast, dest_vekt, dest_rast)
# noņemu lieko
```

# 1) Lauku atbalsta dienesta datu lejupielāde

Nepieciešami tikai dati par teritoriju, kuru aptver otrā uzdevuma Mežu
valsts reģistra dati. Sākumā jāsaprot, kādu teritoriju tie aptvēra.

``` r
centra_mezi <- st_read_parquet("../centra_mezi/centra_mezi_labots.parquet", quiet = TRUE)
teritorija <- st_bbox(centra_mezi) # šajā objektā tagad ir aptvertās teritorijas robežpunkti
```

Vispirms apskatīšu, kas ir pieejams WFS serverī

``` r
wfs_lad <- "https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer"

url <- parse_url(wfs_lad)
url$query <- list(service = "wfs",
                  request = "GetCapabilities"
                  )
request <- build_url(url)
request
```

    ## [1] "https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer?service=wfs&request=GetCapabilities"

Ievietojot šo saiti pārlūkprogrammā, iespējams iepazīties ar WFS saturu
un iespējām. Bet var arī pieprasīt konkrētu informāciju ar R
komandrindām.

Sākumā jāizveido savienojums ar WFS

``` r
client <- WFSClient$new(wfs_lad, 
                            serviceVersion = "2.0.0")
```

Kādi slāņi ir pieejami?

``` r
client$getFeatureTypes(pretty = TRUE)
```

    ##        name title
    ## 1 LAD:Lauki Lauki

Ir tikai viens slānis, kurā pieejami LAD lauku dati ar nosaukumu
LAD:Lauki. Tā kā šie dati pieejami WGS-84 koordinātu sistēmā, tad
jākonvertē teritoriju ierobežojošās koordinātas (tās ir LKS-92)

``` r
teritorija <- st_as_sfc(teritorija)
teritorija <- st_transform(teritorija, crs = 4326)
st_bbox(teritorija)
```

    ##     xmin     ymin     xmax     ymax 
    ## 22.42279 56.24303 25.57287 57.40252

Tos iegūstu, papildus definējot, ka vēlos tikai datus, kas ietilpst
centra mežus iekļaujošajā taisnsnstūrī.

``` r
#teritorija_wgs <- st_transform(teritorija, crs = 4326)
url$query <- list(service = "WFS",
                  request = "GetFeature",
                  typename = "Lauki",
                  srsName = "EPSG:4326",
                  bbox = "22.42279, 56.24303, 25.57287, 57.40252")
request2 <- build_url(url)
LAD_lauki <- st_read(request2, quiet = TRUE)

# Tālāk arī gribu strādāt ar parquet failu, pārveidoju
dir.create("../LAD_dati") # uztaisu jaunu mapi
st_write_parquet(LAD_lauki, "../LAD_dati/LAD_lauki.parquet")
LAD_lauki <- st_read_parquet("../LAD_dati/LAD_lauki.parquet", quiet = TRUE)

rm(client,teritorija,url,request,request2,wfs_lad)
#noņemu lieko
```

# 2) Vektordatu rasterizācija

Nepieciešams 10 m izšķirtspējā rasterizēt LAD laukus tā, lai Latvijā
ietilpstošajā teritorijā vietas, kur ir lauku bloki, kodētas ar “1”,
vietas, kurās nav lauku bloku, kodētas ar “0”, bet vietas ārpus Latvijas
ir bez vērtībām

Ielasu references failu 10 m izšķirtspējā

``` r
ref10m <- rast("../ref_rastri/LV10m_10km.tif")
plot(ref10m) # LV teritorija satur vērtības "1", ārpus LV teritorijas vērtību nav
```

![](Uzd03_Rubene_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

Rasterizēšu LAD laukus. Vispirms jāpārbauda, vai sakrīt koordinātu
sistēmas

``` r
st_crs(LAD_lauki) # WGS-84
```

    ## Coordinate Reference System:
    ##   User input: WGS 84 
    ##   wkt:
    ## GEOGCRS["WGS 84",
    ##     ENSEMBLE["World Geodetic System 1984 ensemble",
    ##         MEMBER["World Geodetic System 1984 (Transit)"],
    ##         MEMBER["World Geodetic System 1984 (G730)"],
    ##         MEMBER["World Geodetic System 1984 (G873)"],
    ##         MEMBER["World Geodetic System 1984 (G1150)"],
    ##         MEMBER["World Geodetic System 1984 (G1674)"],
    ##         MEMBER["World Geodetic System 1984 (G1762)"],
    ##         MEMBER["World Geodetic System 1984 (G2139)"],
    ##         ELLIPSOID["WGS 84",6378137,298.257223563,
    ##             LENGTHUNIT["metre",1]],
    ##         ENSEMBLEACCURACY[2.0]],
    ##     PRIMEM["Greenwich",0,
    ##         ANGLEUNIT["degree",0.0174532925199433]],
    ##     CS[ellipsoidal,2],
    ##         AXIS["geodetic latitude (Lat)",north,
    ##             ORDER[1],
    ##             ANGLEUNIT["degree",0.0174532925199433]],
    ##         AXIS["geodetic longitude (Lon)",east,
    ##             ORDER[2],
    ##             ANGLEUNIT["degree",0.0174532925199433]],
    ##     USAGE[
    ##         SCOPE["Horizontal component of 3D system."],
    ##         AREA["World."],
    ##         BBOX[-90,-180,90,180]],
    ##     ID["EPSG",4326]]

``` r
st_crs(ref10m) # LKS-92
```

    ## Coordinate Reference System:
    ##   User input: LKS-92 / Latvia TM 
    ##   wkt:
    ## PROJCRS["LKS-92 / Latvia TM",
    ##     BASEGEOGCRS["LKS-92",
    ##         DATUM["Latvian geodetic coordinate system 1992",
    ##             ELLIPSOID["GRS 1980",6378137,298.257222101,
    ##                 LENGTHUNIT["metre",1]]],
    ##         PRIMEM["Greenwich",0,
    ##             ANGLEUNIT["degree",0.0174532925199433]],
    ##         ID["EPSG",4661]],
    ##     CONVERSION["Latvian Transverse Mercator",
    ##         METHOD["Transverse Mercator",
    ##             ID["EPSG",9807]],
    ##         PARAMETER["Latitude of natural origin",0,
    ##             ANGLEUNIT["degree",0.0174532925199433],
    ##             ID["EPSG",8801]],
    ##         PARAMETER["Longitude of natural origin",24,
    ##             ANGLEUNIT["degree",0.0174532925199433],
    ##             ID["EPSG",8802]],
    ##         PARAMETER["Scale factor at natural origin",0.9996,
    ##             SCALEUNIT["unity",1],
    ##             ID["EPSG",8805]],
    ##         PARAMETER["False easting",500000,
    ##             LENGTHUNIT["metre",1],
    ##             ID["EPSG",8806]],
    ##         PARAMETER["False northing",-6000000,
    ##             LENGTHUNIT["metre",1],
    ##             ID["EPSG",8807]]],
    ##     CS[Cartesian,2],
    ##         AXIS["northing (X)",north,
    ##             ORDER[1],
    ##             LENGTHUNIT["metre",1]],
    ##         AXIS["easting (Y)",east,
    ##             ORDER[2],
    ##             LENGTHUNIT["metre",1]],
    ##     USAGE[
    ##         SCOPE["Engineering survey, topographic mapping."],
    ##         AREA["Latvia - onshore and offshore."],
    ##         BBOX[55.67,19.06,58.09,28.24]],
    ##     ID["EPSG",3059]]

Nesakrīt - pārveidoju

``` r
LAD_lauki <- st_transform(LAD_lauki, crs = 3059, quiet = TRUE)
```

Tālāk veicu rasterizāciju. Rasterizēju bez argumenta touches = TRUE, jo
šajā gadījumā mums droši vien interesē ar 1 atzīmēt vērtības, kurās ir
lauki - gan jau izpratnē, kur lauki aizņem lielāko daļu pikseļa (domāju
no ekoloģijas aspekta, ka gan jau nevajag ar 1 atzīmēt pikseļus, kur tam
pieskaras tikai maliņa). Bez šī argumenta rasterize funkcija ar vērtību
“1” aizpilda tos pikseļus, kuru centroīds ietilpst lauku poligonos, kas
(visbiežāk) nozīmē, ka pikseļa lielāko daļu aizņem lauki

Tā kā mēs nestrādājam ar visu Latvijas teritoriju, vispirms apgriežu
references slāni, izmantojot LAD lauku faila aptverto teritoriju. Tas
arī paātrinās lietu notikšanu. Tad veicu rasterizāciju

``` r
ref10m_cropped <- crop(ref10m, LAD_lauki)
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
LAD_lauki_rast <- rasterize(LAD_lauki, ref10m_cropped)
plot(LAD_lauki_rast)
```

![](Uzd03_Rubene_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

Šobrīd ar vērtību “1” atzīmēti pikseļi ar laukiem, viss pārējais ir NA.
Jāpanāk, lai pikseļi bez laukiem, kuri ietilpst Latvijas teritorijā,
būtu ar vērtību “0”

``` r
LAD_lauki_10m <- classify(LAD_lauki_rast, cbind(NA, 0)) # nomainu visas NA vērtības uz 0
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
plot(LAD_lauki_10m)
```

![](Uzd03_Rubene_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->

``` r
LAD_lauki_10m[is.na(ref10m_cropped)] <- NA # pārtaisa par NA tās šūnas, kuras references failā ir NA
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

``` r
plot(LAD_lauki_10m)
```

![](Uzd03_Rubene_files/figure-gfm/unnamed-chunk-14-2.png)<!-- -->

``` r
writeRaster(LAD_lauki_10m, "../LAD_dati/LAD_lauki_10m.tif", overwrite = TRUE) # saglabāju gala failu
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
LAD_lauki_10m <- rast("../LAD_dati/LAD_lauki_10m.tif")
rm(LAD_lauki_rast) # noņemu lieko
```

# 3) Terra funkciju izmēģināšana - jauns rastrs

No iepriekš sagatavotā rastra jāizveido jauns rastrs ar malas garumu
100m, kā arī informācija par lauka bloku īpatsvaru.

Es neesmu droša, vai līdz galam izpratu funkcijas aggregate un resample,
bet izmēģināju divus variantus:

Pirmais - uzreiz ar resample funkciju pielīdzinot references rastram,
izmantojot metodi “sum”, kas uzreiz aprēķina pikseļu ar laukiem skaitu
jaunajā 100 m šūnā. Tā kā 100m šūnā ietilpst 100 10m šūnas, tad šajā
gadījumā vērtības norāda uz īpatsvaru

``` r
ref100m <- rast("../ref_rastri/LV100m_10km.tif") # ielasu references rastru 100 m izšķirtspējā
ref100m_cropped <- crop(ref100m, LAD_lauki)

atrums1 <- microbenchmark(
  LAD_lauki_100m_1 <- resample(LAD_lauki_10m, ref100m_cropped, method = "sum"), times = 3)
plot(LAD_lauki_100m_1) # izskatās pareizi
```

![](Uzd03_Rubene_files/figure-gfm/unnamed-chunk-15-1.png)<!-- -->

``` r
atrums1
```

    ## Unit: seconds
    ##                                                                               expr
    ##  LAD_lauki_100m_1 <- resample(LAD_lauki_10m, ref100m_cropped,      method = "sum")
    ##      min       lq     mean   median      uq      max neval
    ##  121.359 121.6304 122.1896 121.9018 122.605 123.3081     3

Otrais - vispirms ar funkciju aggregate, kura savieno kopā šūnas
(arguments fact norāda, cik lielās), aprēķināt lauku šūnu īpatsvaru, un
tikai pēc tam ar resample pielīdzināt jauno rastru references slānim
(izmantoju metodi “near”, kas piemērota kategoriju datiem)

``` r
atrums2 <- microbenchmark(LAD_lauki_100m_2 <- LAD_lauki_10m %>%
    aggregate(fact = 10, fun = sum) %>%
    resample(ref100m_cropped, method = "near"), times = 3)
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

``` r
atrums2
```

    ## Unit: seconds
    ##                                                                                                                       expr
    ##  LAD_lauki_100m_2 <- LAD_lauki_10m %>% aggregate(fact = 10, fun = sum) %>%      resample(ref100m_cropped, method = "near")
    ##     min       lq     mean   median       uq      max neval
    ##  2.3489 2.361621 2.386864 2.374342 2.405845 2.437349     3

``` r
plot(LAD_lauki_100m_2) # arī izskatās pareizi
```

![](Uzd03_Rubene_files/figure-gfm/unnamed-chunk-16-1.png)<!-- -->

Otrā pieeja ir daudzreiz ātrāka.

Funkcijai project neredzēju pielietojumu, jo tā domāta koordinātu
sistēmu saskaņošanai, bet tas šajā gadījumā nebija aktuāli

``` r
rm(atrums1, atrums2, LAD_lauki, LAD_lauki_10m, LAD_lauki_100m_1, ref10m, ref10m_cropped, ref100m, ref100m_cropped) # noņemu lieko
```

# 4) Jaunu slāņu sagatavošana

Pirmais - nepieciešams sagatavot slāni ar informāciju par lauka bloku
platību.

Šis uzdevums mani mazliet mulsina, jo minēts, ka jāizmanto iepriekš
iegūtais rastrs un lauka bloku platības jānoapaļo līdz procentiem. Kā
jau iepriekš minēts, pikseļu izmēru dēļ aprēķinātās vērtības ar sum
atbilst arī procentiem. Ar terminu “lauka bloku platību” es uztvertu, ka
jāaprēķina reālās lauka bloku platības, izmantojot vektordatus, tomēr
specifiski norādīts, ka jāizmanto iepriekš sagatavotais rastrs, kā arī
šāda platību aprēķināšana izklausās smaga datoram. Ņemot vērā visu
iepriekšminēto, kā arī \#29 diskusiju repozitorijā, palieku pie jau
aprēķinātā rastra - tas atbilst uzdevuma aprakstam, šūnas satur
procentuālās vērtības veselos skaitļos, kas aptuveni atbilst lauku
platībām, kā arī teritorija ārpus Latvijas ir bez vērtībām. To
saglabāju.

``` r
writeRaster(LAD_lauki_100m_2, "../LAD_dati/LAD_lauki_100m.tif", overwrite = TRUE)
```

Otrais - izveidot slāni, kurš satur bināri kodētas vērtības - Latvijā
esošā šūnā ir lauku bloki (vērtība 1) vai to nav (vērtība 0), tāpat kā
iepriekš saglabājot šūnas vietām ārpus Latvijas.

Tāpat kā iepriekš, domāju no ekoloģijas puses - ar vērtību “1” varētu
būt interesējoši saglabāt tikai pikseļus, kuros būtisku daļu teritorijas
sastāda lauku platības, nevis atzīmēt ar vērtību “1” jebkuru pikseli,
kurā ir vismaz 1% lauku. Tomēr uzdevuma nosacījumos rakstīts, ka jābūt
vērtībai “1”, ja pikselis vispār satur lauku blokus (uztveru kā - vismaz
1%). Tā kā norādīts, ka slānis jāsagatavo, izmantojot iepriekš
sagatavoto slāni 100 m izšķirtspējā, sākumā saglabāju to objektā ar
jaunu nosaukumu un tad šajā objektā visus pikseļus, kuros ir vērtības
“1-100” nomainu uz vērtību “1”.

``` r
LAD_lauki_100m_bin <- LAD_lauki_100m_2
LAD_lauki_100m_bin[LAD_lauki_100m_2 > 0 & LAD_lauki_100m_2 <= 100] <- 1 # nomaina vērtības 1-100 uz 1
plot(LAD_lauki_100m_bin) # izskatās pareizi
```

![](Uzd03_Rubene_files/figure-gfm/unnamed-chunk-19-1.png)<!-- -->

``` r
writeRaster(LAD_lauki_100m_bin, "../LAD_dati/LAD_lauki_100m_bin.tif", overwrite = TRUE)
```

Visbeidzot nepieciešams salīdzināt izveidoto slāņu izmērus.

Kā mainās faila izmērs uz cietā diska, atkarībā no rastra šūnas izmēra?

``` r
LAD10m <- file.info("../LAD_dati/LAD_lauki_10m.tif")$size / (1024^2)
LAD100m <- file.info("../LAD_dati/LAD_lauki_100m.tif")$size / (1024^2)
print(LAD10m)
```

    ## [1] 12.58721

``` r
print(LAD100m)
```

    ## [1] 0.3876724

Ir ievērojama atšķirība starp dažādām izšķirtspējām

Kā mainās faila izmērs uz cietā diska, atkarībā no tajā ietvertās
informācijas un tās kodējuma. Kāds kodējums (joslas tips) aizņem
vismazāko apjomu uz cietā diska, saglabājot informāciju?

Kodējumu iespējams norādīt, saglabājot failu ar funkciju writeRaster

``` r
LAD100m_bin <- file.info("../LAD_dati/LAD_lauki_100m_bin.tif")$size / (1024^2)
print(LAD100m_bin)
```

    ## [1] 0.335989

Bināri kodēto un procentuāli izteikto failu izmēriem ir atšķirība, tomēr
tā nav īpaši liela.

Izmēģināšu dažādus kodējumus. Šajā gadījumā salīdzināšu INT1S
(iespējamās vērtības no -127 līdz 127), INT1U (0 līdz 255), INT2S
(-32,767 līdz 32,767), INT2U (0 līdz 65,534) un FLT4S (-3.4e+38 līdz
3.4e+38). Intuitīvi šķiet, ka, palielinoties baitu skaitam, pieaugs
aizņemtās vietas daudzums, kā arī daļskaitļi aizņems vairāk vietas

``` r
writeRaster(LAD_lauki_100m_2, "../LAD_dati/LAD_lauki_100m_INT1S.tif", overwrite = TRUE, 
            datatype = "INT1S")
writeRaster(LAD_lauki_100m_2, "../LAD_dati/LAD_lauki_100m_INT1U.tif", overwrite = TRUE, 
            datatype = "INT1U")
writeRaster(LAD_lauki_100m_2, "../LAD_dati/LAD_lauki_100m_INT2S.tif", overwrite = TRUE, 
            datatype = "INT2S")
writeRaster(LAD_lauki_100m_2, "../LAD_dati/LAD_lauki_100m_INT2U.tif", overwrite = TRUE, 
            datatype = "INT2U")
writeRaster(LAD_lauki_100m_2, "../LAD_dati/LAD_lauki_100m_FLT4S.tif", overwrite = TRUE, 
            datatype = "FLT4S")

INT1S <- file.info("../LAD_dati/LAD_lauki_100m_INT1S.tif")$size / (1024^2)
INT1U <- file.info("../LAD_dati/LAD_lauki_100m_INT1U.tif")$size / (1024^2)
INT2S <- file.info("../LAD_dati/LAD_lauki_100m_INT2S.tif")$size / (1024^2)
INT2U <- file.info("../LAD_dati/LAD_lauki_100m_INT2U.tif")$size / (1024^2)
FLT4S <- file.info("../LAD_dati/LAD_lauki_100m_FLT4S.tif")$size / (1024^2)

print(INT1S) # 0.113534
```

    ## [1] 0.113534

``` r
print(INT1U) # 0.1135235
```

    ## [1] 0.1135235

``` r
print(INT2S) # 0.2005644
```

    ## [1] 0.2005644

``` r
print(INT2U) # 0.1887722
```

    ## [1] 0.1887722

``` r
print(FLT4S) # 0.3876724
```

    ## [1] 0.3876724

Tātad vairāk vietas cietajā diskā aizņem faili, kuriem ir lielāks baitu
skaits. Vairāk vietas aizņem slānis ar iespēju sagabāt daļskaitļus.
Atšķirība starp failiem ar negatīvām/bez negatīvām vērtībām ir, tomēr tā
ir maza.
