Uzdevuma izpildei man bija nepieciešamas šādas pakotnes:

    library(curl)
    library(httr)
    library(sf)
    library(archive)
    library(tidyverse)
    library(ows4R)
    library(terra)
    library(httr)
    library(sfarrow)
    library(arrow)
    library(geoarrow)
    library(fasterize)
    library(microbenchmark)

\##Pirmais uzdevums. Datu ielasīšana

Vektordatu ielasīšana

    vurl <- "https://zenodo.org/api/records/14277114/files-archive"
    vektori <- "./vektori.zip"
    curl_download(vurl, destfile = vektori)

    vvieta <- "./vektori"
    invisible(archive_extract(vektori, dir =vvieta))

Rastra ielasīšana

    rurl <- "https://zenodo.org/api/records/14497070/files-archive"
    rastrs <- "./rastrs.zip"
    curl_download(rurl, destfile = rastrs)

    rvieta <- "./rastrs"
    archive_extract(rastrs, dir =rvieta)

Tā kā Uzd02 darbības veicu ar shapefile, ielasīšu to arī šajā uzdevumā,
bet pārveidošu par geoparquet, jo tas padara darbu efektīvāku.

    svieta <-"../Uzd02/combined_shapefile.shp"
    shapefile <-st_read(svieta, quiet = TRUE)
    geoparquet_path <- "centrs_geoparquet.parquet"
    st_write_parquet(shapefile, geoparquet_path)
    centrs<-read_parquet("centrs_geoparquet.parquet")

Savienojamies ar WFS serveri un apskatāmies, kas tur ir.

    wfs<- "https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer"
    jaunais_url <- parse_url(wfs)
    jaunais_url$query <- list(service = "wfs",
                      request = "GetCapabilities"
                      )
    velviens_url <- build_url(jaunais_url)
    velviens_url #Šo linku ieliekam pārlūkā, lai apskatītu, kas tur ir.

    ## [1] "https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer?service=wfs&request=GetCapabilities"

It kā redzu visu, ko vajag, bet nebūtu uzdevuma garā neizmatot iespēju
apskatīt informāciju, lietojot RStudio. Tāpat vajadzēs ielasīt datus no
WFS servera.

Aplūkojam pieejamos slāņus.

    wfs_client<-WFSClient$new(wfs, 
                                serviceVersion = "2.0.0")
    slani <- wfs_client$getFeatureTypes(pretty = TRUE)
    print(slani)

    ##        name title
    ## 1 LAD:Lauki Lauki

Redzam, ka ir pieejams slānis “LAD:Lauki”, taču lai ar to kaut ko varētu
darīt, ir jālejupielādē slānī ietvertie dati. Slānis satur teritoriju
par visu Latvijas teritoriju, kuru ielasīt varētu būt nelietderīgi,
tāpēc ielasīšu tikai to daļu, kas ir ap Latvijas centru. Lai to panāktu,
ir jāizveido bounding box.

    centrs2<-st_as_sf(centrs) #Kodam kaut kas nepatika ar objektu, tāpēc pārveidoju to par sf objektu.
    st_crs(centrs2) #Atklājās, ka šim nav norādīta koordināšu sistēma

    ## Coordinate Reference System: NA

Man neīpaši skaidru iemeslu dēļ, šis objekts nesatur koordinātas.
Gribēju piešķirt WGS84 koordinātas uzreiz, bet tādā gadījumā nosakot
jauno bounding box, koordinātas nelīdzinājās URL redzamajam formātam.
Šis bija vienīgais veids, kā man sanāca iegūt koordinātas līdzīgā
formātā. Novērtētu skaidrojumu.

    centrs2<-st_set_crs(centrs2, 3059)
    centrs3<-st_transform(centrs2, crs = 4326) 
    centra_robezas <- st_bbox(centrs3)
    print(centra_robezas)

    ##     xmin     ymin     xmax     ymax 
    ## 22.44270 56.25237 25.55048 57.41006

