Otrais uzdevums: vektordati, to ģeometrijas, atribūti un failu formāti
================

\#1. uzdevums 1.1 faila lejuplāde un izvēršana vēlamajā vietā

``` r
setwd("C:/Users/vinet/Documents/VPP/2uzd")
centra_url <- "https://data.gov.lv/dati/lv/dataset/40014c0a-90f5-42be-afb2-fe3c4b8adf92/resource/392dfb67-eeeb-43c2-b082-35f9cf986128/download/centra.7z"
centra <- "C:/Users/vinet/Documents/VPP/2uzd/centra.7z"
download.file(centra_url, destfile = centra, mode = "wb", method="curl")

if(!require(archive)){
    install.packages("archive")
    library(archive)
    }
centra_vieta <- "C:/Users/vinet/Documents/VPP/2uzd/centra"
archive_extract(centra, dir =centra_vieta)
```

1.2 ģeotelpisko datu ielasīšana R

``` r
if(!require(sf)){
    install.packages("sf")
    library(sf)
    }
centra_shp_vieta <-"C:/Users/vinet/Documents/VPP/2uzd/centra/nodala2651.shp" 
centra_shp <-st_read(centra_shp_vieta, quiet = TRUE)
```

1.3 ESRI shapefile konvertēšana uz GeoPackage tipa failu un tā
ielasīšana

``` r
centra_gpkg_vieta <-"C:/Users/vinet/Documents/VPP/2uzd/centra.gpkg" 
centra_gpkg <- "centra"
st_write(centra_shp, dsn = centra_gpkg_vieta, layer = centra_gpkg,
driver = "GPKG", quiet = TRUE) 
centra_gpkg <- st_read(dsn = centra_gpkg_vieta, layer
= centra_gpkg, quiet = TRUE)
```

1.4 ESRI shapefile konvertēšana uz geoparquet tipa failu un tā
ielasīšana

``` r
if(!require(sfarrow)){ install.packages("sfarrow")
library(sfarrow) } 
centra_gpq_vieta <-"C:/Users/vinet/Documents/VPP/2uzd/centra.parquet"
st_write_parquet(centra_shp, centra_gpq_vieta) 
centra_gpq <-st_read_parquet(centra_gpq_vieta, quiet = TRUE)
```

1.5 izmers shp

``` r
centra_shp_info <- file.info(centra_shp_vieta)
centra_shp_size_mb <- centra_shp_info$size / (1024^2)
centra_shx_vieta <- "C:/Users/vinet/Documents/VPP/2uzd/centra/nodala2651.sbx"
centra_shx_info <- file.info(centra_shx_vieta)
centra_shx_size_mb <- centra_shx_info$size / (1024^2)
centra_dbf_vieta <- "C:/Users/vinet/Documents/VPP/2uzd/centra/nodala2651.dbf"
centra_dbf_info <- file.info(centra_dbf_vieta)
centra_dbf_size_mb <-centra_dbf_info$size / (1024^2)

centra_shapef_size_mb <- sum(centra_shp_size_mb, centra_shx_size_mb,
centra_dbf_size_mb)
cat("File size in MB:", centra_shapef_size_mb)
```

    ## File size in MB: 775.8615

1.6 izmērs gpkg

``` r
centra_gpkg_info <- file.info(centra_gpkg_vieta)
centra_gpkg_size_mb <- centra_gpkg_info$size / (1024^2)
cat("File size in MB:", centra_gpkg_size_mb)
```

    ## File size in MB: 55.11719

1.7 izmērs parquet

``` r
centra_gpq_info <- file.info(centra_gpq_vieta)
centra_gpq_size_mb <- centra_gpq_info$size / (1024^2)
cat("File size in MB:", centra_gpq_size_mb)
```

    ## File size in MB: 21.08482

1.8 ielasīšanas laika noteikšana shp

``` r
if(!require(microbenchmark)){ install.packages("microbenchmark")
library(microbenchmark) }

ielasisanas_atr_shp <- microbenchmark(
  read_shapefile = {
    centra_shp <- st_read(centra_shp_vieta, quiet = TRUE)
  },
  times = 10
)
print(ielasisanas_atr_shp)
```

    ## Unit: seconds
    ##            expr      min       lq     mean   median       uq      max neval
    ##  read_shapefile 4.350744 4.752861 4.783402 4.813432 4.881288 4.926171    10

