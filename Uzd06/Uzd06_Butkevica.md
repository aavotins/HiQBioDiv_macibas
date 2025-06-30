Sestais uzdevums: dažādu slāņu savienošana vienotai ainavas
aprakstīšanai
================
Jekaterīna Butkeviča
2025. gads 29. jūnijs

## Pakotnes

``` r
if(!require(httr)) {install.packages("httr"); require(httr)}
if(!require(ows4R)) {install.packages("ows4R"); require(ows4R)}
if(!require(here)) {install.packages("here"); require(here)}
if(!require(sfarrow)) {install.packages("sfarrow"); require(sfarrow)}
if(!require(terra)) {install.packages("terra"); require(terra)}
if(!require(concaveman)) {install.packages("concaveman"); require(concaveman)}
if(!require(fasterize)) {install.packages("fasterize"); require(fasterize)}
if(!require(dplyr)) {install.packages("dplyr"); require(dplyr)}
if(!require(sf)) {install.packages("sf"); require(sf)}
```

# Datu sagatavošana

Es izjūtu vajadzību atkārtoti sagatavot LAD datus. Lai to paveiktu,
vispirms izveidoju vienotu Centrālās mežniecības poligonu.

``` r
MVR_Centra <- st_read_parquet(here("3uzd","Output","Data","centra_apvienota.parquet"))

# Apvienot visas ģeometrijas vienā poligonā
MVR_Centra_apvienots <- st_union(MVR_Centra)

#Veidojam poligonu ap Centra mežniecību, balstoties uz koordinātu matricu.
MVR_Centra_koord <- st_coordinates(MVR_Centra_apvienots)
MVR_Centra_poligons <- concaveman(MVR_Centra_koord)
# Pārliecināmies, ka tās ir ievadītas kā koordinātu saraksts
MVR_Centra_poligons <- st_polygon(list(MVR_Centra_poligons[, 1:2]))
# Pārvērst par sf objektu
MVR_Centra_poligons <- st_sfc(MVR_Centra_poligons)
```

Aplūkosim rezultātu:

``` r
plot(MVR_Centra_poligons)
```

![](Uzd06_files/figure-gfm/LAD_poligons_vizualizacija-1.png)<!-- -->

Atgriežam koordinātu sistēmu un veidojam kārbu apkārt Centrālās
mežniecības teritorijai. Pārvēršam to WGS koordinātu sistēmā.

``` r
#Atgriezt koordinātu sistēmu
st_crs(MVR_Centra_poligons) <- st_crs(MVR_Centra)

bbox_Centra <- st_bbox(MVR_Centra_poligons)

#uz wgs
bbox_Centra <- st_transform(bbox_Centra, crs = 4326)
```

Veicām datu pieprasījumu, izmantojot bboxu ap Centrālo mežniecību.

``` r
wfs_LAD <- "https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer"
LAD_client <- WFSClient$new(wfs_LAD, serviceVersion = "2.0.0")
LAD_client$getFeatureTypes(pretty = TRUE)
```

    ##        name title
    ## 1 LAD:Lauki Lauki

``` r
url_LAD <- parse_url(wfs_LAD)
url_LAD$query <- list(
  service = "wfs",
  request = "GetFeature",
  typename = "Lauki",
  srsName = "EPSG:4326",
  bbox_Centra = paste(bbox_Centra, collapse = ",")
)
request <- build_url(url_LAD)
LAD_dati <- st_read(request)
```

    ## Reading layer `Lauki' from data source 
    ##   `https://karte.lad.gov.lv/arcgis/services/lauki/MapServer/WFSServer?service=wfs&request=GetFeature&typename=Lauki&srsName=EPSG%3A4326&bbox_Centra=22.4228498646058%2C56.2430334496512%2C25.5728086891398%2C57.4110067014047' 
    ##   using driver `GML'
    ## Simple feature collection with 3000 features and 11 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 21.02124 ymin: 55.71385 xmax: 28.08746 ymax: 58.01172
    ## Geodetic CRS:  WGS 84

``` r
LAD_dati <- st_transform(LAD_dati, crs = 3059 )

centrs <- st_intersection(LAD_dati, MVR_Centra_poligons)
```

Saglabāju Centrālās mežniecības teritorijas lauka datus.

``` r
dir_Centra_LAD <- here("6uzd","Output", "LAD")
dir.create(dir_Centra_LAD, recursive = TRUE)
LAD_pilns_cels <- file.path(dir_Centra_LAD, "LAD_Centra.parquet")
st_write_parquet(centrs, LAD_pilns_cels)
```

