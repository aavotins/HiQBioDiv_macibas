Uzdevuma izpildei man bija nepieciešamas šādas pakotnes:

    library(httr)
    library(archive)
    library(sf)
    library(arrow)
    library(microbenchmark)
    library(dplyr)
    library(tidyverse)

1.uzdevums Shapefile lejupielāde

    centrs_url <- "https://data.gov.lv/dati/lv/dataset/40014c0a-90f5-42be-afb2-fe3c4b8adf92/resource/392dfb67-eeeb-43c2-b082-35f9cf986128/download/centra.7z"  # Replace with your URL
    lejupielades_direktorija <- "C:/Users/riter/Desktop/HiQBioDiv/Uzd02"
    lejupieladei <- file.path(lejupielades_direktorija, "centra.7z")

    lejuplade <- GET(centrs_url)
    writeBin(content(lejuplade, "raw"), lejupieladei)
    shapefile_direktorija <- file.path(lejupielades_direktorija, "centrs")
    dir.create(shapefile_direktorija, showWarnings = FALSE)
    archive::archive_extract(lejupieladei, dir = shapefile_direktorija)

Shapefile ielasīšana

    nodala2651 <- "C:/Users/riter/Desktop/HiQBioDiv/Uzd02/centrs/nodala2651.shp"
    sf_data <- st_read(nodala2651, quiet = TRUE)

Shapefile pārveide par ģeosaini

    geopackage_path <- "nodala2651_geopackage.gpkg"
    st_write(sf_data, geopackage_path, driver = "GPKG", append= FALSE, quiet= TRUE)

Shapefile pārveide par geoparquet. Funkcija kādu mirkli stādāja,
izmantojot tikai 61 un 62 rindu, taču pēkšņi pārstāja. Tika rasts
risinājums, pārveidojot ģeometrijas datus kā wkt un pēc tam to ierakstot
parquet failā.

    wkt <- st_as_text(sf_data$geometry)
    df<-data.frame(geometry=wkt)
    geoparquet_path <- "nodala2651_geoparquet.parquet"
    write_parquet(df, geoparquet_path)

## 

Failu izmēru noteikšana

    shapefile_izmers <- sum(file.info(list.files(pattern = "nodala2651.*"))$size)
    geosainis_izmers <- file.info("nodala2651_geopackage.gpkg")$size
    geoparquet_izmers <- file.info("nodala2651_geoparquet.parquet")$size

Izmērs ir noteikts baitos, vienkāršības pēc pārveidoju uz MB

    shapefile_izmers_mb<-shapefile_izmers/1e6
    geosainis_izmers_mb<-geosainis_izmers/1e6
    geoparquet_izmers_mb<-geoparquet_izmers/1e6
    failu_izmeri<-data.frame(shapefile_izmers_mb,geosainis_izmers_mb,geoparquet_izmers_mb)
    print(failu_izmeri)

    ##   shapefile_izmers_mb geosainis_izmers_mb geoparquet_izmers_mb
    ## 1            70.94827            57.18835             13.75992

Ielasīšanas laika noteikšana

    shapefile_videjais<-microbenchmark(shapefile_laiks <- st_read(nodala2651, quiet=TRUE), times = 10)
    print(shapefile_videjais, unit="s")

    ## Unit: seconds
    ##                                                  expr      min       lq
    ##  shapefile_laiks <- st_read(nodala2651, quiet = TRUE) 5.039666 5.119221
    ##      mean   median       uq      max neval
    ##  5.389051 5.162088 5.554233 6.095408    10

    geosainis_videjais<-microbenchmark(geosainis_laiks<- st_read("nodala2651_geopackage.gpkg", quiet=TRUE), times = 10)
    print(geosainis_videjais, unit="s")

    ## Unit: seconds
    ##                                                                    expr
    ##  geosainis_laiks <- st_read("nodala2651_geopackage.gpkg", quiet = TRUE)
    ##       min      lq     mean   median       uq      max neval
    ##  3.302366 3.40972 3.574601 3.463934 3.585401 4.401589    10

    geoparquet_videjais<-microbenchmark(geoparquet_laiks <- read_parquet("nodala2651_geoparquet.parquet"), times=10)
    print(geoparquet_videjais, unit="s")

    ## Unit: seconds
    ##                                                               expr      min
    ##  geoparquet_laiks <- read_parquet("nodala2651_geoparquet.parquet") 0.059911
    ##         lq       mean    median       uq       max neval
    ##  0.0631405 0.07307407 0.0641753 0.065549 0.1550176    10

