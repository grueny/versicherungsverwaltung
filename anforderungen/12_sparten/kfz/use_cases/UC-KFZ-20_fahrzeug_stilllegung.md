# UC-KFZ-20: Fahrzeug-Stilllegung durch Zulassungsstelle

## Beschreibung
Die Zulassungsstelle kann ein Fahrzeug **zwangsstilllegen** (Zwangsaußerbetriebsetzung), z. B. wegen fehlender Hauptuntersuchung (TÜV), nicht gezahlter Kfz-Steuer oder eines behördlichen Nutzungsverbots. Die Stilllegung wird über die **GDV-Schnittstelle** (Polling, IMP-05) empfangen und führt zur automatischen **Stornierung des zugehörigen KFZ-Vertrags**. Der Versicherungsnehmer erhält ein **Stornierungsdokument** über die Druckschnittstelle (S5).

Dieser Use Case unterscheidet sich von **UC-KFZ-06 (Ruheversicherung)**, bei dem der Kunde das Fahrzeug freiwillig abmeldet und eine beitragsfreie Ruheversicherung in Kraft tritt. Bei einer Zwangsstilllegung durch die Zulassungsstelle wird der Vertrag **direkt storniert** – eine Ruheversicherung greift in diesem Fall nicht.

Der Use Case wird über den **GDV-Polling-Mechanismus** (stündlich, → `GET /zulassung/abmeldungen`) ausgelöst und läuft primär als **automatisierter Hintergrundprozess** ab. Nur bei Unklarheiten (z. B. mehrere aktive Verträge zum Kennzeichen) erfolgt eine Aussteuerung an den Innendienst.

## Akteure
- **Primär:** System (automatisierter Prozess / Scheduler)
- **Sekundär:** Innendienst (Sachbearbeitung) – bei Aussteuerung
- **Extern:** Zulassungsstelle (via GDV-Schnittstelle)

## Vorbedingungen
- Ein aktiver KFZ-Vertrag existiert für das betroffene Fahrzeug (Status `AKTIV`)
- Die GDV-Schnittstelle (IMP-05) ist erreichbar und das Polling aktiv
- Das Fahrzeug ist anhand von amtlichem Kennzeichen und/oder FIN eindeutig einem Vertrag zuordenbar

## Auslöser
- Stündlicher **Polling-Job** empfängt eine Stilllegungsmeldung über die GDV-Schnittstelle (`GET https://api.gdv.de/v1/zulassung/abmeldungen`)
- Meldung enthält den Stilllegungsgrund `ZWANGSSTILLLEGUNG` (Abgrenzung zur freiwilligen Abmeldung → UC-KFZ-06)

## Ablauf (Hauptszenario)

### Phase 1: Stilllegungsmeldung empfangen
1. Der **Polling-Job** ruft stündlich die GDV-Schnittstelle ab (`GET /zulassung/abmeldungen?seit={letzter_abruf}`)
2. System empfängt eine oder mehrere Stilllegungsmeldungen
3. Jede Meldung wird in einer internen **Stilllegungs-Warteschlange** erfasst (Transactional Outbox)
4. Für jede Meldung wird die Verarbeitung einzeln angestoßen

### Phase 2: Vertragszuordnung
5. System sucht den zugehörigen Vertrag anhand des **amtlichen Kennzeichens** und/oder der **FIN** aus der Meldung
6. System prüft, ob **genau ein aktiver Vertrag** zum Fahrzeug existiert
7. System validiert, dass der Vertrag im Status `AKTIV` ist und die Sparte `KFZ` hat

### Phase 3: Vertragsstornierung
8. System erzeugt einen neuen **Vorgang** vom Typ `STORNO` mit Bezug zum Vertrag
9. System erzeugt einen neuen **Vertragsstand** (nächste Version) mit:
   - `gueltig_bis` = Datum der Stilllegung (aus GDV-Meldung)
   - `stornierungsgrund` = `ZWANGSSTILLLEGUNG`
   - `stornierungsdatum` = Datum der Stilllegung
10. System setzt den Vertragsstatus auf `STORNIERT`
11. System publiziert das Event `VertragStorniertEvent` mit Stornierungsgrund `ZWANGSSTILLLEGUNG`

