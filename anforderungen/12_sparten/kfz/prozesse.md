# Prozesse – Sparte: KFZ

> Spartenspezifische Prozessabweichungen und -ergänzungen zum spartenübergreifenden Kernprozess.
> Nur Abweichungen vom Kernprozess dokumentieren – was gleich bleibt, wird hier nicht wiederholt.

## Prozessübersicht

| Kernprozess | Abweichung in KFZ | Sub-Modul | Beschreibung |
|-------------|-------------------|-----------|-------------|
| Anbahnung / Angebot | Fahrzeugdaten erfassen | kern | HSN/TSN/FIN, Typklasse, Regionalklasse ermitteln |
| Antragsprüfung | eVB-Nummer erzeugen | evb | Elektronische Versicherungsbestätigung für Zulassung |
| Policierung | eVB-Statusmeldung | evb | Rückmeldung an Zulassungsstelle |
| Policierung | VKZ-Kennzeichen zuweisen | vkz | Kennzeichen aus Bestand zuweisen (nur FA-04) |
| Vertragswartung / Nachtrag | Fahrzeugwechsel | kern | Sonderfall: Abmeldung Alt-Fzg, Anmeldung Neu-Fzg |
| Beitragsanpassung | SF-Klassen-Rückstufung | sfr | Rückstufung nach Schadenfall |
| Kündigung / Storno | Abmeldung Fahrzeug | kern | Ruheversicherung oder Kündigung bei Abmeldung |
| Kündigung / Storno | VKZ-Rücknahme | vkz | Kennzeichen zurücknehmen (nur FA-04) |
| Verlängerung | Stichtagskündigung 30.11. | kern | Sonderkündigungsrecht beachten |
| Verlängerung | VKZ-Saisonwechsel | vkz | Neues Kennzeichen zum 01.03. zuweisen |
| – (Eigenständig) | eVB ohne Antrag erstellen | evb | Schnell-eVB ohne Angebot/Antrag |
| – (Eigenständig) | Antragsanmahnung verarbeiten | evb | Zulassungsrückmeldung → Antragserinnerung |
| – (Eigenständig) | VWB Zugang | sfr | SF-Anfrage an Vorversicherer bei Neukunde |
| – (Eigenständig) | VWB Abgang | sfr | SF-Auskunft an Nachversicherer bei Abgang |
| – (Eigenständig) | VKZ-Bestandsverwaltung | vkz | Kontingente bestellen, lagern, inventarisieren |

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

### eVB ohne Antrag erstellen (Sub-Modul: evb)

- **Beschreibung:** Eine eVB-Nummer kann auch ohne vorheriges Angebot oder einen Antrag ausgestellt werden – z. B. wenn ein Kunde kurzfristig eine Versicherungsbestätigung für die Zulassungsstelle benötigt.
- **Auslöser:** Benutzer wählt „eVB ohne Antrag erstellen“; telefonische Kundenanforderung
- **Ablauf:**
  1. Minimalerfassung: Partner + Fahrzeugart + Deckungsumfang (HP/HP+TK/HP+VK)
  2. Optional: HSN/TSN, FIN, Wunschkennzeichen
  3. Plausibilitätsprüfung (Partner aktiv, keine doppelte offene eVB)
  4. eVB-Nummer erzeugen (Status: `ERZEUGT`)
  5. eVB an GDV melden (Status: `GEMELDET`)
  6. eVB dem Kunden bereitstellen (Anzeige, Druck via S5)
- **Besonderheit:** Die eVB hat keinen `antrag_id` / `vertrag_id` – sie ist „frei schwebend“ bis ein Antrag zugeordnet wird
- **Ergebnis:** eVB-Nummer erzeugt und gemeldet, Antrag steht noch aus
- **Detaillierter Use Case:** → UC-KFZ-01

### Antragsanmahnung bei Zulassungsrückmeldung (Sub-Modul: evb)

