# UC-05: Vertragsänderung durchführen (Nachtrag)

## Beschreibung
Ein Benutzer führt eine **Änderung an einem bestehenden Vertrag** durch. Aus dem aktuellen Vertragsstand wird ein **Änderungsangebot**, ein **Änderungsantrag** oder direkt eine **Änderungsschwebe** erzeugt. Welche Felder bearbeitbar sind, wird durch die **spartenspezifische Fachlichkeit** und die **Kompetenz des Benutzers** bestimmt. Nach Prüfung und Freigabe wird die Änderung policiert – dabei entsteht ein neuer Vertragsstand am bestehenden Vertrag. Die nachgelagerten Systeme (DataWarehouse S4, Provision S1, Konzerninkasso S2, Druck S5) werden bei der Policierung versorgt.

Dies ist ein **spartenübergreifender** Kernprozess. Spartenspezifische Besonderheiten (z. B. Adressänderung, Deckungserweiterung) werden über die Extension Points / Hooks der Spartenkonfiguration eingebunden.

> **Hinweis:** Der **Fahrzeugwechsel** bei KFZ wird **nicht** als Nachtrag über UC-05 abgebildet, sondern als eigenständiger Prozess mit Neuvertrag (→ UC-KFZ-04).

## Akteure
- **Primär:** Außendienst (Vertrieb), Innendienst (Sachbearbeitung)
- **Sekundär:** Kunde (via Kundenportal S7), System (automatische Nachtragsverarbeitung)

## Vorbedingungen
- Ein Vertrag im Status **`AKTIV`** oder **`RUHEND`** existiert mit mindestens einem gültigen Vertragsstand
- Benutzer ist am System angemeldet und besitzt mindestens die Kompetenz **`VERTRAG_AENDERN`** (→ Kompetenz-System S8)
- Die Sparte des Vertrags ist aktiv konfiguriert mit gültiger Produktkonfiguration

## Auslöser
- Benutzer öffnet einen bestehenden Vertrag und wählt „Vertragsänderung starten"
- Kunde beantragt eine Änderung über das Kundenportal (S7)
- Systemereignis löst automatische Änderung aus (z. B. Beitragsanpassung zum Hauptfälligkeitstermin, SF-Hochstufung)

## Ablauf (Hauptszenario)

### Phase 1: Änderung initiieren
1. Benutzer öffnet den Vertrag und wählt die Funktion **„Vertragsänderung starten"**
2. System prüft die Kompetenz des Benutzers über das Kompetenz-System (S8)
3. System prüft, ob der Vertrag im Status `AKTIV` oder `RUHEND` ist
4. System prüft, ob bereits eine **offene Änderung** (offenes Änderungsangebot, -antrag oder -schwebe) für diesen Vertrag existiert:
   - Ja → System zeigt Hinweis: „Es existiert bereits eine offene Änderung. Bitte diese zuerst abschließen oder verwerfen."
   - Nein → weiter mit Schritt 5
5. System zeigt die **Änderungsoptionen** basierend auf der Sparte und der Kompetenz des Benutzers:
   - **Einstieg über Änderungsangebot**: Benutzer möchte die Änderung zunächst als Angebot erfassen (z. B. bei Kundengespräch, Beratungssituation)
   - **Einstieg über Änderungsantrag**: Änderung wird direkt als Antrag erfasst (z. B. bei eindeutigen Änderungswünschen)
   - **Einstieg über Änderungsschwebe**: Änderung wird direkt als Innendienst-Schwebe erzeugt (z. B. bei der ausgesteuerten oder manuellen Bearbeitung)

### Phase 2: Änderungsdaten erfassen
6. System erzeugt das gewählte Änderungsobjekt (Änderungsangebot, Änderungsantrag oder Änderungsschwebe) im Status **„Entwurf"** als **Kopie des aktuellen Vertragsstands**
7. System ermittelt die **bearbeitbaren Felder** anhand von:
   - **Spartenspezifische Feldkonfiguration**: Welche Felder sind für diese Sparte bei einer Änderung grundsätzlich bearbeitbar? (konfiguriert in der Sparten-Registry)
   - **Kompetenz des Benutzers**: Welche Felder darf dieser Benutzer bearbeiten? (z. B. Außendienst darf nur bestimmte Felder ändern, Innendienst hat erweiterte Rechte)
   - **Änderungstyp**: Bestimmte Änderungstypen schränken die bearbeitbaren Felder weiter ein (z. B. bei reiner Adressänderung sind nur Adressfelder editierbar)
