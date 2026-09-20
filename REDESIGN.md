# SHMI Catalog — brief za redizajn

## Pre nego što počneš

Ovo je brief za redizajn `index.html` u repou `milosnikolicrs-shmi/catalog`. Redizajn je vizuelni i strukturni — logika aplikacije se ne dira.

Ne menjaj:

- Firebase/Firestore vezu, autentikaciju i upsert logiku uvoza
- format u kom se čuva `perfSheet` (`stih | komentar`, sekcije `[Chorus] (hint)`) — taj format je dobar i ostaje
- shemu podataka pesme (polja u `editBuffer`)
- PWA podešavanja i putanju objavljivanja

Kako radi:

- radi na grani `redesign`, ne na `main`
- idi fazu po fazu redosledom iz ovog dokumenta
- posle svake faze stani, javi šta si uradio i čekaj potvrdu
- posle svake faze proveri da aplikacija radi u oba režima, tamnom i svetlom

Paleta boja ostaje netaknuta. Sve vrednosti u `:root` i `body.light` su tačne i ne menjaju se.

## Faza 0 — razdvoj fajl

Trenutno je sve u jednom `index.html` od 310 KB: CSS zauzima linije 13–1382, JavaScript 1484–3024. Svaka stilska izmena znači čitanje celog fajla.

Uradi:

1. Izdvoj glavni `<style>` blok (linije 13–1382) u `styles.css` i poveži ga sa `<link rel="stylesheet" href="styles.css">`.
2. Mali `<style>` blok za štampu (oko linije 2389) ostavi gde jeste — vezan je za print prikaz.
3. JavaScript za sada ostavi u `index.html`.
4. Proveri da aplikacija radi identično pre i posle.

Ne refaktoriši CSS u ovoj fazi. Samo premesti, znak po znak.

## Faza 1 — tipografska skala

U fajlu ima 30 različitih veličina fonta, od kojih 8 poludecimalnih (9.5, 10.5, 11.5, 12.5, 13.5, 14.5, 15.5, 16.5px). To je glavni razlog zašto aplikacija deluje neuredno.

Dodaj u `:root` sedam promenljivih:

| Promenljiva | Vrednost | Gde se koristi |
| --- | --- | --- |
| `--t-micro` | 11px | oznake sekcija velikim slovima, meta podaci u listi |
| `--t-caption` | 13px | sitan tekst, pomoćne informacije |
| `--t-body` | 15px | telo teksta, nazivi u listi, polja |
| `--t-lead` | 18px | istaknut red, naziv u listi na mobilnom |
| `--t-h3` | 22px | naslovi kartica i albuma |
| `--t-h2` | 30px | brojevi statistike, naslov pesme na mobilnom |
| `--t-h1` | 42px | naslov pesme na desktopu |

Zatim zameni svaku postojeću vrednost najbližom iz skale:

- 9.5, 10, 10.5, 11, 11.5 → `--t-micro`
- 12, 12.5, 13, 13.5, 14, 14.5 → `--t-caption`
- 15, 15.5, 16, 16.5, 17 → `--t-body`
- 20, 21 → `--t-lead`
- 23, 25 → `--t-h3`
- 28, 29, 31 → `--t-h2`
- 38 → `--t-h1`

Izuzetak: `calc(Npx * var(--perf-scale, 1))` u Perform prikazu ostaje kako jeste — to je skaliranje koje korisnik menja dugmetom.

Posle zamene prođi ekran po ekran i javi ako je negde hijerarhija ispala pogrešna. Očekujem da će na par mesta trebati korak gore ili dole.

## Faza 2 — razmaci i radijusi

Razmaci: promenljive `--s1` do `--s8` postoje i koriste se 49 puta, ali tvrdo ukucanih px vrednosti za `padding`, `margin` i `gap` ima 101. Sistem postoji, poštuje se u trećini slučajeva.

Zaokruži svaku tvrdu vrednost na najbližu iz lestvice 4 / 8 / 12 / 16 / 24 / 32 / 48 / 64 i zameni promenljivom. Gde je razlika veća od 4px, javi mi pre zamene.

Radijusi: ima 15 različitih vrednosti (4, 5, 6, 7, 8, 9, 10, 11, 12, 14, 16, 20, 50%, 100px, 999px) i tri načina da se napiše potpuno zaobljeno.

Dodaj u `:root`:

| Promenljiva | Vrednost | Gde |
| --- | --- | --- |
| `--r-sm` | 8px | sitna dugmad, oznake |
| `--r-md` | 12px | dugmad, polja, redovi liste |
| `--r-lg` | 16px | kartice, plutajuće trake |
| `--r-full` | 999px | čipovi i sve potpuno zaobljeno |

Mapiranje: 4–9 → `--r-sm`, 10–14 → `--r-md`, 16–20 → `--r-lg`, a 50%, 100px i 999px → `--r-full`. Izuzetak je krug za avatar ili omot gde 50% ostaje.