- **Beschreibung:** Wenn die Zulassungsstelle eine eVB-Nummer verwendet und das GDV-System eine Rückmeldung sendet, prüft das System ob ein Antrag/Vertrag existiert. Falls nicht, wird automatisch eine Antragsanmahnung erzeugt.
- **Auslöser:** GDV-Zulassungsrückmeldung (Import IMP-08) für eine eVB ohne zugeordneten Antrag/Vertrag
- **Ablauf:**
  1. GDV-Rückmeldung empfangen (eVB-Nummer, Zulassungsdaten, Kennzeichen, FIN)
  2. eVB identifizieren und prüfen:
     - eVB mit Antrag/Vertrag → nur Status-Update auf `VERWENDET`, keine Anmahnung
     - eVB ohne Antrag/Vertrag → weiter mit Schritt 3
  3. **Antragsanmahnung** erzeugen (Status: `OFFEN`)
  4. Daten aus eVB (Partner, Fahrzeugart, Deckungsumfang) und GDV-Rückmeldung (Kennzeichen, FIN, Halter, Erstzulassung) zusammenführen
  5. Frist setzen: 14 Tage (Anmahnungsfrist), 28 Tage (Eskalationsfrist)
  6. Benachrichtigung (NOT-10) an Sachbearbeiter / Vertriebsteam
- **Fristablauf:**
  - 14 Tage → Status `UEBERFAELLIG`, Erinnerung
  - 28 Tage → Status `ESKALIERT`, Eskalation an Teamleiter (NOT-11)
- **Ergebnis:** Antragsanmahnung mit vollständigen Daten in der Aufgabenübersicht
- **Detaillierter Use Case:** → UC-KFZ-01

### Angebot aus Antragsanmahnung erstellen (Sub-Modul: evb)

- **Beschreibung:** Der Sachbearbeiter erstellt aus einer Antragsanmahnung heraus ein Angebot. Alle verfügbaren Daten (Partner, Fahrzeug, eVB, Zulassungsdaten) werden automatisch übernommen.
- **Auslöser:** Sachbearbeiter wählt „Angebot aus Anmahnung erstellen“ in der Aufgabenübersicht
- **Ablauf:**
  1. Daten aus Antragsanmahnung laden
  2. Neues Angebot erzeugen (→ UC-01) mit vorausgefüllten Daten:
     - Partner, Sparte KFZ, Fahrzeugart
     - Fahrzeugdaten (HSN/TSN, FIN, Kennzeichen) aus Zulassungsrückmeldung
     - Deckungsumfang aus eVB
  3. Benutzer ergänzt fehlende Daten (SF-Klasse, Fahrerdaten, Nutzungsart etc.)
  4. Normaler Angebotsprozess (UC-01, Phase 2–5)
  5. Bei Beantragung: bestehende eVB wird dem Antrag zugeordnet (**keine neue eVB**)
  6. Antragsanmahnung-Status → `ERLEDIGT`
- **Ergebnis:** Antrag mit übernommenen Daten und verknüpfter eVB
- **Detaillierter Use Case:** → UC-KFZ-01

---

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

---

## SFR / VWB-Verfahren (Sub-Modul: sfr)

> Das Versicherer-Wechsel-Branchenverfahren (VWB) ist ein standardisiertes GDV-Verfahren zum Austausch von Schadenfreiheitsrabatt-Daten zwischen Versicherungsgesellschaften bei Versichererwechsel.

### VWB Zugang (Neukunde wechselt zu uns)

- **Beschreibung:** Ein neuer Versicherungsnehmer wechselt von einem anderen Versicherer zur LVM. Die SF-Klasse muss beim Vorversicherer abgefragt und übernommen werden.
- **Auslöser:** Antrag enthält Vorversicherer-Angaben (Vorversicherer, bisherige Vertragsnummer, bisherige SF-Klasse)
- **Ablauf:**
  1. VWB-Anfrage an Vorversicherer senden (GDV REST API → Nachrichtentyp: SF-ANFRAGE)
  2. Antwort des Vorversicherers empfangen (Frist: 4 Wochen)
  3. SF-Auskunft prüfen:
     - SF-Klasse stimmt überein → automatisch übernehmen
     - SF-Klasse weicht ab → Schwebe erzeugen, Sachbearbeiter prüft
     - Vorversicherer antwortet nicht → nach Fristablauf mit angegebener SF-Klasse policieren, Wiedervorlage setzen
  4. SF-Klasse in `SfKlassenHistorie` eintragen (Änderungsgrund: `UEBERNAHME`)
- **Ergebnis:** SF-Klasse bestätigt und in Tarifierung übernommen

#### VWB-Anfrage (GDV REST API)

