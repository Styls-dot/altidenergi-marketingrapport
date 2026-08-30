# Void Idle — Territoriesystem "Krigen om Riget" (designdokument)

> Resultat af brainstorm + feel-iterationer i demoen, senest opdateret 2026-08-28.
> Spilbar spec: `voididle-territory-demo.html` (artifact "Krigen om Riget") —
> demoen ER designet; dette dokument er den skriftlige udgave.
> Skelet i spillet: `feature/territory`-grenen i dynastyidle (bag flag, se §16).

## 1. Kernen

- Ét fælles, persistent hex-kort pr. sæson. Alle guilds spiller på samme kort.
- Felter ejes af guilds og betaler dagligt ✦-udbytte (se terræntabellen §3).
- Sæsonpoint bruges i territorie-opgraderinger og bygninger (nulstilles hver
  sæson); ved sæsonslut udbetales engangssum til permanente guild-opgraderinger (§10).
- Kun guilds deltager. Spillere uden guild har ikke adgang til kortet.
- Hver spiller har ÉN figur (avatar med navn + guild-tag) på kortet.

## 2. Tre magt-typer (hentes fra karakteren i spillet)

| Magt | Hovedmagt på | Særlig rolle |
|---|---|---|
| ⚔ Combat power | Eng- og markfelter | Klassisk krig |
| 🪓 Gather power | Skov og bjerge | Terræn-erobring |
| 🔨 Craft power | Guldårer (spirit veins) | Opfører bygninger |

- På ethvert felt vælger spillet automatisk pr. kriger: **max(feltets
  hovedmagt, 20 % af den stærkeste anden magt)**. En ren kriger kan altså
  hugge sig gennem skov — men med 20 % styrke.
- Kampikonet over feltet viser feltets hovedmagt: ⚔ / 🪓 / 🔨 (svingende
  animation, så man på afstand kan se, der kæmpes — og hvilken slags kamp).
- Bidrag pr. tick = effektiv magt / 100 × fyldrate. I spillet kobles det til
  krigsfortjeneste fra rigtige kills/aktioner på feltet.
- ⚔ hentes fra combat-systemet, 🪓 fra gathering-skills, 🔨 fra crafting-skills
  (præcis formel afgøres ved portering).

## 3. Kortet og terræn

| Terræn | Udbytte | March ind | Kamp-magt | Andet |
|---|---|---|---|---|
| Eng (grund) | 1 ✦/døgn | 1× | ⚔ | Ren eng-tile; "fyldt" græs-tile som variation (~25 %, deterministisk hash) |
| Mark | 2 ✦/døgn | 1× | ⚔ | Spredte enkeltfelter over hele kortet |
| Skov | 1 ✦/døgn | 1,5× | 🪓 | Klynge-genereret |
| Bjerg | 1 ✦/døgn | 2× | 🪓 | Kæde-genereret; +60 % garnison i forsvar |
| Guldåre (spirit vein) | 3 ✦/døgn | 1× | 🔨 | Klumper nær kortets midte — kroneprisen |

- Kortet genereres pr. sæson, dimensioneret efter antal guilds; udvides med
  kant-ringe ved behov. Mest værdifulde felter i midten → alle trækkes indad.
- Udbytte kræver 24 timers uafbrudt ejerskab (dræber felt-flipping).
- Alt terræn-artwork foreligger (hex-tiles, pointy-top efter 90°-rotation).

## 4. Bevægelse og marcher

- Bevægelsespoint regenererer over tid (1 pr. 4 t, cap 6); infrastruktur-
  opgraderinger øger regen.
- **Et træk er en march, der tager tid** (rejsetid pr. felt; skov/bjerg 1,5×/2×).
  Undervejs tegnes en stiplet linje fra figurens aktuelle position til målet —
  linjen **krymper**, som figuren rykker frem.
- **Fugleflugt over åbent land:** ruter går den korteste vej over egne og
  neutrale felter. **Andre guilds' territorium spærrer** — ruten lægges udenom.
  (Erstatter den tidligere regel om kun at gå på egne felter.)
