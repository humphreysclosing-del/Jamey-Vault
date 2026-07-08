# Handoff: Jamey // Vault — Persoonlijk CFO-dashboard

## Overview
Jamey // Vault is een persoonlijk financieel dashboard ("persoonlijke CFO") voor een Nederlandse
zelfstandig ondernemer die meerdere bedrijven/entiteiten runt (CKJ, Humphreys Closing, Lumen).
De app laat de gebruiker inkomsten, uitgaven, klanten, facturen, doelen, nettovermogen en cashflow
beheren, met per-bedrijf kostenoverzichten, een financiële-gezondheidsscore en een AI-analyse.

De taal van de volledige UI is **Nederlands**. Bedragen worden in euro's getoond met Nederlandse
notatie (`€ 1.234,56`).

## About the Design Files
De bestanden in dit pakket zijn **ontwerp-referenties, gemaakt in HTML** — een prototype dat het
bedoelde uiterlijk en gedrag laat zien. Het is **geen productie-code om direct te kopiëren**.
De opdracht is om dit ontwerp opnieuw op te bouwen in een echte codebase-omgeving, met de daar
gangbare patronen en libraries.

Er bestaat nog geen codebase. Aanbevolen stack (kies gerust een gelijkwaardig alternatief):
- **Frontend:** React + Next.js (of Vue/SvelteKit) met TypeScript.
- **Styling:** Tailwind CSS of CSS-modules — het ontwerp gebruikt inline stijlen die 1-op-1 naar
  utility-classes of tokens te vertalen zijn.
- **Backend/data:** Supabase of Firebase voor auth + database (het prototype gebruikt nu localStorage;
  dat moet vervangen worden door een echte database met een gebruikers-account zodat data
  synchroniseert tussen apparaten).
- **Grafieken:** de eenvoudige lijn-/taart-/balkgrafieken zijn nu met de hand als SVG getekend.
  In productie kan een library als Recharts of Visx dit overnemen.

Belangrijkste architectuur-upgrade t.o.v. het prototype: **localStorage → server-side database met
authenticatie**, zodat één gebruiker op meerdere apparaten kan inloggen en data behouden blijft.

## Fidelity
**High-fidelity.** Dit is een pixel-nauwkeurig ontwerp met definitieve kleuren, typografie, spacing
en interacties. Bouw de UI exact na. Alle exacte waarden staan hieronder onder Design Tokens.

## Data Model
Alle entiteiten. In het prototype worden ze in localStorage bewaard; in productie worden dit
database-tabellen, allemaal gekoppeld aan een `user_id`.

- **settings** (één per gebruiker): `companyName`, `kvk`, `btw`, `iban`, `address`, `startBalance` (number), `monthlyTarget` (number)
- **businesses** (vast, 3 stuks): `CKJ`, `Humphreys Closing`, `Lumen` — elk met een accentkleur (zie tokens)
- **income[]**: `id`, `company`, `client`, `description`, `amount` (number), `btw` (number, %), `date` (YYYY-MM-DD), `status` (`betaald`|`openstaand`), `method`, `recurring` (`ja`|`nee`), `category` (Freelance/Consulting/Affiliate/SaaS/Dividend/Investeringen/Verhuur/Overig), `business`, `notes`
- **expenses[]**: `id`, `description`, `amount` (number), `date`, `category` (Woning/Auto/Verzekeringen/Telefoon/Internet/Marketing/Software/Abonnementen/Belastingen/Persoonlijke uitgaven/Vakantie/Investeringen/Overig), `business`, `paidVia`, `frequency` (`eenmalig`|`maandelijks`), `note`
- **clients[]**: `id`, `name`, `company`, `email`, `phone`, `address`, `kvk`, `btw`, `notes`
- **invoices[]**: `id`, `clientId` (FK → clients), `number`, `description`, `date`, `dueDate`, `amount` (number), `btw` (number, %), `status` (`open`|`betaald`)
- **goals[]**: `id`, `title`, `target` (number), `saved` (number), `deadline` (date)
- **assets[]** (nettovermogen): `id`, `type` (Bankrekening/Spaarrekening/Crypto/Beleggingen/Vastgoed/Auto/Overig), `name`, `value` (number)
- **debts[]**: `id`, `type` (Hypotheek/Lening/Creditcard/Belastingen/Overig), `name`, `value` (number)
- **cashflow[]** (handmatige verwachte items): `id`, `description`, `amount` (number), `date`

