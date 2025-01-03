02uzd_Zuperka
================

## <br><b> 1) Datu lejupielāde, atarhivēšana un nevajadzīgo dzēšana </b>

    ## Loading required package: httr

    ## Loading required package: archive

    ## Lejuplādējam .7z failu...
    ## Lejuplāde pabeigta!  Centra.7z 
    ## Atarhivējam...

    ## ⠙ 13 extracted | 1.5 GB (697 MB/s) | 2.1s

    ## ⠹ 13 extracted | 1.5 GB (635 MB/s) | 2.4s⠸ 13 extracted | 1.5 GB (587 MB/s) |
    ## 2.6s⠼ 17 extracted | 1.5 GB (547 MB/s) | 2.8s⠴ 17 extracted | 1.7 GB (553 MB/s)
    ## | 3s ⠦ 17 extracted | 1.8 GB (560 MB/s) | 3.2s⠧ 17 extracted | 1.9 GB (567
    ## MB/s) | 3.4s⠇ 17 extracted | 2.1 GB (576 MB/s) | 3.6s⠏ 17 extracted | 2.2 GB
    ## (580 MB/s) | 3.8s⠋ 17 extracted | 2.4 GB (584 MB/s) | 4s ⠙ 19 extracted | 2.5
    ## GB (585 MB/s) | 4.2s⠹ 21 extracted | 2.5 GB (559 MB/s) | 4.5s⠸ 21 extracted |
    ## 2.5 GB (536 MB/s) | 4.7s⠼ 21 extracted | 2.5 GB (516 MB/s) | 4.9s⠴ 25 extracted
    ## | 2.6 GB (503 MB/s) | 5.1s⠦ 25 extracted | 2.7 GB (506 MB/s) | 5.3s⠧ 25
    ## extracted | 2.8 GB (507 MB/s) | 5.5s⠇ 25 extracted | 2.9 GB (516 MB/s) | 5.7s⠏
    ## 25 extracted | 3.1 GB (519 MB/s) | 5.9s⠋ 25 extracted | 3.2 GB (525 MB/s) |
    ## 6.1s⠙ 25 extracted | 3.3 GB (528 MB/s) | 6.3s⠹ 25 extracted | 3.5 GB (532 MB/s)
    ## | 6.5s⠸ 29 extracted | 3.5 GB (520 MB/s) | 6.7s⠼ 29 extracted | 3.5 GB (506
    ## MB/s) | 7s ⠴ 29 extracted | 3.5 GB (493 MB/s) | 7.2s⠦ 33 extracted | 3.6 GB
    ## (482 MB/s) | 7.4s⠧ 33 extracted | 3.7 GB (487 MB/s) | 7.6s⠇ 33 extracted | 3.8
    ## GB (491 MB/s) | 7.8s⠏ 33 extracted | 4.0 GB (497 MB/s) | 8s ⠋ 33 extracted |
    ## 4.1 GB (503 MB/s) | 8.2s⠙ 33 extracted | 4.3 GB (509 MB/s) | 8.4s⠹ 33 extracted
    ## | 4.4 GB (513 MB/s) | 8.6s⠸ 37 extracted | 4.5 GB (508 MB/s) | 8.8s⠼ 37
    ## extracted | 4.5 GB (497 MB/s) | 9s ⠴ 37 extracted | 4.5 GB (488 MB/s) | 9.2s

    ## Atarhivēšana pabeigta. Dati saglabāti:  C:/Users/mark7/Documents/MZ_HiQBioDiv_macibas/Uzd02 
    ## Arhīvs izdzēsts pēc lejupielādes un atarhivēšanas:  Centra.7z

\###<br><b> 2) Konvertācija uz geopackage, geoparquet un ESRI File
Geodatabase </b> \##<br> 2.1) Ielasu tikai shapefile

    ## Loading required package: sf

    ## Linking to GEOS 3.12.2, GDAL 3.9.3, PROJ 9.4.1; sf_use_s2() is TRUE

    ## Loading required package: arrow

    ## 
    ## Attaching package: 'arrow'

    ## The following object is masked from 'package:utils':
    ## 
    ##     timestamp

    ## Izvēlēts lielākais fails: nodala2653.shp 
    ## Reading layer `nodala2653' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala2653.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 112400 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 496133.2 ymin: 299013 xmax: 570134.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM
    ## SHP ielasīts.