``` r
ielasisanas_atr_gpkg <- microbenchmark(
read_gpkg = { 
centra_gpkg <-st_read(dsn = centra_gpkg_vieta, layer = "centra", quiet = TRUE) 
},
times = 10 )
print(ielasisanas_atr_gpkg)
```

    ## Unit: seconds
    ##       expr     min       lq     mean   median       uq      max neval
    ##  read_gpkg 2.86021 3.190228 3.206601 3.248781 3.276101 3.288883    10

``` r
ielasisanas_atr_gpq <- microbenchmark(
read_gpq = {
centra_gpq <-st_read_parquet(centra_gpq_vieta, quiet = TRUE)
},
times = 10 )

print(ielasisanas_atr_gpq)
```

    ## Unit: milliseconds
    ##      expr      min       lq     mean   median       uq      max neval
    ##  read_gpq 781.7266 794.5586 1052.143 1108.806 1175.151 1322.421    10

1.uzd. ATBILDE: ESRI shapefile izmērs - 775,86 MB, ielasīšanas
vid.ātr. - 4,97s; GeoPackage izmērs - 55,12 MB, ielasīšanas vid. ātr.
-3,25s; geoparquet izmērs - 21,08 MB, ielasīšanas vid. ātr. - 1,16s.

\#2. uzdevums 2.1 Visu nodaļu shapefile ielasīšana un apvienošana

``` r
centra_shp_vieta1 <-"C:/Users/vinet/Documents/VPP/2uzd/centra/nodala2651.shp"
centra_shp1 <- st_read(centra_shp_vieta1, quiet = TRUE)
centra_shp_vieta2 <-"C:/Users/vinet/Documents/VPP/2uzd/centra/nodala2652.shp" 
centra_shp2<- st_read(centra_shp_vieta2, quiet = TRUE) 
centra_shp_vieta3 <-"C:/Users/vinet/Documents/VPP/2uzd/centra/nodala2653.shp" 
centra_shp3<- st_read(centra_shp_vieta3, quiet = TRUE) 
centra_shp_vieta4 <-"C:/Users/vinet/Documents/VPP/2uzd/centra/nodala2654.shp" 
centra_shp4<- st_read(centra_shp_vieta4, quiet = TRUE) 
centra_shp_vieta5 <-"C:/Users/vinet/Documents/VPP/2uzd/centra/nodala2655.shp" 
centra_shp5<- st_read(centra_shp_vieta5, quiet = TRUE) 
apv_centra_shp <- rbind(centra_shp1,centra_shp2, centra_shp3, centra_shp4, centra_shp5)
```

2.2 Ģeometriju pārbaude

``` r
geometry_types <- unlist(st_geometry_type(apv_centra_shp)) 
all_multipolygon <- all(geometry_types == "MULTIPOLYGON")
print(all_multipolygon)
```

    ## [1] TRUE

``` r
empty_geometries <- st_is_empty(apv_centra_shp)
invalid_geometries <-st_is_valid(apv_centra_shp)

cat("Number of empty geometries:", sum(empty_geometries), "\n")
```

    ## Number of empty geometries: 6

``` r
cat("Number of invalid geometries:", sum(!invalid_geometries))
```

    ## Number of invalid geometries: 274

2.3 Tukšo un nekorekto ģeometriju noņemšana

``` r
apv_centra_shp2 <-apv_centra_shp[!empty_geometries & invalid_geometries, ] 
```

\#3. uzdevums 3.1 Papildus lauku izveidi aprēķinu veikšanai

``` r
s10 <- apv_centra_shp2$s10
s11 <- apv_centra_shp2$s11 
s12 <- apv_centra_shp2$s12
s13 <- apv_centra_shp2$s13 
s14 <- apv_centra_shp2$s14

g10 <- apv_centra_shp2$g10
g11 <- apv_centra_shp2$g11 
g12 <- apv_centra_shp2$g12
g13 <- apv_centra_shp2$g13 
g14 <- apv_centra_shp2$g14

if(!require(dplyr)){
    install.packages("dplyr")
    library(dplyr)
    }

apv_centra_shp2 <- apv_centra_shp2 %>%
  mutate(priedes_g10 = ifelse(s10 %in% c(1, 14, 22), g10, 0))
apv_centra_shp2 <- apv_centra_shp2 %>%
  mutate(priedes_g11 = ifelse(s11 %in% c(1, 14, 22), g11, 0))
apv_centra_shp2 <- apv_centra_shp2 %>%
  mutate(priedes_g12 = ifelse(s12 %in% c(1, 14, 22), g12, 0))
apv_centra_shp2 <- apv_centra_shp2 %>%
  mutate(priedes_g13 = ifelse(s13 %in% c(1, 14, 22), g13, 0))
apv_centra_shp2 <- apv_centra_shp2 %>%
  mutate(priedes_g14 = ifelse(s14 %in% c(1, 14, 22), g14, 0))
```

