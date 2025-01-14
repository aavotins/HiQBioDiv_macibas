“Otrais uzdevums: vektordati, to ģeometrijas, atribūti un failu formāti”
================
Janis Ozols
2025-01-07

## Uzdevums

Vispirms sagatavosim darba vidi uzdevumu veikšanai. Lai veiktu uzdevumu
ir nepieciešams sagatavot darba direktoriju, sagatavot nepieciešamās
pakotnes un lejupielādēt nepieciešamos datus uzdevumu veikšanai. 1.
sagatavojam darba direktoriju

``` r
darba_direktorija <- dir.create(path = "C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd02\\centra_virsmezniecibas_dati")
```

2.  Iestatam sagatavoto darba direktoriju, sagatavoto darba direktoriju
    iestata pie knitr chunk ar sekojošajām fukcijām *setwd* iestata un
    *getwd* pārbaudam darba direktoriju

3.  sagatavojam nepieciešamās pakotnes uzdevumu veikšanai, ja
    nepieciešamas pakotnes installējam izmantojot *instal.packages()*,
    ja nav nepieciešams izmantojam *library()*, lai atvērtu
    nepieciešamās pakotnes

``` r
library(curl)
library(microbenchmark)
library(sfarrow)
library(tidyverse)
library(archive)
library(sf)
```

3.  lejupielādējam nepieciešamos datus

``` r
curl_download(url = "https://data.gov.lv/dati/lv/dataset/40014c0a-90f5-42be-afb2-fe3c4b8adf92/resource/392dfb67-eeeb-43c2-b082-35f9cf986128/download/centra.7z",destfile = "centra_virsmezniecibas_dati.7z")
```

4.  Atzipojam datus un apskatamies failu sarakstu

``` r
archive_extract("centra_virsmezniecibas_dati.7z")
```

``` r
list.files()
```

    ##  [1] "centra_virsmezniecibas_dati.7z" "nodala2651.cpg"                
    ##  [3] "nodala2651.dbf"                 "nodala2651.parquet"            
    ##  [5] "nodala2651.prj"                 "nodala2651.sbn"                
    ##  [7] "nodala2651.sbx"                 "nodala2651.shp"                
    ##  [9] "nodala2651.shp.xml"             "nodala2651.shx"                
    ## [11] "nodala2652.cpg"                 "nodala2652.dbf"                
    ## [13] "nodala2652.prj"                 "nodala2652.sbn"                
    ## [15] "nodala2652.sbx"                 "nodala2652.shp"                
    ## [17] "nodala2652.shp.xml"             "nodala2652.shx"                
    ## [19] "nodala2653.cpg"                 "nodala2653.dbf"                
    ## [21] "nodala2653.prj"                 "nodala2653.sbn"                
    ## [23] "nodala2653.sbx"                 "nodala2653.shp"                
    ## [25] "nodala2653.shp.xml"             "nodala2653.shx"                
    ## [27] "nodala2654.cpg"                 "nodala2654.dbf"                
    ## [29] "nodala2654.prj"                 "nodala2654.sbn"                
    ## [31] "nodala2654.sbx"                 "nodala2654.shp"                
    ## [33] "nodala2654.shp.xml"             "nodala2654.shx"                
    ## [35] "nodala2655.cpg"                 "nodala2655.dbf"                
    ## [37] "nodala2655.prj"                 "nodala2655.sbn"                
    ## [39] "nodala2655.sbx"                 "nodala2655.shp"                
    ## [41] "nodala2655.shp.xml"             "nodala2655.shx"                
    ## [43] "Uzd02_Ozols.Rmd"

#1. Izmantojot 2651. nodaļas datus (`nodala2651`), salīdziniet *ESRI shapefile*, *GeoPackage* un *geoparquet* failu:

- aizņemto diska vietu;

- ielasīšanas ātrumu vismaz 10 ielasīšanas izmēģinājumos.

1.  ielasam nodala2651 shapefile

``` r
shp2651 <- st_read("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd02\\centra_virsmezniecibas_dati\\nodala2651.shp")
```
2.  pārveidojam geopackage un geoparquet formātos

