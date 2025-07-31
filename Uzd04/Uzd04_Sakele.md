Uzd4_Sakele
================
Vita
2025-01-27

``` r
if (!require("sf")) install.packages("sf")
if (!require("tidyverse")) install.packages("tidyverse")
if (!require("RCurl")) install.packages("RCurl")
if (!require("archive")) install.packages("archive")
if (!require("sfarrow")) install.packages("sfarrow")
if (!require("raster")) install.packages("raster")
if (!require("fasterize")) install.packages("fasterize")
#if (!require("terra")) install.packages("terra")
if (!require("tools")) install.packages("tools")
```

    ## Loading required package: tools

``` r
library(tools)
if (!require("foreach")) install.packages("foreach")
if (!require("doParallel")) install.packages("doParallel")
if (!require("bigparallelr")) install.packages("bigparallelr")




# alternative
#if (!require("pacman")) install.packages("pacman")
#pacman::p_load(sf, tidyverse, RCurl, archive, sfarrow, raster, fasterize)
```

## Meža dati

``` r
if (!dir.exists("data/download")) {
  dir.create("data/download", recursive=TRUE)
  }

url= "https://data.gov.lv/dati/lv/dataset/40014c0a-90f5-42be-afb2-fe3c4b8adf92/resource/392dfb67-eeeb-43c2-b082-35f9cf986128/download/centra.7z"

destfile = "data/download/centra.7z"

download.file(url=url, destfile = destfile, mode='wb')



if (!dir.exists("data/VMD")) {
  dir.create("data/VMD")
}
archive_extract(destfile, dir = "data/VMD")
```

    ## ⠙ 1 extracted | 554 MB (260 MB/s) | 2.1s⠹ 1 extracted | 605 MB (243 MB/s) |
    ## 2.5s⠸ 1 extracted | 640 MB (237 MB/s) | 2.7s⠼ 1 extracted | 673 MB (232 MB/s) |
    ## 2.9s⠴ 1 extracted | 717 MB (231 MB/s) | 3.1s⠦ 2 extracted | 788 MB (238 MB/s) |
    ## 3.3s⠧ 5 extracted | 794 MB (225 MB/s) | 3.5s⠇ 5 extracted | 797 MB (213 MB/s) |
    ## 3.7s⠏ 5 extracted | 799 MB (203 MB/s) | 3.9s⠋ 5 extracted | 802 MB (193 MB/s) |
    ## 4.1s⠙ 5 extracted | 804 MB (185 MB/s) | 4.3s⠹ 5 extracted | 809 MB (178 MB/s) |
    ## 4.6s⠸ 5 extracted | 811 MB (170 MB/s) | 4.8s⠼ 5 extracted | 813 MB (160 MB/s) |
    ## 5.1s⠴ 5 extracted | 814 MB (157 MB/s) | 5.2s⠦ 9 extracted | 829 MB (154 MB/s) |
    ## 5.4s⠧ 9 extracted | 858 MB (153 MB/s) | 5.6s⠇ 9 extracted | 902 MB (155 MB/s) |
    ## 5.8s⠏ 9 extracted | 953 MB (158 MB/s) | 6s ⠋ 9 extracted | 999 MB (160 MB/s) |
    ## 6.2s⠙ 9 extracted | 1.0 GB (160 MB/s) | 6.4s⠹ 9 extracted | 1.1 GB (160 MB/s) |
    ## 6.6s⠸ 9 extracted | 1.1 GB (159 MB/s) | 6.8s⠼ 9 extracted | 1.1 GB (159 MB/s) |
    ## 7s ⠴ 9 extracted | 1.2 GB (161 MB/s) | 7.3s⠦ 9 extracted | 1.2 GB (165 MB/s) |
    ## 7.5s⠧ 9 extracted | 1.2 GB (158 MB/s) | 7.9s⠇ 9 extracted | 1.3 GB (160 MB/s) |
    ## 8.1s⠏ 9 extracted | 1.3 GB (162 MB/s) | 8.3s⠋ 9 extracted | 1.4 GB (167 MB/s) |
    ## 8.5s⠙ 9 extracted | 1.5 GB (169 MB/s) | 8.7s⠹ 13 extracted | 1.5 GB (168 MB/s)
    ## | 8.9s⠸ 13 extracted | 1.5 GB (165 MB/s) | 9.1s⠼ 13 extracted | 1.5 GB (162
    ## MB/s) | 9.3s⠴ 13 extracted | 1.5 GB (159 MB/s) | 9.5s⠦ 13 extracted | 1.5 GB
    ## (156 MB/s) | 9.7s⠧ 13 extracted | 1.5 GB (152 MB/s) | 9.9s⠇ 13 extracted | 1.5
    ## GB (150 MB/s) | 10.1s⠏ 13 extracted | 1.5 GB (147 MB/s) | 10.4s⠋ 15 extracted |
    ## 1.5 GB (144 MB/s) | 10.6s⠙ 17 extracted | 1.5 GB (144 MB/s) | 10.8s⠹ 17
    ## extracted | 1.6 GB (144 MB/s) | 11s ⠸ 17 extracted | 1.6 GB (145 MB/s) | 11.2s⠼
    ## 17 extracted | 1.7 GB (146 MB/s) | 11.4s⠴ 17 extracted | 1.7 GB (147 MB/s) |
    ## 11.6s⠦ 17 extracted | 1.7 GB (147 MB/s) | 11.8s⠧ 17 extracted | 1.8 GB (147
    ## MB/s) | 12s ⠇ 17 extracted | 1.8 GB (147 MB/s) | 12.2s⠏ 17 extracted | 1.8 GB
    ## (147 MB/s) | 12.4s⠋ 17 extracted | 1.9 GB (148 MB/s) | 12.6s⠙ 17 extracted |
    ## 1.9 GB (150 MB/s) | 12.8s⠹ 17 extracted | 2.0 GB (151 MB/s) | 13s ⠸ 17
    ## extracted | 2.0 GB (152 MB/s) | 13.2s⠼ 17 extracted | 2.1 GB (154 MB/s) |
    ## 13.4s⠴ 17 extracted | 2.1 GB (154 MB/s) | 13.7s⠦ 17 extracted | 2.1 GB (154
    ## MB/s) | 13.9s⠧ 17 extracted | 2.2 GB (154 MB/s) | 14.1s⠇ 17 extracted | 2.2 GB
    ## (155 MB/s) | 14.3s⠏ 17 extracted | 2.2 GB (155 MB/s) | 14.5s⠋ 17 extracted |
    ## 2.3 GB (156 MB/s) | 14.7s⠙ 17 extracted | 2.3 GB (157 MB/s) | 14.9s⠹ 17
    ## extracted | 2.4 GB (158 MB/s) | 15.1s⠸ 17 extracted | 2.4 GB (159 MB/s) |
    ## 15.3s⠼ 19 extracted | 2.5 GB (161 MB/s) | 15.5s⠴ 21 extracted | 2.5 GB (159
    ## MB/s) | 15.7s⠦ 21 extracted | 2.5 GB (157 MB/s) | 15.9s⠧ 21 extracted | 2.5 GB
    ## (156 MB/s) | 16.1s⠇ 21 extracted | 2.5 GB (154 MB/s) | 16.3s⠏ 21 extracted |
    ## 2.5 GB (152 MB/s) | 16.5s⠋ 21 extracted | 2.5 GB (151 MB/s) | 16.8s⠙ 21
    ## extracted | 2.5 GB (149 MB/s) | 17s ⠹ 21 extracted | 2.5 GB (147 MB/s) | 17.2s⠸
    ## 21 extracted | 2.5 GB (146 MB/s) | 17.4s⠼ 23 extracted | 2.5 GB (144 MB/s) |
    ## 17.6s⠴ 25 extracted | 2.6 GB (144 MB/s) | 17.8s⠦ 25 extracted | 2.6 GB (144
    ## MB/s) | 18s ⠧ 25 extracted | 2.6 GB (144 MB/s) | 18.2s⠇ 25 extracted | 2.7 GB
    ## (144 MB/s) | 18.4s⠏ 25 extracted | 2.7 GB (144 MB/s) | 18.6s⠋ 25 extracted |
    ## 2.7 GB (144 MB/s) | 18.8s⠙ 25 extracted | 2.7 GB (144 MB/s) | 19s ⠹ 25
    ## extracted | 2.8 GB (144 MB/s) | 19.2s⠸ 25 extracted | 2.8 GB (145 MB/s) |
    ## 19.5s⠼ 25 extracted | 2.8 GB (145 MB/s) | 19.7s⠴ 25 extracted | 2.9 GB (146
    ## MB/s) | 19.9s⠦ 25 extracted | 2.9 GB (144 MB/s) | 20.1s⠧ 25 extracted | 2.9 GB
    ## (144 MB/s) | 20.3s⠇ 25 extracted | 3.0 GB (145 MB/s) | 20.5s⠏ 25 extracted |
    ## 3.0 GB (146 MB/s) | 20.7s⠋ 25 extracted | 3.1 GB (147 MB/s) | 20.9s⠙ 25
    ## extracted | 3.1 GB (147 MB/s) | 21.1s⠹ 25 extracted | 3.1 GB (147 MB/s) |
    ## 21.3s⠸ 25 extracted | 3.2 GB (148 MB/s) | 21.5s⠼ 25 extracted | 3.2 GB (149
    ## MB/s) | 21.7s⠴ 25 extracted | 3.3 GB (151 MB/s) | 21.9s⠦ 25 extracted | 3.4 GB
    ## (153 MB/s) | 22.1s⠧ 25 extracted | 3.4 GB (152 MB/s) | 22.4s⠇ 25 extracted |
    ## 3.4 GB (151 MB/s) | 22.6s⠏ 25 extracted | 3.5 GB (152 MB/s) | 22.8s⠋ 25
    ## extracted | 3.5 GB (152 MB/s) | 23s ⠙ 25 extracted | 3.5 GB (150 MB/s) | 23.5s⠹
    ## 26 extracted | 3.5 GB (148 MB/s) | 23.9s⠸ 27 extracted | 3.5 GB (147 MB/s) |
    ## 24s ⠼ 29 extracted | 3.5 GB (146 MB/s) | 24.2s⠴ 29 extracted | 3.5 GB (145
    ## MB/s) | 24.4s⠦ 29 extracted | 3.5 GB (144 MB/s) | 24.6s⠧ 29 extracted | 3.5 GB
    ## (143 MB/s) | 24.8s⠇ 29 extracted | 3.5 GB (142 MB/s) | 25.1s⠏ 29 extracted |
    ## 3.5 GB (141 MB/s) | 25.3s⠋ 29 extracted | 3.6 GB (139 MB/s) | 25.5s⠙ 29
    ## extracted | 3.6 GB (139 MB/s) | 25.7s⠹ 29 extracted | 3.6 GB (138 MB/s) |
    ## 25.9s⠸ 33 extracted | 3.6 GB (138 MB/s) | 26.1s⠼ 33 extracted | 3.7 GB (139
    ## MB/s) | 26.3s⠴ 33 extracted | 3.7 GB (138 MB/s) | 26.6s⠦ 33 extracted | 3.7 GB
    ## (136 MB/s) | 27s ⠧ 33 extracted | 3.7 GB (135 MB/s) | 27.2s⠇ 33 extracted | 3.7
    ## GB (134 MB/s) | 27.4s⠏ 33 extracted | 3.7 GB (133 MB/s) | 27.6s⠋ 33 extracted |
    ## 3.7 GB (133 MB/s) | 27.8s⠙ 33 extracted | 3.7 GB (132 MB/s) | 28s ⠹ 33
    ## extracted | 3.7 GB (132 MB/s) | 28.2s⠸ 33 extracted | 3.7 GB (132 MB/s) |
    ## 28.4s⠼ 33 extracted | 3.8 GB (132 MB/s) | 28.6s⠴ 33 extracted | 3.8 GB (132
    ## MB/s) | 28.8s⠦ 33 extracted | 3.8 GB (133 MB/s) | 29s ⠧ 33 extracted | 3.9 GB
    ## (132 MB/s) | 29.2s⠇ 33 extracted | 3.9 GB (133 MB/s) | 29.5s⠏ 33 extracted |
    ## 3.9 GB (132 MB/s) | 29.7s⠋ 33 extracted | 4.0 GB (133 MB/s) | 29.9s⠙ 33
    ## extracted | 4.0 GB (133 MB/s) | 30.1s⠹ 33 extracted | 4.0 GB (133 MB/s) |
    ## 30.3s⠸ 33 extracted | 4.1 GB (133 MB/s) | 30.5s⠼ 33 extracted | 4.1 GB (133
    ## MB/s) | 30.7s⠴ 33 extracted | 4.1 GB (133 MB/s) | 30.9s⠦ 33 extracted | 4.2 GB
    ## (133 MB/s) | 31.1s⠧ 33 extracted | 4.2 GB (133 MB/s) | 31.3s⠇ 33 extracted |
    ## 4.2 GB (134 MB/s) | 31.5s⠏ 33 extracted | 4.3 GB (134 MB/s) | 31.8s⠋ 33
    ## extracted | 4.3 GB (135 MB/s) | 32s ⠙ 33 extracted | 4.4 GB (136 MB/s) | 32.2s⠹
    ## 33 extracted | 4.4 GB (136 MB/s) | 32.4s⠸ 33 extracted | 4.5 GB (137 MB/s) |
    ## 32.6s⠼ 35 extracted | 4.5 GB (138 MB/s) | 32.8s⠴ 37 extracted | 4.5 GB (137
    ## MB/s) | 33s ⠦ 37 extracted | 4.5 GB (136 MB/s) | 33.2s⠧ 37 extracted | 4.5 GB
    ## (135 MB/s) | 33.4s⠇ 37 extracted | 4.5 GB (135 MB/s) | 33.6s⠏ 37 extracted |
    ## 4.5 GB (134 MB/s) | 33.8s⠋ 37 extracted | 4.5 GB (133 MB/s) | 34s ⠙ 37
    ## extracted | 4.5 GB (133 MB/s) | 34.2s⠹ 37 extracted | 4.5 GB (132 MB/s) |
    ## 34.4s⠸ 37 extracted | 4.5 GB (131 MB/s) | 34.6s⠼ 37 extracted | 4.5 GB (130
    ## MB/s) | 34.8s⠴ 39 extracted | 4.5 GB (130 MB/s) | 35.1s

