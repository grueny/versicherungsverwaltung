# UC-KFZ-01: eVB ohne Antrag erstellen und Antragsanmahnung

## Beschreibung

Im Standardprozess wird eine eVB-Nummer im Rahmen eines Angebots/Antrags erzeugt (UC-01 → Hook `nach_Antragserstellung`). Es gibt jedoch den fachlichen Bedarf, eine eVB-Nummer **ohne vorheriges Angebot oder einen Antrag** auszustellen – z. B. wenn ein Kunde kurzfristig eine Versicherungsbestätigung für die Zulassungsstelle benötigt und die vollständige Antragserfassung erst später erfolgen soll.

Sobald die Zulassungsstelle die eVB-Nummer verwendet (Rückmeldung über GDV), erzeugt das System automatisch eine **Antragsanmahnung** – eine Aufforderung, den fehlenden Antrag nachzuholen. Die in der Antragsanmahnung gespeicherten Daten (Partner, Fahrzeug, eVB, Zulassungsdaten) können anschließend beim Erstellen des Angebots/Antrags übernommen werden.

## Akteure
- **Primär:** Außendienst (Vertrieb), Innendienst (Sachbearbeitung)
- **Sekundär:** Zulassungsstelle (via GDV-Rückmeldung)

## Vorbedingungen
- Benutzer ist am System angemeldet und besitzt die Kompetenz „eVB erstellen"
- Kundendaten (Partner) sind vorhanden oder werden erfasst
- Mindestens Fahrzeugart und grundlegende Deckungsart (HP) sind bekannt

## Auslöser
- Benutzer wählt die Funktion „eVB ohne Antrag erstellen" (z. B. für Schnellzulassung)
- Alternativ: Telefonische Anforderung einer eVB durch den Kunden

---

## Teil 1: eVB ohne Antrag erstellen

### Ablauf (Hauptszenario)

1. Benutzer wählt die Funktion **„eVB ohne Antrag erstellen"**
2. System zeigt die **Minimalerfassung** an:
   - Partner auswählen / neu erfassen (Pflicht)
   - Fahrzeugart auswählen (Pflicht, → UC-KFZ-00)
   - Deckungsumfang festlegen (Pflicht): HP, HP+TK, HP+VK
   - Optionale Fahrzeugdaten: HSN/TSN, FIN, amtliches Kennzeichen
3. Benutzer bestätigt die Daten
4. System prüft Minimalplausibilitäten:
   - Partner vorhanden und aktiv
   - Deckungsumfang gültig für gewählte Fahrzeugart
   - Keine doppelte offene eVB für denselben Partner + Fahrzeugart
5. System erzeugt eine **eVB-Nummer** (7-stelliger alphanumerischer Code) im Status `ERZEUGT`
6. System meldet die eVB-Nummer an das **GDV-eVB-System** (→ 09_schnittstellen.md, API 3.9)
7. Status wechselt zu `GEMELDET`
8. System zeigt die eVB-Nummer an und stellt sie dem Kunden zur Verfügung (Anzeige, Druck via S5)

### Besonderheiten gegenüber Standard-eVB
- Die eVB-Nummer hat **keinen** `antrag_id` und **keinen** `vertrag_id` → beide Referenzen sind NULL
- Stattdessen wird `partner_id` und optional `fahrzeug_id` direkt an der EvbNummer gespeichert
- Die eVB ist „frei schwebend" bis ein Antrag/Vertrag zugeordnet wird

### Daten der Minimalerfassung

| Feld | Pflicht | Beschreibung | Quelle |
|------|---------|-------------|--------|
| Partner | ✅ | Versicherungsnehmer | Partnersuche / Neuanlage |
| Fahrzeugart | ✅ | FA-01 bis FA-08 | Auswahl (UC-KFZ-00) |
| Deckungsumfang | ✅ | HP, HP+TK, HP+VK | Auswahl |
| HSN/TSN | ❌ | Herstellerschlüssel | Manuelle Eingabe |
| FIN | ❌ | Fahrzeug-Identifikationsnummer | Manuelle Eingabe |
| Amtliches Kennzeichen | ❌ | Wunschkennzeichen / bestehendes | Manuelle Eingabe |

---

## Teil 2: Antragsanmahnung bei Zulassungsrückmeldung

### Ablauf (Hauptszenario)

1. Die **Zulassungsstelle** verwendet die eVB-Nummer zur Zulassung des Fahrzeugs
2. Das **GDV-System** sendet eine Rückmeldung an das Versicherungssystem:
   - eVB-Nummer
   - Bestätigung der Verwendung
   - Zulassungsdaten: amtliches Kennzeichen, Halter, FIN, Erstzulassung
