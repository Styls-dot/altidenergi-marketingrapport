# Void Idle — Territoriesystem for guilds (designdokument)

> Resultat af brainstorm 2026-08-28. Erstatter det nuværende guild-system
> (level + guldkøbte bonusser) med et fælles verdenskort, hvor guilds
> erobrer territorium, tjener opgraderingspoint og fører krig.
> Hører hjemme i `dynastyidle`-repoet (fx `tasks/`) — ligger her, fordi
> brainstorm-sessionen kørte på denne branch.

## 1. Kernen

- Ét fælles, persistent hex-kort pr. sæson. Alle guilds spiller på samme kort.
- Felter ejes af guilds. Hvert felt giver **1 opgraderingspoint pr. 24 timers
  uafbrudt ejerskab** (premium-felttyper kan give 2).
- Sæsonpoint bruges i territorie-opgraderingstræet (nulstilles hver sæson).
  Ved sæsonslut udbetales en engangssum til **permanente guild-opgraderinger**
  (se §8).
- Kun guilds deltager. Spillere uden guild har ikke adgang til kortet.

## 2. Kortet og bevægelse

- Hex-grid. Kortet genereres pr. sæson, dimensioneret efter antal guilds,
  og udvides med en ny kant-ring, hvis spawn-algoritmen ikke kan finde plads.
- Kortets midte rummer de mest værdifulde felter (ruiner, spirit veins,
  verdensboss-hex) → alle trækkes indad mod hinanden; nye guilds spawner
  ved fronten → krig er uundgåelig, aldrig påtvunget.
- Felttyper (differentierer på kortet, IKKE via buffs i idle-spillet):
  - **Spirit vein**: 2 point/dag i stedet for 1
  - **Bjerg**: forsvarsbonus (større garnison)
  - **Vagtpost/ruin**: synsbonus eller andet kort-relateret
- **Bevægelse**: spillere flytter sig individuelt på kortet.
  Bevægelsespoint regenererer over tid (fx 1 pr. 4 timer, cap 6) — ingen
  terninger. Infrastruktur-opgraderinger øger regen; portal-opgradering
  forbinder to egne felter.
- Man kan kun bevæge sig på egne felter — undtagen når man rykker ind på et
  felt for at angribe det.
- Bevægelse er synlig i realtid (langsomt; positions-tick få minutter,
  klienten animerer) for alle, der har syn på feltet.

## 3. Fog of war

- Man ser altid: egne felter + felter, der støder op til dem (radius 1).
- Egne spillere giver syn radius 1-2 omkring deres position. Hele guilden
  deler syn.
- **Borgen ser altid 5 felter ud til alle sider.**
- Vagttårn (infrastruktur på et felt): syn radius 2-3.
- Ingen global oversigt, ingen fri pan — man ser kun det, man har udforsket.
  Tidligere sete felter vises grånet som "sidst kendte info" (kan være
  forældet/lyve).
- Detaljeniveau: på afstand ses kun **antal spillere + guildfarve**; navne og
  detaljer ses kun på felter, der støder direkte op til ens egne.
- Angreb på EGNE felter opdages altid øjeblikkeligt (man ser jo feltet).
  Fog handler om at **se angreb komme** — syn = varselstid, fordi man kan se
  fjender marchere. Forsvarets reelle omkostning er afstand (kan mine folk nå
  frem?), ikke opmærksomhed.

## 4. Erobring og belejring (PvE-kapløb — ingen PvP-motor)

- Angriber rykker ind på målfeltet. Mod guild-ejede felter starter en
  **mobiliserings-forsinkelse** (~1 time), før kampen går i gang; feltet
  vises i "belejring under opsejling" for alle med syn. Neutrale felter:
  kampen starter straks. Angreb er bindende (lille cooldown ved fortrydelse,
  så man ikke kan fake-angribe konstant).
- Selve belejringen er et **kapløb om at fylde sin bar** inden for et vindue
  (12-24 timer): begge siders spillere på feltet kæmper deres normale
  idle-combat mod feltets mobs, og kills/skade tæller som fremdrift.
  Idle-venligt: man bidrager ved at stå der.
- **Bar-længde skalerer med EGET antal felter** (asymmetrisk handicap):
  stor guild = lang bar, lille guild = kort bar. Store guilds kan derfor
  ikke billigt tromle små.
- Kun **top-N bidrag pr. side** tæller (fx 8), så rå medlemstal ikke
  underminerer handicappet.
- Ingen bar fyldt ved vinduets udløb → forsvareren beholder feltet.
- Efter en belejring (uanset udfald) er feltet **beskyttet i 2-3 dage**.
  Neutrale felter har ingen beskyttelse.
- **Borgen (hovedsædet) kan aldrig erobres** — en guild kan bombes tilbage
  til borgen, men aldrig elimineres.
- **Samlingsbanner**: lederen (og udpegede officerer) kan plante et banner på
  et synligt felt; spillere, der står på bannerfeltet ved belejringens
  afgørelse, får fx +15 % bidrag. Banner kan kun plantes på felter, guilden
  kan se (ellers kan man sonde fog gratis).

## 5. Spawn-algoritme ("ankerpunkt i konfliktbåndet")

