Uzd04_Ozols
================
Janis Ozols
2025-01-17

## Uzdevums

Uzdevuma veikšanai nepieciešami Valsts meža dienesta Meža Valsts
reģistra (MVR) Centra virsmežniecības dati no atvērto datu portāla, kas
izmantoti [otrajā uzdevumā](./Uzd02/Uzdevums02.md). 1. Sagatavojiet
funkciju ar nosaukumu `mana_funkcija`, kura (secīgi):

- no piedāvātā MVR faila atlasa mežaudzes, kurās valdošā koku suga ir
  priede (sugas kods ir “1”);

- sagatavo tādu rastru 10 m izšķirtspējā visai valsts teritorijai, kurā
  priežu mežaudzes no iepriekšējā punkta ir apzīmētas ar `1`, pārējās
  Latvijas sauzsemes teritorijā esošās šūnas apzīmētas ar `0` un pārējās
  šūnas ir `NA`, un tas atbilst [projekta *Zenodo*
  repozitorijā](https://zenodo.org/communities/hiqbiodiv/records?q=&l=list&p=1&s=10&sort=newest)
  ievietotajam [references slānim](https://zenodo.org/records/14497070)
  `LV10m_10km.tif`;

- iepriekšējā punkta rastru pārveido uz 100 m šūnu, aprēķinot priežu
  mežaudžu platības īpatsvaru (no kopējās platības) ik 100 m šūnā,
  nodrošinot atbilstību [projekta *Zenodo*
  repozitorijā](https://zenodo.org/communities/hiqbiodiv/records?q=&l=list&p=1&s=10&sort=newest)
  ievietotajam [references slānim](https://zenodo.org/records/14497070)
  `LV100m_10km.tif`;

- saglabā iepriekšējā punktā izveidoto slāni (ar 100 m izšķirtspēju) kā
  GeoTIFF failu ar relatīvo ceļu norādītā vietā cietajā diskā, pieņemot
  ievades faila nosaukumu.

Izveidojam funkcijas loģiku, lai saprastu, kuras daļas atkārtojas un
kuras ir nemainīgas. mana funkcija = ielasa failu, no ievadītā faila
atlasa koku sugu, sagatavo rastru 10m izšķirtspējā, kurā izvēlētā
dominantā koka mežaudzes ir apzīmētas ar viens, pārējās sauszemes
teritorijas apzīmē ar 0 un pārējās šūnas ar na, atbilst 10m references
slānim, pārveido uz 100m šūnu, aprēķinot izvēlētās koku sugas mežaudžu
platības īpatsvaru no kopējās platības ik 100m šūnā, nodrošina
atbilstību 100m references slānim, saglabā izveidoto slāni, kā geotif
slāni, iedodot tam izvades faila nosaukumu tajā ietverot saglabāšanas
direktoriju. Atkāpos no punkta, ka izveido failu ar ievades faila
nosaukumu, jo gribēju izveidot funkciju, kurai var mainīt valdošo koku
sugu kā argumentu, attiecīgi no viena ievades faila izveidojot
atšķirīgus izvades failus. Sadalam, kuras darbības ir argumenti un,
kuras ir funkcijas ķermenis. Argumenti: ielasītais fails, izvēlētā koku
suga, faila nosaukums Funkcijas loģika sagatavota, sagatavojam darba
vidi un nepieciešamos failus.

``` r
library(dplyr)
library(sf)
library(arrow)
library(sfarrow)
library(raster)
library(fasterize)
library(tools)
library(doParallel)
library(foreach)
```

Izveidojam funkciju

``` r
mana_funkcija <- function(ievades_fails,koku_suga,faila_nosaukums) {
  starts <- Sys.time()
  mvr_fails <- st_read_parquet(ievades_fails)
  mezaudze <- mvr_fails %>%
    filter(s10 == {koku_suga})
  LV10m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV10m_10km.tif")
  rastrs_mezaudze_10m <- fasterize(mezaudze, LV10m_10km, background = 0)
  rastrs_mezaudze_10m[is.na(LV10m_10km)] <- NA
  LV100m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV100m_10km.tif")
  rastrs_mezaudze_100m <- resample(rastrs_mezaudze_10m,LV100m_10km, methode = "sum")
  plot(rastrs_mezaudze_100m)
  writeRaster(rastrs_mezaudze_100m, faila_nosaukums, format = "GTiff", overwrite = TRUE)
  finish <- Sys.time()
}
```

Pārbaudam vai funkcija darbojas

``` r
funkcija_darbojas <- mana_funkcija(ievades_fails = "C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd04\\mezicentrs.parquet",koku_suga = 1,faila_nosaukums = "priezu_mezaudzes.tif")
```

![](Uzd04_Ozols_files/figure-gfm/funkcija_darbojas-1.png)<!-- -->
Paskatamies kādu rezultātu iegūst nomainot valdošo sugu

``` r
funkcija_egles <- mana_funkcija(ievades_fails = "C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd04\\mezicentrs.parquet",koku_suga = 3,faila_nosaukums = "eglu_mezaudzes.tif")
```

![](Uzd04_Ozols_files/figure-gfm/funkcija_egles-1.png)<!-- --> \#2.
Sagatavoto funkciju mana_funkcija izmēģiniet otrajā uzdevumā
sagatavotajam gqoparquet failam, kurā ir apvienotas visas Centrālās
virsmežniecības nodaļas. Cik daudz laika aizņem šis uzdevums? Cik CPU
kodolus tas aizņem periodiski, cik - patstāvīgi? Vai pietika oeratīvās
atmiņas uzdevuma veikšanai, kādu tās apjomu R izmantoja?

``` r
system.time(mana_funkcija(ievades_fails = "C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd04\\mezicentrs.parquet",koku_suga = 1,faila_nosaukums = "priezu_mezaudzes.tif"))
```

![](Uzd04_Ozols_files/figure-gfm/laiks-1.png)<!-- -->

    ##    user  system elapsed 
    ##  430.41  184.17  628.22

Atbilde: Uzdevums aizņēma ~10 min, aizņēma 1 CPU kodolu, operatīvā
atmiņa pietika, maksimums tika izmantoti 28gb. \#3. Sagatavojiet katras
nodaļas MVR datus atsevišķā geoparquet failā, kura nosaukums satur
nodaļas nosaukumu. Iteratīva procesa, piemēram, for cikla, veidā,
ieviesiet sagatavoto funkciju katrai nodaļai atsevišķi. Cik daudz laika
aizņem šis uzdevums? Cik CPU kodolus tas aizņem periodiski, cik -
patstāvīgi? Vai pietika oeratīvās atmiņas uzdevuma veikšanai, kādu tās
apjomu R izmantoja?

``` r
mvr_nodalas <- list.files("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd02\\centra_virsmezniecibas_dati",pattern = "\\.shp$")
dir.create(path="C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd04\\nodalas_parqueti")
mvr_nodalas
```

    ## [1] "nodala2651.shp" "nodala2652.shp" "nodala2653.shp" "nodala2654.shp"
    ## [5] "nodala2655.shp"

``` r
parquet <- function(ievades_fails,direktorija) {
  fails <- st_read(ievades_fails)
  ievades_nosaukums <- file_path_sans_ext(basename(ievades_fails))
  izvades_fails <- file.path(direktorija,paste0(ievades_nosaukums, ".parquet"))
  st_write_parquet(fails,izvades_fails)
}
direktorija <- "C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd04\\nodalas_parqueti"
sapply(mvr_nodalas,parquet,direktorija=direktorija)
```

    ## Reading layer `nodala2651' from data source 
    ##   `C:\Users\Ozols\Documents\HiQBioDiv_macibas\Uzd04\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM

    ## Reading layer `nodala2652' from data source 
    ##   `C:\Users\Ozols\Documents\HiQBioDiv_macibas\Uzd04\nodala2652.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 78469 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 420613.3 ymin: 241544.3 xmax: 507491.5 ymax: 305873.1
    ## Projected CRS: LKS-92 / Latvia TM

    ## Reading layer `nodala2653' from data source 
    ##   `C:\Users\Ozols\Documents\HiQBioDiv_macibas\Uzd04\nodala2653.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 112400 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 496133.2 ymin: 299013 xmax: 570134.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM

    ## Reading layer `nodala2654' from data source 
    ##   `C:\Users\Ozols\Documents\HiQBioDiv_macibas\Uzd04\nodala2654.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 113968 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 505799.1 ymin: 266426.8 xmax: 594509.3 ymax: 318075
    ## Projected CRS: LKS-92 / Latvia TM

    ## Reading layer `nodala2655' from data source 
    ##   `C:\Users\Ozols\Documents\HiQBioDiv_macibas\Uzd04\nodala2655.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 109454 features and 70 fields (with 6 geometries empty)
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 405229.7 ymin: 283087.1 xmax: 504232.4 ymax: 352909
    ## Projected CRS: LKS-92 / Latvia TM

    ##            nodala2651.shp         nodala2652.shp        
    ## id         numeric,91369          numeric,78469         
    ## objectid_1 character,91369        character,78469       
    ## adm1       character,91369        character,78469       
    ## adm2       character,91369        character,78469       
    ## atvk       character,91369        character,78469       
    ## kadastrs   character,91369        character,78469       
    ## gtf        numeric,91369          numeric,78469         
    ## kvart      character,91369        character,78469       
    ## nog        character,91369        character,78469       
    ## anog       character,91369        character,78469       
    ## nog_plat   numeric,91369          numeric,78469         
    ## expl_mezs  numeric,91369          numeric,78469         
    ## expl_celi  numeric,91369          numeric,78469         
    ## expl_gravj numeric,91369          numeric,78469         
    ## zkat       character,91369        character,78469       
    ## mt         character,91369        character,78469       
    ## izc        character,91369        character,78469       
    ## s10        character,91369        character,78469       
    ## a10        numeric,91369          numeric,78469         
    ## h10        numeric,91369          numeric,78469         
    ## d10        numeric,91369          numeric,78469         
    ## g10        numeric,91369          numeric,78469         
    ## n10        numeric,91369          numeric,78469         
    ## bv10       character,91369        character,78469       
    ## ba10       character,91369        character,78469       
    ## s11        character,91369        character,78469       
    ## a11        numeric,91369          numeric,78469         
    ## h11        numeric,91369          numeric,78469         
    ## d11        numeric,91369          numeric,78469         
    ## g11        numeric,91369          numeric,78469         
    ## n11        numeric,91369          numeric,78469         
    ## bv11       character,91369        character,78469       
    ## ba11       character,91369        character,78469       
    ## s12        character,91369        character,78469       
    ## a12        numeric,91369          numeric,78469         
    ## h12        numeric,91369          numeric,78469         
    ## d12        numeric,91369          numeric,78469         
    ## g12        numeric,91369          numeric,78469         
    ## n12        numeric,91369          numeric,78469         
    ## bv12       character,91369        character,78469       
    ## ba12       character,91369        character,78469       
    ## s13        character,91369        character,78469       
    ## a13        numeric,91369          numeric,78469         
    ## h13        numeric,91369          numeric,78469         
    ## d13        numeric,91369          numeric,78469         
    ## g13        numeric,91369          numeric,78469         
    ## n13        numeric,91369          numeric,78469         
    ## bv13       character,91369        character,78469       
    ## ba13       character,91369        character,78469       
    ## s14        character,91369        character,78469       
    ## a14        numeric,91369          numeric,78469         
    ## h14        numeric,91369          numeric,78469         
    ## d14        numeric,91369          numeric,78469         
    ## g14        numeric,91369          numeric,78469         
    ## n14        numeric,91369          numeric,78469         
    ## bv14       character,91369        character,78469       
    ## ba14       character,91369        character,78469       
    ## jakopj     numeric,91369          numeric,78469         
    ## jaatjauno  numeric,91369          numeric,78469         
    ## p_darbv    character,91369        character,78469       
    ## p_darbg    numeric,91369          numeric,78469         
    ## p_cirp     character,91369        character,78469       
    ## p_cirg     numeric,91369          numeric,78469         
    ## atj_gads   numeric,91369          numeric,78469         
    ## saimn_d_ie numeric,91369          numeric,78469         
    ## plant_audz character,91369        character,78469       
    ## forestry_c character,91369        character,78469       
    ## vmd_headfo character,91369        character,78469       
    ## shape_Leng numeric,91369          numeric,78469         
    ## shape_Area numeric,91369          numeric,78469         
    ## geometry   sfc_MULTIPOLYGON,91369 sfc_MULTIPOLYGON,78469
    ##            nodala2653.shp          nodala2654.shp         
    ## id         numeric,112400          numeric,113968         
    ## objectid_1 character,112400        character,113968       
    ## adm1       character,112400        character,113968       
    ## adm2       character,112400        character,113968       
    ## atvk       character,112400        character,113968       
    ## kadastrs   character,112400        character,113968       
    ## gtf        numeric,112400          numeric,113968         
    ## kvart      character,112400        character,113968       
    ## nog        character,112400        character,113968       
    ## anog       character,112400        character,113968       
    ## nog_plat   numeric,112400          numeric,113968         
    ## expl_mezs  numeric,112400          numeric,113968         
    ## expl_celi  numeric,112400          numeric,113968         
    ## expl_gravj numeric,112400          numeric,113968         
    ## zkat       character,112400        character,113968       
    ## mt         character,112400        character,113968       
    ## izc        character,112400        character,113968       
    ## s10        character,112400        character,113968       
    ## a10        numeric,112400          numeric,113968         
    ## h10        numeric,112400          numeric,113968         
    ## d10        numeric,112400          numeric,113968         
    ## g10        numeric,112400          numeric,113968         
    ## n10        numeric,112400          numeric,113968         
    ## bv10       character,112400        character,113968       
    ## ba10       character,112400        character,113968       
    ## s11        character,112400        character,113968       
    ## a11        numeric,112400          numeric,113968         
    ## h11        numeric,112400          numeric,113968         
    ## d11        numeric,112400          numeric,113968         
    ## g11        numeric,112400          numeric,113968         
    ## n11        numeric,112400          numeric,113968         
    ## bv11       character,112400        character,113968       
    ## ba11       character,112400        character,113968       
    ## s12        character,112400        character,113968       
    ## a12        numeric,112400          numeric,113968         
    ## h12        numeric,112400          numeric,113968         
    ## d12        numeric,112400          numeric,113968         
    ## g12        numeric,112400          numeric,113968         
    ## n12        numeric,112400          numeric,113968         
    ## bv12       character,112400        character,113968       
    ## ba12       character,112400        character,113968       
    ## s13        character,112400        character,113968       
    ## a13        numeric,112400          numeric,113968         
    ## h13        numeric,112400          numeric,113968         
    ## d13        numeric,112400          numeric,113968         
    ## g13        numeric,112400          numeric,113968         
    ## n13        numeric,112400          numeric,113968         
    ## bv13       character,112400        character,113968       
    ## ba13       character,112400        character,113968       
    ## s14        character,112400        character,113968       
    ## a14        numeric,112400          numeric,113968         
    ## h14        numeric,112400          numeric,113968         
    ## d14        numeric,112400          numeric,113968         
    ## g14        numeric,112400          numeric,113968         
    ## n14        numeric,112400          numeric,113968         
    ## bv14       character,112400        character,113968       
    ## ba14       character,112400        character,113968       
    ## jakopj     numeric,112400          numeric,113968         
    ## jaatjauno  numeric,112400          numeric,113968         
    ## p_darbv    character,112400        character,113968       
    ## p_darbg    numeric,112400          numeric,113968         
    ## p_cirp     character,112400        character,113968       
    ## p_cirg     numeric,112400          numeric,113968         
    ## atj_gads   numeric,112400          numeric,113968         
    ## saimn_d_ie numeric,112400          numeric,113968         
    ## plant_audz character,112400        character,113968       
    ## forestry_c character,112400        character,113968       
    ## vmd_headfo character,112400        character,113968       
    ## shape_Leng numeric,112400          numeric,113968         
    ## shape_Area numeric,112400          numeric,113968         
    ## geometry   sfc_MULTIPOLYGON,112400 sfc_MULTIPOLYGON,113968
    ##            nodala2655.shp         
    ## id         numeric,109454         
    ## objectid_1 character,109454       
    ## adm1       character,109454       
    ## adm2       character,109454       
    ## atvk       character,109454       
    ## kadastrs   character,109454       
    ## gtf        numeric,109454         
    ## kvart      character,109454       
    ## nog        character,109454       
    ## anog       character,109454       
    ## nog_plat   numeric,109454         
    ## expl_mezs  numeric,109454         
    ## expl_celi  numeric,109454         
    ## expl_gravj numeric,109454         
    ## zkat       character,109454       
    ## mt         character,109454       
    ## izc        character,109454       
    ## s10        character,109454       
    ## a10        numeric,109454         
    ## h10        numeric,109454         
    ## d10        numeric,109454         
    ## g10        numeric,109454         
    ## n10        numeric,109454         
    ## bv10       character,109454       
    ## ba10       character,109454       
    ## s11        character,109454       
    ## a11        numeric,109454         
    ## h11        numeric,109454         
    ## d11        numeric,109454         
    ## g11        numeric,109454         
    ## n11        numeric,109454         
    ## bv11       character,109454       
    ## ba11       character,109454       
    ## s12        character,109454       
    ## a12        numeric,109454         
    ## h12        numeric,109454         
    ## d12        numeric,109454         
    ## g12        numeric,109454         
    ## n12        numeric,109454         
    ## bv12       character,109454       
    ## ba12       character,109454       
    ## s13        character,109454       
    ## a13        numeric,109454         
    ## h13        numeric,109454         
    ## d13        numeric,109454         
    ## g13        numeric,109454         
    ## n13        numeric,109454         
    ## bv13       character,109454       
    ## ba13       character,109454       
    ## s14        character,109454       
    ## a14        numeric,109454         
    ## h14        numeric,109454         
    ## d14        numeric,109454         
    ## g14        numeric,109454         
    ## n14        numeric,109454         
    ## bv14       character,109454       
    ## ba14       character,109454       
    ## jakopj     numeric,109454         
    ## jaatjauno  numeric,109454         
    ## p_darbv    character,109454       
    ## p_darbg    numeric,109454         
    ## p_cirp     character,109454       
    ## p_cirg     numeric,109454         
    ## atj_gads   numeric,109454         
    ## saimn_d_ie numeric,109454         
    ## plant_audz character,109454       
    ## forestry_c character,109454       
    ## vmd_headfo character,109454       
    ## shape_Leng numeric,109454         
    ## shape_Area numeric,109454         
    ## geometry   sfc_MULTIPOLYGON,109454

``` r
nodalas <- list.files("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd04\\nodalas_parqueti",pattern = "\\.parquet$")
nodalas
```

    ## [1] "nodala2651.parquet" "nodala2652.parquet" "nodala2653.parquet"
    ## [4] "nodala2654.parquet" "nodala2655.parquet"

Lai funkciju darbinātu uz failu sarakstu, nepieciešams, lai funkcija
atgriež ievades faila nosaukumu, tāpēc modoficējam sākotnējo izveidoto
funkciju un nosaucam to par mana_funkcija2

``` r
mana_funkcija2 <- function(ievades_fails,direktorija) {
  starts <- Sys.time()
  mvr_fails <- st_read_parquet(ievades_fails)
  mezaudze <- mvr_fails %>%
    filter(s10 == 1)
  LV10m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV10m_10km.tif")
  rastrs_mezaudze_10m <- fasterize(mezaudze, LV10m_10km, background = 0)
  rastrs_mezaudze_10m[is.na(LV10m_10km)] <- NA
  LV100m_10km <- raster("C:\\Users\\Ozols\\Documents\\HiQBioDiv_macibas\\Uzd03\\LV100m_10km.tif")
  rastrs_mezaudze_100m <- resample(rastrs_mezaudze_10m,LV100m_10km, methode = "sum")
  plot(rastrs_mezaudze_100m)
  ievades_nosaukums <- file_path_sans_ext(basename(ievades_fails))
  izvades_fails <- file.path(paste0(direktorija,ievades_nosaukums, ".parquet"))
  writeRaster(rastrs_mezaudze_100m, izvades_fails, format = "GTiff", overwrite = TRUE)
  finish <- Sys.time()
}
```

``` r
system.time(sapply(nodalas,mana_funkcija2,direktorija=direktorija))
```

![](Uzd04_Ozols_files/figure-gfm/iterativais_process-1.png)<!-- -->![](Uzd04_Ozols_files/figure-gfm/iterativais_process-2.png)<!-- -->![](Uzd04_Ozols_files/figure-gfm/iterativais_process-3.png)<!-- -->![](Uzd04_Ozols_files/figure-gfm/iterativais_process-4.png)<!-- -->![](Uzd04_Ozols_files/figure-gfm/iterativais_process-5.png)<!-- -->

    ##    user  system elapsed 
    ## 2125.95 1445.90 3604.46

Atbilde: Uzdevums aizņēma ~ 64 min, patstāvīgi aizņēma 1 CPU kodolu, R
izmantoja 32gb operatīvās atmiņas \#4. Atkārtojiet iepriekšējo uzdevumu,
izmantojot {doParallel} un {foreach}, bet nosakiet klāseteri kā tieši
vienu CPU kodolu lielu. Cik daudz laika aizņem šis uzdevums? Cik CPU
kodolus tas aizņem periodiski, cik - patstāvīgi? Vai pietika oeratīvās
atmiņas uzdevuma veikšanai, kādu tās apjomu R izmantoja?

``` r
cluster1 <- makeCluster(1) 
registerDoParallel(cluster1)
laiks1 <- system.time({
  foreach(nodala = nodalas,.packages = c("sfarrow", "raster", "fasterize", "dplyr", "tools")) %dopar% {
    mana_funkcija2(nodala,direktorija)
    }
})
stopCluster(cluster1)
print(laiks1)
```

    ##    user  system elapsed 
    ##    0.01    0.05 3640.34

Atbilde: Strādāja uz 12 logical procesoriem, ilga ~68 minūtes, izmantoja
28 gb operatīvās atmiņas. \#5. Atkārtojiet iepriekšējo uzdevumu paralēli
uz vismaz diviem CPU kodoliem. Cik daudz laika aizņem šis uzdevums? Cik
CPU kodolus tas aizņem periodiski, cik - patstāvīgi? Vai pietika
oeratīvās atmiņas uzdevuma veikšanai, kādu tās apjomu R izmantoja?

``` r
cluster2 <- makeCluster(2) 
registerDoParallel(cluster2)
laiks2 <- system.time({
  foreach(nodala = nodalas,.packages = c("sfarrow", "raster", "fasterize", "dplyr", "tools")) %dopar% {
    mana_funkcija2(nodala,direktorija)
    }
})
stopCluster(cluster2)
print(laiks2)
```

    ##    user  system elapsed 
    ##    0.00    0.03 2222.55

Atbilde: Strādāja uz 6 kodoliem, 12 logical procesoriem, ilga ~36
minūtes, izmantoja 26 gb operatīvās atmiņas.
