Uzd03_Starka
================
Rūta Starka
2025-01-18

# 1. Datu lejupielāde, atpakošana un ielasīšana

Vispirms lejupielādēju un atpakoju visus nepieciešamos slāņus no
HiQBioDiv repozitorija. Izlaižu references vektordatu lejupielādi un
atpakošanu (tikai renderēšanas ātruma dēļ), jo šī uzdevuma veikšanai tas
nav nepieciešams. Bet to esmu tos lejupilādējusi, kā pierādījums aiz
\#komandrindas.

``` r
getwd()
```

    ## [1] "D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03"

``` r
library(curl)
library(archive)

#References vektordati
#saite_vektordati = "https://zenodo.org/api/records/14277114/files-archive"
#dir_vektordati = "D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03/ref_vektordati.zip"
#curl_download(saite_vektordati, destfile = dir_vektordati)
#archive_extract(dir_vektordati, dir = "D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03/ref_vektordati")

#References rastra dati
saite_rastrs = "https://zenodo.org/api/records/14497070/files-archive"
dir_rastrs = "D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03/ref_rastrs.zip"
curl_download(saite_rastrs, destfile = dir_rastrs)
archive_extract(dir_rastrs, dir = "D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03/ref_rastrs")

rm(saite_vektordati,dir_vektordati,saite_rastrs,dir_rastrs)
```

    ## Warning in rm(saite_vektordati, dir_vektordati, saite_rastrs, dir_rastrs):
    ## object 'saite_vektordati' not found

    ## Warning in rm(saite_vektordati, dir_vektordati, saite_rastrs, dir_rastrs):
    ## object 'dir_vektordati' not found

## Piekļuve LAD datiem

Pievienojos LAD wfs serverim. Izveidoju klientu un apskatu kādi dati
pieejami serverī. Redzu, ka datos ir tikai viens slānis “Lauki”.

``` r
#Lauku atbalsta dienesta WFS serveris
library(httr)
```

    ## 
    ## Attaching package: 'httr'

    ## The following object is masked from 'package:curl':
    ## 
    ##     handle_reset

``` r
library(ows4R)# ja nepieciešams install.packages("ows4R")
```

    ## Loading ISO 19139 XML schemas...

    ## Loading ISO 19115 codelists...

``` r
saite_LAD = "https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer"#jāpalaiž
url = parse_url(saite_LAD)
url$query <- list(service = "wfs",
                  version = "2.0.0",
                  request = "GetCapabilities")
request <- build_url(url)
request#šo saiti var apskatīt pārlūkā
```

    ## [1] "https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer?service=wfs&version=2.0.0&request=GetCapabilities"

``` r
RR_client <- WFSClient$new(saite_LAD, 
                            serviceVersion = "2.0.0")

library(tidyverse)
```

    ## -- Attaching core tidyverse packages ------------------------ tidyverse 2.0.0 --
    ## v dplyr     1.1.4     v readr     2.1.5
    ## v forcats   1.0.0     v stringr   1.5.1
    ## v ggplot2   3.5.1     v tibble    3.2.1
    ## v lubridate 1.9.4     v tidyr     1.3.1
    ## v purrr     1.0.2

    ## -- Conflicts ------------------------------------------ tidyverse_conflicts() --
    ## x dplyr::filter()      masks stats::filter()
    ## x httr::handle_reset() masks curl::handle_reset()
    ## x dplyr::lag()         masks stats::lag()
    ## x readr::parse_date()  masks curl::parse_date()
    ## i Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
RR_client$getFeatureTypes() %>% map_chr(function(x){x$getTitle()})
```

    ## [1] "Lauki"

## Sasaiste ar centra virsmežniecību

Ņemot vērā, ka būs jādarbojas tikai ar datiem, kas ietilps MVR Centra
virsmežniecībā, tad nepieciešams ielasīt tā datus, kas jau sagatavoti
iepriekš. Ar funkciju st_crs() pārliecinos par šī slāņa koordinātu
sistēmu. Pēdējā rindā redzu, ka koordinātu sistēmas ID ir EPSG 3059, kas
ir LKS-92 (kas ir likumā noteiktā koordinātu sistēma, tāpēc citādi nemaz
nevajadzētu būt). Vienlaicīgi arī iespējams redzēt šo datu telpiskās
robežas, bet lai šo informāciju izmantotu, lai ierobežotu uz šo pašu
teritoriju arī LAD datus, tad papildus izpildu funkciju st_bbox(), kas
sesijā saglabā šīs robežas kā vērtības.

``` r
library(sfarrow)
library(sf)