Pravilo koncentričnih radijusa: element unutar kontejnera ima radijus manji za vrednost unutrašnjeg razmaka. Kartica sa `--r-lg` (16) i paddingom 4 sadrži dugme sa radijusom 12.

## Faza 3 — komponente

### Lista pesama

Red ima dve linije: naziv na `--t-body`, ispod njega meta podaci na `--t-caption` u `--text-faint`. Zadrži postojeću sličicu omota levo.

Status više nije tekstualna oznaka desno nego vertikalna crtica 3px široka, 30px visoka, levo od sadržaja:

- final → `--ok`
- prod → `--gold`
- draft → `--draft`

Brojevi numera i grupisanje po albumima ostaju kako jesu.

Visina reda 62px na mobilnom, 52px na desktopu.

### Stanja reda

- obično: providna pozadina, naziv u `--text-dim`
- pod mišem: pozadina `--bg-elev`, naziv u `--text`
- izabran: pozadina `--bg-fill`, naziv u `--text`
- prelaz 150ms, postojeći `--ease`

Desni klik na red otvara meni sa istim radnjama koje na mobilnom daje prevlačenje.

### Filteri

Spoji dva odvojena segmentirana reda (vokal i status) u jedan red čipova koji se prelama: All, Final, Prod, Draft, Female, Male. Aktivan čip ima pozadinu `--gold` i tekst `--bg`. Red čipova po albumu ostaje ispod.

### Kartice

Pozadina `--bg-panel`, ivica 1px u `rgba(236,232,225,0.07)`, radijus `--r-lg`. Gornja ivica dobija `rgba(201,168,106,0.16)` — jedva vidljiv zlatni odsjaj.

### Brojevi

Svuda gde stoje BPM, trajanje, procenti i vreme dodaj `font-variant-numeric: tabular-nums` da se kolone poravnavaju.

## Faza 4 — detalj pesme

Pesma ima oko 25 polja. Trenutno su sva jednako vidljiva, pa nijedno nije.

Podeli ih na ono što se gleda i ono što se popunjava jednom.

Uvek vidljivo, u gornjem delu i u karticama:

- naslov, album i broj numere, omot
- status, vokal, tonalitet, BPM, trajanje, žanr
- tekst pesme
- Suno prompt i negative prompt
- pipeline status

U presavijenim sekcijama, podrazumevano zatvorenim:

- Rights & credits: SOKOJ status, procenat vokala, krediti
- Analysis & chords: analiza, akordi, instrumenti, produkcija
- Links & history: linkovi na platforme, istorija verzija teksta, cross-catalog echo

Sekcija je dugme visine 30px sa nazivom levo i strelicom desno, unutar kartice sa radijusom `--r-lg`. Stanje otvorenosti pamti se po sesiji, ne po pesmi.

### Tekst pesme

Ograniči širinu kolone teksta na `max-width: 480px` i na desktopu, iako ima mesta. Prored 1.7. Oznake sekcija `[Verse 1]` idu na `--t-micro` u `--gold`, iznad strofe.

**Dopuna (posle spajanja):** `max-width:480px` je bio na mestu otkad je ova
faza urađena — proveren merenjem, nikad nije popuštao (potvrđeno 480px na
1400px ekranu). Ono što JESTE bilo premalo je sam font: `--t-body` (15px) na
desktopu. Dodata `--t-lyric: 19px` u `:root`, primenjena samo iznad 900px
(`@media (min-width: 901px) { .lyrics { font-size: var(--t-lyric); } }`),
prored ostaje 1.7 (`line-height` se ne menja, samo `font-size`). Ispod 900px
ostaje kako je bilo (15px do 760px, pa 21px ispod 600px iz Faze 5).

### Raspored

Mreža od pet kolona: tekst zauzima tri, desna kolona dve.

**Dopuna:** omot pesme (`.cover-img`) ograničen na `max-height: 380px` —
previsok za deo teksta ispod. Smanjen na `max-height: 320px`. Kartice BPM /
Key / Duration / Genre (`.specs`) su bile `repeat(3, 1fr)` na svim
širinama, što je ostavljalo Genre samu u drugom redu i prazan prostor
desno od nje na širokom ekranu. Dodato `@media (min-width: 901px) { .specs
{ grid-template-columns: repeat(4, 1fr); } }` — sve četiri u jedan red
iznad 900px, ispod ostaje `repeat(3, 1fr)` (vidi Faza 5 za mobilnu
ispravku visine kartica). Oba pravila su namerno upisana POSLE osnovnih
`.lyrics`/`.specs` pravila u fajlu — ista specifičnost, kasniji redosled u
izvoru pobeđuje, bez obzira na media-query poklapanje, pa medijski upit
mora doći posle da bi uopšte radio.

## Faza 5 — mobilni prikaz

Aplikacija se koristi podjednako na desktopu i telefonu, pa mobilni nije smanjeni desktop.

### Navigacija

Sidebar se na telefonu ne izvlači kao meni. Lista je sopstveni ekran, detalj je drugi ekran sa strelicom nazad gore levo. Dodaj i prevlačenje sa leve ivice za povratak.

