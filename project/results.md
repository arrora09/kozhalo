# Eredmények: népszerűség és hálózati központiság a Marvel-univerzumban

## Rövid válasz

**Kérdés:** mennyire adja vissza a gráf szerkezete, hogy kik a legnépszerűbb Marvel-szereplők?

**Válasz:** a gráf megtalálja a csoportot, de a sorrendet nem.

- A szavazási listák szereplőinek kb. 60–70%-a a gráf azonos hosszúságú toplistáján is rajta van, ami nagyságrendekkel több a véletlennél.
- A szavazási sorrenddel a korreláció csak közepes (0,29–0,59).
- A gráf a **jelenlétet** méri, a szavazás a **népszerűséget**. Ez magyarázza a legnagyobb eltéréseket is.

| Lista | Szereplők a gráfban | Átfedés | Spearman-korreláció | Véletlen átfedés |
|---|---|---|---|---|
| 2011, top 40 | 39 | 23–27 | 0,32–0,49 | 0,25 |
| 2019, top 40 | 37 | 22–25 | 0,39–0,56 | 0,25 |
| 2011, top 100 | 91 | 56–63 | 0,49–0,59 | 1,55 |
| 2019, top 50 | 46 | 31–32 | 0,29–0,47 | 0,39 |

Az intervallumok a hat gráfmérték legkisebb és legnagyobb értékét jelentik, nem hibahatárt.

## Hol mit találsz

| Lépés | `beadando.ipynb` fejezet | Fájl |
|---|---|---|
| 1. Tisztítás | Adatok beolvasása és tisztítása | `data/nodes.csv`, `data/edges.csv` |
| 2–5. A gráf leírása | Elemzés | |
| 6–7. Összevetés, top 40 | Összevetés: top 40 listák | `data/baseline_2011.csv`, `data/baseline_2019.csv` |
| 8. Összevetés, 2011 | Összevetés: 2011-es top 100 | `data/baseline_2011.csv` |
| 9. Összevetés, 2019 | Összevetés: 2019-es top 50 | `data/baseline_2019.csv` |

Az adatok leírása a `DATA.md`-ben, a feladat a `project_description.md`-ben van.

## 1. Adattisztítás és a gráf felépítése

**Hol nézd:** „Adatok beolvasása és tisztítása”, 0–6. rész.

**Mire jó:** a hibás bejegyzésekből hamis csúcsok lennének, az eltérő írásmódok pedig egy képregény hőseit több csúcsra osztanák szét. Mindkettő torzítaná a fokszámokat.

**Mit nézz:** a sorok és kódok számát minden lépés után, a végén pedig a „Bipartite: True” sort és a csúcsok, élek számát.

| Lépés | Eredmény |
|---|---|
| A két fájl összevetése | Nincs eltérés, szóköz vagy duplikátum |
| Nem képregény bejegyzések törlése | 10 sor (5 szövegtöredék, 5 szereplőnév) |
| Kódok egységesítése | 52 kód változott, a képregények száma 12 641-ről 12 592-re csökkent |
| Duplikátumok eldobása | 2 sor |
| Átnézendő kódok | 17 kód, ezeken nem változtattunk |

| Végső gráf | Érték |
|---|---|
| Hősök | 6439 |
| Képregények | 12 592 |
| Élek | 96 092 |
| Bipartit sűrűség | 0,0012 |
| Komponensek | 20 |
| Óriáskomponens | 18 974 csúcs (99,7%) |

**Értelmezés:** a gráf nagyon ritka, de szinte teljesen egyben van, ezért az útvonalalapú mértékeket az óriáskomponensen érdemes számolni.

## 2. Fokszám-eloszlás

**Hol nézd:** „Elemzés” → „1. Fokszám-eloszlás”: egy táblázat és egy kétpaneles ábra.

**Mit csinál:** megszámolja, hány képregényben szerepel egy hős, és hány hős van egy képregényben, majd ezt egy ugyanakkora véletlen bipartit gráffal veti össze (`gnmk_random_graph`).

**Mire jó, miért:** megmutatja, hogy néhány szereplő uralja-e a gráfot. Ha igen, a fokszám önmagában is erős fontossági jel lehet, ezért ez az alapja a többi mértéknek.