``` r
geopackage <- st_write(shp2651,dsn="C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd02\\centra_virsmezniecibas_dati\\nodala2651.gpkg")
```

``` r
geoparquet <- st_write_parquet(shp2651,dsn="C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd02\\centra_virsmezniecibas_dati\\nodala2651.parquet")
```

3.  nolasam failu izmērus

``` r
gpkgsize <- fs::file_size("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd02\\centra_virsmezniecibas_dati\\nodala2651.gpkg")
parquetsize <- fs::file_size("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd02\\centra_virsmezniecibas_dati\\nodala2651.parquet")
shpsize <- fs::file_size("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd02\\centra_virsmezniecibas_dati\\nodala2651.shp")
```
```r
gpkgsize
```
    ## 55.1M

``` r
parquetsize
```

    ## 21.1M

``` r
shpsize
```

    ## 25.8M

4.  ielasīsim katru failu 10 reizes, lai noteiktu vidējo ielasīšanas
    ātrumu

``` r
file_speed <- microbenchmark(st_read("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd02\\centra_virsmezniecibas_dati\\nodala2651.shp"),st_read("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd02\\centra_virsmezniecibas_dati\\nodala2651.gpkg"),st_read_parquet("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd02\\centra_virsmezniecibas_dati\\nodala2651.parquet"),times = 10)
```

``` r
rezultati_file_speed <- summary(file_speed)
rezultati_file_speed <- rezultati_file_speed[, c("mean", "neval")]
cat(sprintf("Vidējais: %.2f, Mēģinājumi: %.f\n\n", rezultati_file_speed$mean, rezultati_file_speed$neval))
```

    ## Vidējais: 5659.32, Mēģinājumi: 10
    ## 
    ##  Vidējais: 3619.91, Mēģinājumi: 10
    ## 
    ##  Vidējais: 951.21, Mēģinājumi: 10

#2. Apvienojiet visu Centra virzmežniecību nodaļu datus vienā slānī. Nodrošiniet,ka visas ģeometrijas ir `MULTIPOLYGON`, slānis nesatur tukšas vai nekorektas (*invalid*) ģeometrijas. 

1. Apvienojam visus shp failus vienā sarakstā

``` r
library(tidyverse)
shapefiles <- "C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd02\\centra_virsmezniecibas_dati" |>
  fs::dir_ls(recurse = TRUE, regexp = 'shp$')
shapefiles
```

    ## C:/Users/Ozols/Documents/HiQBioDiv_macibas/Uzd02/centra_virsmezniecibas_dati/nodala2651.shp
    ## C:/Users/Ozols/Documents/HiQBioDiv_macibas/Uzd02/centra_virsmezniecibas_dati/nodala2652.shp
    ## C:/Users/Ozols/Documents/HiQBioDiv_macibas/Uzd02/centra_virsmezniecibas_dati/nodala2653.shp
    ## C:/Users/Ozols/Documents/HiQBioDiv_macibas/Uzd02/centra_virsmezniecibas_dati/nodala2654.shp
    ## C:/Users/Ozols/Documents/HiQBioDiv_macibas/Uzd02/centra_virsmezniecibas_dati/nodala2655.shp

2.  apvienojam visus shp failus vienā datu rāmī

``` r
shpcentrs <- shapefiles |>
  map(st_read) |>
  bind_rows()
```

