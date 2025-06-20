Piektais uzdevums: procesu dalīšana un rezultātu apvienošana
================

## Termiņš

Līdz ~~(2025-01-10)~~ ~~(2025-01-27)~~ **2025-02-07**, izmantojot
[fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo)
un [pull
request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request-from-a-fork)
uz zaru “Dalibnieki”, šī uzdevuma direktorijā pievienojot .Rmd vai .qmd
failu, kura nosaukums ir Uzd05\_\[JusuUzvards\], piemēram,
`Uzd05_Avotins.Rmd`, kas sagatavots izvadei github dokumentā (piemēram,
YAML galvenē norādot `output: rmarkdown::github_document`), un tā radīto
izvades failu.

## Premise

Ārkārtīgi bieži ģeoprocesēšanas uzdevumi ir pārāk apjomīgi, lai tos
nepieciešamajā izšķirtspējā īstenotu visai analīzes telpai vienā procesā
saprātīgā laika periodā. Sevišķi aktuāli tas ir tad, ja šos
ģeoprocešēšanas uzdevumus laiku pa laikam ir nepieciešams atkārtot. Tā
tas ir, piemēram, šajā projektā - tā kā vide mainās, mainās tās apraksti
datubāzēs, attiecīgi ir jāatkārto ekoģeogrāfisko mainīgo sagatavošanas
procedūras, lai modeļu rezultātus projicētu aktuālajā situācijā.

Iepriekšējā uzdevumā vingrojāt ar funkcijām un to ieratīvu atkārtošanu,
cikla vadībai izmantojot atsevišķus failus. Ja uzdevuma veikšana ir
pakārtota informācijai, kas tiek uzglabāta vienādi strukturētā datu kopā
(*dataset* vai *database*), šāda pieeja varētu būt laba, jo darbības
veicamas ar relatīvi nelieliem failiem, tātad, tās nav sevišķi daudz
resursus prasošas (kaut vai lasīšanas un rakstīšanas laiks). Tomēr
visbiežāk ir nepieciešams strādāt vienlaikus ar vairākām datu kopām,
kuras ir atšķirīgi strukturētas - gan datubāzes/atribūtlauku, gan
telpiskā ziņā. Šim vajadzētu būt pašsaprotami, bet (1) var paskatīties
iepriekšējos uzdevumos lejupielādētos Meža Valsts reģistra (MVR) un
Lauku atbalsta dienesta datus kādā interaktīvā GIS - šīm datubāzēm nav
kopīgu atribūtu, to grupēšana vai dalīšana nav skaidri vienotas
datubāzē, nemaz nerunājot par savstarpējo topoloģiju, kā arī (2)
aplūkojot iepriekšējā uzdevumā sagatavotos katras nodaļas rastrus -
katrs no tiem aptver visu valsti, lai gan jēgpilnu informāciju satur
tikai par daļu tās, turklāt tiešā veidā tie nav apvienojami, jo visas
šūnas satur vērtības. Daļu no šiem izaicinājumiem var risināt pārdomātu
procesu telpiskā dalīšana, citus - pārdomāta darbu gaitas dalīšana
soļos. Katrā ziņā, ir vērtīgi ik pa laikam vizuāli izpētīt iegūtos
starprezultātus, lai pamanītu kādas kļūdas vai nepilnības un iespējami
laicīgi tās novērstu. Šajā uzdevumā vairāk par telpisko dalīšanu.

