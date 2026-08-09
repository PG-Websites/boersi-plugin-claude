---
name: boersi-sektor-ueberblick
description: Erstellt eine Branchen-Landschaft mit Vergleichsgruppe. Nutze diese Skill, wenn der Nutzer einen Überblick über einen Sektor oder eine Branche will — welche Firmen dort erfasst sind, wie die Landschaft aussieht, und eine sinnvolle Vergleichsgruppe daraus. Aktiviert bei Fragen wie „Gib mir einen Überblick über den Versicherungssektor", „Wie sieht die Tech-Branche in Deutschland aus?", „Welche Firmen gibt es im Sektor Grundstoffe?".
---

# Boersi Sektor-Überblick

Eine **Branchen-Landschaft**: welche Firmen des Sektors in der Datenbank liegen, wie sie sich
nach Land und Abdeckung verteilen, und eine daraus gebildete **Vergleichsgruppe** mit wenigen
Kennzahlen. Der Überblick beschreibt **den erfassten Bestand**, nicht den Markt.

## Tools

| Tool | Rolle |
|------|-------|
| `search_dossiers({sektor, land?, min_vollstaendigkeit?}, cursor?)` | **Einstieg** — liefert die Landschaft als Projektion `{isin, firma, sektor, land, gesamt_prozent}` |
| `coverage_stats()` / `coverage_stats(isin)` | Aggregat-Abdeckung bzw. Abdeckung einzelner Kandidaten |
| `get_field(isin, field_path)` | gezielt Kennzahlen für die Vergleichsgruppe nachladen |
| `get_dossier(isin)` | **einmal** für eine Referenzfirma, um die exakten Feldpfade zu lernen |

## Ablauf

1. **Sektor-Bezeichnung treffen.** `sektor` ist ein Freitext-Filter und muss zur Schreibweise im
   Bestand passen (z. B. `Technologie`, `Versicherung`, `Finanzen`, `Grundstoffe`,
   `Konsum zyklisch`, `Basiskonsumgueter`, `Industrie`). Liefert die Suche nichts, probiere eine
   naheliegende Schreibweise und sag dem Nutzer, welche Bezeichnung getroffen hat.
2. **Landschaft holen.** `search_dossiers({ sektor })`, bei Bedarf zusätzlich `land` (ISO-2) oder
   `min_vollstaendigkeit`. **Max. 25 Treffer pro Seite** (Plan `plus`: 10). Weitere Seiten nur per
   `cursor`, und nur so viele, wie die Frage wirklich braucht.
3. **Landschaft beschreiben** — allein aus der Projektion, ohne Zusatz-Calls:
   - Anzahl der gefundenen Firmen (und ob weitere Seiten offen sind),
   - Verteilung nach `land`,
   - Spannweite der Abdeckung (`gesamt_prozent`), Median/Beste grob benennen.
4. **Vergleichsgruppe bilden** — **explizit begründet**, typischerweise 3–6 Firmen nach einem
   nachvollziehbaren Kriterium (gleiches Land, höchste Abdeckung, vom Nutzer genannt). Nenne das
   Kriterium in der Ausgabe.
5. **Kennzahlen nachladen.** Feldpfade einmal per `get_dossier` an einer Referenzfirma lernen,
   dann pro Gruppenmitglied `get_field` — nicht pro Firma ein Dossier ziehen
   (`get_dossier` ist kontingentiert, `plus`: 5/Monat). Sinnvolle Dimensionen:

   | Dimension | Woraus |
   |-----------|--------|
   | Umsatz, Margen, Ergebnis | Sektion mit den Finanzkennzahlen |
   | Marktanteil / Marktgröße | Sektion zur Marktposition |
   | Wettbewerbsumfeld, Marktstruktur | Sektion zur Konkurrenz |
   | Mitarbeiterzahl, Sitz, Index | Sektion mit den Stammdaten |

6. **Ausgeben:** erst die Landschaft (Zahlen + Verteilung), dann die Vergleichsgruppe als Tabelle
   mit Stichtagen. Für eine tiefere Gegenüberstellung → Skill *boersi-peer-vergleich*.

## Feste Regeln

- **Nie „der ganze Sektor".** Du siehst den **erfassten Bestand**, gefiltert und auf 25 Treffer je
  Seite gedeckelt. Formuliere entsprechend: „in der Datenbank erfasst" statt „im Sektor gibt es".
- **Abbruch offenlegen.** Ist `nextCursor` gesetzt und du hast nicht weitergeladen, sag es
  („weitere Treffer vorhanden"). Keine Vollständigkeit vortäuschen.
- **Auswahlkriterium nennen.** Eine Vergleichsgruppe ohne offengelegtes Kriterium ist eine
  versteckte Wertung. Sag, warum genau diese Firmen drin sind.
- **Abdeckung ≠ Qualität der Firma.** `gesamt_prozent` misst, wie vollständig das **Dossier** ist —
  nicht, wie gut das Unternehmen dasteht. Nie als Bewertung verwenden.
- **Jeder Wert mit `as_of_date`**, unterschiedliche Stichtage kennzeichnen.
- **Keine Empfehlungen, keine Kursziele, kein „Top-Pick" (§32).** Auch kein implizites Ranking
  („die attraktivsten Titel des Sektors").
- **Kein Bulk.** Keine Erschöpfung aller Seiten, nur um eine lange Liste zu erzeugen.
- **Keine Quellen im Output.** Interne Provenienz wird serverseitig entfernt.

## Beispiel

*„Gib mir einen Überblick über deutsche Technologie-Firmen."*

1. `search_dossiers({ sektor: "Technologie", land: "DE" })` → Trefferliste + evtl. `nextCursor`.
2. Landschaft beschreiben: Anzahl, Abdeckungs-Spannweite, Hinweis auf offene Seiten.
3. Vergleichsgruppe: die 4 Firmen mit der höchsten `gesamt_prozent` — Kriterium nennen.
4. `get_dossier` auf die erste Firma für die Feldpfade, dann `get_field` für Umsatz und
   EBIT-Marge je Gruppenmitglied.
5. Tabelle mit Stichtagen; kein Urteil, welcher Titel „der beste" ist.
