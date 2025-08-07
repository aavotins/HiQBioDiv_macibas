Astotais uzdevums: GEE
================
Jekaterīna Butkeviča
2025. gads 06. augusts

Pirmā uzdevuma daļa (1.–2.3. uzdevumi) ir paredzēta izpildei Google
Earth Engine pārlūkā, izmantojot JavaScript valodu. Komandrindas ir
pieejamas šeit:
<https://code.earthengine.google.com/04b5c4d2ae6258c27dd4e776fd6af07d>.
Tālāk tiek parādīts kods atlikušajai sadaļai, kas pildīta R valodā.

## Trešais uzdevums

Lejupielādējiet 2.3. rezultātu, no tā izveidojiet vienu GeoTIFF slāni,
kura aptvertā telpa un pikseļu izvietojums atbilst projekta Zenodo
repozitorijā dotajam rastra slānim ar 10m izšķirtspēju. Ar starpkvartiļu
amplitūdu raksturojiet NDVI variabilitāti ik 100 m šūnā kā rastra slāni,
kura telpisais pārklājums un pikseļu izvietojums atbilst projekta Zenodo
repozitorijā dotajam rastra slānim ar 100m izšķirtspēju.

No sākuma lejupielādējam nepieciešamās pakotnes.

``` r
if (!require("here")) install.packages("here"); library(here)
if (!require("terra")) install.packages("terra"); library(terra)
if (!require("sp")) install.packages("sp"); library(sp)
```

Uzstādām globālos iestatījumus terra pakotnei, lai apstrādātu lielus
failus.

``` r
terraOptions(memfrac = 0.8)
```

Atsevišķos mainīgajos saglabājam ceļus uz iegūtajiem kartes fragmentiem
(manā gadījumā no GEE beigām saņēmu rezultātu sadalītu divos failos), kā
arī uz Zenodo references rastriem. Definēju arī izvades ceļus un
izveidoju nepieciešamās apakšdirektorijas.

``` r
# Ceļi uz 10 m NDVI slāni
# Ir divi GeoTIFF fragmenti, kas eksportēti no GEE.
ndvi_fragments_1_cels <- here("8uzd", "Data", "GEE", "ndvi_median_annual_medians_10m-0000000000-0000000000.tif")
ndvi_fragments_2_cels <- here("8uzd", "Data", "GEE", "ndvi_median_annual_medians_10m-0000000000-0000032768.tif")
ndvi_files <- c(ndvi_fragments_1_cels, ndvi_fragments_2_cels)


# References rastru faili no Zenodo.
ref_10m_cels <- here("3uzd", "Data", "references_zenodo", "rastrs", "LV10m_10km.tif")
ref_100m_cels <- here("3uzd", "Data", "references_zenodo", "rastrs", "LV100m_10km.tif")


# Izvades failu ceļi
output_ndvi_cels <- here("8uzd", "Output")
dir.create(output_ndvi_cels, recursive = TRUE)
output_iqr_cels <- here("8uzd", "Output", "final_iqr_100m_izlidz.tif")
```

Ielāsām abus GEE rastra fragmentus un references rastrus.

``` r
#Ielāsām
ndvi_fragments_1 <- rast(ndvi_fragments_1_cels)
ndvi_fragments_2 <- rast(ndvi_fragments_2_cels)
ref_10m <- rast(ref_10m_cels)
ref_100m <- rast(ref_100m_cels)
```

Funkcija `merge()` apvieno vairākus rastrus. Tā kā šie rastri
nepārklājas, tiks izveidots viens rastrs, kas aptver abu fragmentu
kopējo pārklājumu.

``` r
#Apvienojām
merged_ndvi <- merge(ndvi_fragments_1, ndvi_fragments_2)
```

    ## |---------|---------|---------|---------|=========================================                                          

Uzreiz pārbaudām, vai rastrs nav tukšs.

``` r
plot(merged_ndvi)
```

![](Uzd08_Butkevica_files/figure-gfm/plot_orig-1.png)<!-- -->

Pārprojektējam NDVI mozaīku no tās sākotnējās koordinātu sistēmas uz
references rastra koordinātu sistēmu.

``` r
projected_ndvi <- project(merged_ndvi, ref_10m, method = "near")
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

Izmantoju `terra::resample`, lai saskaņotu apvienoto NDVI rastru ar
references rastru. Izmantoju metodi `near`, lai izvairītos no vērtību
interpolācijas. Jo tā šķiiet vislabāk piemērota uzdevuma mērķim, t.i.,
saskaņot NDVI rastru ar references slāni, nezaudējot vērtību
precizitāti. Šī metode nemaina datus, bet pielāgo pikseļu novietojumu,
saglabājot GEE eksportēto vērtību fizisko jegu. Savukārt metode
`average` interpolē jaunas vērtības un var “izpludināt” datus. Jo šoreiz
nav mērķa radīt jaunu informācīju, bet nodrošināt saskaņojumu ar
references slāni. Uzreiz saglabājam rezultātu.

``` r
final_ndvi_10m <- resample(projected_ndvi, ref_10m, method = "near")
```

    ## |---------|---------|---------|---------|=========================================                                          

Uzreiz saglabājam rezultātu.

``` r
writeRaster(final_ndvi_10m, filename = paste0(output_ndvi_cels, "/final_ndvi_10m_izlidz.tif"), overwrite = TRUE)
```

Attēlojam rezultātu vizuālai pārbaudei.

``` r
plot(final_ndvi_10m, main = "Finala 10 m NDVI slānis (izlīdzināts)")
```

![](Uzd08_Butkevica_files/figure-gfm/plot_NDVI-1.png)<!-- -->

Aprēķinām NDVI IQR 100 m izšķirtspējā. Izmantojam `aggregate`, lai
samazinātu 10 m rastra izšķirtspēju līdz 100 m. Arguments `fact = 10`
nozīmē, ka 10x10 (100) 10 m šūnas tiek apvienotas vienā 100 m šūnā,
savukārt arguments `fun = IQR` norāda, ka katrai jaunajai šūnai tiek
aprēķināts interkvartila diapazons jeb IQR.

``` r
ndvi_iqr_100m_raw <- aggregate(final_ndvi_10m, fact = 10, fun = IQR, na.rm = TRUE)
```

Tagad saskaņojam šo jauno 100 m IQR rastru ar oficiālo 100 m references
rastru. Un saglabājam rezultātu.

``` r
final_iqr_100m <- resample(ndvi_iqr_100m_raw, ref_100m, method = "near")
writeRaster(final_iqr_100m, filename = output_iqr_cels, overwrite = TRUE)
```

Attēlojam rezultātu vizuālai pārbaudei.

``` r
plot(final_iqr_100m, main = "Finals 100m NDVI IQR slānis (izlīdzināts)")
```

![](Uzd08_Butkevica_files/figure-gfm/plot_IQR-1.png)<!-- -->