\##<br> 2.2) Konvertēju uz geoparquet

    ## Sekmīgi konvertēts uz GeoParquet: Nodala_2651.parquet

\##<br> 2.3) Konvertēju uz geopackage

    ## Writing layer `nodala_2651' to data source `nodala_2651.gpkg' using driver `GPKG'
    ## Writing 112400 features with 70 fields and geometry type Multi Polygon.
    ## GeoPackage fails ir izveidots!

\#<br><b> 3) Aizņemtās diska vietas un ielasīšanas ātrums 10
izmēģinājumos </b>

    ## Shapefile is 25.84 MB large, but with the required DBF file, it is 775.82 MB large.

    ## GeoPackage is 70.11 MB large.

    ## GeoParquet is 22.80 MB large.

    ## Reading layer `nodala_2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala_2651.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 112400 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 496133.2 ymin: 299013 xmax: 570134.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala_2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala_2651.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 112400 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 496133.2 ymin: 299013 xmax: 570134.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala_2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala_2651.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 112400 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 496133.2 ymin: 299013 xmax: 570134.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala_2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala_2651.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 112400 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 496133.2 ymin: 299013 xmax: 570134.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala_2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala_2651.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 112400 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 496133.2 ymin: 299013 xmax: 570134.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala_2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala_2651.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 112400 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 496133.2 ymin: 299013 xmax: 570134.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala_2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala_2651.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 112400 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 496133.2 ymin: 299013 xmax: 570134.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala_2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala_2651.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 112400 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 496133.2 ymin: 299013 xmax: 570134.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala_2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala_2651.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 112400 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 496133.2 ymin: 299013 xmax: 570134.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala_2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala_2651.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 112400 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 496133.2 ymin: 299013 xmax: 570134.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala_2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala_2651.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 112400 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 496133.2 ymin: 299013 xmax: 570134.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM

    ##    Try ShapefileTime GeoPackageTime GeoParquetTime
    ## 1    1  3.099501e-09   2.588487e-09   1.075981e-07
    ## 2    2  3.212227e-09   2.578692e-09   1.081112e-07
    ## 3    3  3.262981e-09   2.737122e-09   1.099743e-07
    ## 4    4  3.209022e-09   2.614453e-09   1.038564e-07
    ## 5    5  3.177969e-09   2.497464e-09   1.044821e-07
    ## 6    6  3.353951e-09   3.136860e-09   1.083135e-07
    ## 7    7  4.048134e-09   3.328990e-09   1.120283e-07
    ## 8    8  4.072484e-09   3.530648e-09   1.238906e-07
    ## 9    9  3.958127e-09   2.456586e-09   9.949310e-08
    ## 10  10  3.006361e-09   2.637365e-09   9.877350e-08

\#<br><b> 4) Apvienoju visus centra mežniecības datus</b>

    ## Reading layer `nodala2651' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala2651.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 91369 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 484571.2 ymin: 234180.7 xmax: 563754.7 ymax: 304084.5
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2652' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala2652.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 78469 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 420613.3 ymin: 241544.3 xmax: 507491.5 ymax: 305873.1
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2653' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala2653.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 112400 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 496133.2 ymin: 299013 xmax: 570134.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2654' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala2654.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 113968 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 505799.1 ymin: 266426.8 xmax: 594509.3 ymax: 318075
    ## Projected CRS: LKS-92 / Latvia TM
    ## Reading layer `nodala2655' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\nodala2655.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 109454 features and 70 fields (with 6 geometries empty)
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 405229.7 ymin: 283087.1 xmax: 504232.4 ymax: 352909
    ## Projected CRS: LKS-92 / Latvia TM
    ## Writing layer `Centrs_combined' to data source 
    ##   `C:/Users/mark7/Documents/MZ_HiQBioDiv_macibas/Uzd02/Centrs_combined.shp' using driver `ESRI Shapefile'
    ## Writing 505654 features with 70 fields and geometry type Multi Polygon.

    ## Warning in CPL_write_ogr(obj, dsn, layer, driver,
    ## as.character(dataset_options), : GDAL Message 1: One or several characters
    ## couldn't be converted correctly from UTF-8 to ISO-8859-1.  This warning will
    ## not be emitted anymore.

    ## Combined shapefile saved as: C:/Users/mark7/Documents/MZ_HiQBioDiv_macibas/Uzd02/Centrs_combined.shp