Ielasu rastra references slāni.

``` r
LV10m_10km <- rast(here("3uzd", "Data", "references_zenodo", "rastrs", "LV10m_10km.tif"))
```

# Pirmais uzdevums

Uzdevuma ietvaros veidojamo rastru pārklājumam un šūnu novietojumam ir
jāsakrīt ar projekta Zenodo repozitorijā ievietoto references slānim
LV10m_10km.tif. Rasterizēšanas nosacījumam izmantojiet šūnas centru.

Rasterizējiet Lauku atbalsta dienestā reģistrētos laukus, izveidojot
klasi “100” (vai izmantojiet trešajā uzdevumā sagatavoto slāni, ja tas
ir korekts);

elasu Centrālās mežniecības teritorijas lauku datus.

``` r
LAD_Centra <- st_read_parquet(LAD_pilns_cels)
```

Lai rasterizētu, vajadzēja vienkāršot formu, tāpēc izmantoju bbox.
Apgriežu referenci atbilstoši interesējošajam teritorijas apgabalam.
Veicu LAD vektorslāņa rasterizāciju. Neesmu lietojusi argumentu
`touches = TRUE`, tāpēc vērtība tiks piešķirta tikai pikseļiem, kuru
centroīds atrodas lauku poligonos.

``` r
#  Lai rastrizētu, vajaj vienkāršot formu
LAD_Centra_bbox <- st_bbox(LAD_Centra)

LV10m_10km_apgriezts <- terra::crop(LV10m_10km, LAD_Centra_bbox)
LAD_Centra_rast <- rasterize(LAD_Centra, LV10m_10km_apgriezts)
```

Saskaņā ar uzdevuma nosacījumiem nepieciešams mainīt rastra vērtību
kodējumu.

``` r
# Aizvietot rastra vērtības: Rastra pikseļiem ar vērtību 1 tiek piešķirta jauna vērtība 100.
LAD_Centra_100_rast <- subst(LAD_Centra_rast,1,100)
```

Paskatīsimies uz rezultātiem:

``` r
plot(LAD_Centra_100_rast) #mmm dzeltens uz balta
```

![](Uzd06_files/figure-gfm/vizual_2-1.png)<!-- -->

# Otrais uzdevums

Izveidojiet rastra slāņus ar skujkoku (klase “204”), šaurlapju (klase
“203”), platlapju (klase “202”) un jauktu koku mežiem (klase “201”) no
sevis ierosinātās klasifikācijas otrajā uzdevumā.

Veidoju sarakstu ar sugu klasifikāciju pēc koku sugām:

``` r
# Sugu nodalījums 
Koku_tips <- list(
  skujkoki = c(1, 3, 13, 14, 15, 22, 23, 28, 29), 
  platlapju_koki = c(10, 11, 17, 18, 24, 25, 26, 27, 32, 35, 50, 61, 63, 64, 65, 66, 67, 69),
  saurlapju_koki =c(4, 6, 8, 9, 12, 16, 19, 20, 21, 62, 68)
)
```

Tālāk izveidoju funkciju, kas aprēķina kopējo šķērs­laukumu, kā arī
atsevišķi — šķērs­laukumu katram koku tipam. Balstoties uz koku tipu
šķērs­laukuma īpatsvaru no kopējā šķērs­laukuma, tiek klasificētas
mežaudzes. Pamatojoties uz mežaudzes veidu, tiks pievienots arī
attiecīgais kods.

