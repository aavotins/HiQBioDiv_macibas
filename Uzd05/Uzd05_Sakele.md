Uzd5_Sakele
================
Vita
2025-07-21

``` r
if (!require("sfarrow")) install.packages("sfarrow")
if (!require("sf")) install.packages("sf")
if (!require("terra")) install.packages("terra")
if (!require("dplyr")) install.packages("dplyr")
if (!require("raster")) install.packages("raster")
if (!require("tools")) install.packages("tools")
if (!require("fasterize")) install.packages("fasterize")
```

``` r
#datu sagatavošana, lejupielāde

if (!dir.exists("data")) {
  dir.create("data")
}

#Meža dati
forest_data <- st_read_parquet("../Uzd04/data/VMD/merged/validated/centra_parq.parquet")
forest_data <- forest_data[forest_data$zkat == 10, ] #lai ir tikai mežaudzes

st_write_parquet(forest_data, "data/forest_data.parquet")


#Kartes lapas
url= "https://zenodo.org/records/14277114/files/tks93_50km.parquet?download=1"
destfile = "data/tks93_50km.parquet"
download.file(url=url, destfile = destfile, mode='wb')

rm(forest_data, url, destfile)
```

``` r
forest_data<-st_read_parquet("data/forest_data.parquet")
```

``` r
map<-st_read_parquet("data/tks93_50km.parquet")
sheets <- map %>% filter(NUMURS %in% c(3331, 3332, 3333, 3334)) # izmantoju QGIS, lai izvēlētos
sheets<-st_transform(sheets, st_crs(forest_data)) #jāvienādo koordinātes, citādi vēlāk nestrādā

sheets_list <- split(sheets, sheets$NUMURS)

dir.create("data/sheet_prqs", recursive = TRUE)

for (nr in names(sheets_list)) {
  data <- sheets_list[[nr]]  
  file_name <- paste0("data/sheet_prqs/sheet_", nr, ".parquet") 
  st_write_parquet(data, file_name) 
}

sheets_list_paths <- list.files(
  path = "data/sheet_prqs",
  pattern = "^sheet_\\d{4}\\.parquet$", 
  full.names = TRUE)
```

``` r
rm(data, map, sheets_list, file_name, nr)
```

``` r
mana_funkcija<-function(input_file){
  output_dir<-paste0("data/results/")
  input_basename <- file_path_sans_ext(basename(input_file)) # Noņem paplašinājumu
  output_file <- file.path(output_dir, paste0(input_basename, ".tif"))
  
 
  data <- st_read_parquet(input_file)
  data<-data%>%
    filter(s10=="1") #tikai priedes
  
  #print(head(data, 20))
  
  raster_10m_10km = raster("../Uzd04/data/reference_raster/LV10m_10km.tif")
  raster_100m_10km = raster("../Uzd04/data/reference_raster/LV100m_10km.tif")
  #plot(raster_100m_10km)

  raster_10m_10km_cropped = terra::crop(raster_10m_10km, data)
  raster_100m_10km_cropped = terra::crop(raster_100m_10km, data)
  
  #plot(raster_100m_10km_cropped)  # apgrieztais apgabals
  
  raster_10m_10km_cropped1=raster::raster(raster_10m_10km_cropped)
  #plot(raster_100m_10km_cropped)
  
  forest_raster_10m <- fasterize(data, raster_10m_10km_cropped1, background = 0)
  #plot(forest_raster_10m) #ok
  
  forest_raster_10m[is.na(raster_10m_10km_cropped)] <- NA
  #plot(forest_raster_10m) #ok
  
  
  forest_raster_100m <- aggregate(forest_raster_10m, fact = 10, fun = mean)
  #plot(forest_raster_100m)
  
  #forest_raster_100m_res <- resample(forest_raster_100m, raster_100m_10km, method = "bilinear")
  #plot(forest_raster_100m_res)
  
  forest_raster_100m_res <- resample(forest_raster_100m, raster_100m_10km, 
                                    method = "bilinear",
                                   filename=output_file,
                                  overwrite=TRUE)
  
   return(TRUE)
}
```

## 1. apakšuzdevums

``` r
start_task1 <- Sys.time()

#1.1. st_join

dir.create("data/results", recursive = TRUE)

for (i in sheets_list_paths) {
  sheet <- st_read_parquet(i)
  data_joint <- st_join(forest_data, sheet, left = FALSE)
  
  
  basename <- file_path_sans_ext(basename(i))
  filename <- file.path("data/results", paste0(basename,"_forest_task1.parquet"))
  st_write_parquet(data_joint, filename)
  
  cat(basename, "_task1 objektu skaits: ", nrow(data_joint),", kolonnu skaits: ", ncol(data_joint), "\n", sep = "")
}
```

    ## sheet_3331_task1 objektu skaits: 14404, kolonnu skaits: 75

    ## sheet_3332_task1 objektu skaits: 26001, kolonnu skaits: 75

    ## sheet_3333_task1 objektu skaits: 30533, kolonnu skaits: 75

    ## sheet_3334_task1 objektu skaits: 30484, kolonnu skaits: 75

