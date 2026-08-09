---
name: boersi-screening
description: Nutze diese Skill, wenn mehrere Firmen aus der Boersi-Datenbank gefunden, gefiltert oder verglichen werden sollen — Screening nach Sektor/Land/Vollständigkeit, „welche Firmen gibt es in Sektor X", Vergleich von Kennzahlen über mehrere Titel. Erklärt, wie man search_dossiers + coverage_stats richtig kombiniert, den Hard-Cap von 25 Treffern respektiert, per Cursor paginiert und ohne Bulk-Abfrage sauber zusammenfasst.
---

# Boersi Screening & Vergleich

Für Fragen über **mehrere** Firmen (finden, filtern, vergleichen). Grundwerkzeug ist
`search_dossiers` — der **einzige** Mehr-Firmen-Endpunkt. Er ist bewusst **klein und gedeckelt**.

## Wichtig: kein Bulk, kein Dump

- Es gibt **kein** Tool, das „alle Firmen" oder die ganze Datenbank zurückgibt. Erwarte das nicht
  und kündige es dem Nutzer nicht an.
- `search_dossiers` liefert **maximal 25 Treffer pro Seite** (Hard-Cap, serverseitig erzwungen).
  Höhere `limit`-Werte werden stillschweigend auf 25 gedeckelt.
- Jeder Treffer ist eine **kleine Projektion**: `{isin, firma, sektor, land, gesamt_prozent}` —
  **kein** vollständiges Dossier. Für Details danach gezielt `get_field`/`get_dossier` (siehe Skill
  *boersi-dossier-lookup*).

## Ablauf

1. **Filtern statt alles holen.** Setze die Suche eng: `sektor`, `land` (ISO-2, z. B. `DE`),
   `min_vollstaendigkeit` (min. `gesamt_prozent`). Beispiel:
   `search_dossiers({ land: "DE", sektor: "Technologie", min_vollstaendigkeit: 80 })`.
2. **Paginieren, wenn nötig.** Ist im Ergebnis `nextCursor` gesetzt, gibt es weitere Seiten.
   Nächste Seite: `search_dossiers({ ...gleicheFilter, cursor: nextCursor })`. Wiederhole, bis
   `nextCursor === null`. Halte die Seitenzahl klein — hol nur so viele Seiten, wie die Frage braucht.
3. **Details nachladen — sparsam.** Für einen Kennzahlen-Vergleich pro relevanter ISIN gezielt
   `get_field(isin, "<sektion>.<feld>")` aufrufen, nicht blind ganze Dossiers. Den Feldpfad
   vorher einmal ermitteln (siehe Skill *boersi-dossier-lookup*), nicht raten.
4. **Datenqualität einordnen.** Mit `coverage_stats(isin)` je Kandidat Pflicht%/Gesamt%/Register%
   zeigen, wenn Vergleichbarkeit/Abdeckung relevant ist. `coverage_stats()` ohne ISIN gibt das
   Aggregat über den Bestand (nur drei Zahlen, keine Firmenliste).

## Ergebnisse zusammenfassen

- Als **kompakte Tabelle**: Firma · ISIN · die verglichene Kennzahl **mit `as_of_date`** · ggf.
  `gesamt_prozent`. Uneinheitliche Stichtage explizit kennzeichnen — nie Werte unterschiedlicher
  Stichtage kommentarlos gegenüberstellen.
- **Sag, wenn abgeschnitten wurde.** Wenn du wegen des 25er-Caps oder aus Sparsamkeit nicht alle
  Seiten geladen hast, weise darauf hin („weitere Treffer vorhanden — bei Bedarf nachladen"),
  statt Vollständigkeit vorzutäuschen.
- **Keine Rankings als Empfehlung.** Ein Vergleich ist Faktenlage, keine Kauf-/Verkaufs-Aussage.
  Keine Empfehlungen, Kursziele oder „bester Titel"-Urteile (§32).

## Beispiel

*„Vergleiche den Umsatz der deutschen Tech-Firmen in der DB."*
1. `search_dossiers({ land: "DE", sektor: "Technologie" })` → Treffer + evtl. `nextCursor`.
2. Ggf. weitere Seiten via `cursor` nachladen, bis `nextCursor === null`.
3. Pro Treffer `get_field(isin, "<sektion>.<feld>")` mit dem zuvor ermittelten Umsatz-Pfad.
4. Tabelle: Firma · ISIN · Umsatz (mit Stichtag). Hinweis, falls Seiten offen blieben.