``` r
MVR_Centra <- MVR_Centra %>%
  mutate(
    kopejais_skerslaukums = g10 + g11 + g12 + g13 + g14,
    
    skujkoku_skerslaukums = 
      ifelse(s10 %in% Koku_tips$skujkoki, g10, 0) + 
      ifelse(s11 %in% Koku_tips$skujkoki, g11, 0) +
      ifelse(s12 %in% Koku_tips$skujkoki, g12, 0) +
      ifelse(s13 %in% Koku_tips$skujkoki, g13, 0) +
      ifelse(s14 %in% Koku_tips$skujkoki, g14, 0),
    
    platlapju_skerslaukums = 
      ifelse(s10 %in% Koku_tips$platlapju_koki, g10, 0) + 
      ifelse(s11 %in% Koku_tips$platlapju_koki, g11, 0) +
      ifelse(s12 %in% Koku_tips$platlapju_koki, g12, 0) +
      ifelse(s13 %in% Koku_tips$platlapju_koki, g13, 0) +
      ifelse(s14 %in% Koku_tips$platlapju_koki, g14, 0),
    
    saurlapju_skerslaukums = 
      ifelse(s10 %in% Koku_tips$saurlapju_koki, g10, 0) + 
      ifelse(s11 %in% Koku_tips$saurlapju_koki, g11, 0) +
      ifelse(s12 %in% Koku_tips$saurlapju_koki, g12, 0) +
      ifelse(s13 %in% Koku_tips$saurlapju_koki, g13, 0) +
      ifelse(s14 %in% Koku_tips$saurlapju_koki, g14, 0),
    
    mezaudzes_veids = case_when(
      kopejais_skerslaukums == 0 ~ NA_character_,
      skujkoku_skerslaukums / kopejais_skerslaukums >= 0.75 ~ "Skujkoku mežs",
      platlapju_skerslaukums / kopejais_skerslaukums >= 0.75 ~ "Platlapju mežs",
      saurlapju_skerslaukums / kopejais_skerslaukums >= 0.75 ~ "Šaurlapju mežs",
      TRUE ~ "Jaukts mežs"
    ),
    
    meza_grupa = case_when(
      mezaudzes_veids == "Skujkoku mežs" ~ 204,
      mezaudzes_veids == "Šaurlapju mežs" ~ 203,
      mezaudzes_veids == "Platlapju mežs" ~ 202,
      mezaudzes_veids == "Jaukts mežs" ~ 201,
      TRUE ~ NA_integer_
    )
  )
```

Atsevišķi atdalu datus par katru meža veidu:

``` r
skujkoku_klase204 <- MVR_Centra %>% filter(meza_grupa == 204)
saurlapju_klase203 <- MVR_Centra %>% filter(meza_grupa == 203)
platlapju_klase202 <- MVR_Centra %>% filter(meza_grupa == 202)
jaukts_klase201 <- MVR_Centra %>% filter(meza_grupa == 201)
```

Balstoties uz šīm datu tabulām, veidoju atsevišķus rastra slāņus.

``` r
r_skujkoku_klase204 <- rasterize(skujkoku_klase204,LV10m_10km_apgriezts, field = "meza_grupa")
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
r_saurlapju_klase203 <- rasterize(saurlapju_klase203,LV10m_10km_apgriezts, field = "meza_grupa")
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
r_platlapju_klase202 <- rasterize(platlapju_klase202,LV10m_10km_apgriezts, field = "meza_grupa")
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
r_jaukts_klase201 <- rasterize(jaukts_klase201,LV10m_10km_apgriezts, field = "meza_grupa")
```

Veicu visu slāņu vizualizāciju.

``` r
rastru_saraksts <- list(
  "Skujkoku klase 204" = r_skujkoku_klase204,
  "Saurlapju klase 203" = r_saurlapju_klase203,
  "Platlapju klase 202" = r_platlapju_klase202,
  "Jaukts klase 201" = r_jaukts_klase201
)

for (nosaukums in names(rastru_saraksts)) {
  plot(rastru_saraksts[[nosaukums]], main = nosaukums, col = "black", legend = FALSE)
}
```

![](Uzd06_files/figure-gfm/mezu_slani_viz-1.png)<!-- -->![](Uzd06_files/figure-gfm/mezu_slani_viz-2.png)<!-- -->![](Uzd06_files/figure-gfm/mezu_slani_viz-3.png)<!-- -->![](Uzd06_files/figure-gfm/mezu_slani_viz-4.png)<!-- -->

Rezultāts izskatās normāli.

# Trešais uzdevums

Papildieniet otrā punkta rastrus, izveidojot jaunus slāņus, ar
informāciju par mežaudzes vecuma grupu kā klases vērtības otro ciparu.
Dalījumam mežaudžu vecuma grupās, izmantojiet vecumgrupas un vecumklases
un, aprēķinot galvenās cirtes vecumu, pieņemiet “I un augstāka”
bonitāti.

### **Informācija par kokaudžu vecumklasem un vecumgrupām**

Informācija no <https://www.letonika.lv/>:

**Kokaudzes vecumgrupa** ir atkarībā no valdošās sugas koku vecuma un
tai noteiktā ciršanas vecuma kokaudzi ieskaita vienā no 5 vecumgrupām:
jaunaudzē, vidēja vecuma audzē, briestaudzē, pieaugušā audzē vai
pāraugušā audzē.