## Screens / Views
De app heeft een vaste linker-sidebar (248px breed) en een hoofd-content-gebied (max 1180px breed,
padding 34px 40px). De sidebar bevat het logo "JAMEY // VAULT" bovenaan, 9 navigatie-items, en een
zoekbalk onderaan. Elk nav-item heeft een gekleurd vierkantje (8px, border-radius 2px) dat op 100%
opacity staat bij de actieve pagina en 30% bij inactieve. Actief item: tekst `#F5F5F7`, achtergrond
`rgba(255,255,255,0.05)`. Inactief: tekst `#8A8F98`.

De hoofd-header toont een kicker (klein, uppercase, `#5A606B`), een titel (27px, 700, `#F5F5F7`) en
rechts het "Beschikbaar saldo" in monospace amber.

### 1. Dashboard
- **Doel:** totaaloverzicht van de financiële situatie.
- **Layout:** verticale stapel van secties.
  - Rij van 4 KPI-kaarten: Omzet deze maand (teal), Winst deze maand (teal/coral o.b.v. teken),
    Kosten deze maand (coral), Netto cashflow (teal/coral).
  - Rij van 4 kleinere KPI-kaarten: Verwacht binnen 30 dagen, Verwachte uitgaven, Jaaromzet, Jaarwinst.
  - Rij (1.15fr / 1fr): Financiële-gezondheidskaart (SVG-ring met score 0-100 + maanddoel-voortgangsbalk)
    en AI-Insights-kaart (amber-getinte achtergrond, knop "Analyse genereren").
  - Rij (1.5fr / 1fr): lijngrafiek "Omzet vs. kosten" (6 maanden) en taartdiagram "Inkomsten per categorie".
  - Volle breedte: lijst "Verwachte betalingen · komende 30 dagen".
- **Gezondheidsscore-berekening:** `clamp(0..100)` van
  `min(1, max(0, winstmarge))*40 + spaarratio*30 + (1 - schuld/(bezit+schuld))*30`.
  Waar winstmarge = maandwinst/maandomzet, spaarratio = min(1, beschikbaar/(maandkosten*3)).
  Label: ≥70 "Uitstekend" (teal), ≥45 "Gezond" (amber), anders "Aandacht nodig" (coral).
- **AI-Insights:** genereert lokaal (of via een echte LLM-call in productie) 3-5 zinnen over
  winstmarge, grootste kostenpost, openstaande facturen, herhalende inkomsten en een aanbeveling.
  Toont een "Analyseren…"-laadstatus (~900ms in prototype).

### 2. Inkomsten
- **Layout:** twee kolommen (340px formulier links, sticky; lijst rechts).
- **Formulier:** bedrijf, klant, omschrijving, bedrag + BTW%, datum, status + herhalend, betaalmethode,
  categorie, bedrijf (CKJ/Humphreys Closing/Lumen), notities, knop "+ Inkomst toevoegen".
- **Lijst:** per regel omschrijving + categorie-badge + herhalend-indicator, submeta (klant · datum · methode),
  klikbare status-badge (wisselt betaald/openstaand), bedrag in teal monospace, verwijder-×.
- **Lege staat:** "Nog geen inkomsten toegevoegd."

### 3. Uitgaven
- **Layout:** zelfde tweekolom-patroon als Inkomsten.
- **Formulier:** omschrijving, bedrag, datum, categorie, bedrijf, betaald via, maandelijks/eenmalig, notitie.
- **Lijst:** omschrijving + categorie-badge + maandelijks-indicator, submeta (datum · betaald via · bedrijf),
  bedrag in coral monospace, verwijder-×.
- **Lege staat:** "Nog geen uitgaven toegevoegd."

### 4. Bedrijven (kosten per bedrijf)
- **Doel:** kosten en resultaat per bedrijf (CKJ, Humphreys Closing, Lumen).
- **Sub-navigatie:** tabs "Overzicht / CKJ / Humphreys Closing / Lumen".
- **Overzicht:** 3 klikbare kaarten (kosten deze maand groot, kosten/omzet per jaar, winst) + horizontale
  balkgrafiek "Kosten per bedrijf · dit jaar".
- **Detail (per bedrijf):** rij van 4 KPI's (kosten deze maand, kosten dit jaar, omzet dit jaar, winst),
  daaronder een "Kosten toevoegen"-formulier (koppelt de uitgave automatisch aan het actieve bedrijf) +
  kostenlijst van dat bedrijf.
