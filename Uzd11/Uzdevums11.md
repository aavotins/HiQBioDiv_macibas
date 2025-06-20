Vienpadsmitais uzdevums: sugai specifiski ekoģeogrāfiskie mainīgie
================

## Termiņš

Līdz ~~(2025-01-20)~~ ~~(2025-02-17)~~ ~~(2025-02-24)~~ **2025-03-03**
projekta *sharepoint* direktorijā “WP2_macibas” savā apakšdirektorijā
iesniedziet devitajā uzdevumā izvēlēto sugu “EGV aptaujas” un aizpildītu
eksperta izvēli modelēšanai nepieciešamajiem EGV.

Līdz ~~(2025-01-20)~~ ~~(2025-02-17)~~ ~~(2025-02-24)~~ **2025-03-10**
projekta *sharepoint* direktorijā “WP2_macibas” savā apakšdirektorijā
iesniedziet ekoģeogrāfisko mainīgo izvēles tabulu, kas aizpildīta par
vienu katra cita dalībnieka izvēlēto sugu, atbilstoši jūsu izpratnei par
“EGV aptaujā” rakstīto.

Līdz ~~(2025-01-20)~~ ~~(2025-02-17)~~ ~~(2025-02-24)~~ **2025-03-23**,
izmantojot
[fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo)
un [pull
request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request-from-a-fork)
uz zaru “Dalibnieki”, šī uzdevuma direktorijā pievienojot .Rmd vai .qmd
failu, kura nosaukums ir Uzd11\_\[JusuUzvards\], piemēram,
`Uzd11_Avotins.Rmd`, kas sagatavots izvadei github dokumentā (piemēram,
YAML galvenē norādot `output: rmarkdown::github_document`), un tā radīto
izvades failu, kurš demonstrē EGV sagatavošanu un to kopas izvēli ik
sugas modelēšanai.

## Premise

Šī repozitorija pamata aprakstā ieteiktajā literatūrā, kas papildināta
ar aktuālajām publikācijām, iepazīstieties ar galvenajiem konceptiem un
pieejām sugu izplatības un plašākā kontekstā - ekoloģiskajā modelēšanā.
Mēģiniet noskaidrot, kādas ir biežāk lietotās datu analīzes metodes un
pieejas sugu izplatības modelēšanā, kādi dati tām ir nepieciešami.
Saprotiet, ka ļoti svarīgas ir pietiekoši precīzi datētas un
ģeoreferencētas drošas klātbūtnes vietas, kuras spēj reprezentēt (vai
vismaz tiecās uz šo spēju) interesējošās sugas interesējošās dzīves
stadijas ekoloģisko nišu. Tomēr līdz ar novērojumiem, ir nepieciešami
tādi ekoģeogrāfiskie mainīgie (EGV), kuri šo nišu raksturo.

Dažādām datu analīzes spējām ir mainīgas prasības pret nepieciešamo
klātbūtnes vietu skaitu (vai novērojumu apjomu, viss kopā - izlases
apjoms) un ieguves dizainu, kā arī iespējām vienā modelī iekļajamo EGV
skaitam un tā saistībai ar izlases apjomu. Mēs šajā projektā galvenokārt
izmantosim maksimuma entropijas analīzi ar Maxent vai maxnet. Šī metode
sniedz regresijveida korelatīvu sugas klātbūtnes vietu pretstatīšanu
videi kopumā EGV vērtībās. Atkarībā no algoritma, maksimuma entropijas
analīze sniedz Puasona punktu procesam līdzīgu pieeju un rezultātu,
tomēr ļauj operēt ar relatīvi nelielu klātbūtnes vietu skaitu un
visnotaļ lielu EGV skaitu.