3.2 Atribūtu lauka prop_priedes izveide

``` r
apv_centra_shp2 <- apv_centra_shp2 %>%
  mutate(
    prop_priedes = (priedes_g10 + priedes_g11 + priedes_g12 + priedes_g13 + priedes_g14) / 
                   (g10 + g11 + g12 + g13 + g14)
  )
```

3.3 NaN vērtību aizvietošana ar 0, jo tur priežu īpatsvars ir 0

``` r
apv_centra_shp2$prop_priedes[is.nan(apv_centra_shp2$prop_priedes) | 
                             is.na(apv_centra_shp2$prop_priedes) | 
                             is.infinite(apv_centra_shp2$prop_priedes)] <- 0
```

3.4 Attribūtu lauka PriezuMezi izveide

``` r
apv_centra_shp2$PriezuMezi <- ifelse(apv_centra_shp2$prop_priedes >=0.75, "1", "0")
```

3.5 Priežu mežu īpatsvara aprēķins

``` r
priezu_m_ipatsv <-sum(apv_centra_shp2$PriezuMezi == 1)/length(apv_centra_shp2$PriezuMezi)
print(priezu_m_ipatsv) 
```

    ## [1] 0.1783628

3.uzd. atbilde: priežu mežaudžuīpatsvars ir 0,17 jeb 17,8%

\#4. uzdevums 4.1 No sākuma, lai atvieglotu darbu, noskaidroju kādas
koku sugas vispār ir esošajās mežaudzēs.Konstatēju, ka sugas ar kodu 65
un 69 nav mežaudzēs, tāpēc tās tālāk neiekļāvu.

``` r
sugu_saraksts <- unique(c(apv_centra_shp2$s10, apv_centra_shp2$s11, apv_centra_shp2$s12, apv_centra_shp2$s13, apv_centra_shp2$s14))
print(sugu_saraksts)
```

    ##  [1] "16" "9"  "0"  "8"  "1"  "4"  "6"  "10" "3"  "11" "20" "24" "21" "32" "19"
    ## [16] "12" "25" "13" "64" "23" "17" "15" "63" "14" "35" "61" "50" "68" "67" "22"
    ## [31] "27" "29" "66" "18" "26" "62"

4.2 Tālāk sadalīju pārējās koku sugas 3 kategorijās - skujkoku,
šaurlapju un platlapju. Pie skujkokiem iekļāvu Pinaceae un Taxaceae
dzimtas pārstāvjus, pie šaurlapjiem - Betuloideae apakšdzimtas,
Salicaceae dzimtas un pie platlapjem visus pārējos. Iedalījumu balstīju
uz sugu aprakstiem un Nacionālās enciklopēdijas informācijas par mežu
ekosistēmām. Dažām koku sugām gan nebija 100% skaidrs, kur labāk tas
likt.

``` r
skujkoki <- c(1, 3, 13, 14, 15, 22, 23, 28, 29)
saurlapji <- c(4, 6, 8, 9, 19, 20, 21)
platlapji <- c(10, 11, 12, 16, 17, 18, 24, 25, 26, 27, 32, 35, 50, 61, 62, 63, 64, 66, 67, 68)
```

4.3 Sasummēju katrā mežaudzē skujoku, šaurlapju un platlapju
šķērslaukumus