- **Lege staat:** "Nog geen kosten voor <bedrijf>."

### 5. Klanten
- **Layout:** formulier links (340px), kaartenraster rechts (2 kolommen).
- **Formulier:** naam, bedrijf, e-mail, telefoon, adres, KvK + BTW, notities.
- **Kaart:** avatar met initialen (amber gradient), naam + bedrijf, e-mail/telefoon/KvK+BTW, verwijder-×.
- **Lege staat:** "Nog geen klanten toegevoegd."

### 6. Facturen
- **Layout:** links (320px) een formulier + factuurlijst; rechts een print-klaar factuurvoorbeeld.
- **Formulier:** klant kiezen (dropdown uit clients), factuurnummer, omschrijving, datum + vervaldatum,
  bedrag + BTW%, status.
- **Factuurvoorbeeld:** lichte "papier"-kaart (`#F7F5F0`, donkere tekst) met bedrijfslogo-blokje,
  bedrijfsgegevens, "FACTUUR" + nummer, klantgegevens, regeltabel (omschrijving + BTW + totaal), IBAN-voettekst.
  Knoppen "Status wisselen" en "⎙ Print / PDF" (roept `window.print()`; print-CSS toont alleen `#invoicePrint`).
- **Lege staat (rechts):** "Selecteer of maak een factuur om het voorbeeld te zien."

### 7. Doelen
- **Layout:** bovenaan een inline-formulier (titel, doelbedrag, al gespaard, deadline), daaronder
  kaartenraster (2 kolommen).
- **Kaart:** titel + deadline, groot percentage in amber, voortgangsbalk (teal gradient), gespaard vs. doel.
- **Lege staat:** "Nog geen doelen."

### 8. Nettovermogen
- **Layout:** 3 samenvattingskaarten (Bezittingen teal, Schulden coral, Nettovermogen amber-getint),
  daaronder een balkgrafiek (bezittingen/schulden/netto), daaronder twee kolommen: bezittingen-beheer
  en schulden-beheer (elk een inline mini-formulier type/naam/waarde/+ en een lijst).

### 9. Cashflow & verwachte betalingen
- **Doel:** alles wat nog binnenkomt, automatisch samengevoegd.
- **Samenvoeglogica:** combineert (a) open facturen (bedrag incl. BTW, op vervaldatum), (b) openstaande
  inkomsten, (c) herhalende inkomsten geprojecteerd naar de volgende maand, en (d) handmatige items.
  Gefilterd op datum ≥ vandaag-3 dagen, gesorteerd op datum.
- **Layout:** 2 totaalkaarten (30 dagen teal, 90 dagen amber), een inline-formulier voor handmatige items,
  en een lijst met bron-badge (Factuur/Openstaand/Herhalend/Handmatig), label, "over N dagen", bedrag.
  Alleen handmatige items zijn verwijderbaar.
- **Lege staat:** "Nog niets verwacht…"

### 10. Instellingen
- **Layout:** max 620px breed, twee kaarten.
  - Bedrijfsgegevens: bedrijfsnaam, KvK, BTW, IBAN, adres (live opgeslagen bij typen).
  - Financiën: beginsaldo, maanddoel omzet, en een knop "Alle data wissen & voorbeeld herstellen".

## Interactions & Behavior
- **Navigatie:** klikken op sidebar-item wisselt de actieve pagina (client-side, geen page reload).
- **Formulieren:** velden schrijven naar een per-formulier "draft"-object; op "toevoegen" wordt een record
  aangemaakt (met unieke id), de lijst vooraan bijgewerkt, en het formulier gereset. Numerieke velden
  (`amount`, `btw`, `target`, `saved`, `value`) worden geparsed naar getallen. Toevoegen alleen als het
  kernveld (bedrag/naam/titel) is ingevuld.
- **Status togglen:** klik op status-badge bij inkomsten (betaald↔openstaand) en facturen (open↔betaald).
- **Verwijderen:** de × verwijdert het record uit de lijst.
- **Print:** factuur-print via `window.print()` met print-CSS die alleen `#invoicePrint` zichtbaar laat.
- **Zoeken:** de zoekbalk filtert inkomsten/uitgaven/klanten op tekst (case-insensitive).
- **Animaties:** pagina-inhoud fade-in `jvFade` (0.3s ease, translateY 6px→0). AI-laadstatus pulse.
  Kaarten mogen een subtiele hover-lift krijgen (translateY, versterkte schaduw).
