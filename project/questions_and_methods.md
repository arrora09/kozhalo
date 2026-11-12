# Kérdések és vizsgálatok

Ez a fájl a `project_description.md` kérdéseit köti össze a vizsgálatokkal: melyik kérdéshez melyik módszert használjuk, miért, hol találod a `beadando.ipynb`-ben, és melyik laborból származik a módszer.

A laborok a `lab_codes/` mappában vannak. Ahol a módszer nem laborból származik, az külön jelölve van.

## Hogyan keresd meg a beadandóban

- **Fejezet és rész:** a notebook számozott címsorai (pl. „2.2. Bipartit centralitás”). VS Code-ban az Outline (Vázlat) panelen vagy a fejezetek összecsukásával gyorsan megtalálhatók.
- **Cella:** a cella sorszáma a notebook elejétől számolva, a markdown cellákat is beleszámítva (pl. „25. cella”). Ha a notebook változik, a sorszám eltolódhat, a fejezet és a kódnév viszont megmarad.
- **Kódnév:** a cellában létrehozott változó vagy függvény, erre a cellában rá lehet keresni (Ctrl+F).
- **Kimenet:** táblázat, kiírt szöveg vagy ábra a cella alatt.
- **Eredmény:** minden elemző kódcella után egy „Eredmény” kezdetű markdown cella foglalja össze, mit kaptunk.

## Laborok röviden

| Labor | Fájl | Amit innen használunk |
|---|---|---|
| 02 | `lab_codes/02/Lab_02_Metrics.ipynb` | Fokszám-, közelségi, köztiségi, sajátvektor-centralitás, PageRank, klaszterezettség, a mértékek összevetése |
| 03 | `lab_codes/03/Lab_03_Connectivity.ipynb` | Összefüggő komponensek, óriáskomponens |
| 04 | `lab_codes/04/Lab_04_Network_visualization.ipynb` | Hálózatrajzolás, részgráfok megjelenítése |
| 05 | `lab_codes/05/Lab_05_Random_Networks.ipynb` | Fokszám-eloszlás, összevetés véletlen hálózatokkal, óriáskomponens |
| 09 | `lab_codes/09/Lab_09_Equivalence.ipynb` | Jaccard- és koszinusz-hasonlóság a szomszédhalmazok között (tervezett) |

A 06-os (közösségkeresés), a 07-es (befolyásterjedés) és a 08-as (járványmodellezés) labort nem használjuk, mert egyik kérdéshez sem kapcsolódnak.

## Előkészítés: a gráf felépítése

Minden kérdés alapja a megtisztított bipartit hős–képregény gráf.

| Vizsgálat | Miért | Hol a beadandóban | Módszer forrása |
|---|---|---|---|
| Adattisztítás | A hibás bejegyzések és eltérő írásmódok torzítanák a szereplések számát | „1.1. A nyers adatok beolvasása” – „1.5. Átnézendő képregénykódok”, 4–13. cella. Kódnevek: `NOT_COMICS`, `COMIC_FIXES`, `normalize_comic`, `COMICS_TO_REVIEW`. Kimenet: kiírt sorszámok, átnézendő kódok listája | Saját, nem laborból |
| Komponensek és óriáskomponens | Az útvonalalapú mértékek csak összefüggő gráfon értelmezhetők | „1.6. A bipartit gráf felépítése”, 15. cella. Kódnevek: `B`, `components`, `giant`. Kimenet: csúcs-, él- és komponensszám | Lab 03 (`connected_components`), Lab 05 (`giant_component_subgraph`) |
| Kirajzolás | A legfontosabb hősök és képregényeik kapcsolatának szemléltetése | „1.7. Kirajzolás”, 17. cella (`TOP_HEROES`, `TOP_COMICS`), valamint „1.7.1. A teljes óriáskomponens”, 19. cella (`SHOW_FULL_GRAPH`). Kimenet: ábra | Lab 04 (rajzolás), a bipartit elrendezés saját (`bipartite_layout`) |
| Alapigazság: olvasói szavazások | A közkedveltséget egy gráftól független forrás méri | `data/baseline_2011.csv`, `data/baseline_2019.csv`. A notebookban az „3.1. Összevetés a szavazásokkal”, 34. cella olvassa be (`polls`, `poll_top`) | Saját, nem laborból |

## Főkérdés

> **Van-e összefüggés a Marvel-szereplők közkedveltsége és aközött, hogy hány képregényben szerepelnek?**

A főkérdésre az 1. és a 2. alkérdés együtt válaszol, a 3–6. alkérdés ezt árnyalja. Közvetlen vizsgálata a szereplések száma (fokszám) és a szavazási helyezés összevetése átfedéssel és rangkorrelációval.

**Hol a beadandóban:** a válasz a „6. Összegzés” fejezetben van.