Visbeidzot- lejupielādējam slāņa datus, kas atrodas tikai jaunajā bbox.

    wfs_url2 <- parse_url(wfs)
    wfs_url2$query <- list(service = "WFS",
                      request = "GetFeature",
                      typename = "LAD:Lauki",
                      bbox = "22.44270,56.25237,25.55048,57.41006")
    request2 <- build_url(wfs_url2)
    gml_vieta <- ("./centra.gml")
    invisible(GET(url = request2, 
        write_disk(gml_vieta, overwrite = TRUE)))
    centra_gml <- read_sf(gml_vieta, quiet = TRUE)

    #Lai netiku nevajadzīgi aizņemta vieta atmiņā, izdzēšu nevajadzīgos objektus. 


    rm(vurl, vektori, vvieta, rurl, rastrs, rvieta, svieta, shapefile, geoparquet_path, centrs, centrs2, centrs3, centra_robezas, wfs, jaunais_url, velviens_url, wfs_client, slani, wfs_url2, request2, gml_vieta)

## Otrais uzdevums.

Vektordatu rasterizēšana

Ielasu GML datus un pārtaisu par sf.

    gml_data <- vect("./centra.gml")
    gml_data<-st_as_sf(gml_data)

Pārveidoju GML datus par parketu.

    lauki_vieta <- ("./lauki.parquet")
    st_write_parquet(gml_data, lauki_vieta)
    lauki <-st_read_parquet(lauki_vieta)

Ielasu rastra datus.

    rastrs1 <- rast("./rastrs/LV10m_10km.tif")

Lai varētu rasterizēt, koordinātu sistēmām ir jābūt vienādām.

    st_crs(lauki)==st_crs(rastrs1)

    ## [1] TRUE

Par laimi tās ir vienādas, tāpēc varu veikt rasterizāciju, taču
ielasītais rastrs ir Latvijas lielumā, tādēļ es apgriežu to, lai tas
būtu centra lielumā. Lai vaŗetu apgriezt malas, tiem vajadzētu būt ar
saderīgām klasēm, piemēram, sf un raster. Pārbaudu un griežu.

    class(lauki)

    ## [1] "sf"         "data.frame"

    class(rastrs1)

    ## [1] "SpatRaster"
    ## attr(,"package")
    ## [1] "terra"

    rastrins<-terra::crop(rastrs1, lauki)

    ## |---------|---------|---------|---------|=========================================                                          

    invisible(fasterize::raster(rastrins))

Rasterizēju datus, norādot, ka lauki ir kodējami ar 1, bet pārējās
vietas, neskaitot ārzemes, ar 0.

    lauki$lauks=1
    lauki <-lauki %>% select(lauks)
    rastrins<-raster(rastrins)
    lauki10m <- fasterize(lauki, rastrins, field="lauks", background=0)
    lauki10m <- mask(lauki10m, rastrins)

Ierakstu rastru kā GeoTIFF failu.

    lauki10m_vieta <- "./lauki10m.tif"
    writeRaster(lauki10m, filename=lauki10m_vieta, format = "GTiff", overwrite = TRUE)
    rm(centra_gml, gml_data, lauki, rastrs1, rastrins, lauki10m)

## Trešais uzdevums.

Pakotnes “terra” lietojums. Šajā uzdevumā daudzas funkcijas veidoja
garus ziņojumus un brīdinājumus, kuri, jā, satur svarīgu informāciju,
bet arī padara darbu nelasāmu. Tāpēc es izmantoju “message=FALSE,
warning=FALSE” argumentus, taču aprakstīju pamanīto.