\###<br><b> 5) Priežu aprēķini</b>

    ## Reading layer `Centrs_combined' from data source 
    ##   `C:\Users\mark7\Documents\MZ_HiQBioDiv_macibas\Uzd02\Centrs_combined.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 505654 features and 70 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 405229.7 ymin: 234180.7 xmax: 594509.3 ymax: 363287.3
    ## Projected CRS: LKS-92 / Latvia TM

    ## Simple feature collection with 6 features and 71 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 497918.4 ymin: 254136.3 xmax: 545441.8 ymax: 300677.8
    ## Projected CRS: LKS-92 / Latvia TM
    ##         id objectid_1           adm1               adm2    atvk    kadastrs
    ## 1 51012798   51012798 Bauskas novads   B?rbeles pagasts 0025400 40440070007
    ## 2 51596262   51596262 Bauskas novads  Vecsaules pagasts 0025550 40920040202
    ## 3 51858796   51858796 Bauskas novads Vecumnieku pagasts 0025560 40940050048
    ## 4 51978889   51978889 Bauskas novads   Kurmenes pagasts 0025480 32620040019
    ## 5 66691890   66691890 Bauskas novads   B?rbeles pagasts 0025400 40440060043
    ## 6 67294609   67294609 Olaines novads    Olaines pagasts 0041400 80800030085
    ##    gtf kvart nog anog nog_plat expl_mezs expl_celi expl_gravj zkat mt izc s10
    ## 1 2006     1  14    0     1.81      1.40         0       0.41   10  5   1  16
    ## 2 2002     1   3    0     0.11      0.11         0       0.00   10  5   1   9
    ## 3 2012     1  12    0     0.08      0.00         0       0.00   32  0   0   0
    ## 4 2012     4  16    0     0.07      0.00         0       0.00   31  4   0   0
    ## 5 2013     1  11    0     0.15      0.14         0       0.01   10  4   1   8
    ## 6 2017   322  12    0    12.55     11.95         0       0.60   10 12   1   1
    ##   a10 h10 d10 g10  n10 bv10 ba10 s11 a11 h11 d11 g11 n11 bv11 ba11 s12 a12 h12
    ## 1  71  23  29  10    0    0    0   9  56  18  20   7   0    0    0   9  31  16
    ## 2  47  23  19  28    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 3   0   0   0   0    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 4   0   0   0   0    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 5   8   5   6   0 2000    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 6  43   9   7   0 1200    0    0   1  33   6   8   0 720    0    0   4  33   9
    ##   d12 g12 n12 bv12 ba12 s13 a13 h13 d13 g13 n13 bv13 ba13 s14 a14 h14 d14 g14
    ## 1  16   5   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 2   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 3   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 4   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 5   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 6  10   0 380    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ##   n14 bv14 ba14 jakopj jaatjauno p_darbv p_darbg p_cirp p_cirg atj_gads
    ## 1   0    0    0      0         0       0       0      0      0        0
    ## 2   0    0    0      0         0       0       0      0      0        0
    ## 3   0    0    0      0         0       0       0      0      0        0
    ## 4   0    0    0      0         0       0       0      0      0        0
    ## 5   0    0    0      0         0       4    2017     11   2016     2017
    ## 6   0    0    0      0         0       0       0      0      0        0
    ##   saimn_d_ie plant_audz forestry_c vmd_headfo shape_Leng  shape_Area
    ## 1          4          0       2651     Centra  1283.3672  18118.3409
    ## 2          6          0       2651     Centra   136.7248   1096.7627
    ## 3          6          0       2651     Centra   113.7443    757.9693
    ## 4          6          0       2651     Centra   109.7517    743.0716
    ## 5          6          0       2651     Centra   162.6895   1500.9411
    ## 6          6          0       2651     Centra  3350.0701 125452.6530
    ##                         geometry prop_priedes
    ## 1 MULTIPOLYGON (((536650.7 25...            0
    ## 2 MULTIPOLYGON (((528596.9 25...            0
    ## 3 MULTIPOLYGON (((527092.6 27...            0
    ## 4 MULTIPOLYGON (((545441.8 25...            0
    ## 5 MULTIPOLYGON (((544436.8 25...            0
    ## 6 MULTIPOLYGON (((497918.4 30...           76

    ## Simple feature collection with 6 features and 72 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 497918.4 ymin: 254136.3 xmax: 545441.8 ymax: 300677.8
    ## Projected CRS: LKS-92 / Latvia TM
    ##         id objectid_1           adm1               adm2    atvk    kadastrs
    ## 1 51012798   51012798 Bauskas novads   B?rbeles pagasts 0025400 40440070007
    ## 2 51596262   51596262 Bauskas novads  Vecsaules pagasts 0025550 40920040202
    ## 3 51858796   51858796 Bauskas novads Vecumnieku pagasts 0025560 40940050048
    ## 4 51978889   51978889 Bauskas novads   Kurmenes pagasts 0025480 32620040019
    ## 5 66691890   66691890 Bauskas novads   B?rbeles pagasts 0025400 40440060043
    ## 6 67294609   67294609 Olaines novads    Olaines pagasts 0041400 80800030085
    ##    gtf kvart nog anog nog_plat expl_mezs expl_celi expl_gravj zkat mt izc s10
    ## 1 2006     1  14    0     1.81      1.40         0       0.41   10  5   1  16
    ## 2 2002     1   3    0     0.11      0.11         0       0.00   10  5   1   9
    ## 3 2012     1  12    0     0.08      0.00         0       0.00   32  0   0   0
    ## 4 2012     4  16    0     0.07      0.00         0       0.00   31  4   0   0
    ## 5 2013     1  11    0     0.15      0.14         0       0.01   10  4   1   8
    ## 6 2017   322  12    0    12.55     11.95         0       0.60   10 12   1   1
    ##   a10 h10 d10 g10  n10 bv10 ba10 s11 a11 h11 d11 g11 n11 bv11 ba11 s12 a12 h12
    ## 1  71  23  29  10    0    0    0   9  56  18  20   7   0    0    0   9  31  16
    ## 2  47  23  19  28    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 3   0   0   0   0    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 4   0   0   0   0    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 5   8   5   6   0 2000    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 6  43   9   7   0 1200    0    0   1  33   6   8   0 720    0    0   4  33   9
    ##   d12 g12 n12 bv12 ba12 s13 a13 h13 d13 g13 n13 bv13 ba13 s14 a14 h14 d14 g14
    ## 1  16   5   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 2   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 3   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 4   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 5   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 6  10   0 380    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ##   n14 bv14 ba14 jakopj jaatjauno p_darbv p_darbg p_cirp p_cirg atj_gads
    ## 1   0    0    0      0         0       0       0      0      0        0
    ## 2   0    0    0      0         0       0       0      0      0        0
    ## 3   0    0    0      0         0       0       0      0      0        0
    ## 4   0    0    0      0         0       0       0      0      0        0
    ## 5   0    0    0      0         0       4    2017     11   2016     2017
    ## 6   0    0    0      0         0       0       0      0      0        0
    ##   saimn_d_ie plant_audz forestry_c vmd_headfo shape_Leng  shape_Area
    ## 1          4          0       2651     Centra  1283.3672  18118.3409
    ## 2          6          0       2651     Centra   136.7248   1096.7627
    ## 3          6          0       2651     Centra   113.7443    757.9693
    ## 4          6          0       2651     Centra   109.7517    743.0716
    ## 5          6          0       2651     Centra   162.6895   1500.9411
    ## 6          6          0       2651     Centra  3350.0701 125452.6530
    ##                         geometry prop_priedes PriezuMezi
    ## 1 MULTIPOLYGON (((536650.7 25...            0          0
    ## 2 MULTIPOLYGON (((528596.9 25...            0          0
    ## 3 MULTIPOLYGON (((527092.6 27...            0          0
    ## 4 MULTIPOLYGON (((545441.8 25...            0          0
    ## 5 MULTIPOLYGON (((544436.8 25...            0          0
    ## 6 MULTIPOLYGON (((497918.4 30...           76          1

    ## Centra virsmežniecībā ir 114533 priežu mežu poligoni, kuri veido 22.65 % no kopējā poligonu skaita ( 505654 ).

