Uzd05_Starka
================
Rūta Starka
2025-02-07

### Par sistēmas laika mērījumiem

Vēlos informēt, ka lai gan uzdevumā bija norādīts, kur jāsāk mērīt
sistēmas laiks un kur jābeidz, bet es ļoti vēlējos pa starpu apskatīt
kartes tuvāk visādos veidos, tāpēc mērīju laikus tieši pašām funkcijām,
nevis visa uzdevuma veikšanai kopā. Citādi man būtu jāmet ārā ggploti,
bet es gribēju viņus atstāt pati priekš sevis vizuālam priekšstatam.
Tāpēc šis uzdevuma risinājums ir garš, bet manuprāt vizuāli informatīvs.
Secinājumos ir tabula, kur šie laiki ir apkopoti.

# 1. Sagatavošanās

Palaižu visas uzdevuma veikšanai nepieciešamās pakotnes.

Ielasu uzdevuma veikšanai neipeciešamos references vektordatus un
apskatu tos.

``` r
tks93_50km=st_read_parquet("D:/Users/RutaStarka/Desktop/Git local Ruuta/HiQBioDiv_macibas/Uzd03/ref_vektordati/tks93_50km.parquet")
plot(tks93_50km)
```

![](Uzd05_Starka_files/figure-gfm/ref.vektordati-1.png)<!-- -->

Atlasu četrus blakus esošus kvadrātus. Ielasu kā kopīgu 4 kvadrātu
objektu, un kā atsevišķus neatkarīgus objektus pa vienam kvadrātam.

``` r
atlase <- tks93_50km %>% 
  filter(NOSAUKUMS %in% c("Rīga", "Jūrmala", "Jelgava", "Baldone"))
rm(tks93_50km)#noņemu pārējo no vides
plot(atlase)
```

![](Uzd05_Starka_files/figure-gfm/atlase-1.png)<!-- --> \## 1.1. Spatial
join MVR atlasei Tagad, kad zinu ar kurām kartes lapām gribu strādāt,
sasaistīšu MVR datus ar tām. Sākumā jāpārbauda koordinātu sistēmas. Tās
ir vienādas, bet par cik vienai ir klāt /Latvia Tm, tad tālākās
funkcijas tās kā tādas neatpazina. Tāpēc it jāvineādo, ko arī izdaru.Un
pēc tam saglabāju kā geoparquet, lai var ērtāk strādāt tālāk, un ielasu
atpakaļ.

Tagad apvienošu šo slāņu atribūtus un apskatīšu, kas iznācis. Uzriez jau
pie jaunā objekta izveidošanas redzu, ka ir palielinājies atribūtu
skaits, kas ir uz objekta “atlase” rēķina, kam vienīgais kopīgais lauks
bija “Shape” (multipolygon).

``` r
join_laiks=system.time({
  MVR_atlase=sf::st_join(MVR_centrs,atlase, left =FALSE, join=st_intersects)
})
print(join_laiks)
```

    ##    user  system elapsed 
    ##   19.53    1.13   21.41

Apvienošana prasa 15-16 sekundes (kā kuru reizi). Pirms apvienošanas
MVR_centrs ir 505660 rindas. Pēc apvienošanas - 85523 rindas. Papildus
ir 4 jauni atribūtu lauki, kas mantoti no objekta “atlase”.

Izveidošu atsevišķas atlases no šī apvienotā objekta, izdalot atsevišķas
kartes lapas, lai varētu pēc tam paskatīties, kas notiek to robežošanās
vietās. Pie reizes arī šīs atlases saglabāšu kā atsevišķus geoparquet
failus, varbūt vēlāk noderēs.

``` r
join_atlase=system.time({
MVR_jelgava=MVR_atlase %>% filter(NOSAUKUMS=="Jelgava")
MVR_baldone=MVR_atlase %>% filter(NOSAUKUMS=="Baldone")
MVR_riga=MVR_atlase %>% filter(NOSAUKUMS=="Rīga")
MVR_jurmala=MVR_atlase %>% filter(NOSAUKUMS=="Jūrmala")

st_write_parquet(MVR_jelgava, dsn="MVR_jelgava.parquet")
st_write_parquet(MVR_baldone, dsn="MVR_baldone.parquet")
st_write_parquet(MVR_riga, dsn="MVR_riga.parquet")
st_write_parquet(MVR_jurmala, dsn="MVR_jurmala.parquet")
})
```

Katrā kartes lapā ir sekojošs skaits rindu: Jelgava - 22807 Baldone -
33449 Rīga - 16705 Jūrmala - 12562

Saskaitot kopā ir tāds pats skaits, kā kopējai kartei (objekts
“atlase”), kas pēc manas loģikas nozīmē, ka viens poligons pieder tikai
vienai lapai. Būs jāpārliecinās, vai tā ir.

Apskatīšu vizuāli, kā sanāca.

``` r
ggplot(MVR_atlase) +
  geom_sf(aes(color = NOSAUKUMS), size = 1) +  
  theme_minimal() +
  labs(color = "NOSAUKUMS")
```

![](Uzd05_Starka_files/figure-gfm/view.spat.join-1.png)<!-- --> Redzu,
ka visi poligoni, kas ir uz karšu lapu robežām ir iekļauti. Bet
interesē, kurai kartes lapai pieder tie poligoni, kas ir uz to
savstarpējām robežām.

### Kas notiek uz karšu lapu savstarpējām robežām?

Lai apskatītu katru kartes lapu pa vienai, izmantošu iepriekš izveidotās
atlases un katrai izveidošu savu karti, uzmanību pievēršot poligoniem,
kas atrodas uz robežām. P.S. tīšprāt neizmantoju facetwrap, kas ļautu ar
vienu ggplot visu atspoguļot, jo gribu paskatīties tuvāk un pareizo
novietojumu varu noteikt pati (ar facetwrap sanāk katru kartes lapu
ielikt “savā vietā” no kopējā extent, bet tad ir daudz tukšu laukumu, un
tas nav informatīvi tam ko gribu redzēt).

``` r
jur=ggplot(MVR_jurmala) +
  geom_sf(aes(), size = 0.2) +  
  theme_minimal() 

rig=ggplot(MVR_riga) +
  geom_sf(aes(), size = 0.2) +  
  theme_minimal() 

jel=ggplot(MVR_jelgava) +
  geom_sf(aes(), size = 0.2 )+  
  theme_minimal() 

bal=ggplot(MVR_baldone) +
  geom_sf(aes(), size = 0.2) +  
  theme_minimal() 

ggarrange(jur, rig, jel, bal,
    labels = c("Jūrmala", "Rīga", "Jelgava", "Baldone"),
    font.label = list(size = 8, face = "bold"),  
    ncol = 2, nrow = 2)
```

![](Uzd05_Starka_files/figure-gfm/view.lapas-1.png)<!-- -->