``` r
summary(shpcentrs)
```

    ##        id             objectid_1            adm1               adm2          
    ##  Min.   : 49299331   Length:505660      Length:505660      Length:505660     
    ##  1st Qu.: 51822812   Class :character   Class :character   Class :character  
    ##  Median :103724456   Mode  :character   Mode  :character   Mode  :character  
    ##  Mean   : 98253381                                                           
    ##  3rd Qu.:136314020                                                           
    ##  Max.   :149523080                                                           
    ##      atvk             kadastrs              gtf          kvart          
    ##  Length:505660      Length:505660      Min.   :1976   Length:505660     
    ##  Class :character   Class :character   1st Qu.:2012   Class :character  
    ##  Mode  :character   Mode  :character   Median :2017   Mode  :character  
    ##                                        Mean   :2016                     
    ##                                        3rd Qu.:2021                     
    ##                                        Max.   :2024                     
    ##      nog                anog              nog_plat         expl_mezs      
    ##  Length:505660      Length:505660      Min.   :  0.000   Min.   :  0.000  
    ##  Class :character   Class :character   1st Qu.:  0.390   1st Qu.:  0.350  
    ##  Mode  :character   Mode  :character   Median :  0.790   Median :  0.750  
    ##                                        Mean   :  1.248   Mean   :  1.132  
    ##                                        3rd Qu.:  1.530   3rd Qu.:  1.450  
    ##                                        Max.   :667.220   Max.   :154.280  
    ##    expl_celi         expl_gravj          zkat                mt           
    ##  Min.   :0.00000   Min.   :0.00000   Length:505660      Length:505660     
    ##  1st Qu.:0.00000   1st Qu.:0.00000   Class :character   Class :character  
    ##  Median :0.00000   Median :0.00000   Mode  :character   Mode  :character  
    ##  Mean   :0.01066   Mean   :0.02879                                        
    ##  3rd Qu.:0.00000   3rd Qu.:0.01000                                        
    ##  Max.   :1.73000   Max.   :4.22000                                        
    ##      izc                s10                 a10              h10       
    ##  Length:505660      Length:505660      Min.   :  0.00   Min.   : 0.00  
    ##  Class :character   Class :character   1st Qu.: 14.00   1st Qu.: 6.00  
    ##  Mode  :character   Mode  :character   Median : 45.00   Median :19.00  
    ##                                        Mean   : 50.75   Mean   :16.54  
    ##                                        3rd Qu.: 80.00   3rd Qu.:26.00  
    ##                                        Max.   :322.00   Max.   :51.00  
    ##       d10              g10               n10              bv10          
    ##  Min.   :  0.00   Min.   :   0.00   Min.   :    0.0   Length:505660     
    ##  1st Qu.:  7.00   1st Qu.:   0.00   1st Qu.:    0.0   Class :character  
    ##  Median : 19.00   Median :  14.00   Median :    0.0   Mode  :character  
    ##  Mean   : 18.51   Mean   :  13.35   Mean   :  436.8                     
    ##  3rd Qu.: 28.00   3rd Qu.:  22.00   3rd Qu.:    0.0                     
    ##  Max.   :381.00   Max.   :1135.00   Max.   :35000.0                     
    ##      ba10               s11                 a11              h11       
    ##  Length:505660      Length:505660      Min.   :  0.00   Min.   : 0.00  
    ##  Class :character   Class :character   1st Qu.:  0.00   1st Qu.: 0.00  
    ##  Mode  :character   Mode  :character   Median : 17.00   Median : 7.00  
    ##                                        Mean   : 34.18   Mean   :11.31  
    ##                                        3rd Qu.: 67.00   3rd Qu.:23.00  
    ##                                        Max.   :355.00   Max.   :50.00  
    ##       d11              g11               n11               bv11          
    ##  Min.   :   0.0   Min.   :  0.000   Min.   :    0.00   Length:505660     
    ##  1st Qu.:   0.0   1st Qu.:  0.000   1st Qu.:    0.00   Class :character  
    ##  Median :   7.0   Median :  0.000   Median :    0.00   Mode  :character  
    ##  Mean   :  12.8   Mean   :  3.005   Mean   :   51.13                     
    ##  3rd Qu.:  26.0   3rd Qu.:  6.000   3rd Qu.:    0.00                     
    ##  Max.   :3301.0   Max.   :764.000   Max.   :35000.00                     
    ##      ba11               s12                 a12              h12        
    ##  Length:505660      Length:505660      Min.   :  0.00   Min.   : 0.000  
    ##  Class :character   Class :character   1st Qu.:  0.00   1st Qu.: 0.000  
    ##  Mode  :character   Mode  :character   Median :  0.00   Median : 0.000  
    ##                                        Mean   : 16.54   Mean   : 5.395  
    ##                                        3rd Qu.: 14.00   3rd Qu.: 5.000  
    ##                                        Max.   :497.00   Max.   :46.000  
    ##       d12                g12               n12              bv12          
    ##  Min.   :   0.000   Min.   :  0.000   Min.   :   0.00   Length:505660     
    ##  1st Qu.:   0.000   1st Qu.:  0.000   1st Qu.:   0.00   Class :character  
    ##  Median :   0.000   Median :  0.000   Median :   0.00   Mode  :character  
    ##  Mean   :   6.274   Mean   :  0.896   Mean   :  12.58                     
    ##  3rd Qu.:   6.000   3rd Qu.:  0.000   3rd Qu.:   0.00                     
    ##  Max.   :4940.000   Max.   :509.000   Max.   :4000.00                     
    ##      ba12               s13                 a13              h13       
    ##  Length:505660      Length:505660      Min.   :  0.00   Min.   : 0.00  
    ##  Class :character   Class :character   1st Qu.:  0.00   1st Qu.: 0.00  
    ##  Mode  :character   Mode  :character   Median :  0.00   Median : 0.00  
    ##                                        Mean   :  4.97   Mean   : 1.58  
    ##                                        3rd Qu.:  0.00   3rd Qu.: 0.00  
    ##                                        Max.   :292.00   Max.   :36.00  
    ##       d13               g13               n13               bv13          
    ##  Min.   :  0.000   Min.   : 0.0000   Min.   :   0.000   Length:505660     
    ##  1st Qu.:  0.000   1st Qu.: 0.0000   1st Qu.:   0.000   Class :character  
    ##  Median :  0.000   Median : 0.0000   Median :   0.000   Mode  :character  
    ##  Mean   :  1.889   Mean   : 0.2001   Mean   :   1.931                     
    ##  3rd Qu.:  0.000   3rd Qu.: 0.0000   3rd Qu.:   0.000                     
    ##  Max.   :262.000   Max.   :17.0000   Max.   :2940.000                     
    ##      ba13               s14                 a14               h14         
    ##  Length:505660      Length:505660      Min.   :  0.000   Min.   : 0.0000  
    ##  Class :character   Class :character   1st Qu.:  0.000   1st Qu.: 0.0000  
    ##  Mode  :character   Mode  :character   Median :  0.000   Median : 0.0000  
    ##                                        Mean   :  1.053   Mean   : 0.3286  
    ##                                        3rd Qu.:  0.000   3rd Qu.: 0.0000  
    ##                                        Max.   :224.000   Max.   :35.0000  
    ##       d14                g14                n14               bv14          
    ##  Min.   :  0.0000   Min.   : 0.00000   Min.   :  0.0000   Length:505660     
    ##  1st Qu.:  0.0000   1st Qu.: 0.00000   1st Qu.:  0.0000   Class :character  
    ##  Median :  0.0000   Median : 0.00000   Median :  0.0000   Mode  :character  
    ##  Mean   :  0.4042   Mean   : 0.03427   Mean   :  0.2089                     
    ##  3rd Qu.:  0.0000   3rd Qu.: 0.00000   3rd Qu.:  0.0000                     
    ##  Max.   :285.0000   Max.   :11.00000   Max.   :800.0000                     
    ##      ba14               jakopj         jaatjauno        p_darbv         
    ##  Length:505660      Min.   :   0.0   Min.   :   0.0   Length:505660     
    ##  Class :character   1st Qu.:   0.0   1st Qu.:   0.0   Class :character  
    ##  Mode  :character   Median :   0.0   Median :   0.0   Mode  :character  
    ##                     Mean   : 189.1   Mean   : 119.1                     
    ##                     3rd Qu.:   0.0   3rd Qu.:   0.0                     
    ##                     Max.   :2034.0   Max.   :2034.0                     
    ##     p_darbg        p_cirp              p_cirg        atj_gads   
    ##  Min.   :   0   Length:505660      Min.   :   0   Min.   :   0  
    ##  1st Qu.:   0   Class :character   1st Qu.:   0   1st Qu.:   0  
    ##  Median :2005   Mode  :character   Median :2003   Median :   0  
    ##  Mean   :1128                      Mean   :1144   Mean   : 651  
    ##  3rd Qu.:2020                      3rd Qu.:2016   3rd Qu.:2003  
    ##  Max.   :2024                      Max.   :8003   Max.   :2024  
    ##    saimn_d_ie     plant_audz         forestry_c         vmd_headfo       
    ##  Min.   :1.000   Length:505660      Length:505660      Length:505660     
    ##  1st Qu.:6.000   Class :character   Class :character   Class :character  
    ##  Median :6.000   Mode  :character   Mode  :character   Mode  :character  
    ##  Mean   :5.529                                                           
    ##  3rd Qu.:6.000                                                           
    ##  Max.   :6.000                                                           
    ##    shape_Leng        shape_Area               geometry     
    ##  Min.   :    0.0   Min.   :      0   MULTIPOLYGON :505660  
    ##  1st Qu.:  301.0   1st Qu.:   3852   epsg:3059    :     0  
    ##  Median :  441.7   Median :   7947   +proj=tmer...:     0  
    ##  Mean   :  515.6   Mean   :  12485                         
    ##  3rd Qu.:  635.4   3rd Qu.:  15343                         
    ##  Max.   :52009.0   Max.   :6672216