8. System zeigt die Vertragsdetails an; bearbeitbare Felder sind editierbar, nicht bearbeitbare Felder sind schreibgeschützt dargestellt
9. → **Hook `vor_Nachtrag`** wird ausgelöst (spartenspezifische Vorabprüfungen und Zusatzerfassungen, z. B. Fahrzeugwechsel-Plausibilitäten bei KFZ)
10. Benutzer ändert die gewünschten Felder:
    - **Produkte hinzufügen / entfernen** (z. B. Teilkasko hinzufügen)
    - **Deckungsbausteine anpassen** (z. B. Versicherungssumme erhöhen)
    - **Tarifmerkmale ändern** (z. B. Fahrerkreis, Zahlungsweise, Selbstbeteiligung)
    - **Vertragsdaten anpassen** (z. B. Laufzeitänderung, Hauptfälligkeit)
    - **Spartenspezifische Daten ändern** (z. B. Fahrzeugwechsel, Adressänderung)
11. Benutzer setzt das **Änderungsdatum** (= Gültig-ab des neuen Vertragsstands):
    - Standard: nächster Tag oder nächster Hauptfälligkeitstermin
    - Rückwirkende Änderungen nur mit Kompetenz `NACHTRAG_RUECKWIRKEND`
12. System führt bei jeder Eingabe **Plausibilitätsprüfungen** durch (spartenübergreifend + spartenspezifisch aus Regelengine)
13. System zeigt Plausibilitätsfehler und -hinweise in Echtzeit an

### Phase 3: Änderung berechnen und prüfen
14. Benutzer löst die **Beitragsberechnung** aus (oder diese erfolgt automatisch)
15. System berechnet den neuen Beitrag auf Basis der geänderten Daten
16. System zeigt eine **Beitragsdifferenz-Ansicht**:
    - Alter Jahresbeitrag (aktueller Vertragsstand)
    - Neuer Jahresbeitrag (nach Änderung)
    - Differenz absolut und prozentual
    - Aufschlüsselung nach Produkten / Bausteinen
    - Ggf. Nachberechnung / Rückerstattung für den laufenden Versicherungszeitraum
17. Benutzer löst die **Gesamtprüfung** aus
18. System führt alle Plausibilitätsprüfungen durch (spartenübergreifend + spartenspezifisch)
19. System zeigt das Prüfergebnis an:
    - ✅ **Prüfung bestanden** → Änderung kann beantragt / freigegeben werden
    - ❌ **Prüfung nicht bestanden** → Fehler werden angezeigt, Beantragung blockiert

### Phase 4: Änderung beantragen / freigeben

#### Pfad A: Einstieg über Änderungsangebot
20a. Benutzer wählt **„Änderungsangebot beantragen"** (analog UC-01, Phase 5)
21a. System erzeugt einen **Änderungsantrag** aus dem Änderungsangebot
22a. Benutzer entscheidet, ob das Änderungsangebot erhalten bleibt oder gelöscht wird
23a. Weiter mit Schritt 24 (Änderungsantrag freigeben)

#### Pfad B: Einstieg über Änderungsantrag (direkt)
20b. Änderungsantrag wurde direkt erzeugt (Schritt 6) → weiter mit Schritt 24

#### Pfad C: Einstieg über Änderungsschwebe (direkt)
20c. Änderungsschwebe wurde direkt erzeugt (Schritt 6) → weiter mit Schritt 29 (Policierung)

#### Änderungsantrag freigeben
24. Benutzer wählt die Funktion **„Änderungsantrag freigeben"**
25. System prüft die Kompetenz `ANTRAG_FREIGEBEN`
26. System ermittelt den **Vorgangstyp** → `AENDERUNG`
27. System legt eine **Schwebe** an (→ GR-A05)
28. System prüft **Aussteuerungsregeln** (analog UC-02, Phase 4):
    - **Nicht ausgesteuert** → weiter mit Schritt 29 (Policierung)
    - **Ausgesteuert** → Schwebe bleibt offen, Innendienst bearbeitet (analog UC-02, Pfad B)

### Phase 5: Policierung (neuer Vertragsstand)
29. → **Hook `vor_Policierung`** wird ausgelöst (spartenspezifische Prüfungen, z. B. SF-Klasse validieren bei KFZ)
30. System erzeugt einen **neuen Vertragsstand** (Version n+1) am bestehenden Vertrag:
    - Aktueller Vertragsstand erhält `gueltig_bis` = Änderungsdatum − 1 Tag
    - Neuer Vertragsstand erhält `gueltig_ab` = Änderungsdatum
    - Neuer Vertragsstand enthält die geänderten Daten und den neu berechneten Beitrag