## 1. alkérdés: Azok a szereplők a legkedveltebbek, akik a legtöbb képregényben szerepelnek?

| Vizsgálat | Miért | Hol a beadandóban | Módszer forrása |
|---|---|---|---|
| Fokszám (a szereplések száma) | Ez maga a szereplésszám, a kérdés egyik oldala | „2.2. Bipartit centralitás”, 25. cella. Kódnevek: `centrality["degree"]`, `ranks["degree"]`. Kimenet: top 10 táblázat, `degree` oszlop | Lab 02 (`degree_centrality`) |
| Top-K átfedés a szavazással | Megmutatja, hány kedvelt szereplő van a legtöbbet szereplők között | „3.1. Összevetés a szavazásokkal”, 34. cella; „4.1. Összevetés a szavazással”, 40. cella; „5.1. Összevetés a szavazással”, 46. cella. Kódnevek: `compare_with_poll`, `comparison_table`. Kimenet: táblázat, `overlap` oszlop, `degree` sor | Saját, nem laborból |
| Véletlen viszonyítási alap | Az átfedés csak a véletlenhez mérve értelmezhető | Ugyanezekben a részekben (3.1., 4.1., 5.1.), a táblázat fölötti „Expected overlap of a random top …” sor | Saját, nem laborból |

**Eredmény a beadandóban:** a 3.1., 4.1. és 5.1. rész „Eredmény” cellája.

## 2. alkérdés: Ha a szereplőket a szereplések száma szerint sorba állítjuk, mennyire hasonlít ez a sorrend az olvasói szavazás sorrendjére?

| Vizsgálat | Miért | Hol a beadandóban | Módszer forrása |
|---|---|---|---|
| Spearman-féle rangkorreláció | A két sorrend egyezését méri, nem csak azt, hogy kik vannak a listán | 34., 40. és 46. cella. Kódnév: `compare_with_poll` (`spearmanr`). Kimenet: táblázat, `spearman` és `p_value` oszlop, `degree` sor | Saját, nem laborból (`scipy.stats.spearmanr`) |
| Pontdiagram: szavazási és gráfbeli helyezés | Láthatóvá teszi, hol egyezik és hol tér el a sorrend | „3.2.”, „4.2.” és „5.2. A legnagyobb eltérések”: 37., 43. és 49. cella. Kódnév: `show_gaps`. Kimenet: pontdiagram a táblázatok alatt | Saját, nem laborból |

**Eredmény a beadandóban:** a 3.1., 4.1. és 5.1. rész „Eredmény” cellája.

## 3. alkérdés: Erősebb lesz-e az összefüggés, ha nemcsak a szereplések számát nézzük, hanem azt is, hogy kikkel és hogyan kapcsolódnak a szereplők a többiekhez?

| Vizsgálat | Miért | Hol a beadandóban | Módszer forrása |
|---|---|---|---|
| Közelségi centralitás | Azt méri, milyen közel van egy szereplő mindenki máshoz | „2.2. Bipartit centralitás”, 25. cella. Kódnevek: `closeness`, `ranks["closeness"]` | Lab 02 (`closeness_centrality`) |
| Köztiségi centralitás | Azt méri, mennyire köti össze a szereplő a hálózat részeit | 25. cella. Kódnevek: `betweenness`, `BETWEENNESS_SOURCES` (a mintacsúcsok száma) | Lab 02 (`betweenness_centrality`), a mintavételes közelítés saját |
| Sajátvektor-centralitás | Azt méri, mennyire kapcsolódik fontos szereplőkhöz | 25. cella. Kódnév: `eigenvector` | Lab 02 (`eigenvector_centrality`) |
| PageRank | Egy véletlen bolyongás alapján rangsorol | 25. cella. Kódnév: `pagerank` | Lab 02 (`pagerank`) |
| Sorozatok száma | A kiadón belüli elérést méri, nem egy sorozat terjedelmét | „2.3. Sorozatszintű összesítés”, 28. cella. Kódnevek: `series_per_hero`, `ranks["series"]` | Saját, nem laborból |
| A mértékek összevetése egymással | Megmutatja, ugyanazt rangsorolják-e a mértékek | 25. cella. Kimenet: top 10 táblázat (mértékenként egy oszlop) és a „Spearman correlation of the measures” korrelációs tábla | Lab 02 (a mértékek összehasonlító táblája) |
| Átfedés és korreláció mértékenként | Ugyanaz az összevetés, mint az 1–2. kérdésnél, minden mértékre | 34., 40. és 46. cella. Kimenet: táblázat, mértékenként egy sor | Saját, nem laborból |

**Eredmény a beadandóban:** a 2.2. rész „Eredmény” cellája (a mértékek egymáshoz), valamint a 3.1., 4.1. és 5.1. részé (a mértékek a szavazáshoz).