``` r
if (!dir.exists("data/VMD/merged")) {
  dir.create("data/VMD/merged")
}
 
file_list <- list.files("data/VMD", pattern = "*shp$", full.names = TRUE)
shapefile_list <- lapply(file_list, read_sf) #ielasa visus failus
centra_shp <- do.call(rbind, shapefile_list) # apvieno vienā failā

st_write_parquet(centra_shp, "data/VMD/merged/centra.parquet") # ieraksta apvienoto failu, kā parquet uz diska

rm(centra_shp, shapefile_list) #iztīrīt atmiņu

centra_parq<- st_read_parquet("data/VMD/merged/centra.parquet")


# Pārbaudīt, vai visas ģeometrijas ir MULTIPOLYGON
if(all(st_geometry_type(centra_parq) == "MULTIPOLYGON")) {
  print("Visas ģeometrijas ir MULTIPOLYGON")
} else {
  print("Dažas ģeometrijas nav MULTIPOLYGON")
  }

# Pārbaudīt, vai nav tukšu ģeometriju
if(any(st_is_empty(centra_parq))) {
  print("Slānī ir tukšas ģeometrijas")
  # Izdzēst tukšās ģeometrijas
  centra_parq <- centra_parq[!st_is_empty(centra_parq), ] 
}

# Pārbaudīt, vai nav kļūdainu ģeometriju
if(any(!st_is_valid(centra_parq))) {
  print("Slānī ir kļūdainas ģeometrijas")
  # Salabot un izdzēst kļūdainās ģeometrijas
  centra_parq <- st_make_valid(centra_parq)
  print("ir validētas")
  centra_parq <- centra_parq[st_is_valid(centra_parq), ] 
}


if (!dir.exists("data/VMD/merged/validated")) {
  dir.create("data/VMD/merged/validated")
}


st_write_parquet(centra_parq, "data/VMD/merged/validated/centra_parq.parquet")
rm(centra_parq)
```

