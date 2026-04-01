# UC-08: Beitragsrechnung durchführen (Inkasso)

## Beschreibung
Das System führt die **Beitragsrechnung (Inkasso)** für alle aktiven Verträge durch. Dabei wird zwischen **Hauptinkasso** zur Hauptfälligkeit und **Nebeninkasso** (Teilfälligkeiten abhängig von der Zahlungsweise) unterschieden. Jedes Inkasso erzeugt einen **neuen Vertragsstand** am bestehenden Vertrag und bestückt die nachgelagerten Systeme (DataWarehouse S4, Provision S1, Konzerninkasso S2, Druck S5).

Das **Hauptinkasso** findet einmal jährlich zum **Hauptfälligkeitstermin** (HFT) des Vertrags statt. Hierbei wird der Beitrag für das neue Versicherungsjahr berechnet, inklusive etwaiger Tarifanpassungen, SF-Klassen-Fortschreibung (KFZ) oder Beitragsanpassungsklauseln. Das Ergebnis ist ein neuer, policierter Vertragsstand.

Das **Nebeninkasso** findet bei unterjährigen Zahlungsweisen (`HALBJAEHRLICH`, `VIERTELJAEHRLICH`, `MONATLICH`) zu den jeweiligen Teilfälligkeiten statt. Es erzeugt eine **Inkassoforderung** ohne Beitragsneuberechnung – der Ratenbeitrag basiert auf dem zuletzt policierten Jahresbeitrag.

Dies ist ein **spartenübergreifender** Kernprozess. Spartenspezifische Besonderheiten (z. B. SF-Klassen-Fortschreibung bei KFZ, Beitragsanpassung nach GDV-Empfehlung) werden über die Extension Points / Hooks der Spartenkonfiguration eingebunden.

## Akteure
- **Primär:** System (automatischer Batchlauf)
- **Sekundär:** Innendienst (Sachbearbeitung bei Aussteuerung oder manueller Auslösung), Fachbereich (Konfiguration der Inkasso-Parameter)

## Vorbedingungen
- Mindestens ein Vertrag im Status **`AKTIV`** mit erreichtem Fälligkeitstermin existiert
- Die aktuelle **Produktkonfiguration** (UC-03) ist gültig und freigegeben
- Die **Zahlungsweise** des Vertrags ist konfiguriert (`JAEHRLICH`, `HALBJAEHRLICH`, `VIERTELJAEHRLICH`, `MONATLICH`)
- Das **Konzerninkasso (S2)** ist erreichbar (oder Transactional Outbox ist aktiv)

## Auslöser
- **Hauptinkasso:** Zeitgesteuerter Batch-Job zum konfigurierbaren Vorlaufzeitpunkt vor der Hauptfälligkeit (z. B. 30 Tage vorher)
- **Nebeninkasso:** Zeitgesteuerter Batch-Job zum konfigurierbaren Vorlaufzeitpunkt vor der Teilfälligkeit
- **Manuell:** Innendienst löst ein Einzelvertrag-Inkasso manuell aus (z. B. nach Korrektur, Wiederinkraftsetzung)

## Inkassotypen

| Inkassotyp | Bezeichnung | Auslöser | Beitragsneuberechnung | Neuer Vertragsstand | Zahlungsweisen |
|-----------|-------------|----------|----------------------|--------------------:|----------------|
| `HAUPTINKASSO` | Hauptfälligkeits-Inkasso | Batch zum HFT-Vorlauf | ✅ Ja (Neuberechnung) | ✅ Ja (Version n+1) | Alle |
| `NEBENINKASSO_RATE` | Nebeninkasso (Ratenforderung) | Batch zur Teilfälligkeit | ❌ Nein (basiert auf Jahresbeitrag) | ❌ Nein (nur Forderung) | HALBJAEHRLICH, VIERTELJAEHRLICH, MONATLICH |
| `NACHBERECHNUNG` | Nachberechnung (unterjährig) | Policierung (UC-05, UC-06, UC-07) | ✅ Ja (pro rata) | ✅ Ja (eigener Vertragsstand) | Alle |
| `MANUELL` | Manuelles Inkasso (Einzelvertrag) | Innendienst-Aktion | ✅ Ja (Neuberechnung) | ✅ Ja (Version n+1) | Alle |

## Fälligkeitsschema je Zahlungsweise

| Zahlungsweise | Aufschlag | Fälligkeiten pro Jahr | Fälligkeitsmuster (bei HFT 01.01.) | Ratenbeitrag |
|---------------|-----------|----------------------|-------------------------------------|-------------|
| `JAEHRLICH` | 0 % | 1 (nur Hauptinkasso) | 01.01. | Jahresbeitrag × 1,00 |
| `HALBJAEHRLICH` | ~3 % | 2 (1× Haupt + 1× Neben) | 01.01., 01.07. | Jahresbeitrag × 1,03 / 2 |
| `VIERTELJAEHRLICH` | ~5 % | 4 (1× Haupt + 3× Neben) | 01.01., 01.04., 01.07., 01.10. | Jahresbeitrag × 1,05 / 4 |
| `MONATLICH` | ~5 % | 12 (1× Haupt + 11× Neben) | 01.01., 01.02., …, 01.12. | Jahresbeitrag × 1,05 / 12 |

## Ablauf (Hauptszenario – Hauptinkasso)

### Phase 1: Selektion der fälligen Verträge
1. Der **Batch-Job** wird zeitgesteuert gestartet (z. B. täglich um 02:00 Uhr)
2. System selektiert alle Verträge im Status `AKTIV`, deren **Hauptfälligkeitstermin** innerhalb des konfigurierbaren Vorlaufzeitraums liegt:
   - `hauptfaelligkeit − vorlauf_tage ≤ heute` UND `hauptfaelligkeit > heute − toleranz_tage`
   - Filter: Kein offenes Inkasso für denselben HFT vorhanden