``` r
apv_centra_shp2$skujk_g <- rowSums(data.frame(
  ifelse(apv_centra_shp2$s10 %in% skujkoki, apv_centra_shp2$g10, 0),
  ifelse(apv_centra_shp2$s11 %in% skujkoki, apv_centra_shp2$g11, 0),
  ifelse(apv_centra_shp2$s12 %in% skujkoki, apv_centra_shp2$g12, 0),
  ifelse(apv_centra_shp2$s13 %in% skujkoki, apv_centra_shp2$g13, 0),
  ifelse(apv_centra_shp2$s14 %in% skujkoki, apv_centra_shp2$g14, 0)
))

apv_centra_shp2$saurl_g <- rowSums(data.frame(
  ifelse(apv_centra_shp2$s10 %in% saurlapji, apv_centra_shp2$g10, 0),
  ifelse(apv_centra_shp2$s11 %in% saurlapji, apv_centra_shp2$g11, 0),
  ifelse(apv_centra_shp2$s12 %in% saurlapji, apv_centra_shp2$g12, 0),
  ifelse(apv_centra_shp2$s13 %in% saurlapji, apv_centra_shp2$g13, 0),
  ifelse(apv_centra_shp2$s14 %in% saurlapji, apv_centra_shp2$g14, 0)
))

apv_centra_shp2$platl_g <- rowSums(data.frame(
  ifelse(apv_centra_shp2$s10 %in% platlapji, apv_centra_shp2$g10, 0),
  ifelse(apv_centra_shp2$s11 %in% platlapji, apv_centra_shp2$g11, 0),
  ifelse(apv_centra_shp2$s12 %in% platlapji, apv_centra_shp2$g12, 0),
  ifelse(apv_centra_shp2$s13 %in% platlapji, apv_centra_shp2$g13, 0),
  ifelse(apv_centra_shp2$s14 %in% platlapji, apv_centra_shp2$g14, 0)
))
```

4.4 Izveidoju jaunu attribūtu, kurā klasificēju mežaudzi vienā no 4
mežaudžu veidiem. Par skujkoku mežaudzēm uzskatīju tādas, kur skujkoku
šķērsgriezumu summa veido lielāko daļu (līdzīgi kā iepriekšējā uzd
izmantojot 75% kā slieksni). Līdzīgi rīkojos arī ar šaurlapju un
platlapju mežaudzēm. Pārējās, kurās bija vairāku koku veidu pārstāvji,
klasificēju kā jauktu koku mežaudzes.

``` r
apv_centra_shp2$kopejais_g <- g10 + g11 + g12 + g13 + g14
apv_centra_shp2$meza_veids <- case_when(
  apv_centra_shp2$skujk_g == 0 & apv_centra_shp2$saurl_g == 0 & apv_centra_shp2$platl_g == 0 ~ NA,
  apv_centra_shp2$platl_g / apv_centra_shp2$kopejais_g >= 0.75 ~ "platlapju",
  apv_centra_shp2$saurl_g / apv_centra_shp2$kopejais_g >= 0.75 ~ "šaurlapju",
  apv_centra_shp2$skujk_g / apv_centra_shp2$kopejais_g >= 0.75 ~ "skujkoku",
  TRUE ~ "jauktu koku"
)
nenotiektie <- sum(is.na(apv_centra_shp2$meza_veids))
```

Ievēroju, ka ir mežaudzes, kurās šķērslaukumu summa ir 0. Noskaidroju,
ka šādu gadījumu ir daudz tāpēc ņēmu to vērā klasifikācijā.

4.5 Aprēķināju katra mežaudzes veida īpatsvaru

``` r
skujk_ipatsv <- sum(apv_centra_shp2$meza_veids == "skujkoku", na.rm = TRUE) / length(apv_centra_shp2$meza_veids)
saurl_ipatsv <- sum(apv_centra_shp2$meza_veids == "šaurlapju", na.rm = TRUE) / length(apv_centra_shp2$meza_veids)
platl_ipatsv <- sum(apv_centra_shp2$meza_veids == "platlapju", na.rm = TRUE) / length(apv_centra_shp2$meza_veids)
jk_ipatsv <- sum(apv_centra_shp2$meza_veids == "jauktu koku", na.rm = TRUE) / length(apv_centra_shp2$meza_veids)
nenoteikto_ipatsv <- sum(is.na(apv_centra_shp2$meza_veids))/length(apv_centra_shp2$meza_veids)

mezu_klasifikacija_table <- data.frame(
  `Mežaudžu klasifikācija` = c("Skujkoku", "Šaurlapju", "Platlapju", "Jauktu koku", "Nenoteikta"),
  `Īpatsvars` = c(round(skujk_ipatsv, 2), round(saurl_ipatsv, 2), round(platl_ipatsv, 2), round(jk_ipatsv, 2), round(nenoteikto_ipatsv, 2))
)
print(mezu_klasifikacija_table)
```

    ##   Mežaudžu.klasifikācija Īpatsvars
    ## 1               Skujkoku      0.29
    ## 2              Šaurlapju      0.28
    ## 3              Platlapju      0.01
    ## 4            Jauktu koku      0.12
    ## 5             Nenoteikta      0.29

4.  uzd. atbilde: Skujkoku mežaudžu īpatsvars ir 0,29, šaurlapju - 0,28,
    platlapju - 0,01, jauktu koku 0,12, nenoteikto - 0,29.
