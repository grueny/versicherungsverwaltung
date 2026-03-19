# UC-01: Angebot erstellen und beantragen

## Beschreibung
Der Benutzer erstellt ein Angebot für einen Kunden. Das Angebot kann mit Produkt- und Vertragsdaten befüllt, berechnet (Beitragskalkulation) und geprüft (Plausibilitäten) werden. Ist das Angebot vollständig und fehlerfrei, kann der Benutzer es beantragen – dabei wird aus dem Angebot ein Antrag erzeugt. Der Benutzer entscheidet beim Beantragen, ob das zugrunde liegende Angebot erhalten bleibt oder gelöscht wird.

Dies ist ein **spartenübergreifender** Kernprozess. Spartenspezifische Besonderheiten (z. B. eVB-Erzeugung bei KFZ) werden über die Extension Points / Hooks der Spartenkonfiguration eingebunden.

## Akteure
- **Primär:** Außendienst (Vertrieb), Innendienst (Sachbearbeitung)
- **Sekundär:** Kunde (via Kundenportal, über Schnittstelle S7)

## Vorbedingungen
- Benutzer ist am System angemeldet und besitzt die Kompetenz „Angebot erstellen"
- Kundendaten sind vorhanden oder werden im Rahmen des Angebots neu erfasst
- Mindestens eine Sparte ist im System konfiguriert und aktiv

## Auslöser
- Benutzer wählt „Neues Angebot erstellen" im System
- Alternativ: Kunde startet Angebotsprozess über das Kundenportal

## Ablauf (Hauptszenario)

### Phase 1: Angebot anlegen
1. Benutzer wählt die Funktion „Neues Angebot erstellen"
2. System erzeugt ein neues Angebot im Status **„Entwurf"** und vergibt eine eindeutige Angebots-ID
3. Benutzer wählt den Kunden aus (Bestandskunde) oder erfasst einen neuen Kunden
4. Benutzer wählt die gewünschte Sparte aus

### Phase 2: Angebot befüllen
5. System zeigt die für die gewählte Sparte verfügbaren Produkte und Deckungsbausteine an (aus Spartenkonfiguration / DB)
6. → **Hook `vor_Angebotserstellung`** wird ausgelöst (spartenspezifische Vorabprüfungen und Zusatzerfassungen, z. B. Fahrzeugdaten bei KFZ)
7. Benutzer wählt Produkte und Deckungsbausteine aus
8. Benutzer erfasst weitere vertragsrelevante Daten (Laufzeit, Beginn, Selbstbeteiligung etc.)
9. System führt bei jeder Eingabe **Plausibilitätsprüfungen** durch (spartenübergreifend + spartenspezifisch aus Regelengine)
10. System zeigt Plausibilitätsfehler und -hinweise in Echtzeit an

### Phase 3: Angebot berechnen
11. Benutzer löst die **Beitragsberechnung** aus (oder diese erfolgt automatisch bei vollständiger Datenlage)
12. System berechnet den voraussichtlichen Beitrag auf Basis der gewählten Produkte, Tarife und Merkmale
13. System zeigt den berechneten Beitrag und die Beitragsdetails an
14. Angebotsstatus wechselt zu **„Berechnet"**

### Phase 4: Angebot prüfen
15. Benutzer löst die **Gesamtprüfung** des Angebots aus
16. System führt alle relevanten Plausibilitätsprüfungen durch (spartenübergreifend + spartenspezifisch)
17. System zeigt das Prüfergebnis an:
    - ✅ **Prüfung bestanden** → Angebot ist beantragbar, Status wechselt zu **„Geprüft"**
    - ❌ **Prüfung nicht bestanden** → Fehler werden angezeigt, Angebot bleibt im Status „Berechnet" und muss korrigiert werden

### Phase 5: Angebot beantragen
18. Benutzer wählt die Funktion „Angebot beantragen"
19. System fragt den Benutzer: **„Soll das Angebot nach der Überführung erhalten bleiben?"**
    - Ja → Angebot bleibt im Status **„Beantragt"** erhalten
    - Nein → Angebot wird nach erfolgreicher Antragserzeugung gelöscht
20. System erzeugt aus dem Angebot einen neuen **Antrag** im Status **„Offen"**
21. Alle Daten des Angebots (Kunde, Produkte, Beitrag) werden in den Antrag übernommen
22. → **Hook `nach_Antragserstellung`** wird ausgelöst (spartenspezifische Aktionen, z. B. eVB-Erzeugung bei KFZ)
23. System zeigt den erzeugten Antrag an und bestätigt die erfolgreiche Beantragung

## Alternativszenarien

