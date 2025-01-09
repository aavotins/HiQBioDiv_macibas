Uzd02_Starka
================
Rūta Starka
2025-01-01

## Sagatavošanās

Lejupielādēju un atpakoju Meža valsts reģistra Centra virsmežniecības
datus. P.S. Netiku galā ar atzipošanas kļūdu “Error:
archive_extract.cpp:166 archive_read_next_header(): unknown libarchive
error”. Folderī izveidojas mape ar norādīto nosaukumu, bet tajā netiek
ievietoti faili no arhīva. Kādu laiku ar to noņēmos, bet pārāk ilgi,
tāpēc nokāru galvu, izdarīju atzipošanu manuāli folderī un gāju tālāk.
Atstāju šeit kodu, cerībā uz padomu.

``` r
setwd("D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd02")
download.file(url = "https://data.gov.lv/dati/lv/dataset/40014c0a-90f5-42be-afb2-fe3c4b8adf92/resource/392dfb67-eeeb-43c2-b082-35f9cf986128/download/centra.7z", destfile ="VMD_MVR_centra.zip")

library(archive)
#archive_extract("VMD_MVR_centra.zip", dir = "VMD_MVR_centra")

list.files("VMD_MVR_centra")
```

    ##  [1] "nodala2651.cpg"     "nodala2651.dbf"     "nodala2651.prj"    
    ##  [4] "nodala2651.sbn"     "nodala2651.sbx"     "nodala2651.shp"    
    ##  [7] "nodala2651.shp.xml" "nodala2651.shx"     "nodala2652.cpg"    
    ## [10] "nodala2652.dbf"     "nodala2652.prj"     "nodala2652.sbn"    
    ## [13] "nodala2652.sbx"     "nodala2652.shp"     "nodala2652.shp.xml"
    ## [16] "nodala2652.shx"     "nodala2653.cpg"     "nodala2653.dbf"    
    ## [19] "nodala2653.prj"     "nodala2653.sbn"     "nodala2653.sbx"    
    ## [22] "nodala2653.shp"     "nodala2653.shp.xml" "nodala2653.shx"    
    ## [25] "nodala2654.cpg"     "nodala2654.dbf"     "nodala2654.prj"    
    ## [28] "nodala2654.sbn"     "nodala2654.sbx"     "nodala2654.shp"    
    ## [31] "nodala2654.shp.xml" "nodala2654.shx"     "nodala2655.cpg"    
    ## [34] "nodala2655.dbf"     "nodala2655.prj"     "nodala2655.sbn"    
    ## [37] "nodala2655.sbx"     "nodala2655.shp"     "nodala2655.shp.xml"
    ## [40] "nodala2655.shx"

\#1. uzdevums Uzdevuma pirmās daļas - datu apjoma salīdzināšanai
nepieciešams atlasīt 2651. nodaļas datus.

``` r
library(sf) # install.packages("sf"), ja tā jau nav instalēta
ERSI_shp = st_read(dsn="VMD_MVR_centra",layer="nodala2651")
```

    ## Reading layer `nodala2651' from data source 
    ##   `D:\Users\RutaStarka\Desktop\Git local Ruuta\HiQBioDiv_macibas\Uzd02\VMD_MVR_centra' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM

Tagad no šī sheipfaila jāizveido Geopackage un geoparquet formāti. Tam
nepieciešamas “sfarrow” pakotnes funkcijas.

``` r
library(sfarrow) # install.packages("sfarrow", dependencies=TRUE), ja nepieciešams

st_write_parquet(ERSI_shp, dsn="nodala2651.parquet")
```

    ## Warning: This is an initial implementation of Parquet/Feather file support and
    ## geo metadata. This is tracking version 0.1.0 of the metadata
    ## (https://github.com/geopandas/geo-arrow-spec). This metadata
    ## specification may change and does not yet make stability promises.  We
    ## do not yet recommend using this in a production setting unless you are
    ## able to rewrite your Parquet/Feather files.

``` r
st_write(ERSI_shp, dsn="nodala2651.gpkg")
```

    ## Writing layer `nodala2651' to data source `nodala2651.gpkg' using driver `GPKG'
    ## Writing 91369 features with 70 fields and geometry type Multi Polygon.

\##1.1. aizņemtā diska vieta Tagad varam salīdzināt šo trīs formātu
izmērus. Tā kā funkcija kā rezultātu dod izmēru bitos, papildus
izveidoju kolonnu “MB”, kur aprēķināts izmērs megabitos. Secinu, ka
vismazāk vietas aizņem geoparquet formāts (22.1 MB). Pēc rezultātu
apskates ar funkciju rm() noņemu objektu “izmeri” no R vides.

``` r
izmeri=data.frame(biti=file.size("nodala2651.parquet","nodala2651.gpkg","D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd02/VMD_MVR_centra/nodala2651.shp"),
            row.names=c("geoparquet","GeoPackage","Shape"))
izmeri$MB=izmeri$biti/1000000
print(izmeri)
```

    ##                biti       MB
    ## geoparquet 22109041 22.10904
    ## GeoPackage 57794560 57.79456
    ## Shape      27092404 27.09240

``` r
rm(izmeri)
```

\##1.2. ielasīšanas ātrums Tagad jāsalīdzina šo trīs formātu ielasīšanas
ātrums. Lai uzzinātu ekspresijas ātrumus, var izmantot “mikrobenchmark”
pakotnes funkciju mikrobenchmark().Rezultātu tālākai salīdzināšanai, tos
sākotnēji veidoju kā atsevišķus objektus, un pēc tam apkopoju datu
tabulā “atrumi”. Ņemot vērā, ka funkcijas rezultāts ir ielasīšanas laiks
nanosekundēs, pārrēķinu to uz sekundēm un ievietoju rezultātu jaunā
kolonnā, uzskatāmākai tālākai salīdzināšanai.

``` r
library(microbenchmark)#install.packages("microbenchmark"), ja nepieciešams
par=data.frame(microbenchmark(st_read_parquet("nodala2651.parquet"),times=10))
gpk=data.frame(microbenchmark(read_sf("nodala2651.gpkg"),times=10))
shp=data.frame(microbenchmark(st_read(dsn="VMD_MVR_centra",layer="nodala2651"),times=10))
```

    ## Reading layer `nodala2651' from data source 
    ##   `D:\Users\RutaStarka\Desktop\Git local Ruuta\HiQBioDiv_macibas\Uzd02\VMD_MVR_centra' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `D:\Users\RutaStarka\Desktop\Git local Ruuta\HiQBioDiv_macibas\Uzd02\VMD_MVR_centra' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `D:\Users\RutaStarka\Desktop\Git local Ruuta\HiQBioDiv_macibas\Uzd02\VMD_MVR_centra' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `D:\Users\RutaStarka\Desktop\Git local Ruuta\HiQBioDiv_macibas\Uzd02\VMD_MVR_centra' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `D:\Users\RutaStarka\Desktop\Git local Ruuta\HiQBioDiv_macibas\Uzd02\VMD_MVR_centra' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `D:\Users\RutaStarka\Desktop\Git local Ruuta\HiQBioDiv_macibas\Uzd02\VMD_MVR_centra' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `D:\Users\RutaStarka\Desktop\Git local Ruuta\HiQBioDiv_macibas\Uzd02\VMD_MVR_centra' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `D:\Users\RutaStarka\Desktop\Git local Ruuta\HiQBioDiv_macibas\Uzd02\VMD_MVR_centra' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `D:\Users\RutaStarka\Desktop\Git local Ruuta\HiQBioDiv_macibas\Uzd02\VMD_MVR_centra' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `D:\Users\RutaStarka\Desktop\Git local Ruuta\HiQBioDiv_macibas\Uzd02\VMD_MVR_centra' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM

``` r
atrumi=rbind.data.frame(par,gpk,shp)
rm(gpk,par,shp)
atrumi$sek=atrumi$time/1e+09
```

Lai ekspresijas ātrumus salīdzinātu, pēc priekšnosacījumu pārbaudes
veicu šo trīs paraugkopu salīdzināšanu ar parametriskām metodēm. Secinu,
ka visu šo trīs datu formātu ielasīšanas ātrumi savstarpēji būtiski
atšķiras. Geoparquet formāts ielasās būtiski ātrāk.

``` r
tapply(atrumi$sek, atrumi$expr,shapiro.test)
```

    ## $`st_read_parquet("nodala2651.parquet")`
    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  X[[i]]
    ## W = 0.98743, p-value = 0.9926
    ## 
    ## 
    ## $`read_sf("nodala2651.gpkg")`
    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  X[[i]]
    ## W = 0.88462, p-value = 0.1474
    ## 
    ## 
    ## $`st_read(dsn = "VMD_MVR_centra", layer = "nodala2651")`
    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  X[[i]]
    ## W = 0.9369, p-value = 0.519

``` r
s=aov(sek~expr, data=atrumi)
summary(s)
```

    ##             Df Sum Sq Mean Sq F value Pr(>F)    
    ## expr         2 288.08  144.04    7346 <2e-16 ***
    ## Residuals   27   0.53    0.02                   
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

``` r
TukeyHSD(s)
```

    ##   Tukey multiple comparisons of means
    ##     95% family-wise confidence level
    ## 
    ## Fit: aov(formula = sek ~ expr, data = atrumi)
    ## 
    ## $expr
    ##                                                                                                 diff
    ## read_sf("nodala2651.gpkg")-st_read_parquet("nodala2651.parquet")                            3.367774
    ## st_read(dsn = "VMD_MVR_centra", layer = "nodala2651")-st_read_parquet("nodala2651.parquet") 7.575016
    ## st_read(dsn = "VMD_MVR_centra", layer = "nodala2651")-read_sf("nodala2651.gpkg")            4.207242
    ##                                                                                                  lwr
    ## read_sf("nodala2651.gpkg")-st_read_parquet("nodala2651.parquet")                            3.212511
    ## st_read(dsn = "VMD_MVR_centra", layer = "nodala2651")-st_read_parquet("nodala2651.parquet") 7.419753
    ## st_read(dsn = "VMD_MVR_centra", layer = "nodala2651")-read_sf("nodala2651.gpkg")            4.051979
    ##                                                                                                  upr
    ## read_sf("nodala2651.gpkg")-st_read_parquet("nodala2651.parquet")                            3.523038
    ## st_read(dsn = "VMD_MVR_centra", layer = "nodala2651")-st_read_parquet("nodala2651.parquet") 7.730280
    ## st_read(dsn = "VMD_MVR_centra", layer = "nodala2651")-read_sf("nodala2651.gpkg")            4.362505
    ##                                                                                             p adj
    ## read_sf("nodala2651.gpkg")-st_read_parquet("nodala2651.parquet")                                0
    ## st_read(dsn = "VMD_MVR_centra", layer = "nodala2651")-st_read_parquet("nodala2651.parquet")     0
    ## st_read(dsn = "VMD_MVR_centra", layer = "nodala2651")-read_sf("nodala2651.gpkg")                0

``` r
rm(s)
```

Vizuāls ielasīšanas ātruma salīdzinājums:
![](Uzd02_Starka_files/figure-gfm/Ielasīšanas%20ātrums%20vizuāli-1.png)<!-- -->

Atbrīvojos no liekajiem objektiem R vidē:

``` r
rm(atrumi,ERSI_shp)
```

\#2. Centra virzmežniecību nodaļu datu apvienošana vienā slānī Ņemot
vērā, ka geoparquet formāts aizņem vismazāk vietas uz diska, un ar to
dabības ir veicamas ātrāk, turpmāk darbošos ar šo formātu. Sākumā
pārbaudu, cik dažādi nodaļu nosaukumi ir centra virsmežniecībā, atlasot
failus ar .shp formātu. Noskaidroju, ka tādi ir pieci. Šo informāciju
izmantoju, lai uzrakstītu kurus slāņus nepieciešams ielasīt. \##2.1.
faila izveide Sākotnēji ielasu atsevišķus shape slāņus, tad apvienoju
tos vienā shape “centrs_SHP” izmantojot (rbind). Tad, tāpat kā iepriekš,
pārveidoju no shape uz geopackage formātu, saglabājot rezultātu
“centrs_kopa.parquet” darba direktorijā. Atbrīvojos no liekajiem
objektiem un vērtībām R vidē, un visbeidzot zem nosaukuma “centrs_kopa”
ielasu jaunizveidoto apvienoto Centra virsmežniecības ģeosaini.

``` r
list.files("VMD_MVR_centra",(pattern = "\\.shp$"))
```

    ## [1] "nodala2651.shp" "nodala2652.shp" "nodala2653.shp" "nodala2654.shp"
    ## [5] "nodala2655.shp"

``` r
n2651_shp = st_read(dsn="VMD_MVR_centra",layer="nodala2651")
```

    ## Reading layer `nodala2651' from data source 
    ##   `D:\Users\RutaStarka\Desktop\Git local Ruuta\HiQBioDiv_macibas\Uzd02\VMD_MVR_centra' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM

``` r
n2652_shp = st_read(dsn="VMD_MVR_centra",layer="nodala2652")
```

    ## Reading layer `nodala2652' from data source 
    ##   `D:\Users\RutaStarka\Desktop\Git local Ruuta\HiQBioDiv_macibas\Uzd02\VMD_MVR_centra' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 78469 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 420613.3 ymin: 241544.3 xmax: 507491.5 ymax: 305873.1
    ## Projected CRS: LKS-92 / Latvia TM

``` r
n2653_shp = st_read(dsn="VMD_MVR_centra",layer="nodala2653")
```

    ## Reading layer `nodala2653' from data source 
    ##   `D:\Users\RutaStarka\Desktop\Git local Ruuta\HiQBioDiv_macibas\Uzd02\VMD_MVR_centra' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 112400 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 496133.2 ymin: 299013 xmax: 570134.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM

``` r
n2654_shp = st_read(dsn="VMD_MVR_centra",layer="nodala2654")
```

    ## Reading layer `nodala2654' from data source 
    ##   `D:\Users\RutaStarka\Desktop\Git local Ruuta\HiQBioDiv_macibas\Uzd02\VMD_MVR_centra' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 113968 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 505799.1 ymin: 266426.8 xmax: 594509.3 ymax: 318075
    ## Projected CRS: LKS-92 / Latvia TM

``` r
n2655_shp = st_read(dsn="VMD_MVR_centra",layer="nodala2655")
```

    ## Reading layer `nodala2655' from data source 
    ##   `D:\Users\RutaStarka\Desktop\Git local Ruuta\HiQBioDiv_macibas\Uzd02\VMD_MVR_centra' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 109454 features and 70 fields (with 6 geometries empty)
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 405229.7 ymin: 283087.1 xmax: 504232.4 ymax: 352909
    ## Projected CRS: LKS-92 / Latvia TM