Salīdzinot, piemēram, Jūrmalas lapas apakšējo robežu un Jelgavas lapas
augšējo robežu redzams, ka poligoni, kas pārklājās starp šīm abām karšu
lapām pieder gan vienai, gan otrai lapai. Tas mani mulsina, jo tīri
rindu skaita ziņā tam tā nevajadzētu būt, izņemot, ja katrs poligons,
kas atrodas uz karšu lapu savstarpējām robežām, netiek dublēts st_join()
funkcijas laikā. Palasu, un jā, tā tas ir un tā tam jābūt.  
Tas nozīmē, ka apstrādājot šīs lapas atsevišķi, un pēc tam līmējot tās
kopā, būs nodrošināta vizuāla nepārtrauktība. Katrā ziņā vajadzētu jau
būt tā, ka rastru pikseļu vērtības tāpat ir iegūtas tādas pašas, tāpēc
tam nevajadzētu radīt lielu satraukumu, bet tomēr.

Novācu sastrādātos objektus.

``` r
rm(jur,rig,jel,bal)
```

## 1.2. Mana funkcija karšu lapās

Tagad jāregiģē “Mana funkcija” no 4. uzdevuma tā, lai rezultējošais .tif
faila nosaukums satur arī kartes lapas nosaukumu. Tāpat es izņemšu no
funkcijas geoparquet ielasīšanas funkciju, un tā vietā ielikšu atsauci
uz objektu input, ko kontrolēšu ārpus funkcijas (šajā gadījumā viss
atlases fails).

``` r
mana_funkcija=function(input_files){
  MVR_centrs = st_read_parquet(input_files)

  ref_rastrs10m=rast("../Uzd03/ref_rastrs/LV10m_10km.tif")
  ref_rastrs100m=rast("../Uzd03/ref_rastrs/LV100m_10km.tif")
  
  ref_rastrs10m_centrs = terra::crop(ref_rastrs10m,MVR_centrs)
  ref_rastrs100m_centrs = terra::crop(ref_rastrs100m,MVR_centrs)
  
  ref_rastrs10m_centrs2=raster::raster(ref_rastrs10m_centrs)
  
  priedes=MVR_centrs%>%filter(s10 == 1)

  priedes_rastrs10m=fasterize(priedes,ref_rastrs10m_centrs2,background=0)

  priedes2_rastrs10m = rast(priedes_rastrs10m)

  priedes2_rastrs10m[is.na(ref_rastrs10m_centrs)] = NA
  
  output_dir = "../Uzd05/"
  
  filename_prefix <- MVR_centrs %>%
    pull(NOSAUKUMS) %>%
    unique()%>% .[1]
  
  output_filename = paste0(output_dir, filename_prefix, "_1_2_priedes.tif")
  
  priedes100=resample(priedes2_rastrs10m,ref_rastrs100m_centrs,
                      method="average",
                      filename=output_filename,
                      overwrite=TRUE)
}
```

Nu, tad paskatīšos, kas sanāk un pie reizes pamērīšu cik ilgi tas iet.
Atļaušos uzreiz izmantot divus kodolus.

``` r
input_files=c("MVR_jurmala.parquet",
  "MVR_riga.parquet",
  "MVR_baldone.parquet",
  "MVR_jelgava.parquet")

# Kodolu klasera definēšana
cl2 <- makeCluster(2)
registerDoParallel(cl2)

# Paralēla funkcijas palaišana

funkcijas_laiks1=system.time({
  foreach(i = seq_along(input_files), .packages = c("terra", "dplyr","sfarrow","fasterize"), .export="mana_funkcija") %dopar% {
    file=input_files[i]
    mana_funkcija(file)
  }
})
```

    ## Warning in e$fun(obj, substitute(ex), parent.frame(), e$data): already
    ## exporting variable(s): mana_funkcija

``` r
stopCluster(cl2)

print(funkcijas_laiks1)
```

    ##    user  system elapsed 
    ##    4.20    0.55   40.69

Funkcija prasīja tikai 31 sekundi. Faili eksportējās, tie satur kartes
lapas nosaukumu. Apskatot šos failus secinu, ka jā, informācija no malām
nav pazudusi - katra lapa satur kartes extent kā arī “maliņu”, kuras
platumu nosaka tas, cik tālu ārā no kartes lapas ir “izlīdis” kāds
poligons. Tas man liek domāt, ka laikam virtuālā rastra veidošanas
variants izkritīs, jo malas vairs nav pilnīgi sakritīgas starp lapām.

## 1.3. Rastru apvienošana

Tagad interesantākais, jāapvieno šīs četras kartes vienā. Pamēģināšu ar
virtuālo rastru. Lai viss būtu smuki, vajag sākumā panākt, lai visiem
četriem rastriem sakristu malas. Kā iepriekš norādīju, šobrīd katra
rastra extent ir atkarīgs no robežu poligoniem. Tad ir divas pieejas: 1)
Vai nu sākumā apgriežu šos rastrus līdz katras kartes lapas extent (bet
ja 4 kartes lapām šāda pieeja ir ok, likt viens pret viens kartes lapas
rastru pret kartes lapas extent, tad lielākam darba apjomam, piemēram,
visai valstij, es negribētu ar to noņemties). Protams, šo varētu kaut kā
optimizēt un gudri salikt, bet interesantāks, ko pamēģināt šī uzdevuma
kontekstā, man šķiet nākamais variants. 2) Izmantot mosaic funkciju, kas
nosaka, kas notiek ar rastra pikseļiem, kas pārklājas. Par cik iegūtie
rastri ir pakļauti visi vienai funkcijai (mana_funkcija), visas iegūtās
vērtības pikseļos ir tādas pašas katrā punktā (būtu jāuzmanās, ja
“mana_funkcija” ietvertu kādu darbību, kas ietvertu buferus vai ko tādu,
kur katra pikseļa vērtību ietekmē kāds blakus pikselis). Tas nozīmē, ka
šinī brīdī nav nozīmes, kādu funkciju izmanto mozaīkas izveidei
(maksimālās, vidējās, minimālās…), jo tās katrā punktā sakrīt starp
kartes lapām.

Veidošu mozaīku. Izdomāju, ka uztaisīšu rastru apvienošanas funkciju,
kas paņem rastru sarakstu, ielasa tos pa vienam, un ar mosaic() tos
salīmē, un saglabā rezultātu direktorijā. Tad citās reizēs teorētiski
varētu tikai izveidot sarakstu ārpus funkcijas (varētu arī iekšā) un
palaist apvienošanu.

``` r
#atrodu un izveidoju sarakstu ar četriem iepriekšējā apakšuzdevumā izveidotajiem rastriem.
output_dir <- "../Uzd05/"
rastru_saraksts <- list.files(output_dir, pattern = "_1_2_priedes.tif$", full.names = TRUE)
print(rastru_saraksts)# apskatu failu nosaukumus, tur viss ok, ir visi četri rastri.
```

    ## [1] "../Uzd05/Baldone_1_2_priedes.tif" "../Uzd05/Jelgava_1_2_priedes.tif"
    ## [3] "../Uzd05/Jūrmala_1_2_priedes.tif" "../Uzd05/Rīga_1_2_priedes.tif"