MVR_centrs = st_read_parquet("D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd02/centrs_kopa2.parquet")

st_crs(MVR_centrs)
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

``` r
MVR_robezas = st_bbox(MVR_centrs)
MVR_robezu_koord=paste(MVR_robezas["xmin"], MVR_robezas["ymin"], MVR_robezas["xmax"], MVR_robezas["ymax"], sep = ",")
rm(MVR_robezas)
```

## LAD datu iegūšana

Tagad nepieciešams lejupielādēt LAD datus, kas limitēti uz centra
virsmežniecības robežām un saglabāt tos geoparquet formātā tālākām
darbībām uz diska. Redzu, ka ielasītajam LAD datu fragments satur
informāciju par 3000 objektiem, par katru ir 12 lauki ar dažādu
informāciju.

``` r
LAD = parse_url(saite_LAD)
LAD$query = list(
  service = "WFS",
  request = "GetFeature",
  bbox = MVR_robezu_koord,
  srsName = "EPSG:3059",
  typename = "Lauki")

request2 = build_url(LAD)
LAD_dati = st_read(request2)
```

    ## Reading layer `Lauki' from data source 
    ##   `https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer?service=WFS&request=GetFeature&bbox=405229.692%2C234180.666999999%2C594509.32%2C363287.305&srsName=EPSG%3A3059&typename=Lauki' 
    ##   using driver `GML'
    ## Simple feature collection with 3000 features and 11 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 404436.1 ymin: 233795.9 xmax: 594648.1 ymax: 363631.6
    ## Projected CRS: LKS-92 / Latvia TM

``` r
st_write_parquet(LAD_dati, dsn="LAD_centrs.parquet")
LAD_centrs=st_read_parquet(("D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03/LAD_centrs.parquet"))
```

Apskatu abu interesējošo slāņu sakritību. Redzu, ka nepieciešams vēlreiz
apgriezt LAD datus, lai tie precīzi atbilstu centra virsmežniecības
robežām, ne tikai virsmežniecības kartes extent. Izmēģināju
st_intersection, kas atlasīja 854 LAD poligonus, bet tas bija aizdomīgi,
tāpēc izpētot sīkāk sapratu, ka atlasās tikai tie LAD lauki, kas
pieskaras mežiem, kas galīgi nav vajadzīgais rezultāts. Tāpēc pieņēmu
lēmumu abortēt šo ideju un turpināt.

``` r
ggplot() +
  geom_sf(data=MVR_centrs, color = "green") +
  geom_sf(data=LAD_centrs, color = "yellow")
```

![](Uzd03_Starka_files/figure-gfm/slāņu%20sakritība-1.png)<!-- -->

``` r
LAD_centrs_inter=st_intersection(LAD_centrs, MVR_centrs)
LAD_centrs2=LAD_centrs_inter%>%select(1:12)
ggplot() +
  geom_sf(data=MVR_centrs, color = "green") +
  geom_sf(data=LAD_centrs, color = "yellow")+
  geom_sf(data=LAD_centrs2, color = "red")
```

![](Uzd03_Starka_files/figure-gfm/slāņu%20sakritība-2.png)<!-- -->

``` r
rm(LAD_centrs_inter,LAD_centrs2)
```

Redzu, ka līdzīgi kā 2. uzd, arī LAD datos ir multipolygon ģeometrijas,
tāpēc ar tādu pašu pieeju kā iepriekš pārbaudu, vai visas ir korektas.
Aizvācu izveidoto datu tabulu ģeometriju pārbaudei.

