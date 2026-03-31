# UC-KFZ-21: SF-Klasse zwischen Verträgen wechseln (SFR-Transfer)

## Beschreibung
Der **Schadenfreiheitsrabatt (SFR)** – konkret die SF-Klasse – kann von einem bestehenden KFZ-Vertrag auf einen anderen KFZ-Vertrag **übertragen** werden. Dies ist ein häufiger Vorgang, wenn z. B.:

- Ein Versicherungsnehmer ein Zweitfahrzeug versichert und die bessere SF-Klasse des Erstfahrzeugs auf das teurere Fahrzeug übertragen möchte
- Ein Familienmitglied (Ehepartner, Kind) die SF-Klasse des bisherigen Hauptnutzers übernimmt
- Ein verstorbener Versicherungsnehmer seine SF-Klasse an einen Erben überträgt
- Bei einem Fahrzeugtausch innerhalb des Haushalts die SF-Klassen neu verteilt werden sollen

Beim Transfer wird die SF-Klasse des **Quellvertrags** auf den **Zielvertrag** übertragen. Der Quellvertrag erhält anschließend eine neue SF-Klasse – entweder die des Zielvertrags (Tausch) oder eine Rückstufung auf die Basis-SF-Klasse (SF½). Beide Verträge werden neu tarifiert und der Versicherungsnehmer erhält Nachtrags-Dokumente für beide Verträge.

Der SFR-Transfer ist ein **manueller Innendienst-Prozess**, da er immer eine fachliche Prüfung der Berechtigung erfordert (Verwandtschaftsverhältnis, Haushaltszugehörigkeit, Erbberechtigung etc.).

## Akteure
- **Primär:** Innendienst (Sachbearbeitung)
- **Sekundär:** Außendienst (Vertrieb) – stellt den Antrag auf SFR-Transfer
- **Beteiligt:** Versicherungsnehmer des Quell- und Zielvertrags

## Vorbedingungen
- Mindestens zwei aktive KFZ-Verträge existieren, die am SFR-Transfer beteiligt sind
- Beide Verträge enthalten das Produkt **KFZ-Haftpflicht (KFZ-HP)** (SF-Klasse ist HP-gebunden)
- Der Quellvertrag hat eine SF-Klasse > SF½ (es muss ein übertragbarer Rabatt vorhanden sein)
- Der Benutzer besitzt die Kompetenz `VERTRAG_SF_TRANSFER`
- Es liegt ein **Nachweis der Berechtigung** vor (Verwandtschaft, Haushalt, Erbschein etc.)

## Auslöser
- Antrag des Versicherungsnehmers auf SFR-Transfer (schriftlich, telefonisch oder über Außendienst)
- Innendienst-Sachbearbeiter eröffnet den SFR-Transfer im System

## Ablauf (Hauptszenario)

### Phase 1: Transfer-Anforderung erfassen
1. Sachbearbeiter ruft die Funktion **„SF-Klasse übertragen"** auf (über Vertragsdetail-Ansicht oder Suchfunktion)
2. Sachbearbeiter wählt den **Quellvertrag** aus (Vertrag, von dem die SF-Klasse abgegeben wird)
3. Sachbearbeiter wählt den **Zielvertrag** aus (Vertrag, der die SF-Klasse empfangen soll)
4. System zeigt eine **Vorschau** mit:
   - Aktuelle SF-Klassen beider Verträge (HP und ggf. VK)
   - Geplante SF-Klassen nach Transfer
   - Beitragsauswirkung auf beide Verträge (Differenzberechnung)

### Phase 2: Berechtigungsprüfung
5. System prüft die **Transfervoraussetzungen** automatisch:
   - Beide Verträge sind im Status `AKTIV`
   - Beide Verträge gehören zur Sparte `KFZ`
   - Quellvertrag hat SF-Klasse > SF½
   - Keine laufende SF-Rückstufung oder offene Schadenmeldung auf dem Quellvertrag
   - Kein laufender VWB-Vorgang auf einem der beiden Verträge
6. Sachbearbeiter erfasst den **Transfergrund** und den **Berechtigungsnachweis**:
   - Transfergrund: `ZWEITFAHRZEUG`, `FAMILIENTRANSFER`, `ERBFALL`, `HAUSHALTSZUSAMMENFUEHRUNG`, `SONSTIGES`
   - Nachweis-Art: `HAUSHALTSBESCHEINIGUNG`, `HEIRATSURKUNDE`, `ERBSCHEIN`, `EIGENERKLAERUNG`, `SONSTIGES`
   - Optional: Bemerkung / Aktennotiz