3. System erstellt einen **Inkassolauf** (Batch-Protokoll) mit:
   - Lauf-ID, Inkassotyp `HAUPTINKASSO`, Startzeit, Anzahl selektierter Verträge
4. System verarbeitet jeden selektierten Vertrag einzeln (Fehler-Isolation: ein fehlerhafter Vertrag blockiert nicht den gesamten Lauf)

### Phase 2: Beitragsneuberechnung (pro Vertrag)
5. → **Hook `vor_Hauptinkasso`** wird ausgelöst (spartenspezifische Vorbereitungen, z. B. SF-Klassen-Fortschreibung bei KFZ, Beitragsanpassung bei Lebensversicherung)
6. System liest den **aktuellen Vertragsstand** (Version n, `gueltig_bis = NULL`)
7. System berechnet den **neuen Jahresbeitrag** für das kommende Versicherungsjahr:
   - Aktuelle Produktkonfiguration und Tarifmerkmale anwenden (UC-03)
   - Spartenspezifische Faktoren berücksichtigen (über Hook, z. B. neue SF-Klasse)
   - Beitragsanpassungsklauseln prüfen (konfigurierbar pro Sparte)
   - Versicherungssteuer (GR-P02) aufschlagen
   - Zahlungsweise-Aufschlag berechnen
8. System ermittelt die **Beitragsdifferenz** (alter Jahresbeitrag vs. neuer Jahresbeitrag):
   - Beitragssteigerung → ggf. Sonderkündigungsrecht für VN (→ UC-06)
   - Beitragssenkung → Information an VN
   - Keine Änderung → Standardmäßig fortführen
9. System führt **Plausibilitätsprüfungen** durch:
   - Beitrag > 0?
   - Beitragsänderung innerhalb der konfigurierbaren Schwellenwerte? (z. B. max. ±30 %)
   - Alle Produkte noch gültig in der aktuellen Konfiguration?
10. Plausibilitätsprüfung bestanden → weiter mit Phase 3
    Plausibilitätsprüfung nicht bestanden → **Aussteuerung** an den Innendienst

### Phase 3: Vertragsstand-Erzeugung (Policierung)
11. System schließt den aktuellen Vertragsstand: `gueltig_bis` = HFT − 1 Tag
12. System erzeugt einen **neuen Vertragsstand** (Version n+1):
    - `gueltig_ab` = HFT (Hauptfälligkeitstermin)
    - `gueltig_bis` = NULL (aktuell gültig)
    - `jahresbeitrag` = neu berechneter Jahresbeitrag
    - `zahlungsweise` = unverändert (oder geändert, falls über UC-05 gewünscht)
    - Vertragsdaten = Daten des Vorgänger-Vertragsstands (ggf. angepasst durch Spartenhook)
13. System erzeugt einen **Vorgang** mit Vorgangstyp `HAUPTINKASSO`
14. System aktualisiert den **Vertrag**:
    - `aktueller_jahresbeitrag` = neuer Jahresbeitrag
    - `hauptfaelligkeit` = nächster HFT (HFT + 12 Monate bei Jahresverträgen)
15. → **Hook `nach_Hauptinkasso`** wird ausgelöst (spartenspezifische Nachverarbeitung)

### Phase 4: Zahlungsplan erstellen
16. System erstellt den **Zahlungsplan** für das neue Versicherungsjahr:
    - Bei `JAEHRLICH`: 1 Forderung zum HFT
    - Bei `HALBJAEHRLICH`: 2 Forderungen (HFT, HFT + 6 Monate)
    - Bei `VIERTELJAEHRLICH`: 4 Forderungen (HFT, HFT + 3/6/9 Monate)
    - Bei `MONATLICH`: 12 Forderungen (HFT, HFT + 1/2/.../11 Monate)
17. Jede Forderung enthält:
    - Fälligkeitsdatum
    - Ratenbeitrag (Jahresbeitrag × Zahlungsweise-Faktor / Anzahl Raten)
    - Forderungstyp (`HAUPTINKASSO` für die erste Rate, `NEBENINKASSO_RATE` für Folgeraten)
    - Status `OFFEN`

### Phase 5: Nachgelagerte Systeme versorgen
18. System publiziert **`HauptinkassoDurchgefuehrtEvent`** (Spring Event) → löst folgende Schnittstellenaufrufe aus:

| Schnittstelle | Aktion | Payload-Kern | Mechanismus |
|---------------|--------|-------------|-------------|
| **S4 – DataWarehouse** | Vertragsbewegung `HAUPTINKASSO` melden | Vertragsnummer, Sparte, alter Beitrag, neuer Beitrag, HFT, Beitragsdifferenz | Kafka-Topic `vertrag.datawarehouse.bewegungen` |
| **S1 – Provision** | Beitragsereignis melden (Bestandspflege-Provision) | Vertragsnummer, Sparte, Jahresbeitrag, Vermittler-ID, Provisionsanspruch | Kafka-Topic `vertrag.provision.ereignisse` (Ereignistyp `HAUPTINKASSO`) |
| **S2 – Konzerninkasso** | Neuer Zahlungsplan + erste Beitragsforderung | Vertragsnummer, Jahresbeitrag, Zahlungsweise, Zahlungsplan (alle Fälligkeiten), Partner-Bankverbindung | Kafka-Topic `vertrag.inkasso.beitragsforderungen` |
| **S5 – Druck** | Beitragsrechnung drucken | Vertragsnummer, Partner, neuer Beitrag, alter Beitrag, Differenz, Zahlungsplan, Dokumenttyp `BEITRAGSRECHNUNG` | REST `POST /api/v1/druck/auftraege` |