## References rastri

``` r
if (!dir.exists("data/reference_raster")) {
  dir.create("data/reference_raster")
}

url= "https://zenodo.org/records/14497070/files/LV10m_10km.tif?download=1"
destfile = "data/reference_raster/LV10m_10km.tif"
download.file(url=url, destfile = destfile, mode='wb')
rm(url, destfile)

url= "https://zenodo.org/records/14497070/files/LV100m_10km.tif?download=1"
destfile = "data/reference_raster/LV100m_10km.tif"
download.file(url=url, destfile = destfile, mode='wb')
rm(url, destfile)
```

## Mana funkcija

``` r
mana_funkcija1<-function(input_file){
  
  
  output_dir<-"data/VMD/tifs"
  input_basename <- file_path_sans_ext(basename(input_file)) # Noņem paplašinājumu
  output_file <- file.path(output_dir, paste0(input_basename, ".tif"))
  
 
  data <- st_read_parquet(input_file)
  data<-data%>%
    filter(zkat=="10"& s10=="1") #tikai priedes
  
  
  raster_10m_10km = raster("data/reference_raster/LV10m_10km.tif")
  raster_100m_10km = raster("data/reference_raster/LV100m_10km.tif")
  


  forest_raster_10m <- fasterize(data, raster_10m_10km, background = 0)
  forest_raster_10m[is.na(raster_10m_10km)] <- NA
  
  forest_raster_100m <- aggregate(forest_raster_10m, fact = 10, fun = mean)
  #forest_raster_100m_res <- resample(forest_raster_100m, raster_100m_10km, method = "bilinear")
  
  
  forest_raster_100m_res <- resample(forest_raster_100m, raster_100m_10km, 
                                     method = "bilinear",
                                     filename=output_file,
                                     overwrite=TRUE)
  return(TRUE)


#  writeRaster(forest_raster_100m_res, output_file, format = "GTiff", overwrite = TRUE)
  
}
```

