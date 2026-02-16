12.2 uzdevums
================
Jekaterīna Butkeviča
2026-02-16

## Izvēlētā suga

Manas interpretācijas novērtējumam — sugas ekoloģiskais konteksts.

Manis izvēlētā suga ir parastais sīksamtenis *Coenonympha pamphilus*
(COEPAM).

**Apdzīvo**: graudzālēm bagātus biotopus — galvenokārt pļavas un dažādus
zālājus; retāk sastopams mežmalās, gar meža ceļiem, izcirtumos u.
c. Bieži arī izteikti sausākās vietās (*cerambyx.lv*). Apdzīvo arī
atklātus biotopus, piemēram, ekstensīvi apsaimniekotas pļavas, purvus,
barības vielām nabadzīgus zālājus un ganības. Retāk to var atrast arī
lielākos mežu izcirtumos. Agrāk suga bija bieži sastopama
lauksaimniecības laukos, to malās un zālājos gar taciņām; mūsdienās tur
sastopama reti intensifikācijas dēļ (*Wolfgang Wagner*).

**Kāpuru barības augs**: *Dactylis* spp., *Festuca* spp., *Nardus* spp.,
*Poa* spp. (<a href="https://lepidoptera.eu/"
class="uri"><em>https://lepidoptera.eu/</em></a>).

**Apdraudējuma faktori**: pļaušana vairāk nekā divas reizes gadā, purvu
susināšana, intensīva zālāju apsaimniekošana (*Wolfgang Wagner*).

<figure>
<img src="Coenonympha_pamphilus.jpg" width="402"
alt="1. attēls. Coenonympha pamphilus. Valais, Switzerland, July 2022" />
<figcaption aria-hidden="true">1. attēls. Coenonympha pamphilus. Valais,
Switzerland, July 2022</figcaption>
</figure>

Novērojumu izvietojums ir šāds:

<figure>
<img src="points.png"
alt="2. attēls. Coenonympha pamphilus novērojumu izvietojums" />
<figcaption aria-hidden="true">2. attēls. Coenonympha pamphilus
novērojumu izvietojums</figcaption>
</figure>

Ar oranžo krāsu ir attēloti novērojumi, kas izmantoti testa kopai,
savukārt ar zaļo krāsu — treniņa kopas izveidei.

## Izveidotie modeļi:

1.  COEPAM_bez_bias_v0 – modelis bez piepūles noviržu kontroles.

2.  COEPAM_telp_ret_v0 – ar telpisko retināšanu, saglabājot vienu
    novērojumu ik 1 km šūnā, bet bez citas kontroles.

Modeļi bez telpiskās retināšanas, viens kopīgs piepūles noviržu
kontroles slānis vienādi izmantojot visas sugas (jeb visai sugu grupai
“Dienastauriņi”); ar apakšvariantiem:

3.  COEPAM_bez_telp_ret_kopig_piep_bez_lim_v0 – bez apakšgala
    limitācijas;

4.  COEPAM_bez_telp_ret_kopig_piep_10_v0 – apakšgalu limitējot 10% no
    slāņa vidējās vērtības;

5.  COEPAM_bez_telp_ret_kopig_piep_mean_v0 – apakšgalu limitējot pie
    slāņa vidējās vērtības;

Modeļi bez telpiskās retināšanas, piepūles noviržu kontroles slāņa no
sugām atkarīgo daļu veidojot ar sezonālo reģistrāciju pārklāšanās
svarošanu; ar apakšvariantiem:

6.  COEPAM_bez_telp_ret_sezon_piep_bez_lim_v0 – bez apakšgala
    limitācijas;

7.  COEPAM_bez_telp_ret_sezon_piep_10_v0 – apakšgalu limitējot 10% no
    slāņa vidējās vērtības;

8.  COEPAM_bez_telp_ret_sezon_piep_mean_v0 – apakšgalu limitējot pie
    slāņa vidējās vērtības;

Labākais modelis, izslēdzot neloģiskos EGV:

9.  COEPAM_bez_telp_ret_kopig_piep_bez_lim_v1 – izslēdzot EGV, kas rada
    problēmas kartē;
10. COEPAM_bez_telp_ret_kopig_piep_bez_lim_v2 – izslēdzot gan EGV, kas
    rad aproblēmas kartē, gan EGV ar negatīvam standrat novirzerm;
11. COEPAM_bez_telp_ret_kopig_piep_bez_lim_v3 – izslēdzot EGV ar
    negatīvam standratnovirzēm.

Sugas ekoloģija neļauj veikt filtrešanu vides izmaiņam. Svarošana pēc
Eiropas Savienībā aizsargājamiem biotopiem netika veikta, jo nav pamata
uzskatīt, ka novērojumu veikšanas piepule grupējas saskaņoti ar tiem
(konsultējos ar K. Vilku).

## Modeļu salīdzinājums

Kopumā par izveidotajām kartēm. Numurs attēlā atbilst modeļa numuram
sarakstā. Attēlā ir ievietotas pilno karšu .png versijas, savukārt
detalizētāk tika salīdzinātas izstieptās versijas.

Visās kartēs izteikti redzami lielo pilsētu centri (īpaši Rīgas centrs).
Vismazāk tas novērojams 3. modelī
(COEPAM_bez_telp_ret_kopig_piep_bez_lim_v0). Visur izceļas arī ceļi.
Pārāk liela piemērotība tiek prognozēta lieliem ūdenstilpēm, piemēram,
Rīgas HES ūdenskrātuvei vai Ķīšezeram.

Šī suga ir izteikts ģeneralists, un es būtu gatava pieņemt, ja kā
piemērota parādītos lielāka daļa Latvijas teritorijas, jo arī pilsētas,
ja tās nav tik asfaltētas un samākslotas kā Rīga, var būt piemērotas.

<figure>
<img src="salidzinajums.png" width="735"
alt="3. attēls. Prognozētās biotopu piemērotības kartes." />
<figcaption aria-hidden="true">3. attēls. Prognozētās biotopu
piemērotības kartes.</figcaption>
</figure>

Turpmāk modeļu salīdzināšanai izmantosšu šos divus attēlus:

<figure>
<img src="latgale.png" alt="4. attēls. Lauku ainava Latgalē." />
<figcaption aria-hidden="true">4. attēls. Lauku ainava
Latgalē.</figcaption>
</figure>

<figure>
<img src="kurzeme.png" alt="5. attēls. Lauku ainava Kurzemē." />
<figcaption aria-hidden="true">5. attēls. Lauku ainava
Kurzemē.</figcaption>
</figure>

### 1. COEPAM_bez_bias_v0

Karte ir slikta. Prognozētā vispiemērotākā vide ir pilsētās. Ķīšezers ir
piemērotāks nekā lauku ainava. Parāk daudz nepiemēroto teritoriju.
Nezinu, vai ir vērts analizēt metrikas. Iespējams, darīšu tā, ka
apskatīsim visu, kas ir, lai neslēptu no jums rezultātus, bet neredzu
lielu jēgu analizēt ROC u.c., jo, cik atceros no statistikas kursa,
metrikas var radīt dažādus rezultātus, un tās vajadzētu skatīt tikai
modelim, kas pats par sevi ir derīgs. Šis modelis tāds nav.

<figure>
<img src="12_2_uzdevums/1_bez_bias_kontr/PicHSmap_COEPAM_bez_bias.png"
width="735"
alt="7. attēls. Prognozētās biotopu piemērotības karte šim modelim." />
<figcaption aria-hidden="true">7. attēls. Prognozētās biotopu
piemērotības karte šim modelim.</figcaption>
</figure>

Latvija neatrodas šīs sugas areāla robežās, tādēļ man nav skaidrs, kāpēc
līdzīga ainava izpaužas tik atšķirīgi Kurzemē un Latgalē. Izceļas ceļi,
taču tas joprojām ir salīdzinoši piemērota vide.

<figure>
<img src="bez_bias_tiles.png"
alt="8. attēls. Prognozētā biotopu piemērotība lauku ainavā Kurzemē un Latgalē." />
<figcaption aria-hidden="true">8. attēls. Prognozētā biotopu piemērotība
lauku ainavā Kurzemē un Latgalē.</figcaption>
</figure>

ROC līkne testa kopai ir ļoti līdzīga treniņu kopai. Testa kopas AUC
vērtība 0,862 liecina par gandrīz izcilu modeli.

<figure>
<img src="12_2_uzdevums/1_bez_bias_kontr/PicROC_COEPAM.png"
alt="9. attēls. ROC līkne." />
<figcaption aria-hidden="true">9. attēls. ROC līkne.</figcaption>
</figure>

### 2. COEPAM_telp_ret_v0

Šis modelis reķinājas visilgāk. Karte izskatas nedaudz labāk - pilsētās
prognozēta piemērotība kļluva nedaudz zemāka, bet joprojām ir augsta.
Lielie ūdēņi kļūva mazāk piemēroti, kas ir labi. Palielinājas vidēja
piemērotība lauku ainavā (gan Kurzemē, han Latgales pusē).

<figure>
<img src="12_2_uzdevums/2_ar_telp_ret/PicHSmap_COEPAM_telp_ret.png"
width="735"
alt="10. attēls. Prognozētās biotopu piemērotības karte šim modelim." />
<figcaption aria-hidden="true">10. attēls. Prognozētās biotopu
piemērotības karte šim modelim.</figcaption>
</figure>

Ceļi sāka izcelties vairāk, kas vairs nav apmieronoši. Turklāt to
piemērotība ir izteikti pārvertēta Kuzemē, savukart Latgales pusē tie
kļust izteikti sarkanie tikai blakus pilsētam.

<figure>
<img src="telp_ret_tiles.png"
alt="11. attēls. Prognozētā biotopu piemērotība lauku ainavā Kurzemē un Latgalē." />
<figcaption aria-hidden="true">11. attēls. Prognozētā biotopu
piemērotība lauku ainavā Kurzemē un Latgalē.</figcaption>
</figure>

Izcīli.

<figure>
<img src="12_2_uzdevums/2_ar_telp_ret/PicROC_COEPAM.png"
alt="12. attēls. ROC līkne." />
<figcaption aria-hidden="true">12. attēls. ROC līkne.</figcaption>
</figure>

### 3. COEPAM_bez_telp_ret_kopig_piep_bez_lim_v0

Šobrīd labāka karte. Pilsētas joprojām izceļas, tomēr mazāk nekā visās
parejas kartēs. Es arī neredzu baigas problēmas ar cēļiem – tie neradas
kā piemērotāka vieta, bet tikai vīdēji piemērota, kas atbilst
patiesībai.

<figure>
<img
src="12_2_uzdevums/3_bez_telp_ret_kopig_piep_bez_lim/PicHSmap_COEPAM_kop_bez_lim.png"
width="735"
alt="13. attēls. Prognozētās biotopu piemērotības karte šim modelim." />
<figcaption aria-hidden="true">13. attēls. Prognozētās biotopu
piemērotības karte šim modelim.</figcaption>
</figure>

Man patīk, ka ainavas elementu piemērotība ir vienāda Kurzemē un
Latgalē.

<figure>
<img src="kop_bez_lim_tiles.png"
alt="14. attēls. Prognozētā biotopu piemērotība lauku ainavā Kurzemē un Latgalē." />
<figcaption aria-hidden="true">14. attēls. Prognozētā biotopu
piemērotība lauku ainavā Kurzemē un Latgalē.</figcaption>
</figure>

Rezultāti ir lidzīgi treniņā un testa kopās. Tomēr abu ;līkņu izliekums
ir mazāks, kas liecīna, kas šis modelis nav tik labāks par nejaušo
prognozēšanu, nekā iepriekšējie.

<figure>
<img
src="12_2_uzdevums/3_bez_telp_ret_kopig_piep_bez_lim/PicROC_COEPAM.png"
alt="15. attēls. ROC līkne." />
<figcaption aria-hidden="true">15. attēls. ROC līkne.</figcaption>
</figure>

### 4. COEPAM_bez_telp_ret_kopig_piep_10_v0

Otrais labākasi modelis, bet ar mazāko lokālo piemērotības
heteraģenitāti. Ļoti daudz vidēji piemēroto teritoriju, kas būtu
atbilstošs modelētas sugas ekoloģijai. Pilsētas joprojām izceļas.

<figure>
<img
src="12_2_uzdevums/4_bez_telp_ret_kopig_piep_10/PicHSmap_COEPAM_kop_10.png"
width="735"
alt="15. attēls. Prognozētās biotopu piemērotības karte šim modelim." />
<figcaption aria-hidden="true">15. attēls. Prognozētās biotopu
piemērotības karte šim modelim.</figcaption>
</figure>

Parāk augsta piemērotība apdzīvotos punktos. Tagad domāju, ka ja jau
sugai nav tiešas saistības ar pilsētam (tikai kā nepiemērota vide,
piesarņojuma un vajadzīgas veģetācijas trūkuma dēļ) varbūt ir vērts
vispar izslegt tos no modeļa.

<figure>
<img src="kop_10_tiles.png"
alt="16. attēls. Prognozētā biotopu piemērotība lauku ainavā Kurzemē un Latgalē." />
<figcaption aria-hidden="true">16. attēls. Prognozētā biotopu
piemērotība lauku ainavā Kurzemē un Latgalē.</figcaption>
</figure>

Tas pats, kas ar iepriekšējo modeli.

<figure>
<img src="12_2_uzdevums/4_bez_telp_ret_kopig_piep_10/PicROC_COEPAM.png"
alt="17. attēls. ROC līkne." />
<figcaption aria-hidden="true">17. attēls. ROC līkne.</figcaption>
</figure>

### 5. COEPAM_bez_telp_ret_kopig_piep_mean_v0

Man nav nekā jauna, ko pateikt par šo kārti. Parāk liela piemērotība
pilsētās.

<figure>
<img
src="12_2_uzdevums/5_bez_telp_ret_kopig_piep_mean/PicHSmap_COEPAM_kop_mean.png"
width="735"
alt="18. attēls. Prognozētās biotopu piemērotības karte šim modelim." />
<figcaption aria-hidden="true">18. attēls. Prognozētās biotopu
piemērotības karte šim modelim.</figcaption>
</figure>

Patik, ka visas zāļās vietas ir +- minus piemērotas, kā arī piemērotības
gradients mežmalā.

<figure>
<img src="kop_mean_tiles.png"
alt="19. attēls. Prognozētā biotopu piemērotība lauku ainavā Kurzemē un Latgalē." />
<figcaption aria-hidden="true">19. attēls. Prognozētā biotopu
piemērotība lauku ainavā Kurzemē un Latgalē.</figcaption>
</figure>

Bišķin lielāks izliekums abam līknem nekā iepriekšēja modelī.

<figure>
<img
src="12_2_uzdevums/5_bez_telp_ret_kopig_piep_mean/PicROC_COEPAM.png"
alt="20. attēls. ROC līkne." />
<figcaption aria-hidden="true">20. attēls. ROC līkne.</figcaption>
</figure>

### 6. COEPAM_bez_telp_ret_sezon_piep_bez_lim_v0

Kopumā viss ir tas pats. Šeit, salīdzinot ar iepriekšējo modli ir zemāka
piemērotība ūdeņiem.

<figure>
<img
src="12_2_uzdevums/6_bez_telp_ret_sezon_piep_bez_lim/PicHSmap_COEPAM_sez_bez_lim.png"
width="735"
alt="21. attēls. Prognozētās biotopu piemērotības karte šim modelim." />
<figcaption aria-hidden="true">21. attēls. Prognozētās biotopu
piemērotības karte šim modelim.</figcaption>
</figure>

Taču piemerotības klasīfikācij neātšķiras izteikti starp Rietum un
AustrumLatviju, kas priecē.

<figure>
<img src="sezon_bez_lim_tiles.png"
alt="22. attēls. Prognozētā biotopu piemērotība lauku ainavā Kurzemē un Latgalē." />
<figcaption aria-hidden="true">22. attēls. Prognozētā biotopu
piemērotība lauku ainavā Kurzemē un Latgalē.</figcaption>
</figure>

Tas ir pirmais no modeliem, kam testa kopas rezultāti tik atšķiras no
treniņa kopas.

<figure>
<img
src="12_2_uzdevums/6_bez_telp_ret_sezon_piep_bez_lim/PicROC_COEPAM.png"
alt="23. attēls. ROC līkne." />
<figcaption aria-hidden="true">23. attēls. ROC līkne.</figcaption>
</figure>

### 7. COEPAM_bez_telp_ret_sezon_piep_10_v0

Karte kopumā ir tada pati, ka iepriekšējam moelim, tikai amzāk
kontrasta, t.i. ainava netiek klasificēta tik strikti, vērtības virzas
uz vairāk vai mazāk piemērots. Mazāk piemēotība Vidzemē un
Dienvidkurzemē.

<figure>
<img
src="12_2_uzdevums/7_bez_telp_ret_sezon_piep_10/PicHSmap_COEPAM_sez_10.png"
width="735"
alt="24. attēls. Prognozētās biotopu piemērotības karte šim modelim." />
<figcaption aria-hidden="true">24. attēls. Prognozētās biotopu
piemērotības karte šim modelim.</figcaption>
</figure>

Tāda pati problēma ar apdzīvotam vietām.

<figure>
<img src="sezon_10_tiles.png"
alt="25. attēls. Prognozētā biotopu piemērotība lauku ainavā Kurzemē un Latgalē." />
<figcaption aria-hidden="true">25. attēls. Prognozētā biotopu
piemērotība lauku ainavā Kurzemē un Latgalē.</figcaption>
</figure>

<figure>
<img src="12_2_uzdevums/7_bez_telp_ret_sezon_piep_10/PicROC_COEPAM.png"
alt="26. attēls. ROC līkne." />
<figcaption aria-hidden="true">26. attēls. ROC līkne.</figcaption>
</figure>

### 8. COEPAM_bez_telp_ret_sezon_piep_mean_v0

<figure>
<img
src="12_2_uzdevums/8_bez_telp_ret_sezon_piep_mean/PicHSmap_COEPAM_sez_mean.png"
width="735"
alt="27. attēls. Prognozētās biotopu piemērotības karte šim modelim." />
<figcaption aria-hidden="true">27. attēls. Prognozētās biotopu
piemērotības karte šim modelim.</figcaption>
</figure>

<figure>
<img src="sezon_mean_tiles.png"
alt="28. attēls. Prognozētā biotopu piemērotība lauku ainavā Kurzemē un Latgalē." />
<figcaption aria-hidden="true">28. attēls. Prognozētā biotopu
piemērotība lauku ainavā Kurzemē un Latgalē.</figcaption>
</figure>

<figure>
<img
src="12_2_uzdevums/8_bez_telp_ret_sezon_piep_mean/PicROC_COEPAM.png"
alt="29. attēls. ROC līkne." />
<figcaption aria-hidden="true">29. attēls. ROC līkne.</figcaption>
</figure>

### 9. EGV atmēšana COEPAM_bez_telp_ret_kopig_piep_bez_lim

Kopumā visi modeļi nebijqa apmierinoši. Tājos variējas atsevisķu aianvas
elementu attēlojuma veiksmīgums, tomēr visos pastāvēja problēmas ar
apdzīvotājiem punktiem. Tomēr labakais no visiem man likas
COEPAM_bez_telp_ret_kopig_piep_bez_lim_v0 modelis, tādēļ atkārtoju to
trījos apakšvariantos:

- COEPAM_bez_telp_ret_kopig_piep_bez_lim_v1 – izslēdzot EGV, kas rada
  problēmas kartē;

- COEPAM_bez_telp_ret_kopig_piep_bez_lim_v2 – izslēdzot gan EGV, kas rad
  aproblēmas kartē, gan EGV ar negatīvam standrat novirzerm;

- COEPAM_bez_telp_ret_kopig_piep_bez_lim_v3 – izslēdzot EGV ar negatīvam
  standratnovirzēm.

#### COEPAM_bez_telp_ret_kopig_piep_bez_lim_v1

Šaja modelī izsledzu pazīmes, kas visvairāk radīja problēmas kartē -
apbūve. T.i. modeli tika izslēgti sēkojošie EGV: egv_413, “egv_414”,
“egv_415”, kas atbīlst apbūvēs platības dažādos mērogos. (EGV, kas
raksturo attālumu līdz apbūvej arī sākotnēji nebija ieļauts modeli).

<figure>
<img
src="12_2_uzdevums/9_kopig_piep_bez_lim_korekcijas/PicHSmap_COEPAM_v1.png"
alt="30. attēls. Prognozētās biotopu piemērotības karte šim modelim." />
<figcaption aria-hidden="true">30. attēls. Prognozētās biotopu
piemērotības karte šim modelim.</figcaption>
</figure>

Man liekas šajā kartē pilsētas tika prognozētas vairak piemērotas nekā
originālā (3. modelī). Tagad sāku domāt varbū problēma ir ar ceļiem
(Varbūt pilsēta turpina spidēt, neskatoties, ka modelis nav informēts
par apbūvi tiešī ceļu dēļ). Palaidu jaunus modeļus bez ceļiem.

<figure>
<img src="12_2_uzdevums/9_kopig_piep_bez_lim_korekcijas/v1_tiles.png"
alt=". attēls. Raznas ezers un Rīga" />
<figcaption aria-hidden="true">. attēls. Raznas ezers un
Rīga</figcaption>
</figure>

Man liekas, ka Raznas ezeram jābūt prognozētai daudz mazākai
piemērotībai. Arī Daugavai.

Tas ir pirmais modelis, kam ROC līkne testa kopai ir pārsniegusi to
treniņa kopai. Kaut ņemot vērā, ka abas kopas ir samērā nelielas,
manuprāt šis vai nu ir nejausa novirze, vai nu modelis ir nepielāgots un
parāk viekaršs - rezultātā tas nedarbojas pietiekami labi ne ar treniņa,
ne ar testa datiem.

<figure>
<img
src="12_2_uzdevums/9_kopig_piep_bez_lim_korekcijas/PicROC_COEPAM_v1.png"
alt=". attēļ.s ROC līkne." />
<figcaption aria-hidden="true">. attēļ.s ROC līkne.</figcaption>
</figure>

#### COEPAM_bez_telp_ret_kopig_piep_bez_lim_v2

Šajā modeli izslēdzu gan kartē traucējošo apbūvi, gan pazīmes, kuru
standartnovīrzei bija toskait arī negatīvas vērtības: EGV: egv_413,
egv_414, egv_415, egv_298, egv_424, egv_482, kas atbīlst apbūvēs
platības dažādos mērogos (EGV, kas raksturo attālumu līdz apbūvej arī
sākotnēji nebija ieļauts modeli), šaurlapju krājai analīzes šūnā (1 ha),
netaksēto mežu platības īpatsvaram 0,5 km ainavā, mediānai pēdējo gadu
ūdens satura veģetācijā indeksa (NDMI) vērtībai analīzes šūnā (1 ha).

<figure>
<img
src="12_2_uzdevums/9_kopig_piep_bez_lim_korekcijas/PicHSmap_COEPAM_v2.png"
alt=". attēls. Prognozētās biotopu piemērotības karte šim modelim." />
<figcaption aria-hidden="true">. attēls. Prognozētās biotopu
piemērotības karte šim modelim.</figcaption>
</figure>

Šajā kartē man patik, ak ūdeņi beidzos radās kā nepiemēroti. Tomēr Rīga
spīd.

<figure>
<img src="12_2_uzdevums/9_kopig_piep_bez_lim_korekcijas/v2_tiles.png"
alt=". attēls. Raznas ezers un Rīga" />
<figcaption aria-hidden="true">. attēls. Raznas ezers un
Rīga</figcaption>
</figure>

Gribētos bīšķiņ augstāko AUC vērību. Kopumā ir labi. Tas,ka līkne mījas
starp sevi, manuprāt norāda uz to, ka modelis ir parāk vienkarš un
nejaušibai novērojumu sadalījumā ir ietekmē uz rezultātu.

<figure>
<img
src="12_2_uzdevums/9_kopig_piep_bez_lim_korekcijas/PicROC_COEPAM_v2.png"
alt=". attēls. RoC līkne." />
<figcaption aria-hidden="true">. attēls. RoC līkne.</figcaption>
</figure>

No visiem izveidotājiem, šis modelis patik man visvairāk. Apskatīšu
iuzmantoto EGV sarakastu:

<figure>
<img
src="12_2_uzdevums/9_kopig_piep_bez_lim_korekcijas/PicVarImpVIFs_COEPAM_v2.png"
alt=". attēls." />
<figcaption aria-hidden="true">. attēls.</figcaption>
</figure>

<figure>
<img
src="12_2_uzdevums/9_kopig_piep_bez_lim_korekcijas/PicMargResp_COEPAM_v2.png"
alt=". attēls." />
<figcaption aria-hidden="true">. attēls.</figcaption>
</figure>

#### COEPAM_bez_telp_ret_kopig_piep_bez_lim_v3

Šajā modeli izslēdzu tikai EGV, kuru standartnovīrzei bija toskait arī
negatīvas vērtības: EGV: egv_413, egv_414, egv_415, egv_298, egv_424,
egv_482, kas atbīlst apbūvēs platības dažādos mērogos (EGV, kas raksturo
attālumu līdz apbūvej arī sākotnēji nebija ieļauts modeli), šaurlapju
krājai analīzes šūnā (1 ha), netaksēto mežu platības īpatsvaram 0,5 km
ainavā, mediānai pēdējo gadu ūdens satura veģetācijā indeksa (NDMI)
vērtībai analīzes šūnā (1 ha).

<figure>
<img
src="12_2_uzdevums/9_kopig_piep_bez_lim_korekcijas/PicHSmap_COEPAM_v3.png"
alt=". attēls. Prognozētās biotopu piemērotības karte šim modelim." />
<figcaption aria-hidden="true">. attēls. Prognozētās biotopu
piemērotības karte šim modelim.</figcaption>
</figure>

Šeit ūdeņi (īpašī ūdenstilpņu centri) kļūva vairāk piemēroti nekā
iepriekšējā modeli, kas nav labi. Kontrasts starp piemēroto un neitrālo
vīdi kļuva mazāks. Ārpsu šiem novērojumiem karte ir lidzīga iepriekšējam
modelim.

<figure>
<img src="12_2_uzdevums/9_kopig_piep_bez_lim_korekcijas/v3_tiles.png"
alt=". attēls. Raznas ezers un Rīga" />
<figcaption aria-hidden="true">. attēls. Raznas ezers un
Rīga</figcaption>
</figure>

Viss tas pats, ko rakstuju pie iepriekšēja modeli.

<figure>
<img
src="12_2_uzdevums/9_kopig_piep_bez_lim_korekcijas/PicROC_COEPAM_v3.png"
alt=". attēls. ROC līkne." />
<figcaption aria-hidden="true">. attēls. ROC līkne.</figcaption>
</figure>