19. System protokolliert das Ergebnis im **Inkassolauf** (Erfolg/Fehler pro Vertrag)
20. → Vertrag ist für das neue Versicherungsjahr policiert

### Phase 6: Batch-Abschluss
21. System erstellt eine **Zusammenfassung** des Inkassolaufs:
    - Anzahl verarbeiteter Verträge
    - Davon erfolgreich / ausgesteuert / fehlerhaft
    - Summe Gesamtbeitrag (alt vs. neu)
    - Durchschnittliche Beitragsänderung
22. System publiziert **`InkassolaufAbgeschlossenEvent`** (Spring Event) → Monitoring, Durchlaufzeiten (UC-04)

## Ablauf (Nebeninkasso – Ratenforderung)

### Nebeninkasso-Verarbeitung
1. Der **Batch-Job** selektiert alle Verträge mit Zahlungsweise ≠ `JAEHRLICH`, deren nächste **Teilfälligkeit** im Vorlaufzeitraum liegt
2. System prüft pro Vertrag:
   - Status = `AKTIV`?
   - Zahlungsplan für das laufende Versicherungsjahr existiert?
   - Forderung für diese Teilfälligkeit noch nicht erzeugt?
3. System erstellt die **Nebeninkasso-Forderung** basierend auf dem bestehenden Zahlungsplan:
   - Ratenbeitrag = Jahresbeitrag × Zahlungsweise-Faktor / Anzahl Raten
   - Fälligkeitsdatum = Teilfälligkeitstermin
   - Forderungstyp = `NEBENINKASSO_RATE`
   - Status = `OFFEN`
4. **Kein neuer Vertragsstand** wird erzeugt (Beitrag unverändert)
5. System publiziert **`NebeninkassoForderungEvent`** (Spring Event):

| Schnittstelle | Aktion | Payload-Kern | Mechanismus |
|---------------|--------|-------------|-------------|
| **S2 – Konzerninkasso** | Ratenforderung stellen | Vertragsnummer, Fälligkeitsdatum, Ratenbeitrag, Forderungstyp | Kafka-Topic `vertrag.inkasso.beitragsforderungen` |
| **S4 – DataWarehouse** | Nebeninkasso-Ereignis melden | Vertragsnummer, Fälligkeitsdatum, Ratenbeitrag | Kafka-Topic `vertrag.datawarehouse.bewegungen` |

6. **S1 (Provision)** und **S5 (Druck)** werden beim Nebeninkasso **nicht** angesprochen (kein neuer Vertragsstand, keine Beitragsrechnung)

## Alternativszenarien

- **A1: Beitragsanpassung mit Sonderkündigungsrecht**
  Die Beitragsneuberechnung ergibt eine Beitragssteigerung. System kennzeichnet den Vertragsstand mit dem Flag `beitragsanpassung = true`. Bei der Beitragsrechnung (Druckauftrag S5) wird der VN auf sein Sonderkündigungsrecht hingewiesen (→ UC-06, Grund `SONDERKUENDIGUNG_BEITRAG`).

- **A2: Aussteuerung bei Plausibilitätsverletzung**
  Die Beitragsneuberechnung überschreitet den konfigurierten Schwellenwert (z. B. > 30 % Änderung) oder ein Produkt ist entfallen. System erzeugt eine **Inkasso-Schwebe** für den Innendienst. Sachbearbeiter prüft manuell und entscheidet: policieren, anpassen oder zurückstellen.

- **A3: Manuelles Einzelvertrag-Inkasso**
  Der Innendienst löst ein Inkasso für einen einzelnen Vertrag aus (z. B. nach Wiederinkraftsetzung UC-07 oder nach Korrektur). System führt Phase 2–5 für diesen Vertrag durch. Kein Batch-Protokoll, sondern direkte Rückmeldung an den Benutzer.

- **A4: Vertrag mit unterjähriger Änderung**
  Der Vertrag wurde unterjährig geändert (UC-05). Das Hauptinkasso zum HFT berücksichtigt den zuletzt policierten Vertragsstand (inkl. der Änderung) als Basis für die Neuberechnung.

- **A5: Zahlungsweise-Wechsel zum HFT**
  Der VN hat einen Zahlungsweisenwechsel beantragt (UC-05, Änderungstyp `ZAHLUNGSWEISE_AENDERUNG`), der zum HFT wirksam wird. Das Hauptinkasso erzeugt den Zahlungsplan mit der neuen Zahlungsweise.

- **A6: Vertrag mit Beitragsfreistellung**
  Bestimmte Verträge können beitragsfrei gestellt sein (z. B. Ruheversicherung bei KFZ). System erkennt die Beitragsfreistellung und erzeugt eine Forderung mit Betrag 0,00 €. Der Vertragsstand wird trotzdem fortgeschrieben.

- **A7: Stornierung während des Inkassolaufs**
  Ein Vertrag wird zwischen Selektion und Verarbeitung storniert (UC-06). System prüft den Vertragsstatus vor der Policierung erneut. Ist der Vertrag nicht mehr `AKTIV`, wird er übersprungen und im Laufprotokoll als „übersprungen – Vertrag nicht aktiv" dokumentiert.

- **A8: Wiederholungslauf (Retry)**
  Bei einzelnen Fehlern im Batch wird ein Wiederholungslauf ausgelöst, der nur die fehlerhaften Verträge erneut verarbeitet. Die bereits erfolgreich verarbeiteten Verträge werden nicht erneut verarbeitet (Idempotenz über Inkassolauf-ID + Vertrag-ID).

## Fehlerfälle

- **F1: Produktkonfiguration nicht gültig**
  Die aktuelle Produktkonfiguration (UC-03) ist abgelaufen oder nicht für das kommende Versicherungsjahr freigegeben. → System bricht den gesamten Inkassolauf ab und meldet: „Produktkonfiguration für Sparte [Sparte] nicht gültig für [HFT]."