``` r
centrs_SHP=rbind(n2651_shp,n2652_shp,n2653_shp,n2654_shp,n2655_shp)
st_write_parquet(centrs_SHP, dsn="centrs_kopa.parquet")
rm(n2651_shp,n2652_shp,n2653_shp,n2654_shp,n2655_shp,centrs_SHP,pattern)
centrs_kopa=st_read_parquet("centrs_kopa.parquet")
```

\##2.2. faila pārbaude Tālāk jāpārliecinās, ka visas ģeometrijas ir
MULTIPOLYGON, slānis nesatur tukšas vai nekorektas (invalid)
ģeometrijas. To var apskatīt ar visparastāko datu struktūras pārbaudes
funkciju str(). Tās pēdējais rezultāts ir \$geometry, kur redzams, ka
visi 505660 novērojumi (rindas) ir multipoligon.

``` r
str(centrs_kopa$geometry)
```

    ## sfc_MULTIPOLYGON of length 505660; first list element: List of 1
    ##  $ :List of 1
    ##   ..$ : num [1:95, 1:2] 536651 536654 536658 536590 536618 ...
    ##  - attr(*, "class")= chr [1:3] "XY" "MULTIPOLYGON" "sfg"

Lai pārbaudītu, vai slānis nesatur tukšas vērtības, izveidoju tabulu
“iztrukumi”, kur šūnas saturs aizvietots ar TRUE, ja tā ir tukša, un
FALSE, ja tajā ir kāda vērtība. Tad, izmantojot “tidyverse” pakotni
atlasu, kurās rindās no visām kolonnām ir iztrūkstošas vērtības. Secinu,
ka nevienā no rindām nevienā kolonnā nav iztrūkstošu vērtību. Aizvācu
izveidoto tabulu “iztrukumi” no R vides.

``` r
iztrukumi=as.data.frame(is.na(centrs_kopa))
library(tidyverse)
iztrukumi%>% filter_all(any_vars(. %in% c('TRUE')))
```

    ##  [1] id         objectid_1 adm1       adm2       atvk       kadastrs  
    ##  [7] gtf        kvart      nog        anog       nog_plat   expl_mezs 
    ## [13] expl_celi  expl_gravj zkat       mt         izc        s10       
    ## [19] a10        h10        d10        g10        n10        bv10      
    ## [25] ba10       s11        a11        h11        d11        g11       
    ## [31] n11        bv11       ba11       s12        a12        h12       
    ## [37] d12        g12        n12        bv12       ba12       s13       
    ## [43] a13        h13        d13        g13        n13        bv13      
    ## [49] ba13       s14        a14        h14        d14        g14       
    ## [55] n14        bv14       ba14       jakopj     jaatjauno  p_darbv   
    ## [61] p_darbg    p_cirp     p_cirg     atj_gads   saimn_d_ie plant_audz
    ## [67] forestry_c vmd_headfo shape_Leng shape_Area geometry  
    ## <0 rows> (or 0-length row.names)

``` r
rm(iztrukumi)
```

Tagad jāparbauda, vai slānis satur tikai korektas ģeometrijas. Lai
noskaidrotu nekorektas ģeometrijas iemeslus, funkcijai st_is_valid()
jāpievieno reason=TRUE. Ar sort(unique())) atlasu unikālās vērtības
starp validitātes iemesliem. Apskatu visus rezultātus. Redzu, ka 274
poligoniem ir “Ring Self-intersection”, kas nozīmē, ka tie pārklājas
paši ar sevi, jeb to forma satur malu savstarpēju krustošanos (piemēram,
caurumus vai viena poligona malas saskaršanos).