``` r
apvienosanas_funkcija=function(rastru_saraksts){
    #ielasu visus rastrus ar rast()
    rastru_objekti <- lapply(rastru_saraksts, rast)
    
    #pakļauju katru no saraksta rastriem funkcijai mosaic(), ar funkciju "max", jeb, ka tiek paņemta     maksimālā pikseļa vērtība (bet ja ņemtu "min", tam šajā gadījumā nevajadzētu mainīt rezultātu)
    atlases_Mozaika <- do.call(mosaic, c(rastru_objekti, list(fun = "max")))

    # Saglabāju apvienoto rastru mozaīku
    output_path <- "../Uzd05/1_3_atlasesMozaika.tif"
    writeRaster(atlases_Mozaika, output_path, overwrite = TRUE)
  }

#palaižu šo funkciju un izmēru, cik laika tā prasa. 
apvienosanas_laiks1=system.time({
  apvienosanas_funkcija(rastru_saraksts)
})

print(apvienosanas_laiks1)
```

    ##    user  system elapsed 
    ##    0.19    0.07    0.25

Funkcija ir gana ātra. Domāju, ka ja būtu daudz rastru ko apvienot, tas
būtu pieņemams ātrums.

Apskatīšu vizuāli, kas iznāca.

``` r
mozaika1_3=rast("../Uzd05/1_3_atlasesMozaika.tif")
plot(mozaika1_3, main="Spatial join")
```

![](Uzd05_Starka_files/figure-gfm/mozaikas.apskate-1.png)<!-- --> Ir
labās un relatīvi sliktās ziņas. Labās ziņas - ir iegūts nepieciešamais
rezultāts (priežu īpatsvars vērtībās no 0-1, kā to paredz funkcija, jūra
ir NA, vietas, kur nav priežu ir 0), iekšējās lapu robežas ir smuki
salikušās. sliktās ziņas - ārējā robeža. Vizuāli uz pirmo acs uzmetienu
tā izskatās pēc nobīdes, tomēr tā nav. Lapu malas vienkārši nav
apgrieztas - ir saglabājušies objekti ārpus malām. Tas arī bija
sagaidāms, jo mosaic() neko negriež, tikai pārklāj ar nosacījumiem. Par
cik ārējās malās nekā nav, tad šī informācija paliek.

# 2. Clipping izmantošana

Tagad jāpamēģina viss tas pats, bet nevis ar spatial_join() bet ar
clipping. Man jau ir objekts “atlase”, kas satur informāciju par četru
karšu lapu robežām.

## 2.1. Clipping MVR atlasei

Kā clipping funkciju izvēlos intersection, jo man jāiegūst poligoni, kas
pārklājas ar kartes lapu laukumu.

``` r
clip_laiks=system.time({
  MVR_atlase2=sf::st_intersection(MVR_centrs, atlase)
})
print(clip_laiks)
```

    ##    user  system elapsed 
    ##   23.48    0.67   24.35

Šī funkcija ir gandrīz divreiz lēnāka par spatial join. Vairāk
secinājumos.

Līdzīgi kā iepriekš, izveidošu atsevišķas atlases no šī apvienotā
objekta, izdalot atsevišķas kartes lapas, lai varētu pēc paskatīties,
kas notiek to robežošanās vietās un cik objektu ir katrā lapā. Pie
reizes arī šīs atlases saglabāšu kā atsevišķus geoparquet failus.

``` r
clip_atlase=system.time({
MVR_jelgava2=MVR_atlase2 %>% filter(NOSAUKUMS=="Jelgava")
MVR_baldone2=MVR_atlase2 %>% filter(NOSAUKUMS=="Baldone")
MVR_riga2=MVR_atlase2 %>% filter(NOSAUKUMS=="Rīga")
MVR_jurmala2=MVR_atlase2 %>% filter(NOSAUKUMS=="Jūrmala")

st_write_parquet(MVR_jelgava2, dsn="MVR_jelgava2.parquet")
st_write_parquet(MVR_baldone2, dsn="MVR_baldone2.parquet")
st_write_parquet(MVR_riga2, dsn="MVR_riga2.parquet")
st_write_parquet(MVR_jurmala2, dsn="MVR_jurmala2.parquet")
})
```

Katrā kartes lapā ir tāds pats objektu skaits, kā izmantojot spatial
join.

Apskatīšu vizuāli, kādas ir atšķirības.

``` r
ggplot(MVR_atlase2) +
  geom_sf(aes(color = NOSAUKUMS), size = 1) +  
  theme_minimal() +
  labs(color = "NOSAUKUMS")
```

![](Uzd05_Starka_files/figure-gfm/view.clipping-1.png)<!-- -->

Uzreiz ir redzamas skaidras atšķirības - katrs poligons ir nogriezts
līdz ar kartes lapas robežu. Tas nozīmē, ka nebūs nepieciešams izmantot
funkciju mosaic, jo nebūs pikseļu pārklāšanās liekot rastrus kopā.
Paskatīšu katru kartes lapu atstatus vienu no otras, kā iepriekš, lai
pārliecinātos.

``` r
jur=ggplot(MVR_jurmala2) +
  geom_sf(aes(), size = 0.2) +  
  theme_minimal() 

rig=ggplot(MVR_riga2) +
  geom_sf(aes(), size = 0.2) +  
  theme_minimal() 

jel=ggplot(MVR_jelgava2) +
  geom_sf(aes(), size = 0.2 )+  
  theme_minimal() 

bal=ggplot(MVR_baldone2) +
  geom_sf(aes(), size = 0.2) +  
  theme_minimal() 

ggarrange(jur, rig, jel, bal,
    labels = c("Jūrmala", "Rīga", "Jelgava", "Baldone"),
    font.label = list(size = 8, face = "bold"),  
    label.y = c(1, 1, 1.1, 1.1),
    ncol = 2, nrow = 2)
```

![](Uzd05_Starka_files/figure-gfm/view.lapas2-1.png)<!-- --> Novācu
sastrādātos objektus.

``` r
rm(jur,rig,jel,bal)
```

## 2.2. Clipping un mana funkcija

Sākumā tomēr man ir pavisam nedaudz jāmodificē funkcija, lai nosaukumā
iekļautu arī uzdevuma apakšnumuru:

``` r
mana_funkcija=function(input_files){
  MVR_centrs = st_read_parquet(input_files)

  ref_rastrs10m=rast("../Uzd03/ref_rastrs/LV10m_10km.tif")
  ref_rastrs100m=rast("../Uzd03/ref_rastrs/LV100m_10km.tif")
  
  ref_rastrs10m_centrs = terra::crop(ref_rastrs10m,MVR_centrs)
  ref_rastrs100m_centrs = terra::crop(ref_rastrs100m,MVR_centrs)
  
  ref_rastrs10m_centrs2=raster::raster(ref_rastrs10m_centrs)
  
  priedes=MVR_centrs%>%filter(s10 == 1)

  priedes_rastrs10m=fasterize(priedes,ref_rastrs10m_centrs2,background=0)

  priedes2_rastrs10m = rast(priedes_rastrs10m)

  priedes2_rastrs10m[is.na(ref_rastrs10m_centrs)] = NA
  
  output_dir = "../Uzd05/"
  
  filename_prefix <- MVR_centrs %>%
    pull(NOSAUKUMS) %>%
    unique()%>% .[1]
  
  output_filename = paste0(output_dir, filename_prefix, "_2_2_priedes.tif")
  
  priedes100=resample(priedes2_rastrs10m,ref_rastrs100m_centrs,
                      method="average",
                      filename=output_filename,
                      overwrite=TRUE)
}
```

Palaižu funkciju:

``` r
input_files=c("MVR_jurmala2.parquet",
  "MVR_riga2.parquet",
  "MVR_baldone2.parquet",
  "MVR_jelgava2.parquet")

# Kodolu klasera definēšana
cl2 <- makeCluster(2)
registerDoParallel(cl2)

# Paralēla funkcijas palaišana

funkcijas_laiks2=system.time({
  foreach(i = seq_along(input_files), .packages = c("terra", "dplyr","sfarrow","fasterize"), .export="mana_funkcija") %dopar% {
    file=input_files[i]
    mana_funkcija(file)
  }
})
```

    ## Warning in e$fun(obj, substitute(ex), parent.frame(), e$data): already
    ## exporting variable(s): mana_funkcija

``` r
stopCluster(cl2)

print(funkcijas_laiks2)
```

    ##    user  system elapsed 
    ##    4.23    0.26   40.67

Funkcijas laiks ir līdzīgs kā iepriekš, ap 30 sekundēm. Apskatot failus
folderī, redzu atšķirības - vairs nav objektu ārpus kartes lapas
robežām.

## 2.3. Rastru apvienošana pēc Clipping

Tagad ir jāapvieno. Varētu to darīt ar to pašu mosaic(), jo izveidoju
funkciju, un tad varēšu tieši salīdzināt rezultātus.

P.S. Varētu arī taisīt virtuālo rastru, bet mani izbrīnīja, ka lapu
extent nesakrīt, kam tā nevajadzētu būt…

``` r
#atlasu ar clipping uztaisītos rastrus
dir="../Uzd05/"
rastri2_2 <- list.files(dir, pattern = "\\_2_2_priedes.tif$", full.names = TRUE)
print(rastri2_2)#pārbaudu, vai pareizi atlasījās
```

    ## [1] "../Uzd05/Baldone_2_2_priedes.tif" "../Uzd05/Jelgava_2_2_priedes.tif"
    ## [3] "../Uzd05/Jūrmala_2_2_priedes.tif" "../Uzd05/Rīga_2_2_priedes.tif"

``` r
#rediģēju funkcijā nosaukumu apvienotajam rastram (mozaīkai)
apvienosanas_funkcija2_3=function(rastri2_2){
    #ielasu visus rastrus ar rast()
    rastru_objekti <- lapply(rastri2_2, rast)
    
    #pakļauju katru no saraksta rastriem funkcijai mosaic()
    atlases_Mozaika <- do.call(mosaic, c(rastru_objekti, list(fun = "max")))

    # Saglabāju apvienoto rastru mozaīku
    output_path <- "../Uzd05/2_3_atlasesMozaika.tif"
    writeRaster(atlases_Mozaika, output_path, overwrite = TRUE)
  }


#palaižu šo funkciju un izmēru, cik laika tā prasa. 
apvienosanas_laiks2=system.time({
  apvienosanas_funkcija2_3(rastri2_2)
})

print(apvienosanas_laiks2)
```

    ##    user  system elapsed 
    ##    0.23    0.03    0.28

Tas prasīja tikpat neilgi, cik iepriekš.

Apskatīšu kā sanāca, bet blakus pielikšu arī rezultātu no spatial join.

``` r
mozaika1_3=rast("../Uzd05/1_3_atlasesMozaika.tif")
mozaika2_3=rast("../Uzd05/2_3_atlasesMozaika.tif")

par(mfrow = c(1, 2))

plot(mozaika1_3, main = "Spatial join")
plot(mozaika2_3, main = "Clipping")
```

![](Uzd05_Starka_files/figure-gfm/mozaiku.salidzinasana-1.png)<!-- -->

Nu, izskatās diezgan labi. Rastru lapas ir smuki salīmētas kopā, nav
redzamas greizas to nesakritības. Nav objektu ārpus kartes malām. Uz
pirmo acs uzmetienu sākumā domāju, ka šis varētu būt optimāli, vienīgi
es redzu nelielu atšķirību pie Mangaļsalas mola, kas man liek domāt, ka
ir kaut kāda problēma, bet es nespēju identificēt kur tā radās.
Teorētiski tieši pie salīmēšanas, bet izbruacu cauri abām kartēm, vismaz
vizuāli neredzu šādas nobīdes uz leju no aizdomīgās vietas.Par cik šis
te robiņš, par kuru runāju, attiecas uz 0 un NA vērtībām, šoreiz laidīšu
to vaļā, bet vispār tam būtu jāpievērš uzmanība.

Atiestatu grafiku izvietošanu.un novācu objektus.

``` r
par(mfrow = c(1, 1))
rm(mozaika1_3,mozaika2_3)
```

# 3. Spatial filtering

Tagad atkārtošu to pašu spatial join / clipping vietā izmantojot spatial
filtering.

## 3.1. Spatial filtering MVR atlasei

Izmantošu papildargumentu predictate=st_intersects, lai rezultāts būtu
salīdzināmāks ar iepriekšējiem uzdevumiem.

``` r
filter_laiks=system.time({
  MVR_atlase3=sf::st_filter(MVR_centrs, atlase)
})
print(filter_laiks)
```

    ##    user  system elapsed 
    ##   13.32    0.41   13.81

šis prasa līdzīgi ilgi kā spatial join, proti ātrāk kā clipping. Bet kas
ir interesanti - ir mazāk atribūtu - 71, nevis 75 kā pēc cliping vai
spatial join. Arī objektu ir par ~550 mazāk. Tas nozīmē, ka šī funkcija
neapvieno atribūtus, tikai nofiltrē no MVR datiem tos, kas pārklājas ar
karšu lapām. Tas nozīmē, ka tagad es nevarēšu taisīt kopīgo karti, kas
pēc kvadrāta nosaukuma sadala poligonus pa krāsiņām. Tas nozīmē arī to,
ka es nevarēšu pārējo darīt kā iepriekš, jo iepriekš daudz izmantoju
lauku “NOSAUKUMS”, kas ar spatial filtering atribūtos netiek ieviests no
atlases karšu lapām. Tomēr es varu turpināt darboties ar filtrēšanas
operatoriem.