Neatkarīgi no izmantotās statistiskās modelēšanas pieejas, ja tās
rezultātu ir nepieciešams ekstrapolēt, izmantot prognostiski un
rezultātu skaidrojoši, modelī vienlaikus iekļautajiem regresoriem (EGV)
būtu jābūt savā starpā neatkarīgiem, katrā ziņā tādiem, kur jebkuru
vienu ar pietiekošu precizitāti nespēj prognozēt jebkāds apjoms pārējo.
Šajā projektā kā nosacījumu izmantosim VIF $\le$ 10, kuru aprēķināt un
iekļaut regresoru (rastra EGV) izvēlē R vidē piedāvā {usdm}. Vēl bez
tīri statistiskajiem nosacījumiem, lai modeļa rezultāti būtu vismaz kaut
cik skaidrojami, vienlaikus modelī nav iekļaujamas pazīmes, kuras
liecina par apstākļiem uz viena un tā paša gradienta (vienalga kā
vērsta, vienalga kuru tā daļu fokusējot).

Kā jebkurā statistiskajā kompleksā, visiem novērojumiem ir jābūt ar
vērtībām visos regresoros. Lai modeļa rezultātu varētu projicēt visai
interesējošajai teritorijai, visiem EGV ir jābūt reālām vērtībām visās
šo teritoriju raksturojošajās šūnās.

## Uzdevums

Pildot uzdevumu, domājiet arī par devīto un desmito uzdevumiem.

1.  Iepazīstieties ar projekta sanāksmju materiāliem (direktorija
    “Sanaksmes”), ja tajās nepiedalījāties, un direktorijā “WP1”
    esošajiem uzdevumu (Task1.1, Task1.2 un Task1.3) aprakstiem un
    sniegtajiem piemēriem.

2.  Sevis devītajā uzdevumā izvēlētajām sugām veiciet pilnu EGV izvēles
    procedūru, kas iekļauj gan “aptaujas” aprakstu, gan EGV tabulas
    aizpildīšanu. Sagatavotos materiālus līdz **2025-03-03** ievietojiet
    projekta *sharepoint* direktorijas “WP2_macibas” savā
    apakšdirektorijā.

3.  Izvēlieties pa vienai katra cita dalībnieka sugai, iepazīstieties ar
    tās aprakstu “aptaujā”. Līdz **2025-03-10** projekta *sharepoint*
    direktorijas “WP2_macibas” savā apakšdirektorijā iesniedziet
    ekoģeogrāfisko mainīgo izvēles tabulu, atbilstoši savai izpratnei
    par “EGV aptaujā” rakstīto. Sazinieties savā starpā, lai
    nodrošinātu, ka vienu cita dalībnieka sugu izvēlas tikai viens no
    jums.

4.  Līdz **2025-03-23** no sevis devītajā uzdevumā izvēlētajām sugām
    trūkstošajiem EGV izvēlieties pa vienam no sekojošajām vispārīgajām
    grupām un sagatavojiet tos izmantošanai analīzē. Sagatavošanas
    procedūru dokumentējiet iesniedzamajā md failā. Ja “savu sugu”
    ietvaros nav trūkstošu EGV, izvēlieties kādus no kopējiem. EGV
    grupas:

- klimatu/laika apstākļus raksturojošie EGV;

- augsnes īpašības raksturojošie EGV;

- augstas kvantitatīvās izšķirtspējas tikai analīzes šūnā esošie
  gradientu raksturojumi;

- EGV ainavas līmeņos, tos sagatavojot visiem mērogiem (analīzes šūna,
  500, 1250, 3000 un 10000 m ap analīzes šūnas centru).

5.  Līdz **2025-03-23** veiciet sākotnējā modelī iekļaujamo EGV (nevis
    tabulas aizpildīšanu, bet VIF $\le$ 10) izvēli ar divām pieejām un
    salīdziniet iegūtos rezultātus:

- izvēlieties EGV, kuri ir sugai sevišķi nozīmīgi, veiciet visu modelī
  iekļaujamo EGV izvēli tā, lai saglabātos šī obligātās pazīmes;

- veiciet automatizētu EGV izvēli tā, lai saglabātu pēc iespējas lielāku
  skaitu EGV izvēles tabulā par sugai nozīmīgiem atzīmētos.

## Padomi

Jau kopš devītā uzdevuma uzsvērti mācamies arī strādāt kā komanda.
Sazināmies, pārrunājam problēmas, nepieciešamības un iespējamos
risinājumus, izmantojot diskusiju forumu vai tiekoties.
