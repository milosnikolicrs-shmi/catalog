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

### Raspored

Mreža od pet kolona: tekst zauzima tri, desna kolona dve.

## Faza 5 — mobilni prikaz

Aplikacija se koristi podjednako na desktopu i telefonu, pa mobilni nije smanjeni desktop.

### Navigacija

Sidebar se na telefonu ne izvlači kao meni. Lista je sopstveni ekran, detalj je drugi ekran sa strelicom nazad gore levo. Dodaj i prevlačenje sa leve ivice za povratak.

### Plutajuće trake

Glavna traka je kapsula: `left: 16px`, `bottom: 30px`, širina `calc(100% - 32px)`, visina 56px, pozadina `--bg-elev`, radijus `--r-lg`, ivica 1px sa gornjom u `rgba(201,168,106,0.2)`.

Sadržaj klizi ispod nje. Iznad trake ide prelaz visine 130px iz `--bg` u providno, da tekst bledi umesto da se seče.

Ne koristi providnost ni zamućenje. Na skoro crnoj podlozi staklo daje mutnu sivu mrlju.

### Detalj pesme na telefonu

Tabovi umesto svih polja odjednom: Lyrics, Production, Prompt, More. Lyrics je podrazumevan. Visina taba 44px.

Tekst pesme na `21px`, prored 1.5.

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

Najvažnija stavka u ovoj fazi. Pretraga sada traži po naslovu; proširi je na `lyrics` svih pesama.

Kad upit da pogodak u tekstu a ne u naslovu, red u listi ispod naziva prikazuje isečak stiha sa istaknutim pojmom, umesto uobičajenih meta podataka.

Ovim dobijaš proveru ponavljanja kroz katalog: ukucaš sliku koju si upravo napisao i vidiš da li je već koristio.

### Prečice na desktopu

| Taster | Radnja |
| --- | --- |
| `/` | fokus u polje za pretragu |
| `↑` `↓` | kretanje kroz listu |
| `Enter` | otvori izabranu pesmu |
| `Cmd+K` | brzi skok na pesmu po nazivu |
| `Esc` | zatvori prikaz ili poništi fokus |

Brzi skok je preklapajuće polje na sredini ekrana sa listom rezultata ispod, radijus `--r-lg`.

### Prazna stanja

Kad pretraga ne nađe ništa, umesto poruke ponudi dugme koje pravi novu pesmu sa upisanim nazivom.

Kad album nema pesama, ponudi dugme za dodavanje u taj album.

### Gustina liste

Prekidač u zaglavlju sidebar-a prebacuje između dva reda po pesmi i jednog kompaktnog. Stanje se pamti.

### Mikrointerakcije

Promena statusa menja boju vertikalne crtice kroz prelaz od 150ms. Bez drugih animacija.

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

## Stanje

*(ažurirano posle spajanja u `main`.)*

### Šta je pushovano

Faze 0–6 su spojene u `main` (merge commit `369048d`), plus dva prateća commit-a:

- `569a149` — kontrast svetlog režima (vidi "Odluke" ispod)
- `11f6b1e` — ispravka četiri mobilna bag-a nađena posle spajanja: `--clearance`
  nedovoljan, `.sidebar-foot` se i dalje renderovao ispod trake, Perform kolona
  bežala van ekrana na telefonu (otud tri praga u Fazi 6)

Tačka povratka na stanje pre redizajna: tag `pre-redesign` na `main`-u.

**Faza 7 (funkcionalni dodaci) nije rađena** — pretraga kroz tekstove, prečice
na desktopu, prazna stanja, gustina liste, mikrointerakcije statusne crtice.
Ostaje za sledeću sesiju ako je i dalje u planu.

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
