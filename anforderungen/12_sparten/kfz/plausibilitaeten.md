# Plausibilitäten – Sparte: KFZ (basierend auf LVM Versicherung)

> Spartenspezifische Validierungen und Prüfregeln für die KFZ-Versicherung.
> Quelle: Öffentlich zugängliche Produktinformationen der LVM Versicherung (lvm.de, Stand März 2026).
> Spartenübergreifende Plausibilitäten sind in `08_geschaeftsregeln.md` erfasst.

## Plausibilitäten nach Prozessschritt

### Angebotserstellung / Antrag
| PL-Nr. | Feld / Kontext | Prüfregel | Fehlermeldung | Schwere |
|--------|---------------|-----------|---------------|---------|
| PL-KFZ-A01 | Geburtsdatum VN | Mindestalter 18 Jahre (bzw. 17 bei BF17) | "Versicherungsnehmer muss mindestens 18 Jahre alt sein" | 🔴 Fehler |
| PL-KFZ-A02 | HSN/TSN | Muss in GDV-Typklassenverzeichnis vorhanden sein | "Fahrzeugtyp nicht im Typklassenverzeichnis gefunden" | 🔴 Fehler |
| PL-KFZ-A03 | Erstzulassung Fahrzeug | Darf nicht in der Zukunft liegen | "Erstzulassungsdatum darf nicht in der Zukunft liegen" | 🔴 Fehler |
| PL-KFZ-A04 | Fahrzeugalter bei Vollkasko | Warnung bei Fahrzeugalter > 10 Jahre | "Vollkaskoversicherung bei Fahrzeugalter über 10 Jahren prüfen" | 🟡 Warnung |
| PL-KFZ-A05 | Jährliche Fahrleistung | Wert zwischen 5.000 und 100.000 km | "Bitte jährliche Fahrleistung prüfen – Wert außerhalb des üblichen Bereichs" | 🟡 Warnung |
| PL-KFZ-A06 | SF-Klasse | SF-Klasse muss plausibel zum Führerscheinalter sein | "SF-Klasse passt nicht zum Alter des Versicherungsnehmers" | 🟡 Warnung |
| PL-KFZ-A07 | Fahrerkreis / Alter jüngster Fahrer | Jüngster Fahrer muss mindestens 17 Jahre alt sein | "Jüngster Fahrer muss mindestens 17 Jahre alt sein (BF17)" | 🔴 Fehler |
| PL-KFZ-A08 | FIN (Fahrgestellnummer) | Muss 17-stellig sein und Prüfziffer bestehen | "Ungültige Fahrgestellnummer (FIN)" | 🔴 Fehler |
| PL-KFZ-A09 | Amtliches Kennzeichen | Format muss deutschem Kennzeichenformat entsprechen | "Ungültiges Kennzeichenformat" | 🔴 Fehler |
| PL-KFZ-A10 | Produktkombination | VK nur mit HP, TK nur mit HP, VK enthält TK | "Kaskoversicherung erfordert eine KFZ-Haftpflichtversicherung" | 🔴 Fehler |

### Policierung
| PL-Nr. | Feld / Kontext | Prüfregel | Fehlermeldung | Schwere |
|--------|---------------|-----------|---------------|---------|
| PL-KFZ-P01 | eVB-Nummer | Muss erzeugt sein vor Policierung | "eVB-Nummer wurde noch nicht erzeugt" | 🔴 Fehler |
| PL-KFZ-P02 | Versicherungsbeginn | Bei Neuzulassung: Beginn = Zulassungsdatum | "Versicherungsbeginn muss dem Zulassungsdatum entsprechen" | ℹ️ Hinweis |

