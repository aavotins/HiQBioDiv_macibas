Uzd06_Sakele
================
Vita
2025-07-23

``` r
if (!require("sfarrow")) install.packages("sfarrow")
if (!require("sf")) install.packages("sf")
if (!require("terra")) install.packages("terra")
if (!require("dplyr")) install.packages("dplyr")
if (!require("raster")) install.packages("raster")
if (!require("tools")) install.packages("tools")
if (!require("fasterize")) install.packages("fasterize")
if (!require("httr")) install.packages("httr")
```

## 1. LAD slānis

``` r
#Slānis nav iepriekš saglabāts. Taisīšu vēlreiz

#Meža dati, lai no LAD datiem paņemtu tikai atbilstošo teritoriju
forest_data <- st_read_parquet("../Uzd04/data/VMD/merged/validated/centra_parq.parquet")

bbox_forest <- st_bbox(forest_data)
bbox_query <- paste(bbox_forest["xmin"],bbox_forest["ymin"], 
                    bbox_forest["xmax"], bbox_forest["ymax"], "EPSG:3059", sep = ",")


#LAD dati
wfs_lad<-"https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer"

url <- parse_url(wfs_lad)

url$query <- list(
  service = "WFS",
  request = "GetFeature",
  typename = "LAD:Lauki",
  #srsName = "EPSG:4326",
  srsName = "EPSG:3059", 
    bbox = bbox_query  
  )
request <- build_url(url)
LAD_data <- read_sf(request)


if (!dir.exists("data/LAD")) {
  dir.create("data/LAD",recursive = TRUE)
}

st_write_parquet(LAD_data, "data/LAD/LAD.parquet")
LAD_parq <- st_read_parquet("data/LAD/LAD.parquet", quiet = TRUE)
```

``` r
rm(LAD_data, bbox_forest, bbox_query, request, wfs_lad, url)
```

``` r
raster_10m_10km = raster("../Uzd04/data/reference_raster/LV10m_10km.tif")
  
raster_10m_10km_cropped <- crop(raster_10m_10km, LAD_parq)
plot(raster_10m_10km_cropped)
```