### Phase 3: Transfer-Modus festlegen
7. Sachbearbeiter wählt den **Transfer-Modus**:
   - **Einseitiger Transfer:** SF-Klasse wird vom Quellvertrag auf den Zielvertrag übertragen. Der Quellvertrag wird auf SF½ zurückgesetzt.
   - **Tausch:** Die SF-Klassen beider Verträge werden getauscht. (Quell erhält SF von Ziel, Ziel erhält SF von Quell.)
8. System berechnet die **neuen Beiträge** für beide Verträge auf Basis der zukünftigen SF-Klassen

### Phase 4: Transfer durchführen
9. Sachbearbeiter bestätigt den Transfer (nach 4-Augen-Prüfung bei SF-Klasse ≥ SF20)
10. System erzeugt für **beide Verträge** jeweils:
    - Einen neuen **Vorgang** vom Typ `NACHTRAG` mit Bezug `SF_TRANSFER`
    - Einen neuen **Vertragsstand** (nächste Version) mit der neuen SF-Klasse
    - Einen neuen Eintrag in der **SfKlassenHistorie** mit Änderungsgrund `SF_TRANSFER`
11. System aktualisiert die **KfzTarifierung** beider Verträge mit den neuen SF-Klassen
12. System führt eine **Beitragsneuberechnung** für beide Verträge durch

### Phase 5: Folgeprozesse
13. **S2 (Inkasso):** Geänderte Beitragsforderungen für beide Verträge übermitteln → Kafka-Topic `vertrag.inkasso.beitragsforderungen`
14. **S1 (Provision):** Provisionsereignis `NACHTRAG` für beide Verträge publizieren → Kafka-Topic `vertrag.provision.ereignisse`
15. **S5 (Druck):** Nachtragsdokument für **beide Verträge** erzeugen (Dokumenttyp `NACHTRAG`) mit:
    - Hinweis auf SF-Klassen-Änderung und Transferpartner
    - Neue Beitragsberechnung
    - Wirksamkeitsdatum des Transfers
16. System protokolliert den Transfer in der **Änderungshistorie** (Hibernate Envers) beider Verträge mit Querverweisen

## Alternativszenarien

- **A1: Verträge gehören verschiedenen Versicherungsnehmern**
  Quell- und Zielvertrag haben unterschiedliche Versicherungsnehmer (z. B. Ehepartner, Eltern→Kind). Der Sachbearbeiter muss den Berechtigungsnachweis dokumentieren (Haushaltsbescheinigung, Heiratsurkunde, Erbschein). System prüft, ob die Partner-IDs im selben Haushalt registriert sind (gleiche Adresse) oder ob ein expliziter Nachweis hinterlegt ist.

- **A2: Quellvertrag wird nach Transfer gekündigt**
  Der Quellvertrag soll nach dem SF-Transfer beendet werden (z. B. Verkauf des Fahrzeugs). Der Sachbearbeiter kann den Transfer mit gleichzeitiger Kündigung des Quellvertrags kombinieren. In diesem Fall wird kein neuer Beitrag für den Quellvertrag berechnet, sondern der Vertrag wird direkt storniert/gekündigt.

- **A3: Transfer nur der VK-SF-Klasse**
  In manchen Fällen soll nur die Vollkasko-SF-Klasse übertragen werden, während die HP-SF-Klasse unverändert bleibt (oder umgekehrt). Der Sachbearbeiter kann die zu übertragenden SF-Klassen einzeln auswählen (HP, VK oder beide).

- **A4: Transfer mit Wirksamkeitsdatum in der Zukunft**
  Der Transfer soll nicht sofort, sondern zu einem bestimmten zukünftigen Datum wirksam werden (z. B. zum nächsten Fälligkeitstermin 01.01.). Das System erfasst das Wirksamkeitsdatum und führt den Transfer per Scheduler-Job zum Stichtag aus.

- **A5: Transfer bei Vertrag in Ruheversicherung**
  Der Quellvertrag befindet sich im Status `RUHEND` (Ruheversicherung nach Fahrzeugabmeldung). Die SF-Klasse kann trotzdem übertragen werden, da sie während der Ruheversicherung erhalten bleibt (→ GR-KFZ-PR02). Der Quellvertrag bleibt ruhend mit SF½.