- **Persistentie:** in het prototype localStorage (key `jamey_vault_v1`), in productie een database-write
  per mutatie, gekoppeld aan de ingelogde gebruiker.

## State Management
Eén centrale store (React context / Zustand / of server-state via de database):
`page`, `search`, `businessTab`, `selectedInvoice`, `aiInsight`, `aiLoading`, plus de datacollecties
(income, expenses, clients, invoices, goals, assets, debts, cashflow, settings). Afgeleide waarden
(maand/jaar-totalen, gezondheidsscore, verwachte-betalingen-lijst, per-bedrijf-metrics) worden berekend,
niet opgeslagen. In productie: data ophalen bij login, muteren via API, optimistic UI is prettig.

## Design Tokens
**Kleuren**
- Achtergrond app: `#0A0B0D`
- Sidebar achtergrond: `#0C0D10`
- Kaart-achtergrond: `#131519`
- Kaart-achtergrond (input/veld): `#0A0B0D`
- Randen: `rgba(255,255,255,0.06)` (kaart), `rgba(255,255,255,0.08)` (input), `rgba(255,255,255,0.05)` (lijstscheiding)
- Amber (signatuur/accent): `#D4A855`, hover `#E8C179`, gradient-donker `#8A6A2E`
- Teal (positief/inkomsten): `#3ECF8E`
- Coral (negatief/uitgaven): `#E5676D`
- Blauw (bedrijven-accent / Lumen): `#6AA9E5`
- Tekst primair: `#F5F5F7` / `#ECEDEE`
- Tekst secundair: `#8A8F98` / `#DDE0E4`
- Tekst gedempt: `#5A606B`
- Bedrijfsaccenten: CKJ `#D4A855`, Humphreys Closing `#3ECF8E`, Lumen `#6AA9E5`
- Categorie-badges (achtergrond @ ~12% / tekst): Freelance amber, Consulting teal, Affiliate `#6AA9E5`,
  SaaS `#B98CE5`, Dividend `#E5B36A`, Investeringen `#4FC4C4`, Overig `#8A8F98`
- Factuur "papier": achtergrond `#F7F5F0`, tekst `#1A1A1A`, subtekst `#666`/`#888`, lijnen `#E5E0D5`

**Typografie**
- UI-font: **Inter** (400/500/600/700)
- Bedragen/getallen: **JetBrains Mono** (400/500/600) — gebruik monospace voor ALLE geldbedragen
- Titel pagina: 27px / 700 / letter-spacing -0.01em
- KPI-waarde: 22px / 600 (mono); kleine KPI 17px
- Kaarttitel: 13px / 600
- Body/label: 11–13px
- Sidebar-logo: 15px / 700 / letter-spacing 0.14em ("//"-teken in amber)

**Spacing / vorm**
- Kaart border-radius: 16–18px; kleine kaart 16px; inputs/badges 9–10px; pill-badges 20px
- Kaart-padding: 18–24px
- Grid-gaps: 14–20px
- Sidebar: 248px breed; content max 1180px, padding 34px 40px
- Schaduw kaart: `0 8px 24px rgba(0,0,0,0.35)` (zwaarder accent: `0 8px 30px rgba(0,0,0,0.4)`)
- Voortgangsbalk: hoogte 8–10px, radius 6px, track `#0A0B0D`

**Formatting**
- Valuta: `€ ` + `Number.toLocaleString('nl-NL', {minimumFractionDigits:2, maximumFractionDigits:2})`
- Datum weergave: `DD-MM-YYYY` (opslag `YYYY-MM-DD`)

## Assets
Geen externe afbeeldingen. Icoontjes zijn tekst-glyphs (⌕ ✦ ↻ ⎙ × ✉ ☏) en gekleurde vierkantjes;
grafieken en de gezondheids-ring zijn inline SVG. Het logo is puur tekst ("JAMEY // VAULT" / "JV").
Fonts komen van Google Fonts (Inter, JetBrains Mono). In productie mag je een icon-library
(Lucide/Heroicons) gebruiken in plaats van de glyphs.

## Files
- `Jamey Vault.dc.html` — het volledige interactieve ontwerp (bron: markup + logica). Dit is de
  primaire referentie; open het in een browser om alle gedrag te zien.
- `Jamey Vault (app).html` — dezelfde app als één zelfstandig, offline bestand (gebundeld). Handig om
  snel het eindresultaat te tonen; niet bedoeld om uit te kopiëren.
