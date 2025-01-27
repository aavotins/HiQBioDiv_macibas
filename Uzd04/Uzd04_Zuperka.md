Uzd04_Zuperka
================
2025-01-13

<br> Iepriekš biju izdzēsis nevajadzīgos shp failus, tos lejupielādēju
un pārtaisu uz parquet formātu un izdzēšu shp formātu. Tālāk nav
vajadzīga tāpēc vienu reizi izpildu un uzlieku eval=FALSE.

<br> Izmantošu mana funkcija lai apstradātu apvienoto centra mežniecības
failu.

    ## Loading required package: dplyr

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    ## Loading required package: arrow

    ## 
    ## Attaching package: 'arrow'

    ## The following object is masked from 'package:utils':
    ## 
    ##     timestamp

    ## Loading required package: sf

    ## Linking to GEOS 3.12.2, GDAL 3.9.3, PROJ 9.4.1; sf_use_s2() is TRUE

    ## Loading required package: terra

    ## terra 1.7.83

    ## 
    ## Attaching package: 'terra'

    ## The following object is masked from 'package:arrow':
    ## 
    ##     buffer

    ## Loading required package: tools

    ## |---------|---------|---------|---------|=========================================                                          

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

    ## Centra mežniecības apvienotā faila apstrādes laiks: 1.6 minūtes

![](Uzd04_Zuperka_files/figure-gfm/izpildu%20mana_funkcija%20apvienotajam%20failam-1.png)<!-- -->
<br>Iepriekš izmantoju fasterize, kas bija ilgi un ļoti nestabili
attiecībā uz RAM (izmantošana visu laiku lēkāja no 7-8 GB līdz pat 23 GB
un visu Latviju skaitļoja gandrīz 2 stundas). Terra rasterize izmanto
daudz mazāk resursus (īpaši RAM) un visu izdara ļoti ātri. Abas
funkcijas izmantoja 5 kodolus pastāvīgi. <br>Tālāk pārbaudīšu šo pašu
funkciju uz katras nodaļas parquet atsevišķi. Sākumā uzrakstu funkciju
datu ielasīšanai.

    ## Loading required package: tidyverse

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ forcats   1.0.0     ✔ readr     2.1.5
    ## ✔ ggplot2   3.5.1     ✔ stringr   1.5.1
    ## ✔ lubridate 1.9.3     ✔ tibble    3.2.1
    ## ✔ purrr     1.0.2     ✔ tidyr     1.3.1
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ lubridate::duration() masks arrow::duration()
    ## ✖ tidyr::extract()      masks terra::extract()
    ## ✖ dplyr::filter()       masks stats::filter()
    ## ✖ dplyr::lag()          masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

<br> Pielietošu mana_funkcija katram parquet failam ar for un izvadīšu
izpildes laiku.

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

    ## [[1]]
    ## class       : SpatRaster 
    ## dimensions  : 2860, 4700, 1  (nrow, ncol, nlyr)
    ## resolution  : 100, 100  (x, y)
    ## extent      : 302800, 772800, 162900, 448900  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : nodala2655.parquet.tif 
    ## name        : lyr.1 
    ## min value   :     0 
    ## max value   :     1 
    ## 
    ## [[2]]
    ## class       : SpatRaster 
    ## dimensions  : 2860, 4700, 1  (nrow, ncol, nlyr)
    ## resolution  : 100, 100  (x, y)
    ## extent      : 302800, 772800, 162900, 448900  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : nodala2655.parquet.tif 
    ## name        : lyr.1 
    ## min value   :     0 
    ## max value   :     1 
    ## 
    ## [[3]]
    ## class       : SpatRaster 
    ## dimensions  : 2860, 4700, 1  (nrow, ncol, nlyr)
    ## resolution  : 100, 100  (x, y)
    ## extent      : 302800, 772800, 162900, 448900  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : nodala2655.parquet.tif 
    ## name        : lyr.1 
    ## min value   :     0 
    ## max value   :     1 
    ## 
    ## [[4]]
    ## class       : SpatRaster 
    ## dimensions  : 2860, 4700, 1  (nrow, ncol, nlyr)
    ## resolution  : 100, 100  (x, y)
    ## extent      : 302800, 772800, 162900, 448900  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : nodala2655.parquet.tif 
    ## name        : lyr.1 
    ## min value   :     0 
    ## max value   :     1 
    ## 
    ## [[5]]
    ## class       : SpatRaster 
    ## dimensions  : 2860, 4700, 1  (nrow, ncol, nlyr)
    ## resolution  : 100, 100  (x, y)
    ## extent      : 302800, 772800, 162900, 448900  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source      : nodala2655.parquet.tif 
    ## name        : lyr.1 
    ## min value   :     0 
    ## max value   :     1

    ## Kopējais failu apstrādes laiks atsevišķi: 7.4 minūtes.

<br> Izpildes laiks 5 atsevišķiem failiem ir daudz lielāks nekā vienam
apvienotajam failam. RAM izmanto līdzīgi. Kodolus ar - pārsvarā 5
kodoli. <br> CIzmantošu foreach funkciju ar vienu kodolu un izpildīšu
funkciju katram failam atsevišķi. doParallel un terra ļoti dīvaini iet
kopā - dažreiz kods izpildās kā vajag un kļūdu nemet, bet citās reizēs
izmet `Error: external pointer is not valid`. Uzlieku error=TRUE, jo
izskatās ka faili tiek izveidoti un saglabāti.

    ## Loading required package: doParallel

    ## Loading required package: foreach

    ## 
    ## Attaching package: 'foreach'

    ## The following objects are masked from 'package:purrr':
    ## 
    ##     accumulate, when

    ## Loading required package: iterators

    ## Loading required package: parallel

    ## [[1]]
    ## class       : SpatRaster

    ## Error: external pointer is not valid

    ## Kopējais izpildes laiks atsevišķi ar doParallel un 1 kodolu: 7.4 min.

<br> foreach izmantošana ar vienu kodolu aizņem gandrīz tikpat ilgi cik
iterējot ar lapply funkciju. Aizņemtais RAM tāds pats. <br><br>
Pārbaudīšu mana_funkcija ar 5 kodoliem. Izpildās ātri, 1.9 minūtēs. Tā
pati problēma kas ar 1 kodolu, bet atkal izskatās ka faili tiek
izveidoti un ir lasāmi.

    ## [[1]]
    ## class       : SpatRaster

    ## Error: external pointer is not valid

    ## Kopējais izpildes laiks atsevišķi ar doParallel un 5 kodoliem: 3.3 min.

<br> Dīvaini ka terra izmantošana doParallel ir ļoti nestabila - dažbrīd
funkcija izpildās un kļūdu nav, bet citas reizes izmet kļūdu Error:
External pointer is not valid. Vienīgais kas izskatās šajā situācijā
palīdz ir pēc iepr. izpildes izdzēst esošos mazos failus.