- **A6: 4-Augen-Prüfung bei hoher SF-Klasse**
  Bei SF-Klasse ≥ SF20 am Quellvertrag wird eine 4-Augen-Prüfung (Aussteuerung an Teamleiter) erzwungen, da der finanzielle Impact signifikant ist. Eine Schwebe mit dem Aussteuerungsgrund `SF_TRANSFER_HOHE_SFKLASSE` wird erzeugt.

## Fehlerfälle

- **F1: Quellvertrag hat SF½ oder SF0**
  Die SF-Klasse des Quellvertrags bietet keinen übertragbaren Vorteil. → System blockiert den Transfer mit Hinweis: „Die SF-Klasse des Quellvertrags (SF½) kann nicht übertragen werden."

- **F2: Vertrag ist nicht aktiv**
  Einer der beiden Verträge ist nicht im Status `AKTIV` oder `RUHEND`. → System zeigt Fehler: „Der Vertrag {Vertragsnummer} ist im Status {Status} und kann nicht an einem SF-Transfer teilnehmen."

- **F3: Offene Schadenmeldung auf Quellvertrag**
  Auf dem Quellvertrag ist ein Schaden gemeldet, der noch nicht abschließend reguliert ist. → System warnt: „Auf dem Quellvertrag liegt ein offener Schadenvorgang vor. Der Transfer kann erst nach Abschluss der Schadenregulierung durchgeführt werden." Sachbearbeiter kann mit Override fortfahren (Kompetenz `SF_TRANSFER_OVERRIDE`).

- **F4: Laufender VWB-Vorgang**
  Für einen der Verträge läuft ein VWB-Verfahren (SF-Klasse noch nicht bestätigt). → System blockiert den Transfer: „Für den Vertrag {Vertragsnummer} läuft ein VWB-Verfahren. Bitte warten Sie die SF-Bestätigung ab."

- **F5: Kompetenz fehlt**
  Der Benutzer hat nicht die Kompetenz `VERTRAG_SF_TRANSFER` oder `SF_TRANSFER_OVERRIDE`. → HTTP 403 Forbidden.

- **F6: Gleicher Vertrag als Quelle und Ziel**
  Der Sachbearbeiter versucht, denselben Vertrag als Quell- und Zielvertrag zu wählen. → System zeigt Fehler: „Quell- und Zielvertrag müssen unterschiedlich sein."

## Geschäftsregeln

| GR-Nr. | Regel | Auswirkung |
|--------|-------|-----------|
| GR-KFZ-SF01 | Ein SF-Transfer ist nur zwischen KFZ-Verträgen mit HP-Produkt möglich | Verträge ohne KFZ-HP können nicht am Transfer teilnehmen |
| GR-KFZ-SF02 | Bei einseitigem Transfer wird der Quellvertrag auf SF½ zurückgesetzt | Der Quellvertrag startet de facto mit dem Grundbeitrag neu |
| GR-KFZ-SF03 | SF-Transfer zwischen verschiedenen VN ist nur bei nachgewiesener Berechtigung erlaubt (Haushalt, Familie, Erbfall) | Nachweis muss dokumentiert und archiviert werden |
| GR-KFZ-SF04 | Bei SF-Klasse ≥ SF20 auf dem Quellvertrag ist eine 4-Augen-Prüfung erforderlich | Aussteuerung an Teamleiter zur Genehmigung |
| GR-KFZ-SF05 | Der SF-Transfer wird zum Wirksamkeitsdatum in der SfKlassenHistorie beider Verträge eingetragen (Änderungsgrund `SF_TRANSFER`) | Vollständige Nachvollziehbarkeit der SF-Historie |
| GR-KFZ-SF06 | Ein laufender SF-Transfer blockiert gleichzeitige Änderungen an den SF-Klassen beider Verträge (Pessimistic Locking) | Vermeidung von Konflikten bei parallelen Schaden- oder VWB-Vorgängen |
| GR-KFZ-SF07 | Der RabattSchutz (LVM-RabattSchutz) wird durch einen SF-Transfer nicht beeinflusst und bleibt auf dem jeweiligen Vertrag bestehen | Kein Verlust des separat gebuchten RabattSchutzes |
| GR-KFZ-SF08 | Bei Kündigung des Quellvertrags innerhalb von 6 Monaten nach Transfer wird der Transfer nicht rückabgewickelt | Kein automatischer Rücktransfer bei nachträglicher Kündigung |

## Daten (Ein-/Ausgabe)

