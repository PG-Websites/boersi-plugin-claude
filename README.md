# Boersi — Claude-Plugin

Verbindet Claude mit der **Boersi Finanz-Dossier-Datenbank**: strukturierte Fakten zu
Aktien, ETFs und weiteren Wertpapieren — je Firma.

Das Plugin ist eine dünne Hülle. Es enthält keinen Backend-Code, keine Daten und keine
Zugangsdaten; es verweist per URL auf den gehosteten Boersi-Server.

> **Reine Faktenauskunft, keine Anlageberatung.** Boersi liefert keine Empfehlungen,
> Kauf-/Verkaufsurteile oder Kursziele (§32 KWG).

## Installation

1. **Server-URL setzen:**

   ```bash
   export BOERSI_MCP_URL="https://mcp.boersi.eu/mcp"
   ```

   Die `.mcp.json` liest diese Variable. Ohne sie lädt das Plugin, verbindet aber nicht.

2. **Plugin prüfen und installieren:**

   ```bash
   claude plugin validate .
   ```

   Danach über die Plugin-Verwaltung installieren und aktivieren.

3. **Verbinden:** Beim ersten Tool-Aufruf startet Claude den OAuth-Login automatisch
   (OAuth 2.1 mit PKCE). Es gehören keine Tokens oder Zugangsdaten in dieses Repo.

4. **Boersi-Konto erforderlich.** Ohne angemeldetes Konto antwortet der Server mit `401`;
   der Login-Flow führt zur Konto-Verknüpfung.

## Skills

| Skill | Zweck |
|-------|-------|
| `boersi-dossier-lookup` | Grundlagen: Fakten zu **einer** Firma und die Wahl des richtigen Tools |
| `boersi-screening` | Grundlagen: mehrere Firmen finden, filtern, seitenweise durchgehen |
| `boersi-tearsheet` | Erzeugt ein sauberes Einseiten-Firmenprofil |
| `boersi-peer-vergleich` | Stellt mehrere Firmen strukturiert nebeneinander (Kennzahlen, Marktanteile, Moat) |
| `boersi-lieferketten-karte` | Baut die Zulieferer- und Kundenkarte einer Firma |
| `boersi-index-screener` | Filtert Firmen eines Index nach Kriterien und sortiert sie |
| `boersi-sektor-ueberblick` | Erstellt eine Branchen-Landschaft mit Vergleichsgruppe |

Die Skills bringen Claude bei, die fünf **ausschließlich lesenden** Tools korrekt zu nutzen:
`get_dossier`, `get_field`, `search_dossiers`, `coverage_stats` und `list_sections`.
Die fünf Aufgaben-Skills bauen auf den beiden Grundlagen-Skills auf.

## Datenschutz und Sicherheit

- **Nur lesend.** Alle Tools sind read-only. Es gibt kein Tool, das schreibt oder löscht.
- **Kein Massenabruf.** Es gibt keinen Export und keinen Datenbank-Dump. Mehr-Firmen-Abfragen
  sind pro Seite begrenzt und werden seitenweise durchgereicht.
- **Keine Anlageberatung.** Empfehlungen, Kursziele und Anlageurteile werden serverseitig
  herausgefiltert (§32 KWG).
- **Anmeldung über OAuth 2.1** mit PKCE. Tokens hält dein Client, nicht das Plugin.
- **Datenhaltung in der EU.**

**Datenschutzerklärung (Privacy Policy):** https://boersi.app/rechtliches/datenschutz

Bei der Nutzung verarbeitet Boersi deine Kontoidentität und Plan-Stufe, Nutzungs-Metadaten
je Tool-Aufruf (Zeitpunkt, Tool, Ergebnis, abgefragtes Wertpapier) sowie Verbrauchszähler.
Chatverläufe, Prompts und Antworttexte werden **nicht** an Boersi übertragen. Maßgeblich ist
die verlinkte Datenschutzerklärung.

## Lizenz

MIT — siehe [LICENSE](LICENSE). Die Lizenz deckt diese Plugin-Hülle, nicht die über den
Server bereitgestellten Daten.