3. System empfängt die GDV-Rückmeldung (Import → IMP-08)
4. System prüft die eVB-Nummer:
   - eVB existiert und Status = `GEMELDET` → weiter
   - eVB ist bereits einem Antrag/Vertrag zugeordnet → nur Status-Update auf `VERWENDET`, **keine** Antragsanmahnung
   - eVB existiert nicht oder ist storniert → Fehlermeldung loggen
5. Wenn eVB **ohne Antrag/Vertrag**: System erzeugt eine **Antragsanmahnung**:
   - Status: `OFFEN`
   - Verknüpft mit: eVB-Nummer, Partner
   - Übernommene Daten: Zulassungsdaten (Kennzeichen, FIN, Halter, Erstzulassung)
   - Frist: 14 Tage ab Zulassungsdatum (konfigurierbar)
6. eVB-Status wechselt zu `VERWENDET`
7. System erzeugt eine **Benachrichtigung** (NOT-10) an den zuständigen Sachbearbeiter / das Vertriebsteam
8. Antragsanmahnung erscheint in der **Aufgaben-Übersicht** des Innendienstes

### Antragsanmahnung – Gespeicherte Daten

| Datengruppe | Felder | Quelle |
|-------------|--------|--------|
| **Partner** | Partner-ID, Name, Adresse | Aus eVB-Erzeugung |
| **eVB** | eVB-Nummer, Deckungsumfang, Fahrzeugart | Aus eVB-Erzeugung |
| **Zulassungsdaten** | Amtliches Kennzeichen, FIN, Halter-Name, Erstzulassungsdatum | GDV-Rückmeldung |
| **Fahrzeugdaten** | HSN/TSN (falls in Rückmeldung enthalten) | GDV-Rückmeldung |
| **Fristen** | Anmahnungsfrist (14 Tage), Eskalationsfrist (28 Tage) | Konfiguration |

---

## Teil 3: Datenübernahme in Angebot/Antrag

### Ablauf (Hauptszenario)

1. Sachbearbeiter öffnet die **Antragsanmahnung** aus der Aufgaben-Übersicht
2. System zeigt die gesammelten Daten an (Partner, eVB, Zulassungsdaten)
3. Sachbearbeiter wählt **„Angebot aus Anmahnung erstellen"**
4. System erzeugt ein neues **Angebot** (→ UC-01) mit vorausgefüllten Daten:
   - Partner: übernommen
   - Sparte: KFZ
   - Fahrzeugart: übernommen aus eVB
   - Fahrzeugdaten: HSN/TSN, FIN, Kennzeichen aus Zulassungsrückmeldung
   - Deckungsumfang: übernommen aus eVB (HP, HP+TK, HP+VK)
5. Benutzer ergänzt fehlende Daten (z. B. SF-Klasse, Fahrerdaten, Nutzungsart)
6. Normaler Angebotsprozess (UC-01, Phase 2–5) läuft weiter
7. Bei Beantragung (UC-01, Phase 5):
   - Die bestehende eVB-Nummer wird dem Antrag zugeordnet (`EvbNummer.antrag_id = neue Antrag-ID`)
   - **Keine neue eVB** wird erzeugt (Hook `nach_Antragserstellung` erkennt vorhandene eVB)
8. Antragsanmahnung-Status wechselt zu `ERLEDIGT`

### Alternativszenarien

#### A1: Sachbearbeiter ordnet Anmahnung einem bestehenden Angebot/Antrag zu
1. Sachbearbeiter wählt **„Bestehendem Angebot/Antrag zuordnen"**
2. System zeigt offene Angebote/Anträge des Partners an
3. Sachbearbeiter wählt das passende Angebot/den passenden Antrag
4. System übernimmt die Zulassungsdaten und verknüpft die eVB
5. Antragsanmahnung-Status → `ERLEDIGT`

#### A2: eVB verfällt ohne Antrag (Fristablauf)
1. **Anmahnungsfrist** (14 Tage) läuft ab ohne Reaktion
2. System erzeugt eine **Eskalations-Benachrichtigung** (NOT-11) an den Teamleiter
3. **Eskalationsfrist** (28 Tage) läuft ab
4. System setzt Antragsanmahnung-Status auf `ESKALIERT`
5. Manuelle Bearbeitung durch Teamleiter erforderlich

#### A3: Stornierung der Antragsanmahnung
1. Sachbearbeiter stellt fest, dass kein Antrag benötigt wird (z. B. Kunde hat anderweitig versichert)
2. Sachbearbeiter wählt **„Anmahnung stornieren"** mit Begründung
3. Antragsanmahnung-Status → `STORNIERT`
4. eVB-Nummer bleibt im Status `VERWENDET` (Zulassung ist erfolgt)

#### A4: eVB wird nicht von der Zulassungsstelle verwendet
- Keine Rückmeldung → kein Antragsanmahnung
- Nach 6 Monaten: eVB wird automatisch storniert (bestehender Ablauf-Prozess)

---

## Antragsanmahnung – Statusmodell

