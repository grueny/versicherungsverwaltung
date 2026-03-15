# Prozesse – Sparte: KFZ

> Spartenspezifische Prozessabweichungen und -ergänzungen zum spartenübergreifenden Kernprozess.
> Nur Abweichungen vom Kernprozess dokumentieren – was gleich bleibt, wird hier nicht wiederholt.

## Prozessübersicht

| Kernprozess | Abweichung in KFZ | Beschreibung |
|-------------|-------------------|-------------|
| Anbahnung / Angebot | Fahrzeugdaten erfassen | HSN/TSN/FIN, Typklasse, Regionalklasse ermitteln |
| Antragsprüfung | eVB-Nummer erzeugen | Elektronische Versicherungsbestätigung für Zulassung |
| Policierung | eVB-Statusmeldung | Rückmeldung an Zulassungsstelle |
| Vertragswartung / Nachtrag | Fahrzeugwechsel | Sonderfall: Abmeldung Alt-Fzg, Anmeldung Neu-Fzg |
| Beitragsanpassung | SF-Klassen-Rückstufung | Rückstufung nach Schadenfall |
| Kündigung / Storno | Abmeldung Fahrzeug | Ruheversicherung oder Kündigung bei Abmeldung |
| Verlängerung | Stichtagskündigung 30.11. | Sonderkündigungsrecht beachten |

## Detaillierte Prozessabweichungen

### Anbahnung / Angebot: Fahrzeugdaten erfassen

#### Abweichung vom Kernprozess
Bei der Angebotserstellung müssen zusätzlich zum generischen Prozess die Fahrzeugdaten erfasst und gegen die GDV-Datenbank aufgelöst werden (Typklasse, Regionalklasse).

#### Zusätzliche Prozessschritte
1. Erfassung Fahrzeugdaten: HSN (Herstellerschlüsselnummer) und TSN (Typschlüsselnummer)
2. Abfrage GDV-Typklassenverzeichnis → Ermittlung Typklasse HP/TK/VK
3. Ermittlung Regionalklasse anhand des Zulassungsbezirks (PLZ)
4. Erfassung Fahrerdaten: Fahrerkreis, Alter des jüngsten Fahrers, SF-Klasse
5. Erfassung Nutzungsdaten: Jährliche Fahrleistung, Nutzungsart, Stellplatz
6. Prämienberechnung auf Basis aller Tarifmerkmale

#### Bedingungen
- HSN/TSN muss in der GDV-Datenbank vorhanden sein
- SF-Klasse muss durch Vorversicherer-Nachweis belegt werden können

---

### Antragsprüfung: eVB-Nummer erzeugen

#### Abweichung vom Kernprozess
Nach positiver Antragsprüfung muss eine elektronische Versicherungsbestätigung (eVB-Nummer) erzeugt und an den Kunden übermittelt werden. Diese wird für die Zulassung des Fahrzeugs benötigt.

#### Zusätzliche Prozessschritte
1. eVB-Nummer generieren (7-stelliger alphanumerischer Code)
2. eVB-Nummer an zentrales GDV-System melden
3. eVB-Nummer dem Kunden übermitteln (für Zulassungsstelle)

#### Bedingungen
- Nur bei Neuverträgen und Fahrzeugwechseln
- Nur für zulassungspflichtige Fahrzeuge

---

### Vertragswartung: Fahrzeugwechsel

#### Abweichung vom Kernprozess
Ein Fahrzeugwechsel ist ein Sonderfall der Vertragswartung, bei dem das versicherte Fahrzeug ausgetauscht wird. Dies erfordert eine Neuberechnung der Prämie und ggf. eine neue eVB-Nummer.

#### Zusätzliche Prozessschritte
1. Abmeldung des alten Fahrzeugs (FIN, Kennzeichen)
2. Erfassung des neuen Fahrzeugs (HSN/TSN/FIN)
3. Neuermittlung Typklasse und Regionalklasse
4. Prämienneuberechnung
5. Erzeugung neuer eVB-Nummer
6. Nachtrag erstellen mit Dokumentation beider Fahrzeuge

#### Entfallende Prozessschritte
- Keine (es handelt sich um einen erweiterten Nachtragsprozess)

#### Bedingungen
- SF-Klasse bleibt erhalten (wird übertragen)
- Kasko-Selbstbeteiligung wird beibehalten, sofern nicht explizit geändert

---

### Beitragsanpassung: SF-Klassen-Rückstufung

#### Abweichung vom Kernprozess
Bei Meldung eines regulierten Schadens durch die Schadenverwaltung wird die SF-Klasse zum nächsten Fälligkeitstermin zurückgestuft, was eine automatische Beitragsanpassung auslöst.

#### Zusätzliche Prozessschritte
1. Schadensmeldung über Schnittstelle empfangen
2. SF-Rückstufungstabelle anwenden (abhängig von aktueller SF-Klasse und Anzahl Schäden)
3. Neue SF-Klasse zum nächsten Fälligkeitstermin setzen
4. Beitrag neu berechnen
5. Information an Versicherungsnehmer (über Druckschnittstelle)
6. Neue Beitragsdaten an Inkasso übermitteln

#### Bedingungen
- Gilt nur für HP- und VK-Schäden (Teilkasko hat keine SF-Klasse)
- Rabattschutz kann die Rückstufung verhindern (falls vereinbart)

---

### Kündigung / Storno: Abmeldung Fahrzeug

#### Abweichung vom Kernprozess
Bei Abmeldung des Fahrzeugs bei der Zulassungsstelle tritt eine Ruheversicherung in Kraft oder der Vertrag wird beendet.

#### Zusätzliche Prozessschritte
1. Abmeldemeldung über GDV-Schnittstelle empfangen
2. Vertrag in Ruheversicherung überführen (beitragsfrei, eingeschränkter Schutz)
3. Nach 18 Monaten Ruheversicherung: Automatische Vertragsbeendigung
4. Bei Wiederzulassung innerhalb der Frist: Vertrag reaktivieren

#### Bedingungen
- Ruheversicherung ist gesetzlich vorgeschrieben
- SF-Klasse bleibt während der Ruheversicherung erhalten

## Spartenspezifische Prozesse (ohne Kernprozess-Entsprechung)

### eVB-Stornierung
- **Beschreibung:** Stornierung einer eVB-Nummer, wenn das Fahrzeug nicht zugelassen wird
- **Auslöser:** Zeitablauf (6 Monate ohne Zulassung) oder manuelle Stornierung
- **Ablauf:**
  1. Prüfung ob eVB-Nummer verwendet wurde (Rückmeldung Zulassungsstelle)
  2. Falls nicht verwendet: eVB-Nummer stornieren
  3. Antrag/Vertrag stornieren
- **Ergebnis:** eVB-Nummer ungültig, Vertrag nicht zustande gekommen

### Sonderkündigung bei Beitragserhöhung
- **Beschreibung:** Versicherungsnehmer hat bei Beitragserhöhung (z. B. durch Typklassenänderung) ein Sonderkündigungsrecht
- **Auslöser:** Mitteilung über Beitragserhöhung zum neuen Versicherungsjahr
- **Ablauf:**
  1. Beitragserhöhung feststellen und mitteilen
  2. Sonderkündigungsfrist läuft (1 Monat ab Mitteilung)
  3. Bei Kündigung: Vertrag zum Erhöhungszeitpunkt beenden
- **Ergebnis:** Vertrag beendet oder Erhöhung akzeptiert