``` r
v=data.frame(geom_val=st_is_valid(centrs_kopa$geometry, reason=TRUE))
sort(unique(v$geom_val))
```

    ##   [1] "Ring Self-intersection[409740.656 302743.182]"       
    ##   [2] "Ring Self-intersection[412204.3895 304829.202]"      
    ##   [3] "Ring Self-intersection[415798.3584 307527.597899999]"
    ##   [4] "Ring Self-intersection[417864.7424 298315.000499999]"
    ##   [5] "Ring Self-intersection[421443.2496 311582.832699999]"
    ##   [6] "Ring Self-intersection[422721.58 266561.300000001]"  
    ##   [7] "Ring Self-intersection[425828.7308 300455.681700001]"
    ##   [8] "Ring Self-intersection[426087.76 297036.32]"         
    ##   [9] "Ring Self-intersection[426981.2271 271646.909499999]"
    ##  [10] "Ring Self-intersection[427219.1419 322403.1631]"     
    ##  [11] "Ring Self-intersection[427416.1807 260579.8202]"     
    ##  [12] "Ring Self-intersection[429718.002 262071.573999999]" 
    ##  [13] "Ring Self-intersection[429911.2275 332506.468]"      
    ##  [14] "Ring Self-intersection[430626.3681 310118.238]"      
    ##  [15] "Ring Self-intersection[432688.0624 299546.162699999]"
    ##  [16] "Ring Self-intersection[433075.6504 293814.000700001]"
    ##  [17] "Ring Self-intersection[433265.65 309181.684]"        
    ##  [18] "Ring Self-intersection[433323.561 323903.698000001]" 
    ##  [19] "Ring Self-intersection[434274.1774 299687.244000001]"
    ##  [20] "Ring Self-intersection[434430.77 273820.785]"        
    ##  [21] "Ring Self-intersection[437842.6779 295805.413799999]"
    ##  [22] "Ring Self-intersection[438703.1641 309955.749199999]"
    ##  [23] "Ring Self-intersection[438771.3799 254746.6763]"     
    ##  [24] "Ring Self-intersection[439909.3265 334798.7925]"     
    ##  [25] "Ring Self-intersection[440630.7885 266853.781400001]"
    ##  [26] "Ring Self-intersection[440765.6189 322253.314300001]"
    ##  [27] "Ring Self-intersection[442478.1327 266896.661]"      
    ##  [28] "Ring Self-intersection[442921.84 263982.960000001]"  
    ##  [29] "Ring Self-intersection[443704.215 255003.244999999]" 
    ##  [30] "Ring Self-intersection[444406.4722 258631.2885]"     
    ##  [31] "Ring Self-intersection[444537.36 268241.710000001]"  
    ##  [32] "Ring Self-intersection[444642.456 275961.914000001]" 
    ##  [33] "Ring Self-intersection[444932.1911 330642.159]"      
    ##  [34] "Ring Self-intersection[445135.4974 270831.6767]"     
    ##  [35] "Ring Self-intersection[445194.8871 274535.249700001]"
    ##  [36] "Ring Self-intersection[445423.6102 304007.5252]"     
    ##  [37] "Ring Self-intersection[445539.1763 303885.2257]"     
    ##  [38] "Ring Self-intersection[445875.45 327177.762]"        
    ##  [39] "Ring Self-intersection[446391.8916 340194.0373]"     
    ##  [40] "Ring Self-intersection[447554.6768 329462.053300001]"
    ##  [41] "Ring Self-intersection[448054.5729 318712.885399999]"
    ##  [42] "Ring Self-intersection[448551.1674 254378.509]"      
    ##  [43] "Ring Self-intersection[448589.3494 308822.278999999]"
    ##  [44] "Ring Self-intersection[448909.627 319431.863]"       
    ##  [45] "Ring Self-intersection[450161.51 281568.4]"          
    ##  [46] "Ring Self-intersection[450187.7879 320276.6096]"     
    ##  [47] "Ring Self-intersection[451294.3606 276005.694499999]"
    ##  [48] "Ring Self-intersection[451646.6465 274106.901900001]"
    ##  [49] "Ring Self-intersection[451780.6301 270521.220000001]"
    ##  [50] "Ring Self-intersection[451920.1344 324010.733899999]"
    ##  [51] "Ring Self-intersection[451979.6718 273500.256999999]"
    ##  [52] "Ring Self-intersection[452189.4822 321116.1491]"     
    ##  [53] "Ring Self-intersection[452234.6733 269641.9394]"     
    ##  [54] "Ring Self-intersection[455820.5899 267947.8729]"     
    ##  [55] "Ring Self-intersection[458637.7951 319144.4989]"     
    ##  [56] "Ring Self-intersection[458650.4943 320659.303099999]"
    ##  [57] "Ring Self-intersection[458722.1826 323926.876499999]"
    ##  [58] "Ring Self-intersection[459404.4915 284006.640900001]"
    ##  [59] "Ring Self-intersection[461483.66 260809.034]"        
    ##  [60] "Ring Self-intersection[461677.9874 322044.855599999]"
    ##  [61] "Ring Self-intersection[462555.355 299885.8883]"      
    ##  [62] "Ring Self-intersection[463142.24 312071.49]"         
    ##  [63] "Ring Self-intersection[464426.8 318079.93]"          
    ##  [64] "Ring Self-intersection[465274.3925 294306.1908]"     
    ##  [65] "Ring Self-intersection[467600.0219 315805.298900001]"
    ##  [66] "Ring Self-intersection[468730.61 251410.5]"          
    ##  [67] "Ring Self-intersection[471285.7335 316270.9583]"     
    ##  [68] "Ring Self-intersection[471509.81 303617]"            
    ##  [69] "Ring Self-intersection[474565.0292 275382.217900001]"
    ##  [70] "Ring Self-intersection[476280.3089 309152.279999999]"
    ##  [71] "Ring Self-intersection[476319.08 252230.300000001]"  
    ##  [72] "Ring Self-intersection[479656.9869 256752.3618]"     
    ##  [73] "Ring Self-intersection[480399.3151 293114.999299999]"
    ##  [74] "Ring Self-intersection[481595.416 297088.006999999]" 
    ##  [75] "Ring Self-intersection[482150.8322 313327.780200001]"
    ##  [76] "Ring Self-intersection[483431.04 312164.1798]"       
    ##  [77] "Ring Self-intersection[483635.21 313614.27]"         
    ##  [78] "Ring Self-intersection[483779.8134 291922.065300001]"
    ##  [79] "Ring Self-intersection[486609.998 303679.603]"       
    ##  [80] "Ring Self-intersection[489432.23 299764.26]"         
    ##  [81] "Ring Self-intersection[489994.11 303555.630000001]"  
    ##  [82] "Ring Self-intersection[490077.9685 246245.5614]"     
    ##  [83] "Ring Self-intersection[491354.3111 273270.0318]"     
    ##  [84] "Ring Self-intersection[491609.7 288732.17]"          
    ##  [85] "Ring Self-intersection[492480.24 298561.284]"        
    ##  [86] "Ring Self-intersection[495048.7044 280127.7424]"     
    ##  [87] "Ring Self-intersection[496136.075 315503.6556]"      
    ##  [88] "Ring Self-intersection[496215.8598 315511.2105]"     
    ##  [89] "Ring Self-intersection[496458.709 316009.483999999]" 
    ##  [90] "Ring Self-intersection[497721.688 300791.945]"       
    ##  [91] "Ring Self-intersection[498272.4992 280140.476299999]"
    ##  [92] "Ring Self-intersection[499867.41 273235.109999999]"  
    ##  [93] "Ring Self-intersection[500152.996 254447.092499999]" 
    ##  [94] "Ring Self-intersection[500573.97 296423.960000001]"  
    ##  [95] "Ring Self-intersection[500734.757 315118.332]"       
    ##  [96] "Ring Self-intersection[502481.0267 270663.329600001]"
    ##  [97] "Ring Self-intersection[502638.57 270401.800000001]"  
    ##  [98] "Ring Self-intersection[504600.024 271919.25]"        
    ##  [99] "Ring Self-intersection[504801.1285 272870.6458]"     
    ## [100] "Ring Self-intersection[507302.839 323575.261]"       
    ## [101] "Ring Self-intersection[507509.551 304399.027000001]" 
    ## [102] "Ring Self-intersection[507531.813 267542.501]"       
    ## [103] "Ring Self-intersection[510047.0419 293692.1853]"     
    ## [104] "Ring Self-intersection[510587.33 295368]"            
    ## [105] "Ring Self-intersection[511255.147 322876.977]"       
    ## [106] "Ring Self-intersection[512806.3646 288683.737400001]"
    ## [107] "Ring Self-intersection[514093.117 266959.123299999]" 
    ## [108] "Ring Self-intersection[514789.877 320545.401000001]" 
    ## [109] "Ring Self-intersection[515592.723 283785.720100001]" 
    ## [110] "Ring Self-intersection[515766.3145 331932.933700001]"
    ## [111] "Ring Self-intersection[515767.6929 281834.761299999]"
    ## [112] "Ring Self-intersection[515981.645 285955.08]"        
    ## [113] "Ring Self-intersection[515999.9963 331310.614600001]"
    ## [114] "Ring Self-intersection[516545.9137 296732.1065]"     
    ## [115] "Ring Self-intersection[516847.5045 290594.415999999]"
    ## [116] "Ring Self-intersection[516888.5609 282789.2678]"     
    ## [117] "Ring Self-intersection[517278.4715 285451.884500001]"
    ## [118] "Ring Self-intersection[517352.9669 261309.2982]"     
    ## [119] "Ring Self-intersection[517662.25 308921.1006]"       
    ## [120] "Ring Self-intersection[517996.498 302550.577]"       
    ## [121] "Ring Self-intersection[518059.63 333187.75]"         
    ## [122] "Ring Self-intersection[518807.99 309252.51]"         
    ## [123] "Ring Self-intersection[519185.037 320663.0655]"      
    ## [124] "Ring Self-intersection[519891.97 307240.449999999]"  
    ## [125] "Ring Self-intersection[520017.41 312957.050000001]"  
    ## [126] "Ring Self-intersection[520074.5 283203.640000001]"   
    ## [127] "Ring Self-intersection[520424.308 317547.176000001]" 
    ## [128] "Ring Self-intersection[520805.8806 247375.001499999]"
    ## [129] "Ring Self-intersection[520845.514 317171.622]"       
    ## [130] "Ring Self-intersection[521273.88 323167.65]"         
    ## [131] "Ring Self-intersection[521766.5596 323548.6778]"     
    ## [132] "Ring Self-intersection[521808.672 330807.537]"       
    ## [133] "Ring Self-intersection[522338.3749 253029.728399999]"
    ## [134] "Ring Self-intersection[522363.34 336109.130000001]"  
    ## [135] "Ring Self-intersection[522653.4832 335694.895199999]"
    ## [136] "Ring Self-intersection[522689.9778 337410.5934]"     
    ## [137] "Ring Self-intersection[522951.2322 273929.922499999]"
    ## [138] "Ring Self-intersection[523495.16 303609.25]"         
    ## [139] "Ring Self-intersection[523507.739 275321.456]"       
    ## [140] "Ring Self-intersection[524535.75 352993.380000001]"  
    ## [141] "Ring Self-intersection[524649.525 323114.822000001]" 
    ## [142] "Ring Self-intersection[524706.381 316691.518999999]" 
    ## [143] "Ring Self-intersection[524958.1538 314290.293099999]"
    ## [144] "Ring Self-intersection[525050.495 261153.617000001]" 
    ## [145] "Ring Self-intersection[525130.5856 342874.407400001]"
    ## [146] "Ring Self-intersection[525200.5801 342931.1241]"     
    ## [147] "Ring Self-intersection[525651.6061 276445.3891]"     
    ## [148] "Ring Self-intersection[525814.106 288827.619999999]" 
    ## [149] "Ring Self-intersection[526219.84 321320.374]"        
    ## [150] "Ring Self-intersection[526698.03 304351.619999999]"  
    ## [151] "Ring Self-intersection[527070.6068 320975.311100001]"
    ## [152] "Ring Self-intersection[527119.315 340526.123]"       
    ## [153] "Ring Self-intersection[527324.466 308168.514]"       
    ## [154] "Ring Self-intersection[527419.981 309310.625]"       
    ## [155] "Ring Self-intersection[527701.169 350940.044299999]" 
    ## [156] "Ring Self-intersection[527766.296 242506.931]"       
    ## [157] "Ring Self-intersection[528403.33 330514.859999999]"  
    ## [158] "Ring Self-intersection[528566.118 314144.865]"       
    ## [159] "Ring Self-intersection[528598.6 331747.17]"          
    ## [160] "Ring Self-intersection[528927.483 313715.347999999]" 
    ## [161] "Ring Self-intersection[529294.1379 262446.4377]"     
    ## [162] "Ring Self-intersection[529649.3059 305384.1612]"     
    ## [163] "Ring Self-intersection[529649.38 332616]"            
    ## [164] "Ring Self-intersection[529772.3157 266075.509199999]"
    ## [165] "Ring Self-intersection[529995.2051 328027.8026]"     
    ## [166] "Ring Self-intersection[530295.6204 280951.237299999]"
    ## [167] "Ring Self-intersection[530502.03 295899.74]"         
    ## [168] "Ring Self-intersection[530874.592 253324.0133]"      
    ## [169] "Ring Self-intersection[531699.7319 262554.5341]"     
    ## [170] "Ring Self-intersection[532016 244908]"               
    ## [171] "Ring Self-intersection[532041.7715 330259.624700001]"
    ## [172] "Ring Self-intersection[532957.75 330335.939999999]"  
    ## [173] "Ring Self-intersection[533948.5895 326583.8336]"     
    ## [174] "Ring Self-intersection[534134.0721 322432.118799999]"
    ## [175] "Ring Self-intersection[534412.81 331219.9595]"       
    ## [176] "Ring Self-intersection[534895.11 328753.431]"        
    ## [177] "Ring Self-intersection[535134.4927 252288.496200001]"
    ## [178] "Ring Self-intersection[535134.5091 266230.651799999]"
    ## [179] "Ring Self-intersection[535267.5681 252084.4572]"     
    ## [180] "Ring Self-intersection[535275.6316 250345.319800001]"
    ## [181] "Ring Self-intersection[535732.3759 310206.277000001]"
    ## [182] "Ring Self-intersection[536175.1195 265326.4099]"     
    ## [183] "Ring Self-intersection[536386.2554 252028.237500001]"
    ## [184] "Ring Self-intersection[536449.3856 269719.094799999]"
    ## [185] "Ring Self-intersection[537012.71 344622.07]"         
    ## [186] "Ring Self-intersection[537014.6664 258317.252900001]"
    ## [187] "Ring Self-intersection[537090.9131 301398.1613]"     
    ## [188] "Ring Self-intersection[537225.2623 330125.897]"      
    ## [189] "Ring Self-intersection[537316.6489 318814.1592]"     
    ## [190] "Ring Self-intersection[537515.1212 297378.2338]"     
    ## [191] "Ring Self-intersection[537641.7802 315446.752900001]"
    ## [192] "Ring Self-intersection[537665.495 329490.482000001]" 
    ## [193] "Ring Self-intersection[537993.21 334463.82]"         
    ## [194] "Ring Self-intersection[539139.7 298666.189999999]"   
    ## [195] "Ring Self-intersection[539386.57 262016.343900001]"  
    ## [196] "Ring Self-intersection[539660.6372 262562.9648]"     
    ## [197] "Ring Self-intersection[539948.722 303376.727]"       
    ## [198] "Ring Self-intersection[540418.6544 252486.2378]"     
    ## [199] "Ring Self-intersection[540775.77 287050.109999999]"  
    ## [200] "Ring Self-intersection[540996.56 338051.25]"         
    ## [201] "Ring Self-intersection[541162.5654 324967.3265]"     
    ## [202] "Ring Self-intersection[541473.2899 266090.126399999]"
    ## [203] "Ring Self-intersection[542889.1375 354416.6251]"     
    ## [204] "Ring Self-intersection[543036.5118 266069.486199999]"
    ## [205] "Ring Self-intersection[543364.0139 349159.666999999]"
    ## [206] "Ring Self-intersection[543427.3175 345869.5284]"     
    ## [207] "Ring Self-intersection[543473.357 256797.772]"       
    ## [208] "Ring Self-intersection[543715.6522 334054.8564]"     
    ## [209] "Ring Self-intersection[543756.257 340271.5153]"      
    ## [210] "Ring Self-intersection[543962.26 305175.73]"         
    ## [211] "Ring Self-intersection[544362.9046 343812.3002]"     
    ## [212] "Ring Self-intersection[544376.8969 340454.2623]"     
    ## [213] "Ring Self-intersection[544797.0017 340587.922499999]"
    ## [214] "Ring Self-intersection[544925.5276 320148.772600001]"
    ## [215] "Ring Self-intersection[545296.7607 359119.7938]"     
    ## [216] "Ring Self-intersection[545917.3992 279365.9087]"     
    ## [217] "Ring Self-intersection[546170.22 291645.09]"         
    ## [218] "Ring Self-intersection[546177.418 324219.2115]"      
    ## [219] "Ring Self-intersection[546219.9682 255694.7335]"     
    ## [220] "Ring Self-intersection[546467.4224 255924.571900001]"
    ## [221] "Ring Self-intersection[546636.31 295491.5]"          
    ## [222] "Ring Self-intersection[546855.5098 284775.3916]"     
    ## [223] "Ring Self-intersection[547130.3729 320365.6768]"     
    ## [224] "Ring Self-intersection[547446 284395]"               
    ## [225] "Ring Self-intersection[548286.64 262617.2129]"       
    ## [226] "Ring Self-intersection[548724.3636 262231.259500001]"
    ## [227] "Ring Self-intersection[548980.55 333284.029999999]"  
    ## [228] "Ring Self-intersection[549251.6253 330296.7042]"     
    ## [229] "Ring Self-intersection[549303.7074 263971.6042]"     
    ## [230] "Ring Self-intersection[549322.01 257933.335000001]"  
    ## [231] "Ring Self-intersection[549327.338 330051.147500001]" 
    ## [232] "Ring Self-intersection[549351.8482 307628.739700001]"
    ## [233] "Ring Self-intersection[551160.1548 352606.443299999]"
    ## [234] "Ring Self-intersection[551679.6287 332028.240800001]"
    ## [235] "Ring Self-intersection[551936.0834 314413.3291]"     
    ## [236] "Ring Self-intersection[551942.294 355373.357999999]" 
    ## [237] "Ring Self-intersection[552434.83 300052.300000001]"  
    ## [238] "Ring Self-intersection[552870.75 314304.74]"         
    ## [239] "Ring Self-intersection[552924.0606 293889.029100001]"
    ## [240] "Ring Self-intersection[553259.4205 296084.2268]"     
    ## [241] "Ring Self-intersection[553400.5 303798.880000001]"   
    ## [242] "Ring Self-intersection[554814.0635 304042.0612]"     
    ## [243] "Ring Self-intersection[555098.5869 315160.6941]"     
    ## [244] "Ring Self-intersection[558286.84 311926.27]"         
    ## [245] "Ring Self-intersection[559892.85 316562.460000001]"  
    ## [246] "Ring Self-intersection[560068.2919 294218.3177]"     
    ## [247] "Ring Self-intersection[560102.42 302470.199999999]"  
    ## [248] "Ring Self-intersection[562708.4261 300825.1328]"     
    ## [249] "Ring Self-intersection[563022.57 304753.810000001]"  
    ## [250] "Ring Self-intersection[567044.1418 328398.7831]"     
    ## [251] "Ring Self-intersection[567982.014 298366.459000001]" 
    ## [252] "Ring Self-intersection[569704.02 295093.0294]"       
    ## [253] "Ring Self-intersection[570532.6002 294600.2216]"     
    ## [254] "Ring Self-intersection[572655.3829 300255.840500001]"
    ## [255] "Ring Self-intersection[573081.8707 288844.363399999]"
    ## [256] "Ring Self-intersection[574402.5003 303910.999399999]"
    ## [257] "Ring Self-intersection[576394.9936 300453.512]"      
    ## [258] "Ring Self-intersection[576415.5394 299366.6105]"     
    ## [259] "Ring Self-intersection[576704.4061 312749.296499999]"
    ## [260] "Ring Self-intersection[577173.7759 302195.7217]"     
    ## [261] "Ring Self-intersection[577307.5493 295584.0529]"     
    ## [262] "Ring Self-intersection[577583.27 303135.359999999]"  
    ## [263] "Ring Self-intersection[578909.9763 294148.983200001]"
    ## [264] "Ring Self-intersection[580433.9395 292233.7524]"     
    ## [265] "Ring Self-intersection[582168.9291 293627.998]"      
    ## [266] "Ring Self-intersection[582255.97 315076.050000001]"  
    ## [267] "Ring Self-intersection[582377.98 307008.449999999]"  
    ## [268] "Ring Self-intersection[584894.539 304576.022]"       
    ## [269] "Ring Self-intersection[586944.322 307399.088]"       
    ## [270] "Ring Self-intersection[587374.2444 310705.2278]"     
    ## [271] "Ring Self-intersection[587614.437 301566.564099999]" 
    ## [272] "Ring Self-intersection[587949.669 303108.0954]"      
    ## [273] "Ring Self-intersection[589817.8213 309059.3847]"     
    ## [274] "Ring Self-intersection[591282.9251 315362.2621]"     
    ## [275] "Valid Geometry"

