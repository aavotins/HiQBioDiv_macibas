Uzd03_Ozols
================
Janis Ozols
2025-01-09

Pirms uzdevumu veikšanas sagatavojam nepieciešamās pakotnes, atveram ar
*library()*, ja nepieciešams instalējam ar *install.packages()*

``` r
library(sf)
library(sfarrow)
library(concaveman)
library(ows4R)
library(httr)
library(tidyverse)
library(raster)
library(terra)
library(fasterize)
```

\#1. Lejupielādējiet Lauku atbalsta dienesta datus par teritoriju, kuru
aptver otrā uzdevuma Mežu valsts reģistra dati, no WFS, izmantojot šo
saiti
(<https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer>). 1.
sadalīsim uzdevumu soļos: 1) jāsagatavo otrā uzdevuma mežu valsts
reģistra dati 2) Izveidosim taisnleņķa četrstūri ap otrā uzdevuma datiem
un pārmainīsim koordinātes uz wgs 84 3) lejuplādēsim datus no lauku
atbalsta dienesta

1)  2.  uzdevumā mums ir sagatavoti meža reģistra apvienoti dati, kuros
        ģeometrijas ir salabotas un nav iztrūkstošo vērtību, kuru
        nosaucām par *shpcentrs*, pārveidojam šos datus par vienu
        ģeometriju un izveidojam no iegūtajām koordinātēm daudzstūri,
        apskatamies iegūto rezultātu

``` r
shpcentrs <- st_read("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd02\\shpcentrs.shp")
```

    ## Reading layer `shpcentrs' from data source 
    ##   `C:\Users\Ozols\Documents\HiQBioDiv_macibas\Uzd02\shpcentrs.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 505654 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 405229.7 ymin: 234180.7 xmax: 594509.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM

``` r
teritorija <- st_union(shpcentrs)
teritorija_robeza <- concaveman(st_coordinates(teritorija))
teritorija_poligons <- st_polygon(list(teritorija_robeza[, 1:2]))
teritorija_poligons <- st_sfc(teritorija_poligons)
```

Apskatamies iegūto teritoriju

``` r
plot(st_geometry(teritorija_poligons), main = "poligons")
```

![](Uzd03_Ozols_files/figure-gfm/attelo-1.png)<!-- --> 2) Izveidojam ap
iegūto teritoriju taisnleņķa četrstūri un pārveidojam koordinātes uz wgs
84 un pārliecinamies, ka objekts *cetrsturis* un objekts *teritorija
poligons* ir wgs 84 koordinātu sistēmā

``` r
st_crs(teritorija_poligons) <- st_crs(shpcentrs)
cetrsturis <- st_bbox(teritorija_poligons)
cetrsturis <- st_transform(cetrsturis,crs = 4326)
teritorija_poligons
```

    ## Geometry set for 1 feature 
    ## Geometry type: POLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 405229.7 ymin: 234180.7 xmax: 594509.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM

    ## POLYGON ((405230.9 310878.7, 405229.7 310894.4,...

``` r
cetrsturis
```

    ##     xmin     ymin     xmax     ymax 
    ## 22.42279 56.24303 25.57287 57.41234

3)  lejupielādējam datus no lauku atbalsta dienesta (LAD), apskatamies
    datus un pārveidojam datus lks92 koordināšu sistēmā, pārbaudam, ka
    dati ir lks92

``` r
wfs_lad <- "https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer"
client <- WFSClient$new(wfs_lad, serviceVersion = "2.0.0")
client$getFeatureTypes(pretty = TRUE)
```

    ##        name title
    ## 1 LAD:Lauki Lauki