**Mit nézz:**
- **Táblázat:** a medián és a maximum közti távolságot, valamint a felső 1% élarányát.
- **Ábra:** a csúcsok hányadát mutatja, amelyeknek legalább `k` élük van, log-log tengelyen. A kék (megfigyelt) vonal hosszan elnyúlik, a szürke (véletlen) hirtelen leesik. A távolságuk mutatja a különbséget.

| Mutató | Hősök | Hősök (véletlen) | Képregények | Képregények (véletlen) |
|---|---|---|---|---|
| Átlagos fokszám | 14,9 | 14,9 | 7,6 | 7,6 |
| Medián fokszám | 3 | 15 | 6 | 7 |
| Legnagyobb fokszám | 1577 | 32 | 111 | 20 |
| A felső 1% élaránya | 32,7% | 1,8% | 5,4% | 2,1% |

**Értelmezés:** a hősoldal rendkívül egyenlőtlen: a szereplők nagy része alig jelenik meg, néhányan pedig több százszor. Ezt nem a véletlen okozza. A képregényoldal is egyenlőtlen, de sokkal kevésbé.

**Korlát:** nem teszteltük, hogy az eloszlás hatványfüggvényt követ-e, csak azt, hogy eltér a véletlentől.

## 3. Bipartit centralitás

**Hol nézd:** „Elemzés” → „2. Bipartit centralitás”: a top 10 táblázat és a mértékek korrelációs táblája.

**Mire jó, miért:** a „fontos” többféleképpen definiálható, ezért több mértékkel számolunk. Ha csak egyet használnánk, nem tudnánk, hogy az eredmény a fogalomból vagy a választott mértékből következik.

| Mérték | Mit mér | Mire jó | Mire figyelj |
|---|---|---|---|
| Fokszám | Hány képregényben szerepel a hős | Egyszerű, közvetlen jelenlét | Egy rövid szereplés ugyanannyit ér, mint a főszerep |
| Közelség | Hány lépésre van mindenki mástól | A gráf közepén lévő szereplőket találja meg | Sűrű gráfon az értékek közel esnek egymáshoz |
| Köztiség | Milyen gyakran esik legrövidebb utakra | Az összekötő szereplőket mutatja | 1000 mintacsúcsból becsült érték, nem pontos |
| Sajátvektor | Fontos csúcsokhoz kapcsolódás | A sűrű magot emeli ki | Csapatszereplőket erősen előnyben részesít |
| PageRank | A véletlen bolyongás aránya | Stabil, a fokszámhoz közeli, de a környezetet is figyelembe veszi | A fokszámmal erősen együtt mozog |

**Mit nézz:** hogy ugyanazok a nevek vannak-e minden oszlop tetején, és melyik mérték tér el. A korrelációs táblában a magas érték azt jelenti, hogy két mérték szinte ugyanazt rangsorolja.

| Hely | Fokszám | Közelség | Köztiség | Sajátvektor | PageRank |
|---|---|---|---|---|---|
| 1 | Spider-Man | Spider-Man | Spider-Man | Captain America | Spider-Man |
| 2 | Captain America | Captain America | Captain America | Iron Man | Captain America |
| 3 | Iron Man | Iron Man | Iron Man | Thing | Iron Man |
| 4 | Thing | Thing | Wolverine | Human Torch | Hulk |
| 5 | Thor | Mr. Fantastic | Hulk | Mr. Fantastic | Thing |

A mértékek korrelációja az összes hősön 0,54 és 0,95 között van. A legszorosabb a fokszám és a PageRank (0,95), a leggyengébb a közelség és a PageRank (0,54).

**Értelmezés:** a csúcson a mértékek többnyire egyetértenek. A sajátvektor-centralitás más szempontot követ: a Bosszúállók és a Fantasztikus Négyes sűrű magját emeli ki, nem a képregények számát.

## 4. Sorozatszintű összesítés

**Hol nézd:** „Elemzés” → „3. Sorozatszintű összesítés”: kiírt számok és toplisták.

**Mit csinál:** a képregénykód első része a sorozat (az `ASM 22` az `ASM` sorozathoz tartozik), és megszámolja, hány sorozatban szerepel egy hős.

**Mire jó, miért:** a fokszámot egy hosszú sorozat is felviheti. A sorozatok száma a kiadón belüli elérést méri, ezért jó kiegészítő rangsor. Ez lett a hatodik mérték az összevetésekben.

**Mit nézz:** hány hős marad egyetlen sorozatban, és kik érnek el a legtöbbet.

