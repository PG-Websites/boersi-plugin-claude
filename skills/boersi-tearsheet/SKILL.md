---
name: boersi-tearsheet
description: Erzeugt aus dem Dossier ein sauberes Einseiten-Firmenprofil. Nutze diese Skill, wenn der Nutzer einen kompakten Steckbrief / ein Tearsheet / ein Einseiten-Profil zu genau einer Firma will — Stammdaten, Geschäftsmodell, Kennzahlen, Segmente, Wettbewerb auf einen Blick. Aktiviert bei Fragen wie „Mach mir ein Tearsheet zu SAP", „Einseiten-Profil für ISIN DE0007664039", „Firmensteckbrief Allianz", „Fass Volkswagen kompakt zusammen".
---

# Boersi Tearsheet (Einseiten-Firmenprofil)

Ein **Tearsheet** ist ein verdichtetes Faktenblatt zu **einer** Firma: alles Wesentliche auf
einer Seite, jeder Wert mit Stichtag. Grundlage ist **ein einziger** `get_dossier`-Aufruf.

## Tools

| Tool | Rolle im Tearsheet |
|------|--------------------|
| `get_dossier(isin)` | **die eine Quelle** — liefert alle Sektionen auf einmal |
| `coverage_stats(isin)` | Kopfzeile „Datenabdeckung" (Pflicht% / Gesamt% / Register%) |
| `list_sections(isin)` | nur bei Zweifel, ob die Firma überhaupt erfasst ist |
| `get_field(isin, field_path)` | **statt** des Tearsheets, wenn nur 1–3 Fakten gefragt sind |

> **`get_dossier` ist kontingentiert** (Plan `plus`: 5 Dossiers/Monat, `free`: gar keins).
> Rufe es für ein Tearsheet **genau einmal** auf und baue das Blatt vollständig aus dieser
> einen Antwort. Nachladen einzelner Felder danach nur per `get_field`.

## Ablauf

1. **ISIN klären.** Ohne ISIN erst über `search_dossiers` finden (siehe Skill *boersi-screening*).
2. **`get_dossier(isin)`** — einmal. Die Antwort enthält die thematischen Sektionen der Firma.
3. **`coverage_stats(isin)`** für die Abdeckungszeile.
4. **Blatt bauen** — feste Reihenfolge, damit Tearsheets vergleichbar bleiben:

   | Block | Speist sich aus |
   |-------|-----------------|
   | **Kopf** | Firma · ISIN · Sektor · Land · Abdeckung aus `coverage_stats` |
   | **Stammdaten** | Rechtsform, Hauptsitz, Gründungsjahr, Mitarbeiterzahl, Börsen/Index |
   | **Geschäftsmodell** | Kurzbeschreibung, Wertversprechen, Einnahmequellen |
   | **Kennzahlen** | Umsatz, EBIT-/EBITDA-Marge, Nettomarge, Ergebnis je Aktie, Free Cashflow |
   | **Segmente** | Segment- und Länderumsätze samt Anteilen |
   | **Wettbewerb** | Wettbewerbsumfeld und Marktstruktur, Marktanteil |

   Die Zuordnung Block → Sektion ergibt sich aus der Dossier-Antwort; feste Pfade gibt es nicht.

5. **Lücken kennzeichnen.** Fehlt ein Block im Dossier, schreib „nicht erfasst" — nicht weglassen,
   nicht aus anderen Feldern herleiten.

## Feste Regeln

- **Jeder Wert mit `as_of_date`.** Ein Tearsheet ohne Stichtage ist wertlos. Steht in einem Block
  ein einheitlicher Stichtag, nenn ihn einmal als Block-Kopf; sonst je Zeile.
- **Nie schätzen, nie rechnen, was nicht dasteht.** Keine abgeleiteten Kennzahlen (kein selbst
  gebildetes KGV, keine hochgerechneten Jahreswerte), keine Peer-Einordnung aus dem Gedächtnis.
- **Keine Empfehlungen, keine Kursziele (§32).** Das Tearsheet endet bei der Faktenlage. Die
  Analysten-Sektion liefert bewusst keine Urteile — erfinde keine, auch nicht als „Einschätzung".
- **Keine Quellenangaben.** Interne Provenienz wird serverseitig entfernt; der Stichtag ist der
  Beleg. Frage nicht danach und behaupte keine Einzelquelle.
- **Ein Tearsheet = eine Firma.** Für mehrere Firmen nebeneinander → Skill *boersi-peer-vergleich*.
- **Zahlenformat:** Punkt-Dezimal aus der DB für die deutsche Ausgabe auf Komma umstellen; Einheit
  und Währung aus dem Feld übernehmen, nie umrechnen.

## Beispiele

- *„Mach mir ein Tearsheet zu Volkswagen (DE0007664039)."*
  → `get_dossier("DE0007664039")` + `coverage_stats("DE0007664039")` → Blatt in obiger Reihenfolge.
- *„Nur der Umsatz von SAP."* → **kein** Tearsheet: `get_field(...)` (Skill *boersi-dossier-lookup*)
  — spart das Dossier-Kontingent.
- *„Tearsheet zu einer Firma ohne Kennzahlen-Sektion."*
  → Block „Kennzahlen: nicht erfasst" ausweisen, Rest normal bauen.