``` r
url <- parse_url(wfs_lad)
url$query <- list(service = "WFS",
                  request = "GetFeature",
                  typename = "Lauki",
                  srsName = "EPSG:4326",
                  bbox = paste(cetrsturis, collapse = ",")
                  )
request <- build_url(url)
dati <- read_sf(request)
dati <- st_transform(dati, crs = 3059 )
dati
```

    ## Simple feature collection with 3000 features and 11 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 401769.7 ymin: 232909.4 xmax: 597087.7 ymax: 364412.9
    ## Projected CRS: LKS-92 / Latvia TM
    ## # A tibble: 3,000 × 12
    ##    gml_id    OBJECTID PERIOD_CODE PARCEL_ID PRODUCT_CODE AID_FORMS AREA_DECLARED
    ##  * <chr>        <int>       <int>     <int>        <int> <chr>             <dbl>
    ##  1 Lauki.42…  4257005        2024  17426915          710 MLS23              0.3 
    ##  2 Lauki.42…  4257006        2024  17426917          111 MLS23              0.72
    ##  3 Lauki.42…  4257318        2024  17466026          111 MLS23              2.84
    ##  4 Lauki.42…  4257333        2024  17466052          111 ISIP, EK…          1.49
    ##  5 Lauki.42…  4257334        2024  17466057          610 ISIP, EK…          9.06
    ##  6 Lauki.42…  4257339        2024  17465910          111 ISIP, EK…          8.19
    ##  7 Lauki.42…  4257340        2024  17465914          111 ISIP, EK…          0.57
    ##  8 Lauki.42…  4257348        2024  17465918          111 ISIP, EK…          1.24
    ##  9 Lauki.42…  4257349        2024  17465921          111 ISIP, EK…          0.64
    ## 10 Lauki.42…  4257350        2024  17465922          111 ISIP, EK…          0.19
    ## # ℹ 2,990 more rows
    ## # ℹ 5 more variables: DATA_CHANGED_DATE <chr>, PRODUCT_DESCRIPTION <chr>,
    ## #   SHAPE.AREA <dbl>, SHAPE.LEN <dbl>, SHAPE <MULTIPOLYGON [m]>

``` r
ggplot(dati) +geom_sf()
```

![](Uzd03_Ozols_files/figure-gfm/wfs-1.png)<!-- -->