1. Server scorer ledige felter og finder et **anker**:
   - Afstand til nærmeste eksisterende borg: **8-14 felter** (nogle dages
     march — man møder naboen i uge 1, men kan ikke rushes dag 1).
   - Neutral-tæthed: ≥ ~70 % ejerløse felter inden for radius 5.
2. Blandt gyldige ankre vælges det nærmest **guilden med færrest naboer**
   (inden for 15 felter) → konflikt fordeles jævnt.
3. **Valgvindue** (25×25 hexes) centreret om ankeret vises fuldt oplyst;
   spilleren vælger borgfelt frit derinde.
   Regler: min. **2 frie felter** mellem borge; ikke oven på premium-felter.
4. Ingen gyldige ankre → tilføj en frontier-ring til kortet og søg igen.
5. Efter valget falmer vinduet til "sidst kendte info".
6. **Startbeskyttelse**: nyspawnet guild er immun i fx 5 dage eller indtil
   5 erobrede felter (først indtrufne).

## 6. Vedligehold og oprydning

- **Upkeep/aftagende afkast** overvejes, så imperier bliver bløde i kanterne
  (åbent balancespørgsmål).
- **Forfald**: felter, intet guildmedlem har besøgt/forsvaret i X dage,
  bliver neutrale igen.
- Guild uden aktive medlemmer i ~14 dage fjernes helt fra kortet (inkl. borg).

## 7. Opgraderingstræer

- **Sæsontræ** (købes for sæsonpoint, nulstilles): Logistik + Økonomi —
  bevægelses-regen, portaler, vagttårne, garnisonstyrke, pointindtjening,
  billigere upkeep.
- **Permanent træ** (købes for sæsonslut-udbetaling): Kamp — bonusserne til
  idle-spillet, som guilds kender i dag (ATK/DEF/XP m.m. til alle medlemmer).
  Guild-level låser rækker op; point køber dem.
- Territoriet giver INTET direkte til idle-spillet — kun via træerne.
  (Finjustering af indhold/priser udskudt til balancering.)

## 8. Sæsoner

- **30 dage** pr. sæson (kan forlænges senere). Kortet nulstilles, alle
  spawner på ny.
- Ved sæsonslut: hvert territorium, guilden holder, udbetaler sit **daglige
  pointudbytte som engangssum** til det permanente træ.
- **Anti-sniping**: et felt tæller kun i slutopgørelsen, hvis det har været
  ejet uafbrudt i mindst 24 timer (= har betalt mindst ét dagligt point).
  Belejringer, der kører ved sæsonslut, annulleres.

## 9. Misbrug og beskyttelse

- **Én konto pr. spiller** (regel; svær at håndhæve teknisk).
- **Adgangskrav**: guild skal have fx combat level 20-30 blandt stiftere,
  før den kan spawne på kortet → alts koster dage af grind (bagstopper).
- **Krigskarantæne**: nyt medlem kan først bidrage i belejringer efter
  3-7 dage (må gerne bevæge sig/stå vagt) → lukker guild-hopperi.
- 24-timers-reglen dræber point-farming via felt-flipping.
- Opsplitning i flere små guilds ("delt hær") straffer sig selv: pointene
  spredes over flere svage opgraderingstræer i stedet for ét stærkt.
- Minimumsafstand mellem borge (2 frie felter) forhindrer klyngebyggeri.

## 10. Notifikationer

- Ingen Discord. **Begivenhedslog i guild-tabben**: belejring startet,
  felt tabt/vundet, banner plantet, fjender observeret.
- Badge med antal ulæste på tabben. 24-timers belejringsvinduet gør
  "opdaget ved næste login" tidsnok — passer idle-rytmen.

## 11. Byggefaser

1. **MVP**: kort + spawn, bevægelse, fog, erobring af neutrale felter,
   point → sæsontræ.
2. **Krig**: belejringer guild-mod-guild, mobiliserings-forsinkelse,
   banner, beskyttelsesperioder, begivenhedslog.
3. **Dybde**: felttyper, vagttårne/portaler, forfald/upkeep, sæsonslut →
   permanent træ, evt. spejder-rolle (bevæge sig i fremmed land uden at
   erobre; synlig og fangbar).

Teknisk: fog = server-side filtrering af hvilke felter der sendes til
klienten. Belejringer genbruger eksisterende PvE-combat med feltmarkering.
Kortstate er én tabel. Største klientopgave: pan/zoom hex-kort, mobilvenligt.

## 12. Åbne spørgsmål / tal til balancering

| Spørgsmål | Udgangspunkt |
|---|---|
| Valgvinduets størrelse | 25×25 (besluttet) |
| Belejringsvindue | 12-24 t |
| Mobiliserings-forsinkelse | ~1 t |
| Bar-skalering pr. felt ejet | ubestemt — playtest |
| Top-N bidrag | 8 |
| Beskyttelse efter belejring | 2-3 dage |
| Startbeskyttelse | 5 dage / 5 felter |
| Forfald (ubesøgte felter) | X dage — ubestemt |
| Bevægelses-regen | 1 pr. 4 t, cap 6 |
| Syn: spiller / borg / vagttårn | 1-2 / 5 / 2-3 |
| Upkeep eller aftagende afkast | ubestemt |
| Sæsonlængde | 30 dage (kan forlænges) |
| "Vagt-stance" (mindre loot, +syn, alarm) | idé — ikke besluttet |
