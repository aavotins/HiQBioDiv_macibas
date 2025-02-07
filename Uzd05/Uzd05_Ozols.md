Uzd05_Ozols
================
Janis Ozols
2025-01-21

## Uzdevums

Šim uzdevumam ir kopīga sagatavošanās, tomēr tā sekojošās daļas ved pie
viena un tā paša rezultāta, izmantojot dažādus vingrinājumus. Tāds arī
ir tā uzdevums - vingrināt izpratni un intuīciju komandu rindās, R un
darbā ar telpiskiem datiem.

Šim uzdevumam izmantojami pirmajā uzdevumā apvienotie Valsts meža
dienesta Centra virsmežniecības dažādo nodaļu apvienotie MVR dati un
projekta [Zenodo
repozitorijā](https://zenodo.org/communities/hiqbiodiv/records?q=&l=list&p=1&s=10&sort=newest)
pieejamie references dati (gan vektoru, gan rastra). Sakot darbu, brīvi
izvēlieties (vienalga kā, vienalga kurā programmā) četras blakus esošas
`tks93_50km` karšu lapas, kurās ir Centra virsmežniecības MVR dati par
mežaudzēm - šīs lapas ievietojiet atsevišķā R objektā un izmantojiet
turpmākam darbam kā neatkarīgas telpiskās daļas iteratīvā aprēķinu
procesā. 1. Sagatavojam vidi atverot nepieciešamās paketes

``` r
library(sf)
library(arrow)
library(sfarrow)
library(tidyverse)
library(terra)
library(tools)
library(raster)
library(fasterize)
library(units)
```

2.  Ielasam failu

``` r
lapas <- st_read_parquet(dsn = "C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\tks93_50km.parquet")
summary(lapas)
```

    ##   NOSAUKUMS             NUMURS      Shape_Length      Shape_Area       
    ##  Length:131         Min.   :2434   Min.   : 88000   Min.   :420000001  
    ##  Class :character   1st Qu.:3332   1st Qu.:100000   1st Qu.:624999978  
    ##  Mode  :character   Median :4112   Median :100000   Median :625000000  
    ##                     Mean   :3865   Mean   :100901   Mean   :635839694  
    ##                     3rd Qu.:4342   3rd Qu.:100000   3rd Qu.:625000025  
    ##                     Max.   :5411   Max.   :110000   Max.   :750000020  
    ##            Shape    
    ##  MULTIPOLYGON :131  
    ##  epsg:3059    :  0  
    ##  +proj=tmer...:  0  
    ##                     
    ##                     
    ## 

``` r
plot(lapas["NUMURS"])
```

![](Uzd05_Ozols_files/figure-gfm/tks93_50km%20lapas-1.png)<!-- --> 3.
ielasam apvienoto mvr failu

``` r
meza_centra_dati <- st_read_parquet("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd04\\mezicentrs.parquet")
summary(meza_centra_dati)
```

    ##        id             objectid_1            adm1               adm2          
    ##  Min.   : 49299331   Length:505654      Length:505654      Length:505654     
    ##  1st Qu.: 51822913   Class :character   Class :character   Class :character  
    ##  Median :103724956   Mode  :character   Mode  :character   Mode  :character  
    ##  Mean   : 98253948                                                           
    ##  3rd Qu.:136314295                                                           
    ##  Max.   :149523080                                                           
    ##      atvk             kadastrs              gtf          kvart          
    ##  Length:505654      Length:505654      Min.   :1976   Length:505654     
    ##  Class :character   Class :character   1st Qu.:2012   Class :character  
    ##  Mode  :character   Mode  :character   Median :2017   Mode  :character  
    ##                                        Mean   :2016                     
    ##                                        3rd Qu.:2021                     
    ##                                        Max.   :2024                     
    ##      nog                anog              nog_plat         expl_mezs      
    ##  Length:505654      Length:505654      Min.   :  0.000   Min.   :  0.000  
    ##  Class :character   Class :character   1st Qu.:  0.390   1st Qu.:  0.350  
    ##  Mode  :character   Mode  :character   Median :  0.790   Median :  0.750  
    ##                                        Mean   :  1.248   Mean   :  1.132  
    ##                                        3rd Qu.:  1.530   3rd Qu.:  1.450  
    ##                                        Max.   :667.220   Max.   :154.280  
    ##    expl_celi         expl_gravj          zkat                mt           
    ##  Min.   :0.00000   Min.   :0.00000   Length:505654      Length:505654     
    ##  1st Qu.:0.00000   1st Qu.:0.00000   Class :character   Class :character  
    ##  Median :0.00000   Median :0.00000   Mode  :character   Mode  :character  
    ##  Mean   :0.01066   Mean   :0.02879                                        
    ##  3rd Qu.:0.00000   3rd Qu.:0.01000                                        
    ##  Max.   :1.73000   Max.   :4.22000                                        
    ##      izc                s10                 a10              h10       
    ##  Length:505654      Length:505654      Min.   :  0.00   Min.   : 0.00  
    ##  Class :character   Class :character   1st Qu.: 14.00   1st Qu.: 6.00  
    ##  Mode  :character   Mode  :character   Median : 45.00   Median :19.00  
    ##                                        Mean   : 50.75   Mean   :16.54  
    ##                                        3rd Qu.: 80.00   3rd Qu.:26.00  
    ##                                        Max.   :322.00   Max.   :51.00  
    ##       d10              g10               n10              bv10          
    ##  Min.   :  0.00   Min.   :   0.00   Min.   :    0.0   Length:505654     
    ##  1st Qu.:  7.00   1st Qu.:   0.00   1st Qu.:    0.0   Class :character  
    ##  Median : 19.00   Median :  14.00   Median :    0.0   Mode  :character  
    ##  Mean   : 18.51   Mean   :  13.35   Mean   :  436.8                     
    ##  3rd Qu.: 28.00   3rd Qu.:  22.00   3rd Qu.:    0.0                     
    ##  Max.   :381.00   Max.   :1135.00   Max.   :35000.0                     
    ##      ba10               s11                 a11              h11       
    ##  Length:505654      Length:505654      Min.   :  0.00   Min.   : 0.00  
    ##  Class :character   Class :character   1st Qu.:  0.00   1st Qu.: 0.00  
    ##  Mode  :character   Mode  :character   Median : 17.00   Median : 7.00  
    ##                                        Mean   : 34.18   Mean   :11.31  
    ##                                        3rd Qu.: 67.00   3rd Qu.:23.00  
    ##                                        Max.   :355.00   Max.   :50.00  
    ##       d11              g11               n11               bv11          
    ##  Min.   :   0.0   Min.   :  0.000   Min.   :    0.00   Length:505654     
    ##  1st Qu.:   0.0   1st Qu.:  0.000   1st Qu.:    0.00   Class :character  
    ##  Median :   7.0   Median :  0.000   Median :    0.00   Mode  :character  
    ##  Mean   :  12.8   Mean   :  3.005   Mean   :   51.13                     
    ##  3rd Qu.:  26.0   3rd Qu.:  6.000   3rd Qu.:    0.00                     
    ##  Max.   :3301.0   Max.   :764.000   Max.   :35000.00                     
    ##      ba11               s12                 a12              h12        
    ##  Length:505654      Length:505654      Min.   :  0.00   Min.   : 0.000  
    ##  Class :character   Class :character   1st Qu.:  0.00   1st Qu.: 0.000  
    ##  Mode  :character   Mode  :character   Median :  0.00   Median : 0.000  
    ##                                        Mean   : 16.54   Mean   : 5.395  
    ##                                        3rd Qu.: 14.00   3rd Qu.: 5.000  
    ##                                        Max.   :497.00   Max.   :46.000  
    ##       d12                g12               n12              bv12          
    ##  Min.   :   0.000   Min.   :  0.000   Min.   :   0.00   Length:505654     
    ##  1st Qu.:   0.000   1st Qu.:  0.000   1st Qu.:   0.00   Class :character  
    ##  Median :   0.000   Median :  0.000   Median :   0.00   Mode  :character  
    ##  Mean   :   6.274   Mean   :  0.896   Mean   :  12.58                     
    ##  3rd Qu.:   6.000   3rd Qu.:  0.000   3rd Qu.:   0.00                     
    ##  Max.   :4940.000   Max.   :509.000   Max.   :4000.00                     
    ##      ba12               s13                 a13              h13       
    ##  Length:505654      Length:505654      Min.   :  0.00   Min.   : 0.00  
    ##  Class :character   Class :character   1st Qu.:  0.00   1st Qu.: 0.00  
    ##  Mode  :character   Mode  :character   Median :  0.00   Median : 0.00  
    ##                                        Mean   :  4.97   Mean   : 1.58  
    ##                                        3rd Qu.:  0.00   3rd Qu.: 0.00  
    ##                                        Max.   :292.00   Max.   :36.00  
    ##       d13               g13               n13               bv13          
    ##  Min.   :  0.000   Min.   : 0.0000   Min.   :   0.000   Length:505654     
    ##  1st Qu.:  0.000   1st Qu.: 0.0000   1st Qu.:   0.000   Class :character  
    ##  Median :  0.000   Median : 0.0000   Median :   0.000   Mode  :character  
    ##  Mean   :  1.889   Mean   : 0.2001   Mean   :   1.931                     
    ##  3rd Qu.:  0.000   3rd Qu.: 0.0000   3rd Qu.:   0.000                     
    ##  Max.   :262.000   Max.   :17.0000   Max.   :2940.000                     
    ##      ba13               s14                 a14               h14         
    ##  Length:505654      Length:505654      Min.   :  0.000   Min.   : 0.0000  
    ##  Class :character   Class :character   1st Qu.:  0.000   1st Qu.: 0.0000  
    ##  Mode  :character   Mode  :character   Median :  0.000   Median : 0.0000  
    ##                                        Mean   :  1.053   Mean   : 0.3286  
    ##                                        3rd Qu.:  0.000   3rd Qu.: 0.0000  
    ##                                        Max.   :224.000   Max.   :35.0000  
    ##       d14                g14                n14               bv14          
    ##  Min.   :  0.0000   Min.   : 0.00000   Min.   :  0.0000   Length:505654     
    ##  1st Qu.:  0.0000   1st Qu.: 0.00000   1st Qu.:  0.0000   Class :character  
    ##  Median :  0.0000   Median : 0.00000   Median :  0.0000   Mode  :character  
    ##  Mean   :  0.4042   Mean   : 0.03427   Mean   :  0.2089                     
    ##  3rd Qu.:  0.0000   3rd Qu.: 0.00000   3rd Qu.:  0.0000                     
    ##  Max.   :285.0000   Max.   :11.00000   Max.   :800.0000                     
    ##      ba14               jakopj         jaatjauno        p_darbv         
    ##  Length:505654      Min.   :   0.0   Min.   :   0.0   Length:505654     
    ##  Class :character   1st Qu.:   0.0   1st Qu.:   0.0   Class :character  
    ##  Mode  :character   Median :   0.0   Median :   0.0   Mode  :character  
    ##                     Mean   : 189.1   Mean   : 119.1                     
    ##                     3rd Qu.:   0.0   3rd Qu.:   0.0                     
    ##                     Max.   :2034.0   Max.   :2034.0                     
    ##     p_darbg        p_cirp              p_cirg        atj_gads   
    ##  Min.   :   0   Length:505654      Min.   :   0   Min.   :   0  
    ##  1st Qu.:   0   Class :character   1st Qu.:   0   1st Qu.:   0  
    ##  Median :2005   Mode  :character   Median :2003   Median :   0  
    ##  Mean   :1128                      Mean   :1144   Mean   : 651  
    ##  3rd Qu.:2020                      3rd Qu.:2016   3rd Qu.:2003  
    ##  Max.   :2024                      Max.   :8003   Max.   :2024  
    ##    saimn_d_ie     plant_audz         forestry_c         vmd_headfo       
    ##  Min.   :1.000   Length:505654      Length:505654      Length:505654     
    ##  1st Qu.:6.000   Class :character   Class :character   Class :character  
    ##  Median :6.000   Mode  :character   Mode  :character   Mode  :character  
    ##  Mean   :5.529                                                           
    ##  3rd Qu.:6.000                                                           
    ##  Max.   :6.000                                                           
    ##    shape_Leng         shape_Area               geometry     
    ##  Min.   :    0.58   Min.   :      0   MULTIPOLYGON :505654  
    ##  1st Qu.:  301.01   1st Qu.:   3852   epsg:3059    :     0  
    ##  Median :  441.68   Median :   7947   +proj=tmer...:     0  
    ##  Mean   :  515.63   Mean   :  12485                         
    ##  3rd Qu.:  635.41   3rd Qu.:  15343                         
    ##  Max.   :52008.97   Max.   :6672216

``` r
plot(meza_centra_dati["geometry"])
```

![](Uzd05_Ozols_files/figure-gfm/apvienotais%20mvr%20fails-1.png)<!-- -->
4. nomainam tks93 faila koordinātas uz LKS92

``` r
lapas <- st_transform(lapas, crs = 3059)
```

5.  savienojam mvr datus ar lapām izmantojot st_join, izmantojam unique,
    lai iegūtu lapu sarakstu, kurās ir MVR dati

``` r
lapas_mvr_dati <- st_join(meza_centra_dati,lapas)
unique(lapas_mvr_dati$NUMURS)
```

    ##  [1] 3332 3334 4222 3333 3224 3244 3331 3313 3314 3341 4311 3242 3241 3232 3234
    ## [16] 3214 3243 3223 3231 4221 4313 4314 4332 4323 4321 4312 4341 3343 4322 3344
    ## [31] 4212 4223 4213 4211 4214 3233 4232

Izvēlēmies lapas un izveidojam parquet failus.

``` r
lapa3333 <- lapas |> filter(lapas$NUMURS %in% "3333")
st_write_parquet(lapa3333,"C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3333.parquet")
lapa3334 <- lapas |> filter(lapas$NUMURS %in% "3334")
st_write_parquet(lapa3334,"C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3334.parquet")
lapa3343 <- lapas |> filter(lapas$NUMURS %in% "3343")
st_write_parquet(lapa3343,"C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3343.parquet")
lapa3344 <- lapas |> filter(lapas$NUMURS %in% "3344")
st_write_parquet(lapa3344,"C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3344.parquet")
```

Izveidojam list objektu

``` r
lapas_parqueti <- list.files("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05",pattern = "\\.parquet$")
lapas_parqueti
```

    ## [1] "lapa3333.parquet" "lapa3334.parquet" "lapa3343.parquet" "lapa3344.parquet"

Pirms ķeršanās pie uzdevuma, pārskatiet ceturtā uzdevuma funkciju
`mana_funkcija` tā, lai izvades rastrs (priežu mežaudzu īpatsvars (no
kopējās platības) 100 m šūnā) aptvertu tikai to teritoriju, kas pilnībā
iekļaujas iterētajā telpā (kartes lapā) to pilnībā aptverot.

mana_funkcija2 \<- function(ievades_fails,direktorija) { starts \<-
Sys.time() mvr_fails \<- st_read_parquet(ievades_fails) mezaudze \<-
mvr_fails %\>% filter(s10 == 1) LV10m_10km \<-
raster(“C:\Users\Ozols\Documents\HiQBioDiv_macibas\Uzd03\LV10m_10km.tif”)
rastrs_mezaudze_10m \<- fasterize(mezaudze, LV10m_10km, background = 0)
rastrs_mezaudze_10m\[is.na(LV10m_10km)\] \<- NA LV100m_10km \<-
raster(“C:\Users\Ozols\Documents\HiQBioDiv_macibas\Uzd03\LV100m_10km.tif”)
rastrs_mezaudze_100m \<- resample(rastrs_mezaudze_10m,LV100m_10km,
methode = “sum”) plot(rastrs_mezaudze_100m) ievades_nosaukums \<-
file_path_sans_ext(basename(ievades_fails)) izvades_fails \<-
file.path(paste0(direktorija,ievades_nosaukums, “.parquet”))
writeRaster(rastrs_mezaudze_100m, izvades_fails, format = “GTiff”,
overwrite = TRUE) finish \<- Sys.time()

Pārskatot funkciju, nepieciešams, lai kartes lapa un references slāļa
robežas atbilstu viena otrai, references rastru nepieciešams izgriezt
pēc lapas robežām.

Apsveriet ievades parametru sagatavošanu pirms ievietošanas funkcijā kā
alternatīvu funkcijas modificēšanai.

Salīdziniet procesēšanas laiku katrā uzdevuma daļā (sekojošajā
uzskaitījumā) no telpas dalīšanas uzsākšanas (ieskaitot) līdz
noslēdzošajam rastram (ieskaitot).

Pirmais apakšuzdevums. 1.1. solis: izmantojot spatial join saistiet MVR
datus ar izvēlētajām karšu lapām (šis ir sākums laika mērogošanai). Kāds
ir objektu skaits pirms un pēc savienošanas? Apskatiet katrai kartes
lapai piederīgos objektus - vai tie atrodas tikai kartes lapas iekšienē,
vai ir iekļauti visi objekti, kas ir uz kartes lapas robežām?

``` r
spatialjoin_funkcija <- function(ievades_fails){
  starts <- Sys.time()
  lapa <- st_read_parquet(ievades_fails)
  mvr_lapa <- st_join(meza_centra_dati,lapa,left = FALSE)
  plot(mvr_lapa["geometry"])
  ievades_nosaukums <- file_path_sans_ext(basename(ievades_fails))
  izvades_fails <- file.path(paste0(ievades_nosaukums, "mvr.parquet"))
  st_write_parquet(mvr_lapa,izvades_fails)
  finish <- Sys.time()
}
```

Izmēģinam funckiju ar vienu lapu

``` r
spatialjoin_funkcija("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3333.parquet")
```

![](Uzd05_Ozols_files/figure-gfm/funkcija-1.png)<!-- -->

``` r
system.time(sapply(lapas_parqueti,spatialjoin_funkcija))
```

![](Uzd05_Ozols_files/figure-gfm/mvrdati_lapas-1.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/mvrdati_lapas-2.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/mvrdati_lapas-3.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/mvrdati_lapas-4.png)<!-- -->

    ##    user  system elapsed 
    ##   27.67    7.77   35.44

Ielasam katru lapu, lai apskatītos to objektu skaitu

``` r
lapa3333_mvr <- st_read_parquet("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3333mvr.parquet")
lapa3333_mvr
```

    ## Simple feature collection with 33449 features and 74 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 497543.3 ymin: 274441.5 xmax: 525438.7 ymax: 301517.7
    ## Projected CRS: LKS-92 / Latvia TM
    ## First 10 features:
    ##           id objectid_1           adm1            adm2    atvk    kadastrs  gtf
    ## 1   75507144   75507144 Olaines novads Olaines pagasts 0041400 80800130058 2017
    ## 2  108236414  108236414 Bauskas novads Iecavas pagasts 0025460 40640030055 2020
    ## 3  108236411  108236411 Bauskas novads Iecavas pagasts 0025460 40640030055 2020
    ## 4  140176213  140176213 Bauskas novads Iecavas pagasts 0025460 40640030202 2010
    ## 5  142493515  142493515 Bauskas novads Iecavas pagasts 0025460 40640070229 2023
    ## 6  148231191  148231191 Bauskas novads Iecavas pagasts 0025460 40640010146 2023
    ## 7  148584238  148584238 Bauskas novads Iecavas pagasts 0025460 40640050075 2024
    ## 8  149342953  149342953 ?ekavas novads ?ekavas pagasts 0034421 80700170050 2021
    ## 9   49358269   49358269 Bauskas novads Iecavas pagasts 0025460 40640010065 2002
    ## 10  49358270   49358270 Bauskas novads Iecavas pagasts 0025460 40640010065 2002
    ##    kvart nog anog nog_plat expl_mezs expl_celi expl_gravj zkat mt izc s10 a10
    ## 1    141   9    0     0.17      0.00      0.00       0.00   31  0   0   0   0
    ## 2      1   5    0     0.21      0.21      0.00       0.00   10  4   1   1  22
    ## 3      1   2    0     0.15      0.15      0.00       0.00   10  4   1   1  22
    ## 4      1   2    0     0.10      0.10      0.00       0.00   14 25   1   0   0
    ## 5     73   1    0     0.03      0.03      0.00       0.00   10  4   1   4  84
    ## 6     54  28    0     0.82      0.72      0.08       0.02   10  3   2   1  63
    ## 7    322   2    0     0.38      0.38      0.00       0.00   14 19   0   0   0
    ## 8    292  11    1     0.26      0.26      0.00       0.00   10 21   1   6  14
    ## 9      1   1    0     0.14      0.14      0.00       0.00   10  4   2   3  72
    ## 10     1   2    0     0.14      0.14      0.00       0.00   10  3   1   1  72
    ##    h10 d10 g10 n10 bv10 ba10 s11 a11 h11 d11 g11 n11 bv11 ba11 s12 a12 h12 d12
    ## 1    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 2    9  11  39   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 3    9  11  32   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 4    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 5   30  31  15   0    0    0   8  84  30  32   3   0    0    0   0   0   0   0
    ## 6   21  22  23   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 7    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 8   12   9  13   0    0    0   4  14  12   9   1   0    0    0   0   0   0   0
    ## 9   22  25  43   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 10  25  28  15   0    0    0   4  72  21  32   7   0    0    0   1 102  23  36
    ##    g12 n12 bv12 ba12 s13 a13 h13 d13 g13 n13 bv13 ba13 s14 a14 h14 d14 g14 n14
    ## 1    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 2    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 3    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 4    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 5    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 6    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 7    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 8    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 9    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 10   5   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ##    bv14 ba14 jakopj jaatjauno p_darbv p_darbg p_cirp p_cirg atj_gads saimn_d_ie
    ## 1     0    0      0         0       0       0      0      0        0          6
    ## 2     0    0      0         0       0       0      0      0        0          6
    ## 3     0    0      0         0       0       0      0      0        0          6
    ## 4     0    0      0      2029       1    2024     81   2024        0          6
    ## 5     0    0      0         0       0       0      0      0        0          6
    ## 6     0    0      0         0       1    2024     22   2024        0          6
    ## 7     0    0      0      2029       1    2024     11   2024        0          6
    ## 8     0    0      0         0       6    2021     11   2011     2013          6
    ## 9     0    0      0         0       0       0      0      0        0          6
    ## 10    0    0      0         0       0       0      0      0        0          6
    ##    plant_audz forestry_c vmd_headfo shape_Leng shape_Area NOSAUKUMS NUMURS
    ## 1           0       2651     Centra  207.08857  1698.6654   Baldone   3333
    ## 2           1       2651     Centra  240.55874  2123.5491   Baldone   3333
    ## 3           1       2651     Centra  202.33054  1456.8286   Baldone   3333
    ## 4           0       2651     Centra  145.20065  1002.2579   Baldone   3333
    ## 5           0       2651     Centra   96.23192   306.9282   Baldone   3333
    ## 6           0       2651     Centra  378.97765  8165.6619   Baldone   3333
    ## 7           0       2651     Centra  271.68713  3778.7076   Baldone   3333
    ## 8           0       2651     Centra  215.50695  2603.2586   Baldone   3333
    ## 9           0       2651     Centra  159.29683  1419.4795   Baldone   3333
    ## 10          0       2651     Centra  196.70426  1392.8598   Baldone   3333
    ##    Shape_Length Shape_Area                       geometry
    ## 1         1e+05   6.25e+08 MULTIPOLYGON (((501810.4 29...
    ## 2         1e+05   6.25e+08 MULTIPOLYGON (((516402.9 28...
    ## 3         1e+05   6.25e+08 MULTIPOLYGON (((516444.4 28...
    ## 4         1e+05   6.25e+08 MULTIPOLYGON (((517894 2795...
    ## 5         1e+05   6.25e+08 MULTIPOLYGON (((508496.9 27...
    ## 6         1e+05   6.25e+08 MULTIPOLYGON (((507021.6 27...
    ## 7         1e+05   6.25e+08 MULTIPOLYGON (((522809.9 27...
    ## 8         1e+05   6.25e+08 MULTIPOLYGON (((510234.5 28...
    ## 9         1e+05   6.25e+08 MULTIPOLYGON (((501024.1 27...
    ## 10        1e+05   6.25e+08 MULTIPOLYGON (((501061.2 28...

``` r
lapa3334_mvr <- st_read_parquet("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3334mvr.parquet")
lapa3334_mvr
```

    ## Simple feature collection with 32904 features and 74 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 524686.8 ymin: 274567.8 xmax: 550368.6 ymax: 300505.8
    ## Projected CRS: LKS-92 / Latvia TM
    ## First 10 features:
    ##          id objectid_1           adm1               adm2    atvk    kadastrs
    ## 1  51858796   51858796 Bauskas novads Vecumnieku pagasts 0025560 40940050048
    ## 2  99205937   99205937 Bauskas novads Vecumnieku pagasts 0025560 40940040006
    ## 3  49376379   49376379 Bauskas novads Vecumnieku pagasts 0025560 40940050026
    ## 4  49376380   49376380 Bauskas novads Vecumnieku pagasts 0025560 40940050026
    ## 5  49376381   49376381 Bauskas novads Vecumnieku pagasts 0025560 40940050026
    ## 6  49376382   49376382 Bauskas novads Vecumnieku pagasts 0025560 40940050026
    ## 7  49376383   49376383 Bauskas novads Vecumnieku pagasts 0025560 40940050026
    ## 8  49376384   49376384 Bauskas novads Vecumnieku pagasts 0025560 40940050027
    ## 9  49376385   49376385 Bauskas novads Vecumnieku pagasts 0025560 40940050027
    ## 10 49376484   49376484 Bauskas novads Vecumnieku pagasts 0025560 40940050027
    ##     gtf kvart nog anog nog_plat expl_mezs expl_celi expl_gravj zkat mt izc s10
    ## 1  2012     1  12    0     0.08      0.00         0       0.00   32  0   0   0
    ## 2  2020     1  79    0     0.07      0.07         0       0.00   10  4   1   1
    ## 3  2003     1   2    0     0.99      0.99         0       0.00   10  4   1   1
    ## 4  2003     1   3    0     0.34      0.34         0       0.00   10 15   1   4
    ## 5  2003     1   4    0     1.75      1.75         0       0.00   10  4   1   1
    ## 6  2003     1   5    0     0.64      0.64         0       0.00   10  4   1   1
    ## 7  2003     1   6    0     0.43      0.43         0       0.00   10  4   1   3
    ## 8  2003     2   1    0     0.23      0.20         0       0.03   10  4   1   1
    ## 9  2003     2   2    0     1.00      0.92         0       0.08   10  4   1   4
    ## 10 2003     2   3    0     0.37      0.32         0       0.05   10  4   1   1
    ##    a10 h10 d10 g10 n10 bv10 ba10 s11 a11 h11 d11 g11 n11 bv11 ba11 s12 a12 h12
    ## 1    0   0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 2   58  26  27  30   0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 3  112  29  35  25   0    0    0   3  92  29  30  11   0    0    0   0   0   0
    ## 4   77  21  23  22   0    0    0   1  77  21  20   7   0    0    0   0   0   0
    ## 5   87  27  33  15   0    0    0   3  87  27  29  15   0    0    0   4  87  29
    ## 6   82  28  35  26   0    0    0   4  82  28  28   3   0    0    0   0   0   0
    ## 7   72  24  35  33   0    0    0   1  72  23  22  12   0    0    0   4  72  28
    ## 8   82  29  34  33   0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 9   60  27  30  13   0    0    0   3  60  27  33   5   0    0    0   1  60  28
    ## 10  80  28  38  24   0    0    0   4  80  27  27   8   0    0    0   0   0   0
    ##    d12 g12 n12 bv12 ba12 s13 a13 h13 d13 g13 n13 bv13 ba13 s14 a14 h14 d14 g14
    ## 1    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 2    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 3    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 4    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 5   28   8   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 6    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 7   27   5   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 8    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 9   28   1   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 10   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ##    n14 bv14 ba14 jakopj jaatjauno p_darbv p_darbg p_cirp p_cirg atj_gads
    ## 1    0    0    0      0         0       0       0      0      0        0
    ## 2    0    0    0      0         0       1    2023     22   2023        0
    ## 3    0    0    0      0         0       1    2005     72   2005        0
    ## 4    0    0    0      0         0       0       0      0      0        0
    ## 5    0    0    0      0         0       1    2005     72   2005        0
    ## 6    0    0    0      0         0       1    2005     72   2005        0
    ## 7    0    0    0      0         0       1    2005     72   2005        0
    ## 8    0    0    0      0         0       1    2003     22   2003        0
    ## 9    0    0    0      0         0       1    2005     72   2005        0
    ## 10   0    0    0      0         0       1    2005     72   2005        0
    ##    saimn_d_ie plant_audz forestry_c vmd_headfo shape_Leng shape_Area NOSAUKUMS
    ## 1           6          0       2651     Centra   113.7443   757.9693      Ogre
    ## 2           4          0       2651     Centra   117.4964   733.5307      Ogre
    ## 3           6          0       2651     Centra   527.1829  9914.3136      Ogre
    ## 4           6          0       2651     Centra   345.5902  3391.7567      Ogre
    ## 5           6          0       2651     Centra   819.5234 17477.8172      Ogre
    ## 6           6          0       2651     Centra   408.2227  6394.0473      Ogre
    ## 7           6          0       2651     Centra   282.9741  4302.5068      Ogre
    ## 8           6          0       2651     Centra   248.8586  2314.4887      Ogre
    ## 9           6          0       2651     Centra   572.4052  9989.9399      Ogre
    ## 10          6          0       2651     Centra   248.5897  3666.7202      Ogre
    ##    NUMURS Shape_Length Shape_Area                       geometry
    ## 1    3334        1e+05   6.25e+08 MULTIPOLYGON (((527092.6 27...
    ## 2    3334        1e+05   6.25e+08 MULTIPOLYGON (((525860.8 27...
    ## 3    3334        1e+05   6.25e+08 MULTIPOLYGON (((528135.1 27...
    ## 4    3334        1e+05   6.25e+08 MULTIPOLYGON (((528313.6 27...
    ## 5    3334        1e+05   6.25e+08 MULTIPOLYGON (((528135.2 27...
    ## 6    3334        1e+05   6.25e+08 MULTIPOLYGON (((528057 2772...
    ## 7    3334        1e+05   6.25e+08 MULTIPOLYGON (((528251 2771...
    ## 8    3334        1e+05   6.25e+08 MULTIPOLYGON (((527732.2 27...
    ## 9    3334        1e+05   6.25e+08 MULTIPOLYGON (((527745.8 27...
    ## 10   3334        1e+05   6.25e+08 MULTIPOLYGON (((527824.5 27...

``` r
lapa3343_mvr <- st_read_parquet("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3343mvr.parquet")
lapa3343_mvr
```

    ## Simple feature collection with 21466 features and 74 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 549584.9 ymin: 274546.4 xmax: 575991.2 ymax: 300897.2
    ## Projected CRS: LKS-92 / Latvia TM
    ## First 10 features:
    ##           id objectid_1         adm1               adm2    atvk    kadastrs
    ## 1   51261610   51261610 Ogres novads  Madlienas pagasts 0040470 74680090078
    ## 2   99827232   99827232 Ogres novads     Krapes pagasts 0040420 74520030004
    ## 3  109753047  109753047 Ogres novads  Jumpravas pagasts 0040410 74480010075
    ## 4  114256226  114256226 Ogres novads Lielv?rdes pagasts 0040460 74330020794
    ## 5  138000835  138000835 Ogres novads   Lauberes pagasts 0040440 74600020018
    ## 6  140683354  140683354 Ogres novads   L?dmanes pagasts 0040450 74640040028
    ## 7  148912986  148912986 Ogres novads   Rembates pagasts 0040510 74840030197
    ## 8   49341737   49341737 Ogres novads  Jumpravas pagasts 0040410 74480020203
    ## 9   49376744   49376744 Ogres novads  Jumpravas pagasts 0040410 74480020087
    ## 10  49376745   49376745 Ogres novads  Jumpravas pagasts 0040410 74480020087
    ##     gtf kvart nog anog nog_plat expl_mezs expl_celi expl_gravj zkat mt izc s10
    ## 1  2015     1   7    0     0.15      0.00         0       0.00   42  0   0   0
    ## 2  2019     1   7    0     0.49      0.49         0       0.00   14 10   1   0
    ## 3  2020     1   6    0     0.34      0.34         0       0.00   10 21   1   3
    ## 4  2004     1   3    0     0.25      0.25         0       0.00   10  3   1   1
    ## 5  2010   336  22    0     0.70      0.61         0       0.09   10 14   1   4
    ## 6  2023     1   3    0     0.34      0.31         0       0.03   10  5   1   4
    ## 7  2024     1   3    0     0.17      0.17         0       0.00   10  5   1   4
    ## 8  2002     1   1    0     0.61      0.56         0       0.05   10 21   1   9
    ## 9  2003     5   3    0     0.33      0.33         0       0.00   10  5   1   9
    ## 10 2003     5   6    0     0.86      0.86         0       0.00   10  5   1   4
    ##    a10 h10 d10 g10  n10 bv10 ba10 s11 a11 h11 d11 g11 n11 bv11 ba11 s12 a12 h12
    ## 1    0   0   0   0    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 2    0   0   0   0    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 3   37  16  20  29    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 4    3   1   1   0 3000    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 5   34  15  15  11    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 6   68  28  29  13    0    0    0   8  68  29  35   1   0    0    0   0   0   0
    ## 7   65  26  26  20    0    0    0   8  65  28  31   2   0    0    0   0   0   0
    ## 8   14   8  11  16    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 9    7   4   4   0 2450    0    0   3   7   2   3   0 350    0    0  16   7   1
    ## 10   7   4   4   0 2100    0    0   8   7   5   5   0 500    0    0   9   7   4
    ##    d12 g12 n12 bv12 ba12 s13 a13 h13 d13 g13 n13 bv13 ba13 s14 a14 h14 d14 g14
    ## 1    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 2    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 3    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 4    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 5    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 6    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 7    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 8    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 9    2   0 100    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 10   4   0 100    0    0   6   7   3   3   0 100    0    0   0   0   0   0   0
    ##    n14 bv14 ba14 jakopj jaatjauno p_darbv p_darbg p_cirp p_cirg atj_gads
    ## 1    0    0    0      0         0       0       0      0      0        0
    ## 2    0    0    0      0      2024       1    2019     11   2019        0
    ## 3    0    0    0      0         0       0       0      0      0        0
    ## 4    0    0    0   2032         0      10    2022     61   2021     2022
    ## 5    0    0    0      0         0       0       0     21   2004        0
    ## 6    0    0    0      0         0       1    2004     22   2004        0
    ## 7    0    0    0      0         0       0       0      0      0        0
    ## 8    0    0    0      0         0       4    2015     11   2010     2015
    ## 9    0    0    0      0         0       6    2023     11   2014     2019
    ## 10   0    0    0   2024         0       6    2023     11   2014     2019
    ##    saimn_d_ie plant_audz forestry_c vmd_headfo shape_Leng shape_Area NOSAUKUMS
    ## 1           6          0       2654     Centra   195.0041   1537.971  Skrīveri
    ## 2           6          0       2654     Centra   280.3220   4888.540  Skrīveri
    ## 3           6          0       2654     Centra   236.6700   3430.599  Skrīveri
    ## 4           6          0       2654     Centra   604.4541   2495.869  Skrīveri
    ## 5           4          0       2654     Centra   378.8590   7042.177  Skrīveri
    ## 6           6          0       2654     Centra   249.4908   3434.501  Skrīveri
    ## 7           6          0       2654     Centra   183.8692   1691.748  Skrīveri
    ## 8           6          0       2654     Centra   326.7961   6085.816  Skrīveri
    ## 9           6          0       2654     Centra   253.8150   3345.459  Skrīveri
    ## 10          6          0       2654     Centra   378.7362   8641.671  Skrīveri
    ##    NUMURS Shape_Length Shape_Area                       geometry
    ## 1    3343        1e+05   6.25e+08 MULTIPOLYGON (((570010.3 29...
    ## 2    3343        1e+05   6.25e+08 MULTIPOLYGON (((572611.2 29...
    ## 3    3343        1e+05   6.25e+08 MULTIPOLYGON (((558694.7 28...
    ## 4    3343        1e+05   6.25e+08 MULTIPOLYGON (((552877 2858...
    ## 5    3343        1e+05   6.25e+08 MULTIPOLYGON (((562485.6 29...
    ## 6    3343        1e+05   6.25e+08 MULTIPOLYGON (((554142.2 29...
    ## 7    3343        1e+05   6.25e+08 MULTIPOLYGON (((552731.8 29...
    ## 8    3343        1e+05   6.25e+08 MULTIPOLYGON (((558963.8 27...
    ## 9    3343        1e+05   6.25e+08 MULTIPOLYGON (((559202.9 28...
    ## 10   3343        1e+05   6.25e+08 MULTIPOLYGON (((559186.8 28...

``` r
lapa3344_mvr <- st_read_parquet("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3344mvr.parquet")
lapa3344_mvr
```

    ## Simple feature collection with 7874 features and 74 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 574406.4 ymin: 284200.4 xmax: 592046.2 ymax: 300327.3
    ## Projected CRS: LKS-92 / Latvia TM
    ## First 10 features:
    ##           id objectid_1         adm1              adm2    atvk    kadastrs  gtf
    ## 1   93980758   93980758 Ogres novads  Me??eles pagasts 0040490 74760050006 2019
    ## 2   94092550   94092550 Ogres novads    Krapes pagasts 0040420 74520080085 2011
    ## 3  125451619  125451619 Ogres novads  Me??eles pagasts 0040490 74760040057 2015
    ## 4  138760213  138760213 Ogres novads  Me??eles pagasts 0040490 74760050003 2023
    ## 5   49402087   49402087 Ogres novads  Me??eles pagasts 0040490 74760030016 2004
    ## 6   49402090   49402090 Ogres novads  Me??eles pagasts 0040490 74760030016 2004
    ## 7   49477291   49477291 Ogres novads Madlienas pagasts 0040470 74680040083 2000
    ## 8   49477292   49477292 Ogres novads Madlienas pagasts 0040470 74680040083 2000
    ## 9   49476682   49476682 Ogres novads  Me??eles pagasts 0040490 74760040276 2005
    ## 10  49476683   49476683 Ogres novads  Me??eles pagasts 0040490 74760040276 2005
    ##    kvart nog anog nog_plat expl_mezs expl_celi expl_gravj zkat mt izc s10 a10
    ## 1      1   1    0     0.51      0.51         0       0.00   10 25   1   4  67
    ## 2      2  15    0     0.08      0.00         0       0.00   22  5   1   0   0
    ## 3      2   7    0     0.19      0.17         0       0.02   14 21   1   0   0
    ## 4      1  28    0     4.39      4.39         0       0.00   14 19   0   0   0
    ## 5      1   8    0     0.21      0.00         0       0.00   22  0   0   0   0
    ## 6      1  13    0     0.23      0.00         0       0.00   22  0   0   0   0
    ## 7      2   3    0     0.23      0.23         0       0.00   10 21   1   8  23
    ## 8      2   2    0     0.89      0.88         0       0.01   10  5   1   8  23
    ## 9      1   1    0     0.99      0.93         0       0.06   10 24   1   4  26
    ## 10     1   3    0     1.56      1.56         0       0.00   10 19   1   4  26
    ##    h10 d10 g10 n10 bv10 ba10 s11 a11 h11 d11 g11 n11 bv11 ba11 s12 a12 h12 d12
    ## 1   23  26  24   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 2    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 3    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 4    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 5    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 6    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 7   12  17  36   0    0    0   4  23  12  12   3   0    0    0   0   0   0   0
    ## 8   12  17  15   0    0    0   4  23  12  12  11   0    0    0   0   0   0   0
    ## 9   11  15  15   0    0    0   3  26   4   4   1   0    0    0   0   0   0   0
    ## 10  12  12  15   0    0    0   3  26   3   5   2   0    0    0   0   0   0   0
    ##    g12 n12 bv12 ba12 s13 a13 h13 d13 g13 n13 bv13 ba13 s14 a14 h14 d14 g14 n14
    ## 1    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 2    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 3    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 4    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 5    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 6    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 7    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 8    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 9    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 10   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ##    bv14 ba14 jakopj jaatjauno p_darbv p_darbg p_cirp p_cirg atj_gads saimn_d_ie
    ## 1     0    0      0         0       0       0      0      0        0          6
    ## 2     0    0      0         0       1    2012     22   2012        0          6
    ## 3     0    0      0      2026       1    2021     11   2021        0          6
    ## 4     0    0      0      2024       0       0     81   2019        0          6
    ## 5     0    0      0         0       0       0      0      0        0          6
    ## 6     0    0      0         0       0       0      0      0        0          6
    ## 7     0    0      0         0       4    2005     11   2001     2005          6
    ## 8     0    0      0         0       4    2005     11   2001     2005          6
    ## 9     0    0      0         0       1    2022     22   2022     2005          5
    ## 10    0    0      0         0       1    2022     22   2022     2005          5
    ##    plant_audz forestry_c vmd_headfo shape_Leng shape_Area NOSAUKUMS NUMURS
    ## 1           0       2654     Centra   328.1841   5146.992   Koknese   3344
    ## 2           0       2654     Centra   131.1396    756.648   Koknese   3344
    ## 3           0       2654     Centra   261.0201   1897.233   Koknese   3344
    ## 4           0       2654     Centra  8073.9639  43855.126   Koknese   3344
    ## 5           0       2654     Centra   233.9315   2077.357   Koknese   3344
    ## 6           0       2654     Centra   208.7556   2268.000   Koknese   3344
    ## 7           0       2654     Centra   249.8497   2332.361   Koknese   3344
    ## 8           0       2654     Centra   552.2459   8869.708   Koknese   3344
    ## 9           0       2654     Centra   879.4987   9895.713   Koknese   3344
    ## 10          0       2654     Centra  1052.0235  15585.167   Koknese   3344
    ##    Shape_Length Shape_Area                       geometry
    ## 1         1e+05   6.25e+08 MULTIPOLYGON (((582123.8 29...
    ## 2         1e+05   6.25e+08 MULTIPOLYGON (((576215.9 28...
    ## 3         1e+05   6.25e+08 MULTIPOLYGON (((581964.3 29...
    ## 4         1e+05   6.25e+08 MULTIPOLYGON (((583363.9 29...
    ## 5         1e+05   6.25e+08 MULTIPOLYGON (((591625.8 29...
    ## 6         1e+05   6.25e+08 MULTIPOLYGON (((591687.6 29...
    ## 7         1e+05   6.25e+08 MULTIPOLYGON (((576056.7 29...
    ## 8         1e+05   6.25e+08 MULTIPOLYGON (((576057.1 29...
    ## 9         1e+05   6.25e+08 MULTIPOLYGON (((582500.7 29...
    ## 10        1e+05   6.25e+08 MULTIPOLYGON (((582827.5 29...

Atbilde: Objektu skaits pirms savienošanas ir pilnais mvr objektu
skaits, objektu skaits pēc savienošanas katrā lapā ir atšķirīgs (lapai
3333 ir 33449, lapai 3334 ir 32904, lapai 3343 ir 21466 un lapai 3344 ir
7874 nogabali, kopā 95693 nogabali), izmēģināju savienot ar argumentu
join = st_within, ieguvu tikai nogabalus, kas pilnībā ietilpst kartes
lapā, tomēr daļa no teritorijām, kas ietilpst kartes lapā neuzrādījās,
kā arī izgriežot 10m rastru precīzi pa kartes lapas robežām, nogabalu
daļas, kas atradīsies ārpus kartes lapas neietekmēs 10m rastra šūnu
rezultātus. Iekļauti ir visi objekti, kas atrodas lapā, tai skaitā uz
robežām.

1.2. solis: iteratīvā ciklā izmantojiet pārskatīto funkciju no uzdevuma
sākuma, lai saglabātu katras karšu lapas rezultātu savā GeoTIFF failā,
kura nosaukums ir saistāms ar šo apakšuzdevumu un individuālo lapu. Kā
izskatās šūnu aizpildījums pie karšu lapu malām? Vai ir saglabājušies
objekti ārpus kartes lapām?

Vispirms izmēģināsim izveidot vienai lapai rastra failu

``` r
LV10m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV10m_10km.tif")
LV10m_10km_lapa3333 <- crop(LV10m_10km,lapa3333)
priedes_lapa3333 <- lapa3333_mvr %>%
  filter (s10 == 1) 
rastrs_mezaudze_10m <- fasterize(priedes_lapa3333, LV10m_10km_lapa3333, background = 0)
rastrs_mezaudze_10m[is.na(LV10m_10km_lapa3333)] <- NA
LV100m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV100m_10km.tif")
LV100m_10km_lapa3333 <- crop(LV100m_10km,lapa3333)
rastrs_mezaudze_100m <- resample(rastrs_mezaudze_10m,LV100m_10km_lapa3333, methode = "sum")
plot(rastrs_mezaudze_100m)
```

![](Uzd05_Ozols_files/figure-gfm/izmeginam_rastru-1.png)<!-- -->

Izskatās pareizi, saliksim abas funkcijas vienā un izveidosim iteratīvo
ciklu, lai iegūtu rezultātu katrai lapai, kā ievades failu izmantojam
izvēlēto lapu listu

``` r
funkcija_parskatita <- function(ievades_fails) {
  starts <- Sys.time()
  lapa <- st_read_parquet(ievades_fails)
  mvr_lapa <- st_join(meza_centra_dati,lapa,left = FALSE)
  mezaudze <- mvr_lapa %>%
    filter(s10 == 1)
  LV10m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV10m_10km.tif")
  LV10m_10km_lapa <- crop(LV10m_10km,lapa)
  rastrs_mezaudze_10m <- fasterize(mezaudze, LV10m_10km_lapa, background = 0)
  rastrs_mezaudze_10m[is.na(LV10m_10km_lapa)] <- NA
  LV100m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV100m_10km.tif")
  LV100m_10km_lapa <- crop(LV100m_10km,lapa)
  rastrs_mezaudze_100m <- resample(rastrs_mezaudze_10m,LV100m_10km_lapa, methode = "sum")
  plot(rastrs_mezaudze_100m)
  ievades_nosaukums <- file_path_sans_ext(basename(ievades_fails))
  izvades_fails <- file.path(paste0(ievades_nosaukums, "uzd12.parquet"))
  writeRaster(rastrs_mezaudze_100m, izvades_fails, format = "GTiff", overwrite = TRUE)
  finish <- Sys.time()
}
```

``` r
system.time(sapply(lapas_parqueti,funkcija_parskatita))
```

![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_1apaksuzdevums-1.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_1apaksuzdevums-2.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_1apaksuzdevums-3.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_1apaksuzdevums-4.png)<!-- -->

    ##    user  system elapsed 
    ##   35.61    4.17   39.79

Atbilde: Šūnu aizpildījums pie lapu malām izskatās pilnīgs, bet objekti
ārpus lapu malām nav saglabājušies

``` r
Rastrs_lapa3333 <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3333uzd12.tif")
Rastrs_lapa3334 <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3334uzd12.tif")
Rastrs_lapa3343 <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3343uzd12.tif")
Rastrs_lapa3344 <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3344uzd12.tif")
pilna_teritorija <- merge(Rastrs_lapa3333,Rastrs_lapa3334,Rastrs_lapa3343,Rastrs_lapa3344)
plot(pilna_teritorija)
```

![](Uzd05_Ozols_files/figure-gfm/merge-1.png)<!-- -->

``` r
funkcija_parskatita <- function(ievades_fails) {
  starts <- Sys.time()
  lapa <- st_read_parquet(ievades_fails)
  mvr_lapa <- st_join(meza_centra_dati,lapa,left = FALSE)
  mezaudze <- mvr_lapa %>%
    filter(s10 == 1)
  LV10m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV10m_10km.tif")
  LV10m_10km_lapa <- crop(LV10m_10km,lapa)
  rastrs_mezaudze_10m <- fasterize(mezaudze, LV10m_10km_lapa, background = 0)
  rastrs_mezaudze_10m[is.na(LV10m_10km_lapa)] <- NA
  LV100m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV100m_10km.tif")
  LV100m_10km_lapa <- crop(LV100m_10km,lapa)
  rastrs_mezaudze_100m <- resample(rastrs_mezaudze_10m,LV100m_10km_lapa, methode = "sum")
  plot(rastrs_mezaudze_100m)
  ievades_nosaukums <- file_path_sans_ext(basename(ievades_fails))
  izvades_fails <- file.path(paste0(ievades_nosaukums, "uzd12.parquet"))
  writeRaster(rastrs_mezaudze_100m, izvades_fails, format = "GTiff", overwrite = TRUE)
  Rastrs_lapa3333 <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3333uzd12.tif")
Rastrs_lapa3334 <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3334uzd12.tif")
Rastrs_lapa3343 <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3343uzd12.tif")
Rastrs_lapa3344 <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3344uzd12.tif")
pilna_teritorija <- merge(Rastrs_lapa3333,Rastrs_lapa3334,Rastrs_lapa3343,Rastrs_lapa3344)
plot(pilna_teritorija)
  finish <- Sys.time()
}
```

``` r
system.time(sapply(lapas_parqueti,funkcija_parskatita))
```

![](Uzd05_Ozols_files/figure-gfm/rastri_lapam13-1.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam13-2.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam13-3.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam13-4.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam13-5.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam13-6.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam13-7.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam13-8.png)<!-- -->

    ##    user  system elapsed 
    ##   38.14    5.22   43.31

Atbilde 1.3. Apvienotajā slānī nav redzamas karšu lapu robežas, redzama
ir tikai izvēlētās teritorijas ārējā robeža.

2.1. solis: sāciet iteratīvo ciklu, kurā, izmantojot clipping iegūstiet
MVR datus ar apstrādājamo kartes lapu (šis ir sākums laika mērogošanai).
Kāds ir objektu skaits katrā kartes lapā, kā tas saistās ar iepriekšējo
apakšuzdevumu? Ārpus cikla apskatiet katrai kartes lapai piederīgos
objektus - vai tie atrodas tikai kartes lapas iekšienē, vai ir iekļauti
visi objekti, kas ir uz kartes lapas robežām?

``` r
clipping_funkcija <- function(ievades_fails){
  starts <- Sys.time()
  lapa <- st_read_parquet(ievades_fails)
  mvr_lapa <- st_intersection(meza_centra_dati,lapa)
  plot(mvr_lapa["geometry"])
  ievades_nosaukums <- file_path_sans_ext(basename(ievades_fails))
  izvades_fails <- file.path(paste0(ievades_nosaukums, "clipping.parquet"))
  st_write_parquet(mvr_lapa,izvades_fails)
  finish <- Sys.time()
}
```

``` r
system.time(sapply(lapas_parqueti,clipping_funkcija))
```

![](Uzd05_Ozols_files/figure-gfm/mvrdati_lapas_clipping_21-1.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/mvrdati_lapas_clipping_21-2.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/mvrdati_lapas_clipping_21-3.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/mvrdati_lapas_clipping_21-4.png)<!-- -->

    ##    user  system elapsed 
    ##   47.57    8.12   55.89

``` r
lapa3333_clipping <- st_read_parquet("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3333clipping.parquet")
lapa3333_clipping
```

    ## Simple feature collection with 33449 features and 74 fields
    ## Geometry type: GEOMETRY
    ## Dimension:     XY
    ## Bounding box:  xmin: 5e+05 ymin: 275000 xmax: 525000 ymax: 3e+05
    ## Projected CRS: LKS-92 / Latvia TM
    ## First 10 features:
    ##           id objectid_1           adm1            adm2    atvk    kadastrs  gtf
    ## 1   75507144   75507144 Olaines novads Olaines pagasts 0041400 80800130058 2017
    ## 2  108236414  108236414 Bauskas novads Iecavas pagasts 0025460 40640030055 2020
    ## 3  108236411  108236411 Bauskas novads Iecavas pagasts 0025460 40640030055 2020
    ## 4  140176213  140176213 Bauskas novads Iecavas pagasts 0025460 40640030202 2010
    ## 5  142493515  142493515 Bauskas novads Iecavas pagasts 0025460 40640070229 2023
    ## 6  148231191  148231191 Bauskas novads Iecavas pagasts 0025460 40640010146 2023
    ## 7  148584238  148584238 Bauskas novads Iecavas pagasts 0025460 40640050075 2024
    ## 8  149342953  149342953 ?ekavas novads ?ekavas pagasts 0034421 80700170050 2021
    ## 9   49358269   49358269 Bauskas novads Iecavas pagasts 0025460 40640010065 2002
    ## 10  49358270   49358270 Bauskas novads Iecavas pagasts 0025460 40640010065 2002
    ##    kvart nog anog nog_plat expl_mezs expl_celi expl_gravj zkat mt izc s10 a10
    ## 1    141   9    0     0.17      0.00      0.00       0.00   31  0   0   0   0
    ## 2      1   5    0     0.21      0.21      0.00       0.00   10  4   1   1  22
    ## 3      1   2    0     0.15      0.15      0.00       0.00   10  4   1   1  22
    ## 4      1   2    0     0.10      0.10      0.00       0.00   14 25   1   0   0
    ## 5     73   1    0     0.03      0.03      0.00       0.00   10  4   1   4  84
    ## 6     54  28    0     0.82      0.72      0.08       0.02   10  3   2   1  63
    ## 7    322   2    0     0.38      0.38      0.00       0.00   14 19   0   0   0
    ## 8    292  11    1     0.26      0.26      0.00       0.00   10 21   1   6  14
    ## 9      1   1    0     0.14      0.14      0.00       0.00   10  4   2   3  72
    ## 10     1   2    0     0.14      0.14      0.00       0.00   10  3   1   1  72
    ##    h10 d10 g10 n10 bv10 ba10 s11 a11 h11 d11 g11 n11 bv11 ba11 s12 a12 h12 d12
    ## 1    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 2    9  11  39   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 3    9  11  32   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 4    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 5   30  31  15   0    0    0   8  84  30  32   3   0    0    0   0   0   0   0
    ## 6   21  22  23   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 7    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 8   12   9  13   0    0    0   4  14  12   9   1   0    0    0   0   0   0   0
    ## 9   22  25  43   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 10  25  28  15   0    0    0   4  72  21  32   7   0    0    0   1 102  23  36
    ##    g12 n12 bv12 ba12 s13 a13 h13 d13 g13 n13 bv13 ba13 s14 a14 h14 d14 g14 n14
    ## 1    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 2    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 3    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 4    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 5    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 6    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 7    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 8    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 9    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 10   5   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ##    bv14 ba14 jakopj jaatjauno p_darbv p_darbg p_cirp p_cirg atj_gads saimn_d_ie
    ## 1     0    0      0         0       0       0      0      0        0          6
    ## 2     0    0      0         0       0       0      0      0        0          6
    ## 3     0    0      0         0       0       0      0      0        0          6
    ## 4     0    0      0      2029       1    2024     81   2024        0          6
    ## 5     0    0      0         0       0       0      0      0        0          6
    ## 6     0    0      0         0       1    2024     22   2024        0          6
    ## 7     0    0      0      2029       1    2024     11   2024        0          6
    ## 8     0    0      0         0       6    2021     11   2011     2013          6
    ## 9     0    0      0         0       0       0      0      0        0          6
    ## 10    0    0      0         0       0       0      0      0        0          6
    ##    plant_audz forestry_c vmd_headfo shape_Leng shape_Area NOSAUKUMS NUMURS
    ## 1           0       2651     Centra  207.08857  1698.6654   Baldone   3333
    ## 2           1       2651     Centra  240.55874  2123.5491   Baldone   3333
    ## 3           1       2651     Centra  202.33054  1456.8286   Baldone   3333
    ## 4           0       2651     Centra  145.20065  1002.2579   Baldone   3333
    ## 5           0       2651     Centra   96.23192   306.9282   Baldone   3333
    ## 6           0       2651     Centra  378.97765  8165.6619   Baldone   3333
    ## 7           0       2651     Centra  271.68713  3778.7076   Baldone   3333
    ## 8           0       2651     Centra  215.50695  2603.2586   Baldone   3333
    ## 9           0       2651     Centra  159.29683  1419.4795   Baldone   3333
    ## 10          0       2651     Centra  196.70426  1392.8598   Baldone   3333
    ##    Shape_Length Shape_Area                       geometry
    ## 1         1e+05   6.25e+08 POLYGON ((501807.6 295651.4...
    ## 2         1e+05   6.25e+08 POLYGON ((516435 280698.7, ...
    ## 3         1e+05   6.25e+08 POLYGON ((516478.2 280746.8...
    ## 4         1e+05   6.25e+08 POLYGON ((517910.6 279563.1...
    ## 5         1e+05   6.25e+08 POLYGON ((508458.2 277987.3...
    ## 6         1e+05   6.25e+08 POLYGON ((506951.4 278017.1...
    ## 7         1e+05   6.25e+08 POLYGON ((522824.6 277372.4...
    ## 8         1e+05   6.25e+08 POLYGON ((510162.4 287993.5...
    ## 9         1e+05   6.25e+08 POLYGON ((501018.4 279965.1...
    ## 10        1e+05   6.25e+08 POLYGON ((501059.8 280000, ...

``` r
lapa3334_clipping <- st_read_parquet("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3334clipping.parquet")
lapa3334_clipping
```

    ## Simple feature collection with 32904 features and 74 fields
    ## Geometry type: GEOMETRY
    ## Dimension:     XY
    ## Bounding box:  xmin: 525000 ymin: 275000 xmax: 550000 ymax: 3e+05
    ## Projected CRS: LKS-92 / Latvia TM
    ## First 10 features:
    ##          id objectid_1           adm1               adm2    atvk    kadastrs
    ## 1  51858796   51858796 Bauskas novads Vecumnieku pagasts 0025560 40940050048
    ## 2  99205937   99205937 Bauskas novads Vecumnieku pagasts 0025560 40940040006
    ## 3  49376379   49376379 Bauskas novads Vecumnieku pagasts 0025560 40940050026
    ## 4  49376380   49376380 Bauskas novads Vecumnieku pagasts 0025560 40940050026
    ## 5  49376381   49376381 Bauskas novads Vecumnieku pagasts 0025560 40940050026
    ## 6  49376382   49376382 Bauskas novads Vecumnieku pagasts 0025560 40940050026
    ## 7  49376383   49376383 Bauskas novads Vecumnieku pagasts 0025560 40940050026
    ## 8  49376384   49376384 Bauskas novads Vecumnieku pagasts 0025560 40940050027
    ## 9  49376385   49376385 Bauskas novads Vecumnieku pagasts 0025560 40940050027
    ## 10 49376484   49376484 Bauskas novads Vecumnieku pagasts 0025560 40940050027
    ##     gtf kvart nog anog nog_plat expl_mezs expl_celi expl_gravj zkat mt izc s10
    ## 1  2012     1  12    0     0.08      0.00         0       0.00   32  0   0   0
    ## 2  2020     1  79    0     0.07      0.07         0       0.00   10  4   1   1
    ## 3  2003     1   2    0     0.99      0.99         0       0.00   10  4   1   1
    ## 4  2003     1   3    0     0.34      0.34         0       0.00   10 15   1   4
    ## 5  2003     1   4    0     1.75      1.75         0       0.00   10  4   1   1
    ## 6  2003     1   5    0     0.64      0.64         0       0.00   10  4   1   1
    ## 7  2003     1   6    0     0.43      0.43         0       0.00   10  4   1   3
    ## 8  2003     2   1    0     0.23      0.20         0       0.03   10  4   1   1
    ## 9  2003     2   2    0     1.00      0.92         0       0.08   10  4   1   4
    ## 10 2003     2   3    0     0.37      0.32         0       0.05   10  4   1   1
    ##    a10 h10 d10 g10 n10 bv10 ba10 s11 a11 h11 d11 g11 n11 bv11 ba11 s12 a12 h12
    ## 1    0   0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 2   58  26  27  30   0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 3  112  29  35  25   0    0    0   3  92  29  30  11   0    0    0   0   0   0
    ## 4   77  21  23  22   0    0    0   1  77  21  20   7   0    0    0   0   0   0
    ## 5   87  27  33  15   0    0    0   3  87  27  29  15   0    0    0   4  87  29
    ## 6   82  28  35  26   0    0    0   4  82  28  28   3   0    0    0   0   0   0
    ## 7   72  24  35  33   0    0    0   1  72  23  22  12   0    0    0   4  72  28
    ## 8   82  29  34  33   0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 9   60  27  30  13   0    0    0   3  60  27  33   5   0    0    0   1  60  28
    ## 10  80  28  38  24   0    0    0   4  80  27  27   8   0    0    0   0   0   0
    ##    d12 g12 n12 bv12 ba12 s13 a13 h13 d13 g13 n13 bv13 ba13 s14 a14 h14 d14 g14
    ## 1    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 2    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 3    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 4    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 5   28   8   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 6    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 7   27   5   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 8    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 9   28   1   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 10   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ##    n14 bv14 ba14 jakopj jaatjauno p_darbv p_darbg p_cirp p_cirg atj_gads
    ## 1    0    0    0      0         0       0       0      0      0        0
    ## 2    0    0    0      0         0       1    2023     22   2023        0
    ## 3    0    0    0      0         0       1    2005     72   2005        0
    ## 4    0    0    0      0         0       0       0      0      0        0
    ## 5    0    0    0      0         0       1    2005     72   2005        0
    ## 6    0    0    0      0         0       1    2005     72   2005        0
    ## 7    0    0    0      0         0       1    2005     72   2005        0
    ## 8    0    0    0      0         0       1    2003     22   2003        0
    ## 9    0    0    0      0         0       1    2005     72   2005        0
    ## 10   0    0    0      0         0       1    2005     72   2005        0
    ##    saimn_d_ie plant_audz forestry_c vmd_headfo shape_Leng shape_Area NOSAUKUMS
    ## 1           6          0       2651     Centra   113.7443   757.9693      Ogre
    ## 2           4          0       2651     Centra   117.4964   733.5307      Ogre
    ## 3           6          0       2651     Centra   527.1829  9914.3136      Ogre
    ## 4           6          0       2651     Centra   345.5902  3391.7567      Ogre
    ## 5           6          0       2651     Centra   819.5234 17477.8172      Ogre
    ## 6           6          0       2651     Centra   408.2227  6394.0473      Ogre
    ## 7           6          0       2651     Centra   282.9741  4302.5068      Ogre
    ## 8           6          0       2651     Centra   248.8586  2314.4887      Ogre
    ## 9           6          0       2651     Centra   572.4052  9989.9399      Ogre
    ## 10          6          0       2651     Centra   248.5897  3666.7202      Ogre
    ##    NUMURS Shape_Length Shape_Area                       geometry
    ## 1    3334        1e+05   6.25e+08 POLYGON ((527081.2 276160.8...
    ## 2    3334        1e+05   6.25e+08 POLYGON ((525854.7 276484.6...
    ## 3    3334        1e+05   6.25e+08 POLYGON ((528193.2 277387.2...
    ## 4    3334        1e+05   6.25e+08 POLYGON ((528314.1 277266.1...
    ## 5    3334        1e+05   6.25e+08 POLYGON ((528134.5 277310.1...
    ## 6    3334        1e+05   6.25e+08 POLYGON ((528084.7 277253.5...
    ## 7    3334        1e+05   6.25e+08 POLYGON ((528250.5 277179, ...
    ## 8    3334        1e+05   6.25e+08 POLYGON ((527729.7 277384.5...
    ## 9    3334        1e+05   6.25e+08 POLYGON ((527748 277385, 52...
    ## 10   3334        1e+05   6.25e+08 POLYGON ((527832.9 277387, ...

``` r
lapa3343_clipping <- st_read_parquet("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3343clipping.parquet")
lapa3343_clipping
```

    ## Simple feature collection with 21466 features and 74 fields
    ## Geometry type: GEOMETRY
    ## Dimension:     XY
    ## Bounding box:  xmin: 550000 ymin: 275000 xmax: 575000 ymax: 3e+05
    ## Projected CRS: LKS-92 / Latvia TM
    ## First 10 features:
    ##           id objectid_1         adm1               adm2    atvk    kadastrs
    ## 1   51261610   51261610 Ogres novads  Madlienas pagasts 0040470 74680090078
    ## 2   99827232   99827232 Ogres novads     Krapes pagasts 0040420 74520030004
    ## 3  109753047  109753047 Ogres novads  Jumpravas pagasts 0040410 74480010075
    ## 4  114256226  114256226 Ogres novads Lielv?rdes pagasts 0040460 74330020794
    ## 5  138000835  138000835 Ogres novads   Lauberes pagasts 0040440 74600020018
    ## 6  140683354  140683354 Ogres novads   L?dmanes pagasts 0040450 74640040028
    ## 7  148912986  148912986 Ogres novads   Rembates pagasts 0040510 74840030197
    ## 8   49341737   49341737 Ogres novads  Jumpravas pagasts 0040410 74480020203
    ## 9   49376744   49376744 Ogres novads  Jumpravas pagasts 0040410 74480020087
    ## 10  49376745   49376745 Ogres novads  Jumpravas pagasts 0040410 74480020087
    ##     gtf kvart nog anog nog_plat expl_mezs expl_celi expl_gravj zkat mt izc s10
    ## 1  2015     1   7    0     0.15      0.00         0       0.00   42  0   0   0
    ## 2  2019     1   7    0     0.49      0.49         0       0.00   14 10   1   0
    ## 3  2020     1   6    0     0.34      0.34         0       0.00   10 21   1   3
    ## 4  2004     1   3    0     0.25      0.25         0       0.00   10  3   1   1
    ## 5  2010   336  22    0     0.70      0.61         0       0.09   10 14   1   4
    ## 6  2023     1   3    0     0.34      0.31         0       0.03   10  5   1   4
    ## 7  2024     1   3    0     0.17      0.17         0       0.00   10  5   1   4
    ## 8  2002     1   1    0     0.61      0.56         0       0.05   10 21   1   9
    ## 9  2003     5   3    0     0.33      0.33         0       0.00   10  5   1   9
    ## 10 2003     5   6    0     0.86      0.86         0       0.00   10  5   1   4
    ##    a10 h10 d10 g10  n10 bv10 ba10 s11 a11 h11 d11 g11 n11 bv11 ba11 s12 a12 h12
    ## 1    0   0   0   0    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 2    0   0   0   0    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 3   37  16  20  29    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 4    3   1   1   0 3000    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 5   34  15  15  11    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 6   68  28  29  13    0    0    0   8  68  29  35   1   0    0    0   0   0   0
    ## 7   65  26  26  20    0    0    0   8  65  28  31   2   0    0    0   0   0   0
    ## 8   14   8  11  16    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 9    7   4   4   0 2450    0    0   3   7   2   3   0 350    0    0  16   7   1
    ## 10   7   4   4   0 2100    0    0   8   7   5   5   0 500    0    0   9   7   4
    ##    d12 g12 n12 bv12 ba12 s13 a13 h13 d13 g13 n13 bv13 ba13 s14 a14 h14 d14 g14
    ## 1    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 2    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 3    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 4    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 5    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 6    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 7    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 8    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 9    2   0 100    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 10   4   0 100    0    0   6   7   3   3   0 100    0    0   0   0   0   0   0
    ##    n14 bv14 ba14 jakopj jaatjauno p_darbv p_darbg p_cirp p_cirg atj_gads
    ## 1    0    0    0      0         0       0       0      0      0        0
    ## 2    0    0    0      0      2024       1    2019     11   2019        0
    ## 3    0    0    0      0         0       0       0      0      0        0
    ## 4    0    0    0   2032         0      10    2022     61   2021     2022
    ## 5    0    0    0      0         0       0       0     21   2004        0
    ## 6    0    0    0      0         0       1    2004     22   2004        0
    ## 7    0    0    0      0         0       0       0      0      0        0
    ## 8    0    0    0      0         0       4    2015     11   2010     2015
    ## 9    0    0    0      0         0       6    2023     11   2014     2019
    ## 10   0    0    0   2024         0       6    2023     11   2014     2019
    ##    saimn_d_ie plant_audz forestry_c vmd_headfo shape_Leng shape_Area NOSAUKUMS
    ## 1           6          0       2654     Centra   195.0041   1537.971  Skrīveri
    ## 2           6          0       2654     Centra   280.3220   4888.540  Skrīveri
    ## 3           6          0       2654     Centra   236.6700   3430.599  Skrīveri
    ## 4           6          0       2654     Centra   604.4541   2495.869  Skrīveri
    ## 5           4          0       2654     Centra   378.8590   7042.177  Skrīveri
    ## 6           6          0       2654     Centra   249.4908   3434.501  Skrīveri
    ## 7           6          0       2654     Centra   183.8692   1691.748  Skrīveri
    ## 8           6          0       2654     Centra   326.7961   6085.816  Skrīveri
    ## 9           6          0       2654     Centra   253.8150   3345.459  Skrīveri
    ## 10          6          0       2654     Centra   378.7362   8641.671  Skrīveri
    ##    NUMURS Shape_Length Shape_Area                       geometry
    ## 1    3343        1e+05   6.25e+08 POLYGON ((570054.2 294762.8...
    ## 2    3343        1e+05   6.25e+08 POLYGON ((572603.3 291388, ...
    ## 3    3343        1e+05   6.25e+08 POLYGON ((558628 288978.4, ...
    ## 4    3343        1e+05   6.25e+08 POLYGON ((552875.9 285809, ...
    ## 5    3343        1e+05   6.25e+08 POLYGON ((562492.1 299370.8...
    ## 6    3343        1e+05   6.25e+08 POLYGON ((554104.7 292468.8...
    ## 7    3343        1e+05   6.25e+08 POLYGON ((552703 294195.9, ...
    ## 8    3343        1e+05   6.25e+08 POLYGON ((558967 279895.8, ...
    ## 9    3343        1e+05   6.25e+08 POLYGON ((559176.2 280277.6...
    ## 10   3343        1e+05   6.25e+08 POLYGON ((559155.2 280147.2...

``` r
lapa3344_clipping <- st_read_parquet("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3344clipping.parquet")
lapa3344_clipping
```

    ## Simple feature collection with 7874 features and 74 fields
    ## Geometry type: GEOMETRY
    ## Dimension:     XY
    ## Bounding box:  xmin: 575000 ymin: 284200.4 xmax: 592046.2 ymax: 3e+05
    ## Projected CRS: LKS-92 / Latvia TM
    ## First 10 features:
    ##           id objectid_1         adm1              adm2    atvk    kadastrs  gtf
    ## 1   93980758   93980758 Ogres novads  Me??eles pagasts 0040490 74760050006 2019
    ## 2   94092550   94092550 Ogres novads    Krapes pagasts 0040420 74520080085 2011
    ## 3  125451619  125451619 Ogres novads  Me??eles pagasts 0040490 74760040057 2015
    ## 4  138760213  138760213 Ogres novads  Me??eles pagasts 0040490 74760050003 2023
    ## 5   49402087   49402087 Ogres novads  Me??eles pagasts 0040490 74760030016 2004
    ## 6   49402090   49402090 Ogres novads  Me??eles pagasts 0040490 74760030016 2004
    ## 7   49477291   49477291 Ogres novads Madlienas pagasts 0040470 74680040083 2000
    ## 8   49477292   49477292 Ogres novads Madlienas pagasts 0040470 74680040083 2000
    ## 9   49476682   49476682 Ogres novads  Me??eles pagasts 0040490 74760040276 2005
    ## 10  49476683   49476683 Ogres novads  Me??eles pagasts 0040490 74760040276 2005
    ##    kvart nog anog nog_plat expl_mezs expl_celi expl_gravj zkat mt izc s10 a10
    ## 1      1   1    0     0.51      0.51         0       0.00   10 25   1   4  67
    ## 2      2  15    0     0.08      0.00         0       0.00   22  5   1   0   0
    ## 3      2   7    0     0.19      0.17         0       0.02   14 21   1   0   0
    ## 4      1  28    0     4.39      4.39         0       0.00   14 19   0   0   0
    ## 5      1   8    0     0.21      0.00         0       0.00   22  0   0   0   0
    ## 6      1  13    0     0.23      0.00         0       0.00   22  0   0   0   0
    ## 7      2   3    0     0.23      0.23         0       0.00   10 21   1   8  23
    ## 8      2   2    0     0.89      0.88         0       0.01   10  5   1   8  23
    ## 9      1   1    0     0.99      0.93         0       0.06   10 24   1   4  26
    ## 10     1   3    0     1.56      1.56         0       0.00   10 19   1   4  26
    ##    h10 d10 g10 n10 bv10 ba10 s11 a11 h11 d11 g11 n11 bv11 ba11 s12 a12 h12 d12
    ## 1   23  26  24   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 2    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 3    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 4    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 5    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 6    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 7   12  17  36   0    0    0   4  23  12  12   3   0    0    0   0   0   0   0
    ## 8   12  17  15   0    0    0   4  23  12  12  11   0    0    0   0   0   0   0
    ## 9   11  15  15   0    0    0   3  26   4   4   1   0    0    0   0   0   0   0
    ## 10  12  12  15   0    0    0   3  26   3   5   2   0    0    0   0   0   0   0
    ##    g12 n12 bv12 ba12 s13 a13 h13 d13 g13 n13 bv13 ba13 s14 a14 h14 d14 g14 n14
    ## 1    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 2    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 3    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 4    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 5    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 6    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 7    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 8    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 9    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 10   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ##    bv14 ba14 jakopj jaatjauno p_darbv p_darbg p_cirp p_cirg atj_gads saimn_d_ie
    ## 1     0    0      0         0       0       0      0      0        0          6
    ## 2     0    0      0         0       1    2012     22   2012        0          6
    ## 3     0    0      0      2026       1    2021     11   2021        0          6
    ## 4     0    0      0      2024       0       0     81   2019        0          6
    ## 5     0    0      0         0       0       0      0      0        0          6
    ## 6     0    0      0         0       0       0      0      0        0          6
    ## 7     0    0      0         0       4    2005     11   2001     2005          6
    ## 8     0    0      0         0       4    2005     11   2001     2005          6
    ## 9     0    0      0         0       1    2022     22   2022     2005          5
    ## 10    0    0      0         0       1    2022     22   2022     2005          5
    ##    plant_audz forestry_c vmd_headfo shape_Leng shape_Area NOSAUKUMS NUMURS
    ## 1           0       2654     Centra   328.1841   5146.992   Koknese   3344
    ## 2           0       2654     Centra   131.1396    756.648   Koknese   3344
    ## 3           0       2654     Centra   261.0201   1897.233   Koknese   3344
    ## 4           0       2654     Centra  8073.9639  43855.126   Koknese   3344
    ## 5           0       2654     Centra   233.9315   2077.357   Koknese   3344
    ## 6           0       2654     Centra   208.7556   2268.000   Koknese   3344
    ## 7           0       2654     Centra   249.8497   2332.361   Koknese   3344
    ## 8           0       2654     Centra   552.2459   8869.708   Koknese   3344
    ## 9           0       2654     Centra   879.4987   9895.713   Koknese   3344
    ## 10          0       2654     Centra  1052.0235  15585.167   Koknese   3344
    ##    Shape_Length Shape_Area                       geometry
    ## 1         1e+05   6.25e+08 POLYGON ((582103.4 293396.9...
    ## 2         1e+05   6.25e+08 POLYGON ((576177.8 287130.7...
    ## 3         1e+05   6.25e+08 POLYGON ((581976.6 296017.3...
    ## 4         1e+05   6.25e+08 POLYGON ((583430.8 292114.4...
    ## 5         1e+05   6.25e+08 POLYGON ((591632.9 299823.8...
    ## 6         1e+05   6.25e+08 POLYGON ((591686.9 299735.5...
    ## 7         1e+05   6.25e+08 POLYGON ((576056.9 299566, ...
    ## 8         1e+05   6.25e+08 POLYGON ((576057.8 299696.8...
    ## 9         1e+05   6.25e+08 POLYGON ((582861.2 296516.1...
    ## 10        1e+05   6.25e+08 POLYGON ((582882.8 296453.2...

Atbilde 2.1: Objektu skaits katrā lapā sakrīt ar iepriekšējā uzdevumā
iegūto objektu skaitu, objekti atrodas visā šūnā, arī pie robežām, bet
tie neiziet ārpus šūnas kā tas bija 1. apakšuzdevumā. St_intersection
izgrieza nogabalus pa kartes lapas robežu, kas nozīmē, ka ir mainījusies
objektu, kas atrodas uz lapas robežām platība un perimetrs

2.2. solis: izmantojiet pārskatīto funkciju no uzdevuma sākuma, lai
saglabātu katras karšu lapas rezultātu savā GeoTIFF failā, kura
nosaukums ir saistāms ar šo apakšuzdevumu un individuālo lapu. Kā
izskatās šūnu aizpildījums pie karšu lapu malām? Vai ir saglabājušies
objekti ārpus kartes lapām?

``` r
funkcija_parskatita_clipping <- function(ievades_fails) {
  starts <- Sys.time()
  lapa <- st_read_parquet(ievades_fails)
  mvr_lapa <- st_intersection(meza_centra_dati,lapa)
  mezaudze <- mvr_lapa %>%
    filter(s10 == 1)
  LV10m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV10m_10km.tif")
  rastrs_mezaudze_10m <- fasterize(mezaudze, LV10m_10km, background = 0)
  rastrs_mezaudze_10m[is.na(LV10m_10km)] <- NA
  LV100m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV100m_10km.tif")
  LV100m_10km_lapa <- crop(LV100m_10km,lapa)
  rastrs_mezaudze_100m <- resample(rastrs_mezaudze_10m,LV100m_10km_lapa, methode = "sum")
  plot(rastrs_mezaudze_100m)
  ievades_nosaukums <- file_path_sans_ext(basename(ievades_fails))
  izvades_fails <- file.path(paste0(ievades_nosaukums, "uzd22.parquet"))
  writeRaster(rastrs_mezaudze_100m, izvades_fails, format = "GTiff", overwrite = TRUE)
  finish <- Sys.time()
}
```

``` r
system.time(sapply(lapas_parqueti,funkcija_parskatita_clipping))
```

![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_22-1.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_22-2.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_22-3.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_22-4.png)<!-- -->

    ##    user  system elapsed 
    ## 1880.84  813.12 2745.64

Atbilde 2.2: Lapas malas ir aizpildītas pilnīgi, rezultāti no 1.2.
iegūtajiem neatšķiras. Objekti ārpus lapas nebija. Izmantojot
st_intersection var iegūt precīzākas platības nogabaliem, kas iekļaujas
konkrētajā lapā, kā arī tā, kā intersect izgriež datus pa lapu robežu -
izveido objektu, kas ir kopīgs abām daļām, tad nav nepieciešams 10m
rastru izgriezt pa lapas robežām, bet lai savienotu rastrus savā starpā
100m rastru tāpat ir nepieciešams izgriezt. 10m rastra neizgriešana
ievērojami palēnina apstrādes procesu, turklāt mēs neaprēķinām priežu
nogabalu platību 10m šūnā, bet skatamies vai 10m šūnā ir konkrētais
objekts. Līdz ar to spatial_join iedod to pašu rezultātu, bet ātrāk.

2.3. solis: apvienojiet iteratīvajā ciklā radītos GeoTIFF failus vienā
slānī, kura nosaukums ir saistāms ar apakšuzdevumu (laika mērogošanas
beigas). Vai apvienotajā slānī ir redzamas karšu lapu robežas? Kā
vērtības sakrīt ar citiem apakšuzdevumiem?

``` r
funkcija_parskatita_clipp <- function(ievades_fails) {
  starts <- Sys.time()
  lapa <- st_read_parquet(ievades_fails)
  mvr_lapa <- st_intersection(meza_centra_dati,lapa)
  mezaudze <- mvr_lapa %>%
    filter(s10 == 1)
  LV10m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV10m_10km.tif")
  rastrs_mezaudze_10m <- fasterize(mezaudze, LV10m_10km, background = 0)
  rastrs_mezaudze_10m[is.na(LV10m_10km)] <- NA
  LV100m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV100m_10km.tif")
  LV100m_10km_lapa <- crop(LV100m_10km,lapa)
  rastrs_mezaudze_100m <- resample(rastrs_mezaudze_10m,LV100m_10km_lapa, methode = "sum")
  plot(rastrs_mezaudze_100m)
  ievades_nosaukums <- file_path_sans_ext(basename(ievades_fails))
  izvades_fails <- file.path(paste0(ievades_nosaukums, "uzd22.parquet"))
  writeRaster(rastrs_mezaudze_100m, izvades_fails, format = "GTiff", overwrite = TRUE)
  Rastrs_lapa3333_clipp <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3333uzd22.tif")
Rastrs_lapa3334_clipp <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3334uzd22.tif")
Rastrs_lapa3343_clipp <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3343uzd22.tif")
Rastrs_lapa3344_clipp <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3344uzd22.tif")
pilna_teritorija <- merge(Rastrs_lapa3333_clipp,Rastrs_lapa3334_clipp,Rastrs_lapa3343_clipp,Rastrs_lapa3344_clipp)
plot(pilna_teritorija)
 writeRaster(pilna_teritorija, "otrais_apaksuzdevums", format = "GTiff", overwrite = TRUE)
  finish <- Sys.time()
}
```

``` r
system.time(sapply(lapas_parqueti,funkcija_parskatita_clipp))
```

![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_23-1.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_23-2.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_23-3.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_23-4.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_23-5.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_23-6.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_23-7.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_23-8.png)<!-- -->

    ##    user  system elapsed 
    ## 1887.28  724.18 2656.63

Atbilde 2.3: Lapu robežas nav redzamas, vērtības sakrīt ar 1.3.
iegūtajām vērtībām

3.1. solis: sāciet iteratīvo ciklu, kurā, izmantojot spatial filtering
iegūstiet MVR datus ar apstrādājamo kartes lapu (šis ir sākums laika
mērogošanai). Kāds ir objektu skaits katrā kartes lapā, kā tas saistās
ar iepriekšējo apakšuzdevumu? Ārpus cikla apskatiet katrai kartes lapai
piederīgos objektus - vai tie atrodas tikai kartes lapas iekšienē, vai
ir iekļauti visi objekti, kas ir uz kartes lapas robežām?

``` r
filtering_funkcija <- function(ievades_fails){
  starts <- Sys.time()
  lapa <- st_read_parquet(ievades_fails)
  mvr_lapa <- st_filter(meza_centra_dati,lapa,.pred = st_intersects)
  plot(mvr_lapa["geometry"])
  ievades_nosaukums <- file_path_sans_ext(basename(ievades_fails))
  izvades_fails <- file.path(paste0(ievades_nosaukums, "filter.parquet"))
  st_write_parquet(mvr_lapa,izvades_fails)
  finish <- Sys.time()
}
```

``` r
system.time(sapply(lapas_parqueti,filtering_funkcija))
```

![](Uzd05_Ozols_files/figure-gfm/mvrdati_lapas_clipping_31-1.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/mvrdati_lapas_clipping_31-2.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/mvrdati_lapas_clipping_31-3.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/mvrdati_lapas_clipping_31-4.png)<!-- -->

    ##    user  system elapsed 
    ##   34.63   12.31   49.81

``` r
lapa3333_filter <- st_read_parquet("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3333filter.parquet")
lapa3333_filter
```

    ## Simple feature collection with 33449 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 497543.3 ymin: 274441.5 xmax: 525438.7 ymax: 301517.7
    ## Projected CRS: LKS-92 / Latvia TM
    ## First 10 features:
    ##           id objectid_1           adm1            adm2    atvk    kadastrs  gtf
    ## 1   75507144   75507144 Olaines novads Olaines pagasts 0041400 80800130058 2017
    ## 2  108236414  108236414 Bauskas novads Iecavas pagasts 0025460 40640030055 2020
    ## 3  108236411  108236411 Bauskas novads Iecavas pagasts 0025460 40640030055 2020
    ## 4  140176213  140176213 Bauskas novads Iecavas pagasts 0025460 40640030202 2010
    ## 5  142493515  142493515 Bauskas novads Iecavas pagasts 0025460 40640070229 2023
    ## 6  148231191  148231191 Bauskas novads Iecavas pagasts 0025460 40640010146 2023
    ## 7  148584238  148584238 Bauskas novads Iecavas pagasts 0025460 40640050075 2024
    ## 8  149342953  149342953 ?ekavas novads ?ekavas pagasts 0034421 80700170050 2021
    ## 9   49358269   49358269 Bauskas novads Iecavas pagasts 0025460 40640010065 2002
    ## 10  49358270   49358270 Bauskas novads Iecavas pagasts 0025460 40640010065 2002
    ##    kvart nog anog nog_plat expl_mezs expl_celi expl_gravj zkat mt izc s10 a10
    ## 1    141   9    0     0.17      0.00      0.00       0.00   31  0   0   0   0
    ## 2      1   5    0     0.21      0.21      0.00       0.00   10  4   1   1  22
    ## 3      1   2    0     0.15      0.15      0.00       0.00   10  4   1   1  22
    ## 4      1   2    0     0.10      0.10      0.00       0.00   14 25   1   0   0
    ## 5     73   1    0     0.03      0.03      0.00       0.00   10  4   1   4  84
    ## 6     54  28    0     0.82      0.72      0.08       0.02   10  3   2   1  63
    ## 7    322   2    0     0.38      0.38      0.00       0.00   14 19   0   0   0
    ## 8    292  11    1     0.26      0.26      0.00       0.00   10 21   1   6  14
    ## 9      1   1    0     0.14      0.14      0.00       0.00   10  4   2   3  72
    ## 10     1   2    0     0.14      0.14      0.00       0.00   10  3   1   1  72
    ##    h10 d10 g10 n10 bv10 ba10 s11 a11 h11 d11 g11 n11 bv11 ba11 s12 a12 h12 d12
    ## 1    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 2    9  11  39   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 3    9  11  32   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 4    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 5   30  31  15   0    0    0   8  84  30  32   3   0    0    0   0   0   0   0
    ## 6   21  22  23   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 7    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 8   12   9  13   0    0    0   4  14  12   9   1   0    0    0   0   0   0   0
    ## 9   22  25  43   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 10  25  28  15   0    0    0   4  72  21  32   7   0    0    0   1 102  23  36
    ##    g12 n12 bv12 ba12 s13 a13 h13 d13 g13 n13 bv13 ba13 s14 a14 h14 d14 g14 n14
    ## 1    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 2    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 3    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 4    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 5    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 6    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 7    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 8    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 9    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 10   5   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ##    bv14 ba14 jakopj jaatjauno p_darbv p_darbg p_cirp p_cirg atj_gads saimn_d_ie
    ## 1     0    0      0         0       0       0      0      0        0          6
    ## 2     0    0      0         0       0       0      0      0        0          6
    ## 3     0    0      0         0       0       0      0      0        0          6
    ## 4     0    0      0      2029       1    2024     81   2024        0          6
    ## 5     0    0      0         0       0       0      0      0        0          6
    ## 6     0    0      0         0       1    2024     22   2024        0          6
    ## 7     0    0      0      2029       1    2024     11   2024        0          6
    ## 8     0    0      0         0       6    2021     11   2011     2013          6
    ## 9     0    0      0         0       0       0      0      0        0          6
    ## 10    0    0      0         0       0       0      0      0        0          6
    ##    plant_audz forestry_c vmd_headfo shape_Leng shape_Area
    ## 1           0       2651     Centra  207.08857  1698.6654
    ## 2           1       2651     Centra  240.55874  2123.5491
    ## 3           1       2651     Centra  202.33054  1456.8286
    ## 4           0       2651     Centra  145.20065  1002.2579
    ## 5           0       2651     Centra   96.23192   306.9282
    ## 6           0       2651     Centra  378.97765  8165.6619
    ## 7           0       2651     Centra  271.68713  3778.7076
    ## 8           0       2651     Centra  215.50695  2603.2586
    ## 9           0       2651     Centra  159.29683  1419.4795
    ## 10          0       2651     Centra  196.70426  1392.8598
    ##                          geometry
    ## 1  MULTIPOLYGON (((501810.4 29...
    ## 2  MULTIPOLYGON (((516402.9 28...
    ## 3  MULTIPOLYGON (((516444.4 28...
    ## 4  MULTIPOLYGON (((517894 2795...
    ## 5  MULTIPOLYGON (((508496.9 27...
    ## 6  MULTIPOLYGON (((507021.6 27...
    ## 7  MULTIPOLYGON (((522809.9 27...
    ## 8  MULTIPOLYGON (((510234.5 28...
    ## 9  MULTIPOLYGON (((501024.1 27...
    ## 10 MULTIPOLYGON (((501061.2 28...

``` r
lapa3334_filter <- st_read_parquet("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3334filter.parquet")
lapa3334_filter
```

    ## Simple feature collection with 32904 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 524686.8 ymin: 274567.8 xmax: 550368.6 ymax: 300505.8
    ## Projected CRS: LKS-92 / Latvia TM
    ## First 10 features:
    ##          id objectid_1           adm1               adm2    atvk    kadastrs
    ## 1  51858796   51858796 Bauskas novads Vecumnieku pagasts 0025560 40940050048
    ## 2  99205937   99205937 Bauskas novads Vecumnieku pagasts 0025560 40940040006
    ## 3  49376379   49376379 Bauskas novads Vecumnieku pagasts 0025560 40940050026
    ## 4  49376380   49376380 Bauskas novads Vecumnieku pagasts 0025560 40940050026
    ## 5  49376381   49376381 Bauskas novads Vecumnieku pagasts 0025560 40940050026
    ## 6  49376382   49376382 Bauskas novads Vecumnieku pagasts 0025560 40940050026
    ## 7  49376383   49376383 Bauskas novads Vecumnieku pagasts 0025560 40940050026
    ## 8  49376384   49376384 Bauskas novads Vecumnieku pagasts 0025560 40940050027
    ## 9  49376385   49376385 Bauskas novads Vecumnieku pagasts 0025560 40940050027
    ## 10 49376484   49376484 Bauskas novads Vecumnieku pagasts 0025560 40940050027
    ##     gtf kvart nog anog nog_plat expl_mezs expl_celi expl_gravj zkat mt izc s10
    ## 1  2012     1  12    0     0.08      0.00         0       0.00   32  0   0   0
    ## 2  2020     1  79    0     0.07      0.07         0       0.00   10  4   1   1
    ## 3  2003     1   2    0     0.99      0.99         0       0.00   10  4   1   1
    ## 4  2003     1   3    0     0.34      0.34         0       0.00   10 15   1   4
    ## 5  2003     1   4    0     1.75      1.75         0       0.00   10  4   1   1
    ## 6  2003     1   5    0     0.64      0.64         0       0.00   10  4   1   1
    ## 7  2003     1   6    0     0.43      0.43         0       0.00   10  4   1   3
    ## 8  2003     2   1    0     0.23      0.20         0       0.03   10  4   1   1
    ## 9  2003     2   2    0     1.00      0.92         0       0.08   10  4   1   4
    ## 10 2003     2   3    0     0.37      0.32         0       0.05   10  4   1   1
    ##    a10 h10 d10 g10 n10 bv10 ba10 s11 a11 h11 d11 g11 n11 bv11 ba11 s12 a12 h12
    ## 1    0   0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 2   58  26  27  30   0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 3  112  29  35  25   0    0    0   3  92  29  30  11   0    0    0   0   0   0
    ## 4   77  21  23  22   0    0    0   1  77  21  20   7   0    0    0   0   0   0
    ## 5   87  27  33  15   0    0    0   3  87  27  29  15   0    0    0   4  87  29
    ## 6   82  28  35  26   0    0    0   4  82  28  28   3   0    0    0   0   0   0
    ## 7   72  24  35  33   0    0    0   1  72  23  22  12   0    0    0   4  72  28
    ## 8   82  29  34  33   0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 9   60  27  30  13   0    0    0   3  60  27  33   5   0    0    0   1  60  28
    ## 10  80  28  38  24   0    0    0   4  80  27  27   8   0    0    0   0   0   0
    ##    d12 g12 n12 bv12 ba12 s13 a13 h13 d13 g13 n13 bv13 ba13 s14 a14 h14 d14 g14
    ## 1    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 2    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 3    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 4    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 5   28   8   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 6    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 7   27   5   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 8    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 9   28   1   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 10   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ##    n14 bv14 ba14 jakopj jaatjauno p_darbv p_darbg p_cirp p_cirg atj_gads
    ## 1    0    0    0      0         0       0       0      0      0        0
    ## 2    0    0    0      0         0       1    2023     22   2023        0
    ## 3    0    0    0      0         0       1    2005     72   2005        0
    ## 4    0    0    0      0         0       0       0      0      0        0
    ## 5    0    0    0      0         0       1    2005     72   2005        0
    ## 6    0    0    0      0         0       1    2005     72   2005        0
    ## 7    0    0    0      0         0       1    2005     72   2005        0
    ## 8    0    0    0      0         0       1    2003     22   2003        0
    ## 9    0    0    0      0         0       1    2005     72   2005        0
    ## 10   0    0    0      0         0       1    2005     72   2005        0
    ##    saimn_d_ie plant_audz forestry_c vmd_headfo shape_Leng shape_Area
    ## 1           6          0       2651     Centra   113.7443   757.9693
    ## 2           4          0       2651     Centra   117.4964   733.5307
    ## 3           6          0       2651     Centra   527.1829  9914.3136
    ## 4           6          0       2651     Centra   345.5902  3391.7567
    ## 5           6          0       2651     Centra   819.5234 17477.8172
    ## 6           6          0       2651     Centra   408.2227  6394.0473
    ## 7           6          0       2651     Centra   282.9741  4302.5068
    ## 8           6          0       2651     Centra   248.8586  2314.4887
    ## 9           6          0       2651     Centra   572.4052  9989.9399
    ## 10          6          0       2651     Centra   248.5897  3666.7202
    ##                          geometry
    ## 1  MULTIPOLYGON (((527092.6 27...
    ## 2  MULTIPOLYGON (((525860.8 27...
    ## 3  MULTIPOLYGON (((528135.1 27...
    ## 4  MULTIPOLYGON (((528313.6 27...
    ## 5  MULTIPOLYGON (((528135.2 27...
    ## 6  MULTIPOLYGON (((528057 2772...
    ## 7  MULTIPOLYGON (((528251 2771...
    ## 8  MULTIPOLYGON (((527732.2 27...
    ## 9  MULTIPOLYGON (((527745.8 27...
    ## 10 MULTIPOLYGON (((527824.5 27...

``` r
lapa3343_filter <- st_read_parquet("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3343clipping.parquet")
lapa3343_filter
```

    ## Simple feature collection with 21466 features and 74 fields
    ## Geometry type: GEOMETRY
    ## Dimension:     XY
    ## Bounding box:  xmin: 550000 ymin: 275000 xmax: 575000 ymax: 3e+05
    ## Projected CRS: LKS-92 / Latvia TM
    ## First 10 features:
    ##           id objectid_1         adm1               adm2    atvk    kadastrs
    ## 1   51261610   51261610 Ogres novads  Madlienas pagasts 0040470 74680090078
    ## 2   99827232   99827232 Ogres novads     Krapes pagasts 0040420 74520030004
    ## 3  109753047  109753047 Ogres novads  Jumpravas pagasts 0040410 74480010075
    ## 4  114256226  114256226 Ogres novads Lielv?rdes pagasts 0040460 74330020794
    ## 5  138000835  138000835 Ogres novads   Lauberes pagasts 0040440 74600020018
    ## 6  140683354  140683354 Ogres novads   L?dmanes pagasts 0040450 74640040028
    ## 7  148912986  148912986 Ogres novads   Rembates pagasts 0040510 74840030197
    ## 8   49341737   49341737 Ogres novads  Jumpravas pagasts 0040410 74480020203
    ## 9   49376744   49376744 Ogres novads  Jumpravas pagasts 0040410 74480020087
    ## 10  49376745   49376745 Ogres novads  Jumpravas pagasts 0040410 74480020087
    ##     gtf kvart nog anog nog_plat expl_mezs expl_celi expl_gravj zkat mt izc s10
    ## 1  2015     1   7    0     0.15      0.00         0       0.00   42  0   0   0
    ## 2  2019     1   7    0     0.49      0.49         0       0.00   14 10   1   0
    ## 3  2020     1   6    0     0.34      0.34         0       0.00   10 21   1   3
    ## 4  2004     1   3    0     0.25      0.25         0       0.00   10  3   1   1
    ## 5  2010   336  22    0     0.70      0.61         0       0.09   10 14   1   4
    ## 6  2023     1   3    0     0.34      0.31         0       0.03   10  5   1   4
    ## 7  2024     1   3    0     0.17      0.17         0       0.00   10  5   1   4
    ## 8  2002     1   1    0     0.61      0.56         0       0.05   10 21   1   9
    ## 9  2003     5   3    0     0.33      0.33         0       0.00   10  5   1   9
    ## 10 2003     5   6    0     0.86      0.86         0       0.00   10  5   1   4
    ##    a10 h10 d10 g10  n10 bv10 ba10 s11 a11 h11 d11 g11 n11 bv11 ba11 s12 a12 h12
    ## 1    0   0   0   0    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 2    0   0   0   0    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 3   37  16  20  29    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 4    3   1   1   0 3000    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 5   34  15  15  11    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 6   68  28  29  13    0    0    0   8  68  29  35   1   0    0    0   0   0   0
    ## 7   65  26  26  20    0    0    0   8  65  28  31   2   0    0    0   0   0   0
    ## 8   14   8  11  16    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 9    7   4   4   0 2450    0    0   3   7   2   3   0 350    0    0  16   7   1
    ## 10   7   4   4   0 2100    0    0   8   7   5   5   0 500    0    0   9   7   4
    ##    d12 g12 n12 bv12 ba12 s13 a13 h13 d13 g13 n13 bv13 ba13 s14 a14 h14 d14 g14
    ## 1    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 2    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 3    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 4    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 5    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 6    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 7    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 8    0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 9    2   0 100    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 10   4   0 100    0    0   6   7   3   3   0 100    0    0   0   0   0   0   0
    ##    n14 bv14 ba14 jakopj jaatjauno p_darbv p_darbg p_cirp p_cirg atj_gads
    ## 1    0    0    0      0         0       0       0      0      0        0
    ## 2    0    0    0      0      2024       1    2019     11   2019        0
    ## 3    0    0    0      0         0       0       0      0      0        0
    ## 4    0    0    0   2032         0      10    2022     61   2021     2022
    ## 5    0    0    0      0         0       0       0     21   2004        0
    ## 6    0    0    0      0         0       1    2004     22   2004        0
    ## 7    0    0    0      0         0       0       0      0      0        0
    ## 8    0    0    0      0         0       4    2015     11   2010     2015
    ## 9    0    0    0      0         0       6    2023     11   2014     2019
    ## 10   0    0    0   2024         0       6    2023     11   2014     2019
    ##    saimn_d_ie plant_audz forestry_c vmd_headfo shape_Leng shape_Area NOSAUKUMS
    ## 1           6          0       2654     Centra   195.0041   1537.971  Skrīveri
    ## 2           6          0       2654     Centra   280.3220   4888.540  Skrīveri
    ## 3           6          0       2654     Centra   236.6700   3430.599  Skrīveri
    ## 4           6          0       2654     Centra   604.4541   2495.869  Skrīveri
    ## 5           4          0       2654     Centra   378.8590   7042.177  Skrīveri
    ## 6           6          0       2654     Centra   249.4908   3434.501  Skrīveri
    ## 7           6          0       2654     Centra   183.8692   1691.748  Skrīveri
    ## 8           6          0       2654     Centra   326.7961   6085.816  Skrīveri
    ## 9           6          0       2654     Centra   253.8150   3345.459  Skrīveri
    ## 10          6          0       2654     Centra   378.7362   8641.671  Skrīveri
    ##    NUMURS Shape_Length Shape_Area                       geometry
    ## 1    3343        1e+05   6.25e+08 POLYGON ((570054.2 294762.8...
    ## 2    3343        1e+05   6.25e+08 POLYGON ((572603.3 291388, ...
    ## 3    3343        1e+05   6.25e+08 POLYGON ((558628 288978.4, ...
    ## 4    3343        1e+05   6.25e+08 POLYGON ((552875.9 285809, ...
    ## 5    3343        1e+05   6.25e+08 POLYGON ((562492.1 299370.8...
    ## 6    3343        1e+05   6.25e+08 POLYGON ((554104.7 292468.8...
    ## 7    3343        1e+05   6.25e+08 POLYGON ((552703 294195.9, ...
    ## 8    3343        1e+05   6.25e+08 POLYGON ((558967 279895.8, ...
    ## 9    3343        1e+05   6.25e+08 POLYGON ((559176.2 280277.6...
    ## 10   3343        1e+05   6.25e+08 POLYGON ((559155.2 280147.2...

``` r
lapa3344_filter <- st_read_parquet("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3344filter.parquet")
lapa3344_filter
```

    ## Simple feature collection with 7874 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 574406.4 ymin: 284200.4 xmax: 592046.2 ymax: 300327.3
    ## Projected CRS: LKS-92 / Latvia TM
    ## First 10 features:
    ##           id objectid_1         adm1              adm2    atvk    kadastrs  gtf
    ## 1   93980758   93980758 Ogres novads  Me??eles pagasts 0040490 74760050006 2019
    ## 2   94092550   94092550 Ogres novads    Krapes pagasts 0040420 74520080085 2011
    ## 3  125451619  125451619 Ogres novads  Me??eles pagasts 0040490 74760040057 2015
    ## 4  138760213  138760213 Ogres novads  Me??eles pagasts 0040490 74760050003 2023
    ## 5   49402087   49402087 Ogres novads  Me??eles pagasts 0040490 74760030016 2004
    ## 6   49402090   49402090 Ogres novads  Me??eles pagasts 0040490 74760030016 2004
    ## 7   49477291   49477291 Ogres novads Madlienas pagasts 0040470 74680040083 2000
    ## 8   49477292   49477292 Ogres novads Madlienas pagasts 0040470 74680040083 2000
    ## 9   49476682   49476682 Ogres novads  Me??eles pagasts 0040490 74760040276 2005
    ## 10  49476683   49476683 Ogres novads  Me??eles pagasts 0040490 74760040276 2005
    ##    kvart nog anog nog_plat expl_mezs expl_celi expl_gravj zkat mt izc s10 a10
    ## 1      1   1    0     0.51      0.51         0       0.00   10 25   1   4  67
    ## 2      2  15    0     0.08      0.00         0       0.00   22  5   1   0   0
    ## 3      2   7    0     0.19      0.17         0       0.02   14 21   1   0   0
    ## 4      1  28    0     4.39      4.39         0       0.00   14 19   0   0   0
    ## 5      1   8    0     0.21      0.00         0       0.00   22  0   0   0   0
    ## 6      1  13    0     0.23      0.00         0       0.00   22  0   0   0   0
    ## 7      2   3    0     0.23      0.23         0       0.00   10 21   1   8  23
    ## 8      2   2    0     0.89      0.88         0       0.01   10  5   1   8  23
    ## 9      1   1    0     0.99      0.93         0       0.06   10 24   1   4  26
    ## 10     1   3    0     1.56      1.56         0       0.00   10 19   1   4  26
    ##    h10 d10 g10 n10 bv10 ba10 s11 a11 h11 d11 g11 n11 bv11 ba11 s12 a12 h12 d12
    ## 1   23  26  24   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 2    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 3    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 4    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 5    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 6    0   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0
    ## 7   12  17  36   0    0    0   4  23  12  12   3   0    0    0   0   0   0   0
    ## 8   12  17  15   0    0    0   4  23  12  12  11   0    0    0   0   0   0   0
    ## 9   11  15  15   0    0    0   3  26   4   4   1   0    0    0   0   0   0   0
    ## 10  12  12  15   0    0    0   3  26   3   5   2   0    0    0   0   0   0   0
    ##    g12 n12 bv12 ba12 s13 a13 h13 d13 g13 n13 bv13 ba13 s14 a14 h14 d14 g14 n14
    ## 1    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 2    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 3    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 4    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 5    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 6    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 7    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 8    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 9    0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ## 10   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0   0
    ##    bv14 ba14 jakopj jaatjauno p_darbv p_darbg p_cirp p_cirg atj_gads saimn_d_ie
    ## 1     0    0      0         0       0       0      0      0        0          6
    ## 2     0    0      0         0       1    2012     22   2012        0          6
    ## 3     0    0      0      2026       1    2021     11   2021        0          6
    ## 4     0    0      0      2024       0       0     81   2019        0          6
    ## 5     0    0      0         0       0       0      0      0        0          6
    ## 6     0    0      0         0       0       0      0      0        0          6
    ## 7     0    0      0         0       4    2005     11   2001     2005          6
    ## 8     0    0      0         0       4    2005     11   2001     2005          6
    ## 9     0    0      0         0       1    2022     22   2022     2005          5
    ## 10    0    0      0         0       1    2022     22   2022     2005          5
    ##    plant_audz forestry_c vmd_headfo shape_Leng shape_Area
    ## 1           0       2654     Centra   328.1841   5146.992
    ## 2           0       2654     Centra   131.1396    756.648
    ## 3           0       2654     Centra   261.0201   1897.233
    ## 4           0       2654     Centra  8073.9639  43855.126
    ## 5           0       2654     Centra   233.9315   2077.357
    ## 6           0       2654     Centra   208.7556   2268.000
    ## 7           0       2654     Centra   249.8497   2332.361
    ## 8           0       2654     Centra   552.2459   8869.708
    ## 9           0       2654     Centra   879.4987   9895.713
    ## 10          0       2654     Centra  1052.0235  15585.167
    ##                          geometry
    ## 1  MULTIPOLYGON (((582123.8 29...
    ## 2  MULTIPOLYGON (((576215.9 28...
    ## 3  MULTIPOLYGON (((581964.3 29...
    ## 4  MULTIPOLYGON (((583363.9 29...
    ## 5  MULTIPOLYGON (((591625.8 29...
    ## 6  MULTIPOLYGON (((591687.6 29...
    ## 7  MULTIPOLYGON (((576056.7 29...
    ## 8  MULTIPOLYGON (((576057.1 29...
    ## 9  MULTIPOLYGON (((582500.7 29...
    ## 10 MULTIPOLYGON (((582827.5 29...

Atbilde 3.1: Objektu skaits sakrīt ar iepriekšējos apakšzudevumos
iegūtajiem objektu skaitiem, lapas malas ir aizpildītas pilnībā,
iekļauti ir objekti, kas atrodas uz lapas malas robežām, objekti iziet
ārpus lapas malas robežām, tāpēc nepieciešams izgriezt 10m šūnas
konkrētajai lapai, lai nebūtu objektu pārklāšanās.

3.2. solis: izmantojiet pārskatīto funkciju no uzdevuma sākuma, lai
saglabātu katras karšu lapas rezultātu savā GeoTIFF failā, kura
nosaukums ir saistāms ar šo apakšuzdevumu un individuālo lapu. Kā
izskatās šūnu aizpildījums pie karšu lapu malām? Vai ir saglabājušies
objekti ārpus kartes lapām?

``` r
funkcija_parskatita_filter <- function(ievades_fails) {
  starts <- Sys.time()
  lapa <- st_read_parquet(ievades_fails)
  mvr_lapa <- st_filter(meza_centra_dati,lapa,.pred = st_intersects)
  mezaudze <- mvr_lapa %>%
    filter(s10 == 1)
  LV10m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV10m_10km.tif")
  LV10m_10km_lapa <- crop(LV10m_10km,lapa)
  rastrs_mezaudze_10m <- fasterize(mezaudze, LV10m_10km_lapa, background = 0)
  rastrs_mezaudze_10m[is.na(LV10m_10km_lapa)] <- NA
  LV100m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV100m_10km.tif")
  LV100m_10km_lapa <- crop(LV100m_10km,lapa)
  rastrs_mezaudze_100m <- resample(rastrs_mezaudze_10m,LV100m_10km_lapa, methode = "sum")
  plot(rastrs_mezaudze_100m)
  ievades_nosaukums <- file_path_sans_ext(basename(ievades_fails))
  izvades_fails <- file.path(paste0(ievades_nosaukums, "uzd32.parquet"))
  writeRaster(rastrs_mezaudze_100m, izvades_fails, format = "GTiff", overwrite = TRUE)
}
```

``` r
system.time(sapply(lapas_parqueti,funkcija_parskatita_filter))
```

![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_filter-1.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_filter-2.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_filter-3.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_filter-4.png)<!-- -->

    ##    user  system elapsed 
    ##   35.87    4.45   40.53

Atbilde 3.2: Lapas aizpildītas pilnīgi, objekti ārpus lapu malām nav
saglabājušies 3.3. solis: apvienojiet iteratīvajā ciklā radītos GeoTIFF
failus vienā slānī, kura nosaukums ir saistāms ar apakšuzdevumu (laika
mērogošanas beigas). Vai apvienotajā slānī ir redzamas karšu lapu
robežas? Kā vērtības sakrīt ar citiem apakšuzdevumiem?

``` r
funkcija_parskatita_filter33 <- function(ievades_fails) {
  starts <- Sys.time()
  lapa <- st_read_parquet(ievades_fails)
  mvr_lapa <- st_filter(meza_centra_dati,lapa,.pred = intersects)
  mezaudze <- mvr_lapa %>%
    filter(s10 == 1)
  LV10m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV10m_10km.tif")
  LV10m_10km_lapa <- crop(LV10m_10km,lapa)
  rastrs_mezaudze_10m <- fasterize(mezaudze, LV10m_10km_lapa, background = 0)
  rastrs_mezaudze_10m[is.na(LV10m_10km_lapa)] <- NA
  LV100m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV100m_10km.tif")
  LV100m_10km_lapa <- crop(LV100m_10km,lapa)
  rastrs_mezaudze_100m <- resample(rastrs_mezaudze_10m,LV100m_10km_lapa, methode = "sum")
  plot(rastrs_mezaudze_100m)
  ievades_nosaukums <- file_path_sans_ext(basename(ievades_fails))
  izvades_fails <- file.path(paste0(ievades_nosaukums, "uzd32.parquet"))
  writeRaster(rastrs_mezaudze_100m, izvades_fails, format = "GTiff", overwrite = TRUE)
  Rastrs_lapa3333_filter <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3333uzd32.tif")
Rastrs_lapa3334_filter <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3334uzd32.tif")
Rastrs_lapa3343_filter <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3343uzd32.tif")
Rastrs_lapa3344_filter <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd05\\lapa3344uzd32.tif")
pilna_teritorija <- merge(Rastrs_lapa3333_filter,Rastrs_lapa3334_filter,Rastrs_lapa3343_filter,Rastrs_lapa3344_filter)
plot(pilna_teritorija)
 writeRaster(pilna_teritorija, "tresais_apaksuzdevums", format = "GTiff", overwrite = TRUE)
  finish <- Sys.time()
}
```

``` r
system.time(sapply(lapas_parqueti,funkcija_parskatita_filter33))
```

![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_3.3filter-1.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_3.3filter-2.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_3.3filter-3.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_3.3filter-4.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_3.3filter-5.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_3.3filter-6.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_3.3filter-7.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/rastri_lapam_3.3filter-8.png)<!-- -->

    ##    user  system elapsed 
    ##   39.45    4.27   43.61

Atbilde 3.3: Lapu malu robežas nav redzamas, vērtības sakrīt ar pārējiem
apakšuzdevumiem

Ceturtais apakšuzdevums. 4.1. solis: ar sevis izvēlētu pieeju atlasiet
mežaudzes, kuras pieder kādai no izvēlētajām karšu lapām (sākums laika
mērogošanai).

4.2. solis: ieviesiet funkciju atlasītajām mežaudzēm bez iteratīvā
procesa, bet nodrošinot rezultējošā rastra aptveri tikai izvēlētajām
kartes lapām (beigas laika mērogošanai). Kā vērtības sakrīt ar citiem
apakšuzdevumiem?

Atbilde: Ja es pareizi sapratu apakšuzdevumu, tad tas ir izpildīts jau
gatavojoties pirmajam apakšuzdevumam, izvēlētā pieeja ir spatial_join,
vērtības sakrita, izmēģināju ar argumentu st_within, tad lapu malu
aizpildījums nebija pilnīgs, jo neiekļāva objektus uz lapas robežām.
Piektais apakšuzdevums. 5.1. solis: ieviesiet funkciju visai MVR
informācijai (sākums laika mērogošanai).

``` r
ievades_fails <- st_read_parquet("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\tks93_50km.parquet")
ievades_fails <- st_transform(lapas, crs = 3059)
funkcija_5uzd <- function(ievades_fails){
  starts <- Sys.time()
  mvr_lapa <- st_join(meza_centra_dati,ievades_fails)
  mezaudze <- mvr_lapa %>%
    filter(s10 == 1)
  LV10m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV10m_10km.tif")
  rastrs_mezaudze_10m <- fasterize(mezaudze, LV10m_10km, background = 0)
  rastrs_mezaudze_10m[is.na(LV10m_10km)] <- NA
  LV100m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV100m_10km.tif")
  rastrs_mezaudze_100m <- resample(rastrs_mezaudze_10m,LV100m_10km, methode = "sum")
  plot(rastrs_mezaudze_100m)
  writeRaster(rastrs_mezaudze_100m, "5uzdevums", format = "GTiff", overwrite = TRUE)
  finish <- Sys.time()
  izveletas_lapas <- ievades_fails %>%
    filter(NUMURS == 3333 | NUMURS == 3334 | NUMURS == 3343 | NUMURS == 3344)
  plot(izveletas_lapas["NUMURS"])
  izveleta_teritorija <- crop(rastrs_mezaudze_100m,izveletas_lapas)
  plot(izveleta_teritorija)
  writeRaster(izveleta_teritorija, "52uzdevums", format = "GTiff", overwrite = TRUE)
}
```

``` r
system.time(funkcija_5uzd(ievades_fails))
```

![](Uzd05_Ozols_files/figure-gfm/5uzd_funkcija-1.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/5uzd_funkcija-2.png)<!-- -->![](Uzd05_Ozols_files/figure-gfm/5uzd_funkcija-3.png)<!-- -->

    ##    user  system elapsed 
    ##  833.78  274.93 1136.31

5.2. solis: no rezultējošā slāņa izgrieziet tikai to daļu, kas attiecas
uz izvēlētajām karšu lapām un beidziet laika mērogošanu. Ja šis solis ir
jau iestrādāts funkcijā, tad mērogojiet tikai pašu funkciju.

6.  Sestais - jaudīgiem datoriem un dažādu pieeju izmēģināšanai.
    Saistiet atbilstošo mežaudžu informāciju ar 100 m tīklu no projekta
    repozitorija un izveidojiet projekta repozitorijā esošajam 100 m
    rastra tīklam atbilstošu rastru pēc šūnu novietojuma un vērtībām.
    Kad iegūstat salīdzināmu rezultātu, mērogojiet tā izpildei
    nepieciešamo laiku. Kā vērtības sakrīt ar citiem apakšuzdevumiem?

``` r
m100_tikls <- st_read_parquet("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\tikls100_sauzeme.parquet")
m100_tikls <- st_transform(m100_tikls, crs = 3059)
savienots <- st_join(meza_centra_dati,m100_tikls,left=FALSE)
plot(savienots["geometry"])
```

![](Uzd05_Ozols_files/figure-gfm/sestais_apaksuzdevums-1.png)<!-- -->

``` r
priezu_audzes <- savienots %>% filter(s10 == 1)
LV10m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV10m_10km.tif")
rastrs_mezaudze_10m_5uzd <- fasterize(priezu_audzes, LV10m_10km, background = 0)
rastrs_mezaudze_10m_5uzd[is.na(LV10m_10km)] <- NA
LV100m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV100m_10km.tif")
rastrs_mezaudze_100m_5uzd <- resample(rastrs_mezaudze_10m_5uzd,LV100m_10km, methode = "sum")
plot(rastrs_mezaudze_100m_5uzd)
```

![](Uzd05_Ozols_files/figure-gfm/sestais_apaksuzdevums-2.png)<!-- -->

``` r
writeRaster(rastrs_mezaudze_100m_5uzd, "5uzd6apaksuzdevums", format = "GTiff", overwrite = TRUE)
```

Apskatīsimies kādu rezultātu iegūstam atšķirīgā veidā - izsakot priedes
īpatsvaru no šķērslaukuma, kad nogabalā priede kā valdošā suga ir vismaz
0.9 un šos rezultātus rastarizējot uz 100m rastru. Padodos - mēģināju
apskatīties ko iegūšu šādā veidā: m100_tikls \<-
st_read_parquet(“C:\Users\Ozols\Documents\HiQBioDiv_macibas\Uzd03\tikls100_sauzeme.parquet”)
m100_tikls \<- st_transform(m100_tikls, crs = 3059) savienots2 \<-
st_join(meza_centra_dati,m100_tikls,left=FALSE) savienots2 \<-
savienots2 %\>% mutate (priedes_skerslaukums = ifelse(s10 %in% c(“1”,
“14”, “22”), g10, 0) + ifelse(s11 %in% c(“1”, “14”, “22”), g11, 0) +
ifelse(s12 %in% c(“1”, “14”, “22”), g12, 0) + ifelse(s13 %in% c(“1”,
“14”, “22”), g13, 0) + ifelse(s14 %in% c(“1”, “14”, “22”), g14, 0))
savienots2 \<- savienots2 %\>% mutate (kop_skerslaukums =
g10+g11+g12+g13+g14) savienots2 \<- savienots2 %\>% mutate (priedes_prop
= priedes_skerslaukums/kop_skerslaukums) savienots2 \<- savienots2 %\>%
mutate(PriezuMezi = case_when(priedes_prop \>= 0.90 ~ 1, priedes_prop \<
0.90 & !is.na(priedes_prop) ~ 0, TRUE ~ NA_real\_)) LV100m_10km \<-
raster(“C:\Users\Ozols\Documents\HiQBioDiv_macibas\Uzd03\LV100m_10km.tif”)
priezu_rastrs \<- rasterize(savienots2,LV100m_10km, field =
“PriezuMezi”, fun = mean, background = 0)
priezu_rastrs\[is.na(LV100m_10km)\] \<- NA plot(priezu_rastrs) Process
griezās vairāk par 12h bez rezultātiem.