### Eingabedaten

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| quellvertrag_id | UUID | ✅ | Aktiver KFZ-Vertrag mit HP | Vertrag, von dem die SF-Klasse abgegeben wird |
| zielvertrag_id | UUID | ✅ | Aktiver KFZ-Vertrag mit HP, ≠ quellvertrag_id | Vertrag, der die SF-Klasse empfangen soll |
| transfer_modus | Enum | ✅ | `EINSEITIG` oder `TAUSCH` | Art des Transfers |
| sf_klassen_auswahl | Enum | ✅ | `HP`, `VK`, `HP_UND_VK` | Welche SF-Klassen übertragen werden sollen |
| wirksamkeitsdatum | Date (ISO 8601) | ✅ | ≥ heute, ≤ 12 Monate in Zukunft | Datum der Wirksamkeit des Transfers |
| transfer_grund | Enum | ✅ | Gültiger Transfergrund | Begründung für den Transfer |
| nachweis_art | Enum | ✅ | Gültige Nachweisart | Art des Berechtigungsnachweises |
| nachweis_dokument_id | UUID | ⬜ | Referenz auf archiviertes Dokument | Scan/Upload des Nachweisdokuments |
| bemerkung | String(1000) | ⬜ | Max. 1000 Zeichen | Freitext-Bemerkung / Aktennotiz |

### Ausgabedaten

| Feld | Typ | Beschreibung |
|------|-----|-------------|
| transfer_id | UUID | Eindeutige ID des SFR-Transfers |
| transfer_nummer | String | Fachliche Vorgangsnummer (z. B. `SFT-2026-000001`) |
| quellvertrag.sf_klasse_alt | String | Bisherige SF-Klasse des Quellvertrags |
| quellvertrag.sf_klasse_neu | String | Neue SF-Klasse des Quellvertrags |
| quellvertrag.jahresbeitrag_alt | BigDecimal | Bisheriger Jahresbeitrag |
| quellvertrag.jahresbeitrag_neu | BigDecimal | Neuer Jahresbeitrag nach Transfer |
| quellvertrag.vorgang_id | UUID | ID des erzeugten Nachtragsvorgangs |
| zielvertrag.sf_klasse_alt | String | Bisherige SF-Klasse des Zielvertrags |
| zielvertrag.sf_klasse_neu | String | Neue SF-Klasse des Zielvertrags |
| zielvertrag.jahresbeitrag_alt | BigDecimal | Bisheriger Jahresbeitrag |
| zielvertrag.jahresbeitrag_neu | BigDecimal | Neuer Jahresbeitrag nach Transfer |
| zielvertrag.vorgang_id | UUID | ID des erzeugten Nachtragsvorgangs |
| wirksamkeitsdatum | Date | Stichtag des Transfers |
| status | Enum | Status des Transfers (`DURCHGEFUEHRT`, `GENEHMIGUNG_ERFORDERLICH`) |

## API-Endpunkt

> Ergänzung zur KFZ-Sparten-API (→ 09_schnittstellen.md, Abschnitt 2.7)

| Methode | Endpunkt | Beschreibung | Kompetenz |
|---------|---------|-------------|-----------|
| `POST` | `/api/v1/kfz/sf-transfers` | SFR-Transfer durchführen | `VERTRAG_SF_TRANSFER` |
| `POST` | `/api/v1/kfz/sf-transfers/vorschau` | Transfer-Vorschau berechnen (ohne Durchführung) | `VERTRAG_LESEN` |
| `GET` | `/api/v1/kfz/sf-transfers` | SFR-Transfers suchen (Filter: status, vertrag_id, partner_id) | `VERTRAG_LESEN` |
| `GET` | `/api/v1/kfz/sf-transfers/{id}` | SFR-Transfer-Details | `VERTRAG_LESEN` |

### POST `/api/v1/kfz/sf-transfers/vorschau` – Transfer-Vorschau

**Request:**
```json
{
  "quellvertrag_id": "f6a7b8c9-d0e1-2345-fabc-456789012345",
  "zielvertrag_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "transfer_modus": "EINSEITIG",
  "sf_klassen_auswahl": "HP_UND_VK",
  "wirksamkeitsdatum": "2026-07-01"
}
```