Jebkuram lielākam ģeoprocesēšanas darbam ir svarīgi būt ar vīziju par
iegūstamo rezultātu, tālākās izmantošanas mērķiem un darbībām. Diemžēl,
ne vienmēr tas ir pietiekoši skaidrs no paša sākuma, tādēļ darba gaitā
var nākties atgriezties pie kādiem agrākiem soļiem un tos atkārtot.
Tomēr, ciktāl tas attiecas uz ģeoprocesēšanas dalīšanu telpiskās daļās,
ir vienojoši elementi - plašas teritorijas mēs cenšamies dalīt regulārās
daļās, kam par pamatu var izmantot kādas interešu reģionā pieejamās
karšu nomenklatūras/lapas vai vispārīgākā situācijā nacionālos,
reģionālos vai globālos koordinātu tīklus (plašākajos telpiskajos
līmeņos gan ir vērts vērst uzmanību uz platības salīdzināmību koordinātu
sistēmas dažādās daļās - var būt gudri izmantot kādu no *equal area*
projekcijām, piemēram, [Lambert Azimuthal Equal
Area](https://epsg.io/9820-method), kas ir plaši izmantota Eiropā). Ja
uzdevumi ir tādi, kurus ietekmē malas, tad dalījuma zonas ir iespējams
buferēt, starp tām veidojot telpisku pārklāšanos aprēķinu gaitā un to
novēršot pie rezultātu apvienošanas. Šī projekta ietvaros strādāsim ar
1993. gada Latvijas topogrāfisko karšu sistēmas karšu lapām M:50000 un
izmantosim Latvijas taisnleķa projicēto koordinātu sistēmu [LKS-92 /
Latvia TM](https://epsg.io/3059). Karšu lapas ir pieejamas [projekta
*Zenodo*
repozitorijā](https://zenodo.org/communities/hiqbiodiv/records?q=&l=list&p=1&s=10&sort=newest)
kā [references vektordatu slānis](https://zenodo.org/records/14277114)
`tks93_50km.parquet`. Ir iespējamas arī citas dalījuma zonas,
izmantojot, piemēram, slāņos `tikls100_sauzeme.parquet` un
`tikls1km_sauzeme.parquet` pieejamos identifikatorus, tos sadalot - šis
visspecifiskāk attiecas uz lauku `ID1km` abos pēdējos nosauktajos
slāņos.

Biežākās pieejas vektordatu apstrādes telpiskai dalīšanai ir telpiskā
savienošana (*spatial join*), griešana (*clipping*) un telpiskā atlase
(*spatial filtering/subsetting*). Katrai no tām ir savas stiprās un
vājās puses, kuras ir jāapzinās ikreiz izvēloties konkrētiem uzdevumiem
piemērotāko rīku un darbību kombināciju:

- *spatial join* izmanto ģeometriju telpisko pēdu/nospiedumu
  (*footprint*), lai savienotu atribūtlaukus. Izmantojot `sf::st_join`
  ([apraksts](https://r-spatial.github.io/sf/reference/st_join.html))
  pēc noklusējuma tiek izmantots `left_join`, kas kreisās puses
  (pirmajam argumentam) pievieno visus labās puses laukus tām
  ģeometrijām, kurām ir saistīti telpiskie nospiedumi. Ja izmanto
  argumentu `left=FALSE` tiek atgrieztas tikai tās ģeometrijas, kuru
  nospiedumi ir saistīti. Nospiedumu saistīšanai ir izmantojamas dažādas
  *clipping* pieejas, kuras ir maināmas ar argumentu `join`, kurā pēc
  noklusējuma tiek izmantota funkcija `sf::st_intersects` (aprakstam
  skatiet nākošos punktus). Tā kā ir iejaukts gan darbs ar datubāzi, gan
  ģeometriju pākrlāšanos, šī ir visai smagnēja pieeja. Toties ērta, jo
  sevišķi, ja kreisās puses objekti ir punkti. Vēršama uzmanība
  situācijām, kad viena kreisās puses ģeometrija ir saistīta ar vairākām
  labās puses ģeometrijām - tādos gadījumos katra kreisās puses
  ģeometrija tiek dublēta. Tas savukārt nozīmē, ka procesu dalīšanā
  kreisās puses ģeometrijas, pieņemot, ka labās puses ģeometrijas nosaka
  dalīšanas zonas, kuras atrodas uz malām, būs katrā no uzdevumiem. Pats
  par sevi tas nav slikti un nereti pat ir nepieciešams, tomēr tā ir
  lieta, kurai pievēršama uzmanība domājot par rezultātu apvienošanu.

- *clipping* tieši operē ar ģeometriju nospiedumiem
  ([apraksts](https://r.geocompx.org/geometry-operations#clipping) un
  [cits
  apraksts](https://r-spatial.github.io/sf/reference/geos_binary_ops.html)).
  Vispārīgā situācijā - kreisās puses ģeometrijas tiek pārklātas ar
  labās puses ģeometrijām, atgriežot kādu no pārklāšanās vai
  nepārklāšanās daļām ar visiem tās atribūtiem. Šīs funkcijas strādā ar
  ģeometrijām, tām ieviešot [Buleina loģikas
  operatorus](https://r4ds.had.co.nz/transform.html#logical-operators).
  Ja ir daudz ģeometriju ar daudz virsotnēm, šis var būt smagnējs
  uzdevums, kura veikšanai var būt vērts iesaistīt kādas ātrdarbīgās
  datubāzu pieejas. Darba plūsmas vienotībai, protams, tās vadot no R.
  Izmantojot šīs darbības, ir jāpieskata, kādi objekti tiek radīti, vai
  starp tiem nav kaut kā lieka un ir jāpievērš uzmanība jauno ģeometriju
  atribūtu informācijas uzticamībai - pēc ģeometrijas griešanas vismaz
  laukums un perimetrs būs mainījušies, bet, atkarībā no veicamā
  uzdevuma, var būt nepieciešams jaunais (pēc griešanas) vai vecais
  (pirms griešanas), kas nozīmē, ka arī darbu uzdevumu izpildīšanas
  secībai ir nozīme. Tajā pašā laikā, domājot par procesu dalīšanu, šīs
  pieejas nodrošina, ka viena ģeometrija vienlaikus ir tikai vienā
  neatkarīgās apstrādes telpiskajā daļā/zonā.

- *spatial filtering/subsetting* izmanto ģeometriju nospiedumu
  savstarpējo pārklāšanos, lai noteiktu to saistību. Ir pieejami dažādi
  savstarpējā novietojuma attiecību nosacījumi, kas {sf} ir ieviesti
  [atsevišķu funkciju
  veidā](https://r-spatial.github.io/sf/reference/geos_binary_pred.html).
  Turklāt katrai no šīm funkcijām ir iespējami papildargumenti attiecībā
  pret dublēšanos un atgriežamo objektu. Šie argumenti ir izmantojami
  arī *spatial join* darbībās.

- [interesanti piemēri telpiskajām
  atlasēm](https://cran.r-project.org/web/packages/sfnetworks/vignettes/sfn03_join_filter.html)

Nevar noliegt, ka uzdevumos ar vektordatu griešanu klasiskā GIS,
piemēram ESRI produkti un QGIS nereti ir pārāki par R, ja tā nav
paplašināta ar telpiskajām datubāzēm. Tomēr šī projekta ietvaros
fokusējamies uz R, lai nodrošinātu vienotu darba plūsmu, iespējojot
nepieciešamos paplašinājumus un daudz izmantojot rastra datus. Darbā ar
rastru R ir spēcīgāka par vairumu klasisko GIS. Arī rastru ir iespējams
procesēt telpiskās daļās. Visbiežāk tas nozīmē nepieciešamās teritorijas
izgriešanu (*crop*) un atsevišķu telpisko daļu saglabāšanu atsevišķos
failos kā individuālas karšu lapas.

Lai gan vekotrdatu ģeometriju griešanā klasiskā GIS mēdz būt pārāka,
parasti tā ievērojami atpaliek, kad nepieciešama iegūto rezultātu
apvienošana - pēc individuālās telpas daļās veiktās apstrādes
kopīgu/vienotu produktu sagatavošana. Neatkarīgi no izvēlētās
programmatūras, ja nepieciešams vienots produkts, tāds ir jāsagatavo.
Šeit jāatgriežas pie soļa pirms apstrādes procesu dalīšanas - ja ir
zināms, ka nepieciešamais produkts būs rastrs, tad varbūt nav
nepieciešams investēt piepūli vektordatu dalīšanā, ja vien nedalītu
ievades datu rasterizēšana ir iespējama un jēgpilna. Protams, daudz
nosaka arī iegūstamā rezultāta veids - gan pēc būtības, gan saistībā ar
datu veidiem un objektu klasēm un veidiem.

Ja procesēšanas rezultātā ir iegūstama, piemēram, individuāla vērtība
telpiskajai daļai vai vērtību vektors telpiskās daļas iekšienē esošām
zonām, tad laba doma ir šo rezultātu saglabāt/uzturēt kā vērtību vai
vektoru ar telpiskās daļas vai tās iekšienē esošās zonas unikālo
identifikatoru, kuru pēc apstrāde pievienot nepieciešamajai datubāzei,
piemēram, izmantojot {tidyverse}
[*join*](https://r4ds.hadley.nz/joins.html) vai varbūt pat uzreiz
operējot SQL. Ja iepriekš ir nodrošināta (un pārbaudīta) secība, var pat
vienkārši izmantot vērtību ievietošanu laukā.

Ja radītais rezultāts ir apstrādātas ģeometrijas, kuras nepieciešams
apvienot vienā slānīt, tas, R izmantojot *simple features* paradigmu,
piemēram, {sf}, ik ģeometrija ir datubāzes ieraksts. Ja datubāzēm/datu
tabulām ir vienādi lauki un to kodējami, tās vienkārši ir
apvienojamas/sapludināmas ar bāzes `rbind`. Ja dažādajās apvienojamajās
daļās esošie lauki ir ar nolūku atšķirīgi (nevis kļūda), izmantojama
{tidyverse} `bind_rows`.

Tomēr, ja rezultātā nepieciešams rastrs, varbūt pat visa datubāzu
savienošana vai vērtību ievietošana ir pakārtota rastra sagatavošanai,
tad labāka doma varētu būt uzreiz aprēķinus veikt vai vismaz izvadīt
rastra formātā kā apstrādes lapas. Šos atsevišķos failus, kas ir
uztveramas kā ģeogrāfijas atlanta lapas, ir iespējas apvienot, piemēram,
izmantojot {terra} funkcijas `merge` un `mosaic`. Pirmā ir atrāka, bet,
ja lapu robežas pārklājas, to pārklāšanās vietā tiks izmantotas pirmās
(funkcijas argumentu secības vai ielasīšanas kārtībā) lapas vērtības.
Savukārt otrajā ir iespējams definēt dažādas pieejas attiecībā pret
šūnām, kurām ir vērtības vairāk kā vienā lapā. Kā vēlviena alternatīva
ir minams [virtuālais rastrs
(VRT)](https://gdal.org/en/stable/drivers/raster/vrt.html), kas {terra}
ir veidojams ar funkciju
[`vrt`](https://rdrr.io/cran/terra/man/vrt.html). Virtuālais rastrs ir
sevišķi parocīgs plašākās ģeoprocesēšanas plūsmās, kas balstās rastra
apstrādē - tas veido nosacītu katalogu ar aprakstošo informāciju un
ātras/ētras lasīšanas ceļiem par ik lapu (jeb flīzi un Zemes novērošanas
sistēmu datos nereti arī - ainu) - dodot uzdevumu VRT tas tiks nodots ik
lapā. Tomēr ir vēl kāda priekšrocība - virtuālā rastra sagatavošana un
saglabāšana GeoTIFF ir ekvivalents `merge`, bet daudzkārt ātrāks un
mazāk RAM prasošs uzdevums. Bet ir jāņem vērā, ka tas ir `merge`, nevis
`mosaic` līdzinieks - noslēdzošajām lapām vajadzētu būt sagatavotām tā,
lai to malas precīzi saskaras vai vispārīgā situācijā - to ielasīšanas
secībai nebūtu nozīme. To ir vieglāk nodrošināt, ja dažādie
starpproduktus un produktus raksturojošie rastri ir savstarpēji
saistīti - lielāks šūnas izmērs ir iegūstams reizinot mazākas šūnas un
to veidoto zonu malas sakrīt. Tāpat arī šīm zonu robežām būtu jāsakrīt
gan vektordatos, gan rastra datos. Šis, tātad, ir pakārtots
sagatavošanās darbiem pašā projekta sākumā - ir sagaidāmi nopietni
izaicinājumi visā darba gaitā (vienā brīdī vienkāršāk un jēgpilnāk var
būt atgriezties pie šīm definīcijām un visu pārstrādāt), ja par to nebūs
padomāts jau sākumā un uzdevumos nebūs iestrādāti telpiskās saderības
nodrošinājumi.

Kā jau vairākkārt minēts, šajā projektā daudz strādāsim ar rastru,
vairumu starpaprēķinu veiksim rastrā un saglabāsim kā rastru, arī daudzi
modelēšanas tiešo uzdevumu norisināsies rastrā un kā rastrs tiks
saglabāts vairums tiešo rezultātu. Tomēr vektordati neizbēgami būs
iesaistīti dažādos priekšapstrādes un pēcapstrādes soļos, tādēļ ir
nepieciešams nostiprināt intuīciju arī darbā ar tiem.

## Uzdevums

Šim uzdevumam ir kopīga sagatavošanās, tomēr tā sekojošās daļas ved pie
viena un tā paša rezultāta, izmantojot dažādus vingrinājumus. Tāds arī
ir tā uzdevums - vingrināt izpratni un intuīciju komandu rindās, R un
darbā ar telpiskiem datiem.

Šim uzdevumam izmantojami pirmajā uzdevumā apvienotie Valsts meža
dienesta Centra virsmežniecības dažādo nodaļu apvienotie MVR dati un
projekta [Zenodo
repozitorijā](https://zenodo.org/communities/hiqbiodiv/records?q=&l=list&p=1&s=10&sort=newest)
pieejamie references dati (gan vektoru, gan rastra). Sakot darbu, brīvi
izvēlieties (vienalga kā, vienalga kurā programmā) četras blakus esošas
`tks93_50km` karšu lapas, kurās ir Centra virsmežniecības MVR dati par
mežaudzēm - šīs lapas ievietojiet atsevišķā R objektā un izmantojiet
turpmākam darbam kā neatkarīgas telpiskās daļas iteratīvā aprēķinu
procesā.

Pirms ķeršanās pie uzdevuma, pārskatiet ceturtā uzdevuma funkciju
`mana_funkcija` tā, lai izvades rastrs (priežu mežaudzu īpatsvars (no
kopējās platības) 100 m šūnā) aptvertu tikai to teritoriju, kas pilnībā
iekļaujas iterētajā telpā (kartes lapā) to pilnībā aptverot. Apsveriet
ievades parametru sagatavošanu pirms ievietošanas funkcijā kā
alternatīvu funkcijas modificēšanai.

Salīdziniet procesēšanas laiku katrā uzdevuma daļā (sekojošajā
uzskaitījumā) no telpas dalīšanas uzsākšanas (ieskaitot) līdz
noslēdzošajam rastram (ieskaitot).

1.  Pirmais apakšuzdevums.

1.1. solis: izmantojot *spatial join* saistiet MVR datus ar izvēlētajām
karšu lapām (šis ir sākums laika mērogošanai). Kāds ir objektu skaits
pirms un pēc savienošanas? Apskatiet katrai kartes lapai piederīgos
objektus - vai tie atrodas tikai kartes lapas iekšienē, vai ir iekļauti
visi objekti, kas ir uz kartes lapas robežām?

1.2. solis: iteratīvā ciklā izmantojiet pārskatīto funkciju no uzdevuma
sākuma, lai saglabātu katras karšu lapas rezultātu savā GeoTIFF failā,
kura nosaukums ir saistāms ar šo apakšuzdevumu un individuālo lapu. Kā
izskatās šūnu aizpildījums pie karšu lapu malām? Vai ir saglabājušies
objekti ārpus kartes lapām?

1.3. solis: apvienojiet iteratīvajā ciklā radītos GeoTIFF failus vienā
slānī, kura nosaukums ir saistāms ar apakšuzdevumu (laika mērogošanas
beigas). Vai apvienotajā slānī ir redzamas karšu lapu robežas? Kā
vērtības sakrīt ar citiem apakšuzdevumiem?

2.  Otrais apakšuzdevums.

2.1. solis: sāciet iteratīvo ciklu, kurā, izmantojot *clipping*
iegūstiet MVR datus ar apstrādājamo kartes lapu (šis ir sākums laika
mērogošanai). Kāds ir objektu skaits katrā kartes lapā, kā tas saistās
ar iepriekšējo apakšuzdevumu? Ārpus cikla apskatiet katrai kartes lapai
piederīgos objektus - vai tie atrodas tikai kartes lapas iekšienē, vai
ir iekļauti visi objekti, kas ir uz kartes lapas robežām?

2.2. solis: izmantojiet pārskatīto funkciju no uzdevuma sākuma, lai
saglabātu katras karšu lapas rezultātu savā GeoTIFF failā, kura
nosaukums ir saistāms ar šo apakšuzdevumu un individuālo lapu. Kā
izskatās šūnu aizpildījums pie karšu lapu malām? Vai ir saglabājušies
objekti ārpus kartes lapām?

2.3. solis: apvienojiet iteratīvajā ciklā radītos GeoTIFF failus vienā
slānī, kura nosaukums ir saistāms ar apakšuzdevumu (laika mērogošanas
beigas). Vai apvienotajā slānī ir redzamas karšu lapu robežas? Kā
vērtības sakrīt ar citiem apakšuzdevumiem?

3.  Trešais apakšuzdevums.

3.1. solis: sāciet iteratīvo ciklu, kurā, izmantojot *spatial filtering*
iegūstiet MVR datus ar apstrādājamo kartes lapu (šis ir sākums laika
mērogošanai). Kāds ir objektu skaits katrā kartes lapā, kā tas saistās
ar iepriekšējo apakšuzdevumu? Ārpus cikla apskatiet katrai kartes lapai
piederīgos objektus - vai tie atrodas tikai kartes lapas iekšienē, vai
ir iekļauti visi objekti, kas ir uz kartes lapas robežām?

3.2. solis: izmantojiet pārskatīto funkciju no uzdevuma sākuma, lai
saglabātu katras karšu lapas rezultātu savā GeoTIFF failā, kura
nosaukums ir saistāms ar šo apakšuzdevumu un individuālo lapu. Kā
izskatās šūnu aizpildījums pie karšu lapu malām? Vai ir saglabājušies
objekti ārpus kartes lapām?

3.3. solis: apvienojiet iteratīvajā ciklā radītos GeoTIFF failus vienā
slānī, kura nosaukums ir saistāms ar apakšuzdevumu (laika mērogošanas
beigas). Vai apvienotajā slānī ir redzamas karšu lapu robežas? Kā
vērtības sakrīt ar citiem apakšuzdevumiem?

4.  Ceturtais apakšuzdevums.

4.1. solis: ar sevis izvēlētu pieeju atlasiet mežaudzes, kuras pieder
kādai no izvēlētajām karšu lapām (sākums laika mērogošanai).

4.2. solis: ieviesiet funkciju atlasītajām mežaudzēm bez iteratīvā
procesa, bet nodrošinot rezultējošā rastra aptveri tikai izvēlētajām
kartes lapām (beigas laika mērogošanai). Kā vērtības sakrīt ar citiem
apakšuzdevumiem?

5.  Piektais apakšuzdevums.

5.1. solis: ieviesiet funkciju visai MVR informācijai (sākums laika
mērogošanai).

5.2. solis: no rezultējošā slāņa izgrieziet tikai to daļu, kas attiecas
uz izvēlētajām karšu lapām un beidziet laika mērogošanu. Ja šis solis ir
jau iestrādāts funkcijā, tad mērogojiet tikai pašu funkciju.

6.  Setais - jaudīgiem datoriem un dažādu pieeju izmēģināšanai. Saistiet
    atbilstošo mežaudžu informāciju ar 100 m tīklu no projekta
    repozitorija un izveidojiet projekta repozitorijā esošajam 100 m
    rastra tīklam atbilstošu rastru pēc šūnu novietojuma un vērtībām.
    Kad iegūstat salīdzināmu rezultātu, mērogojiet tā izpildei
    nepieciešamo laiku. Kā vērtības sakrīt ar citiem apakšuzdevumiem?

Kura no apakšuzdevumu versijām šķiet efektīvākā, kādēļ?

Ja kādā no šī uzdevuma daļām nevarat sagaidīt rezultātu, jo aprēķini ir
pārāk lēni, vai Jūsu iekārtai nepietiek resursu uzdevuma veikšanai,
saglabājiet izmantotās komandu rindas un raksturojiet padošanās
apstākļus. Piemēram, laiku pēc kāda pātraucāt procesu.

## Padomi

Šis uzdevums ir cieši saistīts ar R pakotņu {sf} un {terra} lietošanu.
Tās ir labi dokumentētas un plaši lietotas, kas nozīmē informācijas
bagātību arī internetā, nevis tikai šī uzdevuma premisē un visa
repozitorija pamata aprakstā piedāvātajos resursos. Protams, nav
jāizmanto tikai šīs divas pakotnes vai pat vispār tieši šīs. Tomēr darbs
ar tām varētu būt vieglāk lasāms un intuitīvāks, jo sevišķi klasisko GIS
lietotājiem. Šie piebilde nav aizliegums uzdot jautājumus, drīzāk tā ir
skaidrojums padomu trūkumam.