| Mutató | Érték |
|---|---|
| Sorozatok | 650 |
| Sorozat / hős | átlag 3,51, medián 1 |
| Csak egy sorozatban szereplő hős | 3655 |
| Legtöbb sorozat | Spider-Man 147, Captain America 141, Thing 134, Human Torch 125, Iron Man 124 |
| Legtöbb hős | `A` 718, `UX` 697, `FF` 512, `ASM` 480, `CA` 460 |

**Értelmezés:** a hősök több mint fele egyetlen sorozathoz kötődik. A legszélesebb elérésűek nagyjából ugyanazok, akik a fokszámrangsort is vezetik.

**Korlát:** a kötetek külön sorozatnak számítanak (`ASM` és `ASM2`), mert az adatból nem biztos, hogy összetartoznak.

## 5. Bipartit klaszterezés és redundancia

**Hol nézd:** „Elemzés” → „4. Bipartit klaszterezés és redundancia”: két táblázat.

**Mit csinál:**
- **Klaszterezés** (`dot` mód): mennyire fednek át egy hős képregényei a vele együtt szereplő hősök képregényeivel.
- **Redundancia:** a hős képregénypárjainak hányadában van egy másik közös hős is, tehát a pár a hős nélkül is összekötve maradna.

**Mire jó, miért:** a bipartit gráfban nincsenek háromszögek, ezért a szokásos klaszterezettség nem értelmezhető. Ezek a mértékek ugyanazt kérdezik: tényleg összetartó csoportok vannak, vagy ez csak a fokszámokból adódik? Ezért hasonlítunk egy ugyanolyan fokszámú konfigurációs modellhez.

**Mit nézz:** a megfigyelt és a modellbeli érték arányát, valamint a redundanciát a nagy hősök táblájában. A magas redundancia csapatszereplőt, az alacsony inkább szólószereplőt jelez.

| Mutató | Megfigyelt | Konfigurációs modell |
|---|---|---|
| Átlagos klaszterezettség | 0,157 | 0,042 |
| Átlagos redundancia | 0,878 | 0,239 |

| Hős | Fokszám | Redundancia |
|---|---|---|
| Spider-Man | 1577 | 0,351 |
| Captain America | 1334 | 0,373 |
| Iron Man | 1150 | 0,373 |
| Thing | 963 | 0,720 |
| Thor | 956 | 0,426 |
| Human Torch | 886 | 0,835 |
| Mr. Fantastic | 854 | 0,875 |
| Hulk | 835 | 0,266 |
| Wolverine | 819 | 0,407 |
| Invisible Woman | 762 | 0,966 |

**Értelmezés:** a csoportos szerkezet valódi, nem a fokszámokból következik. A Fantasztikus Négyes tagjainak redundanciája kiugró, mert a képregényeiket a csapattársaik is összekötik. Spider-Man és Hulk jóval alacsonyabb.

**Korlát:** a nagy hősök klaszterezettsége mindig alacsony (0,005–0,008), mert óriási a szomszédságuk. Ezt nem szabad úgy érteni, hogy nem tartoznak csoporthoz.

## 6. Összevetés a szavazásokkal: top 40

**Hol nézd:** „Összevetés: top 40 listák” → „1. Összevetés a szavazásokkal”: két táblázat.

**Mire jó, miért:** a gráf önmagában nem bizonyít semmit a fontosságról, ezért független külső listával mérjük. A szavazás azért jó alapigazság, mert nem a szereplésszámból készült.

**Átfedés:** hány listás szereplő van a mérték top K-jában is.
- **Mit nézz:** a véletlenhez és a gráfban elérhető szereplők számához viszonyítsd. Top 40-nél a véletlen 0,25.
- **Mire jó:** azt méri, **kik** vannak a listán, a sorrend nem számít.

**Spearman-korreláció:** a szavazási és a gráfbeli sorrend egyezése, a listás szereplőkön.
- **Mit nézz:** az 1 teljes egyezést, a 0 kapcsolat hiányát jelenti. A p-érték azt mutatja, mennyire valószínű ekkora korreláció véletlenül.
- **Mire jó:** azt méri, **milyen sorrendben** vannak a szereplők.