2.uzdevums. Izveidoju kombinēto failu no visiem Centra shapefiles

    shapefile_paths <- list.files(path = "C:/Users/riter/Desktop/HiQBioDiv/Uzd02/centrs", 
                                  pattern = "\\.shp$", 
                                  full.names = TRUE)
    shapefiles<-lapply(shapefile_paths, st_read, quiet=TRUE)
    combined <- do.call(rbind, shapefiles)
    st_write(combined, "combined_shapefile.shp", driver = "ESRI Shapefile", quiet=TRUE)

Pārbaudu, vai nav nederīgu ģeometriju un vai visi ģeometrijas tipi ir
MULTIPOLYGON

    invisible(geometry_types <- st_geometry_type(combined))
    any(is.na(geometry_types))

    ## [1] FALSE

    all(geometry_types %in% c("MULTIPOLYGON"))

    ## [1] TRUE

3.uzdevums Kombinētajā datu failā ir grūti orientēties lielā kolonnu
daudzuma dēļ, tāpēc izdalīju pašas būtiskākās- 1. stāva sugas un to
šķērsgriezumus- atsevišķā objektā. Man neskaidru iemeslu dēļ ģeometrijas
veids seko līdzi jaunajam objektam, taču to bija viegli izņemt.

    koki<-combined %>% rename(sugas1=s10, skers1=g10, sugas2=s11, skers2=g11, sugas3=s12, skers3=g12, sugas4=s13, skers4=g13, sugas5=s14, skers5=g14) %>% select(sugas1, skers1, sugas2, skers2, sugas3, skers3, sugas4, skers4, sugas5, skers5)
    koki<-koki %>% select(-geometry)

DBF specifikācijā priede tiek atzīmēta ar skaitli 1, citas priedes ir
atzīmētas ar skaitli 14, kamēr ciedrpriede ir atzīmēta ar 22

    merksugas <- c("1", "14", "22")

Turpmākajām darbībām nepieciešams objektu koki padarīt par datu rāmi

    koki<-as.data.frame(koki)

Priedes šķērsgriezumu summa

    koki <- koki %>%
      rowwise() %>%
      mutate(
        priedes_skers = sum(
          c_across(c(skers1, skers2, skers3, skers4, skers5))[
            c_across(c(sugas1, sugas2, sugas3, sugas4, sugas5)) %in% merksugas
          ],
          na.rm = TRUE
        )
      ) %>%
      ungroup()

Priedes šķērsgriezumu proporcijas aprēķināšana

    koki <- koki %>%
      rowwise() %>%
      mutate(
        prop_priedes = (c_across(c(priedes_skers))/sum(c_across(c(skers1, skers2, skers3, skers4, skers5)))),
          na.rm = TRUE
        )
    #Lai izvairītos no nederīgām vērtībām, nomainu tās uz 0
    koki$prop_priedes[is.nan(koki$prop_priedes) | 
                                   is.na(koki$prop_priedes) | 
                                   is.infinite(koki$prop_priedes)] <- 0

Priežu mežaudžu noteikšana

    koki<-koki %>% mutate(PriezuMezi=ifelse(prop_priedes>=0.75, 1, 0))

Priežu mežaudžu īpatsvara aprēķināšana centra rajonā

    priezu_mezu_ipatsvars=sum(koki$PriezuMezi==1)/nrow(koki)
    print(priezu_mezu_ipatsvars)

    ## [1] 0.1784925

4.uzdevums Visus Pinaceae dzimtas augus klasificēju kā skujkokus.

    skujkoki<- c(1, 3, 13, 14, 15, 22, 28, 29)

Balstoties uz apskatītajiem avotiem, šaurlapjos apvienoju vītolus,
bērzus, alkšņus un dažus citus

    saurlapji<- c(4, 6, 8, 9, 19, 20, 68)

Pēc izslēgšanas metodes, platlapjos apvienoju pārējās sugas

    platlapji<- c(8, 9, 10, 11, 12, 16, 17, 18, 21, 24, 25, 26, 27, 32, 35, 50, 61, 62, 63, 64, 65, 66, 67, 69)

Ja 3. uzdevumā par priežu mežu bija uzskatāma tāda mežaudze, kurā priežu
īpatsvars ir vismaz 75%, tad šeit uzskatu, ka mežaudze ir šaurlapju,
platlapju vai skujkoku, ja kāds no šiem koku veidiem ir ar vismaz 75%
īpatsvaru. Tā kā var arī būt situācija, kurā neviens no koku veidiem
nesasniedz 75% īpatsvaru, tad šajā gadījumā uzskatu, ka mežaudze ir
jaukta. Var arī būt situācija, kurā nav neviena koka sugas, tos es
atzīmēju kā “nav\_koku”.