- **F2: Beitragsberechnung fehlgeschlagen**
  Die Berechnung schlägt für einen einzelnen Vertrag fehl (z. B. fehlende Tarifmerkmale). → System markiert den Vertrag als `FEHLERHAFT` im Laufprotokoll und setzt mit dem nächsten Vertrag fort. Innendienst wird per Wiedervorlage informiert.

- **F3: Beitragsänderung überschreitet Schwellenwert**
  Die Beitragsänderung überschreitet den konfigurierbaren Schwellenwert. → Aussteuerung an den Innendienst (→ A2). Vertrag wird nicht automatisch policiert.

- **F4: Schnittstellenfehler bei Policierung**
  Eine oder mehrere nachgelagerte Schnittstellen sind nicht erreichbar. → Vertragsstand wird trotzdem erzeugt (→ GR-A11). Transactional Outbox Pattern und Dead-Letter-Topics greifen.

- **F5: Vertragsstand-Erzeugung fehlgeschlagen**
  Die Datenbank-Transaktion für den neuen Vertragsstand schlägt fehl (z. B. Constraint-Verletzung). → System protokolliert den Fehler und markiert den Vertrag im Laufprotokoll. Kein Teilzustand möglich (Transaktion = atomar).

- **F6: Batch-Job-Zeitüberschreitung**
  Der Batch-Job überschreitet das konfigurierbare Zeitfenster. → System bricht den Lauf kontrolliert ab, protokolliert den Fortschritt und plant einen Wiederholungslauf für die nicht verarbeiteten Verträge.

- **F7: Doppelte Verarbeitung (Idempotenzprüfung)**
  Ein Vertrag wurde bereits für denselben HFT verarbeitet (z. B. bei Wiederholungslauf). → System erkennt den vorhandenen Vertragsstand und überspringt den Vertrag.

- **F8: Zahlungsplan-Erzeugung fehlgeschlagen**
  Der Zahlungsplan kann nicht erstellt werden (z. B. ungültige Zahlungsweise-Konfiguration). → System erzeugt den Vertragsstand trotzdem, markiert aber den Zahlungsplan als `FEHLERHAFT`. Innendienst wird informiert.

## Geschäftsregeln

| Regel-ID | Beschreibung | Quelle |
|----------|-------------|--------|
| GR-V02 | Ein Vertragsstand gehört immer zu genau einem Vorgang – hier Vorgangstyp `HAUPTINKASSO` | UC-02, UC-08 |
| GR-V05 | Bei jeder Policierung wird die Vertragsbewegung an das DataWarehouse (S4) gemeldet | UC-05, UC-08 |
| GR-P01 | Die Prämienberechnung erfolgt über das konfigurierbare Scoring-Faktor-Modell | UC-03, UC-08 |
| GR-P02 | Versicherungssteuer (19 %) auf Nettobeitrag + Zahlungsweise-Aufschlag auf Bruttobeitrag | UC-01, UC-08 |
| GR-A08 | Bei der Policierung (Hauptinkasso) werden alle vier Schnittstellen bestückt: S4, S1, S2, S5 | UC-05, UC-08 |
| GR-A09 | Die Policierung erzeugt genau einen Vertragsstand, der genau einem Vorgang zugeordnet ist | UC-02, UC-08 |
| GR-A10 | Beim Hauptinkasso wird der bestehende Vertrag fortgeführt (neuer Vertragsstand am selben Vertrag) | UC-05, UC-08 |
| GR-A11 | Schnittstellenfehler führen nicht zum Abbruch – Retry-Mechanismen liefern nach | UC-05, UC-08 |
| GR-IN-01 | Das **Hauptinkasso** wird zum Hauptfälligkeitstermin durchgeführt und erzeugt einen neuen Vertragsstand mit neuberechnetem Beitrag | UC-08 |
| GR-IN-02 | Das **Nebeninkasso** erzeugt nur eine Teilforderung auf Basis des bestehenden Jahresbeitrags – **kein** neuer Vertragsstand | UC-08 |
| GR-IN-03 | Der **Zahlungsplan** wird beim Hauptinkasso für das gesamte Versicherungsjahr erstellt und enthält alle Fälligkeiten (Haupt- und Nebenfälligkeiten) | UC-08 |
| GR-IN-04 | Der **Ratenbeitrag** ergibt sich aus: Jahresbeitrag × Zahlungsweise-Faktor / Anzahl Raten. Rundungsdifferenzen werden auf die letzte Rate aufgeschlagen | UC-08 |
| GR-IN-05 | Beitragsänderungen > konfigurierbarer Schwellenwert (Standard: 30 %) werden **ausgesteuert** und nicht automatisch policiert | UC-08 |
| GR-IN-06 | Pro Vertrag und Hauptfälligkeitstermin darf maximal ein Hauptinkasso durchgeführt werden (**Idempotenz**) | UC-08 |
| GR-IN-07 | Bei Beitragssteigerung wird der VN auf sein **Sonderkündigungsrecht** hingewiesen (Kennzeichnung im Druckauftrag an S5) | UC-08, UC-06 |
| GR-IN-08 | Der Batch-Job verarbeitet jeden Vertrag **isoliert** – ein Fehler bei einem Vertrag führt nicht zum Abbruch des gesamten Laufs | UC-08 |
| GR-IN-09 | Nebeninkasso-Forderungen werden nur an **S2 (Inkasso)** und **S4 (DWH)** gemeldet – S1 (Provision) und S5 (Druck) werden **nicht** angesprochen | UC-08 |
| GR-IN-10 | Die HFT-Aktualisierung erfolgt beim Hauptinkasso automatisch: `hauptfaelligkeit` = alter HFT + Vertragslaufzeit | UC-08 |
| GR-IN-11 | Bei **beitragsfreigestellten Verträgen** (z. B. Ruheversicherung) wird die Forderung mit Betrag 0,00 € erzeugt und der Vertragsstand trotzdem fortgeschrieben | UC-08 |
| GR-IN-12 | Der **Vorlaufzeitraum** für die Selektion ist konfigurierbar (Standard: 30 Tage vor HFT für Hauptinkasso, 14 Tage vor Teilfälligkeit für Nebeninkasso) | UC-08 |