### Plutajuće trake

Glavna traka je kapsula: `left: 16px`, širina `calc(100% - 32px)`, visina 56px, pozadina `--bg-elev`, ivica 1px sa gornjom u `rgba(201,168,106,0.2)`.

**Ažurirano na Apple/iOS 26 vrednosti** (vidi "Odluke" na kraju): `bottom:
env(safe-area-inset-bottom)` (bez dodatne margine — safe-area inset sam po
sebi je razmak), `border-radius: 28px` (van postojeće skale radijusa, upisano
direktno). Isti tretman na `.perf-jump` u Perform prikazu. Prvobitna
vrednost `bottom: 30px` i `radius: --r-lg` (16px) iz ovog pasusa više ne
važe.

Sadržaj klizi ispod nje. Iznad trake ide prelaz visine 130px iz `--bg` u providno, da tekst bledi umesto da se seče.

Ne koristi providnost ni zamućenje. Na skoro crnoj podlozi staklo daje mutnu sivu mrlju.

### Detalj pesme na telefonu

Tabovi umesto svih polja odjednom: Lyrics, Production, Prompt, More. Lyrics je podrazumevan. Visina taba 44px.

Tekst pesme na `21px`, prored 1.5.

**Dopuna (posle spajanja) — dupla dugmad:** vrh ekrana (`.detail-actions`:
Perform/Edit/Delete) i sadržaj (`.lyric-actions`: Copy lyrics/Print lyrics)
su i dalje bili puni desktop dugmadi na mobilnom, POVRH plutajuće trake
(Copy lyrics/Edit/⋮) i "..." menija (koji već ima Perform/Print
lyrics/Delete) — skoro svaka radnja se pojavljivala dvaput ili triput.
Ispravljeno u mobilnoj medijskoj upitu:

- `.detail-actions .icon-btn:not(:first-child) { display: none; }` — gore
  ostaje samo Perform (prvo dugme u markup-u); Edit i Delete su već
  dostupni preko trake (Edit) i "..." menija (Delete).
- `.lyric-actions { display: none; }` — Copy lyrics je već u traci, Print
  lyrics je već u "..." meniju; ceo red u sadržaju je bio višak.

Ništa nije dodato u "..." meni — sve što se uklonilo iz sadržaja već je
imalo svoje mesto u traci ili meniju.

**Dopuna — omot odsečen na dnu:** `.cover-img` na mobilnom
(`max-height:300px`) je koristio `object-fit: cover` nasleđeno iz osnovnog
pravila — kad je omot uspravniji (portret) od okvira kog `max-height`
pravi (širina 100% × 300px, dakle širi format), `cover` seče vrh i dno da
popuni okvir, uključujući natpis blizu dna slike. Prvi pokušaj
(`object-fit: contain`) je rešio sečenje ali ostavljao prazne trake sa
strane kad odnos stranica ne odgovara. Pošto su omoti kvadratni, ispravan
fix je `aspect-ratio: 1 / 1` (bez `max-height`) uz `object-fit: cover`
vraćen nazad — okvir je uvek kvadrat pa `cover` nema šta da seče kad je i
slika kvadratna. Testirano sa kvadratnom test-slikom (500×500, oznake na
sve četiri ivice) na 390px: box tačno 358×358 (širina sadržaja), sve četiri
ivice vidljive, nema praznih traka. Ovo pravilo važi samo na mobilnom —
desktop `.cover-img` (van medijskog upita, `max-height:320px`) ostaje
namerno "bannner" krupni kadar, ne kvadrat.

**Dopuna — Genre kartica lomi visinu:** `.spec` kartice na mobilnom nisu
imale zajedničku visinu, pa je Genre (duži tekst) padala u tri reda dok su
BPM/Key/Duration ostajale u jednom, praveći neujednačen red kartica.
Dodato `.spec { min-height: 64px; }` (izjednačava kratke kartice) i na
`.spec .v` `-webkit-line-clamp: 2` (ograničava dužu vrednost na dva reda sa
`…`, umesto tri). Izmereno: BPM/Key/Duration 64px, Genre 73px (blago viša
zbog dva reda teksta, ali vizuelno poravnato, ne tri reda kao ranije).

### Gestovi

Prevlačenje reda ulevo otkriva dve radnje: kopiraj tekst i promeni status. Širina otkrivenog dela 112px, dva dugmeta 44×44 sa razmakom 8.

### Obavezno

Sva polja za unos moraju biti najmanje 16px. Ispod toga Safari na iPhone-u automatski zumira ekran pri kliku u polje.

Sve dodirne mete najmanje 44×44px.

## Faza 6 — Perform i editor komentara

Perform je ekran koji pevačica gleda dok snima u studiju, umesto papira. Nema plejera i nema muzike — snimanje ide preko DAW-a.

### Perform prikaz

Problemi sada: prored je toliko veliki da staje osam redova na ekran, a refren ima sedam. Oznaka sekcije zauzima celu levu kolonu za dve reči. Desna polovina ekrana je prazna. Boje oznaka su plava i zelena, van palete.