``` r
st_is_valid(LAD_centrs)
```

    ##    [1] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##   [15] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##   [29] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##   [43] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##   [57] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##   [71] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##   [85] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##   [99] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [113] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [127] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [141] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [155] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [169] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [183] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [197] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [211] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [225] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [239] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [253] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [267] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [281] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [295] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [309] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [323] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [337] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [351] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [365] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [379] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [393] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [407] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [421] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [435] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [449] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [463] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [477] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [491] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [505] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [519] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [533] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [547] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [561] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [575] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [589] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [603] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [617] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [631] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [645] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [659] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [673] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [687] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [701] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [715] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [729] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [743] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [757] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [771] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [785] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [799] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [813] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [827] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [841] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [855] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [869] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [883] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [897] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [911] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [925] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [939] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [953] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [967] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [981] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ##  [995] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1009] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1023] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1037] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1051] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1065] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1079] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1093] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1107] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1121] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1135] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1149] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1163] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1177] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1191] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1205] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1219] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1233] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1247] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1261] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1275] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1289] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1303] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1317] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1331] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1345] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1359] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1373] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1387] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1401] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1415] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1429] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1443] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1457] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1471] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1485] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1499] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1513] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1527] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1541] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1555] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1569] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1583] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1597] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1611] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1625] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1639] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1653] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1667] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1681] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1695] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1709] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1723] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1737] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1751] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1765] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1779] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1793] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1807] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1821] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1835] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1849] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1863] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1877] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1891] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1905] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1919] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1933] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1947] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1961] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1975] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [1989] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2003] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2017] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2031] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2045] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2059] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2073] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2087] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2101] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2115] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2129] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2143] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2157] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2171] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2185] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2199] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2213] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2227] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2241] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2255] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2269] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2283] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2297] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2311] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2325] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2339] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2353] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2367] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2381] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2395] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2409] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2423] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2437] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2451] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2465] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2479] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2493] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2507] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2521] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2535] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2549] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2563] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2577] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2591] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2605] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2619] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2633] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2647] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2661] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2675] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2689] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2703] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2717] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2731] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2745] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2759] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2773] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2787] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2801] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2815] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2829] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2843] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2857] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2871] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2885] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2899] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2913] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2927] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2941] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2955] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2969] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2983] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [2997] TRUE TRUE TRUE TRUE

``` r
v=data.frame(geom_val=st_is_valid(LAD_centrs$SHAPE, reason=TRUE))
sort(unique(v$geom_val))
```

    ## [1] "Valid Geometry"

``` r
rm(v)
```

Tagad tālāk darbs tikai ar lejupielādētiem datiem, tāpēc no R vides
aizvācu arī visu pārējo, kas vairs nebūs nepieciešams

``` r
rm(request2,LAD, LAD_dati,saite_LAD, RR_client, url, request, MVR_robezu_koord, MVR_centrs)
```

# 2. Rasterizēšana

## references rastrs

Lai ielasītos LAD datus rasterizētu, vispirms nepieciešams 10m
references rastrs. Ielasu un pārliecinos par koordinātu sistēmu. Tad
apgriežu rastru atbilstoši ielasīto LAD datu robežām un aizvācu rastru
par visu Latviju, lai atbrīvotu vietu. Visbeidzot, grafiski paskatos, kā
apgriezās. Redzu, ka nav izdevies kā vēlējos, rastrs apgriežas tikai
līdz kartes robežām, nevis ap ģeometriju robežām. Šo varētu risināt kaut
kā, bet eju tālāk. P.S. vēl mēģināju pirms rasterizācijas apgriezt
references rastru līdz MVR_centrs, nevis LAD_centrs, ar domu, ka tad
neizbēgami rasterizācija notiks tikai nepieciešamajās robežās, bet tas
radīja problēmas tālākajā rasterizēšanas solī, tā arī nesapratu kāpēc.
(LAD_rastrs=rasterize(LAD_centrs,ref_rastrs10m_MVRcentrs) \#Error:
external pointer is not valid ).

``` r
library(terra)
```

    ## terra 1.8.10

    ## 
    ## Attaching package: 'terra'

    ## The following object is masked from 'package:tidyr':
    ## 
    ##     extract

``` r
ref_rastrs10m=rast("D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03/ref_rastrs/LV10m_10km.tif")
st_crs(ref_rastrs10m) # EPSG 3059, tātad viss kārtībā, koordinātu sistēmas sakrīt.
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

``` r
ref_rastrs10m_LADam = terra::crop(ref_rastrs10m,LAD_centrs)
rm(ref_rastrs10m)
plot(ref_rastrs10m_LADam)
```

![](Uzd03_Starka_files/figure-gfm/references%20rastra%20ielasisana-1.png)<!-- -->

## rasterizēšana

Sākumā nepieciešams references rastru no terra objekta pārvērst par
rester objektu, lai varētu darboties ar funkciju fasterize(). Tad
rasterizēju LAD vektordatus, norādot, lai viss, kas nepārklājas ar LAD
laukiem tiek pašlaik klasificēts kā 0. Novācu lieko LAD vektordatu
slāni.

``` r
library(raster) #ja nepieciešams install.packages("raster")
```

    ## Loading required package: sp

    ## 
    ## Attaching package: 'raster'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     select

``` r
ref_rastrs2=raster::raster(ref_rastrs10m_LADam)