## Daten (Ein-/Ausgabe)

### Eingabedaten (Batch-Parameter)

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| inkassotyp | Enum | ✅ | HAUPTINKASSO, NEBENINKASSO_RATE, MANUELL | Art des Inkassolaufs |
| stichtag | Date | ✅ | ≥ heute | Fälligkeitsstichtag für die Selektion |
| sparte | Enum | ❌ | Gültige Sparte oder NULL für alle | Einschränkung auf eine Sparte (optional) |
| vertrag_id | UUID | ❌ | Existierender Vertrag im Status AKTIV | Nur bei manuellem Inkasso (Einzelvertrag) |
| vorlauf_tage | Integer | ❌ | 1–90, Standard: 30 (Haupt) / 14 (Neben) | Vorlaufzeitraum in Tagen |
| schwellenwert_prozent | BigDecimal(5,2) | ❌ | 0.01–100.00, Standard: 30.00 | Aussteuerungs-Schwellenwert für Beitragsänderung |

### Ausgabedaten (pro Vertrag)

| Feld | Typ | Beschreibung |
|------|-----|-------------|
| vertrag_id | UUID | Verarbeiteter Vertrag |
| inkasso_ergebnis | Enum | ERFOLGREICH, AUSGESTEUERT, FEHLERHAFT, UEBERSPRUNGEN |
| vertragsstand_id | UUID | ID des neuen Vertragsstands (nur bei Hauptinkasso) |
| vorgang_id | UUID | ID des erzeugten Vorgangs (nur bei Hauptinkasso) |
| alter_jahresbeitrag | BigDecimal(12,2) | Bisheriger Jahresbeitrag |
| neuer_jahresbeitrag | BigDecimal(12,2) | Neu berechneter Jahresbeitrag (nur Hauptinkasso) |
| beitragsdifferenz | BigDecimal(12,2) | Differenz (positiv = Steigerung) |
| beitragsdifferenz_prozent | BigDecimal(5,2) | Differenz in Prozent |
| zahlungsplan | List | Zahlungsplan mit Fälligkeiten und Raten |
| beitragsanpassung | Boolean | Wurde der Beitrag geändert? (Sonderkündigungsrecht-Hinweis) |
| aussteuerungsgrund | String | Grund der Aussteuerung (falls zutreffend) |
| fehler_details | String | Fehlerbeschreibung (falls fehlerhaft) |

### Ausgabedaten (Inkassolauf-Zusammenfassung)

| Feld | Typ | Beschreibung |
|------|-----|-------------|
| inkassolauf_id | UUID | ID des Inkassolaufs |
| inkassotyp | Enum | HAUPTINKASSO oder NEBENINKASSO_RATE |
| startzeit | Timestamp | Startzeitpunkt des Batchlaufs |
| endzeit | Timestamp | Endzeitpunkt des Batchlaufs |
| anzahl_selektiert | Integer | Anzahl selektierter Verträge |
| anzahl_erfolgreich | Integer | Erfolgreich verarbeitete Verträge |
| anzahl_ausgesteuert | Integer | An den Innendienst ausgesteuerte Verträge |
| anzahl_fehlerhaft | Integer | Fehlerhaft verarbeitete Verträge |
| anzahl_uebersprungen | Integer | Übersprungene Verträge (z. B. bereits verarbeitet) |
| gesamtbeitrag_alt | BigDecimal(14,2) | Summe aller alten Jahresbeiträge |
| gesamtbeitrag_neu | BigDecimal(14,2) | Summe aller neuen Jahresbeiträge |
| durchschnittliche_aenderung_prozent | BigDecimal(5,2) | Durchschnittliche Beitragsänderung in % |

## API-Endpunkte

### Inkassolauf starten und überwachen

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| POST | `/api/v1/inkasso/laeufe` | `INKASSO_AUSFUEHREN` | Inkassolauf manuell starten (Batch oder Einzelvertrag) |
| GET | `/api/v1/inkasso/laeufe` | `INKASSO_LESEN` | Inkassoläufe auflisten (Filter: Datum, Typ, Status) |
| GET | `/api/v1/inkasso/laeufe/{id}` | `INKASSO_LESEN` | Inkassolauf-Details inkl. Zusammenfassung |
| GET | `/api/v1/inkasso/laeufe/{id}/ergebnisse` | `INKASSO_LESEN` | Einzelergebnisse pro Vertrag im Inkassolauf |

### Einzelvertrag-Inkasso

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| POST | `/api/v1/vertraege/{vertrag_id}/inkasso` | `INKASSO_AUSFUEHREN` | Manuelles Inkasso für einen einzelnen Vertrag |
| GET | `/api/v1/vertraege/{vertrag_id}/inkasso` | `INKASSO_LESEN` | Inkasso-Historie des Vertrags |
| GET | `/api/v1/vertraege/{vertrag_id}/inkasso/{id}` | `INKASSO_LESEN` | Inkasso-Detail (Beitragsneuberechnung, Zahlungsplan) |

### Zahlungsplan

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| GET | `/api/v1/vertraege/{vertrag_id}/zahlungsplan` | `VERTRAG_LESEN` | Aktuellen Zahlungsplan des Vertrags anzeigen |
| GET | `/api/v1/vertraege/{vertrag_id}/zahlungsplan/forderungen` | `INKASSO_LESEN` | Einzelforderungen (offen, bezahlt, storniert) |