- **A1: Angebot zwischenspeichern**
  Der Benutzer kann das Angebot jederzeit im aktuellen Status speichern und später weiterbearbeiten. Das Angebot bleibt im jeweiligen Status (Entwurf, Berechnet) erhalten.

- **A2: Angebot verwerfen**
  Der Benutzer kann ein Angebot jederzeit löschen, solange es noch nicht beantragt wurde. Das System fragt zur Sicherheit nach: „Angebot wirklich löschen?"

- **A3: Angebot kopieren**
  Der Benutzer kann ein bestehendes Angebot kopieren, um ein ähnliches Angebot für denselben oder einen anderen Kunden zu erstellen. Das kopierte Angebot erhält eine neue Angebots-ID und den Status „Entwurf".

- **A4: Angebot drucken / exportieren**
  Der Benutzer kann ein berechnetes oder geprüftes Angebot als Dokument über die Druckschnittstelle (S5) ausgeben lassen.

- **A5: Beantragung über Kundenportal**
  Der Kunde erstellt und beantragt das Angebot selbstständig über das Kundenportal. Der Prozess ist identisch, aber die Kompetenzprüfung und die verfügbaren Funktionen können eingeschränkt sein.

## Fehlerfälle

- **F1: Plausibilitätsfehler bei Prüfung**
  Das Angebot enthält Fehler (z. B. Pflichtfelder nicht befüllt, ungültige Produktkombination). → System zeigt alle Fehler an, Beantragung wird blockiert.

- **F2: Beitragsberechnung nicht möglich**
  Unvollständige oder widersprüchliche Daten verhindern die Berechnung. → System zeigt die fehlenden/fehlerhaften Felder an.