3.  Pārbaudam, ka visas ģeometrijas ir multipoligoni

``` r
shpcentrs <- shpcentrs %>% mutate(geometry = st_cast(geometry, "MULTIPOLYGON"))
```

4.  pārbaudam vai starp ģeometrijām ir nekorektas, vai tukšas
    ģeometrijas

``` r
invalid_geometrijas <- shpcentrs %>% 
  filter(!st_is_valid(geometry))
print(nrow(invalid_geometrijas))
```

    ## [1] 274

``` r
tuksas_geometrijas <- st_is_empty(shpcentrs)
print(sum(tuksas_geometrijas))
```

    ## [1] 6

5.  labojam nekorektās ģeometrijas un pārbaudam vai salabotas

``` r
shpcentrs <- st_make_valid(shpcentrs)
invalid_geometrijas <- shpcentrs %>% 
  filter(!st_is_valid(geometry))
print(nrow(invalid_geometrijas))
```

    ## [1] 0

6.  ģeometrijas salabotas, izņemam ārā tukšās ģeometrijas un
    pārliecināmies, ka to vairs nav

``` r
shpcentrs <- shpcentrs[!tuksas_geometrijas,]
tuksas_geometrijas <- st_is_empty(shpcentrs)
print(sum(tuksas_geometrijas))
```

    ## [1] 0