## 4. alkérdés: Kik azok a szereplők, akik sokat szerepelnek, mégsem kedveltek, vagy keveset szerepelnek, mégis népszerűek, és mi lehet ennek az oka?

| Vizsgálat | Miért | Hol a beadandóban | Módszer forrása |
|---|---|---|---|
| Eltéréstábla (`gap`: gráfbeli és szavazási helyezés aránya) | A két irányban legjobban eltérő szereplőket emeli ki | „3.2.”, „4.2.” és „5.2. A legnagyobb eltérések”: 37., 43. és 49. cella. Kódnevek: `gap_table`, `show_gaps`, `GAP_MEASURE`. Kimenet: két táblázat („Readers rate them far above…” és „The graph rates them above…”) és pontdiagram | Saját, nem laborból |
| Csak a gráf toplistáján szereplők | Megmutatja, kiket emel ki a gráf a népszerűségükön túl | „3.2. A legnagyobb eltérések”, 37. cella. Kódnév: `top_unlisted`. Kimenet: a cella utolsó kiírt sora | Saját, nem laborból |
| Hasonlóság a sztárokhoz (tervezett) | Megmérhető vele, hogy a túlértékelt mellékszereplők egy sztár képregényeiben szerepelnek-e | Még nincs a beadandóban | Lab 09 (Jaccard- és koszinusz-hasonlóság) |

**Eredmény a beadandóban:** a 3.2., 4.2. és 5.2. rész „Eredmény” cellája.

## 5. alkérdés: Érvényes marad-e az összefüggés, ha hosszabb listával vagy egy másik év szavazásával dolgozunk?

| Vizsgálat | Miért | Hol a beadandóban | Módszer forrása |
|---|---|---|---|
| Összevetés a 2019-es szavazással | Kiszűri, hogy az eredmény egy lista sajátossága legyen | „3.1. Összevetés a szavazásokkal”, 34. cella (a második táblázat, „2019 poll”), valamint „5. Összevetés: 2019-es top 50”, 45–50. cella | Saját, nem laborból |
| Összevetés hosszabb listákkal (2011: top 100, 2019: top 50) | A top 40 szűk tartomány, a hosszabb lista kevésbé népszerű szereplőket is tartalmaz | „4. Összevetés: 2011-es top 100”, 39–44. cella, és „5. Összevetés: 2019-es top 50”, 45–50. cella | Saját, nem laborból |
| Átfedési görbe (K = 10, 20, …) | Megmutatja, hogy az egyezés csak a lista elején áll-e fenn | 40. és 46. cella. Kódnév: `plot_overlap_curve`. Kimenet: két kis táblázat (fokszám, köztiség) és a görbe ábrája | Saját, nem laborból |

**Eredmény a beadandóban:** a 3.1., 4.1. és 5.1. rész „Eredmény” cellája, a lista hosszának hatása a 4.1. és az 5.1. részben.

## 6. alkérdés: Hogyan oszlanak meg a szereplések a Marvel-univerzumban, és ez mennyire magyarázza az eltéréseket?

| Vizsgálat | Miért | Hol a beadandóban | Módszer forrása |
|---|---|---|---|
| Fokszám-eloszlás, összevetés véletlen gráffal | Megmutatja, hogy néhány szereplő uralja-e a szerepléseket | „2.1. Fokszám-eloszlás”, 22. cella. Kódnevek: `hero_degrees`, `random_graph`, `top_share`, `plot_ccdf`. Kimenet: összesítő táblázat és kétpaneles ábra | Lab 05 (fokszám-eloszlás, véletlen gráf), a bipartit véletlen gráf saját (`gnmk_random_graph`) |
| Sorozatszintű összesítés | Megmutatja, egy sorozathoz kötődnek-e a szereplők | „2.3. Sorozatszintű összesítés”, 28. cella. Kódnevek: `hero_series`, `series_per_hero`, `heroes_per_series`. Kimenet: kiírt számok és toplisták | Saját, nem laborból |
| Bipartit klaszterezés és redundancia, összevetés konfigurációs modellel | Megmutatja, csapatokban szerepelnek-e a szereplők, és ez a fokszámokból következik-e | „2.4. Bipartit klaszterezés és redundancia”, 31. cella. Kódnevek: `clustering_summary`, `config_graph`. Kimenet: két táblázat (átlagok és a top 10 hős) | Lab 02, Lab 05 (klaszterezettség), a bipartit változat és a konfigurációs modell saját (`nx.bipartite`) |
| Hasonlóság a sztárokhoz, minden szereplőre (tervezett) | Az eltérések és a szerkezet kapcsolatát minden szereplőre megméri, nem csak példákon | Még nincs a beadandóban | Lab 09 (Jaccard- és koszinusz-hasonlóság) |

**Eredmény a beadandóban:** a 2.1., 2.3. és 2.4. rész „Eredmény” cellája.