Izmene:

- prored na 1.45, veličina 27px, `font-weight: 300`
- oznaka sekcije iznad strofe, `--t-caption`, `--gold`, velika slova, `letter-spacing: 0.22em`; pored nje hint iz `(…)` kurzivom u `--text-faint`
- sve oznake zlatne, plava i zelena izlaze
- kolona teksta 660px, desno margina za komentare
- komentar: kurziv, `--t-caption`, `--text-dim`, sa zlatnom crticom 16×2px iznad, poravnat sa strofom na koju se odnosi
- dole plutajuća traka za skok na sekciju: dugme po sekciji iz teksta, aktivno u `--gold`
- gore desno: A−, A+, prekidač komentara, svetli režim, zatvori
- pozadina `#0a0908`, tamnija od ostatka aplikacije

**Ispravka:** Perform se koristi podjednako na telefonu, iPadu i desktopu —
prvobitna rečenica "ukloni mobilnu verziju" je bila pogrešna i nije važila.
Umesto jednog fiksnog rasporeda, tri praga:

- **do 600px (telefon):** tekst 21px, prored 1.5. Kolona teksta
  `max-width: 100%` sa bočnim paddingom 24px (`--s5`, već je bio podrazumevani
  padding `.perf-inner`-a, nije trebalo posebno podešavanje). Komentar nije u
  margini nego blok ispod strofe: pozadina `--bg-panel`, radijus `--r-md`,
  zlatna vertikalna linija 2px sa leve strane, tekst kurziv `--t-caption` u
  `--text-dim` (crtica 16×2px iznad komentara se ne koristi u ovom prikazu).
  Traka za skok na sekciju se horizontalno skroluje (`overflow-x: auto`).
- **600–900px (iPad uspravno):** isto kao telefon (blok-komentar, skrolujuća
  traka), samo tekst 24px i bočni padding 40px.
- **preko 900px (iPad položeno, desktop):** dve kolone kao ranije opisano —
  tekst levo, komentari u desnoj margini sa zlatnom crticom iznad — ali
  kolona teksta je `max-width: 660px` (`flex: 1 1 auto`), ne fiksnih
  `flex: 0 0 660px`, da se suži umesto da prelije kad je glavna kolona uža
  od 660px.

Prelaz između 600px i 900px, i između 900px i preko, mora da bude čist kad
se iPad rotira iz uspravnog u položeni položaj — proveri oba pravca.

### Traka za skok na sekciju — dinamično aktivno stanje

Traka prati koju sekciju korisnik trenutno čita, preko `IntersectionObserver`
(ne scroll listener-a) nad oznakama sekcija (`.perf-section`), sa `root`
postavljenim na `#perfOverlay` i `rootMargin: '-10% 0px -70% 0px'` (traka
čitanja u gornjoj trećini ekrana). Dva ispravljena problema:

- **Koje dugme se pali:** posmatrač drži `intersectionRatio` svake sekcije u
  mapi koja preživljava između poziva callback-a i bira sekciju sa najvećim
  odnosom kao aktivnu, umesto da samo uključi poslednji unos iz trenutnog
  paketa entry-ja. Prvobitna verzija je to radila naivno
  (`entries.forEach(...)` uključi dugme za svaki entry koji trenutno preseca
  traku i pritom isključi sva ostala) — kad dve susedne kratke sekcije
  istovremeno seku traku čitanja (čest slučaj s kratkim Intro/Outro
  sekcijama), pobeđivalo je šta god je poslednje u paketu, ne ona koja
  stvarno dominira trakom. `threshold: [0, 0.1, 0.25, 0.5, 0.75, 1]` daje
  dovoljno tačaka da se odnos prati glatko tokom skrola.
- **Kad aktivno dugme izađe iz vidljivog dela trake:** kad se aktivna sekcija
  promeni, aktivno dugme se dovodi u vidno polje sa
  `scrollIntoView({ behavior: 'smooth', inline: 'center', block: 'nearest' })`
  (`block: 'nearest'` da ne pokuša i vertikalni skrol cele stranice, samo
  horizontalni unutar `.perf-jump`).

Testirano skrolovanjem kroz celu pesmu (deset sekcija, uključujući kratak
Intro i Outro) na 390px širine: aktivno dugme ide redom 0→1→2→...→N bez
preskakanja i bez vraćanja na pogrešnu sekciju, traka se pomera da prati
aktivno dugme. Kod: `index.html`, `setupPerfJumpObserver()`.

### Razmak oznake sekcije (i mobilni i desktop)

Oznaka sekcije (`[VERSE 2]`) je imala skoro isti razmak iznad i ispod —
`.perf-section { padding: var(--s5) 0; }` (24px gore i dole simetrično) plus
razmak iz flex `gap` (`--s3`, 12px) između oznake i prve linije — pa je
ukupan razmak IZNAD oznake (24px prethodne sekcije + 1px ivica + 24px ove
sekcije ≈ 49px) izgledao vizuelno slično razmaku ISPOD (12px), zbog velikog
`line-height`-a strofe iznad koji "guta" deo praznine.