```json
{
  "nachrichtentyp": "SF_ANFRAGE",
  "anfragender_versicherer": "LVM",
  "vorversicherer_id": "AXA",
  "vorvertrag_nummer": "AXA-KFZ-2024-12345",
  "versicherungsnehmer": {
    "name": "Mustermann",
    "vorname": "Max",
    "geburtsdatum": "1985-03-15"
  },
  "angefragte_sf_klasse_hp": "SF5",
  "angefragte_sf_klasse_vk": "SF3",
  "stichtag": "2026-12-31"
}
```

#### VWB-Auskunft (Antwort vom Vorversicherer)

```json
{
  "nachrichtentyp": "SF_AUSKUNFT",
  "referenz_id": "VWB-2026-000001",
  "bestaetigte_sf_klasse_hp": "SF5",
  "bestaetigte_sf_klasse_vk": "SF3",
  "schaeden_letzte_5_jahre": 0,
  "vertrag_beginn": "2019-01-01",
  "vertrag_ende": "2026-12-31",
  "kuendigungsgrund": "VN_KUENDIGUNG"
}
```

### VWB Abgang (Bestandskunde wechselt weg)

- **Beschreibung:** Ein bestehender Versicherungsnehmer kündigt und wechselt zu einem anderen Versicherer. Der Nachversicherer fragt die SF-Daten bei uns ab.
- **Auslöser:** SF-Anfrage eines Nachversicherers empfangen (GDV REST API)
- **Ablauf:**
  1. SF-Anfrage empfangen und Vertrag identifizieren (über VN-Daten + Vertragsnummer)
  2. SF-Daten aus `SfKlassenHistorie` ermitteln
  3. Schadenhistorie der letzten 5 Jahre zusammenstellen
  4. SF-Auskunft an Nachversicherer senden
- **Bedingungen:**
  - Vertrag muss existiert haben (aktiv oder gekündigt)
  - Antwortfrist: 4 Wochen nach Eingang der Anfrage
  - Automatische Beantwortung bei eindeutiger Zuordnung, sonst Schwebe
- **Ergebnis:** SF-Auskunft an Nachversicherer übermittelt

### VWB-Statusmodell

| Status | Beschreibung | Folgestatus |
|--------|-------------|-------------|
| `ANFRAGE_GESENDET` | SF-Anfrage an Vorversicherer gesendet | AUSKUNFT_EMPFANGEN, FRIST_ABGELAUFEN |
| `ANFRAGE_EMPFANGEN` | SF-Anfrage von Nachversicherer empfangen | AUSKUNFT_GESENDET |
| `AUSKUNFT_EMPFANGEN` | Antwort des Vorversicherers empfangen | UEBERNOMMEN, ABWEICHUNG_PRUEFEN |
| `AUSKUNFT_GESENDET` | Antwort an Nachversicherer gesendet | ABGESCHLOSSEN |
| `ABWEICHUNG_PRUEFEN` | SF-Klasse weicht ab → manuelle Prüfung | UEBERNOMMEN, KORRIGIERT |
| `FRIST_ABGELAUFEN` | Vorversicherer hat nicht innerhalb 4 Wochen geantwortet | UEBERNOMMEN |
| `UEBERNOMMEN` | SF-Klasse übernommen und in Tarifierung eingepflegt | ABGESCHLOSSEN |
| `KORRIGIERT` | SF-Klasse nach Prüfung korrigiert | ABGESCHLOSSEN |
| `ABGESCHLOSSEN` | VWB-Vorgang abgeschlossen | – (Endzustand) |

---

## Versicherungskennzeichen-Verwaltung (Sub-Modul: vkz)

> Für Fahrzeuge mit Versicherungskennzeichen (Mopeds, Mofas, E-Scooter – Fahrzeugart FA-04) gelten besondere Regeln: Kennzeichen werden vom Versicherer ausgegeben, haben einen festen Gültigkeitszeitraum (01.03. bis Ende Februar) und müssen physisch verwaltet werden.

### VKZ-Bestandsverwaltung (Kontingent)

- **Beschreibung:** Der Versicherer bestellt Versicherungskennzeichen in Kontingenten beim Hersteller und verwaltet den physischen Bestand (Lager, Ausgabe, Rücknahme, Vernichtung).
- **Ablauf:**
  1. Kontingent-Bestellung: Nummernkreis und Menge festlegen
  2. Wareneingang: Kennzeichen als `AUF_LAGER` erfassen
  3. Inventur: Periodischer Bestandsabgleich