## Mana funkcija tikai ar samazinātu teritoriju

Tā kā nevienu reizi neizdevās izpildīt `mana_funkcija()`, tad izveidoju
funkciju, kas dara to pašu, bet tikai izmanto apgrieztu rastru, kas
atbilst teritorijai parquet failā.

``` r
mana_funkcija<-function(input_file){
  
  
  output_dir<-"data/VMD/tifs"
  input_basename <- file_path_sans_ext(basename(input_file)) # Noņem paplašinājumu
  output_file <- file.path(output_dir, paste0(input_basename, ".tif"))
  
 
  data <- st_read_parquet(input_file)
  data<-data%>%
    filter(zkat=="10"& s10=="1") #tikai priedes
  #print(head(data, 20))
  
  raster_10m_10km = raster("data/reference_raster/LV10m_10km.tif")
  raster_100m_10km = raster("data/reference_raster/LV100m_10km.tif")
  #plot(raster_100m_10km)

  raster_10m_10km_cropped = terra::crop(raster_10m_10km,data)
  raster_100m_10km_cropped = terra::crop(raster_100m_10km,data)
  
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

## Viens liels parquet

``` r
input_file <- "data/VMD/merged/validated/centra_parq.parquet"

time1 <- system.time(mana_funkcija(input_file))
print(time1)
```

    ##    user  system elapsed 
    ##  165.28   44.08  219.20

Ar brīviem 10.5 GB atmiņas nepietika (Error: std::bad_alloc). Kamēr
pietika (Timing stopped at: 546.5 268.9 878.1),izmantoja 1-2 kodolus.

Apgrieztajā variantā izmanto vienu kodolu un max 9.5 GB RAM, izpilde
bija ~5,5 min.

user system elapsed 276.62 36.21 320.39

## Daudzi parquet secīgi

``` r
# visu parquet uztaisīšana