Šo problēmu tagad ir jārisina. Ar to jābūt uzmanīgam, jo atkarībā no
risinājuma var potenciāli mainīties tālākie rezultāti. Ir divi varianti,
ko darīt ar nekorektajā, ģeometrijām - tās izdzēst (zaudēsim datus,
tāpēc to neizvēlos) vai tās labot ar funkciju st_make valid().Šajā
funkcijā ir divi varianti, ko norādīt pie argumenta geos_method=, kas
nosaka labošanas metodi- “valid linework”, kas cik saprotu izveido
līnijas starp poligona robežas krustpunktiem, un tad ģenerē korektus
poligonus no šīm līnijām, vai arī “valid structure”, kas paredz, ka
poligonu forma (robežas un “caurumi”) ir pareizi kategorizētas, un
norāda, ka tās ir pareizas. Pieņemu lēmumu, ka otrs variants ir labāks
(bet neesmu pārliecināta, tāpēc izmēģināju abus variantus. Pirmais
atrisināja problēmu, otrais radīja sešas tukšas ģeometrijas). Izmēģinu
abus varian

Tad atkārtoju pārbaudi un secinu, ka visas ģeometrijas ir korektas,
papildus, nav tukšu lauku. Šķiet, ka viss ir pareizi, tāpēc atbrīvojos
no liekajiem objektiem, un paturu tikai jaunizveidoto “centrs_kopa2”,
kas satur tikai korektas ģeometrijas.