\###<br><b> 6) Klasifikācija </b> <br> Klasifikācijai pirmkārt norādu
kuri koki pieder kuram tipam un izveidoju jaunas kollonnas (k_s)
(kombinēju atsevišķus kokus koku grupās, balstoties uz interneta
resursiem un savām zināšanām)

    ## Warning in FUN(X[[i]], ...): NAs introduced by coercion

    ## Warning in `[<-.data.frame`(`*tmp*`, cols_to_convert, value = list(s10 = c(16,
    ## : provided 6 variables to replace 5 variables

    ## Simple feature collection with 6 features and 77 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 497918.4 ymin: 254136.3 xmax: 545441.8 ymax: 300677.8
    ## Projected CRS: LKS-92 / Latvia TM
    ##         id objectid_1           adm1               adm2    atvk    kadastrs
    ## 1 51012798   51012798 Bauskas novads   B?rbeles pagasts 0025400 40440070007
    ## 2 51596262   51596262 Bauskas novads  Vecsaules pagasts 0025550 40920040202
    ## 3 51858796   51858796 Bauskas novads Vecumnieku pagasts 0025560 40940050048
    ## 4 51978889   51978889 Bauskas novads   Kurmenes pagasts 0025480 32620040019
    ## 5 66691890   66691890 Bauskas novads   B?rbeles pagasts 0025400 40440060043
    ## 6 67294609   67294609 Olaines novads    Olaines pagasts 0041400 80800030085
    ##    gtf kvart nog anog nog_plat expl_mezs expl_celi expl_gravj zkat mt izc s10
    ## 1 2006     1  14    0     1.81      1.40         0       0.41   10  5   1  16
    ## 2 2002     1   3    0     0.11      0.11         0       0.00   10  5   1   9
    ## 3 2012     1  12    0     0.08      0.00         0       0.00   32  0   0   0
    ## 4 2012     4  16    0     0.07      0.00         0       0.00   31  4   0   0
    ## 5 2013     1  11    0     0.15      0.14         0       0.01   10  4   1   8
    ## 6 2017   322  12    0    12.55     11.95         0       0.60   10 12   1   1
    ##   a10 h10 d10 g10  n10 bv10 ba10 s11 a11 h11 d11 g11 n11 bv11 ba11 s12 a12 h12
    ## 1  71  23  29  10    0    0    0   9  56  18  20   7   0    0    0   9  31  16
    ## 2  47  23  19  28    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 3   0   0   0   0    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 4   0   0   0   0    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 5   8   5   6   0 2000    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 6  43   9   7   0 1200    0    0   1  33   6   8   0 720    0    0   4  33   9
    ##   d12 g12 n12 bv12 ba12 s13 a13 h13 d13 g13 n13 bv13 ba13 s14 a14 h14 d14 g14
    ## 1  16   5   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 2   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 3   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 4   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 5   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 6  10   0 380    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ##   n14 bv14 ba14 jakopj jaatjauno p_darbv p_darbg p_cirp p_cirg atj_gads
    ## 1   0    0    0      0         0       0       0      0      0        0
    ## 2   0    0    0      0         0       0       0      0      0        0
    ## 3   0    0    0      0         0       0       0      0      0        0
    ## 4   0    0    0      0         0       0       0      0      0        0
    ## 5   0    0    0      0         0       4    2017     11   2016     2017
    ## 6   0    0    0      0         0       0       0      0      0        0
    ##   saimn_d_ie plant_audz forestry_c vmd_headfo shape_Leng  shape_Area
    ## 1          4          0       2651     Centra  1283.3672  18118.3409
    ## 2          6          0       2651     Centra   136.7248   1096.7627
    ## 3          6          0       2651     Centra   113.7443    757.9693
    ## 4          6          0       2651     Centra   109.7517    743.0716
    ## 5          6          0       2651     Centra   162.6895   1500.9411
    ## 6          6          0       2651     Centra  3350.0701 125452.6530
    ##                         geometry prop_priedes PriezuMezi     k_s10     k_s11
    ## 1 MULTIPOLYGON (((536650.7 25...            0          0 Platlapju Šaurlapju
    ## 2 MULTIPOLYGON (((528596.9 25...            0          0 Šaurlapju         0
    ## 3 MULTIPOLYGON (((527092.6 27...            0          0         0         0
    ## 4 MULTIPOLYGON (((545441.8 25...            0          0         0         0
    ## 5 MULTIPOLYGON (((544436.8 25...            0          0 Šaurlapju         0
    ## 6 MULTIPOLYGON (((497918.4 30...           76          1  Skujkoku  Skujkoku
    ##       k_s12 k_s13 k_s14
    ## 1 Šaurlapju     0     0
    ## 2         0     0     0
    ## 3         0     0     0
    ## 4         0     0     0
    ## 5         0     0     0
    ## 6 Šaurlapju     0     0