Promenjeno na `padding: var(--s2) 0 var(--s5)` (8px gore, 24px dole — oba
postojeći `--s` tokeni, ne nova magična vrednost). Razmak ispod ostaje
nepromenjen (već je bio ispravan). Izmereno (1400px ekran, realan
`.perf-lyric` sa `line-height: 1.45`): razmak iznad 52px → **36px**, razmak
ispod ostaje **13px** — traženo je bilo 32/12; 1px do 4px odstupanja dolazi
od `line-height` (leading) fonta, ne od `padding`-a, i nije vizuelno
primetno. Pravilo je van bilo kog media upita, pa važi svuda (`OBA` iz
zahteva).

### Editor komentara

Format čuvanja ostaje isti: `stih | komentar`, sekcije `[Chorus] (hint)`. Menja se samo prikaz.

Sada su dve odvojene kutije teksta sa uputstvom da se piše „u istom redu". To se lako razilazi i ne vidi se šta ide uz šta.

Novi prikaz je red po red:

- levo stih na `--t-body`, sa rednim brojem u `--text-faint`
- desno polje za komentar, visina 38px, radijus `--r-md`
- red sa komentarom: stih u `--text`, polje sa pozadinom `--bg-elev`
- prazan red: stih u `--text-dim`, polje providno sa tanjom ivicom
- red sekcije: naziv u `--gold` velikim slovima, polje sa zlatnom ivicom za hint
- Tab vodi na sledeće polje

Prečice (`/` za pauzu, `(breath)`, `^` za podizanje) idu u traku ispod naslova kao čipovi, ne u pasus uputstva.

Dugmad Save, Cancel i Reset from lyrics ostaju, gore desno, sa vremenom poslednjeg čuvanja pored njih.

## Faza 7 — funkcionalni dodaci

### Pretraga kroz tekstove

**Urađeno:** pretraga (`renderSidebar()`) sad traži i po `lyrics`, ne samo po
naslovu — haystack je `[title, lyrics, album, genre, analysis, instruments]`.
Ovim dobijaš proveru ponavljanja kroz katalog: ukucaš sliku koju si upravo
napisao i vidiš da li je već korišćena.

**Nije urađeno:** kad upit pogodi u tekstu a ne u naslovu, red u listi i
dalje prikazuje uobičajene meta podatke (žanr/tonalitet/BPM), ne isečak
stiha sa istaknutim pojmom kako je prvobitno traženo. Ostaje za kasnije ako
je i dalje potrebno.

### Prečice na desktopu — nije rađeno

Ni jedna od ovih ne postoji u kodu:

| Taster | Radnja |
| --- | --- |
| `/` | fokus u polje za pretragu |
| `↑` `↓` | kretanje kroz listu |
| `Enter` | otvori izabranu pesmu |
| `Cmd+K` | brzi skok na pesmu po nazivu |
| `Esc` | zatvori prikaz ili poništi fokus |

Brzi skok je preklapajuće polje na sredini ekrana sa listom rezultata ispod, radijus `--r-lg`.

(Postoje `Cmd/Ctrl+S` za snimanje i `Cmd/Ctrl+N` za novu pesmu dok se uređuje
— to je starija prečica iz koda pre redizajna, ne stavka iz ove faze.)

### Prazna stanja — nije rađeno

Kad pretraga ne nađe ništa, umesto poruke ponudi dugme koje pravi novu pesmu sa upisanim nazivom.

Kad album nema pesama, ponudi dugme za dodavanje u taj album.

I dalje se prikazuje samo tekstualna poruka ("No songs yet." / "No matches."), bez dugmeta.

### Gustina liste — nije rađeno

Prekidač u zaglavlju sidebar-a prebacuje između dva reda po pesmi i jednog kompaktnog. Stanje se pamti.

### Mikrointerakcije — nije rađeno

Promena statusa menja boju vertikalne crtice kroz prelaz od 150ms. Bez drugih animacija.

Ove četiri stavke (prečice, prazna stanja, gustina liste, mikrointerakcije)
nisu rađene i za sada se ne rade.

## Provera pre spajanja

### Svetli režim

Ceo redizajn je rađen u tamnom. Svetli set boja postoji ali nije proveren.

Izmeri kontrast svakog para teksta i pozadine i javi rezultate pre nego što spojimo granu. Sumnjivi su:

- `--gold` svetli (`#96783e`) na `--bg-panel` (`#fdfcf9`) za sitan tekst
- `--text-faint` (`#a29b8b`) na `--bg` (`#f4f1ea`) — verovatno pada ispod praga
- `--draft` (`#a29b8b`) kao boja statusa

Prag je 4.5:1 za tekst do 24px, 3:1 iznad. Gde ne prolazi, potamni boju teksta umesto da menjaš pozadinu.

### Čeklista