``` r
centrs_kopa2=st_make_valid(centrs_kopa, geos_method = "valid_structure")
v2=data.frame(geom_val=st_is_valid(centrs_kopa2$geometry, reason=TRUE))
sort(unique(v2$geom_val))
```

    ## [1] "Valid Geometry"

``` r
iztrukumi2=as.data.frame(is.na(centrs_kopa2))
iztrukumi2%>% filter_all(any_vars(. %in% c('TRUE')))
```

    ##  [1] id         objectid_1 adm1       adm2       atvk       kadastrs  
    ##  [7] gtf        kvart      nog        anog       nog_plat   expl_mezs 
    ## [13] expl_celi  expl_gravj zkat       mt         izc        s10       
    ## [19] a10        h10        d10        g10        n10        bv10      
    ## [25] ba10       s11        a11        h11        d11        g11       
    ## [31] n11        bv11       ba11       s12        a12        h12       
    ## [37] d12        g12        n12        bv12       ba12       s13       
    ## [43] a13        h13        d13        g13        n13        bv13      
    ## [49] ba13       s14        a14        h14        d14        g14       
    ## [55] n14        bv14       ba14       jakopj     jaatjauno  p_darbv   
    ## [61] p_darbg    p_cirp     p_cirg     atj_gads   saimn_d_ie plant_audz
    ## [67] forestry_c vmd_headfo shape_Leng shape_Area geometry  
    ## <0 rows> (or 0-length row.names)

``` r
rm(v, v2, iztrukumi2, centrs_kopa)
```

\#3. Priežu īpatsvari Tagad jāaprēķina pirmā stāva šķērslaukumu
īpatsvaru, ko veido priedes (jāizveido jauns lauks “prop_priedes”).

Jārēķinās ar datu bāzes struktūru. Pirmkārt, priedes pirmajā stāvā gan
var būt kā pirmā (s10), gan kā otrā(s11), trešā(s12), ceturtā(s13) vai
piektā suga(s14). Otrkārt, datu bāzē ir trīs priežu sugas - parastā
priede (kods 1), citas priedes (kods 14) un ciedru priede (kods 22).
Attiecīgi laukiem s10:s14 jāsatur kodi 1, 14 vai 22, kas apzīmē priedes.
Pirmā stāva sugu šķērslaukumi (m2/ha) atrodas attiecīgi laukos g10-g14.

Var izmantot ifelse() funkciju kombināciju, lai saskaitītu šķērslakumu
tikai tām rindām, kas satur vērtību 1,14 vai 22 attiecīgi laukos
s10-s14. Saskaitot jāuzmanās, lai netiktu saskaitīti visu sugu
šķērslaukumi, gadījumos, ja piemēram tikai pirmā stāva pirmā suga ir
priede, bet pārējās pirmā stāva sugas ir citas. Tāpēc izvēlos garāku,
bet manuprāt drošāku risinājumu, saskaitot šos šķērslaukumus jaunā laukā
“skersl_priedes1st”.