- Neutrale felter kan **passeres uden at blive erobret** — kun ordre-målet
  erobres. Dermed er "kun connected angreb" droppet: man kan angribe alt, man
  kan marchere hen til, og fjendens varsel er selve marchen gennem vildmarken
  (synlig efter fog-reglerne).
- Belejringen starter først **ved ankomst** — marchen er forsvarerens varsel.
- **Sammenhængende rige:** man kan kun erobre felter, der grænser op til eget
  territorium — ingen løsrevne felter "midt i ingenting". Bee-flyvnings-marchen
  bruges til at flytte/repositionere; selve erobringen sker felt-for-felt ved fronten.
- **Kaskade-forfald:** mister man et felt, bliver alle egne felter, der derved
  afskæres fra borgen, straks neutrale. Under en fjendtlig belejring pulser de
  felter, der står til at blive afskåret, i svag rød som varsel — så holdet kan
  nå at forsvare choke-pointet (ikke bare feltet, der angribes).

## 5. Fog of war

- Syn: egne felter (radius 1), egne figurer (radius 1-2), borgen (radius 5),
  vagttårn (+2 ud over normalt = radius 3). Hele guilden deler syn.
- Ingen global oversigt eller fri pan — kun det udforskede; tidligere sete
  felter vises grånet som "sidst kendte info" (kan være forældet).
- Angreb på EGNE felter opdages altid øjeblikkeligt. Fog handler om at SE
  angreb komme: syn = varselstid; forsvarets reelle omkostning er afstand.
- **Synlighed følger feltet under figurens aktuelle position:** en fjende, der
  marcherer hen over felter i dit syn, ses med både linje OG avatar. Kommer
  de fra tågen, ses kun linjens spids "vokse frem" ved tågekanten (linjer
  klippes til synlige del-stykker), og avataren dukker op, når positionen
  krydser ind i dit syn.
- **Fjendtlige marcher er anonyme:** flere enheder ad samme rute = én linje;
  hverken antal eller identitet afsløres — kun retning og fremdrift.
- Detaljeniveau: på afstand ses antal + guildfarve; navne kun på felter, der
  støder op til ens egne.

## 6. Erobring og belejring (PvE-kapløb, ingen PvP-motor)

- Angriber ankommer → mod guild-ejede felter først **mobiliserings-forsinkelse**
  (~1 t, synlig "belejring under opsejling"); neutrale felter starter straks.
- Belejringen er et **kapløb om at fylde sin bar** inden for et vindue (12-24 t).
  Barens længde skalerer med EGET antal felter (asymmetrisk handicap: store
  riger har lange barer). Kun **top-8 bidrag pr. side** tæller (valgt efter
  effektiv magt for feltet); bidrag pr. spiller trackes.
- **Flerparts-kapløb:** ankommer et hold til et felt med igangværende kamp,
  får de deres egen bar (fra 0) — først fyldte bar vinder feltet, taberne
  (inkl. andre angribere) trækkes hjem til borgen. Forsvareren vinder ved at
  fylde sin bar eller ved vinduets udløb.
- Forsvarerens bar fyldes af forsvarere på feltet + passiv garnison
  (× 1,6 på bjerg, × 1,75 med fort, + garnison-opgradering).
- Efter en belejring (uanset udfald) er feltet **beskyttet 2-3 døgn**.
- **Borgen kan aldrig erobres** — en guild kan aldrig elimineres.
- **Samlingsbanner** (officer+): plantes på synligt felt; +15 % bidrag for
  egne krigere på feltet.

## 7. Belejringspanel (tap på felt i kamp) — UI-spec

Pr. deltagende hold, sorteret efter føring:
- Bar i guildfarve + procent + **⏱ ETA-timer**: "fuld om X:XX" ved nuværende
  tempo (krigere, banner, garnison/terræn/fort, evt. resterende mobilisering);
  "⏱ i stå" ved nul fremdrift; "⏱ når det ikke" hvis tempoet ikke rækker
  inden vinduet.