### Inkasso-Schweben (Aussteuerung)

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| GET | `/api/v1/inkasso/schweben` | `INKASSO_LESEN` | Offene Inkasso-Schweben auflisten |
| GET | `/api/v1/inkasso/schweben/{id}` | `INKASSO_LESEN` | Inkasso-Schwebe-Details (Vertrag, Beitragsvergleich, Aussteuerungsgrund) |
| POST | `/api/v1/inkasso/schweben/{id}/aktionen/policieren` | `SCHWEBE_POLICIEREN` | Ausgesteuerte Inkasso-Schwebe manuell policieren |
| POST | `/api/v1/inkasso/schweben/{id}/aktionen/anpassen` | `INKASSO_AUSFUEHREN` | Beitrag manuell anpassen und anschließend policieren |
| POST | `/api/v1/inkasso/schweben/{id}/aktionen/zurueckstellen` | `INKASSO_AUSFUEHREN` | Inkasso für diesen Vertrag zurückstellen (z. B. Klärung) |

### Metadaten und Konfiguration

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| GET | `/api/v1/inkasso/konfiguration` | `INKASSO_LESEN` | Aktuelle Inkasso-Konfiguration (Vorlauftage, Schwellenwerte) |
| PUT | `/api/v1/inkasso/konfiguration` | `INKASSO_KONFIGURIEREN` | Inkasso-Konfiguration anpassen |
| GET | `/api/v1/inkasso/naechste-faelligkeiten` | `INKASSO_LESEN` | Vorschau: Verträge mit den nächsten Fälligkeiten (Haupt- und Nebenfälligkeiten) |

## Events

| Event | Auslöser | Payload | Konsument |
|-------|----------|---------|-----------|
| `InkassolaufGestartetEvent` | Batch-Job gestartet | inkassolauf_id, inkassotyp, stichtag, anzahl_selektiert | Monitoring, Durchlaufzeiten (UC-04) |
| `HauptinkassoDurchgefuehrtEvent` | Hauptinkasso policiert (pro Vertrag) | vertrag_id, vertragsstand_id, alter_beitrag, neuer_beitrag, beitragsanpassung | S1 (Provision), S2 (Inkasso), S4 (DWH), S5 (Druck) |
| `NebeninkassoForderungEvent` | Nebeninkasso-Forderung erzeugt | vertrag_id, faelligkeitsdatum, ratenbeitrag, forderungstyp | S2 (Inkasso), S4 (DWH) |
| `InkassoAusgesteuertEvent` | Vertrag im Inkasso ausgesteuert | vertrag_id, inkassolauf_id, aussteuerungsgrund, beitragsdifferenz_prozent | Schweben-Dashboard, Innendienst |
| `InkassolaufAbgeschlossenEvent` | Batch-Job abgeschlossen | inkassolauf_id, zusammenfassung (Anzahlen, Beitragssummen) | Monitoring, Berichtswesen |

## Statusmodell – Inkasso-Forderung

```
              ┌──────────┐
              │  OFFEN   │
              └────┬─────┘
                   │
        ┌──────────┼──────────┐
        │          │          │
   fällig +    storniert    zurück-
   bezahlt    (UC-05/06)   gestellt
        │          │          │
  ┌─────▼─────┐ ┌─▼────────┐ ┌─▼───────────┐
  │  BEZAHLT  │ │STORNIERT │ │ZURUECKGEST. │
  └───────────┘ └──────────┘ └─────────────┘
```

## Statusmodell – Inkassolauf

```
  ┌──────────┐      ┌──────────────┐      ┌──────────────┐
  │ GEPLANT  │─────►│ IN_AUSFUEHR. │─────►│ABGESCHLOSSEN │
  └──────────┘      └──────┬───────┘      └──────────────┘
                           │
                      abgebrochen
                           │
                    ┌──────▼───────┐
                    │ ABGEBROCHEN  │
                    └──────────────┘
```

## Datenmodell-Erweiterungen

### Inkassolauf

| Attribut | Typ | Pflicht | Beschreibung |
|----------|-----|---------|-------------|
| id | UUID | ✅ | Technischer Primärschlüssel |
| inkassotyp | Enum | ✅ | HAUPTINKASSO, NEBENINKASSO_RATE |
| status | Enum | ✅ | GEPLANT, IN_AUSFUEHRUNG, ABGESCHLOSSEN, ABGEBROCHEN |
| stichtag | Date | ✅ | Fälligkeitsstichtag |
| sparte | Enum | ❌ | Einschränkung auf Sparte (NULL = alle) |
| startzeit | Timestamp | ❌ | Startzeitpunkt |
| endzeit | Timestamp | ❌ | Endzeitpunkt |
| anzahl_selektiert | Integer | ❌ | Selektierte Verträge |
| anzahl_erfolgreich | Integer | ❌ | Erfolgreich verarbeitet |
| anzahl_ausgesteuert | Integer | ❌ | Ausgesteuert |
| anzahl_fehlerhaft | Integer | ❌ | Fehlerhaft |
| erstellt_von | String(100) | ✅ | Benutzer-ID oder `SYSTEM` |
| erstellt_am | Timestamp | ✅ | Erstellzeitpunkt |

### Inkasso-Ergebnis