#3. Apvienotajā slānī aprēķiniet priežu (kumulatīvo dažādām sugām) pirmā stāva šķērslaukumu īpatsvaru, kuru saglabājiet laukā prop_priedes. Laukā PriezuMezi ar vērtību “1” atzīmējiet tās mežaudzes, kurās priedes šķērslaukuma īpatsvars pirmajā stāvā ir vismaz 75% un ar “0” tās mežaudzes, kurās īpatsvars ir mazāks, pārējos ierakstus atstājot bez vērtībām. Kāds ir priežu mežaudžu īpatsvars no visām mežaudzēm?

1.  Aprēķinam priežu šķērslaukumu, izvēloties s1 (parastā priede), s14
    (citas priedes) un s22 (ciedru priede)

``` r
shpcentrs <- shpcentrs %>% mutate (priedes_skerslaukums = 
ifelse(s10 %in% c("1", "14", "22"), g10, 0) +
ifelse(s11 %in% c("1", "14", "22"), g11, 0) +
ifelse(s12 %in% c("1", "14", "22"), g12, 0) +
ifelse(s13 %in% c("1", "14", "22"), g13, 0) +
ifelse(s14 %in% c("1", "14", "22"), g14, 0))
```

2.  Aprēķinam kopējo šķērslaukumu

``` r
shpcentrs <- shpcentrs %>% mutate (kop_skerslaukums = g10+g11+g12+g13+g14)
```

3.  Aprēķinam priežu šķērslaukuma proporciju