if (!dir.exists("data/VMD/parquets")) {
  dir.create("data/VMD/parquets")
}


input_file_list <- list.files("data/VMD", pattern = "*shp$", full.names = TRUE)
print(input_file_list)

conv_to_parquet <- function(input_file) {
  
  file <- st_read(input_file)
  input_basename <- file_path_sans_ext(basename(input_file))
  output_file <- file.path("data/VMD/parquets",paste0(input_basename, ".parquet"))
  st_write_parquet(file,output_file)
}


for (file in input_file_list) {
  conv_to_parquet(file)
}
```

``` r
# katra parquet secīga apstrāde

list_parquets <- list.files("data/VMD/parquets", pattern = "\\.parquet$", full.names = TRUE)

dir_tifs <- "data/VMD/tifs"
dir.create(dir_tifs, recursive = TRUE)


time2<-system.time(
  for (prq in list_parquets) {
    mana_funkcija(prq)
  }
  )

print(time2)
```

    ##    user  system elapsed 
    ##  151.81   66.08  232.67

Pilnajā variantā ar brīviem 11GB atmiņas nepietiek. Beidzas ar Error:
std::bad_alloc

Ar apgrieztu teritoriju pietiek ar 10 GB RAM (izmanto praktiski gandrīz
visu pieejamo atmiņu, bet ar to pietiek), izmanto 1-2 kodolus. Izpilde
aizņem ~ 7,5 min.

user system elapsed 324.86 71.94 439.81

## Daudzi parquet paralēli ar vienu kodolu

``` r
cl1 <- makeCluster(1) 
registerDoParallel(cl1)