Iepriekšējā punkta/uzdevuma veikuma ielasīšana.

    rastradati<-rast("lauki10m.tif")
    print(rastradati)

    ## class       : SpatRaster 
    ## dimensions  : 13061, 19367, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 403200, 596870, 233800, 364410  (xmin, xmax, ymin, ymax)
    ## coord. ref. : +proj=tmerc +lat_0=0 +lon_0=24 +k=0.9996 +x_0=500000 +y_0=-6000000 +ellps=GRS80 +units=m +no_defs 
    ## source      : lauki10m.tif 
    ## name        : lauki10m 
    ## min value   :        0 
    ## max value   :        1

    st_crs(rastradati)

    ## Coordinate Reference System:
    ##   User input: PROJCRS["unknown",
    ##     BASEGEOGCRS["unknown",
    ##         DATUM["Unknown based on GRS 1980 ellipsoid using towgs84=0,0,0,0,0,0,0",
    ##             ELLIPSOID["GRS 1980",6378137,298.257222101004,
    ##                 LENGTHUNIT["metre",1],
    ##                 ID["EPSG",7019]]],
    ##         PRIMEM["Greenwich",0,
    ##             ANGLEUNIT["degree",0.0174532925199433,
    ##                 ID["EPSG",9122]]]],
    ##     CONVERSION["Transverse Mercator",
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
    ##         AXIS["easting",east,
    ##             ORDER[1],
    ##             LENGTHUNIT["metre",1,
    ##                 ID["EPSG",9001]]],
    ##         AXIS["northing",north,
    ##             ORDER[2],
    ##             LENGTHUNIT["metre",1,
    ##                 ID["EPSG",9001]]]] 
    ##   wkt:
    ## PROJCRS["unknown",
    ##     BASEGEOGCRS["unknown",
    ##         DATUM["Unknown based on GRS 1980 ellipsoid using towgs84=0,0,0,0,0,0,0",
    ##             ELLIPSOID["GRS 1980",6378137,298.257222101004,
    ##                 LENGTHUNIT["metre",1],
    ##                 ID["EPSG",7019]]],
    ##         PRIMEM["Greenwich",0,
    ##             ANGLEUNIT["degree",0.0174532925199433,
    ##                 ID["EPSG",9122]]]],
    ##     CONVERSION["Transverse Mercator",
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
    ##         AXIS["easting",east,
    ##             ORDER[1],
    ##             LENGTHUNIT["metre",1,
    ##                 ID["EPSG",9001]]],
    ##         AXIS["northing",north,
    ##             ORDER[2],
    ##             LENGTHUNIT["metre",1,
    ##                 ID["EPSG",9001]]]]

Pamanīju, ka rastram nav norādīta koordinātu sistēma.

    crs(rastradati) <- "EPSG:3059"
    st_crs(rastradati) #tagad ir koordinātu sistēma!

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

Tātad- man ir jāpārveido 10m izšķirtspējas rastrs par 100m izšķirtspējas
rastru. Tā kā tagad rastra šūnas ir lielākas, tas nozīmē, ka katrā
rastrā ir 10x10 10m šūnas, kurā katrā ir informācija par to, vai tās
sastāvā ir lauki vai nav. Nosacījumos teikts, ka man ir jāparāda
īpatsvars laukiem katrā šūnā.

Rastra pārveide no 10m uz 100m pikseļu izmēru izmantojot terra funkciju
“aggregate”. Tiek izmantota funkcija mean, jo tā atspoguļo īpatsvaru.
Cik es noprotu, tas strādā šādi- ja no 100 pikseļiem 10 satur laukus,
tad vērtība, ko parādīs šūnā, tiek aprēķināta (1*10)+(0*90)/100-
respektīvi tieši tā kā tiktu aprēķināts īpatsvars.

    a_100m <- aggregate(rastradati, fact=10, fun="mean") 

    ## |---------|---------|---------|---------|=========================================                                          

    print(a_100m) #paskatos, vai viss ir ok.

    ## class       : SpatRaster 
    ## dimensions  : 1307, 1937, 1  (nrow, ncol, nlyr)
    ## resolution  : 100, 100  (x, y)
    ## extent      : 403200, 596900, 233710, 364410  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source(s)   : memory
    ## name        : lauki10m 
    ## min value   :        0 
    ## max value   :        1

    plot(a_100m) #papriecājos par izskatu

![](Uzd03_Riters_files/figure-markdown_strict/unnamed-chunk-19-1.png)

Resample() un project() funkcijām nepieciešams references rastrs. Zinu,
ka iepriekšējā punktā gūtais rastrs atspoguļo datus tikai par centru,
tādēļ tas ir mazāks nekā jau pieejamais 100m rastrs, kas parāda visu
Latviju, tādēļ apgriežu lielo rastru.

    simts<- crop((rast("./rastrs/LV100m_10km.tif")), rastradati)
    st_crs(simts) #Arī LKS92 sistēma- tātad viss ok!

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