``` r
shpcentrs <- shpcentrs %>% mutate (priedes_prop = priedes_skerslaukums/kop_skerslaukums)
```

4.  Atzīmējam mežaudzes, kurās priežu īpatsvars 1. stāvā ir 75% un
    lielāks

``` r
shpcentrs <- shpcentrs %>%
  mutate(
    PriezuMezi = case_when(
      priedes_prop >= 0.75 ~ 1,
      priedes_prop < 0.75 & !is.na(priedes_prop) ~ 0,
      TRUE ~ NA_real_
    )
  )
```

5.  noskaidrojam, cik mežaudzes klasificējas kā priežu mežs un izsakam
    attiecību

``` r
table(shpcentrs$PriezuMezi)
```

    ## 
    ##      0      1 
    ## 266396  90205

``` r
priezu_mezi <- 90205/505660
print(priezu_mezi)
```

    ## [1] 0.1783906

6.  17,8% klasificējas kā priežu mežu nogabali, bet apskatam kāda ir
    priežu mežu īpatsvars no kopējās mezu platības

``` r
aggregate(expl_mezs ~ PriezuMezi, data=shpcentrs, sum)
```

    ##   PriezuMezi expl_mezs
    ## 1          0  281182.5
    ## 2          1  138334.2

``` r
priezu_mezu_platiba <-138334.2
kopeja_meza_platiba <- sum(shpcentrs$expl_mezs)
priezu_mezu_platibas_ipatsvars <- priezu_mezu_platiba/kopeja_meza_platiba
print(priezu_mezu_platibas_ipatsvars)
```

    ## [1] 0.2417067

Priežu meži sastāda 24,17% jeb 138334 ha no kopējās mežu platības centra
virsmezniecībā 
#4. Apvienotajā slānī, izmantojot informāciju par pirmā stāva koku sugām un to šķērslaukumiem, veiciet mežaudžu klasifikāciju skujkoku, šaurlapju, platlapju un jauktu koku mežos. Paskaidrojiet izmantoto pieeju un izvēlētos robežlielumus. Kāds ir katra veida mežu īpatsvars no visiem ierakstiem? 1. izveidosim mezaudzu klasifikaciju sadalot kokus grupās - platlapji, saurlapji un skujkoki, izmantosim tikai vietējās un pirmā stāva mežu sugas

``` r
Mezaudzu_tips <- list(skujkoki = c(1, 3), platlapju_koki = c(10, 11, 12, 16, 17, 18, 20, 24), saurlapju_koki =c(4, 6, 8, 9))
```

2.  tabulai pievienosim laukus šaurlapju šķērslaukums, platlapju
    šķērslaukums un skujkoku šķērslaukums

``` r
shpcentrs <- shpcentrs %>% mutate (skujkoku_skerslaukums = 
ifelse(s10 %in% Mezaudzu_tips$skujkoki, g10, 0) +
ifelse(s11 %in% Mezaudzu_tips$skujkoki, g11, 0) +
ifelse(s12 %in% Mezaudzu_tips$skujkoki, g12, 0) +
ifelse(s13 %in% Mezaudzu_tips$skujkoki, g13, 0) +
ifelse(s14 %in% Mezaudzu_tips$skujkoki, g14, 0),
platlapju_skerslaukums = 
ifelse(s10 %in% Mezaudzu_tips$platlapju_koki, g10, 0) +
ifelse(s11 %in% Mezaudzu_tips$platlapju_koki, g11, 0) +
ifelse(s12 %in% Mezaudzu_tips$platlapju_koki, g12, 0) +
ifelse(s13 %in% Mezaudzu_tips$platlapju_koki, g13, 0) +
ifelse(s14 %in% Mezaudzu_tips$platlapju_koki, g14, 0),
saurlapju_skerslaukums = 
ifelse(s10 %in% Mezaudzu_tips$saurlapju_koki, g10, 0) +
ifelse(s11 %in% Mezaudzu_tips$saurlapju_koki, g11, 0) +
ifelse(s12 %in% Mezaudzu_tips$saurlapju_koki, g12, 0) +
ifelse(s13 %in% Mezaudzu_tips$saurlapju_koki, g13, 0) +
ifelse(s14 %in% Mezaudzu_tips$saurlapju_koki, g14, 0))
```