- **Ergebnis:** Verfügbare Kennzeichen im System abrufbar

### VKZ-Zuweisung an Vertrag

- **Beschreibung:** Bei Abschluss eines Versicherungskennzeichen-Vertrags wird ein verfügbares Kennzeichen aus dem Bestand dem Vertrag zugewiesen.
- **Auslöser:** Policierung eines VKZ-Vertrags (Hook `nach_Policierung`)
- **Ablauf:**
  1. Verfügbares Kennzeichen aus Bestand auswählen (Status `AUF_LAGER`)
  2. Kennzeichen dem Vertrag zuweisen (Status → `ZUGEWIESEN`)
  3. Gültigkeitszeitraum setzen: 01.03. des aktuellen Jahres bis 28./29.02. des Folgejahres
  4. Kennzeichen dem Kunden aushändigen oder zusenden (→ S5 Druck/Versand)
- **Bedingungen:**
  - Nur für Fahrzeugart `VERSICHERUNGSKENNZEICHEN` (FA-04)
  - Ein Kennzeichen pro Vertrag im Gültigkeitszeitraum
  - Farbe des Kennzeichens wechselt jährlich (Schwarz/Blau/Grün im 3-Jahres-Rhythmus)
- **Ergebnis:** Kennzeichen zugewiesen und Vertrag aktiv

### VKZ-Saisonwechsel (Jährliche Erneuerung)

- **Beschreibung:** Zum 01.03. jeden Jahres müssen alle laufenden VKZ-Verträge ein neues Kennzeichen erhalten. Das alte Kennzeichen wird ungültig.
- **Auslöser:** Scheduler-Job vor dem 01.03. (Vorlauf: 4 Wochen)
- **Ablauf:**
  1. Alle aktiven VKZ-Verträge mit Ablaufdatum Ende Februar ermitteln
  2. Für jeden Vertrag: Neues Kennzeichen aus Bestand zuweisen
  3. Altes Kennzeichen → Status `ABGELAUFEN`
  4. Neues Kennzeichen an Kunden versenden (→ S5 Druck)
  5. Beitrag für neue Saison berechnen und an Inkasso (S2) melden
- **Bedingungen:**
  - Vertrag muss aktiv sein und nicht gekündigt zum 01.03.
  - Ausreichend Kennzeichen im Bestand (Kontingent-Prüfung 8 Wochen vorher)
- **Ergebnis:** Neues Kennzeichen zugewiesen, altes ungültig, Beitrag gebucht

### VKZ-Rücknahme

- **Beschreibung:** Bei Kündigung oder Ablauf eines VKZ-Vertrags wird das Kennzeichen zurückgenommen.
- **Auslöser:** Vertragskündigung oder -ablauf
- **Ablauf:**
  1. Kennzeichen-Status → `ZURUECKGENOMMEN`
  2. Physische Rückgabe prüfen (Wiedervorlage falls nicht zurückgegeben)
  3. Kennzeichen nach Prüfung → `VERNICHTET` (darf nicht wiederverwendet werden)
- **Ergebnis:** Kennzeichen aus dem Umlauf, Vertrag beendet

### VKZ-Kennzeichen-Statusmodell

| Status | Beschreibung | Folgestatus |
|--------|-------------|-------------|
| `BESTELLT` | Kennzeichen beim Hersteller bestellt | AUF_LAGER |
| `AUF_LAGER` | Im Bestand, verfügbar zur Zuweisung | ZUGEWIESEN |
| `ZUGEWIESEN` | Einem Vertrag zugeordnet, in Nutzung | ABGELAUFEN, ZURUECKGENOMMEN |
| `ABGELAUFEN` | Gültigkeitszeitraum abgelaufen (Ende Feb.) | ZURUECKGENOMMEN, VERNICHTET |
| `ZURUECKGENOMMEN` | Physisch zurückgegeben | VERNICHTET |
| `VERNICHTET` | Endgültig aus dem Umlauf | – (Endzustand) |
| `VERLOREN` | Kennzeichen nicht zurückgegeben (Wiedervorlage) | VERNICHTET |