- [ ] nijedna `font-size` vrednost nije van skale, osim `--perf-scale` računica
- [ ] nijedan `border-radius` van četiri promenljive, osim 50% za krug
- [ ] sva polja za unos na mobilnom najmanje 16px
- [ ] sve dodirne mete najmanje 44×44px
- [ ] svetli režim prolazi kontrast na svim ekranima
- [ ] uvoz i izvoz kataloga rade nepromenjeno
- [ ] Perform čita postojeće `perfSheet` zapise bez izmene podataka
- [ ] prijava i odjava rade
- [ ] PWA se i dalje instalira

### Redosled

Faze 0 do 3 su mehaničke i mogu brzo. Faze 4 do 7 traže odluke — posle svake mi javi šta si uradio pre nego što nastaviš.

## Istraga: "Cracks" — zlatna crtica u listi naspram DRAFT/IDEA u detalju

Nije menjano u kodu — samo pročitano, na traženje vlasnika.

U detalju pesme postoje dva ODVOJENA, namerno različita polja, oba
prikazana kao bedž:

- **`s.status`** (`draft` / `production` / `final`) — pokreće i zlatnu/sivu/
  zelenu crticu u glavnoj bočnoj listi (`.song-item::before`,
  `status-production` → `--gold`) I prvi bedž u detalju
  (`statusLabel()` → "Draft" / "In production" / "Final", ispisano velikim
  slovima kroz `.badge{text-transform:uppercase}` → "DRAFT").
- **`s.pipelineStatus`** (`idea` / `draft-lyrics` / `demo` / `recording` /
  `mix` / `master` / `distributed`) — pokreće SAMO drugi bedž u detalju
  (`pipelineLabel()`, podrazumevano "Idea" ako polje nije postavljeno →
  "IDEA"). Album tracklist prikaz (`.track-row`) uopšte nema statusnu
  crticu, samo ovaj pipeline bedž — potvrđeno u `styles.css`, nema
  `::before` pravila za `.track-row`.

Ovo samo po sebi nije bag — dva bedža su dva različita polja po dizajnu.

**Ali:** i zlatna crtica u glavnoj listi I prvi bedž u detalju čitaju ISTO
polje (`s.status`). Ako lista za "Cracks" pokazuje zlatnu (`production`)
crticu, a detalj pokazuje "DRAFT", to znači da `s.status` u tom trenutku
NIJE isti na oba mesta — a oba čitaju isti objekat iz istog `catalog` niza
u memoriji, pa razlika ne može doći iz koda koji sam pregledao. Mogući
uzroci van koda: dva različita zapisa pesme sa istim nazivom "Cracks" u
Firestore-u, ili je status promenjen u međuvremenu pa lista prikazuje
stariji render koji nije osvežen. Nisam mogao dalje da proverim bez
pristupa Firestore-u — ako želiš, mogu privremeno da dodam `console.log`
za `s.status`/`s.pipelineStatus` pri otvaranju pesme da uhvatimo tačnu
vrednost uživo.

## Stanje

*(ažurirano 2026-09-20.)*

### Deset stavki, detalj + Perform prikaz (ova sesija)

Testirano u browseru na 390px i na širokom desktop ekranu (izolovani test
sa pravim `styles.css`, pošto app zahteva Firebase login pa se ne može
testirati direktno kroz app):

1. `--t-lyric: 19px` za tekst pesme iznad 900px (bilo `--t-body`=15px), prored ostaje 1.7 — izmereno 19px/32.3px line-height na 1400px ekranu
2. `.lyrics{max-width:480px}` — proveren u izolovanom testu, već je ispravan (tačno 480px na 1400px ekranu). **Otvoreno:** vlasnik na živom sajtu i dalje vidi tekst kako se pruža preko 600px — ili je test premalo verodostojan, ili u pravoj aplikaciji nešto drugo pobeđuje. Vlasnik proverava direktno u bazi.
3. `.cover-img{max-height:380px}` → `320px` (desktop)
4. `.specs` 4 kolone u jedan red iznad 900px (bilo `repeat(3,1fr)` svuda)
5. mobilni omot: prvi pokušaj `object-fit:contain` je ostavljao prazne trake kod nepodudarnog odnosa stranica; pošto su omoti kvadratni, konačno rešenje je `aspect-ratio: 1/1` (bez `max-height`) uz `object-fit: cover` vraćen — kvadratni okvir + kvadratna slika = ništa se ne seče. Testirano kvadratnom test-slikom (500×500) sa oznakama na sve četiri ivice.
6. sadržajni "Copy lyrics"/"Print lyrics" (`.lyric-actions`) sakriveni na mobilnom — traka i "..." meni ih već imaju
7. sadržajna "Edit"/"Delete" dugmad sakrivena na mobilnom — ostaje samo Perform gore
8. `.spec` kartice: `min-height:64px` + `-webkit-line-clamp:2` na Genre vrednosti, umesto loma u tri reda
9. razmak oznake sekcije u Perform-u: `padding: var(--s2) 0 var(--s5)` — izmereno 36px iznad / 13px ispod (traženo 32/12)
10. istraga statusa "Cracks" — namerno odloženo, vlasnik sam proverava u bazi