``` r
  cat("Meža datu objektu skaits: ",  nrow(forest_data),", kolonnu skaits: ", ncol(forest_data),"\n", sep = "")
```

    ## Meža datu objektu skaits: 466559, kolonnu skaits: 71

``` r
#1.2

task1_file_paths <- list.files("data/results", pattern = "^sheet_\\d{4}_forest_task1\\.parquet$", full.names = TRUE)

for (prq in task1_file_paths) {
    mana_funkcija(prq)
  }

#1.3 tif-u apvienošana

task1_tif_files <- list.files("data/results", pattern = "^sheet_\\d{4}_forest_task1\\.tif$", full.names = TRUE)
task1_rasters <- lapply(task1_tif_files, raster)

task1_merged <- do.call(mosaic, c(task1_rasters, fun = max)) 
writeRaster(task1_merged, "data/results/task1_merged.tif", overwrite = TRUE)


end_task1 <- Sys.time()

cat("1. apakšuzdevums izpildīts ", round(difftime(end_task1, start_task1, units = "mins"), 2), " minūtēs\n", sep = "")
```

    ## 1. apakšuzdevums izpildīts 1.69 minūtēs

``` r
rm(data_joint, sheet, task1_merged, task1_rasters, basename, end_task1, filename, i, prq, start_task1, task1_file_paths, task1_tif_files)
```

1.1. Ar QGIS apskatījos, ir iekļauti arī objekti, kas ir uz malām, t.i.
ir arī tādi, kas iet lapai pāri.

1.2. Izveidotie .tif ir lielāki par kartes lapām. Izskatās, ka lapas
malās esošās vērtības kaut ko neņem vērā (vai arī problēmas ar
vērtībām/robežpoligoniem, kas ir ārpus lapas, bet vēl ietilpst tif-ā).

1.3. apvienojot `fun = mean` var redzēt robežu starp lapām. Izmantojot
`fun = max`, robeža nav redzama. Tik vien cik malās var redzēt atšķirību
apvienoto lapu izmēros.

sheet_3331 \_task1 objektu skaits: 14404 , kolonnu skaits: 75

sheet_3332 \_task1 objektu skaits: 26001 , kolonnu skaits: 75

sheet_3333 \_task1 objektu skaits: 30533 , kolonnu skaits: 75

sheet_3334 \_task1 objektu skaits: 30484 , kolonnu skaits: 75

Meža datu objektu skaits: 466559 , kolonnu skaits: 71

1.  apakšuzdevuma izpildes laiks bija aptuveni 1.5 min (atšķirās starp
    mērījumiem).

## 2. apakšuzdevums

``` r
#2.1  intersection

start_task2 <- Sys.time()

for (i in sheets_list_paths) {
  sheet <- st_read_parquet(i)
  data_joint <- st_intersection(forest_data, sheet)
  
  basename <- file_path_sans_ext(basename(i))
  filename <- file.path("data/results", paste0(basename,"_forest_task2.parquet"))
  st_write_parquet(data_joint, filename)
  
  cat(basename, "_task2 objektu skaits: ", nrow(data_joint),", kolonnu skaits: ", ncol(data_joint), "\n", sep = "")
}
```

    ## sheet_3331_task2 objektu skaits: 14404, kolonnu skaits: 75

    ## sheet_3332_task2 objektu skaits: 26001, kolonnu skaits: 75

    ## sheet_3333_task2 objektu skaits: 30533, kolonnu skaits: 75

    ## sheet_3334_task2 objektu skaits: 30484, kolonnu skaits: 75

``` r
#2.2.

task2_file_paths <- list.files("data/results", pattern = "^sheet_\\d{4}_forest_task2\\.parquet$", full.names = TRUE)

for (prq in task2_file_paths) {
    mana_funkcija(prq)
  }



#2.3 tif-u apvienošana

task2_tif_files <- list.files("data/results", pattern = "^sheet_\\d{4}_forest_task2\\.tif$", full.names = TRUE)
task2_rasters <- lapply(task2_tif_files, raster)

task2_merged <- do.call(merge, task2_rasters) #merge, jo tif-i tieši saskaras
writeRaster(task2_merged, "data/results/task2_merged.tif", overwrite = TRUE)



end_task2 <- Sys.time()

cat("2. apakšuzdevums izpildīts ", round(difftime(end_task2, start_task2, units = "mins"), 2), " minūtēs\n", sep = "")
```

    ## 2. apakšuzdevums izpildīts 1.48 minūtēs

