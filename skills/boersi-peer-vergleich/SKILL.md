---
name: boersi-peer-vergleich
description: Stellt mehrere Firmen strukturiert nebeneinander (Kennzahlen, Marktanteile, Moat). Nutze diese Skill, wenn zwei oder mehr Firmen direkt verglichen werden sollen — Peer-Group, Konkurrenzvergleich, „X gegen Y", Kennzahlen-Gegenüberstellung. Aktiviert bei Fragen wie „Vergleiche SAP und Microsoft", „Wie steht BMW gegenüber seinen Peers da?", „Stell mir die drei größten deutschen Versicherer gegenüber".
---

# Boersi Peer-Vergleich

Mehrere Firmen **spaltenweise** gegenüberstellen: dieselben Felder, derselbe Aufbau, jeder Wert
mit Stichtag. Der Vergleich ist eine **Faktentabelle**, kein Ranking und kein Urteil.

## Tools

| Tool | Rolle im Vergleich |
|------|--------------------|
| `search_dossiers({sektor?, land?, min_vollstaendigkeit?}, cursor?)` | Peer-**Kandidaten** finden, wenn der Nutzer keine Liste vorgibt |
| `get_dossier(isin)` | **einmal** für die Referenzfirma — liefert die exakten `field_path`s |
| `get_field(isin, field_path)` | **das Arbeitspferd** — je Firma × je Vergleichsfeld ein Aufruf |
| `coverage_stats(isin)` | Vergleichbarkeit einordnen, wenn die Abdeckung stark schwankt |

## Warum nicht N × `get_dossier`

`get_dossier` ist kontingentiert (Plan `plus`: **5 Dossiers/Monat**), `get_field` nur
rate-limitiert. Ein Vergleich über 8 Firmen als 8 Dossiers verbrennt das Monatskontingent in
einem Aufruf. Deshalb:

> **Ein** `get_dossier` auf die Referenzfirma, um die genauen Feldpfade zu lernen —
> danach dieselben Pfade per `get_field` für alle übrigen Firmen.

## Ablauf

1. **Peer-Set festlegen.** Nennt der Nutzer die Firmen, nimm genau die. Sonst
   `search_dossiers({ sektor, land })` und wähle daraus eine **begründete** Auswahl.
2. **Set klein halten.** Richtwert **≤ 8 Firmen × ≤ 5 Felder**. Der Aufwand ist das Produkt
   beider Zahlen und läuft sonst gegen das Rate-Limit (`free` 15, `plus` 60, `pro` 240 Calls/Min).
3. **Feldpfade bestimmen.** `get_dossier` auf die erste Firma und die Pfade der gewünschten
   Kennzahlen ablesen. Ein Pfad hat die Form `<sektion>.<feld>` und kann mehrstufig sein —
   **nie raten**, immer aus der Dossier-Antwort übernehmen.
4. **Dimensionen wählen** — passend zur Frage, typischerweise:

   | Dimension | Woraus |
   |-----------|--------|
   | Kennzahlen (Umsatz, Margen, Ergebnis je Aktie, Cashflow) | Sektion mit den Finanzkennzahlen |
   | Marktanteil / Marktgröße | Sektion zur Marktposition |
   | Moat (Marke, Kostenführerschaft, Netzwerkeffekte) | Sektion zum wirtschaftlichen Burggraben |
   | Segment- und Länderstruktur | Sektion zu den Geschäftsbereichen |
   | Wettbewerbsumfeld | Sektion zur Konkurrenz |

5. **Je Firma × Feld `get_field`.** Fehlt ein Feld bei einer Firma: Zelle „nicht erfasst" —
   **nicht** durch einen ähnlichen Wert ersetzen und **nicht** aus anderen Zeilen schätzen.
6. **Tabelle bauen:** Zeilen = Kennzahl, Spalten = Firma. Stichtag je Zelle (oder je Zeile, wenn
   einheitlich).

## Feste Regeln

- **Stichtage müssen zusammenpassen.** Werte mit unterschiedlichen `as_of_date` niemals
  kommentarlos nebeneinanderstellen — Abweichung ausdrücklich markieren („Stand 2024-12-31 vs.
  2023-12-31, eingeschränkt vergleichbar").
- **Keine Einheiten mischen.** Unterschiedliche Währungen oder Bezugsgrößen kennzeichnen, nicht
  umrechnen. Der Server liefert keine Wechselkurse.
- **Kein Ranking als Empfehlung (§32).** „Höchste Marge" ist eine Beobachtung, „bester Titel",
  „attraktivste Bewertung" oder ein Kursziel sind es nicht. Keine Kauf-/Verkaufs-Aussagen.
- **Kein Bulk.** Der Vergleich bleibt bei der angefragten Auswahl. Kein „alle Firmen des Sektors",
  keine Tabelle über Dutzende Titel — `search_dossiers` liefert max. 25 Treffer pro Seite.
- **Auswahl offenlegen.** Sag, **wie** die Peer-Gruppe zustande kam (Nutzer-Vorgabe oder
  Sektor-/Land-Filter) und dass sie nicht der vollständige Wettbewerb ist.
- **Keine Quellen im Output.** Interne Provenienz wird serverseitig entfernt.

## Beispiel

*„Vergleiche SAP und Microsoft bei Umsatz und EBIT-Marge."*

1. `get_dossier("DE0007164600")` → Feldpfade für Umsatz und EBIT-Marge ablesen.
2. `get_field("DE0007164600", "<umsatz-pfad>")`, `get_field("DE0007164600", "<ebitmarge-pfad>")`.
3. Dieselben zwei Pfade für `US5949181045` per `get_field`.
4. Tabelle: Zeilen Umsatz/EBIT-Marge, Spalten SAP/Microsoft, je Zelle Wert + Stichtag.
   Abweichende Geschäftsjahre ausdrücklich vermerken. Kein Urteil, welche Firma „besser" ist.