library(fasterize) # ja nepieciešams, install.packages("fasterize")
```

    ## 
    ## Attaching package: 'fasterize'

    ## The following object is masked from 'package:graphics':
    ## 
    ##     plot

    ## The following object is masked from 'package:base':
    ## 
    ##     plot

``` r
LAD_rastrs=fasterize(LAD_centrs,ref_rastrs2,background=0) 
rm(LAD_centrs)
```

Tagad jāpārbauda, kas sanācis. Pārbaudīšu vizuāli. Redzu, ka jebkas
cits, kas nav LAD lauki ir NA. Tomēr vēlamais rezultāts ir rastrs, kurā
ir vērtība “1”, ja šūnā ir LAD lauki, vērtība “0”, ja šūnā nav LAD lauki
, un vērtība “NA”, ja šūna atrodas ārpus Latvijas tetitorijas.

``` r
plot(LAD_rastrs)
```

![](Uzd03_Starka_files/figure-gfm/pirmā%20rastra%20vizualizēšana-1.png)<!-- -->

Tātad jāklasificē NA vērtības divās daļās. Tam varētu izmantot
references rastru, jo to vizualizējot iepriekš (chunk “references rastra
ielasisana”) bija redzams, ka tukšas (NA) ir šūnas ārpus Latvijas.
Noteikšu, ka ja references rastrā tās ir NA, tad arī šajā rastrā tās
jādefinē kā NA. Par cik ar <RasterLayer> šādi neļauj izrīkoties, sākumā
to pārvēršu uz <SpatRaster> objektu ar nosaukumu LAD_rastrs2.

``` r
LAD_rastrs2=rast(LAD_rastrs)
LAD_rastrs2[is.na(ref_rastrs10m_LADam)] = NA 
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

Apskatu rezultātu grafiski un secinu, ka viss ir sanācis.

``` r
plot(LAD_rastrs2)
```

![](Uzd03_Starka_files/figure-gfm/rastrs%20pēc%20klasifikācijas-1.png)<!-- -->

Tagad saglabāju kā GeoTIFF slāni bez kompresijas. Aizvācu liekos
references slāņus.

``` r
terra::writeRaster(LAD_rastrs2, filename="D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03/LAD_rastrs.tif", gdal=c("COMPRESS=DEFLATE", "TFW=YES"),overwrite=TRUE)
rm(ref_rastrs10m_LADam,ref_rastrs2)
```

# 3. Rastri ar 100m pikseļa malas garumu

## references rastrs

Tagad lai no iepirkšējā punktā sagatavotā rastra izveidotu jaunu rastru,
kam pikseļa malas garums ir 100m pašreizējo 10m vietā, sākumā jāielasa
un līdz LAD_rastrs robežām jāapgriež atbilsosais references slānis.

``` r
ref_rastrs100m=rast("D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03/ref_rastrs/LV100m_10km.tif")
st_crs(ref_rastrs100m) # LKS-92, protams.
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

``` r
ref_rastrs100m_LADam = terra::crop(ref_rastrs100m,LAD_rastrs)
rm(ref_rastrs100m)
plot(ref_rastrs100m_LADam)
```

![](Uzd03_Starka_files/figure-gfm/100m%20references%20rastrs-1.png)<!-- -->
\## funkciju ātruma salīdzināšana Tagad pārbaudīšu ar kuru no funkcijām
ātrāk iet rastra transformēšana no 10m rastra par rastru ar 100m malas
garumu. bet man nepatīk, ka ne aggregate(), ne project() funkcijās nav
iepsējams to piesaistīt references rastram, respektīvi, iespējams varētu
gadīties kādas nevēlamas nobīdes. Vismaz project() ir nedaudz labāks, jo
var piesaistīt koordinātu sistēmai, kas varētu šādu varbūtību mazināt.

``` r
library(microbenchmark)#install.packages("microbenchmark"), ja nepieciešams
resample_a=data.frame(microbenchmark(resample(LAD_rastrs2,ref_rastrs100m_LADam),times=5))
aggregate_a=data.frame(microbenchmark(aggregate(LAD_rastrs2,fact=10,fun=all),times=5))
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