Pie jaunaudzēm pieder pirmas 2 vecumklasašu audzes. Vidēja vecuma audzes
atrodas starp jaunaudzēm un briestaudzēm. Pie briestaudzēm — viena
vecumklase pirms ciršanas vecuma. Pie pieaugušām audzēm — divas
vecumklases, sākot ar ciršanas vecumu. Audzes, kuru vecums pārsniedz
pieaugušo audžu vecumu, ieskaita pāraugušās audzēs.

**Kokaudzes vecumklase** — kategorija kokaudžu iedalīšanai pēc vecuma,
ņemot vērā atšķirības augšanas gaitā.

Skujkoku un cieto lapkoku audzēm pieņemtais vecumklases intervāls ir 20
gadu. Saskaņā ar <https://tezaurs.lv> , pie cietiem lapu kokiem pieder
goba, ozols, osis. Mīkstajiem lapu kokiem — 10 gadu. Īpaši
ātraudzīgajiem kokiem (baltalkšņiem, blīgznām un vītoliem) — 5 gadi.

Tātad vispirms ir nepieciešams izveidot kolonnu ar kokaudzes vecumklasi.
Lai to panāktu, veicu koku sugu klasifikāciju vecumklasēs.

``` r
# Apskatu vērtības
unique(MVR_Centra$s10) 
```

    ##  [1] "11" "9"  "4"  "1"  "3"  "10" "8"  "6"  "12" "24" "21" "19" "16" "13" "20"
    ## [16] "23" "15" "32" "63" "14" "25" "64" "17" "61" "50" "35" "68" "67" "65" "22"

``` r
# Viss, par ko nebija informācijas aprakstā, tika pieskaitīts 10 gadu klasei.

vecumklases_saraksts  <- list(
  klase_5gadi = c(9, 20, 21), 
  klase_10gadi = c(4, 6, 8, 12, 17, 19, 24, 25, 32, 35, 50, 63, 67, 68),
  klase_20gadi =c(1, 3, 10, 11, 13, 14, 15, 16, 22, 23, 28, 29, 61, 64, 65)
)

# Piešķiru vērtības
MVR_Centra <- MVR_Centra %>%
  mutate(
    Koku_vecumklase = case_when(
      s10 %in% vecumklases_saraksts $klase_5gadi ~ 5,
      s10 %in% vecumklases_saraksts $klase_10gadi ~ 10,
      s10 %in% vecumklases_saraksts $klase_20gadi ~ 20,
      TRUE ~ NA_real_
    )
  )
```

Tā kā kokaudzes vecumgrupas noteikšanai ir nepieciešama informācija par
sugai noteikto ciršanas vecumu, es izmantoju informāciju no [“Meža
likuma”](https://likumi.lv/ta/id/2825#p9) un veicu kokaudžu
klasifikāciju.

**Saskaņā ar Mežu likumu**, bonītates klasei “I un augstāka”, galvenas
cirtes vecums ir:

- Ozols, Priede un lapegle - 101 gads;
- Egle, osis, liepa, goba, vīksna un kļava – 81;
- Bērzs, Melnalksnis - 71;
- Apse - 41.

Tās sugas, kuru galvenās cirtes vecums Meža likumā nebija norādīts,
sadalīju pēc šāda principa: 1) Ja līdzīgas grupas ir kāda no kategorijām
(piemēram, ja ozola galvenās cirtes vecums ir 81 gads, tad grupai “citi
ozoli” izvēlējos to pašu vecumu).

2)  Ja sugas vecumklase ir 5 gadi, tad paņemu minimālo no piedāvātajiem
    ciršanas vecumiem.

3)  Ja vecumklase ir 10 vai 20 gadi, kā galveno cirtes vecumu izmantoju
    81 gadu.

``` r
# Koku sugu klasifikācija galvenās cirtes vecumā

galvcirt_vecums_saraksts  <- list(
  galv_cirt101 = c(1, 10, 13, 14, 22, 61, 63), 
  galv_cirt81 = c(3, 11, 12, 15, 16, 17, 19, 23, 24, 25, 28, 29, 32, 35, 50, 64, 65, 67, 68),
  galv_cirt71 = c(4, 6, 9 ),
  galv_cirt41 = c(8, 20, 21)
)

MVR_Centra <- MVR_Centra %>%
  mutate(
    galvcirt_vecums = case_when(
      s10 %in% galvcirt_vecums_saraksts$galv_cirt101 ~ 101,
      s10 %in% galvcirt_vecums_saraksts$galv_cirt81 ~ 81,
      s10 %in% galvcirt_vecums_saraksts$galv_cirt71 ~ 71,
      s10 %in% galvcirt_vecums_saraksts$galv_cirt41 ~ 41,
      TRUE ~ NA_real_
    )
  )
```