<br> Tālāk summēju A kolonnu summārās vērtības katram koku tipam un
izvadu jaunās kolonnās (Skuj_proc utt)

    ## Simple feature collection with 6 features and 80 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 497918.4 ymin: 254136.3 xmax: 545441.8 ymax: 300677.8
    ## Projected CRS: LKS-92 / Latvia TM
    ##         id objectid_1           adm1               adm2    atvk    kadastrs
    ## 1 51012798   51012798 Bauskas novads   B?rbeles pagasts 0025400 40440070007
    ## 2 51596262   51596262 Bauskas novads  Vecsaules pagasts 0025550 40920040202
    ## 3 51858796   51858796 Bauskas novads Vecumnieku pagasts 0025560 40940050048
    ## 4 51978889   51978889 Bauskas novads   Kurmenes pagasts 0025480 32620040019
    ## 5 66691890   66691890 Bauskas novads   B?rbeles pagasts 0025400 40440060043
    ## 6 67294609   67294609 Olaines novads    Olaines pagasts 0041400 80800030085
    ##    gtf kvart nog anog nog_plat expl_mezs expl_celi expl_gravj zkat mt izc s10
    ## 1 2006     1  14    0     1.81      1.40         0       0.41   10  5   1  16
    ## 2 2002     1   3    0     0.11      0.11         0       0.00   10  5   1   9
    ## 3 2012     1  12    0     0.08      0.00         0       0.00   32  0   0   0
    ## 4 2012     4  16    0     0.07      0.00         0       0.00   31  4   0   0
    ## 5 2013     1  11    0     0.15      0.14         0       0.01   10  4   1   8
    ## 6 2017   322  12    0    12.55     11.95         0       0.60   10 12   1   1
    ##   a10 h10 d10 g10  n10 bv10 ba10 s11 a11 h11 d11 g11 n11 bv11 ba11 s12 a12 h12
    ## 1  71  23  29  10    0    0    0   9  56  18  20   7   0    0    0   9  31  16
    ## 2  47  23  19  28    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 3   0   0   0   0    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 4   0   0   0   0    0    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 5   8   5   6   0 2000    0    0   0   0   0   0   0   0    0    0   0   0   0
    ## 6  43   9   7   0 1200    0    0   1  33   6   8   0 720    0    0   4  33   9
    ##   d12 g12 n12 bv12 ba12 s13 a13 h13 d13 g13 n13 bv13 ba13 s14 a14 h14 d14 g14
    ## 1  16   5   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 2   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 3   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 4   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 5   0   0   0    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ## 6  10   0 380    0    0   0   0   0   0   0   0    0    0   0   0   0   0   0
    ##   n14 bv14 ba14 jakopj jaatjauno p_darbv p_darbg p_cirp p_cirg atj_gads
    ## 1   0    0    0      0         0       0       0      0      0        0
    ## 2   0    0    0      0         0       0       0      0      0        0
    ## 3   0    0    0      0         0       0       0      0      0        0
    ## 4   0    0    0      0         0       0       0      0      0        0
    ## 5   0    0    0      0         0       4    2017     11   2016     2017
    ## 6   0    0    0      0         0       0       0      0      0        0
    ##   saimn_d_ie plant_audz forestry_c vmd_headfo shape_Leng  shape_Area
    ## 1          4          0       2651     Centra  1283.3672  18118.3409
    ## 2          6          0       2651     Centra   136.7248   1096.7627
    ## 3          6          0       2651     Centra   113.7443    757.9693
    ## 4          6          0       2651     Centra   109.7517    743.0716
    ## 5          6          0       2651     Centra   162.6895   1500.9411
    ## 6          6          0       2651     Centra  3350.0701 125452.6530
    ##                         geometry prop_priedes PriezuMezi     k_s10     k_s11
    ## 1 MULTIPOLYGON (((536650.7 25...            0          0 Platlapju Šaurlapju
    ## 2 MULTIPOLYGON (((528596.9 25...            0          0 Šaurlapju         0
    ## 3 MULTIPOLYGON (((527092.6 27...            0          0         0         0
    ## 4 MULTIPOLYGON (((545441.8 25...            0          0         0         0
    ## 5 MULTIPOLYGON (((544436.8 25...            0          0 Šaurlapju         0
    ## 6 MULTIPOLYGON (((497918.4 30...           76          1  Skujkoku  Skujkoku
    ##       k_s12 k_s13 k_s14 Shaur_proc Plat_proc Skuj_proc
    ## 1 Šaurlapju     0     0         87        71         0
    ## 2         0     0     0         47         0         0
    ## 3         0     0     0          0         0         0
    ## 4         0     0     0          0         0         0
    ## 5         0     0     0          8         0         0
    ## 6 Šaurlapju     0     0         33         0        76