Sve promene su samo u `styles.css`.

### Šta je pushovano

Faze 0–6 su spojene u `main` (merge commit `369048d`), plus prateći commit-i:

- `569a149` — kontrast svetlog režima (vidi "Odluke" ispod)
- `11f6b1e` — ispravka četiri mobilna bag-a nađena posle spajanja: `--clearance`
  nedovoljan, `.sidebar-foot` se i dalje renderovao ispod trake, Perform kolona
  bežala van ekrana na telefonu (otud tri praga u Fazi 6)
- `8083d3f` — traka za skok na sekciju u Perform prikazu prati aktivnu
  sekciju preko `IntersectionObserver` (ispravljena trka pri simultanim
  entry-jima) i sama se pomera kad aktivno dugme izađe iz vidljivog dela
  trake; plutajuća traka spuštena na `bottom: calc(12px + safe-area)`
  (kasnije dalje ispravljeno, vidi sledeću stavku); emodži ikonice (🔍 📋
  ☀ ☾) zamenjene SVG-om
- `c499c75` — plutajuća traka i Perform traka na tačnim Apple/iOS 26
  vrednostima: `bottom: env(safe-area-inset-bottom)` (bez dodatne margine),
  bočna margina 16px, radijus 28px; `--clearance` i `.perf-inner` padding
  preračunati

Tačka povratka na stanje pre redizajna: tag `pre-redesign` na `main`-u.

**Faza 7 (funkcionalni dodaci): samo pretraga kroz tekstove je urađena**
(pretraga sad filtrira i po `lyrics`, ne samo po naslovu — vidi Faza 7 gore).
Isečak stiha sa istaknutim pojmom u rezultatima, prečice na desktopu, prazna
stanja sa dugmetom, gustina liste i mikrointerakcije statusne crtice **nisu
rađene i za sada se ne rade**.

**Otvoreno, van faza:** `box-shadow: var(--shadow-2)` je i dalje na obe
plutajuće trake — ako "bez senke" treba da važi kao deo Apple vrednosti,
ovo čeka ukljanjanje u kodu (vidi "Odluke").

### Šta ostaje za testiranje uživo

Sledeće nisam mogao da proverim bez pristupa Firestore/Firebase Auth, pa je
neprovereno kroz kod, ne uživo:

- uvoz i izvoz kataloga (Export → kopiraj JSON → Import istog teksta)
- Perform čita postojeći `perfSheet` zapis bez izmene (otvori pesmu koja već
  ima sačuvane performance beleške)
- prijava i odjava
- PWA instalacija na pravom HTTPS URL-u (manifest/ikonice nisu dirani, ali
  install-prompt ponašanje treba proveriti na živom sajtu)

Preporuka: posle poslednje ispravke (Perform pragovi, `--clearance`,
`.sidebar-foot`) vredi još jednom proći sva tri mobilna ekrana (lista, detalj,
Perform) na stvarnom telefonu, ne samo u simuliranom viewport-u.

### Odluke donete usput, van prvobitnog brief-a

- **`--clearance`** (Faza 2): uvedena kao placeholder (90px desktop / 70px
  mobilni) za prostor koji će zauzeti plutajuća traka iz Faze 5. Kad je traka
  stvarno napravljena, mobilna vrednost od 70px se pokazala nedovoljnom —
  stvarni otisak trake je `bottom(30px) + visina(56px)` ≈ 86px plus sigurnosna
  zona uređaja. Ispravljeno na `calc(102px + env(safe-area-inset-bottom))`.
- **Traka niže, bliže Apple Music razmaku** (posle spajanja): `.mobile-bar`
  je sedela previsoko — `bottom: calc(30px + safe-area)` ostavljao je veliki
  razmak do sistemske crtice na iPhone-u. Promenjeno na
  `calc(12px + env(safe-area-inset-bottom))`, tako da traka sedi tik iznad
  sistemske crtice (12px na uređajima bez crtice). `--clearance` je pratio
  razliku dole: `12 + 56 (visina trake) + 16 (vazduh) = 84px`, pa
  `--clearance: calc(84px + env(safe-area-inset-bottom))` (bilo 102px).
  Isti tretman na Perform ekranu: `.perf-jump` sa `bottom: 24px` (bez
  sigurnosne zone uopšte — bio bi propust na uređajima sa crticom) postaje
  `calc(12px + env(safe-area-inset-bottom))`; `.perf-inner`-ov donji padding
  ide sa 100px na `calc(80px + env(safe-area-inset-bottom))`
  (`12 + 54 (izmerena visina trake, sa ivicom) + ~14 vazduha`).
  Izmereno u browseru na 390×844 (bez safe-area, tj. uređaj bez crtice):
  `.mobile-bar` bottom 12px, visina 56px, razmak do dna ekrana 12px,
  `.song-list` padding-bottom 84px (razmak iznad trake 16px); `.perf-jump`
  bottom 12px, visina 54px, `.perf-inner` padding-bottom 80px (razmak iznad
  trake ~14px). Na uređaju sa sigurnosnom zonom (npr. 34px) oba razmaka rastu
  za tu vrednost jer je ona u istoj `calc()` na obe strane.
