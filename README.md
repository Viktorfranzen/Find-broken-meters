# Larmanalys — mätare

En liten, fristående webbapp för att analysera larmexporter från mätarnätet.
Ingen backend, inget byggsteg — allt körs i webbläsaren.

## Vad appen gör

- Läser in en CSV- eller tabbseparerad larmexport (kolumner: `Topic`, `Timestamp`,
  `Received`, `Severity`, `Details`, `Metering point`, `Device`, `Organisation`,
  `Correlation ID`, `Initiated by`). Avgränsare (tabb, komma eller semikolon)
  detekteras automatiskt — semikolon är vanligt i CSV-exporter från Excel med
  svenska/europeiska regioninställningar.
- Rättar automatiskt vanlig teckenkodningströtthet (`Ã¤` → `ä`) om den upptäcks.
- Grupperar larm per mätpunkt/enhet. Om varken mätpunkt eller enhet finns i sina
  egna kolumner (t.ex. vid `UnknownCommunicationSource`) faller den tillbaka på
  id:n i JSON-fältet `Details` (`DeviceId`, `GatewayDeviceId`, `CommunicationId`, …).
- Läser filen i bitar (streaming) istället för allt på en gång, så även
  exporter på flera hundra MB går att öppna utan att fliken kör slut på minne.
- **Topplista** — de mätare/enheter som larmat mest under vald period.
- **Stigande trend** — mätare vars larmfrekvens ökar tydligt över perioden
  (linjär regression per tidsbucket: timme/dag/vecka, auto-vald efter periodens
  längd). Dessa markeras separat eftersom de kan missas i topplistan om
  totalantalet ännu är lågt.
- **Filtrera på larmtyp** — via dropdownen "Larmtyp" eller genom att klicka på
  en rad i "Larmtyper i perioden", t.ex. för att bara se ett specifikt larm
  som `Device.PowerGridMonitoring.E11`. Klicka igen för att rensa filtret.
- Klickbara rader visar samtliga enskilda larmhändelser för en mätare.

## Använda appen

Det finns ingen byggprocess. Öppna `index.html` i valfri webbläsare, eller
servera mappen statiskt:

```bash
python3 -m http.server 8000
# öppna http://localhost:8000
```

Eller publicera den som en statisk sida (GitHub Pages, Netlify, Vercel,
intranät …) — filen är helt fristående.

## Filstruktur

```
.
├── index.html   # hela appen (HTML + CSS + JS)
└── README.md
```

## Anpassa

Öppna `index.html` och leta upp:

- `resolveMeterIdentity()` — logiken för vilket id ett larm grupperas på.
- `computeTrends()` — tröskelvärden för vad som räknas som "stigande trend"
  (lutning, korrelation, min. antal larm).
- `detectDelimiter()` — vilka tecken som provas som kolumnavgränsare.
- CSS-variablerna högst upp i `<style>` — färgtema.

## Licens

Valfri — lägg till en `LICENSE`-fil om du vill göra villkoren explicita
(t.ex. MIT).