<br> Iegūstu slāni kurā katram mežam ir parādīts cik % daudz ir
skujkoki, platlapji un šaurlapji. Tālāk sāku klasifikāciju un nosaku
mežu tipus, par kuriem ir dati tikai par 1 koku sugu, nosaku 20%
robežvērtību (Ministru kabineta Meža likums).

<br> Tad definēju jauktos mežus - mežus, kur starpība starp koku tipiem
nav lielāka par 20%, piem. 50% skujkoki un 30% lapu koki tiks atzīmēts
kā jaukts mežs.

<br> Turpinu klasificēt un ierakstu !nav mežs! tām rindām, kurās
kumulatīvais koku segums ir zem 20%

<br> Atlikuši tikai poligoni, kuros ir vairāk par 20% starpība. Tajos
ņemu to meža nosaukumu, kura segums ir lielākais.

    ## [[1]]
    ## [1] "Jaukts mežs"
    ## 
    ## [[2]]
    ## [1] "Šaurlapju mežs"
    ## 
    ## [[3]]
    ## [1] "!nav mežs!"
    ## 
    ## [[4]]
    ## character(0)
    ## 
    ## [[5]]
    ## [1] "Skujkoku mežs"
    ## 
    ## [[6]]
    ## [1] "Platlapju mežs"

    ## Found 57316 'Jaukts mežs' values and kept them.

    ## Replaced 327351 existing values with new classifications.