``` r
centrs <- st_intersection(dati, teritorija_poligons)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout
    ## all geometries

``` r
plot(centrs["gml_id"])
```

![](Uzd03_Ozols_files/figure-gfm/wfs-2.png)<!-- -->

``` r
centrs
```

    ## Simple feature collection with 2349 features and 11 fields
    ## Geometry type: GEOMETRY
    ## Dimension:     XY
    ## Bounding box:  xmin: 408066.1 ymin: 234349 xmax: 576395.3 ymax: 360220.9
    ## Projected CRS: LKS-92 / Latvia TM
    ## # A tibble: 2,349 × 12
    ##    gml_id    OBJECTID PERIOD_CODE PARCEL_ID PRODUCT_CODE AID_FORMS AREA_DECLARED
    ##  * <chr>        <int>       <int>     <int>        <int> <chr>             <dbl>
    ##  1 Lauki.42…  4257005        2024  17426915          710 MLS23              0.3 
    ##  2 Lauki.42…  4257006        2024  17426917          111 MLS23              0.72
    ##  3 Lauki.42…  4257318        2024  17466026          111 MLS23              2.84
    ##  4 Lauki.42…  4257333        2024  17466052          111 ISIP, EK…          1.49
    ##  5 Lauki.42…  4257334        2024  17466057          610 ISIP, EK…          9.06
    ##  6 Lauki.42…  4257339        2024  17465910          111 ISIP, EK…          8.19
    ##  7 Lauki.42…  4257340        2024  17465914          111 ISIP, EK…          0.57
    ##  8 Lauki.42…  4257348        2024  17465918          111 ISIP, EK…          1.24
    ##  9 Lauki.42…  4257349        2024  17465921          111 ISIP, EK…          0.64
    ## 10 Lauki.42…  4257350        2024  17465922          111 ISIP, EK…          0.19
    ## # ℹ 2,339 more rows
    ## # ℹ 5 more variables: DATA_CHANGED_DATE <chr>, PRODUCT_DESCRIPTION <chr>,
    ## #   SHAPE.AREA <dbl>, SHAPE.LEN <dbl>, SHAPE <POLYGON [m]>

Novelkot datus no wfs servera, ieguva datus ar 3000 rindām, pēc
intersection ar centrālo virsmežniecības robežu, konkrētajā teritorijā
ietilpst 2349 lauki \#2. Atbilstoši referencei (10m izšķirtspējā),
rasterizējiet iepriekšējā punktā lejupielādētos vektordatus, sagatavojot
GeoTIFF slāni ar informāciju par vietām, kurās ir lauku bloki (kodētas
ar 1) vai tās atrodas Latvijā, bet tajās nav lauku bloki vai par tiem
nav informācijas (kodētas ar 0). Vietām ārpus Latvijas saglabājiet šūnas
bez vērtībām. Sadalīsim uzdevumu soļos: 1) sagatavot references failu 2)
rasterizēt lejupielādētos vektorfailus 3)sagatavot geotiff slāni ar
informāciju: vietas ar lauku blokiem = 1; atrodas Latvijā, bet nav lauku
bloki vai nav informācijas = 0;vietas ārpus Latvijas = NULL 1) Ielasam
references rastru

``` r
LV10m_10km <- raster("LV10m_10km.tif")
```

2)  rasterizējam lejupielādētos vektorfailus

``` r
Centrs_10m <- fasterize(centrs, LV10m_10km, background = 0)
Centrs_10m <- mask(Centrs_10m, LV10m_10km)
```

3)  apskatam rasterizētos vektorfailu datus, saglabājam Geotiff formātā

``` r
Centrs_10m
```

    ## class      : RasterLayer 
    ## dimensions : 28600, 47000, 1344200000  (nrow, ncol, ncell)
    ## resolution : 10, 10  (x, y)
    ## extent     : 302800, 772800, 162900, 448900  (xmin, xmax, ymin, ymax)
    ## crs        : +proj=tmerc +lat_0=0 +lon_0=24 +k=0.9996 +x_0=500000 +y_0=-6000000 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs 
    ## source     : r_tmp_2025-01-16_102651.896971_9812_04862.grd 
    ## names      : layer 
    ## values     : 0, 1  (min, max)

``` r
plot(Centrs_10m)
```

![](Uzd03_Ozols_files/figure-gfm/geotiff-1.png)<!-- -->

``` r
writeRaster(Centrs_10m,"C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\Centrs_10m.tif", format = "GTiff", overwrite = TRUE)
```

\#3. Izmēģiniet {terra} funkcijas resample(), aggregate() un project(),
lai no iepriekšējā punktā sagatavotā rastra izveidotu jaunu: 1) ar 100m
pikseļa malas garumu un atbilstību references slānim; 2)informāciju par
platības, kurā ir lauka bloki, īpatsvaru. 1)Resample funkcija -
pielīdzina rastrus vienam kvadrātu tīklam, aggregate - apvieno mazāka
rastra tīklu lielākā, sapludinot kopā tīkla šūnas, project - izmaina
references koordinātu sistēmu Aggregate izmantojam funkciju sum, lai
iegūtu skaitu ar 10m šūnām, kurās ir lauka bloki un fact = 10, lai
iegūtu režģi 100x100m

``` r
centrs_100m_agregate <- aggregate(Centrs_10m, fact=10, fun=sum)
centrs_100m_agregate
```

    ## class      : RasterLayer 
    ## dimensions : 2860, 4700, 13442000  (nrow, ncol, ncell)
    ## resolution : 100, 100  (x, y)
    ## extent     : 302800, 772800, 162900, 448900  (xmin, xmax, ymin, ymax)
    ## crs        : +proj=tmerc +lat_0=0 +lon_0=24 +k=0.9996 +x_0=500000 +y_0=-6000000 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs 
    ## source     : r_tmp_2025-01-16_103813.366835_9812_21846.grd 
    ## names      : layer 
    ## values     : 0, 100  (min, max)

``` r
plot(centrs_100m_agregate)
```

![](Uzd03_Ozols_files/figure-gfm/terra-1.png)<!-- --> Izmēģinam funckiju
resample, Centrs_10m pielāgojot references rastram, kā metodi izmantojam
sum, lai saskaitītu šūnas, kurās ir lauki

``` r
LV100m_10km <- raster("LV100m_10km.tif")
centrs_100m_resample <- resample(Centrs_10m,LV100m_10km, methode = "sum")
centrs_100m_resample
```

    ## class      : RasterLayer 
    ## dimensions : 2860, 4700, 13442000  (nrow, ncol, ncell)
    ## resolution : 100, 100  (x, y)
    ## extent     : 302800, 772800, 162900, 448900  (xmin, xmax, ymin, ymax)
    ## crs        : +proj=tmerc +lat_0=0 +lon_0=24 +k=0.9996 +x_0=500000 +y_0=-6000000 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs 
    ## source     : memory
    ## names      : layer 
    ## values     : 0, 1  (min, max)

``` r
plot(centrs_100m_resample)
```

![](Uzd03_Ozols_files/figure-gfm/resample-1.png)<!-- --> Īsti nesaprotu
kā ar funkciju project() var nomainīt šūnu izmēru, koordinātu sistēmu
skaidrs. Ātrāk strādāja funckija aggregate, bet ar funkciju resample
uzreiz izdevās iegūt lauku platības īpatsvaru pieņemot, ka 10m šūnās
lauki aizņēma visu šūnu teritoriju, tāpēc ar šo metodi katras 100m šūnas
lauka platības īpatsvaru nebija jāpārrēķina. 2)informāciju par platības,
kurā ir lauka bloki, īpatsvaru - pieņemot, ka lauka bloks aizņēma visu
10m šūnu, izmantojot *resample* un kā metodi norādot - sum, uzreiz
iegūšt īpatsvaru, cik no 100 10m šūnām satur lauka blokus Arī pārvērst
procentos nav nepieciešams, jo 100x100m šūna sastāv no 100 10m šūnām,
kas nozīmē, ka ar aggregate funkciju sum, saskaitot 10m šūnas ar lauku
blokiem uzreiz jau iegūst platības īpatsvaru, respektīvi, ja 100 metru
šūnā ir uzskaitītas 20 10m šūnas ar lauka blokiem, tad arī izsakot
īpatsvarā ir tie paši 20% no 100m šūnas platības \#4 Izmantojot
iepriekšējā punktā radīto 100m šūnas izmēra slāni, sagatavojiet divus
jaunus, tā, lai: 1. slānis satur informāciju par lauka bloku platību
noapaļotu līdz procentam un izteiktu kā veselu skaitli, tāpat kā
iepriekš saglabājot šūnas vietām ārpus Latvijas;

``` r
raster_100m_proc <- writeRaster(centrs_100m_agregate,"C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\raster_100m_proc.tif",overwrite = TRUE)
```

2.  slānis satur bināri kodētas vērtības: Latvijā esošā šūnā ir lauku
    bloki (vērtība 1) vai to nav (vērtība 0), tāpat kā iepriekš
    saglabājot šūnas vietām ārpus Latvijas.

``` r
centrs_100m_bin <- aggregate(Centrs_10m, fact=10, fun=max)
raster_100m_bin <- writeRaster(centrs_100m_bin,"C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\raster_100m_bin.tif",overwrite = TRUE)
```

Salīdziniet izveidotos slāņus, lai raksturotu: kā mainās faila izmērs uz
cietā diska, atkarībā no rastra šūnas izmēra;

``` r
Rastrs_10m <- file.info("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\Centrs_10m.tif")
Rastrs_100m <- file.info("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\raster_100m_bin.tif")
Rastrs_10m$size
```

    ## [1] 53235330

``` r
Rastrs_100m$size
```

    ## [1] 1381572

Rastrs ar lielākām šūnām aizņem daudz mazāk vieta - 1381572 (100m) :
53235330 (10m)

kā mainās faila izmērs uz cietā diska, atkarībā no tajā ietvertās
informācijas un tās kodējuma. Kāds kodējums (joslas tips) aizņem
vismazāko apjomu uz cietā diska, saglabājot informāciju?

writeRaster pieņem šādus joslas tipus, iekavās norādītas to min - max
vērtības: “INT1S” (-127 127), “INT1U” (0 255), “INT2U” (0 65,534),
“INT2S” (-32,767 32,767), “INT4U” (0 4,294,967,296), “INT4S”
(-2,147,483,647 2,147,483,647), “FLT4S” (-3.4e+38 3.4e+38), “FLT8S”
(-1.7e+308 1.7e+308) Salīdzināsim 2 vienkāršākos joslas tipus un 1
sarežģītāku, lai pārliecinātos, ka pārejot uz lielākiem joslu tipiem
palielinās faila izmērs: “INT1S”, “INT1U” un “INT2U”

``` r
Rastrs_INT1S <- writeRaster(centrs_100m_agregate,"C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\rastrs_INT1S.tif", datatype = "INT1S", overwrite = TRUE)
Rastrs_INT1S <-file.info("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\rastrs_INT1S.tif")
Rastrs_INT1S$size
```

    ## [1] 500490

``` r
Rastrs_INT1U <- writeRaster(centrs_100m_agregate,"C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\rastrs_INT1U.tif", datatype = "INT1U", overwrite = TRUE)
Rastrs_INT1U <-file.info("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\rastrs_INT1U.tif")
Rastrs_INT1U$size
```

    ## [1] 500478

``` r
Rastrs_INT2U <- writeRaster(centrs_100m_agregate,"C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\rastrs_INT2U.tif", datatype = "INT2U", overwrite = TRUE)
Rastrs_INT2U <-file.info("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\rastrs_INT2U.tif")
Rastrs_INT2U$size
```

    ## [1] 698798

Vismazāko vietu aizņem INT1U (500478), bet tas nepieņem negatīvas
vērtības un daļskaitļus, INT1S pieņem negatīvas vērtības, bet nepieņem
daļskaitļus un aizņem nedaudz vairāk vietas (500490), pārejot uz
lielākiem joslu tipiem kā INT2U, kas jau satur arī daļskaitļus, faila
izmērs palielinās (698798), kas norāda, ka lielākiem joslu tipiem faila
izmērs būs vēl lielāks