31. System erzeugt einen **Vorgang** mit Vorgangstyp `AENDERUNG` und verknüpft den neuen Vertragsstand
32. System aktualisiert den **Vertrag**: `aktueller_jahresbeitrag` wird auf den neuen Beitrag gesetzt
33. → **Hook `nach_Policierung`** wird ausgelöst (spartenspezifische Aktionen, z. B. neue eVB bei Fahrzeugwechsel)

### Phase 6: Nachgelagerte Systeme versorgen
34. System publiziert **`VertragsstandErzeugtEvent`** (Spring Event) → löst folgende Schnittstellenaufrufe aus:

| Schnittstelle | Aktion | Payload-Kern | Mechanismus |
|---------------|--------|-------------|-------------|
| **S4 – DataWarehouse** | Vertragsbewegung melden | Vertragsnummer, Sparte, Vorgangstyp `AENDERUNG`, alter + neuer Beitrag, Änderungsdatum | Kafka-Topic `vertrag.datawarehouse.bewegungen` |
| **S1 – Provision** | Änderungsereignis melden | Vertragsnummer, Sparte, alter + neuer Beitrag, Produktänderungen, Vermittler-ID | Kafka-Topic `vertrag.provision.ereignisse` (Ereignistyp `AENDERUNG`) |
| **S2 – Konzerninkasso** | Neue Beitragsforderung / Zahlungsplan | Vertragsnummer, neuer Beitrag, Zahlungsweise, Fälligkeit, ggf. Nachberechnung | Kafka-Topic `vertrag.inkasso.beitragsforderungen` |
| **S5 – Druck** | Nachtrag drucken | Vertragsnummer, Partner, Vertragsstand (Version n+1), Produkte, Dokumenttyp `NACHTRAG` | REST `POST /api/v1/druck/auftraege` |

35. System setzt die Schwebe auf Status **„Erledigt"**
36. System bestätigt die erfolgreiche Vertragsänderung und zeigt den neuen Vertragsstand an

## Änderungstypen (spartenübergreifend)

| Änderungstyp | Beschreibung | Typische bearbeitbare Felder | Beitragsrelevant |
|-------------|-------------|------------------------------|-----------------|
| `PRODUKTAENDERUNG` | Produkte / Deckungsbausteine hinzufügen oder entfernen | Produktauswahl, Deckungsbausteine | ✅ Ja |
| `TARIFMERKMAL_AENDERUNG` | Tarifmerkmale anpassen (z. B. SB, Fahrerkreis) | Tarifmerkmale, Selbstbeteiligung | ✅ Ja |
| `ZAHLUNGSWEISE_AENDERUNG` | Zahlungsweise ändern (jährlich → monatlich etc.) | Zahlungsweise | ✅ Ja (Aufschlag) |
| `LAUFZEIT_AENDERUNG` | Vertragslaufzeit anpassen | Laufzeit, Vertragsende | ⚠️ Ggf. |
| `ADRESSAENDERUNG` | Adresse des VN ändern | PLZ, Ort, Straße | ⚠️ Spartenabhängig (KFZ: Regionalklasse) |
| `SPARTENDATEN_AENDERUNG` | Spartenspezifische Daten ändern | Spartenspezifische Felder (z. B. Fahrerdaten, Nutzungsart) | ✅ Ja |
| `SONSTIGE_AENDERUNG` | Allgemeine Änderung (manuell) | Kompetenzabhängig | ⚠️ Prüfung |

## Feldberechtigungsmatrix (Beispiel)

> Die konkreten Feldberechtigungen werden pro Sparte in der Sparten-Registry konfiguriert. Hier ein Beispiel für die spartenübergreifende Standard-Matrix.

| Feldgruppe | Außendienst (`VERTRAG_AENDERN`) | Innendienst (`NACHTRAG_INNENDIENST`) | Innendienst-Senior (`NACHTRAG_ERWEITERT`) |
|------------|-------------------------------|--------------------------------------|------------------------------------------|
| Produkte hinzufügen/entfernen | ✅ | ✅ | ✅ |
| Deckungsbausteine anpassen | ✅ | ✅ | ✅ |
| Tarifmerkmale ändern | ✅ (eingeschränkt) | ✅ | ✅ |
| Zahlungsweise ändern | ✅ | ✅ | ✅ |
| Selbstbeteiligung ändern | ✅ | ✅ | ✅ |
| Vertragsbeginn/-ende ändern | ❌ | ✅ | ✅ |
| Rückwirkende Änderung | ❌ | ❌ | ✅ (`NACHTRAG_RUECKWIRKEND`) |
| Spartenspezifische Daten | ✅ (über Hook) | ✅ (über Hook) | ✅ (über Hook) |
| Manuelle Beitragskorrektur | ❌ | ❌ | ✅ (`BEITRAG_MANUELL`) |

## Alternativszenarien