| Mérték | 2011 átfedés | 2011 korreláció | 2011 p | 2019 átfedés | 2019 korreláció | 2019 p |
|---|---|---|---|---|---|---|
| Fokszám | 23 | 0,490 | 0,0015 | 22 | 0,524 | 0,0009 |
| Közelség | 25 | 0,416 | 0,0085 | 23 | 0,488 | 0,0022 |
| Köztiség | 27 | 0,489 | 0,0016 | 24 | 0,561 | 0,0003 |
| Sajátvektor | 24 | 0,318 | 0,0482 | 24 | 0,388 | 0,0175 |
| PageRank | 25 | 0,484 | 0,0018 | 24 | 0,558 | 0,0003 |
| Sorozatok | 26 | 0,407 | 0,0101 | 25 | 0,497 | 0,0017 |

**Értelmezés:** az átfedés kb. százszorosa a véletlennek, a korreláció közepes. A mértékek között nincs egyértelmű győztes, mert kevesebb mint 40 szereplőnél a különbségek kicsik. A sajátvektor-centralitás korrelációja a legalacsonyabb.

**Korlát:** a p-értékek nincsenek többszörös tesztelésre korrigálva, és csak népszerű szereplők közti sorrendet mérünk.

## 7. A legnagyobb eltérések: top 40

**Hol nézd:** „Összevetés: top 40 listák” → „2. A legnagyobb eltérések”: két táblázat, a csak a gráf top 40-ében lévők listája és egy pontdiagram.

**Mit csinál:** a `gap` a gráfbeli és a szavazási helyezés hányadosa, a fokszám alapján.

**Mire jó, miért:** az átlagos egyezésnél többet mond, mert megmutatja, *miért* tér el a gráf a szavazástól.

**Mit nézz:** az 1-nél sokkal nagyobb `gap` azt jelenti, hogy az olvasók sorolják előrébb, az 1 alatti, hogy a gráf. A pontdiagramon a függőleges tengely logaritmikus: a magasan lévő pontok a gráfban hátul vannak.

| Irány | Szereplő | Szavazás | Gráf |
|---|---|---|---|
| Az olvasók értékelik többre | Deadpool | 15. | 255. |
| | Bucky Barnes | 24. | 348. |
| | The Multiple Man | 32. | 263. |
| | Emma Frost | 17. | 117. |
| A gráf értékeli többre | Mister Fantastic | 30. | 7. |
| | Iron Man | 7. | 3. |
| | The Thing | 11. | 4. |
| | Scarlet Witch | 31. | 11. |

Csak a gráf top 40-ében vannak például: Mary Jane Watson, J. Jonah Jameson, Edwin Jarvis, May Parker, Joe Robertson, Rick Jones.

**Értelmezés:** a gráf a csapattagokat és a mellékszereplőket jutalmazza, mert sokat szerepelnek. Az olvasók olyanokat is kedvelnek, akik ritkán jelennek meg, vagy akiknek a népszerűsége az adathalmaz vége utánra esik. Ez az eltérések magyarázata, de a gráf önmagában nem bizonyítja.

**Korlát:** a `gap` arány, ezért a lista elején lévő szereplőknél kis helyezéskülönbség is nagy értéket adhat.

## 8. Összevetés: 2011-es top 100

**Hol nézd:** „Összevetés: 2011-es top 100”: táblázat, átfedési görbe és pontdiagram.

**Mire jó, miért:** a top 40 szűk tartomány. A hosszabb lista kevésbé népszerű szereplőket is tartalmaz, így kiderül, hogy az egyezés csak a lista elején áll-e fenn.

**Mit nézz:** az **átfedési görbén** azt, hogy a lista hosszával esik-e az arány. A vízszintes vonal azt jelenti, hogy az egyezés minden hossznál hasonló. A szürke vonal a véletlen.

| Mérték | Átfedés (91-ből) | Korreláció | p |
|---|---|---|---|
| Fokszám | 62 | 0,567 | < 0,0001 |
| Közelség | 61 | 0,565 | < 0,0001 |
| Köztiség | 63 | 0,585 | < 0,0001 |
| Sajátvektor | 56 | 0,485 | < 0,0001 |
| PageRank | 61 | 0,567 | < 0,0001 |
| Sorozatok | 63 | 0,584 | < 0,0001 |

Véletlen átfedés: 1,55. Az átfedés aránya K = 10-től 100-ig 0,55 és 0,72 között mozog, és nem esik vissza.

