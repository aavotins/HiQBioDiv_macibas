Vienpadsmitais uzdevums: sugai specifiski ekoģeogrāfiskie mainīgie
================
Jekaterīna Butkeviča
2025. gads 19. augusts

Man piešķirtais EGV ir *atkusnis* jeb atkušņa (diena ar pozitīvu vidējo
temperatūru pēc dienas ar negatīvu minimālo temperatūru; ja vienā dienā
ir pozitīva vidējā un negatīva minimālā, tā tiek pieskaitīta atkusnim)
dienu skaits, vidēji gadā.

Kopumā uzdevumu saprotu šādi: 1. Veikt nepieciešamos aprēķinus GEE
pārlūkā; 2. Nodrošināt sakritību ar referenci; 3. Tikt galā ar tukšumiem
rastrā; 4. Mikstināt oriģinālo pikseļu robežas; 5. Veikt rastra
centrēšanu un mērogošanu (pārveidot z-scores).

# Pirmais uzdevums

Pirmā uzdevuma daļa ir paredzēta izpildei Google Earth Engine pārlūkā,
izmantojot JavaScript valodu. Komandrindas ir pieejamas
[šeit](https://code.earthengine.google.com/6f1962047686975c751ea086498c4356).
Tālāk tiek parādīts kods atlikušajai sadaļai, kas pildīta R valodā.

# Otrais uzdevums

## Nepieciešamās pakotnes

Lejupielādējam visas nepieciešamās pakotnes.

``` r
if (!require("here")) install.packages("here"); library(here)
if (!require("terra")) install.packages("terra"); library(terra)
if (!require("whitebox")) install.packages("whitebox"); library(whitebox)
```

## Datu ielāde un atbilstība referencei

Lejupielādējam atkušņa rastru no Google Drive (to izdarīju manuāli), kā
arī referenču rastru. Pārbaudām, vai sakrīt koordinātu sistēmas —
sakrīt.

``` r
reference_100m <- rast(here("3uzd", "Data", "references_zenodo", "rastrs", "LV100m_10km.tif"))
crs(reference_100m) #"LKS-92 / Latvia TM\"
```

    ## [1] "PROJCRS[\"LKS-92 / Latvia TM\",\n    BASEGEOGCRS[\"LKS-92\",\n        DATUM[\"Latvian geodetic coordinate system 1992\",\n            ELLIPSOID[\"GRS 1980\",6378137,298.257222101,\n                LENGTHUNIT[\"metre\",1]]],\n        PRIMEM[\"Greenwich\",0,\n            ANGLEUNIT[\"degree\",0.0174532925199433]],\n        ID[\"EPSG\",4661]],\n    CONVERSION[\"Latvian Transverse Mercator\",\n        METHOD[\"Transverse Mercator\",\n            ID[\"EPSG\",9807]],\n        PARAMETER[\"Latitude of natural origin\",0,\n            ANGLEUNIT[\"degree\",0.0174532925199433],\n            ID[\"EPSG\",8801]],\n        PARAMETER[\"Longitude of natural origin\",24,\n            ANGLEUNIT[\"degree\",0.0174532925199433],\n            ID[\"EPSG\",8802]],\n        PARAMETER[\"Scale factor at natural origin\",0.9996,\n            SCALEUNIT[\"unity\",1],\n            ID[\"EPSG\",8805]],\n        PARAMETER[\"False easting\",500000,\n            LENGTHUNIT[\"metre\",1],\n            ID[\"EPSG\",8806]],\n        PARAMETER[\"False northing\",-6000000,\n            LENGTHUNIT[\"metre\",1],\n            ID[\"EPSG\",8807]]],\n    CS[Cartesian,2],\n        AXIS[\"northing (X)\",north,\n            ORDER[1],\n            LENGTHUNIT[\"metre\",1]],\n        AXIS[\"easting (Y)\",east,\n            ORDER[2],\n            LENGTHUNIT[\"metre\",1]],\n    USAGE[\n        SCOPE[\"Engineering survey, topographic mapping.\"],\n        AREA[\"Latvia - onshore and offshore.\"],\n        BBOX[55.67,19.06,58.09,28.24]],\n    ID[\"EPSG\",3059]]"

``` r
atkusnis_GEE <- rast(here("11uzd", "Data", "average_thaw_days_2016_2024.tif"))
crs(atkusnis_GEE) #"LKS-92 / Latvia TM\"
```

    ## [1] "PROJCRS[\"LKS-92 / Latvia TM\",\n    BASEGEOGCRS[\"LKS-92\",\n        DATUM[\"Latvian geodetic coordinate system 1992\",\n            ELLIPSOID[\"GRS 1980\",6378137,298.257222101,\n                LENGTHUNIT[\"metre\",1]]],\n        PRIMEM[\"Greenwich\",0,\n            ANGLEUNIT[\"degree\",0.0174532925199433]],\n        ID[\"EPSG\",4661]],\n    CONVERSION[\"Latvian Transverse Mercator\",\n        METHOD[\"Transverse Mercator\",\n            ID[\"EPSG\",9807]],\n        PARAMETER[\"Latitude of natural origin\",0,\n            ANGLEUNIT[\"degree\",0.0174532925199433],\n            ID[\"EPSG\",8801]],\n        PARAMETER[\"Longitude of natural origin\",24,\n            ANGLEUNIT[\"degree\",0.0174532925199433],\n            ID[\"EPSG\",8802]],\n        PARAMETER[\"Scale factor at natural origin\",0.9996,\n            SCALEUNIT[\"unity\",1],\n            ID[\"EPSG\",8805]],\n        PARAMETER[\"False easting\",500000,\n            LENGTHUNIT[\"metre\",1],\n            ID[\"EPSG\",8806]],\n        PARAMETER[\"False northing\",-6000000,\n            LENGTHUNIT[\"metre\",1],\n            ID[\"EPSG\",8807]]],\n    CS[Cartesian,2],\n        AXIS[\"northing (X)\",north,\n            ORDER[1],\n            LENGTHUNIT[\"metre\",1]],\n        AXIS[\"easting (Y)\",east,\n            ORDER[2],\n            LENGTHUNIT[\"metre\",1]],\n    USAGE[\n        SCOPE[\"Engineering survey, topographic mapping.\"],\n        AREA[\"Latvia - onshore and offshore.\"],\n        BBOX[55.67,19.06,58.09,28.24]],\n    ID[\"EPSG\",3059]]"

Ar funkciju `resample()` nodrošinām atkušņa rastra atbilstību references
rastram.

``` r
atkusnis_resample <- resample(atkusnis_GEE, 
                              reference_100m, 
                              method = "bilinear", 
                              filename = here("11uzd", "Output", "atkusnis.tif"), 
                              overwrite = TRUE) # sakritībai
```

# Trešais uzdevums

Paskatoties uz iegūto rastru, redzu, ka tajā gar jūru ir palikuši
tukšumi. Tukšumi pie jūras rodas datu maskēšanas dēļ, jo ERA5-Land datu
komplekts satur tikai sauszemes datus. Pikseļi ūdens objektiem,
piemēram, Baltijas jūrai, tiek aizpildīti ar `NA` vērtībām, jo dati tiem
netiek nodrošināti. Tā kā rastra izšķirtspēja ir 100 m, šie tukšumi ir
ļoti redzami.

Pieeju šīs problēmas risināšanai paņēmu no Betijas koda, bet, lai
saglabātu savu ieguldījumu, cenšos to izskaidrot.

No sākuma tiek aprēķināts attālums no katra NA pikseļa līdz tuvākajam
pikselim ar vērtību, lai noteiktu spraugu lielumu rastra datos.

``` r
tuksumi <- is.na(atkusnis_resample) & !is.na(reference_100m)
tuksumi_count <- global(tuksumi, fun="sum", na.rm=TRUE)
tuksumi_count
```

    ##                 sum
    ## avg_thaw_days 70118

``` r
aizpilditie <- ifel(!tuksumi, 1, NA)
platums <- distance(aizpilditie)
plot(platums, main = "Attālums līdz tuvākajai ne-NA vērtībai")
```

![](KlimataEGV_Butkevica_files/figure-gfm/tuksumi-1.png)<!-- -->

``` r
max_attalums <- terra::global(platums, fun = "max", na.rm = TRUE)
print(max_attalums) # lielākais platums ir 3551
```

    ##                    max
    ## avg_thaw_days 3551.056

Tā kā iegūtais skaits sakrita, secināju, ka tas patiešām ir ERA5-Land
datu nodrošinājuma problēma un nav atkarīgs no mūsu aprēķiniem.

Problēmu risināšanai tika izmantota funkcija `wbt_fill_missing_data()`
no jau daudzos aprakstos pieminētās pakotnes `whitebox`. Tā izmanto
svērto attāluma pieeju, lai aprēķinātu vērtības tukšajām šūnām,
balstoties uz to ne-tukšajām kaimiņšūnām. Katram tukšajam pikselim tiek
meklēti derīgie kaimiņi noteiktā radiusa zonā, aprēķināts katra kaimiņa
svars atkarībā no attāluma (tuvākie kaimiņi iegūst lielāku svaru) un pēc
tam pikselim piešķirta svaroto kaimiņu vidējā vērtība, tādējādi
aizpildot tukšumus. Arguments `filter` nosaka zonu radiusu rastra
pikseļos.

Betija izmantoja vērtību 72. Lai iegūtu šo vērtību, tika ņemts
iepriekšējā koda blokā aprēķinātais attālums metros, un, ņemot vērā
rastra izšķirtspēju, tas tika dalīts ar 100 un reizināts ar 2.
Reizināšana nepieciešama tāpēc, ka filtrs darbojas no centra pikseļa uz
malas, tāpēc, lai segtu visu attālumu līdz kaimiņiem abās pusēs, 36
pikseļus jāreizina ar 2.

Bet tā kā, izmantojot augstāk sniegtās komandas, es vēlreiz pārbaudīju
tukšumu klātbūtni, un nelielā daļā no tiem tomēr palika, es savā versijā
liku vērtību 110, jo tikai ar to izdevās pilnībā novērst tukšumus.

``` r
wbt_fill_missing_data(
  i = here("11uzd", "Data", "atkusnis.tif"),
  output = here("11uzd", "Output", "atkusnis_filled.tif"),
  filter = 110, # platumu aprēķina rezultāts    
  weight = 1,
  no_edges = FALSE # lai notiktu ekstrapolācija gar malām
)

result <- rast(here("11uzd", "Output", "atkusnis_filled.tif"))
```

# Ceturtais uzdevums

Lai padarītu oriģinālo pikseļu “mālas” mazāk redzamas, var izmantot
funkciju `focal(.., fun = "mean")`. Tā veic “pārvietojamā loga” aprēķinu
rastrā, pārvietojot noteikta izmēra kvadrātu (parametrs w) pāri katram
rastra pikselim. Katram logam tiek aprēķināta statistika visiem logā
esošajiem pikseļiem, un iegūtā vērtība tiek piešķirta loga centrālajam
pikselim jaunā rastrā. Kvadrāta izmērs nosaka, cik gluds būs rezultāts.
Es izvēlējos vērtību 81, jo, apskatot dažādus variantus, šis šķita
vispiemērotākais, taču piekrītu Betijai, ka šeit ir jādefinē kopīga
pieeja.

``` r
smooth_atkusnis <- focal(result, w=81, fun = "mean") # Izmēģināju dažādus varinatus, no visiem, šī w vērība man partika visvairāk.
plot(smooth_atkusnis)
```

![](KlimataEGV_Butkevica_files/figure-gfm/pikselu_malu_izlidzinasana-1.png)<!-- -->

Tāda veida tika iegūts rastrs ar izlīdzinātām vērtībām un aizpildītiem
tukšumiem. To tagad ir jāapgriež, lai izvairītos no vērtībām ārpus
Latvijas teritorijas.

``` r
dir.create(here("11uzd", "EGV"))

atkusnis_raw <- mask(smooth_atkusnis, reference_100m, filename = here("11uzd", "EGV", "Climate_ThawDays_cell_RAW.tif"), overwrite=TRUE)
```

# Piektais uzdevums

Tagad rastru vajag pārvērst z-scores. To ir ērti izdarīt ar
`terra::scale`.

``` r
atkusnis_merogots <- terra::scale(atkusnis_raw, center = TRUE, scale = TRUE)
writeRaster(atkusnis_merogots, filename = here("11uzd", "EGV", "Climate_ThawDays_cell_scaled.tif"), overwrite=TRUE)
```