### Phase 4: Folgeprozesse
12. **S2 (Inkasso):** Beitragsforderung wird anteilig storniert (pro-rata-temporis-Erstattung ab Stilllegungsdatum) → Kafka-Topic `vertrag.inkasso.beitragsforderungen`
13. **S1 (Provision):** Provisionsereignis `STORNO` wird publiziert → Kafka-Topic `vertrag.provision.ereignisse`
14. **S5 (Druck):** Stornierungsdokument wird erzeugt (Dokumenttyp `STILLLEGUNGSBESTAETIGUNG`) mit:
    - Vertragsnummer, Versicherungsnehmer-Daten
    - Fahrzeugdaten (Kennzeichen, Hersteller, Modell)
    - Stilllegungsdatum und -grund
    - Hinweis auf entfallenen Versicherungsschutz
    - Hinweis auf etwaige Beitragserstattung
15. **eVB-Nummer:** Falls eine aktive eVB-Nummer existiert, wird diese beim GDV storniert (`POST /evb/meldungen/{evb_nr}/storno`)
16. System erstellt eine **UI-Benachrichtigung** (NOT-10) an den zuständigen Sachbearbeiter zur Information

## Alternativszenarien

- **A1: Freiwillige Abmeldung statt Zwangsstilllegung**
  Die GDV-Meldung enthält den Grund `FREIWILLIGE_ABMELDUNG` statt `ZWANGSSTILLLEGUNG`. → Verarbeitung wird an **UC-KFZ-06 (Ruheversicherung)** delegiert. Kein Storno, sondern Überführung in Ruheversicherung.

- **A2: Mehrere aktive Verträge zum Kennzeichen**
  Das System findet mehr als einen aktiven Vertrag für das betroffene Kennzeichen (z. B. durch Datenfehler oder Fahrzeugwechsel in Bearbeitung). → **Aussteuerung an Innendienst:** Eine Schwebe wird erzeugt mit dem Hinweis „Stilllegung: Mehrere Verträge zum Kennzeichen – manuelle Zuordnung erforderlich". Der Sachbearbeiter wählt den korrekten Vertrag und löst die Stornierung manuell aus.

- **A3: Vertrag bereits in Ruheversicherung**
  Das Fahrzeug wurde bereits freiwillig abgemeldet und der Vertrag befindet sich im Status `RUHEND` (Ruheversicherung). → Die Zwangsstilllegung beendet auch die Ruheversicherung. Der Vertrag wird von `RUHEND` auf `STORNIERT` gesetzt. Beitragserstattung entfällt (Ruheversicherung ist beitragsfrei).

- **A4: Vertrag mit Versicherungskennzeichen (FA-04)**
  Bei Fahrzeugen mit Versicherungskennzeichen (Moped, E-Scooter etc.) wird zusätzlich der **VKZ-Rücknahme-Prozess** angestoßen (Kennzeichen-Status → `ZURUECKGENOMMEN`). Eine Wiedervorlage zur physischen Rückgabe des Kennzeichens wird erstellt.

- **A5: Manuelle Stilllegungserfassung durch Innendienst**
  Der Innendienst erfährt auf anderem Weg von der Stilllegung (z. B. Anruf des Kunden, schriftliche Mitteilung der Behörde) und erfasst die Stilllegung manuell im System. Der weitere Ablauf (Phase 3+4) ist identisch.

## Fehlerfälle

- **F1: Kein Vertrag zum Kennzeichen/FIN gefunden**
  Das System findet keinen aktiven Vertrag für die Fahrzeugdaten aus der GDV-Meldung. → Meldung wird als `NICHT_ZUORDENBAR` protokolliert. Eine UI-Benachrichtigung an den System-Admin wird erstellt. Mögliche Ursachen: Fahrzeug ist nicht bei uns versichert, Datenfehler beim GDV.

- **F2: GDV-Schnittstelle nicht erreichbar**
  Das Polling schlägt fehl (HTTP-Fehler, Timeout). → Resilience4j Circuit Breaker greift (→ 3.9 Schnittstellen). Retry nach exponentiellem Backoff. Bei dauerhaftem Ausfall: UI-Benachrichtigung an System-Admin (NOT-03).

- **F3: Vertrag bereits storniert**
  Der zugeordnete Vertrag ist bereits im Status `STORNIERT`. → Meldung wird als `BEREITS_VERARBEITET` protokolliert und übersprungen. Kein erneuter Druckauftrag.

- **F4: Druckschnittstelle (S5) nicht erreichbar**
  Der Druckauftrag kann nicht übermittelt werden. → Auftrag wird in der Outbox-Tabelle gespeichert und bei nächster Verfügbarkeit nachgesendet. Stornierung wird trotzdem durchgeführt.