**Új eltérések:** az olvasók sokkal előrébb sorolják Squirrel Girlt (79., illetve 4645.), Rocket Raccoont (77., 1475.) és Taskmastert (84., 625.). A gráf Human Torchot (43., 6.), Invisible Womant (47., 10.), Mary Jane Watsont (64., 14.) és J. Jonah Jamesont (71., 20.) sorolja előrébb.

**Értelmezés:** a korreláció minden mértékre magasabb, mint a top 40-nél, és az egyezés a lista végéig megmarad.

## 9. Összevetés: 2019-es top 50

**Hol nézd:** „Összevetés: 2019-es top 50”: táblázat, átfedési görbe és pontdiagram.

**Mire jó, miért:** ez az ellenőrzés. Ha a következtetés csak egy listán állna fenn, az lehetne a lista sajátossága.

**Mit nézz:** ugyanazt, mint a 8. lépésben, és azt, hogy a korreláció a top 40-hez képest merre mozdul.

| Mérték | Átfedés (46-ból) | Korreláció | p |
|---|---|---|---|
| Fokszám | 31 | 0,416 | 0,0041 |
| Közelség | 32 | 0,379 | 0,0095 |
| Köztiség | 32 | 0,473 | 0,0009 |
| Sajátvektor | 31 | 0,291 | 0,0494 |
| PageRank | 31 | 0,434 | 0,0026 |
| Sorozatok | 32 | 0,371 | 0,0112 |

Véletlen átfedés: 0,39. Az átfedés aránya K = 10-től 50-ig 0,50 és 0,70 között mozog.

**Értelmezés:** az átfedés a korábbiakkal egyezik, a korreláció viszont minden mértékre alacsonyabb, mint a 2019-es top 40-nél. A hosszabb lista tehát nem javít automatikusan: a 2011-es javulás nem általános szabály, az okát nem tudjuk biztosan. Az eltérések mintázata ugyanaz, mint 2011-ben: Rocket Raccoon (50., 1475.), Deadpool (10., 255.), Winter Soldier (36., 348.), illetve Human Torch (48., 6.).

## Következtetések

1. **A gráf megtalálja a népszerű szereplőket:** az átfedés minden listán nagyságrendekkel a véletlen fölött van.
2. **A sorrendet csak részben adja vissza:** a korreláció közepes, és a lista hosszabbítása nem javít rajta egyértelműen.
3. **A mértékek nem rangsorolhatók ezzel az összevetéssel:** a fokszám, a közelség, a köztiség, a PageRank és a sorozatok száma nagyjából egyformán teljesít. A sajátvektor-centralitás korrelációja minden listán a legalacsonyabb.
4. **A jelenlét nem azonos a népszerűséggel:** a gráf a csapattagokat és a mellékszereplőket felülértékeli, a ritkán szereplő kedvenceket alulértékeli.
5. **A következtetések mindkét szavazásra érvényesek.**

## Korlátok

- **Időbeli eltérés:** a szavazások 10, illetve 18 évvel újabbak az adathalmaz végénél (kb. 2001).
- **Egyenlő szereplések:** a főszerep és egy rövid megjelenés ugyanannyit ér.
- **Névváltozatok:** egy szereplő több csúcsra is szétoszolhat, ami csökkentheti a mért centralitását.
- **Névpárosítás:** néhány párosítás feltételezés (pl. Ghost Rider: Johnny Blaze, Captain Marvel: Mar-Vell). Ezek a CSV-k `match_note` oszlopában vannak jelölve.
- **Hiányzó szereplők:** a gráfban nem szereplő karakterek kimaradnak, ezért a legnagyobb elérhető átfedés kisebb a lista hosszánál.
- **Statisztika:** a p-értékek nincsenek többszörös tesztelésre korrigálva, a köztiség pedig 1000 mintacsúcsból becsült érték.
- **Adat:** 17 képregénykód nyitott maradt, de ezek csak kb. 20 élt érintenek.

## Kérdések és válaszok

A kérdések a `project_description.md` főkérdése és alkérdései.

### Főkérdés: Mennyire adja vissza a gráf szerkezete, hogy kik a legnépszerűbb Marvel-szereplők?

**Részben: a csoportot megtalálja, a sorrendet csak közepesen adja vissza.** A szavazási listák szereplőinek kb. 60–70%-a a gráf azonos hosszúságú toplistáján is rajta van, a sorrenddel viszont csak közepes a korreláció (0,29–0,59). A gráf a jelenlétet méri, ami összefügg a népszerűséggel, de nem azonos vele.

