Ceturtais uzdevums: funkcijas, cikli, vienkodola un daudzkodolu
skaitļošana
================
Jekaterīna Butkeviča
2025. gads 10. janvāris

Iepriekšejos uzdevumos neesmu saglabājusi apvienoto Centra
virsmežņiecības failu, tapēc izdarīšu to tagad pasleptājos komandu
blokos.

# Pirmais uzdevums

Sagatavojiet funkciju ar nosaukumu mana_funkcija, kura (secīgi): - no
piedāvātā MVR faila atlasa mežaudzes, kurās valdošā koku suga ir priede
(sugas kods ir “1”);

- sagatavo tādu rastru 10 m izšķirtspējā visai valsts teritorijai, kurā
  priežu mežaudzes no iepriekšējā punkta ir apzīmētas ar 1, pārējās
  Latvijas sauzsemes teritorijā esošās šūnas apzīmētas ar 0 un pārējās
  šūnas ir NA, un tas atbilst projekta Zenodo repozitorijā ievietotajam
  references slānim LV10m_10km.tif;

- iepriekšējā punkta rastru pārveido uz 100 m šūnu, aprēķinot priežu
  mežaudžu platības īpatsvaru (no kopējās platības) ik 100 m šūnā,
  nodrošinot atbilstību projekta Zenodo repozitorijā ievietotajam
  references slānim LV100m_10km.tif;

- saglabā iepriekšējā punktā izveidoto slāni (ar 100 m izšķirtspēju) kā
  GeoTIFF failu ar relatīvo ceļu norādītā vietā cietajā diskā, pieņemot
  ievades faila nosaukumu.

## Funkcijas izveide

Izveidosim funkciju.

``` r
mana_funkcija <- function(ievades_parquet, izvades_direktorija) {
  
  # Piefiksēt uzsākšanas laiku 
  stakuma_laiks <- Sys.time()
  
  #Aktivēt epieciešamās pakotnes
  library(sfarrow)
  library(raster)
  library(fasterize)
  library(tools)
  
  # 1. Ielasīt MVR datus no GeoParquet faila
  MVR_data <- st_read_parquet(ievades_parquet)
  
  # 2. Atlasīt tikai audzes, kur valdošā koku suga ir priede (sugas kods ir “1”)
  priede_mezaudzes <- MVR_data %>%
    filter(s10 == 1)
  
  # 3. Ielādēt 10m referenci
  ref_10m <- raster("C:/Users/user/Desktop/HiQBioDiv_macibas/3uzd/references_zenodo/rastrs/LV10m_10km.tif")
  
  # 4. Rastrēt mežu vektoru uz 10 metru izšķirtspēju
  rast_priede_mezaud_10m <- fasterize(priede_mezaudzes, ref_10m, background = 0)
  
  # 5. Nodrošināt NA ārpus LAtvijas
  rast_priede_mezaud_10m[is.na(ref_10m)] <- NA
  
  # 6. Samazināt izšķirtspēju uz 100m, summējot "1" šūnas
  rast_priede_mezaud_100m <- aggregate(rast_priede_mezaud_10m, fact = 10, fun = sum)
  
  # 7. Ielādēt 100m referenci
  ref_100m <- raster("C:/Users/user/Desktop/HiQBioDiv_macibas/3uzd/references_zenodo/rastrs/LV100m_10km.tif")
  
  # 8. Nodrošīnāt atbilstību 100m referencei
  
  rast_priede_mezaud_100m_valid <- resample(rast_priede_mezaud_100m, ref_100m, method = "bilinear")
  
  # 9. Apreķināt īpatsvaru
  
  rast_priede_mezaud_100m_valid_ipatsvars <- rast_priede_mezaud_100m_valid / 100
  
  # 10. Ģenerēt GeoTIFF faila nosaukumu no ievades faila nosaukuma
  ievades_basename <- file_path_sans_ext(basename(ievades_parquet)) # Noņem paplašinājumu
  izvades_fails <- file.path(izvades_direktorija, paste0(ievades_basename, ".tif"))
  
  # 11. Saglabāt 100m rastru kā GeoTIFF
  writeRaster(rast_priede_mezaud_100m_valid_ipatsvars, izvades_fails, format = "GTiff", overwrite = TRUE)
  
  #beigas laiks
  beigas_laiks <- Sys.time()
}
```

# Otrais uzdevums

Izpildes laika mērīšanu veicu, izmantojot funkciju {system.time()}.

``` r
dir_Centra <- "C:/Users/user/Desktop/HiQBioDiv_macibas/4uzd/Centra_visi.parquet"
dir_izvade <- "C:/Users/user/Desktop/HiQBioDiv_macibas/4uzd/"

system.time(mana_funkcija(dir_Centra, dir_izvade))
```

    