| Attribut | Typ | Pflicht | Beschreibung |
|----------|-----|---------|-------------|
| id | UUID | ✅ | Technischer Primärschlüssel |
| inkassolauf_id | UUID (FK) | ✅ | Referenz auf den Inkassolauf |
| vertrag_id | UUID (FK) | ✅ | Referenz auf den Vertrag |
| ergebnis | Enum | ✅ | ERFOLGREICH, AUSGESTEUERT, FEHLERHAFT, UEBERSPRUNGEN |
| vertragsstand_id | UUID (FK) | ❌ | Referenz auf neuen Vertragsstand (bei Erfolg) |
| alter_jahresbeitrag | BigDecimal(12,2) | ❌ | Beitrag vor Neuberechnung |
| neuer_jahresbeitrag | BigDecimal(12,2) | ❌ | Beitrag nach Neuberechnung |
| aussteuerungsgrund | String(500) | ❌ | Grund der Aussteuerung |
| fehler_details | String(2000) | ❌ | Fehlerbeschreibung |
| erstellt_am | Timestamp | ✅ | Verarbeitungszeitpunkt |

**Constraints:**
- UNIQUE(inkassolauf_id, vertrag_id) – Idempotenz: pro Lauf und Vertrag nur ein Ergebnis

### Inkasso-Forderung

| Attribut | Typ | Pflicht | Beschreibung |
|----------|-----|---------|-------------|
| id | UUID | ✅ | Technischer Primärschlüssel |
| vertrag_id | UUID (FK) | ✅ | Referenz auf den Vertrag |
| vertragsstand_id | UUID (FK) | ✅ | Referenz auf den zugehörigen Vertragsstand |
| forderungstyp | Enum | ✅ | HAUPTINKASSO, NEBENINKASSO_RATE |
| faelligkeitsdatum | Date | ✅ | Fälligkeitstermin |
| ratenbeitrag | BigDecimal(12,2) | ✅ | Forderungsbetrag |
| status | Enum | ✅ | OFFEN, BEZAHLT, STORNIERT, ZURUECKGESTELLT |
| inkassolauf_id | UUID (FK) | ❌ | Referenz auf den erzeugenden Inkassolauf |
| erstellt_am | Timestamp | ✅ | Erstellzeitpunkt |
| geaendert_am | Timestamp | ✅ | Letzte Änderung |

**Constraints:**
- UNIQUE(vertrag_id, faelligkeitsdatum, forderungstyp) – keine doppelten Forderungen

## Nachbedingungen

### Hauptinkasso
- Ein **neuer Vertragsstand** (Version n+1) wurde am bestehenden Vertrag erzeugt mit `gueltig_ab` = HFT
- Ein **Vorgang** mit Typ `HAUPTINKASSO` wurde erzeugt und dem Vertragsstand zugeordnet
- Der **Zahlungsplan** für das neue Versicherungsjahr wurde erstellt (alle Fälligkeiten)
- Die **Hauptfälligkeit** des Vertrags wurde auf den nächsten HFT aktualisiert
- **DataWarehouse (S4)** hat die Hauptinkasso-Bewegung erhalten (Kafka)
- **Provision (S1)** hat das Beitragsereignis erhalten (Kafka)
- **Konzerninkasso (S2)** hat den neuen Zahlungsplan und die erste Beitragsforderung erhalten (Kafka)
- **Druckschnittstelle (S5)** hat den Druckauftrag für die Beitragsrechnung erhalten (REST)
- Spartenspezifische Hooks (`vor_Hauptinkasso`, `nach_Hauptinkasso`) wurden ausgeführt
- Der Inkassolauf ist protokolliert (Zusammenfassung und Einzelergebnisse)
- Durchlaufzeit-Meilensteine (UC-04) wurden erzeugt

### Nebeninkasso
- Eine **Nebeninkasso-Forderung** mit Status `OFFEN` wurde erzeugt
- **Konzerninkasso (S2)** hat die Ratenforderung erhalten (Kafka)
- **DataWarehouse (S4)** hat das Nebeninkasso-Ereignis erhalten (Kafka)
- **Kein** neuer Vertragsstand wurde erzeugt

## Neue Kompetenzen (→ Kompetenz-System S8)

| Kompetenz | Beschreibung | Typische Rolle |
|-----------|-------------|----------------|
| `INKASSO_AUSFUEHREN` | Inkassolauf manuell starten (Batch oder Einzelvertrag) | Innendienst-Senior, Batchadministrator |
| `INKASSO_LESEN` | Inkassoläufe, Ergebnisse und Forderungen einsehen | Innendienst, Controlling |
| `INKASSO_KONFIGURIEREN` | Inkasso-Konfiguration anpassen (Vorlauftage, Schwellenwerte) | Fachbereich, Systemadministration |

## Akzeptanzkriterien
- [ ] Der Batch-Job selektiert alle Verträge mit fälliger Hauptfälligkeit im konfigurierbaren Vorlaufzeitraum
- [ ] Beim Hauptinkasso wird der Beitrag für das neue Versicherungsjahr neuberechnet
- [ ] Ein neuer Vertragsstand (Version n+1) wird beim Hauptinkasso erzeugt
- [ ] Der Zahlungsplan wird gemäß Zahlungsweise mit allen Fälligkeiten erstellt
- [ ] Ratenbeiträge = Jahresbeitrag × Zahlungsweise-Faktor / Anzahl Raten (Rundungsdifferenz auf letzte Rate)
- [ ] Beim Nebeninkasso wird nur eine Forderung erzeugt – kein neuer Vertragsstand
- [ ] Alle vier Schnittstellen werden beim Hauptinkasso bedient: S4, S1, S2, S5
- [ ] Beim Nebeninkasso werden nur S2 und S4 bedient
- [ ] Beitragsänderungen über dem Schwellenwert werden ausgesteuert
- [ ] Bei Beitragssteigerung wird der VN auf sein Sonderkündigungsrecht hingewiesen
- [ ] Pro Vertrag und HFT wird maximal ein Hauptinkasso durchgeführt (Idempotenz)
- [ ] Fehler bei einem Vertrag blockieren nicht den gesamten Batch-Lauf
- [ ] Spartenspezifische Hooks (`vor_Hauptinkasso`, `nach_Hauptinkasso`) werden ausgelöst
- [ ] Inkassoläufe werden mit Zusammenfassung und Einzelergebnissen protokolliert
- [ ] Durchlaufzeit-Meilensteine (UC-04) werden korrekt erzeugt
- [ ] Manuelles Einzelvertrag-Inkasso funktioniert analog zum Batch (Phase 2–5)