- **Traka na pravim Apple/iOS 26 vrednostima** (posle prethodne ispravke):
  vlasnik je izmerio uživo na telefonu i doneo tačne Apple vrednosti za
  plutajuću traku — donja margina ispod trake je 0 (safe-area inset JESTE
  razmak, ništa se ne dodaje povrh), bočna margina 16px, radijus 28px.
  Promenjeno na sva tri mesta (lista, detalj — oba kroz `.mobile-bar` — i
  Perform kroz `.perf-jump`):
  - `bottom: calc(12px + env(safe-area-inset-bottom))` → `bottom:
    env(safe-area-inset-bottom)` na obe trake.
  - `.mobile-bar` bočna margina je već bila 16px (`left: 16px`, `width:
    calc(100% - 32px)`) — nije trebalo menjati. `.perf-jump` je imao
    `max-width: calc(100% - 48px)` (24px margina), promenjeno na
    `calc(100% - 32px)` (16px) da odgovara.
  - `border-radius` obe trake sa `var(--r-lg)` (16px) na `28px` (van
    postojeće skale radijusa — Apple-ova vrednost za ovu traku, ne uklapa se
    u `--r-sm/md/lg/full`, pa je upisana direktno, ne kao nova promenljiva).
  - `--clearance`: otisak trake sad je samo `visina(56px) + 16px vazduha`
    (bottom više ne dodaje 12px pošto ga je safe-area već pokrila) =
    `calc(72px + env(safe-area-inset-bottom))` (bilo 84px).
  - `.perf-inner` donji padding: `visina(54px) + 16px vazduha` =
    `calc(70px + env(safe-area-inset-bottom))` (bilo 80px).
  Ova izmena je pushovana bez uživo testiranja u browseru — vlasnik testira
  direktno na telefonu i doneo je Apple vrednosti odatle.

  **Nije urađeno:** `box-shadow: var(--shadow-2)` je i dalje na obe trake
  (`.mobile-bar` i `.perf-jump`) u `styles.css`. Ako je "bez senke" deo istih
  Apple vrednosti (pomenuto naknadno), ovo ostaje da se ukloni — nije
  uklonjeno u kodu, samo je ovaj dokument ranije to prevideo.
- **Emodži → SVG ikonice** (van prvobitnog brief-a): lupa za pretragu u
  plutajućoj traci, clipboard za "Copy lyrics" u redu pesme i sunce/mesec za
  prekidač teme su bili sistemski emodži (🔍 📋 ☀ ☾) — u boji, sa senkom na
  iOS, upadali iz dizajna. Zamenjeni inline SVG ikonicama u
  `index.html` (konstante `ICON_SEARCH`, `ICON_COPY`, `ICON_SUN`,
  `ICON_MOON` blizu vrha JS-a): `stroke="currentColor"`,
  `stroke-width="1.6"`, `fill="none"`, `stroke-linecap="round"`, 18×18.
  Boja dolazi iz postojećeg CSS-a dugmeta (`--text-dim` / `--text` na
  hover-u), nije trebalo dodatno stilizovanje. Nije dirano: strelice za
  reorder albuma, strelice za sidebar collapse, muzička nota za Perform, X
  za zatvaranje, kružna strelica za promenu statusa, tri tačke za "more"
  meni — to su obični tipografski simboli, ne emodži u boji.
- **`--gold-text`** (Provera pre spajanja): nova promenljiva, definisana samo
  u `body.light` kao `#7e6534`, korišćena isključivo tamo gde je `--gold`
  sitan čitljiv tekst (section-label, perf-label, detail-track-num, bedževi,
  `.tag`, itd.). Sama `--gold` ostaje `#96783e` netaknuta za pozadine dugmadi,
  ivice i akcente, jer tamo kontrast teksta nije relevantan. `--text-faint` je
  potamnjen na `#6e6858` (bio ispod praga svuda). `--draft`/`--prod`/`--ok`/
  `--final` i `--text-dim` su namerno ostavljeni — prva četiri su statusna
  grafika sa pragom 3:1 koji već prolazi, `--text-dim` je promašio prag za
  manje od pola procenta, ispod granice vidljive razlike.
- **Perform, tri praga** (Faza 6): prvobitno uputstvo "ukloni mobilnu verziju"
  je bilo pogrešno — Perform se koristi na telefonu i iPadu, ne samo na
  desktopu. Tačne vrednosti sad pišu direktno u Fazi 6 iznad.
- **`.sidebar-foot { display: none }` ispod 760px** (ispravka posle Faze 5):
  Faza 5 je uvela plutajuću traku sa "..." menijem kao zamenu za stari
  sidebar-foot (+ New song/Import/Export/tema/odjava), ali sama Faza 5 nije
  eksplicitno tražila da se sidebar-foot sakrije — samo da traka postoji.
  Posledica: oba su se prikazivala istovremeno na mobilnom. Dodato naknadno.