```
         ┌─────────────────────────────────────────────────┐
         │                                                 │
         ▼                                                 │
    ┌─────────┐    Angebot/Antrag    ┌──────────┐         │
    │  OFFEN  │─────erstellt────────►│ ERLEDIGT │         │
    └────┬────┘                      └──────────┘         │
         │                                                 │
         │ 14 Tage Frist                                   │
         ▼                                                 │
    ┌────────────┐   28 Tage Frist   ┌───────────┐        │
    │ UEBERFAELLIG│──────────────────►│ ESKALIERT │        │
    └────────────┘                   └─────┬─────┘        │
                                           │               │
                                           │ manuell       │
                                           └───────────────┘
                                           
    Jeder Status → STORNIERT (manuell mit Begründung)
```

| Status | Beschreibung | Folgestatus |
|--------|-------------|-------------|
| `OFFEN` | Antragsanmahnung erzeugt, Bearbeitung steht aus | ERLEDIGT, UEBERFAELLIG, STORNIERT |
| `UEBERFAELLIG` | 14-Tage-Frist abgelaufen, noch nicht bearbeitet | ERLEDIGT, ESKALIERT, STORNIERT |
| `ESKALIERT` | 28-Tage-Frist abgelaufen, Teamleiter muss handeln | ERLEDIGT, STORNIERT |
| `ERLEDIGT` | Antrag wurde erstellt oder zugeordnet | – (Endzustand) |
| `STORNIERT` | Manuell storniert (kein Antrag nötig) | – (Endzustand) |

---

## Geschäftsregeln

| ID | Regel | Beschreibung |
|----|-------|-------------|
| GR-KFZ-EVB-01 | eVB ohne Antrag: Minimalprüfung | Partner + Fahrzeugart + Deckungsumfang müssen angegeben sein |
| GR-KFZ-EVB-02 | Keine doppelte offene eVB | Pro Partner + Fahrzeugart darf max. eine offene eVB ohne Antrag existieren |
| GR-KFZ-EVB-03 | Antragsanmahnung bei Zulassung | Bei GDV-Rückmeldung „eVB verwendet" und keinem zugeordneten Antrag/Vertrag → automatisch Antragsanmahnung erzeugen |
| GR-KFZ-EVB-04 | Anmahnungsfrist | Antragsanmahnung muss innerhalb von 14 Tagen bearbeitet werden (konfigurierbar) |
| GR-KFZ-EVB-05 | Eskalation | Nach 28 Tagen ohne Bearbeitung → Eskalation an Teamleiter |
| GR-KFZ-EVB-06 | Datenübernahme | Beim Erstellen eines Angebots aus einer Anmahnung werden alle verfügbaren Daten (Partner, Fahrzeug, eVB, Zulassungsdaten) vorausgefüllt |
| GR-KFZ-EVB-07 | Keine neue eVB bei Beantragung | Existiert bereits eine verwendete eVB für den Antrag, wird im Hook `nach_Antragserstellung` keine neue eVB erzeugt |
| GR-KFZ-EVB-08 | eVB-Pflichtversicherung | Bei Zulassungsrückmeldung ohne Antrag ist der Versicherer in der Leistungspflicht – Antragsanmahnung hat hohe Priorität |

---

## Betroffene Entitäten

| Entität | Änderung | Beschreibung |
|---------|----------|-------------|
| `EvbNummer` | Erweiterung | Neue Felder: `partner_id`, `fahrzeug_id`, `deckungsumfang`, `antragsanmahnung_id` |
| `Antragsanmahnung` | **NEU** | Neue Entität für den Anmahnungsprozess |
| `Angebot` | Erweiterung | Neues optionales Feld: `antragsanmahnung_id` zur Rückverfolgung |

## Betroffene Schnittstellen

| Schnittstelle | Änderung | Beschreibung |
|---------------|----------|-------------|
| KFZ-Sparten-API (2.7) | Neue Endpunkte | eVB ohne Antrag erstellen, Antragsanmahnungen CRUD |
| GDV REST API (3.9) | Neuer Import | Zulassungsrückmeldung empfangen (IMP-08) |
| Events (6) | Neue Events | EvbOhneAntragErstelltEvent, AntragsanmahnungErstelltEvent, AntragsanmahnungErledigtEvent |
| Benachrichtigungen (7) | Neue Notifications | NOT-10 (Antragsanmahnung), NOT-11 (Eskalation) |

## Einordnung im Sub-Modul

Dieser Use Case gehört zum **Sub-Modul `evb`** (eVB-Verwaltung), mit Abhängigkeiten zu:
- **kern**: Partner- und Fahrzeugdaten
- **sfr**: SF-Klasse wird erst im Antragsprozess ermittelt (nicht bei eVB ohne Antrag)
- **Kernprozess**: UC-01 (Angebot erstellen) für die Datenübernahme aus Anmahnung