## Wireframe / Skizze

### Inkassolauf-Übersicht (Batch-Monitoring)

```
┌──────────────────────────────────────────────────────────────────────┐
│  Inkasso – Laufübersicht                                            │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  [📅 Stichtag: 01.01.2028]  [Typ: Hauptinkasso ▼]  [🔍 Filtern]   │
│  [Sparte: Alle ▼]           [+ Manueller Lauf starten]             │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │ Lauf-ID      │ Typ          │ Stichtag    │ Status   │ Erg.   │  │
│  ├──────────────┼──────────────┼─────────────┼──────────┼────────┤  │
│  │ IL-2027-0042 │ Hauptinkasso │ 01.01.2028  │ ✅ Fertig │ 12.847 │  │
│  │ IL-2027-0041 │ Nebeninkasso │ 01.10.2027  │ ✅ Fertig │  3.421 │  │
│  │ IL-2027-0040 │ Nebeninkasso │ 01.07.2027  │ ✅ Fertig │  3.398 │  │
│  │ IL-2027-0039 │ Hauptinkasso │ 01.01.2027  │ ✅ Fertig │ 12.103 │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌── Lauf-Details: IL-2027-0042 ───────────────────────────────────┐ │
│  │ Startzeit: 02.12.2027 02:00:12  │  Endzeit: 02.12.2027 02:47:38│ │
│  │                                                                  │ │
│  │  Selektiert:    12.847          Gesamtbeitrag Alt:  6.234.891 € │ │
│  │  Erfolgreich:   12.614  (98,2%) Gesamtbeitrag Neu:  6.412.033 € │ │
│  │  Ausgesteuert:     187  (1,5%)  Ø Änderung:             +2,84 % │ │
│  │  Fehlerhaft:        31  (0,2%)                                   │ │
│  │  Übersprungen:      15  (0,1%)                                   │ │
│  │                                                                  │ │
│  │  [📋 Ergebnisse anzeigen]  [⚠ Fehler anzeigen]  [🔄 Retry]    │ │
│  └──────────────────────────────────────────────────────────────────┘ │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### Einzelvertrag – Inkasso-Detail

```
┌──────────────────────────────────────────────────────────────────────┐
│  Inkasso – VN-2026-100001                    Sparte: KFZ            │
├──────────────────────────────────────────────────────────────────────┤
│  VN: Max Mustermann │ HFT: 01.01.2028 │ Zahlungsweise: Halbjährl.  │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌── Beitragsvergleich ─────────────────────────────────────────┐   │
│  │                                                               │   │
│  │  Alter Jahresbeitrag:       487,32 €                         │   │
│  │  Neuer Jahresbeitrag:       502,15 €  (+14,83 € / +3,04 %)  │   │
│  │  Versicherungssteuer:        95,41 €  (19 %)                 │   │
│  │  Zahlungsweise-Aufschlag:    15,06 €  (3 %)                  │   │
│  │                                                               │   │
│  │  Produkte:                                                    │   │
│  │    KFZ-HP:  321,40 € → 331,20 €  (+9,80 €)                  │   │
│  │    KFZ-TK:  165,92 € → 170,95 €  (+5,03 €)                  │   │
│  │                                                               │   │
│  └───────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌── Zahlungsplan 2028 ─────────────────────────────────────────┐   │
│  │  Nr. │ Fälligkeit  │ Typ          │ Betrag    │ Status       │   │
│  │  ────┼─────────────┼──────────────┼───────────┼──────────── │   │
│  │   1  │ 01.01.2028  │ Hauptinkasso │  258,61 € │ ⬤ Offen    │   │
│  │   2  │ 01.07.2028  │ Nebeninkasso │  258,60 € │ ⬤ Geplant  │   │
│  │  ────┼─────────────┼──────────────┼───────────┼──────────── │   │
│  │  Σ   │             │              │  517,21 € │             │   │
│  └───────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ℹ Beitragssteigerung +3,04 % – Hinweis auf Sonderkündigungsrecht   │
│                                                                      │
│  [📄 Beitragsrechnung anzeigen]  [📤 Manuell policieren]           │
└──────────────────────────────────────────────────────────────────────┘
```

## Offene Fragen
- Soll der Batch-Job die Verträge **aller Sparten** in einem Lauf verarbeiten oder pro Sparte separate Läufe durchführen?
- Wie wird mit Verträgen umgegangen, bei denen zum HFT eine offene Änderung (UC-05) oder Stornierung (UC-06) existiert? Soll das Inkasso warten oder trotzdem durchgeführt werden?
- Soll es eine **Vorschau-Funktion** geben, die die Beitragsberechnung simuliert, bevor der Inkassolauf tatsächlich policiert?
- Wie wird die Zahlungsavis für den VN gestaltet – nur per Druckschnittstelle (S5) oder auch per E-Mail?
- Soll bei der Beitragsrechnung auch der alte Zahlungsplan (Vorjahr) verglichen/angezeigt werden?
- Gibt es eine **Kulanzregelung** für geringe Beitragsänderungen (z. B. < 1 €), bei denen keine separate Beitragsrechnung gedruckt wird?
- Wie wird mit dem Nebeninkasso bei unterjährigen Vertragsänderungen umgegangen – wird der Zahlungsplan komplett neu erstellt oder nur die Folgeraten angepasst?