``` r
project_a=data.frame(microbenchmark(project(LAD_rastrs2, "EPSG:3059"),times=5))
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

``` r
atrumi=rbind.data.frame(resample_a,aggregate_a,project_a)
atrumi$sek=atrumi$time/1e+09
plot(atrumi$sek)
```

![](Uzd03_Starka_files/figure-gfm/rastra%20izšķirtspējas%20pārveidošanas%20ātrumi%20ar%20dažādām%20funkcijām-1.png)<!-- -->

``` r
ggplot(atrumi, aes(expr, sek)) + geom_boxplot()+ 
      labs(x = "Formāts", y = "Ielasīšanas ātrums (sekundes)")+
  scale_x_discrete(labels=c('resample()', 'aggregate()', 'project()'))
```

![](Uzd03_Starka_files/figure-gfm/rastra%20izšķirtspējas%20pārveidošanas%20ātrumi%20ar%20dažādām%20funkcijām-2.png)<!-- -->
Secinu, ka ātrāk šo darbiņu paveic aggregate(), bet tālāk darbojos tikai
ar funkciju resample(), jo tā rada lielāku uzticību. Novācu liekos
objektus.

``` r
rm(resample_a,aggregate_a,project_a, atrumi)
```

Jāpaskatās vizuāli uz rastriem, kas rodas no šīm funkcijām.

``` r
resample_rastrs=resample(LAD_rastrs2,ref_rastrs100m_LADam)
aggregate_rastrs=aggregate(LAD_rastrs2,fact=10,fun=all)
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
project_rastrs=project(LAD_rastrs2, "EPSG:3059")
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
plot(resample_rastrs)#te nedaudz aizdomīgi ir tas, ka pikseļos nav diskrētas vērtības. Tas nozīmē, ka pēc noklusējuma pikseļu vērības laikam tiek saskaitītas.
```

![](Uzd03_Starka_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

``` r
plot(aggregate_rastrs)#te viss izskatās normāli
```

![](Uzd03_Starka_files/figure-gfm/unnamed-chunk-1-2.png)<!-- -->

``` r
plot(project_rastrs)#te arī viss normāli
```

![](Uzd03_Starka_files/figure-gfm/unnamed-chunk-1-3.png)<!-- --> \## LAD
platību īpatsvari Tagad ar tām pašām funkcijām jāpamēģina iegūt platību,
kurās ir lauka bloki, īpatsvaru. Izmantošu terra::freq, kas parāda
vērību biežumu. Aprēķinu īpatsvaru kā daļu no kopējā pikseļu skaita.
Rezultāts ir pikseļu skaita īpatsvars (nav faktiskā lauku platība ha).

``` r
library(tidyverse)
res_freq=freq(resample_rastrs)
agr_freq=freq(aggregate_rastrs)
proj_freq=freq(project_rastrs)

data.frame(rastraFunkcija=c('resample', 'aggregate', 'project'),
          Īpatsvars_proc=
            c(res_freq$count[2]/(res_freq$count[2] + res_freq$count[1])*100,
              agr_freq$count[2]/(agr_freq$count[2] + agr_freq$count[1])*100,
              proj_freq$count[2]/(proj_freq$count[2] + proj_freq$count[1])*100))
