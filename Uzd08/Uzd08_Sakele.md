Uzd08_Sakele
================
Vita
2025-07-31

``` r
if (!require("raster")) install.packages("raster"); library (raster)
```

Uzdevums ir veikts pārlūkā. Uzrakstitais kods ir
[pieejams](https://code.earthengine.google.com/?accept_repo=users/vita/uzd08).

Kodu nosaukumi atspoguļo uzdevumus. \_print attēlo attiecīgos slāņus.
\_download lejuplādē uz Google Drive.

Šajā uzdevumā ir apzinātas paviršības, kas ieviestas, lai taupītu laiku.
Sākot ar uzdevumu 1.3 mākoņu un to ēnu maskai ir izmantoti tikai 20
attēli (limit 20). Paņemtot vienkārši mazāku teritoriju, droši vien būtu
bijis labāks risinājums.

Šī lēmuma dēļ nav iespējams salīdzināt uzdevumu 1.1., 1.2 rezultātu ar
1.3. Bet neskatoties uz to ir redzams, ka 1.1. uzdevumā ir iekļauti visi
mākoņi, gan jau kaut kādiem mākoņainības pētījumiem varētu arī noderēt.
1.2 ir noņemti neņemti mākoņi, izņemot šaurā joslā starp Daugavpili un
Preiļiem, no Kokneses uz Lietuvas robežu, kā arī daži citi atsevišķi
mākoņi.  
Savukārt 1.3. ir redzams, ka apdzīvotās vietas, apbūves attēlo baltu, jo
to atstarošanās ir augsta. Vēl balts parādās attēlu “salaiduma” vietās.

2.  uzdevumā ar katrā variantā ir iegūti nedaudz atšķirīgi indeksi.
    Vizuāli identificējamās atšķirības ir pikseļos uz valsts robežas.
    Toties 2.1. izpildijās gandrīz 2x ātrāk, nekā citi.

Salīdzinot, slāņu minimālās un maksimālās vērtības, redzams, ka min
sakrīt 2.2. un 2.3, bet max visiem vienādas.

# 3. uzdevums - GeoTIFF izveide

``` r
ndvi_raster<-raster('data/NDVI_2_3.tif')
plot(ndvi_raster)
```

![](Uzd08_Sakele_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

``` r
raster_100m_10km <- raster("../Uzd04/data/reference_raster/LV100m_10km.tif")


ndvi10_iqr <- aggregate(ndvi_raster, fact = 10, fun = IQR, na.rm = TRUE)

compareCRS(raster_100m_10km, ndvi10_iqr)
```

    ## [1] FALSE

``` r
ndvi_raster_proj <- projectRaster(ndvi10_iqr, raster_100m_10km)

ndvi_raster_proj_resampled <- resample(ndvi_raster_proj, raster_100m_10km, 
                                       method = "bilinear", filename='NDVI_iqr.tif', 
                                       overwrite=TRUE)
```

``` r
ndvi_raster_plot <- raster('NDVI_iqr.tif')
plot(ndvi_raster_plot)
```

![](Uzd08_Sakele_files/figure-gfm/unnamed-chunk-3-1.png)<!-- --> Nezinu
vai tāpēc, ka izmantoju limitētu daudzumu (tikai 20 attēlus) vai kāda
cita iemesla dēļ, ir diezgan labi redzamas kvadrātu robežas.
