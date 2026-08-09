---
name: boersi-lieferketten-karte
description: Baut die Zulieferer- und Kundenkarte einer Firma. Nutze diese Skill für Fragen zur Lieferkette, zu Zulieferern, Lieferanten, Abhängigkeiten, Produktionsstandorten, Rohstoffen oder zur Kundenstruktur einer Firma. Aktiviert bei Fragen wie „Wer beliefert Volkswagen?", „Zeig mir die Lieferkette von BMW", „Von welchen Zulieferern hängt die Firma ab?", „Wer sind die größten Kunden von Infineon?".
---

# Boersi Lieferketten-Karte

Die Karte hat **zwei Seiten**: *upstream* (Zulieferer, Standorte, Rohstoffe) aus der
**Lieferketten-Sektion** und *downstream* (Kunden, Konzentration, Kanäle) aus der
**Kunden-Sektion** der Firma.

## Wichtig: die beiden Seiten sind unterschiedlich gut erfasst

| Seite | Datenlage | Konsequenz |
|-------|-----------|------------|
| **Upstream** | Die Lieferketten-Sektion kann eine **strukturierte Zulieferer-Liste** enthalten — Einträge mit Partnername, Typ, Kategorie und Stichtag. Vorhanden für einen **Teil** der Firmen, nicht für alle. | Namentliche Zulieferer sind belastbar auflistbar — **wenn** das Feld existiert. |
| **Downstream** | Für die Kundenseite gibt es in aller Regel **kein** namentliches Register, sondern Fließtext plus Rahmen- und Konzentrationsangaben. | Kunden **nur** so wiedergeben, wie der Text sie nennt. Keine Kundenliste konstruieren. |

Diese Asymmetrie **musst du dem Nutzer sagen**, wenn er nach beiden Seiten fragt. Eine
symmetrische „Zulieferer ↔ Kunden"-Tabelle vorzutäuschen, wo nur eine Seite Struktur hat, ist
ein Fehler.

## Tools

| Tool | Rolle |
|------|-------|
| `list_sections(isin)` | prüfen, ob Lieferketten- und Kunden-Sektion überhaupt erfasst sind |
| `get_dossier(isin)` | wenn beide Seiten + Standorte/Rohstoffe/Abhängigkeiten gebraucht werden |
| `get_field(isin, "<sektion>.<feld>")` | **sparsamster Weg**, wenn nur die Zulieferer-Liste gefragt ist |

> `get_dossier` liefert **immer das ganze Dossier** — es gibt keinen Sektions-Parameter. Und es
> ist kontingentiert (`plus`: 5/Monat). Reicht ein einzelnes Feld, nimm `get_field` — den
> genauen Pfad vorher aus `list_sections`/`get_dossier` ermitteln, **nie raten**.

## Ablauf

1. **Umfang klären.** Nur Zulieferer und der Pfad ist bekannt? → ein `get_field`. Ganze Karte
   inkl. Standorten, Rohstoffen, Abhängigkeiten und Kundenseite? → ein `get_dossier`.
2. **Vorhandensein prüfen** mit `list_sections(isin)` — fehlt die Lieferketten-Sektion, sag das
   direkt, statt sie über andere Sektionen zu rekonstruieren.
3. **Upstream strukturieren** aus der Lieferketten-Sektion, soweit vorhanden:

   | Bereich | Inhalt |
   |---------|--------|
   | Zulieferer-Liste | namentliche Partner mit Typ, Kategorie und Stichtag → Kernliste |
   | Wichtigste Lieferanten | Anzahlen je Kategorie (Rohstoff / Komponente / Auftragsfertiger), Abdeckungsgrad |
   | Abhängigkeiten | Single-Source-Abhängigkeiten, geografische Konzentration, Dual-Sourcing |
   | Standorte, Rohstoffe, Logistik | Fertigungsmodell, Produktionsstandorte, Materialexposure |
   | Profil | Fließtext-Überblick zur Lieferkette |

4. **Downstream strukturieren** aus der Kunden-Sektion: Fließtext-Profil, Offenlegungsgrad und
   Anzahl namentlich genannter Kunden, Angaben zur Kundenkonzentration bzw. zum Klumpenrisiko,
   Kundentyp und Kanalanteile sowie regionale Verteilung.
5. **Als Karte ausgeben:** zwei Abschnitte („Zulieferer / Upstream", „Kunden / Downstream"),
   die Partner als Liste oder Tabelle mit Kategorie und Stichtag, darunter Abhängigkeiten und
   Konzentration als kurze Faktenzeilen.

## Feste Regeln

- **Nur Partner nennen, die im Feld stehen.** Keine Zulieferer aus Branchenwissen ergänzen, keine
  „typischen" Lieferanten annehmen, keine Beziehung aus einem Rohstoff ableiten.
- **Stichtag je Beziehung.** Jeder Eintrag trägt einen eigenen Stichtag — mit ausgeben.
  Uneinheitliche Stände kennzeichnen.
- **Abdeckung ehrlich benennen.** Die Sektion führt Angaben dazu, wie vollständig die Offenlegung
  ist. Eine Liste mit sieben Partnern ist **nicht** „die Lieferkette", sondern der offengelegte
  Ausschnitt — sag das.
- **Fehlt die Liste, gibt es keine Liste.** Dann nur den Fließtext wiedergeben und klar sagen,
  dass keine namentlichen Zulieferer erfasst sind. Nicht ausweichen, nicht erfinden.
- **Keine firmenübergreifende Kette bauen.** Zulieferer nicht ihrerseits nachschlagen, um ein
  mehrstufiges Netz zu zeichnen — das wäre Bulk-Nutzung. Eine Karte = eine Firma.
- **Keine Risikobewertung als Anlageurteil (§32).** Abhängigkeiten und Klumpenrisiken sind Fakten
  aus der Offenlegung. Keine Schlussfolgerung auf Kurs, Bewertung oder Kauf/Verkauf.
- **Keine Quellen im Output.** Der Stichtag ist der Beleg.

## Beispiele

- *„Wer beliefert Volkswagen (DE0007664039)?"*
  → Pfad der Zulieferer-Liste ermitteln, dann `get_field("DE0007664039", "<sektion>.<feld>")`
  → Partner mit Kategorie und Stichtag auflisten, dazu der Hinweis, dass das die offengelegten
  Zulieferer sind.
- *„Komplette Lieferketten- und Kundenkarte für BMW."*
  → `get_dossier(isin)` → Upstream-Abschnitt aus der Lieferketten-Sektion, Downstream-Abschnitt
  aus der Kunden-Sektion, dabei sagen, dass die Kundenseite meist nur als Text und
  Konzentrationsangaben vorliegt.
- *„Von welchen Rohstoffen hängt die Firma ab?"*
  → Rohstoff- und Abhängigkeitsangaben der Lieferketten-Sektion mit Stichtag wiedergeben;
  fehlt beides, klar „nicht erfasst" sagen.