Rastra pārveide no 10m uz 100m pikseļu izmēru izmantojot terra funkciju
“resample”. Method=“average” lietojums skaidrojams tā pat kā argumenta
mean lietojums funkcijā “aggregate”. It kā kaut ko līdzīgu varētu panākt
ar method=sum, taču tā aizņem ļoti daudz laika salīdzinot ar šo un
rezultāts nebūtu daļskaitlis, tātad tas nebūtu īsti īpatsvars.

    r_100m<-resample(rastradati, simts, method="average")
    print(r_100m) #paskatos, vai viss ir ok.

    ## class       : SpatRaster 
    ## dimensions  : 1306, 1937, 1  (nrow, ncol, nlyr)
    ## resolution  : 100, 100  (x, y)
    ## extent      : 403200, 596900, 233800, 364400  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source(s)   : memory
    ## varname     : LV100m_10km 
    ## name        : lauki10m 
    ## min value   :        0 
    ## max value   :        1

    plot(r_100m) #papriecājos par izskatu

![](Uzd03_Riters_files/figure-markdown_strict/unnamed-chunk-21-1.png)

Rastra pārveide no 10m uz 100m pikseļu izmēru izmantojot terra funkciju
“project”. Ar average metodi šeit tas pats stāsts, kas iepriekš.

    p_100m<-project(rastradati, simts, method="average")
    print(p_100m) #paskatos, vai viss ir ok.

    ## class       : SpatRaster 
    ## dimensions  : 1306, 1937, 1  (nrow, ncol, nlyr)
    ## resolution  : 100, 100  (x, y)
    ## extent      : 403200, 596900, 233800, 364400  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source(s)   : memory
    ## varname     : LV100m_10km 
    ## name        : lauki10m 
    ## min value   :        0 
    ## max value   :        1

    plot(p_100m) #papriecājos par izskatu

![](Uzd03_Riters_files/figure-markdown_strict/unnamed-chunk-22-1.png)

Laiku izvērtēšana

    aggregate_laiks<-microbenchmark(aggregate(rastradati, fact=10, fun="mean"), times=10)

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

    print(aggregate_laiks, unit="s")

    ## Unit: seconds
    ##                                            expr      min       lq     mean
    ##  aggregate(rastradati, fact = 10, fun = "mean") 2.929123 3.053326 4.405093
    ##    median       uq      max neval
    ##  4.243885 5.737274 5.999736    10

Aggregate pati par sevi ir ātra, taču tajai ir īpašība, ka tā nobīda
kooridnātas, tādēļ šī funkcija īsti nav derīga pati par sevi un ir
jālieto tandēmā ar project().

    projaggregate_laiks<-microbenchmark(project(aggregate(rastradati, fact=10, fun="mean"),simts), times=10)

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

    print(projaggregate_laiks, unit="s")

    ## Unit: seconds
    ##                                                            expr      min
    ##  project(aggregate(rastradati, fact = 10, fun = "mean"), simts) 3.185169
    ##        lq     mean   median       uq      max neval
    ##  3.259388 3.360825 3.305448 3.413742 3.757109    10

    resample_laiks<-microbenchmark(resample(rastradati, simts, method="average"), times=10)
    print(resample_laiks, unit="s")

    ## Unit: seconds
    ##                                             expr      min       lq    mean
    ##  resample(rastradati, simts, method = "average") 3.948238 4.454003 4.91903
    ##    median       uq      max neval
    ##  4.632337 4.691655 8.611335    10

    project_laiks<-microbenchmark(project(rastradati, simts, method="average"), times=10)
    print(project_laiks, unit="s")

    ## Unit: seconds
    ##                                            expr      min       lq     mean
    ##  project(rastradati, simts, method = "average") 8.489797 8.599701 8.656356
    ##    median       uq      max neval
    ##  8.642704 8.674605 8.992143    10