``` r
rm(data_joint, sheet, task2_merged, task2_rasters, basename, end_task2, filename, i, prq, start_task2, task2_file_paths, task2_tif_files)
```

2.1. Objektu skaits tāds pats kā ar `st_join`. Ir iekļauti objekti, kas
atrodas uz malām. Tieši atbilst lapas robežām.

sheet_3331_task2 objektu skaits: 14404, kolonnu skaits: 75

sheet_3332_task2 objektu skaits: 26001, kolonnu skaits: 75

sheet_3333_task2 objektu skaits: 30533, kolonnu skaits: 75

sheet_3334_task2 objektu skaits: 30484, kolonnu skaits: 75

2.2. Šūnu aizpildījums gar malām izskatās atbilstošs.

2.3. Salaiduma vieta nav manāma.

2.  apakšuzdevuma izpilde aizņēma līdzīgi - 1,5 minūtes. It kā ir
    neliels samazinājums laika patēriņā, tomēr ir pārāk maz mērījumu,
    lai par to droši spriestu. `merge` teorētiski vajadzētu būt ātrākai,
    jo pikseļi nav jāsalīdzina, bet tikai jāsaliek blakus. Iespējams, ka
    vienkārši ir pārāk maz salipināmo lapu, lai šo efektu novērotu.

## 3. uzdevums st_filter

Šim uzdevumam uztaisīšu divus variantus pirmo ar `st_within`, un otro -
`st_intersection`.

``` r
start_task3a <- Sys.time()

#3.1 st_filter ar st_within

for (i in sheets_list_paths) {
  sheet <- st_read_parquet(i)
  data_joint <- st_filter(forest_data, sheet, .predicate = st_within)
  
  basename <- file_path_sans_ext(basename(i))
  filename <- file.path("data/results", paste0(basename,"_forest_task3a.parquet"))
  st_write_parquet(data_joint, filename)
  
  cat(basename, "_task3a objektu skaits: ", nrow(data_joint),", kolonnu skaits: ", ncol(data_joint), "\n", sep = "")
}
```

    ## sheet_3331_task3a objektu skaits: 13992, kolonnu skaits: 71

    ## sheet_3332_task3a objektu skaits: 25258, kolonnu skaits: 71

    ## sheet_3333_task3a objektu skaits: 29901, kolonnu skaits: 71

    ## sheet_3334_task3a objektu skaits: 29751, kolonnu skaits: 71

``` r
#3.2

task3a_file_paths <- list.files("data/results", pattern = "^sheet_\\d{4}_forest_task3a\\.parquet$", full.names = TRUE)

for (prq in task3a_file_paths) {
    mana_funkcija(prq)
  }


#3.3 tif-u apvienošana

task3a_tif_files <- list.files("data/results", pattern = "^sheet_\\d{4}_forest_task3a\\.tif$", full.names = TRUE)
task3a_rasters <- lapply(task3a_tif_files, raster)

task3a_merged <- do.call(merge, task3a_rasters) #merge, jo tif-i tieši saskaras
writeRaster(task3a_merged, "data/results/task3a_merged.tif", overwrite = TRUE)


end_task3a <- Sys.time()

cat("3.a apakšuzdevums izpildīts ", round(difftime(end_task3a, start_task3a, units = "mins"), 2), " minūtēs\n", sep = "")
```

    ## 3.a apakšuzdevums izpildīts 1.4 minūtēs

``` r
rm(data_joint, sheet, task3a_merged, task3a_rasters, basename, end_task3a, filename, i, prq, start_task3a, task3a_file_paths, task3a_tif_files)
```

3.a) izmantojot st_within, kas atgriež tikai tos poligonus, kas ir
pilnībā lapā iekšā. Tie, kas ir kaut nedaudz ārpus lapas, netiek
iekļauti.

3.1 Objektu skaits katrā lapā ir par ~500 mazāks nekā abos iepriekšējos
uzdevumos. Kolonnu skaits ir tik pat liels kā meža datos (71), un nevis
75, kā tas bija abās iepriekšējās pieejās.

sheet_3331_task3 objektu skaits: 13992, kolonnu skaits: 71

sheet_3332_task3 objektu skaits: 25258, kolonnu skaits: 71

sheet_3333_task3 objektu skaits: 29901, kolonnu skaits: 71

sheet_3334_task3 objektu skaits: 29751, kolonnu skaits: 71

3.2. Ir iekļauti tikai tie objekti, kas pilnībā ietilpst lapā. Tā
rezultātā gar lapu malām ir pazaudēti dati.

3.3. Apvienojot ir nojaušamas vietas, kur lapas ir savienotas. Uzdevums
paveikts 1.5 minūtēs. Mazliet pārsreidza, tas, ka neskatoties uz mazāku
objektu skaitu, izpildes laiku tas nemaina.