3.  aprēķinām katra meža tipa proporcijas

``` r
shpcentrs <- shpcentrs %>% mutate (mezaudze = case_when(kop_skerslaukums == 0 ~ "nav mezs",skujkoku_skerslaukums/kop_skerslaukums >= 0.90 ~ "skujkoku mezs", saurlapju_skerslaukums/kop_skerslaukums >= 0.90 ~ "saurlapju mezs", platlapju_skerslaukums/kop_skerslaukums >= 0.90 ~ "platlapju mezs", TRUE ~ "Jaukts mezs"))
```

4.  noskaidrojam meža veidu īpatsvaru pēc nogabalu skaita, izsakam
    proporciju

``` r
mezaudzes_skaits <- table(shpcentrs$mezaudze)
mezaudzu_proporcija <- mezaudzes_skaits/sum(mezaudzes_skaits)
print(mezaudzu_proporcija)
```

    ## 
    ##    Jaukts mezs       nav mezs platlapju mezs saurlapju mezs  skujkoku mezs 
    ##    0.250455845    0.294772710    0.005209096    0.215952015    0.233610334

5.  noskaidrojam mežaudžu veidu platību īpatsvaru

``` r
meza_platiba <- sum(shpcentrs$expl_mezs)
mezu_tipu_platibas <- aggregate(expl_mezs ~ mezaudze, data=shpcentrs, sum)
mezaudzu_area_proporcija <- mezu_tipu_platibas$expl_mezs/sum(mezu_tipu_platibas$expl_mezs)
mezu_tipu_platibas <- mezu_tipu_platibas %>% mutate ( mezaudzu_skaits = mezaudzes_skaits, mezaudzu_platibas_proporcija = mezaudzu_area_proporcija, mezaudzu_skaita_proporcija = mezaudzu_proporcija)
print(mezu_tipu_platibas)
```

    ##         mezaudze expl_mezs mezaudzu_skaits mezaudzu_platibas_proporcija
    ## 1    Jaukts mezs 141746.44          126644                    0.2476688
    ## 2       nav mezs 152805.84          149053                    0.2669925
    ## 3 platlapju mezs   1844.71            2634                    0.0032232
    ## 4 saurlapju mezs 103721.63          109197                    0.1812293
    ## 5  skujkoku mezs 172203.93          118126                    0.3008862
    ##   mezaudzu_skaita_proporcija
    ## 1                0.250455845
    ## 2                0.294772710
    ## 3                0.005209096
    ## 4                0.215952015
    ## 5                0.233610334

# Izvēlētā pieeja

Mezaudzes tipos ir izvēlētas sadalīt izmantojot *case_when()*, jo tā
vienkārši un loģiski ļauj sadalīt datus pēc to vērtībām, kā robežvērtība
ir izmantota 90%, kas ir izvēlēta tādēļ, ka pētījumi parāda, ka
saproksilofāgo vaboļu daudzveidību būtiski atšķiras atkarībā no
mežaudzes daudzveidības, turklāt, daudzām sugām, kurām ir šaura
ekoloģiskā niša, ir nepieciešamas monodominantas audzes, kur to
sastopamību negatīvi ietekmē jau neliels citu koku sugu piejaukums.
Papildus ir izvēlēts apskatīt mežaudžu tipu attiecību centra
virsmežniecībā ne tikai pēc skaita, bet arī pēc platības, jo daudzām
sugām ir svarīgi, lai piemērotā mežaudze būtu lielās platībās. Kā
redzams tabulā pēc skaita visvairāķ ir nogabali, kuros nav 1. stāva -
jaunaudzes un izcirtumi, pēc tam ir jauktu mežu nogabali, bet aprēķinot
pēc platības, vislielākā meža platība proporcionāli ir skujkoku meža
tipa. Kas norāda, ka lai gan nogabalu skaits ir mazāks, tie ir
monodominantāki un visdrīzāk nogabalu vidējā platība ir lielāku par citu
mežaudžu tipu nogabalu platībām.