![](Uzd06_Sakele_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

``` r
LAD_raster_10m <- fasterize(LAD_parq, raster_10m_10km_cropped)
  
LAD_raster_10m<-rast(LAD_raster_10m)
plot(LAD_raster_10m, col = "brown")
```

![](Uzd06_Sakele_files/figure-gfm/unnamed-chunk-4-2.png)<!-- -->

``` r
  #100- lauki, NA - nav
LAD_raster_10m_100 <- subst(LAD_raster_10m,1,100) 
LAD_raster_10m_100 <- subst(LAD_raster_10m_100,0, NA)
plot(LAD_raster_10m_100, col = "brown")
```

![](Uzd06_Sakele_files/figure-gfm/unnamed-chunk-4-3.png)<!-- -->

``` r
writeRaster(LAD_raster_10m_100, "data/LAD/LAD_fields100.tif", overwrite = TRUE)
```

``` r
rm(LAD_raster_10m, raster_10m_10km)
```

## 2. Mežu klases

``` r
conifers_codes = c('1', '3', '13', '14', '15', '22', '23', '28', '29')
broadleaves_codes = c( '10', '11', '12', '16', '17', '18', '24', 
                       '61', '62', '63', '64', '65', '66', '67')
narrowleaves_codes = c('4', '6', '8', '9', '19', '20', '21', '25', '26',
                       '27', '32', '35', '50','68', '69')

# vispār te vajadzētu ciklu vai funkciju
forest_data <- forest_data%>%
  filter(zkat=='10') %>% #tikai mežaudzes
  mutate(
    total_basal_area=g10+g11+g12+g13+g14,
    conifers_basal_area = 
      (s10 %in% conifers_codes * g10) + 
      (s11 %in% conifers_codes * g11) + 
      (s12 %in% conifers_codes * g12) + 
      (s13 %in% conifers_codes * g13) + 
      (s14 %in% conifers_codes * g14),
    broadleaves_basal_area = 
      (s10 %in% broadleaves_codes * g10) + 
      (s11 %in% broadleaves_codes * g11) + 
      (s12 %in% broadleaves_codes * g12) + 
      (s13 %in% broadleaves_codes * g13) + 
      (s14 %in% broadleaves_codes * g14),
    narrowleaves_basal_area = 
      (s10 %in% narrowleaves_codes * g10) + 
      (s11 %in% narrowleaves_codes * g11) + 
      (s12 %in% narrowleaves_codes * g12) + 
      (s13 %in% narrowleaves_codes * g13) + 
      (s14 %in% narrowleaves_codes * g14),
    forest_type=
      case_when(
      narrowleaves_basal_area / total_basal_area >= 0.75 ~ 203, #"Šaurlapju mežs"
      broadleaves_basal_area / total_basal_area >= 0.75 ~ 202, #"Platlapju mežs"
      conifers_basal_area / total_basal_area >= 0.75 ~ 204, #"Skujkoku mežs"
      TRUE ~ 201)#"Jauktu koku mežs"
  )
```

``` r
rm(broadleaves_codes, conifers_codes, narrowleaves_codes)
```

``` r
#katram mežaudzes tipam savs rastrs

forest_types <- unique(forest_data$forest_type)

if (!dir.exists("data/forest_types")) {
  dir.create("data/forest_types",recursive = TRUE)
}

for(ft in forest_types){
  forest_data_filtered <- forest_data %>%  
    filter(forest_type == ft)

  forest_raster<-fasterize(forest_data_filtered, raster_10m_10km_cropped, field = "forest_type") 
  
  filename=paste0("data/forest_types/forest_", ft, ".tif")
  writeRaster(forest_raster, filename, overwrite = TRUE)
}
```

``` r
rm(forest_data_filtered, forest_raster, filename, forest_types, ft)
```

## 3. Mežaudzes vecuma grupa

Jāizburas cauri dokumentiem, lai saprastu, kas īsti prasīts un kā to
dabūt…

Vecumklase

-Skujkoki un cietie lapkoki – 20 gadi (skujkoki, goba, ozols, osis)
-Mīkstie lapkoki – 10 gadi (visi pārējie, kas nav cietie un ātraudzīgie)
-Ātraudzīgie – 5 gadi (baltalksnis, blīgzna, vītols)

Galvenās cirtes vecums no likumi.lv + mans papildinājums:

- 101: Ozols, priede, lapegle + līdzīgās sugas
- 81: Egle, osis, liepa, goba, vīksna un kļava + līdzīgās sugas un visas
  citas
- 71: Bērzs, melnalksnis
- 41: Apse + ātraudzīgie + papele + hibrīdā apse

Vecumgrupas:

1.  jaunaudze – pirmās divas vecumklases (mazāks par 2 x vecumklase)
2.  vidēja vecuma audze – starp jaunaudzi un briestaudzi (2 x vecumklase
    līdz ciršanas vecums – 1 vecumklase)
3.  briestaudze – viena vecumklase pirms pieaugušas audzes (ciršanas)
    (ciršanas vecum – 1 vecumklase līdz ciršanas vecums)
4.  pieauguša audze – divas vecumklases, sākot ar ciršanas vecumu
    (ciršanas vecums līdz ciršanas vecums + 2 x vecumklase)
5.  pārauguša audze – pārsniedz pieaugušas audzes vecumu (lielāks par
    ciršanas vecums +2 vecumklases)

``` r
#sugas pa vecumklasēm
age_class_20 = c('1', '3', '13', '14', '15', '22', '23', '28', '29', 
                '10', '11', '16', '61', '64', '65')
age_class_10 = c('4', '6', '8', '12', '17','18','19','24', '25',
                '26', '27', '32', '35', '50', '62', '63', '66', '67', '68','69')
age_class_5 = c('9', '20', '21')

#sugas pa galvenās ciršanas vecumiem
cutting_class_101 = c('1', '10', '13', '14', '22', '61')
cutting_class_81 = c('3', '11', '12', '15', '16', '17', '18', '23', '24', 
                     '25', '26', '27', '28', '29', '32', '35', '50', '62', 
                     '63', '64', '65', '66', '67', '69')
cutting_class_71 = c('4', '6')
cutting_class_41 = c('8', '9', '19', '20', '21', '68')


#pieškir galvenajai sugai vecumklasi
forest_data <- forest_data %>%
  mutate(
    age_class = case_when(
      s10 %in% age_class_20 ~ 20,
      s10 %in% age_class_10 ~ 10,
      s10 %in% age_class_5  ~ 5,
      TRUE ~ NA_real_ 
    ))

#pieškir galvenajai sugai galvenās cirtes vecumu
forest_data <- forest_data %>%
  mutate(
    cutting_age = case_when(
      s10 %in% cutting_class_101 ~ 101,
      s10 %in% cutting_class_81 ~ 81,
      s10 %in% cutting_class_71 ~ 71,
      s10 %in% cutting_class_41 ~ 41,
      TRUE ~ NA_real_
    )
  )

#piešķirt vecumgrupu galvenajai sugai
forest_data <- forest_data %>%
  mutate(
    age_group = case_when(
      a10 <= 2 * age_class ~ 1,  
      a10 > 2 * age_class & a10 < (cutting_age - age_class) ~ 2,
      a10 >= (cutting_age - age_class) & a10 < cutting_age ~ 3,
      a10 >= cutting_age & a10 < (cutting_age + 2 * age_class) ~ 4,
      a10 >= (cutting_age + 2 * age_class) ~ 5,
      TRUE ~ NA_real_ 
    ))
```

``` r
rm(age_class_5, age_class_10, age_class_20, cutting_class_101, cutting_class_81, cutting_class_71,
   cutting_class_41)
```

Vecumgrupu vajag ievietot jau esošajā meža tipu klasifikatorā kā otro
ciparu. Tad, to pareizinot ar 10, iegūst, ka tas būs pirmais cipars
divciparu skaitlī. Turklāt abi jau glabājas kā skaitliskas vērtības.

``` r
forest_data$class <- forest_data$forest_type + forest_data$age_group *10
```

``` r
if (!dir.exists("data/forest_classes")) {
  dir.create("data/forest_classes",recursive = TRUE)
}
forest_class <- unique(forest_data$class)

for(fc in forest_class){
  forest_data_filtered <- forest_data %>%  
    filter(class == fc)

  forest_raster<-fasterize(forest_data_filtered, raster_10m_10km_cropped, field = "class")

  filename=paste0("data/forest_classes/forest_", fc, ".tif")
  writeRaster(forest_raster, filename, overwrite = TRUE)
}
```

Sanāca arī viens tif ar klasi NA. To kādreiz vajadzētu izpētīt, kāpēc
tāda radās. Mežaudzes tipu līmenī visas klases bija normālas.

``` r
rm(forest_data, forest_class, forest_data_filtered, fc, filename, forest_raster)
```

## 4. Rastru apvienošana, prioritizējot tos

``` r
#visi mežu slāņi

tif_dir <- "data/forest_classes"
tif_files <- list.files(tif_dir, pattern = "\\.tif$", full.names = TRUE)
tif_files <- sort(tif_files)  # augošā secībā
rasters_list <- lapply(tif_files, rast)
#merged_forest_rasters <- do.call(cover, rasters_list) #nestrādā, jo nav S4
merged_forest_rasters <- Reduce(cover, rasters_list)
plot(merged_forest_rasters)
```

![](Uzd06_Sakele_files/figure-gfm/unnamed-chunk-15-1.png)<!-- -->

``` r
writeRaster(merged_forest_rasters, "data/forest_classes/forest_classes_merged.tif", 
            overwrite = TRUE)
```

``` r
rm(tif_dir, tif_files, rasters_list)
```

``` r
#Izrādījās, ka projekcijas nesakrita
crs(LAD_raster_10m_100)==crs(merged_forest_rasters)

#izlīdzinu
LAD_raster_10m_100_proj <- project(LAD_raster_10m_100, merged_forest_rasters)
LAD_raster_10m_100_resampled <- resample(LAD_raster_10m_100_proj, merged_forest_rasters, 
                                         method = "near")

#izgriežu, lai atbilst mežniecībai
LAD_raster_10m_100_cropped <- crop(LAD_raster_10m_100_resampled, merged_forest_rasters)
plot(LAD_raster_10m_100_cropped, col = "brown") 
```

![](Uzd06_Sakele_files/figure-gfm/unnamed-chunk-17-1.png)<!-- -->

``` r
#Nav jau īsti precīzi, jo tiek izmantots taisnstūris un ir dati, kas tomēr reāli nepieder mežniecībai, piemēram, kreisais augšējais stūris.

#LAD savienošana ar mežiem
LAD_plus_forests<-cover(LAD_raster_10m_100_cropped, merged_forest_rasters)

plot(LAD_plus_forests, main="Meži un lauki")
```

![](Uzd06_Sakele_files/figure-gfm/unnamed-chunk-17-2.png)<!-- -->

``` r
rm(LAD_raster_10m_100, LAD_raster_10m_100_resampled, LAD_raster_10m_100_cropped)
```

## 5. Šūnas vienlaicīgi ar mežaudžu un lauku informāciju

``` r
overlapping <- ifel(!is.na(LAD_raster_10m_100_proj) & !is.na(merged_forest_rasters), 1, NA)
sum_over = sum(values(overlapping) == 1, na.rm = TRUE)
cat("Šūnas ar pārklājošos informāciju:", sum_over)
```

## 6. Sauszemes šūnu skaits bez informācijas

Jāatrod, cik šūnām references rastrā nav datu

``` r
raster_10m_10km_cropped <- rast(raster_10m_10km_cropped)
#plot(raster_10m_10km_cropped)

LV <- cover(LAD_plus_forests, raster_10m_10km_cropped)
#plot(LV)

# Saskaita, cik ir NA vērtības
sum_1 <- global(LV==1, "sum", na.rm = TRUE)
cat("Sauszemes šūnas bez datiem:", sum_1[[1]])
```

Tā gan nebūs īsti tikai sauszemes teritorija, bet drīzāk gan iekšzemes
teritorija, jo ezeri, upes ir ieskaitītas šeit kā sauszeme.

``` r
rm(list = ls())
```