<br> Pēc šīs klasifikācijas ir redzams, ka datu kopā paliek 1054
character(0) vērtības - tie meži, kuros vērtības ir visos 3 laukos un
starpība ir mazāka par 20. Tie klasificējami kā jauktie meži.

    ## Found 1054 character(0) fields and replaced with 'Jaukts mežs'.

<br> Klasifikācija pabeigta. Pārbaudu rezultātus divos veidos - apskatos
kategoriju vērtības datu kopā (nodrošinu, ka visi meži ir klasificēti
atbilstoši nosacījumiem) un apskatos primitīvu statistiku (vidējās
vērtības) katram mežu tipam. Pēc koku seguma vidējām vērtībām un mežu
poligonu savstarpējās attiecības nosaku, ka rezultāti ir ticami un
turpmāko apstrādi neveicu.

    ## 
    ##     !nav mežs!    Jaukts mežs Platlapju mežs  Skujkoku mežs Šaurlapju mežs 
    ##         116468          58370           5706          69713         255397

    ##        Kalk_tips Shaur_mean Skuj_mean  Plat_mean Max_diff A_mean
    ## 1    Jaukts mežs   72.41948  66.15294   7.649786 64.76969    NaN
    ## 2 Šaurlapju mežs   117.3694    14.643    3.81961 113.5498    NaN
    ## 3     !nav mežs!   4.797876  1.366075 0.03849985 4.759376    NaN
    ## 4  Skujkoku mežs   9.151234   117.959  0.6645819 117.2944    NaN
    ## 5 Platlapju mežs   58.49877  9.936383    194.391 184.4546    NaN