**Response (200 OK):**
```json
{
  "quellvertrag": {
    "vertrag_id": "f6a7b8c9-d0e1-2345-fabc-456789012345",
    "vertragsnummer": "VN-2026-100001",
    "partner_name": "Max Mustermann",
    "fahrzeug": "MS-LV 1234 | VW Golf VIII",
    "sf_klasse_hp_aktuell": "SF12",
    "sf_klasse_vk_aktuell": "SF8",
    "sf_klasse_hp_nach_transfer": "SF½",
    "sf_klasse_vk_nach_transfer": "SF½",
    "jahresbeitrag_aktuell": "487.32",
    "jahresbeitrag_nach_transfer": "1124.50",
    "beitragsdifferenz": "+637.18"
  },
  "zielvertrag": {
    "vertrag_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "vertragsnummer": "VN-2026-100002",
    "partner_name": "Lisa Mustermann",
    "fahrzeug": "MS-LM 5678 | BMW 320i",
    "sf_klasse_hp_aktuell": "SF2",
    "sf_klasse_vk_aktuell": "SF1",
    "sf_klasse_hp_nach_transfer": "SF12",
    "sf_klasse_vk_nach_transfer": "SF8",
    "jahresbeitrag_aktuell": "1856.00",
    "jahresbeitrag_nach_transfer": "892.40",
    "beitragsdifferenz": "-963.60"
  },
  "gesamtbeitragsdifferenz": "-326.42",
  "wirksamkeitsdatum": "2026-07-01",
  "vier_augen_erforderlich": false,
  "warnungen": []
}
```

### POST `/api/v1/kfz/sf-transfers` – Transfer durchführen

**Request:**
```json
{
  "quellvertrag_id": "f6a7b8c9-d0e1-2345-fabc-456789012345",
  "zielvertrag_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "transfer_modus": "EINSEITIG",
  "sf_klassen_auswahl": "HP_UND_VK",
  "wirksamkeitsdatum": "2026-07-01",
  "transfer_grund": "FAMILIENTRANSFER",
  "nachweis_art": "HAUSHALTSBESCHEINIGUNG",
  "nachweis_dokument_id": "d1e2f3a4-b5c6-7890-defa-123456789abc",
  "bemerkung": "SF-Transfer von Ehemann auf Ehefrau, Haushaltsbescheinigung liegt vor."
}
```

**Response (200 OK – Transfer durchgeführt):**
```json
{
  "transfer_id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
  "transfer_nummer": "SFT-2026-000001",
  "status": "DURCHGEFUEHRT",
  "wirksamkeitsdatum": "2026-07-01",
  "quellvertrag": {
    "vertrag_id": "f6a7b8c9-d0e1-2345-fabc-456789012345",
    "vertragsnummer": "VN-2026-100001",
    "sf_klasse_hp_alt": "SF12",
    "sf_klasse_hp_neu": "SF½",
    "sf_klasse_vk_alt": "SF8",
    "sf_klasse_vk_neu": "SF½",
    "jahresbeitrag_alt": "487.32",
    "jahresbeitrag_neu": "1124.50",
    "vorgang": {
      "id": "d4e5f6a7-b8c9-0123-defa-234567890123",
      "vorgangsnummer": "VG-2026-000051",
      "vorgangstyp": "NACHTRAG"
    },
    "vertragsstand_version": 3,
    "druckauftrag_id": "uuid-druck-quell-001"
  },
  "zielvertrag": {
    "vertrag_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "vertragsnummer": "VN-2026-100002",
    "sf_klasse_hp_alt": "SF2",
    "sf_klasse_hp_neu": "SF12",
    "sf_klasse_vk_alt": "SF1",
    "sf_klasse_vk_neu": "SF8",
    "jahresbeitrag_alt": "1856.00",
    "jahresbeitrag_neu": "892.40",
    "vorgang": {
      "id": "e5f6a7b8-c9d0-1234-efab-345678901234",
      "vorgangsnummer": "VG-2026-000052",
      "vorgangstyp": "NACHTRAG"
    },
    "vertragsstand_version": 2,
    "druckauftrag_id": "uuid-druck-ziel-001"
  },
  "transfer_grund": "FAMILIENTRANSFER",
  "nachweis_art": "HAUSHALTSBESCHEINIGUNG",
  "erstellt_von": "id-schmidt",
  "erstellt_am": "2026-04-01T10:15:00Z"
}
```

**Response (200 OK – 4-Augen-Prüfung erforderlich):**
```json
{
  "transfer_id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
  "transfer_nummer": "SFT-2026-000002",
  "status": "GENEHMIGUNG_ERFORDERLICH",
  "wirksamkeitsdatum": "2026-07-01",
  "schwebe": {
    "id": "c3d4e5f6-a7b8-9012-cdef-123456789012",
    "schwebenummer": "SW-2026-000099",
    "status": "OFFEN",
    "aussteuerungsgrund": "SF-Transfer mit SF-Klasse ≥ SF20 erfordert 4-Augen-Prüfung",
    "zustaendiges_team": "ID-KFZ-SFR"
  }
}
```