- **A1: Änderung zwischenspeichern**
  Der Benutzer kann das Änderungsangebot / den Änderungsantrag jederzeit speichern und später weiterbearbeiten. Der Status bleibt „Entwurf".

- **A2: Änderung verwerfen**
  Der Benutzer kann eine offene Änderung verwerfen. System fragt: „Änderung wirklich verwerfen?" Bei Bestätigung wird das Änderungsobjekt auf Status **„Verworfen"** gesetzt. Der Vertrag bleibt unverändert.

- **A3: Änderung über Kundenportal (S7)**
  Der Kunde beantragt eine Änderung über das Kundenportal. Die Änderung wird als Änderungsantrag im Status „Offen" ins System übernommen und automatisch ausgesteuert (immer Innendienst-Prüfung bei Kundenportal-Änderungen).

- **A4: Automatische Änderung (Systemereignis)**
  Bestimmte Änderungen werden automatisch ausgelöst (z. B. jährliche SF-Hochstufung, Beitragsanpassung zum Hauptfälligkeitstermin). System erzeugt direkt eine Änderungsschwebe, rechnet neu, und policiert automatisch (Dunkelverarbeitung ohne Benutzerinteraktion).

- **A5: Mehrere Änderungen bündeln**
  Der Benutzer kann in einem Änderungsvorgang mehrere Änderungstypen kombinieren (z. B. Fahrerkreisänderung + Teilkasko hinzufügen). Alle Änderungen werden in einem einzelnen neuen Vertragsstand zusammengefasst.

- **A6: Stornierung statt Änderung**
  Stellt sich bei der Änderung heraus, dass der Vertrag beendet werden soll, kann der Benutzer zur Vertragskündigung wechseln (→ UC-XX: Vertrag kündigen). Das Änderungsobjekt wird verworfen.

- **A7: Beitragsfreie Änderung**
  Bestimmte Änderungen (z. B. reine Adressänderung ohne Beitragsauswirkung) durchlaufen einen vereinfachten Prozess: Prüfung → direkte Policierung (ohne Aussteuerung), und Provision/Inkasso werden nur informiert, nicht mit neuen Beitragsdaten versorgt.

## Fehlerfälle

- **F1: Offene Änderung existiert bereits**
  Für den Vertrag existiert bereits ein offenes Änderungsangebot, -antrag oder -schwebe. → System zeigt Hinweis und Link zur bestehenden Änderung. Neue Änderung wird blockiert.

- **F2: Vertrag nicht im Status AKTIV oder RUHEND**
  Der Vertrag ist gekündigt, storniert oder abgelaufen. → System zeigt Fehlermeldung: „An diesem Vertrag können keine Änderungen vorgenommen werden."

- **F3: Kompetenz nicht ausreichend**
  Benutzer versucht ein Feld zu bearbeiten, für das er keine Kompetenz besitzt. → Feld bleibt schreibgeschützt. Falls der Benutzer die Kompetenz `VERTRAG_AENDERN` gar nicht besitzt: „Sie verfügen nicht über die erforderliche Berechtigung."

- **F4: Plausibilitätsfehler**
  Die geänderten Daten verletzen Plausibilitätsregeln (spartenübergreifend oder spartenspezifisch). → System zeigt Fehler an, Freigabe/Beantragung wird blockiert.

- **F5: Beitragsberechnung fehlgeschlagen**
  Unvollständige oder widersprüchliche Daten verhindern die Berechnung. → System zeigt die fehlenden/fehlerhaften Felder an.

- **F6: Rückwirkende Änderung ohne Kompetenz**
  Benutzer setzt ein Änderungsdatum in der Vergangenheit, besitzt aber nicht `NACHTRAG_RUECKWIRKEND`. → System zeigt Fehlermeldung: „Rückwirkende Änderungen erfordern eine erweiterte Berechtigung."

- **F7: Schnittstellenfehler bei Policierung**
  Eine oder mehrere nachgelagerte Schnittstellen (S1–S5) sind nicht erreichbar. → Vertragsstand wird trotzdem erzeugt (→ GR-A11). Fehlgeschlagene Aufrufe werden über Retry-Mechanismen (Transactional Outbox / Dead-Letter-Topic) nachgeliefert. Wiedervorlage wird angelegt.

- **F8: Parallelitätskonflikt**
  Zwei Benutzer versuchen gleichzeitig eine Änderung am selben Vertrag. → Optimistic Locking (Version auf Vertragsstand): Der zweite Benutzer erhält einen Hinweis, dass der Vertrag zwischenzeitlich geändert wurde.

## Geschäftsregeln