- **F5: Inkasso-Stornierung schlägt fehl**
  Die Beitragserstattung kann nicht an S2 übermittelt werden. → Kafka Dead-Letter-Topic (`vertrag.inkasso.beitragsforderungen.dlt`). Manueller Retry über Admin-Dashboard. Vertragsstornierung wird trotzdem durchgeführt.

## Geschäftsregeln

| GR-Nr. | Regel | Auswirkung |
|--------|-------|-----------|
| GR-KFZ-SL01 | Eine Zwangsstilllegung durch die Zulassungsstelle führt immer zur Vertragsstornierung – eine Ruheversicherung ist nicht möglich | Kein Wahlrecht des VN wie bei freiwilliger Abmeldung |
| GR-KFZ-SL02 | Das Stornierungsdatum entspricht dem Stilllegungsdatum aus der GDV-Meldung, nicht dem Verarbeitungsdatum | Rückwirkende Stornierung möglich |
| GR-KFZ-SL03 | Bei Zwangsstilllegung erfolgt eine anteilige Beitragserstattung (pro rata temporis) ab dem Stilllegungsdatum | Erstattung an VN über Exkasso (S3) oder Verrechnung über Inkasso (S2) |
| GR-KFZ-SL04 | Aktive eVB-Nummern zum stornierten Vertrag werden automatisch beim GDV storniert | Zulassungsstelle erhält aktuellen Versicherungsstatus |
| GR-KFZ-SL05 | Der VN erhält ein Stornierungsdokument (Stilllegungsbestätigung) per Post oder über den hinterlegten Versandweg | Dokumentpflicht gegenüber dem Kunden |
| GR-KFZ-SL06 | Bei Vertrag mit Versicherungskennzeichen (FA-04) wird zusätzlich die VKZ-Rücknahme ausgelöst | Kennzeichen muss aus dem Umlauf genommen werden |

## Daten (Ein-/Ausgabe)

### Eingabedaten (GDV-Stilllegungsmeldung)

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| amtliches_kennzeichen | String | ✅ | Format: 1-3 Buchstaben + 1-2 Buchstaben + 1-4 Ziffern | Kennzeichen des stillgelegten Fahrzeugs |
| fin | String | ⬜ | 17-stellig, Prüfziffer | Fahrzeugidentnummer (falls vom GDV geliefert) |
| stilllegungsdatum | Date (ISO 8601) | ✅ | Darf nicht in der Zukunft liegen | Datum der behördlichen Stilllegung |
| stilllegungsgrund | Enum | ✅ | `ZWANGSSTILLLEGUNG`, `FREIWILLIGE_ABMELDUNG` | Art der Außerbetriebsetzung |
| zulassungsstelle_id | String | ✅ | Gültiger Bezirkscode | Kennung der meldenden Zulassungsstelle |
| gdv_referenz_id | String | ✅ | Eindeutige GDV-Referenz | Zur Deduplizierung und Nachverfolgung |
| meldezeitpunkt | DateTime (ISO 8601) | ✅ | – | Zeitpunkt der GDV-Meldung |

### Ausgabedaten (Stornierung)

| Feld | Typ | Beschreibung |
|------|-----|-------------|
| vertragsnummer | String | Nummer des stornierten Vertrags |
| stornierungsdatum | Date | Datum der Vertragsstornierung (= Stilllegungsdatum) |
| stornierungsgrund | Enum (`ZWANGSSTILLLEGUNG`) | Grund der Stornierung |
| neuer_vertragsstand_version | Integer | Versionsnummer des Storno-Vertragsstands |
| vorgang_id | UUID | ID des erzeugten Storno-Vorgangs |
| beitragserstattung_betrag | BigDecimal | Berechneter Erstattungsbetrag (pro rata temporis) |
| druckauftrag_id | UUID | ID des Druckauftrags an S5 |

## API-Endpunkt (Manuelle Stilllegungserfassung)

> Ergänzung zur KFZ-Sparten-API (→ 09_schnittstellen.md, Abschnitt 2.7)

| Methode | Endpunkt | Beschreibung | Kompetenz |
|---------|---------|-------------|-----------|
| `POST` | `/api/v1/kfz/vertraege/{vertrag_id}/aktionen/stilllegen` | Zwangsstilllegung manuell erfassen | `VERTRAG_STORNIEREN` |

### POST `/api/v1/kfz/vertraege/{vertrag_id}/aktionen/stilllegen`