- Krigere som rækker: navn · anvendt magt med ikon (⚔/🪓/🔨) og "(20 %)"-
  markering ved sekundær magt · personligt bidrag i % af sidens bar.
  Uden for top-8 vises nedtonet som "(reserve)".
- Kapløbs-hint ved 2+ angribere; mobiliserings-nedtælling; vinduets resttid;
  terræn/bygnings-modifikatorer. Opdateres live; lukker selv ved afgørelse.

## 8. Bygninger (🔨 craft-arbejde)

| Bygning | Pris | Effekt |
|---|---|---|
| 🗼 Vagttårn | 4 ✦ | Syn +2 ud over normalt (radius 3) fra feltet |
| 🏰 Fort | 6 ✦ | +75 % garnison i forsvar (stakker med bjerg) |

- **Leder/vice/officer placerer ordren** (betaler ✦) på et eget felt → en
  **gennemsigtig spøgelses-udgave** af bygningen står på feltet med
  fremdriftsbar. **Håndværkere bygger på stedet** med 🔨 craft power
  (andre magter 20 %); hammer-animation når nogen bygger.
- Én bygning pr. felt. Bygning mistes ved erobring; halvfærdigt byggeri
  forfalder, hvis feltet mistes.
- Systemet er generisk — senere bygninger (portal, garnison, borg-udvidelse)
  er nye rækker i samme tabel. Artwork foreligger for begge + guild-borgen.


**Bygningstyper (demo):** 🗼 Vagttårn (syn +2), 🏰 Fort (+75% forsvar), 🪚 Savværk (+1 træ/døgn) og ⛏ Mine (+1 sten/døgn). **Alle bygninger opføres udelukkende med craft-power (🔨)** — andre magt-typer bidrager ikke. Kun ét byggeri pr. felt. **Ressource-økonomi:** savværk/mine koster ✦ (bootstrap) og producerer træ/sten i guildens lager; vagttårn koster 4 🪵 træ og fort koster 5 🪨 sten + 3 🪵 træ. Så point → savværk/mine → træ/sten → tårn/fort.
## 9. Grupper (party) på kortet

- Spillere kan joine et **pt**; **pt-lederen flytter hele gruppen** med ét
  tryk. Krav: alle medlemmer står samlet og har bevægelse klar.
- Splittes gruppen (fx tvungen retræte), må den samles igen før fælles march.
- Medlemmer markeres visuelt (ring); leder/medlem vises i statuslinjen.

## 10. Point, opgraderinger og sæsoner

- 1-3 ✦ pr. felt pr. 24 t ejerskab (terræntabellen). Sæsonpoint bruges på
  sæsontræet (Logistik/Økonomi: bevægelses-regen, garnison, portaler …) og
  bygninger.
- **Sæson: 30 dage.** Kortet nulstilles, alle spawner på ny. Ved sæsonslut
  udbetaler hvert felt, der er holdt ≥ 24 t, sit daglige udbytte som
  engangssum til **permanente guild-opgraderinger** (Kamp-grenen — de
  bonusser, guilds kender i dag). Anti-sniping: felter taget < 24 t før
  slut tæller ikke; kørende belejringer annulleres.
- Slutspurten om guldårerne er sæsonens naturlige klimaks.


**Sæson-opgraderinger (demo, koblet til reel matematik):** Swift Boots (+25% bevægelses-regen/niv), Supply Lines (+1 max bevægelse/niv), Vanguard (+20% belejrings-angreb/niv), Garrison (+50% passivt forsvar/niv), Watchmen (+1 syn på egne felter og borg/niv). Bygninger (🗼/🏰) placeres direkte på felter.

**Realm Standings (leaderboard):** guild-tabben viser holdene rangeret efter antal ejede felter (tie-break: dagligt ✦-udbytte) med banner, sigil, søjle og ✦/dag; live-opdateret.
## 11. Spawn ("ankerpunkt i konfliktbåndet") og borgen