Funkcijām ir līdzīgs laiks, kas atšķiras sekunžu ietvaros. Vispār, manā
datorā visātrākā funkcija mainījās un katru reizi ir citādāka. Es
pieturos pie funkcijas resample, jo tā, manuprāt, liekas drošākā.

## Ceturtais uzdevums.

Sagatavojam slāni, kurā būs redzams īpatsvars procentos, kas ir
noapaļoti.

    r_100m_procenti <- round(r_100m*100)#nezinu vai tiešām ir tik viegli, bet šķiet, ka ir. To pašu var panākt ar resample(method=sum).
    plot(r_100m_procenti) 

![](Uzd03_Riters_files/figure-markdown_strict/unnamed-chunk-27-1.png)

Sagatavojam slāni, kurā būs redzams vai šūnā ir vai nav lauki. Izmantoju
metodi “max”, jo tā atspoguļo jebkādu klātesamību kā 1, bet neesamību kā
0.

    r_100m_binari<-resample(rastradati, simts, method="max")
    plot(r_100m_binari)

![](Uzd03_Riters_files/figure-markdown_strict/unnamed-chunk-28-1.png)

Lai zinātu cik daudz vietas aizņem slāņi, tie arī ir jāsaglabā un
jāielasa.

    r_100m_procenti_vieta <- "./r_100m_procenti.tif"
    writeRaster(r_100m_procenti, filename=r_100m_procenti_vieta, overwrite = TRUE)
    procenti<- raster(r_100m_procenti_vieta)

    r_100m_binari_vieta <- "./r_100m_binari.tif"
    writeRaster(r_100m_binari, filename=r_100m_binari_vieta, overwrite = TRUE)
    binari<- raster(r_100m_binari_vieta)

Tātad- ja vēlamies salīdzināt, cik daudz vietas aizņem slāņi, atkarībā
no izšķirtspējas, tad var salīdzināt centra 10m rastru un 100m bināro
rastru, jo principā tiem ir ļoti līdzīga informācija.

    file.info(lauki10m_vieta)$size/1024

    ## [1] 12789.96

    file.info(r_100m_binari_vieta)$size/1024

    ## [1] 342.2178

Redzams, ka 100m binārais rastrs ir daudz reizes mazāks nekā 10m
izšķirtspējas rastrs.

    writeRaster(r_100m_binari, "./r_100m_binari_INT1S.tif", overwrite = TRUE, datatype = "INT1S")
    writeRaster(r_100m_binari, "./r_100m_binari_INT1U.tif", overwrite = TRUE, datatype = "INT1U")
    writeRaster(r_100m_binari, "./r_100m_binari_INT2S.tif", overwrite = TRUE, datatype = "INT2S")
    writeRaster(r_100m_binari, "./r_100m_binari_INT2U.tif", overwrite = TRUE, datatype = "INT2U")
    writeRaster(r_100m_binari, "./r_100m_binari_FLT4S.tif", overwrite = TRUE, datatype = "FLT4S")

    INT1S <- file.info("./r_100m_binari_INT1S.tif")$size / (1024)
    INT1U <- file.info("./r_100m_binari_INT1U.tif")$size / (1024)
    INT2S <- file.info("./r_100m_binari_INT2S.tif")$size / (1024)
    INT2U <- file.info("./r_100m_binari_INT2U.tif")$size / (1024)
    FLT4S <- file.info("./r_100m_binari_FLT4S.tif")$size / (1024)

    kodejumi <- data.frame(
      kodejums = c("INT1S", "INT1U", "INT2S", "INT2U", "FLT4S"),
      kb = c(INT1S, INT1U, INT2S, INT2U, FLT4S))
    print(kodejumi)

    ##   kodejums        kb
    ## 1    INT1S  82.15918
    ## 2    INT1U  82.14746
    ## 3    INT2S 159.52051
    ## 4    INT2U 148.09766
    ## 5    FLT4S 342.21777

Redzams, ka vismazāk vietas aizņem INT1S, bet visvairāk FLT4S. Tas
skaidrojams ar to, ka kodējuma veidiem atšķiras datu uzglabāšanas veids.
Daži ir spējīgi uzglabāt informāciju daļskaitļos, bet daži tikai veselos
skaitļos.
