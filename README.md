# Madplan

Ugens retter, indkøbsliste og madspildsregnskab for husstanden.
Vælg retter, få indkøbet omregnet til hele pakker, og se hvad der bliver
til overs — og hvilke retter der kan bruge resterne.

Live: `https://mtechmichael-sketch.github.io/madplan/`

## Filer

```
index.html          Hele appen. Én selvstændig fil, virker offline.
manifest.json       Så den kan lægges på hjemmeskærmen
icon-192.png
icon-512.png
data/
  raavarer.json     Råvarekatalog med pakkestørrelser, priser og Nemlig-ID
  retter.json       Retterne med ingredienser og mængder
```

## Hvordan data hænger sammen

`index.html` har hele datasættet indbygget, så filen virker selv når man
åbner den direkte fra disken uden netforbindelse.

Ligger appen på en server, henter den derudover `data/raavarer.json` og
`data/retter.json` og lægger dem ovenpå. Reglerne er med vilje forskellige:

- **Råvarer overskrives.** Priser og pakkestørrelser skal følge med, når de
  bliver opdateret fra Nemlig.
- **Retter tilføjes kun.** Har du selv rettet i en ret på din egen enhed,
  bliver den aldrig kørt over.
- **Dine egne råvarer og retter røres ikke.** Alt med et id der ikke findes i
  filerne, bliver stående.

Hentningen styres af `version`-nummeret øverst i hver JSON-fil. Hæv det, når
du har ændret noget — ellers opdager appen det ikke. Åbnes appen som en lokal
fil, blokerer browseren hentningen, og appen kører videre på det indbyggede.
Det er meningen.

## Datamodel

**Råvare** — det hele hviler på `pakke`. Madspild opstår, fordi en ret bruger
600 g og butikken sælger 1.100 g.

| Felt | Betydning |
|---|---|
| `id` | Nøgle. Bruges af retterne. Må ikke ændres |
| `navn`, `kategori`, `enhed` | `g`, `ml` eller `stk` |
| `pakke` | Pakkestørrelse i den enhed. Ved dåser: **nettovægt**, ikke dåsevægt |
| `pakkenavn` | Som det står hos Nemlig, fx `"1,10 kg / Danpo"` |
| `pris` | Kroner pr. pakke |
| `holdbar` | Dage efter køb. Styrer hvor hårdt madspildsmotoren presser på |
| `frys` | Kan varen fryses, er resten ikke spild |
| `nid` | Nemligs produkt-ID. Varen ligger på `nemlig.com/<navn>-<nid>` |
| `est` | `true` = holdbarheden er et skøn. Nemlig oplyser den ikke for frugt og grønt |

**Ret** — `ing` er råvare-id → mængde **for hele retten**, ikke pr. portion.
`opskrift` er en liste af trin (eller `null`).

```json
{ "id": "karry", "navn": "Kylling i karry m. ris", "portioner": 4,
  "ing": { "kyllingebryst": 600, "log": 200, "karrypasta": 50 },
  "opskrift": ["Skær kyllingen i tern og løget i både.", "..."] }
```

## Fælder der allerede har bidt

- **Dåsevægt er ikke nettovægt.** Kidneybønner er en 410 g dåse med 250 g
  drænet. `pakke` skal være 250.
- **Enhed og mængde skal passe sammen.** Da squash skiftede fra `g` til `stk`,
  stod en opskrift stadig med 300 — og ville have bestilt 300 squash.
- **Billigst pr. kg er ikke altid mindst spild.** Kyllingebryst er billigst i
  1,10 kg, men en ret på 600 g efterlader 500 g. Afvejningen tages vare for vare.

## Data ligger i browseren

Alt hvad man vælger og retter — inklusive hvilken ret der ligger på hvilken
dag, og adressen til Home Assistant — ligger under nøglen `mtech_madplan_v1`
i den enkelte browser, ikke på serveren. Hver enhed har altså sin egen plan.
Fælles familiedata kommer først med backenden på Raspberry Pi'en.

Brug **Gem data** / **Indlæs data** i appen til at flytte eller sikkerhedskopiere.

**Tokenet til Home Assistant ligger for sig selv**, under nøglen
`mtech_madplan_ha_token`, og er med vilje ikke en del af `mtech_madplan_v1`.
«Gem data» skriver hele planen ud i en fil der bliver flyttet rundt mellem
enheder — et token må aldrig følge med i den. Det står heller aldrig i koden
eller i en fil på serveren, kun i den ene browser det er sat ind i.

## Køkkenkopien hos Home Assistant

Ligger appen i Home Assistants `www`-mappe (adressen begynder med `/local/`),
sker der tre ting af sig selv:

- Adressen til Home Assistant findes automatisk — det er serveren siden kom fra.
  «Hjem»-fanen virker kun her, fordi Home Assistant kun lader sig vise i en
  ramme på en side fra samme server.
- Tomgangsskærmen viser vejr, når der er sat et token ind under Mere.
- Appen ser en gang i timen efter en nyere udgave og genindlæser selv. Det er
  nødvendigt, fordi Home Assistant beder browseren gemme filerne i 31 dage.
  **Hæv `VERSION` i `index.html` ved hver ændring**, ellers opdager skærmen det ikke.

## Hvad der IKKE må ligge her

Repoet er offentligt, fordi GitHub Pages kræver det på en gratis konto.
Opskrifter og indkøbslister er harmløse.

**Kalenderdata må aldrig i repoet** — børnenes navne, skole, tidspunkter og
hvornår huset er tomt. Køkkenskærmens aftaler hentes fra Google Kalender, mens
skærmen kører, og rører aldrig disse filer. Det skal blive ved med at være sådan.