Tātad, lai darbotos tālāk ar individuālām četrām lapām, man ir jāatlasa
atsevišķi tie poligoni no MVR, kas pārklājas ar katru no tām objektā
“atlase”. Visādīgi pamēģināju, bet beigās man sanāca, ka šim ir jāpieiet
no cita gala, proti, izveidoju četras atsevišķas kartes lapas (nevis
vienu atlasi, kurā jau ir četras lapas), un tad pielietoju st_filter()
funkciju. Faktiski, ja es gribētu no tā izvairīties, vajadzētu lietot
st_join(). Tieši priekš šī uzdevuma, tamdēļ, st_filter() man neliekas
parocīgākais variants, bet citās dzīves situācijās tas noteikti ir
derīgs (kad nav nepieciešama atribūtu apvienošana).

``` r
filter_atlase=system.time({
atlase_riga <- atlase %>% filter(NOSAUKUMS == "Rīga")
atlase_jelgava <- atlase %>% filter(NOSAUKUMS == "Jelgava")
atlase_jurmala <- atlase %>% filter(NOSAUKUMS == "Jūrmala")
atlase_baldone <- atlase %>% filter(NOSAUKUMS == "Baldone")

#tagad katrai kartes lapai atsevišķi lietošu st_filer(), lai iegūtu vēlamo rezultātu.
MVR_riga3 <- st_filter(MVR_centrs, atlase_riga)
MVR_jelgava3 <- st_filter(MVR_centrs, atlase_jelgava)
MVR_jurmala3 <- st_filter(MVR_centrs, atlase_jurmala)
MVR_baldone3 <- st_filter(MVR_centrs, atlase_baldone)

#eksportēju kā geoparquet
st_write_parquet(MVR_jelgava3, dsn="MVR_jelgava3.parquet")
st_write_parquet(MVR_baldone3, dsn="MVR_baldone3.parquet")
st_write_parquet(MVR_riga3, dsn="MVR_riga3.parquet")
st_write_parquet(MVR_jurmala3, dsn="MVR_jurmala3.parquet")
})
```

Redzu, ka šādi atlasot objektus, iegūstu tikpat poligonus vienā kartē kā
ar st_join un clipping pieeju. Atšķirība ir tajā, ka 1) šis man prasa
lielāku piepūli, kas nozīmē arī vairāk patērēta laika, kas nav
optimāli.  
2) filtrējot pēc visām 4 lapām kopā ieguvu mazāk objektus. Laikam jau
tāpēc, ka “dublieri” tagad ir katrs savā lapā, bet kopējā tie vienkārši
neeksistē, jo visi poligoni pieder visai kopējai kartes lapai. Šis
varētu būt pateisais poligonu skaits, kas ir saistīts ar šo teritoriju.

``` r
jur=ggplot(MVR_jurmala3) +
  geom_sf(aes(), size = 0.2) +  
  theme_minimal() 

rig=ggplot(MVR_riga3) +
  geom_sf(aes(), size = 0.2) +  
  theme_minimal() 

jel=ggplot(MVR_jelgava3) +
  geom_sf(aes(), size = 0.2 )+  
  theme_minimal() 

bal=ggplot(MVR_baldone3) +
  geom_sf(aes(), size = 0.2) +  
  theme_minimal() 

ggarrange(jur, rig, jel, bal,
    labels = c("Jūrmala", "Rīga", "Jelgava", "Baldone"),
    font.label = list(size = 8, face = "bold"),  
    label.y = c(1, 1, 1.1, 1.1),
    ncol = 2, nrow = 2)
```

![](Uzd05_Starka_files/figure-gfm/view.lapas3-1.png)<!-- --> Kā jau
paredzēju, te ir tāda pati situācija kā ar st_join(), proti, ir
poligoni, kas šķērso kartes lapas malas. Tie nav apgrizti kā ar
clipping. Vienīgā atšķirība no st_join() ir, ka atribūti nav apvienoti.
Nez kā ies ar kopā līmēšanu.

Novācu sastrādātos objektus.

``` r
rm(jur,rig,jel,bal)
```

## 3.2. Filtering un mana funkcija

Rediģēju funkciju tā, lai tā veido faila nosaukumu ar apakšuzdevuma
numuru galā.

``` r
mana_funkcija=function(input_files){
  MVR_centrs = st_read_parquet(input_files)

  ref_rastrs10m=rast("../Uzd03/ref_rastrs/LV10m_10km.tif")
  ref_rastrs100m=rast("../Uzd03/ref_rastrs/LV100m_10km.tif")
  
  ref_rastrs10m_centrs = terra::crop(ref_rastrs10m,MVR_centrs)
  ref_rastrs100m_centrs = terra::crop(ref_rastrs100m,MVR_centrs)
  
  ref_rastrs10m_centrs2=raster::raster(ref_rastrs10m_centrs)
  
  priedes=MVR_centrs%>%filter(s10 == 1)

  priedes_rastrs10m=fasterize(priedes,ref_rastrs10m_centrs2,background=0)

  priedes2_rastrs10m = rast(priedes_rastrs10m)

  priedes2_rastrs10m[is.na(ref_rastrs10m_centrs)] = NA
  
  output_dir = "../Uzd05/"
  
  file_name = tools::file_path_sans_ext(basename(input_files))
  
  output_filename = paste0(output_dir, file_name, "_2_priedes.tif")
  
  priedes100=resample(priedes2_rastrs10m,ref_rastrs100m_centrs,
                      method="average",
                      filename=output_filename,
                      overwrite=TRUE)
}
```

Palaižu funkciju:

``` r
input_files=c("MVR_jurmala3.parquet",
  "MVR_riga3.parquet",
  "MVR_baldone3.parquet",
  "MVR_jelgava3.parquet")

# Kodolu klasera definēšana
cl2 <- makeCluster(2)
registerDoParallel(cl2)

# Paralēla funkcijas palaišana

funkcijas_laiks3=system.time({
  foreach(i = seq_along(input_files), .packages = c("terra", "dplyr","sfarrow","fasterize"), .export="mana_funkcija") %dopar% {
    file=input_files[i]
    mana_funkcija(file)
  }
})
```

    ## Warning in e$fun(obj, substitute(ex), parent.frame(), e$data): already
    ## exporting variable(s): mana_funkcija

``` r
stopCluster(cl2)

print(funkcijas_laiks3)
```

    ##    user  system elapsed 
    ##    2.45    0.42   39.58

Izpildes laiks ir līdzīgs.

## 3.3. Rastru apvienošana pēc spatial filtering

Līmēšu kopā. Par cik šim ir malas ar pārklāšanos, izmantošu arī
mosaic().