**Atbilde:** Funkcijas izpilde aizņēma 528,49 sekundes jeb 8,8 minūtes,
tika izmantoti 21,09 GB RAM, un R sesijā pastāvīgi bija nodarbināti 3
CPU, periodiski mazāk, bet vairāk laikam nebija. Papildus līdz 3 CPU
tika izmantoti uz System un Memory Compression.

# Trešais uzdevums

Sagatavojiet katras nodaļas MVR datus atsevišķā geoparquet failā, kura
nosaukums satur nodaļas nosaukumu. Iteratīva procesa, piemēram, for
cikla, veidā, ieviesiet sagatavoto funkciju katrai nodaļai atsevišķi.
Cik daudz laika aizņem šis uzdevums? Cik CPU kodolus tas aizņem
periodiski, cik - patstāvīgi? Vai pietika oeratīvās atmiņas uzdevuma
veikšanai, kādu tās apjomu R izmantoja?

``` r
dir_visas_nodalas_parquet <- "C:/Users/user/Desktop/HiQBioDiv_macibas/2uzd/centra_virsmezn_dati/parquet"
dir_nodalas_tif <- "C:/Users/user/Desktop/HiQBioDiv_macibas/4uzd/nodalas_tif"
dir.create(dir_nodalas_tif, recursive = TRUE)

visas_nodalas_parquet <- list.files(dir_visas_nodalas_parquet, full.names = TRUE)


system.time(
  for (katrs in visas_nodalas_parquet) {
    mana_funkcija(katrs, dir_nodalas_tif)
  }
  )
```



**Atbilde:** Funkcijas izpilde aizņēma 2226,89 sekundes jeb 37,11
minūtes, tika izmantoti 24,8 GB RAM, un R sesija pastāvīgi izmantoja 3
CPU kodolus, periodiski mazāk. Līdz 7 CPU aizņema System un Memory
Compression.

# Ceturtais uzdevums

Atkārtojiet iepriekšējo uzdevumu, izmantojot {doParallel} un {foreach},
bet nosakiet klāseteri kā tieši vienu CPU kodolu lielu. Cik daudz laika
aizņem šis uzdevums? Cik CPU kodolus tas aizņem periodiski, cik -
patstāvīgi? Vai pietika oeratīvās atmiņas uzdevuma veikšanai, kādu tās
apjomu R izmantoja?

``` r
library(doParallel)
library(foreach)

# Izveidot klāsteri ar vienu CPU kodolu
cl <- makeCluster(1) 
registerDoParallel(cl)

# Paralēlās apstrādes laika mērījums
laika_merijums <- system.time({
  foreach(katrs = visas_nodalas_parquet, .packages = c("sfarrow", "raster", "fasterize", "dplyr", "tools")) %dopar% {
    mana_funkcija(katrs, dir_nodalas_tif)
  }
})

# Izslēgt klāsteri
stopCluster(cl)

# Parādīt laika mērījumu
print(laika_merijums)
```

    ##    user  system elapsed 
    ##    0.25    0.31 1673.54

**Atbilde:** Neredzu nekādas īpašas atšķirības no iepriekšējā varianta.
Vienīgais, ka ar R sesiju iedarbinātie CPU nemainās un vienmēr ir
stabils skaits – 3. Izmantotas 24.2 GB no RAM. Patērētais laiks: 1670.99
sekundes jeb 27.85 minūtes.

# Piektais uzdevums

Atkārtojiet iepriekšējo uzdevumu paralēli uz vismaz diviem CPU kodoliem.
Cik daudz laika aizņem šis uzdevums? Cik CPU kodolus tas aizņem
periodiski, cik - patstāvīgi? Vai pietika oeratīvās atmiņas uzdevuma
veikšanai, kādu tās apjomu R izmantoja?

Izmeģināju ar četriem, bet saņēu kļūdu: Error in { : task 1 failed -
“cannot allocate vector of size 10.0 Gb”.

Veicu izmēģinājumu ar diviem:

``` r
library(doParallel)
library(foreach)

# Izveidot klāsteri ar vienu CPU kodolu
cl <- makeCluster(2) 
registerDoParallel(cl)

# Paralēlās apstrādes laika mērījums
laika_merijums <- system.time({
  foreach(katrs = visas_nodalas_parquet, .packages = c("sfarrow", "raster", "fasterize", "dplyr", "tools")) %dopar% {
    mana_funkcija(katrs, dir_nodalas_tif)
  }
})

# Izslēgt klāsteri
stopCluster(cl)

# Parādīt laika mērījumu
print(laika_merijums)
```



**Atbilde:** Pastāvīgi tika izmantoti 6 CPU: trīs un trīs paralēli, kā
arī periodiski vēl 1–3. Izmantotas 25,1 GB RAM. Patērētais laiks:
1077,97 sekundes jeb 17,97 minūtes.