Atkal izveidoju jaunu objektu, kurā ir tikai koku sugas un to
šķērsgriezumi, kam atkal neskaidru iemeslu dēļ seko līdzi ģeometrijas
veids, kuru atkal bez problēmām varēju izņemt.

    koki4<-combined %>% rename(sugas1=s10, skers1=g10, sugas2=s11, skers2=g11, sugas3=s12, skers3=g12, sugas4=s13, skers4=g13, sugas5=s14, skers5=g14) %>% select(sugas1, skers1, sugas2, skers2, sugas3, skers3, sugas4, skers4, sugas5, skers5)
    koki4<-koki4 %>% select(-geometry)

Tiek saskaitīti šķērsgriezumi katram koka veidam katrā kokaudzē

    koki4 <- koki4 %>% 
      mutate (skujkoki = 
                ifelse(sugas1 %in% skujkoki, skers1, 0)+
                ifelse(sugas2 %in% skujkoki, skers2, 0)+
                ifelse(sugas3 %in% skujkoki, skers3, 0)+
                ifelse(sugas4 %in% skujkoki, skers4, 0)+
                ifelse(sugas5 %in% skujkoki, skers5, 0))

    koki4 <- koki4 %>% 
      mutate (platlapji = 
                ifelse(sugas1 %in% platlapji, skers1, 0)+
                ifelse(sugas2 %in% platlapji, skers2, 0)+
                ifelse(sugas3 %in% platlapji, skers3, 0)+
                ifelse(sugas4 %in% platlapji, skers4, 0)+
                ifelse(sugas5 %in% platlapji, skers5, 0))

    koki4 <- koki4 %>% 
      mutate (saurlapji = 
                ifelse(sugas1 %in% saurlapji, skers1, 0)+
                ifelse(sugas2 %in% saurlapji, skers2, 0)+
                ifelse(sugas3 %in% saurlapji, skers3, 0)+
                ifelse(sugas4 %in% saurlapji, skers4, 0)+
                ifelse(sugas5 %in% saurlapji, skers5, 0))

Aprēķinu proporcijas katram koka veidam katrā kokaudzē

    koki4<- koki4 %>%
      rowwise %>%
      mutate(prop_skujkoki = (c_across(c(skujkoki))/sum(c_across(c(skujkoki, saurlapji, platlapji)))))
    koki4<- koki4 %>%
      rowwise %>%
      mutate(prop_platlapji = (c_across(c(platlapji))/sum(c_across(c(skujkoki, saurlapji, platlapji)))))
    koki4<- koki4 %>%
      rowwise %>%
      mutate(prop_saurlapji = (c_across(c(saurlapji))/sum(c_across(c(skujkoki, saurlapji, platlapji)))))
    #Nomainu nederīgās vērtības uz 0
    koki4$prop_skujkoki[is.nan(koki4$prop_skujkoki) | 
                                   is.na(koki4$prop_skujkoki) | 
                                   is.infinite(koki4$prop_skujkoki)] <- 0
    koki4$prop_platlapji[is.nan(koki4$prop_platlapji) |
                           is.na(koki4$prop_platlapji) |
                           is.infinite(koki4$prop_platlapji)] <- 0
    koki4$prop_saurlapji[is.nan(koki4$prop_saurlapji) |
                           is.na(koki4$prop_saurlapji) |
                           is.infinite(koki4$prop_saurlapji)] <- 0

Kokaudžu veidu noteikšana

    koki4<-koki4 %>% mutate(audzes_veids=ifelse(prop_skujkoki>=0.75, "skujkoku", ifelse(prop_platlapji>=0.75, "platlapju", ifelse(prop_saurlapji>=0.75, "saurlapju", "jaukta"))))
    koki4<-koki4 %>% mutate(audzes_veids=ifelse(prop_skujkoki==0 & prop_platlapji==0 & prop_saurlapji==0, "nav_koku", audzes_veids))

Kokaudžu veidu proporcijas(%) aprēķināšana

    procenti_skujkoki=sum(koki4$audzes_veids=="skujkoku")/nrow(koki4)
    procenti_platlapji=sum(koki4$audzes_veids=="platlapju")/nrow(koki4)
    procenti_saurlapji=sum(koki4$audzes_veids=="saurlapju")/nrow(koki4)
    procenti_jaukta=sum(koki4$audzes_veids=="jaukta")/nrow(koki4)
    procenti_nav_koku=sum(koki4$audzes_veids=="nav_koku")/nrow(koki4)

    meži<-data.frame(procenti_skujkoki, procenti_platlapji, procenti_saurlapji, procenti_jaukta, procenti_nav_koku)
    print(meži)

    ##   procenti_skujkoki procenti_platlapji procenti_saurlapji procenti_jaukta
    ## 1         0.2880011        0.009480534          0.1473754       0.2673806
    ##   procenti_nav_koku
    ## 1         0.2877623