```

    ##   rastraFunkcija Īpatsvars_proc
    ## 1       resample      1.4025998
    ## 2      aggregate      0.6097325
    ## 3        project      1.4462159

redzams, ka vērtības būtiski atšķiras, aggregate() rezultējās mazāk
pikseļos. Man jau tāpat šī funkcija nepatika iepriekš aprakstīto iemeslu
dēļ. Tomēr arī starp resample() un project() ir atšķirības. Ņemot vērā,
ka šajā uzdevumā ir svarīga piesaiste tieši references rastram, tad
kopumā domāju, ka resample() funkcija ir ne tikai ātra, bet arī
visatbilstošākā.

Noņemtu liekos objektus

``` r
rm(aggregate_rastrs,project_rastrs,agr_freq,res_freq,proj_freq)
```

\#4. jauni 100m rastri ar īpatsvariem \## binārais rastrs Šoreiz
resample() funkcijai pievienoju argumentu method=“max”, lai rastrs
katram pikselim piešķirtu maksimālo vērtību, kādu satur jebkurš no to
veidojošajiem 10m pikseļiem. Dēļ rastra, no kā šis tiek būvēts, šī
maksimālā vērtība var būt tikai 1. Tādējādi iegūts rastrs, kas satur
bināri kodētas vērtības: Latvijā esošā šūnā ir lauku bloki (vērtība 1)
vai to nav (vērtība 0), tāpat kā iepriekš saglabājot šūnas vietām ārpus
Latvijas, jau ir izveidots (resample_rastrs). Varam to apskatīt vēlreiz.

``` r
resample_rastrs2=resample(LAD_rastrs2,ref_rastrs100m_LADam,method="max")
plot(resample_rastrs2)
```

![](Uzd03_Starka_files/figure-gfm/binārais%20rastrs-1.png)<!-- -->
Aprēķinu īpatsvaru (%). Tas tagad ir lielāks kā ieprieš, veidojot ar
resample() noklusējuma iestatījumiem. Abi varianti var būt noderīgi
kādās situācijās, bet šajā kontekstā šis būs pareizais.

``` r
bin_freq=freq(resample_rastrs2)
bin_ip=bin_freq$count[2]/(bin_freq$count[2] + bin_freq$count[1])*100
bin_ip
```

    ## [1] 2.370664

Eksportēju šo rastru.

``` r
terra::writeRaster(resample_rastrs2, filename="D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03/LAD_rastrs_100bin.tif", gdal=c("COMPRESS=DEFLATE", "TFW=YES"), overwrite=TRUE)
```

## Rastrs ar % īpatsvaru

Tagad, jāiegūst rastrs, kas satur informāciju par lauka bloku platību
noapaļotu līdz procentam un izteiktu kā veselu skaitli, tāpat kā
iepriekš saglabājot šūnas vietām ārpus Latvijas. Manuprāt, lai šo
izdarītu, ir jāmaina metodes arguments pie rasterize(), lai tas summē
mazo (10x10m) pikseļu skaitu lielajā. Vienā 10x10m šūnā ietilpst 100 gab
10x10m šūnas, kas nozīmē, ka ja saskaitīsim cik daudz mazajos pikseļos
ir vērtības “1”, iegūsim lielo pikseli, kas jau satur % veselos
skaitļos.

``` r
resample_rastrs3=resample(LAD_rastrs2,ref_rastrs100m_LADam,method="sum")
plot(resample_rastrs3)
```

![](Uzd03_Starka_files/figure-gfm/summētais%20rastrs-1.png)<!-- -->

Kā redzms, tiek attēlotas vērtības skalā no 0-100, ka tā arī ir, un par
to šajā vakarā liels prieks. Tagad īpatsvars jāaprēķina visai
apskatītajai platībai. Teorētiski, rezultējošajam īpatsvaram ir jābūt
mazākam kā bināriem 100m pikseļiem.

``` r
proc_freq=freq(resample_rastrs3)
proc_ip=sum(proc_freq$count[2:101])/sum(proc_freq$count[1:101])*100
proc_ip
```

    ## [1] 2.37045

Jā, kā redzams, tas ir nedaudz, nedaudz mazāks. Es gan biju sagaidījusi
lielākas atšķirības, tāpēc domāju, ka varbūt kāda cita pieeja ir
precīzāka, bet šobrīd nevaru iztēloties kāda. Varbūt neesmu sapratusi
jautājumu.

Saglabāšu arī šo rastru uz diska.

``` r
terra::writeRaster(resample_rastrs3, filename="D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03/LAD_rastrs_100sum.tif", gdal=c("COMPRESS=DEFLATE", "TFW=YES"),overwrite=TRUE)
```

un novācu lieko.

``` r
rm(bin_freq,proc_freq,LAD_rastrs,ref_rastrs100m_LADam, bin_ip,proc_ip)
```

## Salīdzinājums

Sāksim ar aizņemto diska vietu. Salīdzināšu trīs failus - 10m rastru,
100m bināro rastru un 100m summēto rastru. Nezinu kāpēc šis chunks
nereaģē uz palaišanu (erroru arī nav, fona nekādi procesi nenotiek)…
kodam vajadzētu būt pareizam

``` r
izmeri=data.frame(
  biti=file.size("LAD_rastrs.tif","LAD_rastrs_100bin.tif","LAD_rastrs_100sum.tif"),
  row.names=c("10m rastrs","100m binārs rastrs","100m summēts rastrs"))