Balstoties uz uzdevuma sākumā sniegto aprakstu, saprotu, ka kokaudžu
vecumgrupu formula ir šāda:

Kopā piecas grupas:

1.  jaunaudze = (0 ; 1., 2. vecumklase\]
2.  vidēja vecuma audzē = (jaunaudze; briestaudze)
3.  briestaudze = \[ciršanas vecums - 1 vecumklase ; ciršanas vecums)
4.  pieaugušā audze = \[ciršanas vecums + 2 vecumklases\]
5.  pāraugušā audze = (pieaugušā audze; +∞) Tālāk vecumgrupu kodēšanai
    ir izmantots to secīgais numurs (1–5).

Kolonnā “a10” atrodams 1. stāvā reģistrētās pirmās sugas vecums.

``` r
# Veicu vecumgrupu aprēķinu

MVR_Centra <- MVR_Centra %>%
  mutate(
    vecumgrupa = case_when(
      a10 <= 2 * Koku_vecumklase ~ 1,  
      a10 > 2 * Koku_vecumklase & a10 < (galvcirt_vecums - Koku_vecumklase) ~ 2,
      a10 >= (galvcirt_vecums - Koku_vecumklase) & a10 < galvcirt_vecums ~ 3,
      a10 >= galvcirt_vecums & a10 <= (galvcirt_vecums + 2 * Koku_vecumklase) ~ 4,
      a10 > (galvcirt_vecums + 2 * Koku_vecumklase) ~ 5,
      TRUE ~ NA_real_ # visi, kas neatbilst, būs NA
    ))
```

Tagad pievienoju vecumgrupu kā otro ciparu meža klases kodā.

``` r
# Veidoju jaunu kolonnu, kur vecumgrupa ir atspoguļota, ka mežu klases otrais cipars
MVR_Centra$meza_grupa_vecums <- MVR_Centra$vecumgrupa * 10 + MVR_Centra$meza_grupa
```

Veicu vērtību pārbaudi.

``` r
unique(MVR_Centra$meza_grupa_vecums)
```

    ##  [1] 221 223 243 233 244  NA 241 234 251 253 214 211 224 242 213 222 254 231 252
    ## [20] 232 212

Redzu, ka ir `NA` vērtības. Pārbaudīju — daļai audžu nav datu par
šķerslaukumiem. Tādos gadījumos klasifikāciju veikt nevaram. Tāpēc,
veidojot sarakstu ar vērtībām, izslēdzu ierakstus ar `NA`. Saglabāju
visas unikālās vērtības, izņemot `NA`, atsevišķā vektorā. Veidoju tukšu
sarakstu, kurā glabāju izveidotos rastrus.

``` r
meza_grupa_vecums_saraksts <- unique(na.omit(MVR_Centra$meza_grupa_vecums))

# tukšs saraksts rezultātiem
meza_grupa_vecums_rastri <- list()

for(katrs in meza_grupa_vecums_saraksts){
  MVR_Centra_klase <- MVR_Centra %>%  
    filter(meza_grupa_vecums == katrs)
  
  meza_grupa_vecums_rastri[[as.character(katrs)]] <- rasterize(
    MVR_Centra_klase,
    LV10m_10km_apgriezts, 
    field = "meza_grupa_vecums")
}
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

Saglabāšu starprezultātu.

``` r
dir_rastri <- here("6uzd", "Output", "rastri")
dir.create(dir_rastri, recursive = TRUE)

