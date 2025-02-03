Uzd2_Sakele
================
Vita
2024-12-15

\#Lejupielāde

``` r
if (!require("RCurl")) install.packages("RCurl");library(RCurl)
if (!require("archive")) install.packages("archive");library(archive)

if (!dir.exists("download")) {
  dir.create("download")
}


url= "https://data.gov.lv/dati/lv/dataset/40014c0a-90f5-42be-afb2-fe3c4b8adf92/resource/392dfb67-eeeb-43c2-b082-35f9cf986128/download/centra.7z"

destfile = "Download/centra.7z"

download.file(url=url, destfile = destfile, mode='wb')



if (!dir.exists("data")) {
  dir.create("data")
}
archive_extract(destfile, dir = "data")
```

\#Konvertācija un izmēri

``` r
if (!require("sf")) install.packages("sf")
library(sf)


if (!dir.exists("converted_data")) {
  dir.create("converted_data")
}

# Path to your shapefile
shapefile_path <- "data/nodala2651.shp" 

# Read the shapefile into an sf object
shp <- st_read(shapefile_path)
```

    ## Reading layer `nodala2651' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\data\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM

``` r
# Convert to GeoPackage
st_write(shp, "converted_data/output.gpkg", driver = "GPKG")
```

    ## Writing layer `output' to data source `converted_data/output.gpkg' using driver `GPKG'
    ## Writing 91434 features with 69 fields and geometry type Multi Polygon.

``` r
# Convert to GeoParquet
if (!require("sfarrow")) install.packages("sfarrow")
library(sfarrow)
st_write_parquet(shp, "converted_data/output.parquet")
```

``` r
file.size("converted_data/output.parquet") # 21 795 679
```

    ## [1] 21795679

``` r
file.size("converted_data/output.gpkg") # 57 188 352
```

    ## [1] 57188352

``` r
file.size("data/nodala2651.shp") #27 090 844 + pārējie ar slāni saistītie faili
```

    ## [1] 27090844

``` r
rm(shp)
```

\#Ielasīšanas laika mērījumi

``` r
if (!require("microbenchmark")) install.packages("microbenchmark")
library(microbenchmark)
```

``` r
ielades_atrums <- microbenchmark(
  read_shape = st_read("data/nodala2651.shp"), # vidēji 9.6 sek
  read_gpkg = st_read("converted_data/output.gpkg"), # vidēji 6.5 sek
  read_parquet = st_read_parquet("converted_data/output.parquet"), #vidēji 2.5 sek
  times = 10,
  unit="s"
)
```

    ## Reading layer `nodala2651' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\data\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `output' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\converted_data\output.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `output' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\converted_data\output.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\data\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `output' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\converted_data\output.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\data\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `output' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\converted_data\output.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\data\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `output' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\converted_data\output.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\data\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\data\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\data\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `output' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\converted_data\output.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `output' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\converted_data\output.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\data\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\data\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\data\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `output' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\converted_data\output.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `output' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\converted_data\output.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `output' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\converted_data\output.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 91434 features and 69 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM

``` r
print(ielades_atrums)
```

    ## Unit: seconds
    ##          expr       min        lq     mean   median       uq      max neval cld
    ##    read_shape 5.2155069 5.2698870 6.027450 5.395213 7.129023 7.720759    10 a  
    ##     read_gpkg 3.5313904 3.6139545 4.088534 3.854840 4.500886 5.374719    10  b 
    ##  read_parquet 0.8010399 0.9397704 1.049045 1.000856 1.191482 1.377800    10   c

\#Failu apvienošana

``` r
if (!dir.exists("data/combined")) {
  dir.create("data/combined")
}

file_list <- list.files("data", pattern = "*shp$", full.names = TRUE)
shapefile_list <- lapply(file_list, read_sf) #ielasa visus failus
combined_shapefile <- do.call(rbind, shapefile_list) # apvieno vienā failā
st_write(combined_shapefile, "data/combined/centra.gpkg", driver = "GPKG")
```

    ## Writing layer `centra' to data source `data/combined/centra.gpkg' using driver `GPKG'
    ## Writing 506828 features with 69 fields and geometry type Multi Polygon.

``` r
rm(combined_shapefile, shapefile_list)
```

\#Ģeometriju pārbaude

``` r
centra_gpkg <- st_read("data/combined/centra.gpkg")
```

    ## Reading layer `centra' from data source 
    ##   `C:\Working\VPP\Mācības\HiQBioDiv_macibas\Uzd02\data\combined\centra.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 506828 features and 69 fields (with 6 geometries empty)
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 405229.7 ymin: 234180.7 xmax: 594509.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM

``` r
# Pārbaudīt, vai visas ģeometrijas ir MULTIPOLYGON
if(all(st_geometry_type(centra_gpkg) == "MULTIPOLYGON")) {
  print("Visas ģeometrijas ir MULTIPOLYGON")
} else {
  print("Dažas ģeometrijas nav MULTIPOLYGON")
  }
```

    ## [1] "Visas ģeometrijas ir MULTIPOLYGON"

``` r
# Pārbaudīt, vai nav tukšu ģeometriju
if(any(st_is_empty(centra_gpkg))) {
  print("Slānī ir tukšas ģeometrijas")
  # Izdzēst tukšās ģeometrijas
  centra_gpkg <- centra_gpkg[!st_is_empty(centra_gpkg), ] 
}
```

    ## [1] "Slānī ir tukšas ģeometrijas"

``` r
# Pārbaudīt, vai nav kļūdainu ģeometriju
if(any(!st_is_valid(centra_gpkg))) {
  print("Slānī ir kļūdainas ģeometrijas")
 # Salabot ķļūdainās ģeomtrijas
  centra_gpkg <- st_make_valid(centra_gpkg)
  # Izdzēst kļūdainās ģeometrijas
  centra_gpkg <- centra_gpkg[st_is_valid(centra_gpkg), ] 
}
```

    ## [1] "Slānī ir kļūdainas ģeometrijas"

\#Priežu meži

``` r
# Ģeometrijas vairs nav vajadzīgas, var izmanto arī parquet failu
centra_df<-data.frame(centra_gpkg)
rm(centra_gpkg)
```

``` r
if (!require("dplyr")) install.packages("dplyr")
library(dplyr)

pine_codes = c(1, 14, 22)
centra_df <- centra_df %>%
 mutate(
   pine_basal_area = (
      ifelse(s10 %in% pine_codes, g10, 0) +
        ifelse(s11 %in% pine_codes, g11, 0) +
        ifelse(s12 %in% pine_codes, g12, 0) +
        ifelse(s13 %in% pine_codes, g13, 0) +
        ifelse(s14 %in% pine_codes, g14, 0)),
     
  total_basal_area = g10+g11+g12+g13+g14,
  prop_priede = pine_basal_area/total_basal_area,
  PriezuMezi = ifelse(is.nan(prop_priede), 0, 
               ifelse(prop_priede >= 0.75, 1, 0)) 
  )

centra_df$pine_basal_area<-NULL

#Priežu meži no visiem mežiem mežaudze (zkat=="10")
sum(centra_df$PriezuMezi)/nrow(centra_df[centra_df$zkat == "10", ])
```

    ## [1] 0.1952374

\#Citi meži

Ja sugas, kas pieder konkrētam meža tipam, veido vismaz 75%
šķērslaukuma, tad tiek izskatīts, ka mežs ir attiecigā tipa. Pretējā
gadījumā tas ir jauktu koku mežs, jo neviena sugu grupa nedominē.

``` r
#Sugu piederība
conifers_codes = c(1, 3, 13, 14, 15, 22, 23, 28, 29)
broadleaves_codes = c( 10, 11, 12, 16, 17, 18, 19, 20, 21, 24, 25, 26, 27, 32, 35, 50, 61, 62, 63, 64, 65, 66, 67, 69)
narrowleaves_codes = c(4, 6, 8, 9, 68)

# vispār te vajadzētu funkciju
centra_df <- centra_df%>%
  filter(zkat=='10') %>% #tikai mežaudzes
  mutate(
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
      narrowleaves_basal_area / total_basal_area >= 0.75 ~ "Šaurlapju mežs",
      broadleaves_basal_area / total_basal_area >= 0.75 ~ "Platlapju mežs",
      conifers_basal_area / total_basal_area >= 0.75 ~ "Skujkoku mežs",
      TRUE ~ "Jauktu koku mežs")
  )
```

``` r
nrows=nrow(centra_df)
forest_ratios<-centra_df %>%
  group_by(forest_type) %>%
  summarize(ratio = n() / nrows) 

forest_ratios
```

    ## # A tibble: 4 × 2
    ##   forest_type       ratio
    ##   <chr>             <dbl>
    ## 1 Jauktu koku mežs 0.363 
    ## 2 Platlapju mežs   0.0104
    ## 3 Skujkoku mežs    0.320 
    ## 4 Šaurlapju mežs   0.306

\#Visu mežaudžu failu dzēšana

``` r
unlink("data", recursive = TRUE)
unlink("converted_data", recursive = TRUE)
unlink("download", recursive = TRUE)

rm(centra_df)
```