``` r
#atlasu uztaisītos rastrus
dir="../Uzd05/"
rastri3_2 = c("MVR_jurmala3_2_priedes.tif",
  "MVR_riga3_2_priedes.tif",
  "MVR_baldone3_2_priedes.tif",
  "MVR_jelgava3_2_priedes.tif")

#rediģēju funkcijā nosaukumu apvienotajam rastram (mozaīkai)
apvienosanas_funkcija3_3=function(rastri3_2){
    #ielasu visus rastrus ar rast()
    rastru_objekti <- lapply(rastri3_2, rast)
    
    #pakļauju katru no saraksta rastriem funkcijai mosaic()
    atlases_Mozaika <- do.call(mosaic, c(rastru_objekti, list(fun = "max")))

    # Saglabāju apvienoto rastru mozaīku
    output_path <- "../Uzd05/3_3_atlasesMozaika.tif"
    writeRaster(atlases_Mozaika, output_path, overwrite = TRUE)
  }

#palaižu šo funkciju un izmēru, cik laika tā prasa. 
apvienosanas_laiks3=system.time({
  apvienosanas_funkcija3_3(rastri3_2)
})

print(apvienosanas_laiks3)
```

    ##    user  system elapsed 
    ##    0.18    0.06    0.26

ir OK.

Apskatīšu kontekstā ar rezultātiem no spatial join un clipping.

``` r
mozaika1_3=rast("../Uzd05/1_3_atlasesMozaika.tif")
mozaika2_3=rast("../Uzd05/2_3_atlasesMozaika.tif")
mozaika3_3=rast("../Uzd05/3_3_atlasesMozaika.tif")

par(mfrow = c(1, 3))

plot(mozaika1_3, main = "Spatial join")
plot(mozaika2_3, main = "Clipping")
plot(mozaika3_3, main = "Spatial filtering")
```

![](Uzd05_Starka_files/figure-gfm/mozaiku.salidzinasana2-1.png)<!-- -->
Redzams, ka tiešām līdzīgi kā pie spatial_join() šeit ir labi
salīmējušās kartes to savstarpējās robežās, bet malās ir palikusi
informācija par poligoniem, kas iet tām pāri. Arī nav šis te “robiņš”
pie mola, kas parādījās pie Clipping.

Atiestatu grafiku izvietošanu un noņemu lieko.

``` r
par(mfrow = c(1, 1))
rm(mozaika1_3,mozaika2_3,mozaika3_3)
```

# 4. Cits risinājums

Man kopumā patika spatial join funkcija, tāpēc pamēģināšu
papildargumentus (papildfunkcijas). Pamēģināšu atlasīt mežaudzes, kas ir
iekšā kvadrātos, bet nerobežojas ar malām. Praksē šis risinājums nebūtu
pielietojams projekta darbos, jo zaudētu informāciju par daudz
poligoniem un būs problēmas ar jēgpilnu salīmēšanu, bet ieliku šeit kā
variāciju pašas pieredzes bagātināšanai. Pēc aprakstu izpētīšanas
manuprāt to izdarīs savienošanas arguments join=st_within (likās, ka arī
st_contains_properly darīs to pašu, tomēr ar šo papildargumentu tika
atgriezti 0 objektu… prātoju kāpēc tā).

## 4.1. Spatial join ar papildargumentu “Within”

Šis papildarguments paredz, ka tiks atlasīti poligoni, kas pilnībā
atrodas kartes lapā, proti, neiziet no kartes robežām.

``` r
within_laiks=system.time({
  MVR_atlase4=sf::st_join(MVR_centrs, atlase, left =FALSE, join=st_within)
})
print(within_laiks)
```

    ##    user  system elapsed 
    ##   12.78    0.23   13.11

Tiek atlasīti 83367 objekti. Ar st_intersects papildargumentu bija
85523, kas jau apliecina paredzēto rezultātu par informācijas zudumu uz
kartes robežām. Darba izpildes laiks līdzīgs.

Izveidošu atsevišķas atlases no šī apvienotā objekta, izdalot atsevišķas
kartes lapas, lai varētu pēc tam paskatīties, kas notiek to robežošanās
vietās. Pie reizes arī šīs atlases saglabāšu kā atsevišķus geoparquet
failus.

``` r
within_atlase=system.time({
MVR_jelgava4=MVR_atlase4 %>% filter(NOSAUKUMS=="Jelgava")
MVR_baldone4=MVR_atlase4 %>% filter(NOSAUKUMS=="Baldone")
MVR_riga4=MVR_atlase4 %>% filter(NOSAUKUMS=="Rīga")
MVR_jurmala4=MVR_atlase4 %>% filter(NOSAUKUMS=="Jūrmala")

st_write_parquet(MVR_jelgava4, dsn="MVR_jelgava4.parquet")
st_write_parquet(MVR_baldone4, dsn="MVR_baldone4.parquet")
st_write_parquet(MVR_riga4, dsn="MVR_riga4.parquet")
st_write_parquet(MVR_jurmala4, dsn="MVR_jurmala4.parquet")
})
```

Katrā kartes lapā ir sekojošs skaits rindu: Jelgava - 22171
(salīdzinājumam st_intersect atlasīja 22807) Baldone - 32753
(salīdzinājumam st_intersect atlasīja 33449) Rīga - 16209
(salīdzinājumam st_intersect atlasīja 16705) Jūrmala - 12234
(salīdzinājumam st_intersect atlasīja 12562)

Apskatīšu vizuāli, kā sanāca.

``` r
ggplot(MVR_atlase4) +
  geom_sf(aes(color = NOSAUKUMS), size = 1) +  
  theme_minimal() +
  labs(color = "NOSAUKUMS")
```

![](Uzd05_Starka_files/figure-gfm/view.within-1.png)<!-- --> Redzu, ka
poligoni, kas ir uz karšu lapu robežām nav iekļauti. To saskatīt nav
viegli šādā nelielā artē, bet ir redzams ka robežās ir gaiši “robiņi”
(labi redzams starp Jelgavas un Jūrmalas kvadrātu, kur iepriekš bija
lielo poligonu pārklāšanās).

## 4.2. st_within un mana funkcija

Nedaudz rediģēju funkciju pareiza faila nosaukuma veidošanai:

``` r
mana_funkcija=function(input_files){
  MVR_centrs = st_read_parquet(input_files)

  ref_rastrs10m=rast("../Uzd03/ref_rastrs/LV10m_10km.tif")
  ref_rastrs100m=rast("../Uzd03/ref_rastrs/LV100m_10km.tif")
  
  ref_rastrs10m_centrs = terra::crop(ref_rastrs10m,MVR_centrs)
  ref_rastrs100m_centrs = terra::crop(ref_rastrs100m,MVR_centrs)
  
  ref_rastrs10m_centrs2=raster::raster(ref_rastrs10m_centrs)
  
  priedes=MVR_centrs%>%filter(s10 == 1)

  priedes_rastrs10m=fasterize(priedes,ref_rastrs10m_centrs2,background=0)

  priedes2_rastrs10m = rast(priedes_rastrs10m)

  priedes2_rastrs10m[is.na(ref_rastrs10m_centrs)] = NA
  
  output_dir = "../Uzd05/"
  
  filename_prefix <- MVR_centrs %>%
    pull(NOSAUKUMS) %>%
    unique()%>% .[1]
  
  output_filename = paste0(output_dir, filename_prefix, "_4_2_priedes.tif")
  
  priedes100=resample(priedes2_rastrs10m,ref_rastrs100m_centrs,
                      method="average",
                      filename=output_filename,
                      overwrite=TRUE)
}
```