- **F3: Kompetenz nicht vorhanden**
  Benutzer besitzt nicht die erforderliche Kompetenz für einen Schritt (z. B. „Angebot beantragen"). → System zeigt Hinweis an: „Keine Berechtigung für diese Aktion."

- **F4: Sparte nicht konfiguriert**
  Die gewählte Sparte hat keine aktive Konfiguration. → System zeigt Hinweis an: „Sparte derzeit nicht verfügbar."

## Geschäftsregeln
- GR-01: Ein Angebot kann nur beantragt werden, wenn es den Status „Geprüft" hat
- GR-02: Bei der Beantragung entscheidet der Benutzer, ob das Angebot erhalten bleibt oder gelöscht wird
- GR-03: Ein Antrag übernimmt alle Daten des Angebots; nachträgliche Änderungen am Angebot wirken sich nicht auf den Antrag aus
- GR-04: Ein Angebot ohne Aktivität kann nach einem konfigurierbaren Zeitraum automatisch verfallen (→ offene Frage: Verfallszeitraum)

## Daten (Ein-/Ausgabe)

### Eingabedaten
| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| Kunde | Referenz / Neuerfassung | ✅ | Kunde muss vollständig erfasst sein | Zuordnung des Angebots zu einem Kunden |
| Sparte | Auswahl | ✅ | Sparte muss aktiv konfiguriert sein | Gewünschte Versicherungssparte |
| Produkte | Mehrfachauswahl | ✅ | Produktkombination muss gültig sein | Gewählte Produkte und Deckungsbausteine |
| Vertragsbeginn | Datum | ✅ | ≥ heute | Gewünschter Versicherungsbeginn |
| Laufzeit | Auswahl / Eingabe | ✅ | Gemäß Spartenregeln | Vertragslaufzeit |
| Selbstbeteiligung | Betrag | Spartenabhängig | Gemäß Produktdefinition | Gewünschte Selbstbeteiligung |
| Zahlungsweise | Auswahl | ✅ | jährlich / halbjährlich / vierteljährlich / monatlich | Beitragszahlweise |
| Spartenspezifische Daten | Variabel | Spartenabhängig | Über Spartenplausibilitäten | z. B. Fahrzeugdaten, Gebäudedaten |
| Angebot behalten | Boolean | ✅ (bei Beantragung) | – | Soll das Angebot nach Beantragung erhalten bleiben? |

### Ausgabedaten
| Feld | Typ | Beschreibung |
|------|-----|-------------|
| Angebots-ID | String (eindeutig) | Systemseitig vergebene Kennung des Angebots |
| Angebotsstatus | Enum | Entwurf → Berechnet → Geprüft → Beantragt / Gelöscht |
| Berechneter Beitrag | Betrag | Voraussichtlicher Jahresbeitrag (brutto) |
| Beitragsdetails | Liste | Aufschlüsselung nach Produkten/Bausteinen |
| Prüfergebnis | Liste | Alle Plausibilitätsergebnisse (Fehler / Warnungen / Hinweise) |
| Antrags-ID | String (eindeutig) | ID des erzeugten Antrags (nach Beantragung) |

## Statusmodell Angebot

```
  ┌──────────┐    befüllen /     ┌──────────┐    Prüfung      ┌──────────┐
  │ Entwurf  │───berechnen──────►│Berechnet │───bestanden────►│ Geprüft  │
  └──────────┘                   └──────────┘                  └────┬─────┘
       │                              │                             │
       │ löschen                      │ löschen              beantragen
       ▼                              ▼                             │
  ┌──────────┐                   ┌──────────┐                ┌─────▼──────┐
  │ Gelöscht │                   │ Gelöscht │                │ Beantragt  │
  └──────────┘                   └──────────┘                │ oder       │
                                                             │ Gelöscht   │
                                                             └────────────┘
                                                                    │
                                                              erzeugt
                                                                    ▼
                                                             ┌────────────┐
                                                             │  Antrag    │
                                                             │  (Offen)   │
                                                             └────────────┘
```

## Nachbedingungen
- Ein neuer Antrag im Status „Offen" wurde erzeugt mit allen Daten des Angebots
- Das Angebot befindet sich im Status „Beantragt" (wenn beibehalten) oder wurde gelöscht
- Spartenspezifische Hooks (`nach_Antragserstellung`) wurden ausgeführt
- Alle Statuswechsel sind revisionssicher protokolliert (Benutzer, Zeitstempel)

## Akzeptanzkriterien
- [ ] Ein Angebot kann angelegt, befüllt, berechnet und geprüft werden
- [ ] Plausibilitätsprüfungen (spartenübergreifend + spartenspezifisch) werden korrekt ausgeführt und Fehler angezeigt
- [ ] Die Beitragsberechnung liefert einen korrekten Beitrag auf Basis der Produktkonfiguration
- [ ] Ein geprüftes Angebot kann beantragt werden; dabei wird ein Antrag mit allen Daten erzeugt
- [ ] Der Benutzer wird gefragt, ob das Angebot erhalten bleiben soll, und die Entscheidung wird korrekt umgesetzt
- [ ] Ein Angebot im Status „Entwurf" oder „Berechnet" kann nicht beantragt werden
- [ ] Spartenspezifische Hooks (`vor_Angebotserstellung`, `nach_Antragserstellung`) werden ausgelöst
- [ ] Alle Statuswechsel werden revisionssicher historisiert
- [ ] Der Prozess funktioniert identisch über alle konfigurierten Sparten hinweg

## Wireframe / Skizze

```
┌─────────────────────────────────────────────────────────┐
│  Angebot #AG-2026-001234          Status: ⬤ Berechnet  │
├─────────────────────────────────────────────────────────┤
│  Kunde: Max Mustermann (KD-4711)         [Ändern]      │
│  Sparte: KFZ                                           │
├─────────────────────────────────────────────────────────┤
│  Produkte:                                              │
│  ☑ KFZ-Haftpflicht          ☑ Teilkasko                │
│  ☐ Vollkasko                ☑ LVM-Schutzbrief           │
├─────────────────────────────────────────────────────────┤
│  Vertragsbeginn: 01.01.2027    Laufzeit: 1 Jahr        │
│  Zahlweise: Jährlich          SB Teilkasko: 150 €      │
├─────────────────────────────────────────────────────────┤
│  ┌─── Spartenspezifisch (KFZ) ──────────────────────┐  │
│  │ Fahrzeug: VW Golf VIII (HSN/TSN: 0603/BPM)      │  │
│  │ Typklasse HP: 15 | TK: 20 | VK: 17              │  │
│  │ SF-Klasse: SF 5   Fahrerkreis: ab 25 Jahre       │  │
│  └──────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────┤
│  Berechneter Jahresbeitrag:            € 487,32        │
│  ├─ KFZ-Haftpflicht:                  € 312,00        │
│  ├─ Teilkasko:                         € 167,11        │
│  └─ LVM-Schutzbrief:                  €   8,21        │
├─────────────────────────────────────────────────────────┤
│  ⚠ Hinweis: Gelegentliche Fahrer ab 23 mitversichert  │
├─────────────────────────────────────────────────────────┤
│  [Speichern]  [Berechnen]  [Prüfen]  [Beantragen]     │
└─────────────────────────────────────────────────────────┘
```

## Offene Fragen
- Nach welchem Zeitraum verfällt ein nicht beantragtes Angebot automatisch?
- Sollen Angebote versioniert werden (z. B. bei Neuberechnung nach Datenänderung)?
- Gibt es eine maximale Anzahl offener Angebote pro Kunde?