``` r
centrs_kopa2$skersl_priedes1st=ifelse(centrs_kopa2$s10=="1"|centrs_kopa2$s10=="14"|centrs_kopa2$s10=="22",centrs_kopa2$g10,0)+
+                                    ifelse(centrs_kopa2$s11=="1"|centrs_kopa2$s11=="14"|centrs_kopa2$s11=="22",centrs_kopa2$g11,0)+
+                                    ifelse(centrs_kopa2$s12=="1"|centrs_kopa2$s12=="14"|centrs_kopa2$s12=="22",centrs_kopa2$g12,0)+
+                                    ifelse(centrs_kopa2$s13=="1"|centrs_kopa2$s13=="14"|centrs_kopa2$s13=="22",centrs_kopa2$g13,0)+
+                                    ifelse(centrs_kopa2$s14=="1"|centrs_kopa2$s14=="14"|centrs_kopa2$s14=="22",centrs_kopa2$g14,0)
```

Tālāk, lai iegūtu proporciju, jau aprēķināto priežu kumulatīvo
šķērslaukumu jādala ar visu sugu šķērslaukumu. Ņemot vērā, ka atsevišķām
mežaudžu kategorijām kopējais sķērslaukums ir 0 (piemēram, kailcirtēm),
tad aprēķinot proporciju šajos gadījumos sanāk saucējā 0, kas atgriež
“NaN” (not a number). Tas jāņem vērā tālākos aprēķinos. Starpaprēķinu
kolonnas izņemu no tabulas.

``` r
centrs_kopa2$skersl_visi=centrs_kopa2$g10+centrs_kopa2$g11+centrs_kopa2$g12+centrs_kopa2$g13+centrs_kopa2$g14

centrs_kopa2$prop_priedes=centrs_kopa2$skersl_priedes1st/centrs_kopa2$skersl_visi
centrs_kopa2$skersl_priedes1st=NULL
```

Tālāk veidoju jaunu lauku “PriezuMezi”, kurā ar vērtību “1” atzīmētas
tās mežaudzes, kurās priedes šķērslaukuma īpatsvars pirmajā stāvā ir
vismaz 75% un ar “0” tās mežaudzes, kurās īpatsvars ir mazāks (bet tomēr
ne 0), bet pārējām mežaudzēm, kurās priedes vispār neveido nekādu
šķērslaukumu (faktiski 0 vai NaN), šajā laukā jāatstāj tukšumi (NA).

``` r
centrs_kopa2$PriezuMezi=ifelse(centrs_kopa2$prop_priedes>=0.75,1,ifelse(centrs_kopa2$prop_priedes>0,0,NA))
```

Tālāk jānoskaidro, kāds ir priežu mežaudžu īpatsvars no visām mežaudzēm.
Ja pieņem, ka priežu mežaudzes ir tās, kurās priedes veido vismaz 75% no
kopējā šķērslaukuma, tad to noskaidrot var vienkārši saskaitot cik
mežaudzēm no kopējām 505660 mežaudzēm laukā “PriezuMezi” ir vērtība “1”
(ignorējot tukšās šūnas, tāpēc arguments na.rm=TRUE), un izdalot to ar
kopējo mežaudžu skaitu, kas atbilst rindu skaitam matricā. Lai iegūtu %,
pareizinu iegūto proporciju ar 100. Noapaļoju rezultātu ar diviem
cipariem aiz komata. Secinu, ka Centra virsmežniecībā 17.83% ir priežu
mežaudes.

``` r
round(sum(centrs_kopa2$PriezuMezi,na.rm = TRUE)/nrow(centrs_kopa2)*100,2)
```

    ## [1] 17.84