Ieviešu funkciju

``` r
input_files=c("MVR_jurmala4.parquet",
  "MVR_riga4.parquet",
  "MVR_baldone4.parquet",
  "MVR_jelgava4.parquet")

# Kodolu klasera definēšana
cl2 <- makeCluster(2)
registerDoParallel(cl2)

# Paralēla funkcijas palaišana

funkcijas_laiks4=system.time({
  foreach(i = seq_along(input_files), .packages = c("terra", "dplyr","sfarrow","fasterize"), .export="mana_funkcija") %dopar% {
    file=input_files[i]
    mana_funkcija(file)
  }
})
```

    ## Warning in e$fun(obj, substitute(ex), parent.frame(), e$data): already
    ## exporting variable(s): mana_funkcija

``` r
stopCluster(cl2)

print(funkcijas_laiks4)
```

    ##    user  system elapsed 
    ##    2.56    0.38   37.50

Atkal jau laiks ir līdzīgs.

Itkā nav uzdevums līmēt kopā, bet man šķiet, ka salīmešana varētu labi
ilustrēt st_within īpašības.

``` r
rastri4_2 = c("Jūrmala_4_2_priedes.tif",
  "Rīga_4_2_priedes.tif",
  "Baldone_4_2_priedes.tif",
  "Jelgava_4_2_priedes.tif")

apvienosanas_funkcija4_3=function(rastri4_2){
    rastru_objekti <- lapply(rastri4_2, rast)
    atlases_Mozaika <- do.call(mosaic, c(rastru_objekti, list(fun = "max")))
    output_path <- "../Uzd05/4_3_atlasesMozaika.tif"
    writeRaster(atlases_Mozaika, output_path, overwrite = TRUE)
  }

apvienosanas_laiks4=system.time({
  apvienosanas_funkcija4_3(rastri4_2)
})

print(apvienosanas_laiks4)
```

    ##    user  system elapsed 
    ##    0.17    0.05    0.23

Gana ātri.

Apskatīšu vizuāli kontekstā ar rezultātiem no st_intersects, kas ir
līdzīgākā funkcija.

``` r
mozaika1_3=rast("../Uzd05/1_3_atlasesMozaika.tif")
mozaika4_3=rast("../Uzd05/4_3_atlasesMozaika.tif")

par(mfrow = c(1, 3))

plot(mozaika1_3, main = "join=st_intersects")
plot(mozaika4_3, main = "join=st_within")
```

![](Uzd05_Starka_files/figure-gfm/mozaiku.salidzinasana4-1.png)<!-- -->
Redzams, ka te ir jau paredzētās problēmas. Šeit ir ne tikai “robiņš”
pie mola, kas parādījās pie Clipping, bet arī taisnā līnijā uz leju no
tā josla bez informācijas par mežiem, kā arī perpendikulāri tai tāda
pati josla, kur karšu lapas saskaras horizontāli.

Atiestatu grafiku izvietošanu.

``` r
par(mfrow = c(1, 1))
```

# 5. Mana funkcija visai MVR informācijai

Pakļaušu visus centra virsmežniecības datus manai funkcijai, paturot
galvā, ka iepriekšējās pieejas, sadalot darbu pa kartes lapām, aizņēma
ap 30 sekundēm.

``` r
#rediģēju nosaukuma veidošanu
mana_funkcija=function(input_file){
  MVR_centrs = st_read_parquet(input_file)

  ref_rastrs10m=rast("../Uzd03/ref_rastrs/LV10m_10km.tif")
  ref_rastrs100m=rast("../Uzd03/ref_rastrs/LV100m_10km.tif")
  
  ref_rastrs10m_centrs = terra::crop(ref_rastrs10m,MVR_centrs)
  ref_rastrs100m_centrs = terra::crop(ref_rastrs100m,MVR_centrs)
  
  ref_rastrs10m_centrs2=raster::raster(ref_rastrs10m_centrs)
  
  priedes=MVR_centrs%>%filter(s10 == 1)

  priedes_rastrs10m=fasterize(priedes,ref_rastrs10m_centrs2,background=0)

  priedes2_rastrs10m = rast(priedes_rastrs10m)

  priedes2_rastrs10m[is.na(ref_rastrs10m_centrs)] = NA
  
  output_dir = "../Uzd05/"
  
  output_filename = paste0(output_dir, "MVRcentrs_5_1_priedes.tif")
  
  priedes100=resample(priedes2_rastrs10m,ref_rastrs100m_centrs,
                      method="average",
                      filename=output_filename,
                      overwrite=TRUE)
}
```

Ieviešu funkciju.

``` r
input_file_path <- "../Uzd02/centrs_kopa2.parquet"

funkcijas_laiks5=system.time({
  mana_funkcija(input_file_path)
})
```

    ## |---------|---------|---------|---------|=========================================                                          |---------|---------|---------|---------|=========================================                                          

``` r
print(funkcijas_laiks5)
```

    ##    user  system elapsed 
    ##  121.52   44.55  170.74

Protams, šis jau prasīja ievērojami ilgāku laiku (apmēram 5x ilgāk).

Un tagad apgriežu izveidoto rastru līdz izvēlēto četru kartes lapu
kopējām ārējām robežām.

``` r
apgriesanas_laiks5 <- system.time({
  #nosaku robežas
  atlases_robezas = st_bbox(atlase)
  atlases_robezu_extent <- ext(
    as.numeric(atlases_robezas["xmin"]), 
    as.numeric(atlases_robezas["xmax"]), 
    as.numeric(atlases_robezas["ymin"]), 
    as.numeric(atlases_robezas["ymax"]))

    #apgriežu
  MVRcentrs_5_1_priedes = rast("../Uzd05/MVRcentrs_5_1_priedes.tif")
  MVR_cents_atlase=terra::crop(MVRcentrs_5_1_priedes,atlases_robezu_extent)
})

print(apgriesanas_laiks5)
```

    ##    user  system elapsed 
    ##    0.02    0.02    0.03

Šis gan bija zibenīgi.

Apskatam.

``` r
plot(MVR_cents_atlase)
```

![](Uzd05_Starka_files/figure-gfm/kartes.lapa5view-1.png)<!-- --> Nu
lūk, tas pats rezultātas, kas iegūts ar st_join(join=st_intersects),
vienīgi šis prasīja ilgāk sistēmas laika ziņā (tieši manas funkcijas
palaišana uz visiem datiem, ne pati apgriešana, jo tās laiks bija
niecīgs). Komandrindu ziņā šī pieeja bija īsāka. Par laika salīdzinājumu
un komandrindām skatīt secinājumus.

# Secinājumi