### 1. Mekkora az átfedés a külső lista és a gráf azonos hosszúságú toplistája között?

**Nagy, minden listán nagyságrendekkel a véletlen fölött van.**

| Lista | Átfedés | Szereplők a gráfban | Véletlen átfedés |
|---|---|---|---|
| 2011, top 40 | 23–27 | 39 | 0,25 |
| 2019, top 40 | 22–25 | 37 | 0,25 |
| 2011, top 100 | 56–63 | 91 | 1,55 |
| 2019, top 50 | 31–32 | 46 | 0,39 |

### 2. Mennyire egyezik a sorrend a külső lista és a gráfból számolt rangsorok között?

**Közepesen.** A Spearman-korreláció 0,29 és 0,59 között van, mindig pozitív, és minden érték p = 0,05 alatt van, többszörös tesztelésre való korrekció nélkül. A sorrend tehát hasonlít a szavazáséra, de nem azonos vele.

### 3. Különböznek-e a centralitási mértékek abban, mennyire állnak közel a külső listához?

**Alig, kivéve a sajátvektor-centralitást.** A fokszám, a közelség, a köztiség, a PageRank és a sorozatok száma nagyjából egyformán teljesít, a 2011-es top 40-nél pl. a fokszám 0,490, a köztiség 0,489. A sajátvektor-centralitás korrelációja mind a négy listán a legalacsonyabb. A mértékek különbségét nem teszteltük statisztikailag, ezért ez leíró eredmény.

### 4. Mely szereplők térnek el a legjobban, és mi magyarázhatja az eltérést?

**Az olvasók a ritkán szereplő kedvenceket, a gráf a sokat szereplő csapattagokat és mellékszereplőket sorolja előrébb.**

- **Az olvasók sorolják előrébb:** Squirrel Girl (79. a szavazásban, 4645. a gráfban), Rocket Raccoon (77., 1475.), Deadpool (15., 255.), Bucky Barnes (24., 348.).
- **A gráf sorolja előrébb:** Human Torch (43., 6.), Mister Fantastic (30., 7.), Mary Jane Watson (64., 14.), J. Jonah Jameson (71., 20.).

A lehetséges magyarázat, hogy egy szereplő sok képregényben jelenhet meg kedveltség nélkül is, és fordítva, valamint hogy az adathalmaz vége (kb. 2001) utáni népszerűség nincs a gráfban. Ez értelmezés, mert a népszerűség oka nincs az adatban.

### 5. Fennmaradnak-e a következtetések, ha a lista hosszát vagy az évét megváltoztatjuk?

**A fő következtetések igen, a korreláció nagysága nem.** Az átfedés aránya hosszabb listán és a másik évben is hasonló, és az eltérések mintázata is ugyanaz. A korreláció viszont érzékeny a választásra: a 2011-es listán a hosszabbítással nőtt (a fokszámnál 0,490-ről 0,567-re), a 2019-esen csökkent (0,524-ről 0,416-ra). A különbség okát nem tudjuk biztosan.

### 6. Milyen szerkezeti tulajdonságai vannak a gráfnak (egyenlőtlenség, csoportok, sorozatok), és ezek hogyan magyarázzák az eltéréseket?

**A gráf erősen egyenlőtlen és csoportos, de ez az eltéréseket csak részben magyarázza.**

- **Egyenlőtlenség:** a hősök felső 1%-a birtokolja az élek 32,7%-át, a véletlen gráfban ez 1,8%. Ezért a sokat szereplő hősök eleve kiemelkednek a rangsorokban.
- **Csoportok:** az átlagos klaszterezettség 0,157 a konfigurációs modell 0,042-jével szemben. A Fantasztikus Négyes tagjainak redundanciája magas (0,72–0,97), ami illik ahhoz, hogy a gráf a csapattagokat előrébb sorolja. Ez nem általános: Iron Man redundanciája 0,373, mégis előrébb van a gráfban.
- **Sorozatok:** a hősök több mint fele egyetlen sorozatban szerepel, a legszélesebb elérésűek pedig nagyjából ugyanazok, akik a fokszámrangsort vezetik.

A szerkezet és az eltérések kapcsolatát nem mértük minden szereplőre, ezért ez a kapcsolat nem bizonyított.