for (katrs in names(meza_grupa_vecums_rastri)) {
  writeRaster(meza_grupa_vecums_rastri[[katrs]], 
              filename = file.path(dir_rastri, paste0(katrs, ".tif")), 
              overwrite = TRUE)
}
```

Vizualizēsim rezultātu.

``` r
test <- rast(here("6uzd", "Output", "rastri", "214.tif"))
plot(test) # Viss darbojas!
```

![](Uzd06_files/figure-gfm/viz_parb-1.png)<!-- -->

# Ceturtais uzdevums

Savienojiet pirmajā un trešajā punktos izveidotos slāņus, tā, lai mazāka
skaitliskā vērtība nozīmē augstāku prioritāti/svaru/dominanci rastra
šūnas vērtības piešķiršanai.

Funkcija `merge()` savieno rastrus dotajā secībā. Tāpēc, lai tos
apvienotu saskaņā ar uzdevuma nosacījumiem, jānorāda secība atbilstoši
koda vērtībām.

``` r
meza_rastri_seciba <- meza_grupa_vecums_rastri[order(names(meza_grupa_vecums_rastri))]
names(meza_rastri_seciba)
```

    ##  [1] "211" "212" "213" "214" "221" "222" "223" "224" "231" "232" "233" "234"
    ## [13] "241" "242" "243" "244" "251" "252" "253" "254"

Veicu rastru savienošanu.

``` r
savienoti_rastri <- Reduce(terra::merge, c(list(LAD_Centra_100_rast), meza_rastri_seciba))
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

Vizualizēsim rezultātu.

``` r
plot(savienoti_rastri)
```

![](Uzd06_files/figure-gfm/viz_rastri-1.png)<!-- -->

Saglabāšu starprezultātu.

``` r
savienoti_rastri_pilns_cels <- file.path(here("6uzd","Output", "rastri"), "savienoti_rastri.tif")
writeRaster(savienoti_rastri, filename = savienoti_rastri_pilns_cels, overwrite = TRUE)
```

# Piektais uzdevums

Cik šūnās ir gan mežaudžu, gan lauku informācija?

Šo var pārbaudīt ar funkciju `ifel()`, kas ir `ifelse()` funkcijas
analogs `SpatRaster` objektiem.

``` r
kopigas_sunas <- ifel(!is.na(LAD_Centra_100_rast) & !is.na(meza_rastri_seciba), 1, NA)
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

Tagad vizualizēsim rezultātu:

``` r
plot(kopigas_sunas)
```

![](Uzd06_files/figure-gfm/viz_parklajas-1.png)<!-- -->

Saskaitīsim, cik šūnās ir gan mežaudžu, gan lauku informācija.

``` r
kopsumma <- sum(!is.na(values(kopigas_sunas)))
cat("Šūnu skaits, kurās ir gan lauku, gan meža informācija, ir", kopsumma, "šūnas.\n")
```

    ## Šūnu skaits, kurās ir gan lauku, gan meža informācija, ir 256371 šūnas.

# Sestais uzdevums

Cik šūnas atrodas Latvijas sauszemes teritorijā, bet nav raksturotas šī
uzdevuma iepriekšējos punktos?

Lai atbildētu uz jautājumu, savienoššu apvienoto rastru ar referenču
rastru, kas aptver visu Latvijas teritoriju.

``` r
LVuzd6rastrs <- merge(savienoti_rastri, LV10m_10km_apgriezts)
```

Vizualizēsim rezultātu:

``` r
plot(LVuzd6rastrs) #Smuki
```

![](Uzd06_files/figure-gfm/6uzd_viz-1.png)<!-- -->

Apskatisīm vērtības:

``` r
LVuzd6rastrs
```

    ## class       : SpatRaster 
    ## size        : 11368, 17831, 1  (nrow, ncol, nlyr)
    ## resolution  : 10, 10  (x, y)
    ## extent      : 407600, 585910, 236380, 350060  (xmin, xmax, ymin, ymax)
    ## coord. ref. : LKS-92 / Latvia TM (EPSG:3059) 
    ## source(s)   : memory
    ## varname     : LV10m_10km 
    ## name        : layer 
    ## min value   :     1 
    ## max value   :   254

Vērtība viens apzīmē šūnas, kas atrodas sauszemes teritorijā (references
rastrs apraksta sauszemes teritoriju), bet kurās nav datu no apvienotā
lauku un mežu rastra. Lai atbildētu uz uzdevumā izvirzīto jautājumu, var
izmantot funkciju `global()` ar argumentu `sum`, kas ļauj aprēķināt
“globālos” statistiskos rādītājus jeb rādītājus visam `SpatRaster`
objektam.

``` r
atbilde <- global(LVuzd6rastrs == 1, "sum", na.rm=TRUE)
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
cat("Šūnu skaits, kurās atrodas Latvijas teritorijā, bet kuras nav raksturotas apvienotājā lauku un meža slānī, ir", atbilde[[1]], "šūnas.\n")
```

    ## Šūnu skaits, kurās atrodas Latvijas teritorijā, bet kuras nav raksturotas apvienotājā lauku un meža slānī, ir 127661931 šūnas.
