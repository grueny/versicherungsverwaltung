# Use Cases – Sparte: KFZ (basierend auf LVM Versicherung)

> Spartenspezifische Use Cases, die nur in der KFZ-Sparte vorkommen oder sich wesentlich vom spartenübergreifenden Kernprozess unterscheiden.  
> Quelle: Öffentlich zugängliche Produktinformationen der LVM Versicherung (lvm.de, Stand März 2026).  
> Spartenübergreifende Use Cases sind unter `anforderungen/05_funktionale_anforderungen/` erfasst.  
> Nutze `anforderungen/05_funktionale_anforderungen/_template.md` als Vorlage für neue Use Cases.

## Use-Case-Verzeichnis

### Kernprozesse KFZ

| UC-Nr. | Titel | Priorität | Status |
|--------|-------|-----------|--------|
| [UC-KFZ-00](UC-KFZ-00_fahrzeugart_auswaehlen.md) | Fahrzeugart auswählen und Angebot eröffnen | 🔴 Muss | ✅ Fertig |
| UC-KFZ-01 | Fahrzeugdaten erfassen und Typklasse ermitteln | 🔴 Muss | 🔲 Offen |
| UC-KFZ-02 | eVB-Nummer erzeugen | 🔴 Muss | 🔲 Offen |
| UC-KFZ-03 | SF-Klasse übernehmen / nachweisen | 🔴 Muss | 🔲 Offen |
| UC-KFZ-04 | Fahrzeugwechsel durchführen | 🔴 Muss | 🔲 Offen |
| UC-KFZ-05 | SF-Rückstufung nach Schadenfall | 🔴 Muss | 🔲 Offen |
| UC-KFZ-06 | Ruheversicherung aktivieren | 🔴 Muss | 🔲 Offen |
| UC-KFZ-07 | eVB-Nummer stornieren | 🟡 Soll | 🔲 Offen |
| UC-KFZ-08 | Sonderkündigung bei Beitragserhöhung | 🟡 Soll | 🔲 Offen |
| UC-KFZ-09 | Fahrerkreis ändern | 🟡 Soll | 🔲 Offen |

### LVM-Zusatzbausteine verwalten

| UC-Nr. | Titel | Priorität | Status |
|--------|-------|-----------|--------|
| UC-KFZ-10 | LVM-RabattSchutz hinzufügen/entfernen (ab SF4, 1×/Jahr HP+VK) | 🟡 Soll | 🔲 Offen |
| UC-KFZ-11 | LVM-FahrerKasko hinzufügen/entfernen (Personenschäden Fahrer bis 15 Mio.) | 🟡 Soll | 🔲 Offen |
| UC-KFZ-12 | LVM-AuslandPlus hinzufügen/entfernen (Regulierung nach dt. Bedingungen) | 🟡 Soll | 🔲 Offen |
| UC-KFZ-13 | LVM-Schutzbrief hinzufügen/entfernen (20+ Leistungen, ab 8,21 €/Jahr) | 🟡 Soll | 🔲 Offen |

### Kurzfristige KFZ-Versicherungen

| UC-Nr. | Titel | Priorität | Status |
|--------|-------|-----------|--------|
| UC-KFZ-14 | Kurzfristige Versicherung: Zusatzfahrer abschließen (ab 2,99 €/Tag) | 🟡 Soll | 🔲 Offen |
| UC-KFZ-15 | Kurzfristige Versicherung: Mietwagen abschließen (SB bis 2.000 €) | 🟡 Soll | 🔲 Offen |
| UC-KFZ-16 | Kurzfristige Versicherung: Probefahrt abschließen (SB bis 1.000 €) | 🟢 Kann | 🔲 Offen |
| UC-KFZ-17 | Kurzfristige Versicherung: Carsharing abschließen (SB bis 1.500 €) | 🟢 Kann | 🔲 Offen |

### LVM-SchadenService

| UC-Nr. | Titel | Priorität | Status |
|--------|-------|-----------|--------|
| UC-KFZ-18 | SchadenService beauftragen (Partnerwerkstatt, Abholung, Ersatzwagen) | 🟢 Kann | 🔲 Offen |

> **Priorität:** 🔴 Muss | 🟡 Soll | 🟢 Kann  
> **Status:** 🔲 Offen | ✏️ In Arbeit | ✅ Fertig | 🔍 In Review