- Server finder anker: 8-14 felter fra nærmeste borg, ≥ ~70 % neutralt
  omkring, nærmest guilden med færrest naboer (jævn konflikt).
- Leder får et **25×25-valgvindue** oplyst omkring ankeret og placerer borgen
  (med bekræftelses-trin og spøgelses-preview af borg-artworket). Min. 2 frie
  felter mellem borge; ikke på guldårer. Start: borgfelt + ring 1.
- Borgen ser altid 5 felter ud og kan aldrig erobres. Guild-borg-artwork
  (pagode-borgen) bruges på alle borgfelter.
- **Startbeskyttelse**: ny guild immun ~5 døgn eller til 5 erobrede felter.

## 12. Vedligehold og oprydning

- Felter, intet guildmedlem har besøgt/forsvaret i X døgn, forfalder til
  neutrale; guild uden aktive medlemmer i ~14 døgn fjernes fra kortet.
- Upkeep/aftagende afkast for store riger: åbent balancespørgsmål.

## 13. Misbrug og beskyttelse

- Én konto pr. spiller (regel) + adgangskrav (min. combat level for at
  spawne) som teknisk bagstopper.
- Krigskarantæne: nyt medlem kan først bidrage i belejringer efter 3-7 døgn.
- 24-timers-reglen dræber point-farming; opsplitning i småguilds straffer
  sig selv (pointene spredes over flere svage træer); min. borg-afstand
  forhindrer klyngebyggeri.

## 14. Notifikationer

- Ingen Discord. **Begivenhedslog i guild-tabben** med ulæst-badge:
  belejring startet/felt tabt/vundet, banner, byggeri, nye kapløbs-deltagere,
  point. 24 t-vinduet gør "opdaget ved næste login" tidsnok.

## 15. Guild-identitet og opslag (den store guild-opdatering)

Krigen om Riget rulles ud sammen med en bredere guild-opdatering, hvor
guild-tabben bliver holdets samlingspunkt:

**Heraldik (bygget i demoen).** Hver guild vælger **bannerfarve** (12
illustrerede bannere) og **sigil** (29 guld-tegn: dyr, natur, våben og
redskaber — 348 unikke kombinationer) i guild-indstillingerne. Banneret
vises på alle ejede felter, bæres af hver figur på kortet, og sigilet
tegnes på stoffet med mørk silhuet, i tag-chippen og i felt-vinduet.
Server: `banner_color` + `sigil` på guilds-tabellen, kun leder/vice kan
ændre; eksponeres via territory `/state`, så fjenders bannere følger med
fog-reglerne.

**Guild-vindue (demo, faner):** en Guild-knap åbner et vindue med seks faner —
**Info** (banner, navn, rang, tiles, ✦/dag, bankede point, bygninger, captures;
sæson-nedtælling med udbytte pr. felttype + samlet payout ved sæsonslut;
at-risk-advarsel med "Show"; leder-flag med hop-til-kort; **rally-punkt** som
leder sætter og alle kan hoppe til; territorie-besked adskilt fra Void Idle-chatten),
**Members-fanen** (roster med roller — Leader/Vice/Officer/Member, som leder kan
forfremme — position og Jump-til-figur), **War-fanen** (aktive belejringer: egne
angreb og felter under angreb med ETA og Jump), **Upgrades** (Season Tree +
banner-editor), **Ranking** (leaderboard) og **News** (kamp-log med ulæst-badge).
Rally vises som pulserende 📣 på kortet. Roller gater i det rigtige spil hvem der
må sætte flag, købe opgraderinger og skrive beskeden.

**Guild-opslag (announcements).** Ledelsen kan skrive ét fast opslag til
holdet (community-forslag — kendt fra lignende spil):

- **Hvem:** leder og vice kan skrive/redigere; officer kan ikke (holdes
  simpelt fra start). Maks ~500 tegn, ren tekst.