**Response (409 Conflict – Transfer nicht möglich):**
```json
{
  "type": "https://api.versicherungsverwaltung.de/errors/sf-transfer-conflict",
  "title": "SF-Transfer nicht möglich",
  "status": 409,
  "detail": "Der Quellvertrag VN-2026-100001 hat einen offenen Schadenvorgang. Der SF-Transfer kann erst nach Abschluss der Schadenregulierung durchgeführt werden.",
  "errors": [
    {
      "code": "SFT-E03",
      "field": "quellvertrag_id",
      "message": "Offener Schadenvorgang auf dem Quellvertrag."
    }
  ]
}
```

## SFR-Transfer-Statusmodell

| Status | Beschreibung | Folgestatus |
|--------|-------------|-------------|
| `ENTWURF` | Transfer angelegt, noch nicht bestätigt | DURCHGEFUEHRT, GENEHMIGUNG_ERFORDERLICH, ABGEBROCHEN |
| `GENEHMIGUNG_ERFORDERLICH` | 4-Augen-Prüfung erforderlich (SF ≥ SF20) | GENEHMIGT, ABGELEHNT |
| `GENEHMIGT` | Transfer durch Teamleiter genehmigt | DURCHGEFUEHRT |
| `DURCHGEFUEHRT` | Transfer erfolgreich durchgeführt | – (Endzustand) |
| `ABGELEHNT` | Transfer durch Teamleiter abgelehnt | – (Endzustand) |
| `ABGEBROCHEN` | Transfer vom Sachbearbeiter abgebrochen | – (Endzustand) |
| `GEPLANT` | Transfer mit Wirksamkeit in der Zukunft geplant | DURCHGEFUEHRT, ABGEBROCHEN |

## Events

| Event | Auslöser | Consumer | Beschreibung |
|-------|---------|---------|-------------|
| `SfTransferAngefordertEvent` | POST /sf-transfers | Schwebe-Service (bei 4-Augen), Logging | SF-Transfer wurde angefordert |
| `SfTransferDurchgefuehrtEvent` | Phase 4, Schritt 10 | S1, S2, S5, SfKlassenHistorie | SF-Transfer wurde durchgeführt |
| `VertragGeaendertEvent` (×2) | Phase 4, Schritt 12 | S1, S2 | Beitrag beider Verträge geändert |

## SfKlassenHistorie-Einträge (Beispiel)

Nach einem einseitigen Transfer von VN-2026-100001 (SF12) → VN-2026-100002 (SF2):

**Quellvertrag (VN-2026-100001):**
```json
{
  "vertrag_id": "f6a7b8c9-d0e1-2345-fabc-456789012345",
  "sf_klasse_hp": "SF½",
  "sf_klasse_vk": "SF½",
  "gueltig_ab": "2026-07-01",
  "aenderungsgrund": "SF_TRANSFER",
  "sf_transfer_id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
  "sf_transfer_partner_vertrag_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "rabattschutz_eingesetzt": false
}
```

**Zielvertrag (VN-2026-100002):**
```json
{
  "vertrag_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "sf_klasse_hp": "SF12",
  "sf_klasse_vk": "SF8",
  "gueltig_ab": "2026-07-01",
  "aenderungsgrund": "SF_TRANSFER",
  "sf_transfer_id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
  "sf_transfer_partner_vertrag_id": "f6a7b8c9-d0e1-2345-fabc-456789012345",
  "rabattschutz_eingesetzt": false
}
```

## Nachbedingungen
- Die SF-Klassen beider Verträge sind gemäß dem Transfer-Modus aktualisiert
- Neue Vertragsstände mit den aktualisierten SF-Klassen sind für beide Verträge angelegt
- Die SfKlassenHistorie enthält für beide Verträge einen Eintrag mit Änderungsgrund `SF_TRANSFER` und gegenseitigem Querverweis
- Die Beiträge beider Verträge sind neu berechnet und an S2 (Inkasso) übermittelt
- Nachtragsdokumente sind für beide Verträge über S5 (Druck) erzeugt
- Die Änderungen sind revisionssicher historisiert (Hibernate Envers)
- Bei 4-Augen-Prüfung: Schwebe mit Aussteuerungsgrund ist angelegt