**Request:**
```json
{
  "stilllegungsdatum": "2026-03-15",
  "stilllegungsgrund": "ZWANGSSTILLLEGUNG",
  "zulassungsstelle_id": "MS",
  "bemerkung": "Telefonische Mitteilung der Zulassungsstelle Münster"
}
```

**Response (200 OK):**
```json
{
  "vertrag_id": "f6a7b8c9-d0e1-2345-fabc-456789012345",
  "vertragsnummer": "VN-2026-100001",
  "status": "STORNIERT",
  "stornierungsdatum": "2026-03-15",
  "stornierungsgrund": "ZWANGSSTILLLEGUNG",
  "vorgang": {
    "id": "d4e5f6a7-b8c9-0123-defa-234567890123",
    "vorgangsnummer": "VG-2026-000042",
    "vorgangstyp": "STORNO"
  },
  "vertragsstand": {
    "id": "e5f6a7b8-c9d0-1234-efab-345678901234",
    "version": 2,
    "gueltig_bis": "2026-03-15"
  },
  "beitragserstattung": {
    "betrag": "237.18",
    "zeitraum_ab": "2026-03-15",
    "zeitraum_bis": "2026-12-31"
  },
  "druckauftrag_id": "a1b2c3d4-e5f6-7890-abcd-druckauftrag01"
}
```

**Response (409 Conflict – Vertrag nicht aktiv):**
```json
{
  "type": "https://api.versicherungsverwaltung.de/errors/status-conflict",
  "title": "Statuskonflikt",
  "status": 409,
  "detail": "Der Vertrag VN-2026-100001 ist im Status STORNIERT und kann nicht erneut stillgelegt werden."
}
```

## GDV-Stilllegungsmeldung (Polling-Response)

> Erweiterung des bestehenden GDV-Endpunkts (→ 09_schnittstellen.md, Abschnitt 3.9)

### GET `https://api.gdv.de/v1/zulassung/abmeldungen?seit=2026-03-19T13:00:00Z`

**Response (200 OK):**
```json
{
  "meldungen": [
    {
      "gdv_referenz_id": "GDV-ABM-2026-045678",
      "meldungstyp": "ZWANGSSTILLLEGUNG",
      "amtliches_kennzeichen": "MS-LV 1234",
      "fin": "WVWZZZ3CZWE123456",
      "stilllegungsdatum": "2026-03-15",
      "zulassungsstelle_id": "MS",
      "zulassungsstelle_name": "Zulassungsstelle Münster",
      "meldezeitpunkt": "2026-03-19T14:00:00Z",
      "grund_detail": "Hauptuntersuchung überfällig (> 6 Monate)"
    },
    {
      "gdv_referenz_id": "GDV-ABM-2026-045679",
      "meldungstyp": "FREIWILLIGE_ABMELDUNG",
      "amtliches_kennzeichen": "MS-AB 5678",
      "fin": "WVWZZZ3CZWE654321",
      "abmeldedatum": "2026-03-18",
      "zulassungsstelle_id": "MS",
      "zulassungsstelle_name": "Zulassungsstelle Münster",
      "meldezeitpunkt": "2026-03-19T14:05:00Z"
    }
  ],
  "anzahl": 2,
  "naechster_abruf_ab": "2026-03-19T14:05:00Z"
}
```

> Meldungen mit `meldungstyp: ZWANGSSTILLLEGUNG` → **Dieser Use Case (UC-KFZ-20)**  
> Meldungen mit `meldungstyp: FREIWILLIGE_ABMELDUNG` → **UC-KFZ-06 (Ruheversicherung)**

## Events

| Event | Auslöser | Consumer | Beschreibung |
|-------|---------|---------|-------------|
| `FahrzeugStillgelegtEvent` | GDV-Meldung empfangen oder manuelle Erfassung | Stornierungsprozess | Startet die Vertragsstornierung |
| `VertragStorniertEvent` | Phase 3, Schritt 11 | S1 (Provision), S2 (Inkasso), S5 (Druck) | Vertrag wegen Stilllegung storniert |
| `EvbStornoEvent` | Phase 4, Schritt 15 | GDV-eVB-Service | eVB-Nummer wird beim GDV storniert |

## Druckdokument (Stilllegungsbestätigung)

> Neuer Dokumenttyp für die Druckschnittstelle (S5, → 09_schnittstellen.md, Abschnitt 3.4)