### Vertragswartung / Nachtrag
| PL-Nr. | Feld / Kontext | Prüfregel | Fehlermeldung | Schwere |
|--------|---------------|-----------|---------------|---------|
| PL-KFZ-N01 | Fahrzeugwechsel – neues Fahrzeug | HSN/TSN des neuen Fahrzeugs muss gültig sein | "Neues Fahrzeug nicht im Typklassenverzeichnis gefunden" | 🔴 Fehler |
| PL-KFZ-N02 | Fahrerkreisänderung | Neuer jüngster Fahrer mindestens 17 Jahre | "Jüngster Fahrer muss mindestens 17 Jahre alt sein" | 🔴 Fehler |
| PL-KFZ-N03 | SF-Klassen-Übernahme | SF-Nachweis muss vorliegen | "SF-Nachweis des Vorversicherers erforderlich" | 🟡 Warnung |
| PL-KFZ-N04 | Änderung Jährliche Fahrleistung | Abweichung > 50% zur bisherigen Angabe | "Starke Abweichung der Fahrleistung – bitte bestätigen" | 🟡 Warnung |

### Kündigung / Storno
| PL-Nr. | Feld / Kontext | Prüfregel | Fehlermeldung | Schwere |
|--------|---------------|-----------|---------------|---------|
| PL-KFZ-K01 | Kündigungsdatum | Bei ordentlicher Kündigung: Mindestens 1 Monat vor Ablauf | "Kündigungsfrist nicht eingehalten" | 🔴 Fehler |
| PL-KFZ-K02 | Pflichtversicherung | Bei Kündigung HP: Nachweis Folgeversicherung oder Abmeldung | "KFZ-Haftpflicht ist Pflichtversicherung – Abmeldung oder Folgeversicherung erforderlich" | 🔴 Fehler |
| PL-KFZ-K03 | Sonderkündigung | Sonderkündigungsrecht nur bei qualifizierendem Ereignis (Beitragserhöhung, Schadenfall) | "Kein Sonderkündigungsgrund erkennbar" | 🔴 Fehler |

## Abhängigkeiten zwischen Plausibilitäten
- PL-KFZ-A06 (SF-Klasse plausibel) setzt voraus, dass Geburtsdatum (PL-KFZ-A01) erfasst ist
- PL-KFZ-A10 (Produktkombination) wird bei jeder Produktauswahl erneut geprüft
- PL-KFZ-K02 (Pflichtversicherung) kann durch Abmeldemeldung der Zulassungsstelle automatisch aufgelöst werden

## LVM-spezifische Plausibilitäten

| PL-Nr. | Feld / Kontext | Prüfregel | Fehlermeldung | Schwere |
|--------|---------------|-----------|---------------|--------|
| PL-KFZ-L01 | RabattSchutz | LVM-RabattSchutz nur buchbar ab SF-Klasse 4 oder besser | "RabattSchutz ist erst ab SF-Klasse 4 verfügbar" | 🔴 Fehler |
| PL-KFZ-L02 | Gelegentliche Fahrer | Gelegentliche Fahrer außerhalb des Fahrerkreises müssen mindestens 23 Jahre alt sein | "Gelegentliche Fahrer müssen mindestens 23 Jahre alt sein" | 🔴 Fehler |
| PL-KFZ-L03 | Kurzfristige Versicherung: Zusatzfahrer | Setzt bestehende LVM-KFZ-Versicherung voraus | "Zusatzfahrerschutz erfordert bestehende LVM-KFZ-Versicherung" | 🔴 Fehler |
| PL-KFZ-L04 | Kurzfristige Versicherung: Mietwagen | SB-Betrag darf 2.000 EUR nicht überschreiten | "Maximale absicherbare Selbstbeteiligung: 2.000 EUR" | 🔴 Fehler |
| PL-KFZ-L05 | Kurzfristige Versicherung: Probefahrt | Nur PKW mit Zulassung/Standort in Deutschland | "Nur PKW mit deutscher Zulassung versicherbar" | 🔴 Fehler |
| PL-KFZ-L06 | Kurzfristige Versicherung: Carsharing | Keine Wohnmobile, Trikes, Quads etc. | "Fahrzeugtyp nicht für Carsharing-Schutz zugelassen" | 🔴 Fehler |
| PL-KFZ-L07 | Schutzbrief: Entfernung | Bestimmte Leistungen erst ab 50 km Luftlinie vom Wohnort | "Übernachtung/Weiterfahrt: Schadensort muss mind. 50 km vom Wohnort entfernt sein" | ℹ️ Hinweis |