## Akzeptanzkriterien
- [ ] Der SF-Transfer kann aus der Vertragsdetail-Ansicht für aktive KFZ-Verträge gestartet werden
- [ ] Quell- und Zielvertrag können per Vertragsnummer oder Vertragssuche ausgewählt werden
- [ ] Die Vorschau zeigt korrekt die SF-Klassen und Beitragsauswirkungen für beide Verträge an
- [ ] Bei einseitigem Transfer wird der Quellvertrag auf SF½ zurückgesetzt
- [ ] Bei Tausch werden die SF-Klassen beider Verträge korrekt getauscht
- [ ] Nur HP-SF, nur VK-SF oder beide SF-Klassen können einzeln für den Transfer ausgewählt werden
- [ ] Der Transfergrund und Berechtigungsnachweis sind Pflichtfelder
- [ ] Bei SF-Klasse ≥ SF20 wird automatisch eine 4-Augen-Prüfung ausgelöst (Schwebe)
- [ ] Für beide Verträge werden neue Vertragsstände, Vorgänge und SfKlassenHistorie-Einträge erzeugt
- [ ] Die Beitragsneuberechnung wird für beide Verträge korrekt durchgeführt
- [ ] Nachtragsdokumente werden für beide Verträge über S5 erzeugt
- [ ] Offene Schäden und laufende VWB-Vorgänge blockieren den Transfer (mit Sachbearbeiter-Override für Schäden)
- [ ] Ein Transfer auf denselben Vertrag wird vom System verhindert
- [ ] Transfers zwischen verschiedenen VN sind nur mit Berechtigungsnachweis möglich
- [ ] Der Transfer wird vollständig in der Audit-Historie (Envers) protokolliert mit Querverweisen

## Wireframe / Skizze

### Transfer-Übersicht (Vorschau)

```
┌──────────────────────────────────────────────────────────────────────────┐
│  SF-Klasse übertragen (SFR-Transfer)                                    │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Transfer-Modus:  ● Einseitig (Quell → SF½)   ○ Tausch (SF ↔ SF)      │
│  SF-Klassen:      ● HP + VK   ○ Nur HP   ○ Nur VK                      │
│  Wirksamkeit:     [ 01.07.2026       📅 ]                               │
│                                                                          │
│  ┌─────────────────────────────┐    ──►    ┌─────────────────────────┐  │
│  │  📋 QUELLVERTRAG            │           │  📋 ZIELVERTRAG          │  │
│  │                             │           │                         │  │
│  │  VN-2026-100001             │           │  VN-2026-100002         │  │
│  │  Max Mustermann             │           │  Lisa Mustermann        │  │
│  │  MS-LV 1234 | VW Golf VIII │           │  MS-LM 5678 | BMW 320i │  │
│  │                             │           │                         │  │
│  │  SF-HP:  SF12 → SF½  ⬇️     │           │  SF-HP:  SF2 → SF12 ⬆️  │  │
│  │  SF-VK:  SF8  → SF½  ⬇️     │           │  SF-VK:  SF1 → SF8  ⬆️  │  │
│  │                             │           │                         │  │
│  │  Beitrag: 487,32 €         │           │  Beitrag: 1.856,00 €   │  │
│  │       → 1.124,50 €         │           │       →   892,40 €     │  │
│  │  Diff: +637,18 € ▲        │           │  Diff: -963,60 € ▼    │  │
│  └─────────────────────────────┘           └─────────────────────────┘  │
│                                                                          │
│  ┌── Gesamtauswirkung ─────────────────────────────────────────────────┐ │
│  │  Beitragsdifferenz gesamt: -326,42 € / Jahr (Ersparnis)            │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  Transfergrund*:   [ Familientransfer                          ▾ ]      │
│  Nachweis*:        [ Haushaltsbescheinigung                    ▾ ]      │
│  Dokument:         [ 📎 Haushaltsbescheinigung_2026.pdf   ]  [Upload]   │
│  Bemerkung:        [ SF-Transfer von Ehemann auf Ehefrau,          ]    │
│                    [ Haushaltsbescheinigung liegt vor.              ]    │
│                                                                          │
│                          [ Abbrechen ]  [ Transfer durchführen ]        │
└──────────────────────────────────────────────────────────────────────────┘
```

### Bestätigungsdialog