``` r
start_task3b <- Sys.time()

#3.1 st_filter ar st_intersection

for (i in sheets_list_paths) {
  sheet <- st_read_parquet(i)
  data_joint <- st_filter(forest_data, sheet, .predicate = st_intersects)
  
  basename <- file_path_sans_ext(basename(i))
  filename <- file.path("data/results", paste0(basename,"_forest_task3b.parquet"))
  st_write_parquet(data_joint, filename)
  
  cat(basename, "_task3b objektu skaits: ", nrow(data_joint),", kolonnu skaits: ", ncol(data_joint), "\n", sep = "")
}
```

    ## sheet_3331_task3b objektu skaits: 14404, kolonnu skaits: 71

    ## sheet_3332_task3b objektu skaits: 26001, kolonnu skaits: 71

    ## sheet_3333_task3b objektu skaits: 30533, kolonnu skaits: 71

    ## sheet_3334_task3b objektu skaits: 30484, kolonnu skaits: 71

``` r
#3.2

task3b_file_paths <- list.files("data/results", pattern = "^sheet_\\d{4}_forest_task3b\\.parquet$", full.names = TRUE)

for (prq in task3b_file_paths) {
    mana_funkcija(prq)
  }


#3.3 tif-u apvienošana

task3b_tif_files <- list.files("data/results", pattern = "^sheet_\\d{4}_forest_task3b\\.tif$", full.names = TRUE)
task3b_rasters <- lapply(task3b_tif_files, raster)

task3b_merged <- do.call(mosaic, c(task3b_rasters, fun = max)) 
writeRaster(task3b_merged, "data/results/task3b_merged.tif", overwrite = TRUE)


end_task3b <- Sys.time()

cat("3.b apakšuzdevums izpildīts ", round(difftime(end_task3b, start_task3b, units = "mins"), 2), " minūtēs\n", sep = "")
```

    ## 3.b apakšuzdevums izpildīts 3.48 minūtēs

``` r
rm(data_joint, sheet, task3b_merged, task3b_rasters, basename, end_task3b, filename, i, prq, start_task3b, task3b_file_paths, task3b_tif_files)
```

3.b) 3.1. Ir tik pat objektu, cik ar `st_join` un `st_intersect`. Bet
kolonnu skaits ir 71, nevis 75. 3.2. Tifi iziet ārpus lapas. Bija
jāizmanto `mosaic`. 3.3. Uzdevums tika paveikts 1.5 min.

## 4. apakšuzdevums

Ja es pareizi saprotu, tad šajā uzdevumā vispirms izvēlas lapas, tad tās
apvieno un strādā, kā ar vienu lapu.

``` r
start_task4 <- Sys.time()

#izmatošu sheets - mainīgais no lapu atlases

data_joint <- st_intersection(forest_data, sheets)
filename <- file.path("data/results/task4.parquet")
st_write_parquet(data_joint, filename)

cat("task4 objektu skaits: ", nrow(data_joint),", kolonnu skaits: ", ncol(data_joint), "\n", sep = "")
```

    ## task4 objektu skaits: 101422, kolonnu skaits: 75

``` r
mana_funkcija(filename)
```

    ## [1] TRUE

``` r
end_task4 <- Sys.time()

cat("4. apakšuzdevums izpildīts ", round(difftime(end_task4, start_task4, units = "mins"), 2), " minūtēs\n", sep = "")
```

    ## 4. apakšuzdevums izpildīts 1.94 minūtēs

``` r
rm(data_joint, sheets, forest_data)
```

Mežu poligoni uz lapu robežas ir sadalīti.

task4 objektu skaits: 101422, kolonnu skaits: 75. Šis objektu skaits
atbilst objektu summai, kas ir bija telpiskajā savienošanā un griešanā.

Iegūtais tif ir bez redzamām lapu salaiduma vietām un tas precīzi
atbilst lapu robežām. Šajā variantā noteikti visprecīzāk ir aprēķināts
priežu mežu īpatsvars.

4.  apakšuzdevums izpildīts 0.83 min, kas ir visātrāk iegūtais
    rezultāts. Visi iepriekšējie bija gandrīz divas reizes ilgāki. Tā kā
    netaisīju atsevišķus laika mērījumus katram solim, tad atliek vien
    minēt, ka lapu salīmēšana un apvienotā faila izveidošana aizņem tik
    pat daudz laika, kā pārējā ģeoprocesēšana.

## 5. apakšuzdevums

``` r
start_task5 <- Sys.time()

mana_funkcija("data/forest_data.parquet")
```

    ## [1] TRUE

``` r
end_task5 <- Sys.time()

cat("5. apakšuzdevums izpildīts ", round(difftime(end_task5, start_task5, units = "mins"), 2), " minūtēs\n", sep = "")
```

    ## 5. apakšuzdevums izpildīts 8.8 minūtēs