| Regel-ID | Beschreibung | Quelle |
|----------|-------------|--------|
| GR-V02 | Ein Vertragsstand gehört immer zu genau einem Vorgang (Neugeschäft, Änderung, Stornierung) | UC-02, UC-05 |
| GR-V03 | Der Vorgangstyp wird automatisch aus dem Antragskontext ermittelt – bei bestehendem Vertrag: `AENDERUNG` | UC-02, UC-05 |
| GR-V05 | Bei jeder Policierung wird die Vertragsbewegung an das DataWarehouse (S4) gemeldet | UC-05 |
| GR-A05 | Bei jeder Freigabe wird eine Schwebe angelegt | UC-02, UC-05 |
| GR-A06 | Aussteuerungsregeln bestimmen, ob der Änderungsantrag dem Innendienst vorgelegt wird | UC-02, UC-05 |
| GR-A08 | Bei der Policierung werden immer alle vier Schnittstellen bestückt: S4, S1, S2, S5 | UC-05 |
| GR-A09 | Die Policierung erzeugt immer genau einen Vertragsstand, der genau einem Vorgang zugeordnet ist | UC-02, UC-05 |
| GR-A10 | Bei Änderung wird der bestehende Vertrag fortgeführt (neuer Vertragsstand am selben Vertrag) | UC-05 |
| GR-A11 | Schnittstellenfehler führen nicht zum Abbruch der Vertragsstand-Erzeugung; Retry-Mechanismen liefern nach | UC-05 |
| GR-A12 | Manuelle Policierung einer Schwebe erfordert die Kompetenz `SCHWEBE_POLICIEREN` | UC-02, UC-05 |
| GR-NA-01 | Pro Vertrag darf zu jedem Zeitpunkt maximal ein offenes Änderungsangebot, -antrag oder -schwebe existieren | UC-05 |
| GR-NA-02 | Die bearbeitbaren Felder werden durch die Kombination aus Spartenkonfiguration und Benutzerkompetenz bestimmt | UC-05 |
| GR-NA-03 | Rückwirkende Änderungen (Änderungsdatum < heute) erfordern die Kompetenz `NACHTRAG_RUECKWIRKEND` | UC-05 |
| GR-NA-04 | Beitragsfreie Änderungen (z. B. reine Adressänderung ohne Beitragsauswirkung) durchlaufen den vereinfachten Pfad ohne Aussteuerung | UC-05 |
| GR-NA-05 | Das Änderungsdatum des neuen Vertragsstands muss nach dem `gueltig_ab` des aktuellen Vertragsstands liegen (außer bei rückwirkender Änderung) | UC-05 |
| GR-NA-06 | Beitragsänderungen, die sich auf den laufenden Versicherungszeitraum auswirken, lösen eine Nachberechnung / Rückerstattung aus (→ S2 Konzerninkasso) | UC-05 |
| GR-NA-07 | Bei Änderungen über das Kundenportal (S7) wird immer eine Aussteuerung an den Innendienst erzwungen | UC-05 |

## Daten (Ein-/Ausgabe)

### Eingabedaten

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| vertrag_id | UUID | ✅ | Existierender Vertrag im Status AKTIV/RUHEND | Zu ändernder Vertrag |
| einstiegstyp | Enum | ✅ | AENDERUNGSANGEBOT, AENDERUNGSANTRAG, AENDERUNGSSCHWEBE | Einstiegspfad |
| aenderungsdatum | Date | ✅ | ≥ heute (oder < heute mit `NACHTRAG_RUECKWIRKEND`) | Gültig-ab des neuen Vertragsstands |
| aenderungsgrund | String(500) | ❌ | – | Optionaler Grund für die Änderung |
| geaenderte_felder | JSONB | ✅ | Mindestens ein Feld geändert | Geänderte Daten (Produkte, Tarifmerkmale, Spartendaten etc.) |
| Spezifische Änderungsdaten | Variabel | Spartenabhängig | Über Spartenplausibilitäten | z. B. neues Fahrzeug bei Fahrzeugwechsel |

### Ausgabedaten

| Feld | Typ | Beschreibung |
|------|-----|-------------|
| aenderungsobjekt_id | UUID | ID des Änderungsangebots / -antrags / -schwebe |
| aenderungsobjekt_typ | Enum | AENDERUNGSANGEBOT, AENDERUNGSANTRAG, AENDERUNGSSCHWEBE |
| status | Enum | Aktueller Status des Änderungsobjekts |
| beitrag_alt | BigDecimal(12,2) | Alter Jahresbeitrag |
| beitrag_neu | BigDecimal(12,2) | Neuer Jahresbeitrag nach Änderung |
| beitrag_differenz | BigDecimal(12,2) | Differenz (kann negativ sein) |
| nachberechnung_betrag | BigDecimal(12,2) | Nachberechnung/Rückerstattung für laufenden Zeitraum |
| neuer_vertragsstand_id | UUID | ID des neuen Vertragsstands (nach Policierung) |
| neuer_vertragsstand_version | Integer | Version des neuen Vertragsstands |
| vorgang_id | UUID | ID des erzeugten Vorgangs |