- **Hvor det vises:** 📌 fast øverst i guild-tabben med forfatter og
  tidsstempel — og som **popup når spilleren henter offline-tid**, hvis
  opslaget er nyt siden sidst ("mens du var væk"-skærmen er det sikreste
  sted at ramme alle).
- **Ulæst-logik:** klienten husker `announcement_seen_at`; nyt/ændret
  opslag giver badge på guild-tabben og én linje i begivenhedsloggen
  ("📣 Nyt opslag fra ledelsen").
- **Misbrug:** redigering er rate-limited (fx 1 pr. 5 min), og opslag
  går gennem samme tekstfilter som chatten.
- **Server:** `announcement`, `announcement_by`, `announcement_at` på
  guilds-tabellen + `POST /api/guild/announcement` (leder/vice). Klient:
  GuildPanel-banner + offline-claim-modalen.

Sammen med krigs-tabben, sæsonpoint i guild-butikken og heraldikken er
det én samlet "guild content update".

## 16. Aflæselighed på kortet (fastlagt i demoen)

- Avatarer: spillets Character-ikon for egne figurer, navneskilt med guild-tag
  under; fjender i guildfarve. Avatarer **mod-skaleres** (z^0,4), så de kun
  vokser svagt med zoom. Zoom 0,35-6×; teksturer skarpe på retina (480 px
  kilder, 12× sprite-render).
- Belejringsbarer stables under feltet (op til 3 angribere + forsvarer);
  erobringer giver ekspanderende ring-effekt; tryk-blink på hvert tap;
  bekræftelses-trin på borg-placering.

## 17. Implementeringsstatus og faser

- **Demo (komplet spilbar spec):** alt ovenstående kører i
  `voididle-territory-demo.html` med accelereret tid (1 døgn = 30 s) og
  localStorage-persistens (op til 3 døgns offline-opsamling) — inkl.
  heraldik (12 bannere × 29 sigiler) og felt-popover med magt-liste
  (hovedmagt 100 %, øvrige 20 %).
- **Spillet (`feature/territory`, bag flag `territory_config.enabled`):**
  fase-1-skelet er bygget (kort, spawn, fog, marcher u. rejsetid, simple
  belejringer, point, admin-endpoints). Skal opdateres med: rejsetid/
  march-linjer, kapløb, tre magt-typer, bygninger m. spøgelses-flow,
  grupper, ETA-panel, artwork.
- Flag er OFF som default; admins ser altid tabben; alle tal kan overstyres
  på runtime via `territory_config` (accelereret testsæson på staging).

## 18. Tal til balancering (demo-værdier → reelle udgangspunkter)

| Parameter | Reelt udgangspunkt |
|---|---|
| Døgn / sæson | 24 t / 30 døgn |
| Bevægelses-regen / cap | 1 pr. 4 t / 6 |
| March pr. felt (eng/skov/bjerg) | grundtid × 1 / 1,5 / 2 — grundtid afgøres i playtest |
| Belejringsvindue / mobilisering | 12-24 t / ~1 t |
| Bar-skalering | target × (1 + 0,35 × √(egne felter)) |
| Top-N bidrag | 8 (efter effektiv magt) |
| Off-power | 20 % |
| Garnison-bonus: bjerg / fort | +60 % / +75 % |
| Banner | +15 % |
| Beskyttelse efter kamp / start | 2-3 døgn / 5 døgn el. 5 felter |
| Vagttårn / fort | 4 ✦, syn +2 / 6 ✦ (byggearbejde skaleres af 🔨) |
| Udbytte: eng / mark / guldåre | 1 / 2 / 3 ✦ pr. døgn (≥ 24 t ejerskab) |
| Syn: figur / borg / vagttårn | 1-2 / 5 / 3 |
| Valgvindue ved spawn | 25×25, anker 8-14 fra nærmeste borg |
| Forfald / guild-oprydning | X døgn (ubestemt) / ~14 døgn |
