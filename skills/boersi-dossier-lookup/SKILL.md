---
name: boersi-dossier-lookup
description: Nutze diese Skill, sobald es um Fakten zu einer einzelnen Firma / einem Wertpapier aus der Boersi-Datenbank geht (per ISIN) — Kennzahlen, Bilanz, Dividende, Governance, ESG, Stammdaten, Coverage. Sie erklärt die fünf Boersi-MCP-Tools (get_dossier, get_field, search_dossiers, coverage_stats, list_sections) und wann welches richtig ist. Aktiviert bei Fragen wie „Wie hoch war der Umsatz von SAP?", „Zeig mir das Dossier zu ISIN DE0008404005", „Welche Sektionen gibt es zu Toyota?".
---

# Boersi Dossier-Lookup

Die Boersi-Datenbank liefert **strukturierte Fakten** zu Firmen/Wertpapieren, gegliedert in
**thematische Sektionen** je Firma — u. a. Stammdaten, Geschäftsmodell, Kennzahlen, Bilanz,
Dividende, Aktionärsstruktur, Governance, ESG und Risiken. Welche Sektionen eine Firma
tatsächlich hat, sagt `list_sections`. Adressiert wird eine Firma immer über ihre **ISIN**.

## Die fünf Tools — und wann welches

| Tool | Zweck | Wann nutzen |
|------|-------|-------------|
| `get_dossier(isin)` | **ganzes Dossier** einer Firma | Nutzer will einen Gesamtüberblick / mehrere Sektionen auf einmal |
| `get_field(isin, field_path)` | **ein einzelnes Feld** → `{wert, as_of_date}` | gezielte Einzelfrage („nur der Umsatz") — sparsamste Wahl |
| `search_dossiers({sektor?, land?, min_vollstaendigkeit?}, cursor?, limit≤25)` | **Entdeckung** mehrerer Firmen (kleine Projektion) | ISIN unbekannt; nach Sektor/Land filtern; Kandidaten finden |
| `coverage_stats(isin?)` | die **drei Kennzahlen** Pflicht% / Gesamt% / Register% | Datenqualität/-abdeckung einschätzen (einzeln oder aggregiert) |
| `list_sections(isin)` | **Sektions-Katalog** (id, titel, Pflicht-Flag), ohne Werte | herausfinden, welche `field_path`-Sektionen es gibt |

### Entscheidungsleitfaden

1. **ISIN bekannt + eine konkrete Zahl gefragt?** → `get_field(isin, "<sektion>.<feld>")`.
   **Den Pfad nie raten:** `list_sections(isin)` zeigt die vorhandenen Sektionen, ein einmaliges
   `get_dossier(isin)` die darin enthaltenen Feldnamen. Erst dann gezielt `get_field`.
2. **ISIN bekannt + Gesamtbild gewünscht?** → `get_dossier(isin)`.
3. **ISIN unbekannt / mehrere Firmen?** → `search_dossiers(...)` (siehe Skill *boersi-screening*),
   dann pro Treffer gezielt `get_field`/`get_dossier`.
4. **Frage nach Vollständigkeit/Datenqualität?** → `coverage_stats(isin)` (oder ohne ISIN fürs Aggregat).

## Feste Regeln bei der Nutzung

- **Immer den Stichtag nennen.** Jeder Feldwert kommt als `{wert, as_of_date}` — gib den Wert
  nie ohne sein `as_of_date` wieder („Umsatz 34,2 Mrd. € — Stand 2024-12-31").
- **Keine Anlageberatung (§32).** Nenne **keine** Empfehlungen, Kursziele, Kauf-/Verkaufs-Urteile.
  Der Server liefert solche Felder bewusst nicht — erfinde sie auch nicht. Bleib bei Fakten.
- **Quellen erscheinen bewusst nicht im Output.** Interne Provenienz (Quellenverweise, Register-/
  Objekt-IDs) wird serverseitig entfernt. Frage nicht danach und behaupte keine Einzelquellen.
- **Zahlenformat:** Werte kommen im DB-Format mit Punkt-Dezimal (z. B. `34.2`). Für deutsche
  Ausgabe darfst du auf Komma umstellen (`34,2`) und die Einheit aus dem `field_path`/Kontext
  ergänzen (z. B. `_mrd_eur` → „Mrd. €").
- **Sparsam bleiben.** Für eine Einzelzahl `get_field` statt `get_dossier`. Nur so viele Sektionen
  holen, wie die Frage braucht.
- **Not-found sauber behandeln.** Existiert eine ISIN/ein Feld nicht, sag das klar, statt zu raten.

## Beispiele

- *„Wie hoch war der Umsatz von SAP (DE0007164600)?"*
  → Feldpfad für den Umsatz ermitteln, dann `get_field("DE0007164600", "<sektion>.<feld>")`
  → „34,2 Mrd. € (Stand 2024-12-31).
- *„Gib mir einen Überblick über Allianz, ISIN DE0008404005."*
  → `get_dossier("DE0008404005")` → Sektionen zusammenfassen, je Wert den Stichtag nennen.
- *„Welche Bereiche sind zu Toyota (JP3633400001) erfasst?"*
  → `list_sections("JP3633400001")`.
- *„Wie vollständig ist das Allianz-Dossier?"*
  → `coverage_stats("DE0008404005")` → Pflicht% / Gesamt% / Register% erläutern.