| Feld | Beschreibung |
|------|-------------|
| `dokumenttyp` | `STILLLEGUNGSBESTAETIGUNG` |
| Empfänger | Versicherungsnehmer (Name, Adresse aus Partner-Daten) |
| Vertragsnummer | Nummer des stornierten Vertrags |
| Fahrzeugdaten | Amtliches Kennzeichen, Hersteller, Modell, FIN |
| Stilllegungsdatum | Datum der behördlichen Stilllegung |
| Stilllegungsgrund | Klartextbeschreibung (z. B. „Zwangsstilllegung durch Zulassungsstelle Münster") |
| Versicherungsschutz-Hinweis | „Der Versicherungsschutz für das o. g. Fahrzeug ist mit Wirkung zum {Datum} erloschen." |
| Beitragserstattung | Erstattungsbetrag und -zeitraum (falls zutreffend) |
| Kontaktdaten | Telefon/E-Mail der zuständigen Geschäftsstelle |

### Payload an S5

```json
{
  "dokumenttyp": "STILLLEGUNGSBESTAETIGUNG",
  "vertragsnummer": "VN-2026-100001",
  "partner": {
    "partnernummer": "KD-2026-004711",
    "anrede": "HERR",
    "vorname": "Max",
    "nachname": "Mustermann",
    "strasse": "Kolde-Ring",
    "hausnummer": "21",
    "plz": "48151",
    "ort": "Münster"
  },
  "sparte": "KFZ",
  "stornierungsdatum": "2026-03-15",
  "stornierungsgrund": "ZWANGSSTILLLEGUNG",
  "fahrzeug": {
    "amtliches_kennzeichen": "MS-LV 1234",
    "hersteller": "Volkswagen",
    "modell": "Golf VIII 1.5 TSI",
    "fin": "WVWZZZ3CZWE123456"
  },
  "beitragserstattung": {
    "betrag": "237.18",
    "zeitraum_ab": "2026-03-15",
    "zeitraum_bis": "2026-12-31"
  },
  "versandweg": "BRIEF",
  "prioritaet": "HOCH"
}
```

## Nachbedingungen
- Der KFZ-Vertrag ist im Status `STORNIERT`
- Ein Storno-Vorgang und ein neuer Vertragsstand (mit `gueltig_bis`) sind angelegt
- Die Beitragserstattung ist an S2 (Inkasso) übermittelt
- Das Provisionsereignis ist an S1 publiziert
- Das Stornierungsdokument (Stilllegungsbestätigung) ist an S5 übermittelt
- Eine etwaige aktive eVB-Nummer ist beim GDV storniert
- Der Sachbearbeiter ist per UI-Benachrichtigung informiert
- Die GDV-Stilllegungsmeldung ist als verarbeitet markiert (Deduplizierung)

## Akzeptanzkriterien
- [ ] Stilllegungsmeldungen werden stündlich über die GDV-Schnittstelle abgerufen (Polling)
- [ ] Meldungen mit `ZWANGSSTILLLEGUNG` werden korrekt von `FREIWILLIGE_ABMELDUNG` unterschieden (Routing UC-KFZ-20 vs. UC-KFZ-06)
- [ ] Der zugehörige Vertrag wird anhand von Kennzeichen und/oder FIN eindeutig identifiziert
- [ ] Bei genau einem Treffer wird der Vertrag automatisch storniert (Dunkelverarbeitung)
- [ ] Bei mehreren Treffern wird eine Schwebe für den Innendienst erzeugt (Aussteuerung)
- [ ] Das Stornierungsdatum im Vertrag entspricht dem Stilllegungsdatum (nicht dem Verarbeitungsdatum)
- [ ] Ein Storno-Vorgang und ein neuer Vertragsstand werden angelegt
- [ ] Die Beitragserstattung wird korrekt pro rata temporis berechnet und an S2 übermittelt
- [ ] Ein Stornierungsdokument (Stilllegungsbestätigung) wird über S5 erzeugt und an den VN gesendet
- [ ] Aktive eVB-Nummern werden beim GDV storniert
- [ ] Der Innendienst erhält eine UI-Benachrichtigung über die Stilllegung
- [ ] Duplikate werden erkannt und nicht erneut verarbeitet (`gdv_referenz_id`)
- [ ] Der Use Case kann alternativ manuell durch den Innendienst ausgelöst werden (API-Endpunkt)
- [ ] Bei GDV-Nichtverfügbarkeit greift der Circuit Breaker mit Retry

## Wireframe / Skizze (Manuelle Stilllegungserfassung)

```
┌─────────────────────────────────────────────────────────────────┐
│  Vertrag VN-2026-100001 – Aktionen ▾                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─── Fahrzeug-Stilllegung erfassen ──────────────────────────┐ │
│  │                                                             │ │
│  │  Fahrzeug:     MS-LV 1234 | VW Golf VIII | WVWZZZ3CZ...   │ │
│  │                                                             │ │
│  │  Stilllegungsdatum*:   [ 15.03.2026       📅 ]             │ │
│  │                                                             │ │
│  │  Stilllegungsgrund*:   [ Zwangsstilllegung         ▾ ]     │ │
│  │                                                             │ │
│  │  Zulassungsstelle*:    [ MS – Münster              ▾ ]     │ │
│  │                                                             │ │
│  │  Bemerkung:            [ Telefonische Mitteilung der    ]   │ │
│  │                        [ Zulassungsstelle Münster       ]   │ │
│  │                                                             │ │
│  │  ⚠️  Der Vertrag wird unwiderruflich storniert.             │ │
│  │      Beitragserstattung: ca. 237,18 € (pro rata temporis)  │ │
│  │      Ein Stornierungsdokument wird an den VN gesendet.      │ │
│  │                                                             │ │
│  │                    [ Abbrechen ]  [ Stilllegung erfassen ]  │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Bestätigungsdialog

```
┌───────────────────────────────────────────────┐
│  ⚠️  Fahrzeug-Stilllegung bestätigen          │
├───────────────────────────────────────────────┤
│                                               │
│  Vertrag:    VN-2026-100001                   │
│  Fahrzeug:   MS-LV 1234                       │
│  Storno zum: 15.03.2026                       │
│  Erstattung: 237,18 €                         │
│                                               │
│  Der Vertrag wird storniert und der           │
│  Kunde erhält eine Stilllegungsbestätigung.   │
│                                               │
│  Diese Aktion kann nicht rückgängig gemacht   │
│  werden.                                      │
│                                               │
│         [ Abbrechen ]  [ Bestätigen ]         │
└───────────────────────────────────────────────┘
```

## Sequenzdiagramm

```
GDV-API        Polling-Job     KFZ-Service     Vertrag-Service    S2 (Inkasso)    S5 (Druck)
   │                │               │                │                │               │
   │  GET /abmeld.  │               │                │                │               │
   │◄───────────────│               │                │                │               │
   │  Meldungen     │               │                │                │               │
   │───────────────►│               │                │                │               │
   │                │  Stilllegung  │                │                │               │
   │                │──────────────►│                │                │               │
   │                │               │  Vertrag suchen│                │               │
   │                │               │───────────────►│                │               │
   │                │               │  Vertrag gefund.│               │               │
   │                │               │◄───────────────│                │               │
   │                │               │  Storno auslösen│               │               │
   │                │               │───────────────►│                │               │
   │                │               │                │  VertragStorniertEvent          │
   │                │               │                │───────────────►│               │
   │                │               │                │  Beitragserstattung             │
   │                │               │                │                │               │
   │                │               │                │───────────────────────────────►│
   │                │               │                │  Druckauftrag Stilllegungsbest. │
   │                │               │                │                │               │
   │  POST /evb/    │               │                │                │               │
   │  storno        │               │                │                │               │
   │◄───────────────│───────────────│                │                │               │
   │                │               │                │                │               │
```

## Offene Fragen

| Nr. | Frage | Kontext |
|-----|-------|---------|
| UC-KFZ-20-01 | Soll bei einer Zwangsstilllegung eine Karenzzeit gewährt werden (z. B. 14 Tage bis zur tatsächlichen Stornierung), damit der VN reagieren kann? | Kundenfreundlichkeit vs. Regulatorik |
| UC-KFZ-20-02 | Gibt es Fälle, in denen eine Zwangsstilllegung rückgängig gemacht wird (z. B. TÜV nachgeholt)? Soll der Vertrag dann reaktiviert werden können? | Vertragskontinuität |
| UC-KFZ-20-03 | Soll die Beitragserstattung über S2 (Inkasso) oder S3 (Exkasso) abgewickelt werden? | Zahlungsverkehr |
| UC-KFZ-20-04 | Muss die Zwangsstilllegung dem Datawarehouse (S4) gesondert gemeldet werden (z. B. für Betrugsanalyse)? | Berichtswesen / Compliance |
| UC-KFZ-20-05 | Soll der Dokumenttyp `STILLLEGUNGSBESTAETIGUNG` ein eigenes Template erhalten oder als Variante der `KUENDIGUNGSBESTAETIGUNG` geführt werden? | Druckschnittstelle / Dokumentendesign |