Apskatīsim kopējos laikus katram apakšuzdevumam: (tabulā nav iekļauti
grafiku veidošanas laiki, jo ne visos uzdevumos tie tika taisīti)

``` r
sec.laiki=data.frame(
  Uzd_Nr=c("1.","2.","3.","4.","5."),
  Pieeja=c("spatial_join(join=intersects)", "clipping, jeb st_intersection()", "spatial filtering", "spatial_join(join=within)","Apgriešana post-factum"),
  Funkcijas_laiks=c(join_laiks[3],clip_laiks[3],filter_laiks[3],within_laiks[3],NA),
  Atlasu_veidosanas_laiks=c(join_atlase[3],clip_atlase[3],filter_atlase[3],within_atlase[3],NA),
  Manas_funkcijas_laiks=c(funkcijas_laiks1[3],funkcijas_laiks2[3],funkcijas_laiks3[3],funkcijas_laiks4[3],funkcijas_laiks5[3]),
  Apvienosanas_laiks=c(apvienosanas_laiks1[3],apvienosanas_laiks2[3],apvienosanas_laiks3[3],apvienosanas_laiks4[3],NA),
  Apgriesanas_laiks=c(NA,NA,NA,NA,apgriesanas_laiks5[3])
)
sec.laiki$Kopejais_laiks=rowSums(sec.laiki[,c(3:7)],na.rm = TRUE)

library(knitr)
```

    ## 
    ## Attaching package: 'knitr'

    ## The following object is masked from 'package:terra':
    ## 
    ##     spin

``` r
kable(sec.laiki, format = "markdown")
```

| Uzd_Nr | Pieeja | Funkcijas_laiks | Atlasu_veidosanas_laiks | Manas_funkcijas_laiks | Apvienosanas_laiks | Apgriesanas_laiks | Kopejais_laiks |
|:---|:---|---:|---:|---:|---:|---:|---:|
| 1\. | spatial_join(join=intersects) | 21.41 | 7.39 | 40.69 | 0.25 | NA | 69.74 |
| 2\. | clipping, jeb st_intersection() | 24.35 | 7.09 | 40.67 | 0.28 | NA | 72.39 |
| 3\. | spatial filtering | 13.81 | 36.49 | 39.58 | 0.26 | NA | 90.14 |
| 4\. | spatial_join(join=within) | 13.11 | 6.75 | 37.50 | 0.23 | NA | 57.59 |
| 5\. | Apgriešana post-factum | NA | NA | 170.74 | NA | 0.03 | 170.77 |

## Kura no apakšuzdevumu versijām ir labākā?

Sāksim ar to, ka 4. uzdevuma risinājums vienkārši neatbilst vēlamajam
iegūtajam rezultātam, līdz ar to tas automātiski atkrīt kā labākais
risinājums. Tad nākamais, ko varētu ņemt vērā ir laiks. Apskatot tabulu
redzams, ka risinājums no 5. uzdevuma ir 2x-3x ilgāks kā pārējie
piemērotie, tāpēc šis, lai gan komandrindu ziņā īss, tomēr sistēmas
laika ziņā nav optimālākais. No tā arī izriet galvenais secinājums, ka
darbu sadalīšana pa kartes lapām un paralēla izpilde uz vairākiem
kodoliem ir laika ziņā efektīvākā pieeja. Nākamais, ko es atmestu kā
mazāk optimālu ir clipping. Lai gan šķietami tas varētu būt efektīvi
malu nepārklāšanās dēļ, es novēroju, ka lai gan itkā ir tie paši ievades
dati, par koordinātu sistēmām esmu parūpējusies utml., tomēr novēroju
nelielu defektu vienā kartes vietā pēc četru rastru apvienošanas.
Otrkārt - clipping prasa ilgāku laiku kā spatial_join funkcija. Tā arī
ir parādīts salīdzinājumā par st_intersects (kas lietots par argumentu
manā st_join 1. apakšuzdevumā) un st_intersection, kas tiek lietots manā
clipping funkcijā -
<https://r-spatial.org/r/2017/06/22/spatial-index.html> Šeit uz četrām
kartes lapām tas ir varbūt maznozīmīgi, tomēr ja būtu jāstrādā ar lielu
apjomu datu, piemēram, visas Latvijas vai piemēram, Eiropas līmenī, jau
tas sasummētos un radītu papildus skaitļošanas izmaksas.

Manuprāt drošākie ir spatial join un spatial filering risinājumi, kas ir
ļoti līdzīgi, bet atšķiras ar to, vai atribūti tiek salīmēti vai netiek.
Abos gadījumos veidojas maliņas. Ja maliņas nepatīk, tās var apgriezt
(šajā uzdevumā tas manuprāt nebija svarīgi). Pārklāšanās, ja visas
apvienojamās kartes lapas ir balstītas uz datiem, kas homogēni pieejami
par visu interesējošo vidi, un ja to aprēķins ir baltīts uz vienotu
funkciju, var būt laba lieta, jo var kalpot kā droša puzles salikšanas
sistēma (manuprāt). Šo pārdomu es balstu iegūtajos rezultātos, kur abos
gadījumos apvienotais rastrs bija bez problēmām (vismaz neko
nemanīju).Tomēr ja jāizvēlas starp spatial join un spatial filtering, es
izvēlētos spatial join, jo atribūtu salīmēšana atvieglo tālākas darbības
ar atlasēm, failu nosaukum veidošanu un tamlīdzīgi, kas ietaupa laiku.
Var vienkārši darboties ar vienu failu. Dēļ tā, ka atribūti netiek
salīmēti, ir jāveic papildus darbības pie datu atlasīšanas, kas kā
redzams tabulā, rada papildus laika patēriņu (vispār ironiski). Citādi
funkciju sistēmas laika ziņā abas metodes ir līdzīgas, un noteikti arī
spatial filtering domāju ir labs risinājums pie situācijas kurā atribūtu
apvienošanai nav nozīmes.

## Papildu pārdomas

Personīgi domāju, ka ir svarīgi veidot īsas un vienkāršas komandrindas.
“Labs” piemērs tam ir mans gandrīz 1000 rindiņu garais risinājums šim
uzdevumam, kur pa vidu veicu visādas pašdarbības kā ggploti un grafiskas
salīdzināšanas, geoparquet failu rakstīšana un lasīšana, filtrēšana
priekš atsevišķajām četrām kartes lapām, no kā varētu arī izvairīties
utml.) Varētu visu izpildīt uz 3x mazāk komandrindām, izmantojot vairāk
gudras kombinācijas, kas cilvēkam ar pieredzi tāpat būtu gana lasāmas un
saprotamas. Viss atkarīgs no situācijas. Manā gadījumā komandrindu ir
daudz, bet tās ir vienkāršas. Prasksē, darbojoties ar šādu uzdevumu, kas
nebūtu mācību darbs, “nododamo” skriptu vajadzētu optimizēt, no tā
dzēšot darbības, kas nav nepieciešamas, bet šeit ir iekļautas tikai
“priekš sevis”.
