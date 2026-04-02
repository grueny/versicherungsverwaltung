# Funktionale Anforderungen – Spartenübergreifend

> Hier werden **spartenübergreifende** Use Cases erfasst, die für alle Sparten gleich ablaufen.  
> **Spartenspezifische** Use Cases werden unter `12_sparten/<spartenname>/use_cases/` erfasst.  
> Nutze `_template.md` als Vorlage für neue Use Cases.

## Use-Case-Verzeichnis (spartenübergreifend)

| UC-Nr. | Titel | Priorität | Status |
|--------|-------|-----------|--------|
| [UC-01](UC-01_angebot_erstellen_und_beantragen.md) | Angebot erstellen und beantragen | 🔴 Muss | ✅ Fertig |
| [UC-02](UC-02_antrag_bearbeiten_und_freigeben.md) | Antrag bearbeiten und freigeben | 🔴 Muss | ✅ Fertig |
| [UC-03](UC-03_produktkonfiguration_und_kalkulation.md) | Produktkonfiguration und Kalkulation einstellen | 🔴 Muss | ✅ Fertig |
| [UC-04](UC-04_durchlaufzeiten_erfassen_und_auswerten.md) | Durchlaufzeiten erfassen und auswerten | 🟡 Soll | ✅ Fertig |
| [UC-05](UC-05_vertragsaenderung_durchfuehren.md) | Vertragsänderung durchführen (Nachtrag) | 🔴 Muss | ✅ Fertig |
| [UC-06](UC-06_vertragsstornierung_durchfuehren.md) | Vertragsstornierung durchführen | 🔴 Muss | ✅ Fertig |
| [UC-07](UC-07_wiederinkraftsetzung_durchfuehren.md) | Wiederinkraftsetzung durchführen | 🔴 Muss | ✅ Fertig |
| [UC-08](UC-08_beitragsrechnung_inkasso.md) | Beitragsrechnung durchführen (Inkasso) | 🔴 Muss | ✅ Fertig |
| [UC-09](UC-09_rueckwirkende_aenderung_und_stornierung.md) | Rückwirkende Änderung und Stornierung durchführen | 🔴 Muss | ✅ Fertig |
| [UC-10](UC-10_nachlaesse_und_zuschlaege_verwalten.md) | Nachlässe und Zuschläge verwalten | 🔴 Muss | ✅ Fertig |
| [UC-11](UC-11_versicherungsgewerbetarif_verwalten.md) | Versicherungsgewerbetarif als Nachlass verwalten | 🔴 Muss | ✅ Fertig |

> **Priorität:** 🔴 Muss | 🟡 Soll | 🟢 Kann  
> **Status:** 🔲 Offen | ✏️ In Arbeit | ✅ Fertig | 🔍 In Review

## Themenblöcke (Epics) – Spartenübergreifend

### Vertragsverwaltung (Kernprozess)
- [UC-05: Vertragsänderung durchführen (Nachtrag)](UC-05_vertragsaenderung_durchfuehren.md) ✅
- [UC-06: Vertragsstornierung durchführen](UC-06_vertragsstornierung_durchfuehren.md) ✅
- [UC-07: Wiederinkraftsetzung durchführen](UC-07_wiederinkraftsetzung_durchfuehren.md) ✅
- [UC-08: Beitragsrechnung durchführen (Inkasso)](UC-08_beitragsrechnung_inkasso.md) ✅
- [UC-09: Rückwirkende Änderung und Stornierung durchführen](UC-09_rueckwirkende_aenderung_und_stornierung.md) ✅
- [UC-10: Nachlässe und Zuschläge verwalten](UC-10_nachlaesse_und_zuschlaege_verwalten.md) ✅
- [UC-11: Versicherungsgewerbetarif als Nachlass verwalten](UC-11_versicherungsgewerbetarif_verwalten.md) ✅
- UC-XX: Vertrag anlegen (generischer Ablauf)
- UC-XX: Vertragsübersicht anzeigen
- UC-XX: Vertragshistorie einsehen

### Kundenverwaltung
- UC-XX: Kunde anlegen
- UC-XX: Kundendaten ändern
- UC-XX: Kundenübersicht anzeigen

### Vertrieb / Anbahnung
- [UC-01: Angebot erstellen und beantragen](UC-01_angebot_erstellen_und_beantragen.md) ✅
- [UC-02: Antrag bearbeiten und freigeben](UC-02_antrag_bearbeiten_und_freigeben.md) ✅

### Systemadministration / Konfiguration
- [UC-03: Produktkonfiguration und Kalkulation einstellen](UC-03_produktkonfiguration_und_kalkulation.md) ✅

### Berichtswesen
- [UC-04: Durchlaufzeiten erfassen und auswerten](UC-04_durchlaufzeiten_erfassen_und_auswerten.md) ✅
- UC-XX: Berichte generieren
- UC-XX: Statistiken anzeigen

## Hinweis zur Spartentrennung

> Spartenspezifische Abweichungen vom Kernprozess werden **nicht** hier, sondern in der  
> jeweiligen Spartenkonfiguration unter `12_sparten/<spartenname>/` dokumentiert:  
> - Prozessabweichungen → `prozesse.md`  
> - Zusätzliche Use Cases → `use_cases/`  
> - Validierungen → `plausibilitaeten.md`  
> - Geschäftsregeln → `geschaeftsregeln.md`