list_parquets <- list.files("data/VMD/parquets", pattern = "\\.parquet$", full.names = TRUE)

time3 <- system.time({
  foreach(prq = list_parquets,
          .packages = c("raster", "fasterize", "sf", "sfarrow", "tools", "dplyr")) %dopar% {
          mana_funkcija(prq)
  }
})

print(time3)
```

    ##    user  system elapsed 
    ##    0.03    0.02  239.47

``` r
stopCluster(cl1)
```

Ar pilno teritoriju daudz biežāk novērojama strauja atmiņas atbrīvošana.
Pieaugot tās patēriņam vispirms pieaug arī CPU izmantošana, bet viena
kodola ietvaros. Pēc tam krīt atpakaļ, bet atmiņas patēriņš saglabājas
augsts kādu laiku un pēc tam krīt. Bet tif fails tā arī pa to laiku
netiek uztaisīts. Atmiņas patēriņa grafiks izskatās pēc zāģa zobiem. Pēc
vairāk kā 30 min funkcija ir manuāli apturēta, jo reuzltāts nav
sasniegts.

Apgrieztajā - viens kodols, max 9.5 GB RAM un 8 min. Rodas iepaids par
vienmērīgāku atmiņas izmantošanu, nekā pilnajā versijā, nav tādi strauji
kāpumi kritumi. Intuitīvi šķiet, ka atbilda iterācijām for ciklā.
Atmiņas patēriņš izskatās drīzāk pēc sinusoīdas.

user system elapsed 0.60 0.55 479.03

## Daudzi parquet paralēli ar vairākiem kodoliem (3)

``` r
num_cores <- nb_cores() - 1  
cl2 <- makeCluster(num_cores)
registerDoParallel(cl2)
list_parquets <- list.files("data/VMD/parquets", pattern = "\\.parquet$", full.names = TRUE)

time4 <- system.time({
  foreach(prq = list_parquets,
          .packages = c("raster", "fasterize", "sf", "sfarrow", "tools", "dplyr")) %dopar% {
          mana_funkcija(prq)
  }
})

print(time4)
```

    ##    user  system elapsed 
    ##    0.01    0.02  150.36

``` r
stopCluster(cl2)
```

Apgrieztajā pamatā izmanto 2 kodolus. Izmantoja maksimāli pieejamo
atmiņu 10 GB RAM, bet ar to pietika. Apstrāde notika visātrāk ~ 4,5 min.
user system elapsed 0.55 0.48 260.39

Pilnajā versijā pēc 4,5 min pietrūka atmiņa.