izmeri$MB=izmeri$biti/1000000
print(izmeri)
```

    ##                        biti       MB
    ## 10m rastrs          2144224 2.144224
    ## 100m binārs rastrs   110600 0.110600
    ## 100m summēts rastrs  203938 0.203938

Rezultāti ir absolūti loģiski, jo augstāka izšķirstpēja, jo vairāk
vietas uz diska aizņem. Savukārt mazākā izšķirtspējā starp diviem
lielākais ir tas, kur pikseļi satur nevis binārus datus, bet dažādas
vērtības.

Tagad pēdējais, izmantotā kodējuma ietekme uz izmēru. Kodējumu iespējams
norādīt kā argumentu eksportējot rastru ar writeRaster() funkciju. Pēc
noklusējuma tiek izmantots kodējums ‘FLT4S’, tāpēc to neatkārtošu.
Pārējie varianti šeit -
<https://search.r-project.org/CRAN/refmans/raster/html/dataType.html>
Nemēģināšu no tiem ‘LOG1S’, jo tas jau faktiski tika izdarīts un
iepriekš apskatīts. Paskatīšos visus INTxx formātus un ‘FLT8S’. Visiem
trīs slāņiem to nedarīšu. Paņemšu pēc izmēra vidējo, 100m summēto
rastru.

``` r
writeRaster(resample_rastrs3, filename="D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03/LAD_rastrs_100sum_INT1S.tif",datatype = "INT1S", overwrite=TRUE)
writeRaster(resample_rastrs3, filename="D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03/LAD_rastrs_100sum_INT1U.tif",datatype = "INT1U", overwrite=TRUE)

writeRaster(resample_rastrs3, filename="D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03/LAD_rastrs_100sum_INT2S.tif",datatype = "INT2S", overwrite=TRUE)
writeRaster(resample_rastrs3, filename="D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03/LAD_rastrs_100sum_INT2U.tif",datatype = "INT2U", overwrite=TRUE)

writeRaster(resample_rastrs3, filename="D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03/LAD_rastrs_100sum_INT4S.tif",datatype = "INT4S", overwrite=TRUE)
writeRaster(resample_rastrs3, filename="D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03/LAD_rastrs_100sum_INT4U.tif",datatype = "INT4U", overwrite=TRUE)

writeRaster(resample_rastrs3, filename="D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03/LAD_rastrs_100sum_FLT8S.tif",datatype = "FLT8S", overwrite=TRUE)
```

Un izmēru salīdzinājums:

``` r
izmeri2=data.frame(
  biti=file.size("LAD_rastrs_100sum_INT1S.tif", 
              "LAD_rastrs_100sum_INT1U.tif",
              "LAD_rastrs_100sum_INT2S.tif",
              "LAD_rastrs_100sum_INT2U.tif",
              "LAD_rastrs_100sum_INT4S.tif",
              "LAD_rastrs_100sum_INT4U.tif",
              "LAD_rastrs_100sum_FLT8S.tif"),
  row.names=c("INT1S","INT1U","INT2S", "INT2U", "INT4S", "INT4U", "FLT8S"))
izmeri2$MB=izmeri2$biti/1000000
print(izmeri2)
```

    ##         biti       MB
    ## INT1S 119247 0.119247
    ## INT1U 119236 0.119236
    ## INT2S 209027 0.209027
    ## INT2U 197092 0.197092
    ## INT4S 384610 0.384610
    ## INT4U 329601 0.329601
    ## FLT8S 603048 0.603048

Secinājums: Visvairāk vietu uz diska no šeit apskatītajiem aizņem FLT8S
kodējums, bet vismazāk - INT1U. Kopumā tātad, kodējumam ir nozīme, ja
strādā ar lieliem datu apjomiem un ir nepieciešamība taupīt diska vietu.

Noņemu lieko

``` r
rm(izmeri2, resample_rastrs3)
```
