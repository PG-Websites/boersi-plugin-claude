---
name: boersi-index-screener
description: Filtert Firmen eines Index nach Kriterien und sortiert sie. Nutze diese Skill für Screening-Fragen mit Index-Bezug — DAX, MDAX, EURO STOXX, S&P 500 — etwa „welche DAX-Firmen erfüllen Kriterium X" oder „sortiere die MDAX-Titel nach Y". Aktiviert bei Fragen wie „Welche DAX-Werte haben die höchste Datenabdeckung?", „Screene den MDAX nach Sektor Industrie", „Sortiere die deutschen Index-Titel nach Umsatz".
---

# Boersi Index-Screener

Screening mit Index-Bezug — mit einer Einschränkung, die du **immer** mitkommunizieren musst.

## Index-Mitgliedschaft ist KEIN Suchfilter

`search_dossiers` filtert **ausschließlich** nach `sektor`, `land` und `min_vollstaendigkeit`.
Es gibt **keinen** `index`-Parameter und **keine** Namenssuche. Die Index-Zugehörigkeit steht nur
**pro Firma** als einzelnes Feld in der Stammdaten-Sektion des Dossiers — den genauen Pfad
ermittelst du einmal über `list_sections`/`get_dossier` und verwendest ihn dann wieder.

Daraus folgt: Ein Index lässt sich **nicht** abfragen, nur **firmenweise prüfen**. Behaupte
niemals, du hättest „alle DAX-Firmen" oder „den kompletten Index" abgedeckt.

## Zwei Wege — beide legitim, beide offenzulegen

| Weg | Wann | Vorgehen |
|-----|------|----------|
| **A — Konstituenten-Liste** (bevorzugt) | Der Nutzer nennt die Firmen oder ISINs | Je ISIN das Index-Feld per `get_field` bestätigen, dann die Screening-Felder laden |
| **B — Proxy-Screening** | Der Nutzer nennt nur den Index | Über `sektor`/`land` eingrenzen, Treffer laden, Mitgliedschaft je Kandidat per `get_field` prüfen |

Bei **B** ist `land` der beste Proxy (DAX/MDAX → `land: "DE"`), aber eben nur ein Proxy: der
Filter findet auch nicht-gelistete Firmen und verfehlt Index-Mitglieder mit anderem Sitzland.

### Mitgliedschaft prüfen — die Regel

Prüfe die Zugehörigkeit **je Kandidat** per `get_field` und nimm nur bestätigte Treffer in die
Ergebnistabelle. Deckle die Prüfung bei **25 Kandidaten** (ein Aufruf je Kandidat, und das
Rate-Limit liegt bei 15/60/240 Calls pro Minute je nach Plan). Was du nicht geprüft hast, kommt
nicht als Index-Mitglied in die Tabelle — führe es allenfalls getrennt als „ungeprüft" auf.
Liefert das Feld nichts, gilt die Firma als **nicht bestätigt**, nicht als Mitglied.

## Ablauf

1. **Index und Kriterium klären.** Kriterium = wonach gefiltert (Sektor, Land, Abdeckung, eine
   Kennzahl) und wonach sortiert wird.
2. **Kandidaten beschaffen** — Weg A oder B. Bei B: `search_dossiers({ land, sektor? })`,
   max. **25 Treffer pro Seite** (Plan `plus`: 10), weitere Seiten nur per `cursor` und nur so
   viele, wie nötig. Nennt der Nutzer Firmennamen statt ISINs, gleiche sie gegen das Feld `firma`
   in der Trefferprojektion ab — eine Namenssuche gibt es nicht.
3. **Mitgliedschaft prüfen** (siehe Regel oben).
4. **Screening-Feld laden.** Ist das Kriterium eine Kennzahl, Feldpfad einmal per `get_dossier` an
   einer Referenzfirma lernen und dann je Kandidat `get_field` — nie ein Dossier pro Firma
   (`get_dossier` ist kontingentiert, `plus`: 5/Monat). Ist das Kriterium die Datenabdeckung,
   genügt `gesamt_prozent` aus der Suchprojektion — **kein** zusätzlicher Call.
5. **Sortieren und ausgeben.** Sortiert wird **clientseitig über die geladenen Kandidaten** —
   der Server sortiert nicht. Tabelle: Firma · ISIN · Kriteriumswert **mit `as_of_date`** ·
   ggf. `gesamt_prozent`.

## Feste Regeln

- **Grundgesamtheit offenlegen.** Schreib dazu, worüber sortiert wurde: „geprüft wurden 18 von
  dir genannte Titel" bzw. „Kandidaten aus `land: DE`, davon 12 als Index-Mitglied bestätigt".
  Eine Rangliste ohne offengelegte Grundgesamtheit ist irreführend.
- **Nie „alle Firmen" behaupten.** Weder für den Index noch für die Datenbank. Ist `nextCursor`
  gesetzt und nicht weitergeladen, sag es.
- **Index-Zugehörigkeit nicht aus Wissen ergänzen.** Nicht aus dem Gedächtnis entscheiden, wer im
  DAX ist — nur das Feld zählt. Stichtag der Zugehörigkeit mit ausgeben; Index-Zusammensetzungen
  ändern sich.
- **Sortierung ist kein Ranking im Sinne einer Empfehlung (§32).** Keine Kursziele, keine
  „Top-Picks", kein „bester Wert im Index". Sortieren nach einer Faktenspalte ist erlaubt,
  daraus eine Anlageaussage abzuleiten nicht.
- **Jeder Wert mit `as_of_date`**, abweichende Stichtage kennzeichnen — sonst vergleicht die
  Sortierung Äpfel mit Birnen.
- **Kein Bulk.** Keine Seiten erschöpfen, um eine Gesamtliste zu erzeugen; keine Ausgabe, die als
  Index-Export dient.
- **Keine Quellen im Output.** Interne Provenienz wird serverseitig entfernt.

## Beispiel

*„Welche deutschen Index-Titel im Sektor Industrie haben die beste Datenabdeckung?"*

1. `search_dossiers({ land: "DE", sektor: "Industrie", min_vollstaendigkeit: 50 })`.
2. Je Treffer `get_field(isin, "<sektion>.<feld>")` mit dem ermittelten Index-Pfad
   → bestätigte Mitglieder behalten, Rest getrennt als „ungeprüft/nicht bestätigt" ausweisen.
3. Sortieren nach `gesamt_prozent` aus der Projektion — kein weiterer Call nötig.
4. Tabelle Firma · ISIN · Index (mit Stichtag) · Abdeckung, darunter der Satz, wie viele
   Kandidaten geprüft wurden und dass offene Seiten existieren.