## API-Endpunkte

### Änderung initiieren und bearbeiten

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| POST | `/api/v1/vertraege/{vertrag_id}/aenderungen` | `VERTRAG_AENDERN` | Änderung starten (erzeugt Änderungsangebot, -antrag oder -schwebe) |
| GET | `/api/v1/vertraege/{vertrag_id}/aenderungen` | `VERTRAG_LESEN` | Offene und abgeschlossene Änderungen des Vertrags auflisten |
| GET | `/api/v1/vertraege/{vertrag_id}/aenderungen/{id}` | `VERTRAG_LESEN` | Änderungsdetails abrufen |
| PUT | `/api/v1/vertraege/{vertrag_id}/aenderungen/{id}` | `VERTRAG_AENDERN` | Änderungsdaten aktualisieren (Felder ändern) |
| DELETE | `/api/v1/vertraege/{vertrag_id}/aenderungen/{id}` | `VERTRAG_AENDERN` | Änderung verwerfen (logisches Löschen → Status „Verworfen") |

### Berechnung und Prüfung

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| POST | `/api/v1/vertraege/{vertrag_id}/aenderungen/{id}/aktionen/berechnen` | `VERTRAG_AENDERN` | Beitrag neu berechnen |
| POST | `/api/v1/vertraege/{vertrag_id}/aenderungen/{id}/aktionen/pruefen` | `VERTRAG_AENDERN` | Gesamtprüfung (Plausibilitäten) |
| GET | `/api/v1/vertraege/{vertrag_id}/aenderungen/{id}/beitragsdifferenz` | `VERTRAG_LESEN` | Beitragsdifferenz Alt/Neu abrufen |

### Beantragung und Freigabe

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| POST | `/api/v1/vertraege/{vertrag_id}/aenderungen/{id}/aktionen/beantragen` | `ANGEBOT_BEANTRAGEN` | Änderungsangebot → Änderungsantrag |
| POST | `/api/v1/vertraege/{vertrag_id}/aenderungen/{id}/aktionen/freigeben` | `ANTRAG_FREIGEBEN` | Änderungsantrag freigeben → Schwebe + ggf. Vertragsstand |

### Feldberechtigungen

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| GET | `/api/v1/vertraege/{vertrag_id}/aenderungen/bearbeitbare-felder` | `VERTRAG_AENDERN` | Bearbeitbare Felder für den aktuellen Benutzer ermitteln |

## Events

| Event | Auslöser | Payload | Konsument |
|-------|----------|---------|-----------|
| `AenderungGestartetEvent` | Änderung initiiert | vertrag_id, aenderungsobjekt_id, einstiegstyp, benutzer_id | Durchlaufzeiten (UC-04), Audit-Log |
| `AenderungBerechnetEvent` | Beitrag neu berechnet | vertrag_id, beitrag_alt, beitrag_neu, differenz | Audit-Log |
| `AenderungFreigegebenEvent` | Änderungsantrag freigegeben | vertrag_id, antrag_id, ausgesteuert | Schweben-Dashboard, Innendienst |
| `VertragsstandErzeugtEvent` | Neuer Vertragsstand (Policierung) | vertrag_id, vertragsstand_id, version, vorgangstyp `AENDERUNG`, jahresbeitrag | S1 (Provision), S2 (Inkasso), S4 (DWH), S5 (Druck) |
| `NachtragPoliciert Event` | Nachtrag abgeschlossen | vertrag_id, vertragsstand_id, aenderungstyp, beitrag_alt, beitrag_neu | UC-04 (Meilenstein `POLICIERT`), Audit-Log |

## Statusmodell – Änderungsobjekt

```
                    ┌──────────┐
                    │ Entwurf  │
                    └────┬─────┘
                         │
              ┌──────────┼──────────┐
              │          │          │
        berechnen    berechnen   berechnen
              │          │          │
        ┌─────▼──┐  ┌───▼────┐  ┌──▼───────┐
        │ Änd.-  │  │ Änd.-  │  │ Änd.-    │
        │Angebot │  │Antrag  │  │Schwebe   │
        │(Ber.)  │  │(Ber.)  │  │(Bearb.)  │
        └───┬────┘  └───┬────┘  └────┬─────┘
            │            │            │
       beantragen   freigeben    policieren
            │            │            │
        ┌───▼────┐  ┌───▼────┐       │
        │ Änd.-  │  │ Freige-│       │
        │Antrag  │  │ geben /│       │
        │(Offen) │  │ Ausge- │       │
        └───┬────┘  │ steuert│       │
            │       └───┬────┘       │
       freigeben        │            │
            │           │            │
            └─────┬─────┘────────────┘
                  │
            ┌─────▼──────┐
            │ Neuer      │
            │ Vertrags-  │
            │ stand (n+1)│
            └────────────┘

    Sonderstatus: Verworfen (aus jedem Status heraus, außer Policiert)
```

## Nachbedingungen
- Ein neuer **Vertragsstand** (Version n+1) wurde am bestehenden Vertrag erzeugt mit den geänderten Daten
- Der vorherige Vertragsstand hat ein `gueltig_bis`-Datum erhalten
- Ein **Vorgang** mit Typ `AENDERUNG` wurde erzeugt und dem Vertragsstand zugeordnet
- Die **Schwebe** wurde auf „Erledigt" gesetzt
- **DataWarehouse (S4)** hat die Vertragsbewegung erhalten (Kafka)
- **Provision (S1)** hat das Änderungsereignis erhalten (Kafka)
- **Konzerninkasso (S2)** hat die neue Beitragsforderung / Nachberechnung erhalten (Kafka)
- **Druckschnittstelle (S5)** hat den Nachtrag-Druckauftrag erhalten (REST)
- Alle Statuswechsel und der `erstellt_am`-Zeitstempel des neuen Vertragsstands sind revisionssicher protokolliert
- Spartenspezifische Hooks (`vor_Nachtrag`, `vor_Policierung`, `nach_Policierung`, `nach_Nachtrag`) wurden ausgeführt

## Neue Kompetenzen (→ Kompetenz-System S8)

| Kompetenz | Beschreibung | Typische Rolle |
|-----------|-------------|----------------|
| `VERTRAG_AENDERN` | Änderung initiieren und bearbeiten (Standard-Felder) | Außendienst, Innendienst |
| `NACHTRAG_INNENDIENST` | Erweiterte Felder bei Änderung bearbeiten | Innendienst |
| `NACHTRAG_ERWEITERT` | Alle Felder inkl. Vertragsbeginn, manuelle Beiträge | Innendienst-Senior |
| `NACHTRAG_RUECKWIRKEND` | Rückwirkende Änderungen (Änderungsdatum < heute) zulassen | Innendienst-Senior, Teamleiter |
| `BEITRAG_MANUELL` | Manuelle Beitragskorrektur (Über-/Unterschreitung der Berechnung) | Innendienst-Senior |

## Akzeptanzkriterien
- [ ] Aus einem bestehenden Vertrag kann ein Änderungsangebot, Änderungsantrag oder eine Änderungsschwebe erzeugt werden
- [ ] Die Änderung wird als Kopie des aktuellen Vertragsstands initialisiert
- [ ] Bearbeitbare Felder werden dynamisch anhand von Spartenkonfiguration und Benutzerkompetenz ermittelt
- [ ] Nicht bearbeitbare Felder werden schreibgeschützt dargestellt
- [ ] Beitragsberechnung zeigt Alt/Neu-Vergleich mit Differenz und Nachberechnung
- [ ] Plausibilitätsprüfungen (spartenübergreifend + spartenspezifisch) werden korrekt ausgeführt
- [ ] Der Prozess Änderungsangebot → Änderungsantrag → Freigabe → Vertragsstand funktioniert durchgängig
- [ ] Direkt-Einstieg als Änderungsantrag und Änderungsschwebe funktioniert
- [ ] Aussteuerungsregeln werden analog UC-02 angewendet
- [ ] Policierung erzeugt einen neuen Vertragsstand (n+1) mit korrektem `gueltig_ab`/`gueltig_bis`
- [ ] Vorgang mit Typ `AENDERUNG` wird erzeugt und dem Vertragsstand zugeordnet
- [ ] Alle vier Schnittstellen werden bei Policierung bedient: S4 (DWH), S1 (Provision), S2 (Inkasso), S5 (Druck)
- [ ] Schnittstellenfehler führen nicht zum Abbruch – Retry-Mechanismen greifen
- [ ] Pro Vertrag ist maximal eine offene Änderung gleichzeitig möglich
- [ ] Rückwirkende Änderungen sind nur mit Kompetenz `NACHTRAG_RUECKWIRKEND` möglich
- [ ] Spartenspezifische Hooks (`vor_Nachtrag`, `vor_Policierung`, `nach_Policierung`, `nach_Nachtrag`) werden ausgelöst
- [ ] Durchlaufzeit-Meilensteine (UC-04) werden korrekt erzeugt

## Wireframe / Skizze

```
┌─────────────────────────────────────────────────────────────────────┐
│  Vertragsänderung – VN-2026-100001        Status: ⬤ Entwurf       │
├─────────────────────────────────────────────────────────────────────┤
│  Vertrag: VN-2026-100001 │ Sparte: KFZ │ VN: Max Mustermann       │
│  Änderungsdatum: [01.04.2027 📅]        │ Grund: [Fahrerkreis ▼]  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌── Vertragsdetails (geändert hervorgehoben) ─────────────────┐   │
│  │                                                              │   │
│  │  Produkte:              (bearbeitbar ✏)                      │   │
│  │  ☑ KFZ-Haftpflicht     ☑ Teilkasko    ☐ Vollkasko           │   │
│  │                                                              │   │
│  │  Zahlungsweise: Jährlich ✏     Laufzeit: 1 Jahr 🔒          │   │
│  │  SB Teilkasko: 150 € ✏                                      │   │
│  │                                                              │   │
│  │  ┌── Spartenspezifisch (KFZ) ─────────────────────────┐     │   │
│  │  │ Fahrzeug: VW Golf VIII               🔒             │     │   │
│  │  │ Fahrerkreis: [Nur VN + Partner ▼]    ✏ GEÄNDERT    │     │   │
│  │  │ SF-Klasse: SF 5                      🔒             │     │   │
│  │  └──────────────────────────────────────────────────────┘     │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌── Beitragsdifferenz ────────────────────────────────────────┐   │
│  │  Position           │ Alt (€/Jahr) │ Neu (€/Jahr) │ Diff   │   │
│  │  KFZ-Haftpflicht    │   312,00     │   285,00     │ -27,00 │   │
│  │  Teilkasko          │   167,11     │   167,11     │   0,00 │   │
│  │  ────────────────────┼──────────────┼──────────────┼────────│   │
│  │  Gesamt             │   487,32     │   460,32     │ -27,00 │   │
│  │                     │              │              │  -5,5 % │   │
│  │  Nachberechnung (01.04.–31.12.): Erstattung -20,25 €       │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  🔒 = nicht bearbeitbar (Kompetenz/Sparte)                         │
│  ✏ = bearbeitbar                                                    │
│                                                                     │
│  [💾 Speichern]  [📊 Berechnen]  [✔ Prüfen]  [📤 Freigeben]      │
└─────────────────────────────────────────────────────────────────────┘
```

## Technische Hinweise
- **Feldberechtigungen**: Die Kombination aus Spartenkonfiguration (welche Felder sind bei Änderung grundsätzlich bearbeitbar) und Benutzerkompetenz (welche Felder darf der Benutzer ändern) wird über einen `FeldberechtigungsService` aufgelöst, der die Sparten-Registry und das Kompetenz-System (S8) abfragt
- **Änderungsdiff**: Beim Speichern wird automatisch ein Diff zwischen dem Original-Vertragsstand und den geänderten Daten erzeugt und im Änderungsobjekt als JSONB gespeichert (für Audit und Innendienst-Anzeige)
- **Parallelitätsschutz**: Optimistic Locking über `version`-Feld auf dem Vertragsstand verhindert Race Conditions bei gleichzeitigem Zugriff
- **Transactional Outbox Pattern**: Schnittstellenaufrufe (S1, S2, S4) werden über eine Outbox-Tabelle + Polling sichergestellt, damit keine Kafka-Messages bei DB-Rollback verloren gehen
- **Nachberechnung**: Bei unterjährigen Änderungen wird die Pro-Rata-Differenz berechnet (Tage im laufenden Versicherungsjahr × Beitragsdifferenz / 365) und als separate Position an Inkasso (S2) gemeldet

## Offene Fragen
- Soll bei jeder Produktänderung zwingend ein Änderungsangebot erzwungen werden (z. B. bei beitragsrelevanten Änderungen), oder kann der Benutzer frei wählen?
- Wie wird mit Nachträgen umgegangen, die den Hauptfälligkeitstermin verändern? Muss das Inkasso-System komplett neue Zahlungspläne erhalten?
- Gibt es einen Genehmigungsworkflow für besonders große Beitragsänderungen (z. B. > 50 % Erhöhung)?
- Soll der Kunde den Status seiner Änderung über das Kundenportal verfolgen können?
- Wie wird mit mehreren aufeinanderfolgenden, noch nicht policieren Änderungen umgegangen – Stapelverarbeitung oder streng sequentiell?
- Sollen automatische Änderungen (z. B. SF-Hochstufung) auch eine Beitragsdifferenz-Ansicht erzeugen oder nur im Hintergrund policiert werden?