\#4. Mežaudžu klasifikācija Lai sāktu mežu klasifikāciju, vispirms
jānodefinē, kādi koki veido šos mežu veidus. Balstoties uz Latvijas
biotopu klasifikatoru (Kabucis 2001), mežu biotopus var izdalīt pēc
dominējošās/-ām (1. stāva) koku sugas/-ām. Pastāv dažādas dominances
skalas. Nesenā pētījumā par dominējošām koku sugām uzskatītas tādas,
kuru šķērslaukums veido vismaz 10% no teritorijas kopējā šķērslaukuma
(<https://doi.org/10.1111/geb.13889>), bet šāds slieksnis nešķiet
piemērots šim kontekstam. Par dominēšanas slieksni izmantošu 75%
(augšējo kvartilli), kā tas jau iepriekš izmantots, un ir saskaņā ar
daļu no eksistējošās dominances skalām. Ja pirmajā stāvā nav vienas
dominējošas sugas, un vienlaicīgi sastopamas gan skujkoku un platlapju,
vai skujkoku un šaurlapju, vai visas trīs kategorijas kopā, tad šādu
mežu var klasificēt kā jauktu koku mežu

Pēdējais jautājums, kas jānoskaidro pirms klasificēšanas sākšanas, ir
kuras koku sugas pieder pie šaurlapjiem, kuras - pie platlapjiem, bet
kuras - pie skujkokiem. Ja ar pēdējo kategoriju problēmu nav, tad
informāciju par to, kādai kategorijai pieder lapukoki ir problemātiskāk.
Vispār pieņemts, ka bērzi, apses un alkšņi ir šaurlapji, bet dažādos
informācijas avotos nemaz neizmanto grupēšanu šaurlapjos, un piemēram,
purva bērzu iedala kā platlapi
(<https://www.ucc.ie/en/tree-explorers/trees/a-z/betulapubescens/> vai
<https://landscapeplants.oregonstate.edu/plants/betula-pubescens>).

Sākumā, balstoties uz sugu aprakstiem , kādas koku sugas vispār
sastopamas Centra virsmežniecības mežos. Pēc apskatas vērtības izņemu no
R vides.

``` r
sugas1st=unique(c(unique(centrs_kopa2$s10),unique(centrs_kopa2$s11),unique(centrs_kopa2$s12),unique(centrs_kopa2$s13),unique(centrs_kopa2$s14)))
sort(as.numeric(sugas1st))
```

    ##  [1]  0  1  3  4  6  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27
    ## [26] 29 32 35 50 61 62 63 64 66 67 68

``` r
rm(sugas1st)
```

Tā kā neatrodu vienu informācijas avotu, kur būtu klasificētas šādās
kategorijās visas meža valsts reģistrā iekļautās un šajos datos
sastopamās 1. stāva koku sugas, tad šajā uzdevumā 1. stāvā sastopamās
koku sugas pa kategorijām iedalīju šādi:

skujkoki (koki ar zvīņveida lapām vai skujām)- priedes (1, 14, 22),
egles (3, 13, 15, 23), īve (29); šaurlapji (ātri augoši koki ar šaurām
lapām)- bērzi (4), alkšņi (6,9), apses (8,68), vītoli (19,20,21);
platlapji (koki ar platām lapām)- ozoli (10, 61), oši (11, 64), kļavas
(24, 63), liepas (12, 62), gobas un vīksnas (16, 65), skabārži (17, 18),
zirgkastaņi (67), riekstkoki (66), papildus šeit kategorizēju ķiršus,
mežābeles un bumbieres (25, 26, 27), kā arī pīlādžus, ievas un akācijas
(32,35,50) (skat. komentāru zemāk); jauktu koku meži - meži, kuros 1.
stāvā dominē gan skujkoku, gan lapkoku (šaurlapju vai platlapju) sugas.

Diskutabls ir ābeļu, ķiršu, bumbieru, pīlādžu, ievu un akāciju
iedalījums. Šoreiz iedalīju pie platlapjiem, bet pēc būtības grūti
iztēloties šos mežus, kuros 1. stāvā ir šīs sugas. Tas varētu būt kādas
jaunaudzes vai parki, bet katrā ziņā noteikti tās nesastāda lielu
īpatsvaru. Būtībā šāds iedalījums platlapju un šaurlapju mežos vairāk
aktuāls ir meža augsšanas apstākļu tipu kontekstā, atkarībā no augsnes
auglības un mitruma. Tāpēc tā vienkārši pēc sugām kategorizēt var, ja
norāda kuras sugas kur ieliek, bet tikpat labi, varētu ņemt vērā
vairākus faktorus, lai šīs kategorijas būtu precīzākas un labāk
izmantojamas.

Tehniskam izpildījumam, sākumā izveidoju sarakstus ar koku sugām,
atkarībā no to koda datubāzē. Tad, ieviesu trīs jaunus laukus, kur
aprēķināts šo mežaudžu kategoriju veidojošo koku sugu kopējais
šķerslaukums 1. stāvā.

``` r
skujkoki = c(1, 3, 13, 14, 15, 22, 23, 28, 29)
saurlapji =c(4, 6, 8, 9, 19, 20, 21, 68)
platlapji = c(10, 11, 12, 16, 17, 18, 24, 25, 26, 27, 32, 35, 50, 61, 63, 64, 65, 66, 67)

centrs_kopa2$skersl_skujkoki = 
  ifelse(centrs_kopa2$s10 %in% skujkoki, centrs_kopa2$g10,0) +
  ifelse(centrs_kopa2$s11 %in% skujkoki, centrs_kopa2$g11,0) + 
  ifelse(centrs_kopa2$s12 %in% skujkoki, centrs_kopa2$g12,0) + 
  ifelse(centrs_kopa2$s13 %in% skujkoki, centrs_kopa2$g13,0) + 
  ifelse(centrs_kopa2$s14 %in% skujkoki, centrs_kopa2$g14,0)

centrs_kopa2$skersl_saurlapji = 
  ifelse(centrs_kopa2$s10 %in% saurlapji, centrs_kopa2$g10,0) +
  ifelse(centrs_kopa2$s11 %in% saurlapji, centrs_kopa2$g11,0) + 
  ifelse(centrs_kopa2$s12 %in% saurlapji, centrs_kopa2$g12,0) + 
  ifelse(centrs_kopa2$s13 %in% saurlapji, centrs_kopa2$g13,0) + 
  ifelse(centrs_kopa2$s14 %in% saurlapji, centrs_kopa2$g14,0)

centrs_kopa2$skersl_platlapji = 
  ifelse(centrs_kopa2$s10 %in% platlapji, centrs_kopa2$g10,0) +
  ifelse(centrs_kopa2$s11 %in% platlapji, centrs_kopa2$g11,0) + 
  ifelse(centrs_kopa2$s12 %in% platlapji, centrs_kopa2$g12,0) + 
  ifelse(centrs_kopa2$s13 %in% platlapji, centrs_kopa2$g13,0) + 
  ifelse(centrs_kopa2$s14 %in% platlapji, centrs_kopa2$g14,0)
```

Tālāk jānosaka proporcija (jeb jānoskaidro dominance). To var
noskaidrot, dalot iegūto šķērslaukumu ar kopējo, kas jau iepriekš
aprēķināts laukā “skersl_visi”. Šeit atkal jārēķinās, ka atsevišķām
mežaudžu kategorijām kopējais sķērslaukums ir 0 (piemēram, kailcirtēm),
tad aprēķinot proporciju šajos gadījumos sanāk saucējā 0, kas atgriež
“NaN” (not a number). Šīs mežaudzes nevar klasificēt tālā pēc būtības.
Pārbaudot ar str() šādas situācijas ir visās trīs izveidotajās mežaudžu
grupās.

``` r
centrs_kopa2$prop_skujkoki=centrs_kopa2$skersl_skujkoki/centrs_kopa2$skersl_visi
centrs_kopa2$prop_saurlapji=centrs_kopa2$skersl_saurlapji/centrs_kopa2$skersl_visi
centrs_kopa2$prop_platlapji=centrs_kopa2$skersl_platlapji/centrs_kopa2$skersl_visi

str(centrs_kopa2$prop_skujkoki)
```

    ##  num [1:505660] 0 0 NaN NaN NaN NaN NaN NaN 1 NaN ...

``` r
str(centrs_kopa2$prop_saurlapji)
```

    ##  num [1:505660] 0.545 1 NaN NaN NaN ...

``` r
str(centrs_kopa2$prop_platlapji)
```

    ##  num [1:505660] 0.455 0 NaN NaN NaN ...

Tagad beidzot var veikt klasifikāciju. Tam ieviesīšu jaunu lauku
“Klasifikacija”. Ja mežaudzē dominē kāda no trijām sugu grupām
(īpatsvars \>=0.75), tad tā tiks attiecīgi klasificēta. Ja tajā nedominē
neviena no šīm grupām, tad tā tiks klasificēta kā “Jauktu koku”.

``` r
centrs_kopa2$Klasifikacija=
  ifelse(centrs_kopa2$prop_skujkoki>=0.75,"Skujkoku",
  ifelse(centrs_kopa2$prop_saurlapji>=0.75,"Šaurlapju",
  ifelse(centrs_kopa2$prop_platlapji>=0.75,"Platlapju","Jauktu koku")))
```

Lai atbildētu uz jautājumu, Kāds ir katra veida mežu īpatsvars no visiem
ierakstiem, sākumā pārbaudu, vai klasifikācija ir izvevusies, atlasot
unikālās vērtības laukā “Klasifikacija”. Redzu, ka pastāv mežaudzes, kas
nav klasificētas. Augstāk jau paskaidroju, ka tas ir iespējams
atsevišķos gadījumos. Gribētu zināt cik tādu gadījumu ir, tāpēc tabulas
veidā apskatu mežaudžu skaitu katrā klasē. Secinu, ka neklasificētu
mežaudžu ir ļoti daudz.

``` r
unique(centrs_kopa2$Klasifikacija)
```

    ## [1] "Jauktu koku" "Šaurlapju"   NA            "Skujkoku"    "Platlapju"

``` r
table(centrs_kopa2$Klasifikacija, useNA = 'always')
```

    ## 
    ## Jauktu koku   Platlapju    Skujkoku   Šaurlapju        <NA> 
    ##       62901        4101      148046      141557      149055

Noslēgumā atbildot uz jautājumu, mežaudžu īpatsvari (%) katrā klasē ir
sekojoši:

``` r
data.frame(Klase=c('Skujkoku', 'Šaurlapju', 'Platlapju', 'Jauktu koku',"Bez kategorijas"),
          Īpatsvars_proc=                    c(round(length(which(centrs_kopa2$Klasifikacija=="Skujkoku"))/nrow(centrs_kopa2)*100,2),                                round(length(which(centrs_kopa2$Klasifikacija=="Šaurlapju"))/nrow(centrs_kopa2)*100,2),                    round(length(which(centrs_kopa2$Klasifikacija=="Platlapju"))/nrow(centrs_kopa2)*100,2),
            round(length(which(centrs_kopa2$Klasifikacija=="Jauktu koku"))/nrow(centrs_kopa2)*100,2),                  round(length(which(is.na(centrs_kopa2$Klasifikacija)))/nrow(centrs_kopa2)*100,2)))
```

    ##             Klase Īpatsvars_proc
    ## 1        Skujkoku          29.28
    ## 2       Šaurlapju          27.99
    ## 3       Platlapju           0.81
    ## 4     Jauktu koku          12.44
    ## 5 Bez kategorijas          29.48