```
┌───────────────────────────────────────────────────────────┐
│  ⚠️  SF-Transfer bestätigen                               │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  Quellvertrag:  VN-2026-100001 (Max Mustermann)           │
│    SF-HP: SF12 → SF½  |  SF-VK: SF8 → SF½                │
│    Neuer Beitrag: 1.124,50 € (+637,18 €)                 │
│                                                           │
│  Zielvertrag:   VN-2026-100002 (Lisa Mustermann)          │
│    SF-HP: SF2 → SF12  |  SF-VK: SF1 → SF8                │
│    Neuer Beitrag: 892,40 € (-963,60 €)                   │
│                                                           │
│  Wirksamkeit:   01.07.2026                                │
│  Grund:         Familientransfer                          │
│                                                           │
│  Beide Versicherungsnehmer erhalten                       │
│  ein Nachtragsdokument.                                   │
│                                                           │
│  Diese Aktion kann nicht automatisch                      │
│  rückgängig gemacht werden.                               │
│                                                           │
│            [ Abbrechen ]  [ Transfer bestätigen ]         │
└───────────────────────────────────────────────────────────┘
```

## Sequenzdiagramm

```
Sachbearbeiter     Frontend        SF-Transfer-API     Vertrag-Svc     SfHistorie-Svc     S2        S5
     │                │                  │                  │                │             │         │
     │  SF-Transfer   │                  │                  │                │             │         │
     │  starten       │                  │                  │                │             │         │
     │───────────────►│                  │                  │                │             │         │
     │                │  POST /vorschau  │                  │                │             │         │
     │                │─────────────────►│                  │                │             │         │
     │                │                  │  Verträge laden  │                │             │         │
     │                │                  │─────────────────►│                │             │         │
     │                │                  │  SF + Beiträge   │                │             │         │
     │                │                  │◄─────────────────│                │             │         │
     │                │  Vorschau-Daten  │                  │                │             │         │
     │                │◄─────────────────│                  │                │             │         │
     │  Prüft + Best. │                  │                  │                │             │         │
     │───────────────►│                  │                  │                │             │         │
     │                │  POST /transfers │                  │                │             │         │
     │                │─────────────────►│                  │                │             │         │
     │                │                  │  Quell-VT ändern │                │             │         │
     │                │                  │─────────────────►│                │             │         │
     │                │                  │  Ziel-VT ändern  │                │             │         │
     │                │                  │─────────────────►│                │             │         │
     │                │                  │  SF-Historie (×2)│                │             │         │
     │                │                  │─────────────────────────────────►│             │         │
     │                │                  │  Beitragsänderung (×2)           │             │         │
     │                │                  │──────────────────────────────────────────────►│         │
     │                │                  │  Nachtragsdokument (×2)                       │         │
     │                │                  │─────────────────────────────────────────────────────────►│
     │                │  Transfer-Ergebnis│                 │                │             │         │
     │                │◄─────────────────│                  │                │             │         │
     │  Erfolg ✅      │                  │                  │                │             │         │
     │◄───────────────│                  │                  │                │             │         │
```

## Offene Fragen

| Nr. | Frage | Kontext |
|-----|-------|---------|
| UC-KFZ-21-01 | Soll ein SF-Transfer auch rückwirkend möglich sein (z. B. zum letzten Fälligkeitstermin)? | Rückwirkende Beitragskorrektur erhöht Komplexität |
| UC-KFZ-21-02 | Darf ein SF-Transfer zu einem Vertrag eines anderen Versicherers erfolgen (z. B. über GDV-VWB)? | Aktuell nur intern innerhalb des LVM-Bestands |
| UC-KFZ-21-03 | Soll die Adressprüfung (gleicher Haushalt) automatisch über das Partner-System (S7) erfolgen oder genügt die manuelle Prüfung durch den Sachbearbeiter? | Automatisierungsgrad vs. Aufwand |
| UC-KFZ-21-04 | Gibt es eine maximale SF-Klassen-Differenz, ab der ein Transfer auffällig wird (Betrugsprävention)? | Compliance / Fraud Detection |
| UC-KFZ-21-05 | Soll ein Transfer auch für Verträge mit Versicherungskennzeichen (FA-04) möglich sein? Diese haben keine SF-Klasse. | Abgrenzung Fahrzeugarten |
| UC-KFZ-21-06 | Wie wird der Transfer behandelt, wenn der Quellvertrag einen RabattSchutz hat und nach Rücksetzung auf SF½ die Voraussetzung (SF ≥ 4) nicht mehr erfüllt? Fällt der RabattSchutz automatisch weg? | Produkt-Konsistenz |
