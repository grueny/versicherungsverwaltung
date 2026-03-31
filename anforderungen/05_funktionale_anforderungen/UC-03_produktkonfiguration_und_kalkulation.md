# UC-03: Produktkonfiguration und Kalkulation einstellen

## Beschreibung
Berechtigte Mitarbeiter (z. B. Produktmanager, Aktuare) können die **Produkte, Deckungsbausteine, Tarifmerkmale und Kalkulationsparameter** einer Sparte konfigurieren und anpassen. Änderungen werden zunächst als **Entwurf** erfasst und können mit einer **Simulation** validiert werden, bevor sie aktiviert werden. Die Simulation berechnet Beispielbeiträge mit den neuen Parametern, ohne die produktive Konfiguration zu verändern.

Dies ist ein **spartenübergreifender** Verwaltungsprozess. Jede Sparte pflegt ihre eigenen Produkte und Kalkulationsparameter über dieselbe Oberfläche und dieselben Mechanismen. Die konfigurierbare Datenstruktur ist im Datenmodell (→ Abschnitte 2.1–2.6) definiert.

## Akteure
- **Primär:** Produktmanager, Aktuar (Kalkulationsverantwortlicher)
- **Sekundär:** Abteilungsleiter (Freigabe), System-Administrator (Notfallzugriff)

## Vorbedingungen
- Benutzer ist am System angemeldet und besitzt mindestens die Kompetenz **`KONFIGURATION_BEARBEITEN`** (→ Kompetenz-System S8)
- Die zu bearbeitende Sparte ist im System registriert und hat den Status `AKTIV` oder `GEPLANT`
- Aktuelle Produktkonfiguration der Sparte ist im System vorhanden (mindestens eine Basisversion)

## Auslöser
- Benutzer navigiert im Admin-Bereich zu „Produktkonfiguration" und wählt eine Sparte aus
- Alternativ: Batch-Import über Datei-Upload (CSV / JSON) für initiale Konfiguration oder Massenänderungen

## Ablauf (Hauptszenario)

### Phase 1: Sparte und Konfigurationsbereich auswählen
1. Benutzer öffnet den Admin-Bereich „Produktkonfiguration"
2. System prüft die Kompetenz des Benutzers über das Kompetenz-System (S8)
3. System zeigt die verfügbaren Sparten an, für die der Benutzer berechtigt ist
4. Benutzer wählt eine Sparte aus
5. System zeigt die aktuelle Produktkonfiguration der Sparte in einer Übersicht:
   - **Produkte** (Hauptprodukte, Zusatzbausteine, Kurzfrist-Produkte)
   - **Deckungsbausteine** je Produkt
   - **Tarifmerkmale** je Produkt
   - **Produktabhängigkeiten** (Erfordert, Enthält, Ausschließt, Voraussetzung)
   - **Aussteuerungsregeln**

### Phase 2: Konfigurationsversion erstellen (Entwurf)
6. Benutzer wählt „Neue Version erstellen"
7. System erzeugt eine **Konfigurationsversion** im Status **„Entwurf"** auf Basis der aktuellen aktiven Version (Kopie)
8. System vergibt eine eindeutige Versions-ID und erfasst den Ersteller sowie den Erstellzeitpunkt
9. Benutzer legt das geplante **Gültig-ab-Datum** fest (= Zeitpunkt, ab dem die neue Konfiguration gelten soll)

### Phase 3: Konfiguration bearbeiten
10. Benutzer bearbeitet die gewünschten Konfigurationsbereiche:

#### 3a: Produkte verwalten
- Neues Produkt anlegen (Bezeichnung, Produkttyp, Gültigkeitszeitraum)
- Bestehendes Produkt bearbeiten (Bezeichnung, Status, Gültigkeitszeitraum)
- Produkt deaktivieren (Status → `INAKTIV`, nur wenn keine laufenden Verträge betroffen oder mit Übergangsfrist)

#### 3b: Deckungsbausteine verwalten
- Deckungsbaustein anlegen / bearbeiten (Bezeichnung, Versicherungssumme min/max, Pflicht/Optional)
- Deckungsbaustein einem Produkt zuordnen oder entfernen
- Selbstbeteiligung-Optionen konfigurieren

#### 3c: Tarifmerkmale verwalten
- Tarifmerkmal anlegen / bearbeiten (Merkmalname, Datentyp, Wertebereich als JSONB, Einfluss auf Prämie)
- Tarifmerkmale einem Produkt zuordnen
- **Tariftabellen** pflegen: Zuordnung von Merkmalskombinationen zu Beitragsfaktoren
- Zu-/Abschläge konfigurieren (prozentual oder absolut)

#### 3d: Produktabhängigkeiten verwalten
- Abhängigkeit anlegen: Quellprodukt ↔ Zielprodukt mit Typ (`ERFORDERT`, `ENTHAELT`, `AUSSCHLIESST`, `VORAUSSETZUNG`)
- Abhängigkeit entfernen
- System prüft auf Zirkularität und Widersprüche

#### 3e: Aussteuerungsregeln verwalten
- Regel anlegen / bearbeiten (Name, Bedingungsausdruck in Drools DRL oder SpEL, Aktion, Priorität)
- Regel aktivieren / deaktivieren
- Syntax-Validierung des Bedingungsausdrucks in Echtzeit

#### 3f: Basisbeiträge und Scoringfaktoren verwalten (KFZ)
- **Basisbeiträge** je Produkt und Fahrzeugart anlegen / bearbeiten (→ Entity `KfzBasisbeitrag`)
  - Tabellenansicht: Zeilen = Fahrzeugarten, Spalten = Produkte (HP / TK / VK)
  - Wert in EUR, muss > 0 sein
- **Scoringfaktoren** je Faktortyp und Schlüsselwert anlegen / bearbeiten (→ Entity `KfzScoringfaktor`)
  - Tabellenansicht je Faktortyp (SF-Klasse, Typklasse, Regionalklasse, Fahrerkreis, Fahrleistung, Alter, Stellplatz, Nutzungsart, Selbstbeteiligung)
  - Produktübergreifende Faktoren: eine Spalte (Faktorwert)
  - Produktspezifische Faktoren: Spalten je Produkt (HP / TK / VK)
  - Wert als Multiplikator (1,0000 = neutral); muss > 0 sein
- **Massenänderung:** Prozentuale Anpassung aller Faktoren eines Typs (z. B. „alle SF-Faktoren HP +2 %")
- System prüft bei jeder Änderung: Wert > 0, Vollständigkeit der Faktor-Schlüssel, keine doppelten Einträge
- Gültigkeitsdatum wird von der Konfigurationsversion übernommen

11. System führt bei jeder Änderung **Plausibilitätsprüfungen** durch:
    - Pflichtfelder vollständig?
    - Wertebereich valide?
    - Keine widersprüchlichen Abhängigkeiten?
    - Gültigkeitszeiträume konsistent?
12. System zeigt Validierungsfehler und Warnungen in Echtzeit an

### Phase 4: Simulation
13. Benutzer wählt die Funktion **„Simulation starten"** (erfordert Kompetenz **`KONFIGURATION_SIMULIEREN`**)
14. System zeigt eine Simulationsansicht mit Eingabemöglichkeiten für **Testfälle**:
    - Benutzer kann Testfälle manuell anlegen (Kundenprofil, gewählte Produkte, Merkmalsausprägungen)
    - Benutzer kann **bestehende Verträge** als Simulationsbasis auswählen (anonymisiert)
    - Benutzer kann vordefinierte **Standardtestfälle** der Sparte verwenden
15. Benutzer löst die **Simulationsberechnung** aus
16. System berechnet die Beiträge mit den **Entwurfs-Parametern** (neue Konfigurationsversion), ohne die produktive Konfiguration zu beeinflussen
17. System zeigt die Simulationsergebnisse in einer Vergleichsansicht:
    - **Alt-Berechnung** (aktive Konfiguration) ↔ **Neu-Berechnung** (Entwurf)
    - Differenz absolut und prozentual je Testfall
    - Aggregierte Statistik: Durchschnittliche Beitragsänderung, Min/Max, Anzahl betroffener Konstellationen
18. Benutzer kann Testfälle anpassen und die Simulation erneut durchführen
19. Simulationsergebnisse können als **Report exportiert** werden (CSV, PDF über Druckschnittstelle S5)

### Phase 5: Freigabe und Aktivierung
20. Benutzer wählt die Funktion **„Zur Freigabe einreichen"**
21. System prüft, ob alle Pflichtfelder befüllt und alle Validierungen bestanden sind
22. Konfigurationsversion wechselt zu Status **„Eingereicht"**
23. Ein Benutzer mit Kompetenz **`KONFIGURATION_FREIGEBEN`** (z. B. Abteilungsleiter) prüft die Version:
    - Einsicht in die Änderungen (Diff-Ansicht: Alt vs. Neu)
    - Einsicht in die Simulationsergebnisse
24. Freigeber entscheidet:
    - **Freigeben** → Status wechselt zu **„Freigegeben"**
    - **Ablehnen** → Status wechselt zu **„Abgelehnt"**, mit Begründung; Bearbeiter kann den Entwurf überarbeiten
25. Bei Freigabe: System plant die **Aktivierung** zum konfigurierten Gültig-ab-Datum
    - Liegt das Datum in der Zukunft → Konfiguration wird als **„Geplant"** markiert und zum Stichtag automatisch aktiviert (Scheduler)
    - Liegt das Datum in der Vergangenheit oder ist „sofort" → Konfiguration wird **unmittelbar** aktiviert
26. System erstellt einen **Audit-Eintrag** (Hibernate Envers) über alle Änderungen

## Alternativszenarien

- **A1: Entwurf zwischenspeichern**
  Der Benutzer kann den Entwurf jederzeit speichern und später weiterbearbeiten. Der Status bleibt „Entwurf".

- **A2: Entwurf verwerfen**
  Der Benutzer kann einen Entwurf verwerfen. System fragt: „Entwurf wirklich verwerfen?" Bei Bestätigung wird der Entwurf auf Status **„Verworfen"** gesetzt (nicht physisch gelöscht – Nachvollziehbarkeit).

- **A3: Konfigurationsversion kopieren**
  Der Benutzer kann eine bestehende Version (auch abgelehnte oder verworfene) als Basis für einen neuen Entwurf verwenden, um Nachbesserungen effizient umzusetzen.

- **A4: Batch-Import**
  Der Benutzer lädt eine Konfigurationsdatei (CSV / JSON) hoch. System validiert die Daten und erstellt daraus einen Entwurf. Validierungsfehler werden in einem Import-Protokoll angezeigt.

- **A5: Notfallaktivierung (ohne Simulation)**
  Bei kritischen Tarifkorrekturen kann ein Benutzer mit Kompetenz **`KONFIGURATION_NOTFALL`** eine Version ohne Simulation direkt freigeben und aktivieren. Jede Notfallaktivierung wird gesondert im Audit-Log protokolliert.

- **A6: Rollback auf vorherige Version**
  Bei Fehlern nach Aktivierung kann ein Benutzer mit Kompetenz `KONFIGURATION_FREIGEBEN` ein Rollback auf die vorherige aktive Version auslösen. System erzeugt dafür eine neue Konfigurationsversion als Kopie der Vorgängerversion.

- **A7: Massenänderung Tarifmerkmale**
  Der Benutzer kann Tarifmerkmale per Tabellenansicht bearbeiten (z. B. prozentuale Anpassung aller Beitragsfaktoren um +3 %) statt einzelner Pflege.

## Fehlerfälle

- **F1: Fehlende Berechtigung**
  Benutzer besitzt nicht die erforderliche Kompetenz. → System zeigt Fehlermeldung: „Sie verfügen nicht über die erforderliche Berechtigung für diese Aktion." Zugriff wird verweigert.

- **F2: Validierungsfehler beim Einreichen**
  Der Entwurf enthält unvollständige oder fehlerhafte Einträge. → System zeigt alle Validierungsfehler an. Einreichung wird blockiert, bis alle Fehler behoben sind.

- **F3: Zirkuläre Produktabhängigkeit**
  Benutzer legt eine Abhängigkeit an, die eine zirkuläre Kette erzeugt (A erfordert B erfordert A). → System erkennt die Zirkularität und verhindert das Speichern mit entsprechender Fehlermeldung.

- **F4: Widersprüchliche Abhängigkeiten**
  Produkt A „erfordert" Produkt B, gleichzeitig „schließt aus" Produkt B. → System erkennt den Widerspruch und verhindert das Speichern.

- **F5: Konfliktierende Gültigkeitszeiträume**
  Zwei aktive Konfigurationsversionen überlappen sich im Gültigkeitszeitraum. → System verhindert die Freigabe und zeigt den Konflikt an.

- **F6: Simulationsberechnung fehlgeschlagen**
  Ein Testfall kann mit den neuen Parametern nicht berechnet werden (z. B. Division durch Null, fehlende Tariftabelle). → System zeigt den fehlerhaften Testfall mit Fehlerbeschreibung an. Simulation kann für die übrigen Testfälle fortgeführt werden.

- **F7: Aktivierung fehlgeschlagen**
  Der Scheduler kann die geplante Aktivierung nicht durchführen (z. B. DB-Fehler). → System erzeugt einen Alert an den Administrator. Alte Konfiguration bleibt aktiv. Manuelle Wiederholung möglich.

- **F8: Import-Fehler**
  Die hochgeladene Datei enthält ungültige Struktur oder Daten. → System zeigt ein Import-Protokoll mit Zeilennummern und Fehlerbeschreibungen. Kein Entwurf wird erstellt.

## Geschäftsregeln

| Regel-ID | Beschreibung | Quelle |
|----------|-------------|--------|
| GR-PK-01 | Nur Benutzer mit Kompetenz `KONFIGURATION_BEARBEITEN` dürfen Entwürfe erstellen und bearbeiten | UC-03 |
| GR-PK-02 | Nur Benutzer mit Kompetenz `KONFIGURATION_SIMULIEREN` dürfen Simulationen durchführen | UC-03 |
| GR-PK-03 | Nur Benutzer mit Kompetenz `KONFIGURATION_FREIGEBEN` dürfen Versionen freigeben – **4-Augen-Prinzip**: Ersteller ≠ Freigeber | UC-03 |
| GR-PK-04 | Eine Konfigurationsversion muss ein geplantes Gültig-ab-Datum haben, das in der Zukunft oder gleich dem aktuellen Datum liegt | UC-03 |
| GR-PK-05 | Es darf pro Sparte zu jedem Zeitpunkt maximal eine aktive Konfigurationsversion geben (keine überlappenden Gültigkeitszeiträume) | UC-03 |
| GR-PK-06 | Ein Produkt darf nur deaktiviert werden, wenn keine laufenden Verträge mit Status `AKTIV` dieses Produkt verwenden – oder eine Übergangsfrist definiert ist | UC-03 |
| GR-PK-07 | Produktabhängigkeiten dürfen keine zirkulären Ketten bilden | UC-03 |
| GR-PK-08 | Widersprüchliche Abhängigkeiten (`ERFORDERT` + `AUSSCHLIESST` zwischen denselben Produkten) sind nicht zulässig | UC-03 |
| GR-PK-09 | Das Gültig-ab-Datum einer neuen Version muss nach dem Gültig-ab-Datum der aktuell aktiven Version liegen | UC-03 |
| GR-PK-10 | Simulationsberechnungen verwenden ausschließlich die Entwurfs-Parameter und dürfen keine produktiven Daten verändern | UC-03 |
| GR-PK-11 | Notfallaktivierungen (ohne Simulation) erfordern die Kompetenz `KONFIGURATION_NOTFALL` und werden gesondert im Audit-Log dokumentiert | UC-03 |
| GR-PK-12 | Alle Konfigurationsänderungen werden vollständig historisiert (Hibernate Envers) – Wer hat wann was geändert | UC-03 |

## Daten (Ein-/Ausgabe)

### Eingabedaten – Konfigurationsversion

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| sparte_id | UUID | ✅ | Existierende Sparte im Status AKTIV oder GEPLANT | Zu konfigurierende Sparte |
| gueltig_ab | Date | ✅ | ≥ heute; > gueltig_ab der aktiven Version | Geplanter Aktivierungszeitpunkt |
| bezeichnung | String(200) | ✅ | Nicht leer | Beschreibende Bezeichnung der Version (z. B. „Tarifanpassung Q3 2027") |
| kommentar | String(2000) | ❌ | – | Optionaler Änderungskommentar |

### Eingabedaten – Produkt (innerhalb der Version)

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| bezeichnung | String(200) | ✅ | Nicht leer | Produktbezeichnung |
| produkttyp | Enum | ✅ | HAUPTPRODUKT, ZUSATZBAUSTEIN, KURZFRISTIG | Art des Produkts |
| status | Enum | ✅ | AKTIV, INAKTIV, GEPLANT | Produktstatus |
| gueltig_ab | Date | ✅ | ≥ Versions-Gültigkeitsdatum | Produktgültigkeit ab |
| gueltig_bis | Date | ❌ | > gueltig_ab | Produktgültigkeit bis (optional) |

### Eingabedaten – Deckungsbaustein

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| bezeichnung | String(200) | ✅ | Nicht leer | Name des Bausteins |
| versicherungssumme_min | BigDecimal(12,2) | ❌ | ≥ 0 | Mindest-Versicherungssumme |
| versicherungssumme_max | BigDecimal(12,2) | ❌ | ≥ min | Höchst-Versicherungssumme |
| pflicht_optional | Enum | ✅ | PFLICHT, OPTIONAL | Pflichtbaustein oder wählbar |
| selbstbeteiligung_optionen | JSONB | ❌ | Valides JSON-Array | Verfügbare SB-Stufen |

### Eingabedaten – Tarifmerkmal

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| merkmal_name | String(100) | ✅ | Eindeutig innerhalb Produkt | Technischer Merkmalname |
| datentyp | Enum | ✅ | STRING, INTEGER, DECIMAL, BOOLEAN, DATE | Datentyp des Merkmals |
| wertebereich | JSONB | ❌ | Valides JSON | zulässige Werte / Bereiche |
| einfluss_praemie | Enum | ✅ | HOCH, MITTEL, NIEDRIG | Einfluss auf Beitragsberechnung |
| beitragsfaktoren | JSONB | ✅ | Valides JSON | Zuordnung Merkmalswert → Faktor |

### Eingabedaten – Basisbeitrag (KFZ, Phase 3f)

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| produkt_id | UUID | ✅ | Existierendes KFZ-Produkt (HP/TK/VK) | Zugeordnetes Produkt |
| fahrzeugart | Enum | ✅ | Gültige Fahrzeugart aus 5.13 | Fahrzeugart |
| basisbeitrag | BigDecimal(10,2) | ✅ | > 0 | Jahresbasisbeitrag in EUR |

### Eingabedaten – Scoringfaktor (KFZ, Phase 3f)

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| faktor_typ | Enum | ✅ | Gültiger ScoringfaktorTyp aus 5.17 | Kategorie des Faktors |
| produkt_id | UUID | Bedingt | NULL bei produktübergreifend, gesetzt bei SF/Typ/Reg/SB | Produktbezug |
| faktor_schluessel | String(50) | ✅ | Nicht leer, passt zum faktor_typ | Schlüsselwert (z. B. „SF5", „TK15", „GARAGE") |
| faktor_wert | BigDecimal(6,4) | ✅ | > 0 | Multiplikator (1,0000 = neutral) |

### Eingabedaten – Simulation (Testfall)

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| testfall_bezeichnung | String(200) | ✅ | Nicht leer | Name des Testfalls |
| produkt_ids | UUID[] | ✅ | Mindestens 1 Produkt | Gewählte Produkte |
| merkmalsauspraegungen | JSONB | ✅ | Valide Merkmalswerte | Testfall-Merkmalswerte |
| referenz_vertrag_id | UUID | ❌ | Existierender Vertrag | Bestehender Vertrag als Vorlage |

### Ausgabedaten – Simulationsergebnis

| Feld | Typ | Beschreibung |
|------|-----|-------------|
| testfall_bezeichnung | String | Name des Testfalls |
| beitrag_alt | BigDecimal(12,2) | Berechneter Beitrag mit aktiver Konfiguration |
| beitrag_neu | BigDecimal(12,2) | Berechneter Beitrag mit Entwurfs-Konfiguration |
| differenz_absolut | BigDecimal(12,2) | beitrag_neu − beitrag_alt |
| differenz_prozent | BigDecimal(5,2) | Prozentuale Abweichung |
| berechnungsdetails | JSONB | Aufschlüsselung der Beitragsbestandteile |
| fehler | String | Fehlermeldung (falls Testfall nicht berechenbar) |

### Ausgabedaten – Simulationsstatistik (aggregiert)

| Feld | Typ | Beschreibung |
|------|-----|-------------|
| anzahl_testfaelle | Integer | Gesamtanzahl Testfälle |
| anzahl_erfolgreich | Integer | Erfolgreich berechnete Testfälle |
| anzahl_fehlerhaft | Integer | Nicht berechenbare Testfälle |
| durchschnitt_differenz_prozent | BigDecimal(5,2) | Ø prozentuale Beitragsänderung |
| min_differenz_prozent | BigDecimal(5,2) | Minimale Beitragsänderung |
| max_differenz_prozent | BigDecimal(5,2) | Maximale Beitragsänderung |

## API-Endpunkte

> Ergänzt die bestehende Produktkonfigurations-API (→ Schnittstellen 2.6) um schreibende und Simulations-Endpunkte.

### Konfigurationsversionen

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| GET | `/api/v1/sparten/{sparte_id}/konfigurationen` | `KONFIGURATION_LESEN` | Alle Versionen einer Sparte auflisten |
| GET | `/api/v1/sparten/{sparte_id}/konfigurationen/{version_id}` | `KONFIGURATION_LESEN` | Version im Detail anzeigen |
| POST | `/api/v1/sparten/{sparte_id}/konfigurationen` | `KONFIGURATION_BEARBEITEN` | Neue Version als Entwurf erstellen (Kopie der aktiven Version) |
| PUT | `/api/v1/sparten/{sparte_id}/konfigurationen/{version_id}` | `KONFIGURATION_BEARBEITEN` | Entwurf aktualisieren |
| DELETE | `/api/v1/sparten/{sparte_id}/konfigurationen/{version_id}` | `KONFIGURATION_BEARBEITEN` | Entwurf verwerfen (logisches Löschen → Status „Verworfen") |
| GET | `/api/v1/sparten/{sparte_id}/konfigurationen/{version_id}/diff` | `KONFIGURATION_LESEN` | Diff-Ansicht: Entwurf vs. aktive Version |

### Freigabe-Workflow

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| POST | `/api/v1/sparten/{sparte_id}/konfigurationen/{version_id}/einreichen` | `KONFIGURATION_BEARBEITEN` | Version zur Freigabe einreichen |
| POST | `/api/v1/sparten/{sparte_id}/konfigurationen/{version_id}/freigeben` | `KONFIGURATION_FREIGEBEN` | Version freigeben (4-Augen-Prinzip) |
| POST | `/api/v1/sparten/{sparte_id}/konfigurationen/{version_id}/ablehnen` | `KONFIGURATION_FREIGEBEN` | Version ablehnen (mit Begründung) |
| POST | `/api/v1/sparten/{sparte_id}/konfigurationen/{version_id}/aktivieren` | `KONFIGURATION_FREIGEBEN` | Sofortige Aktivierung (bei sofort-Gültigkeit) |
| POST | `/api/v1/sparten/{sparte_id}/konfigurationen/{version_id}/rollback` | `KONFIGURATION_FREIGEBEN` | Rollback auf Vorgängerversion |

### Simulation

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| POST | `/api/v1/sparten/{sparte_id}/konfigurationen/{version_id}/simulation` | `KONFIGURATION_SIMULIEREN` | Simulation mit Testfällen starten |
| GET | `/api/v1/sparten/{sparte_id}/konfigurationen/{version_id}/simulation/{sim_id}` | `KONFIGURATION_SIMULIEREN` | Simulationsergebnis abrufen |
| POST | `/api/v1/sparten/{sparte_id}/konfigurationen/{version_id}/simulation/{sim_id}/export` | `KONFIGURATION_SIMULIEREN` | Simulationsergebnis als Report exportieren |

### Import

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| POST | `/api/v1/sparten/{sparte_id}/konfigurationen/import` | `KONFIGURATION_BEARBEITEN` | Konfiguration per Datei-Upload importieren |

### Basisbeiträge und Scoringfaktoren (KFZ)

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| GET | `/api/v1/sparten/{sparte_id}/konfigurationen/{version_id}/basisbeitraege` | `KONFIGURATION_LESEN` | Alle Basisbeiträge der Version |
| PUT | `/api/v1/sparten/{sparte_id}/konfigurationen/{version_id}/basisbeitraege` | `KONFIGURATION_BEARBEITEN` | Basisbeiträge aktualisieren (Batch) |
| GET | `/api/v1/sparten/{sparte_id}/konfigurationen/{version_id}/scoringfaktoren` | `KONFIGURATION_LESEN` | Alle Scoringfaktoren (Filter: faktor_typ, produkt_id) |
| PUT | `/api/v1/sparten/{sparte_id}/konfigurationen/{version_id}/scoringfaktoren` | `KONFIGURATION_BEARBEITEN` | Scoringfaktoren aktualisieren (Batch) |
| POST | `/api/v1/sparten/{sparte_id}/konfigurationen/{version_id}/scoringfaktoren/massenanpassung` | `KONFIGURATION_BEARBEITEN` | Prozentuale Massenanpassung (Body: faktor_typ, produkt_id?, prozent) |

## Events

| Event | Auslöser | Payload | Konsument |
|-------|----------|---------|-----------|
| `KonfigurationErstelltEvent` | Neuer Entwurf erstellt | sparte_id, version_id, ersteller | Audit-Log |
| `KonfigurationEingereichtEvent` | Version zur Freigabe eingereicht | sparte_id, version_id, ersteller | Freigabe-Dashboard, Benachrichtigung an Freigeber |
| `KonfigurationFreigegebenEvent` | Version freigegeben | sparte_id, version_id, freigeber, gueltig_ab | Scheduler (Aktivierungs-Planung), Audit-Log |
| `KonfigurationAbgelehntEvent` | Version abgelehnt | sparte_id, version_id, freigeber, begruendung | Benachrichtigung an Bearbeiter |
| `KonfigurationAktiviertEvent` | Version aktiviert (produktiv) | sparte_id, version_id, gueltig_ab | Sparten-Registry (Cache invalidieren), DWH (S4), alle Berechnungsservices |
| `KonfigurationRollbackEvent` | Rollback auf Vorgängerversion | sparte_id, alte_version_id, neue_version_id | Sparten-Registry, Audit-Log |
| `SimulationAbgeschlossenEvent` | Simulation beendet | sparte_id, version_id, sim_id, anzahl_testfaelle, ergebnis_zusammenfassung | Audit-Log |

## Statusmodell – Konfigurationsversion

```
                    ┌──────────┐
                    │ Entwurf  │
                    └────┬─────┘
                         │ einreichen
                    ┌────▼──────────┐
              ┌─────│  Eingereicht  │─────┐
              │     └───────────────┘     │
              │ freigeben                 │ ablehnen
        ┌─────▼──────┐            ┌──────▼──────┐
        │ Freigegeben │            │  Abgelehnt  │ ─── überarbeiten ──→ Entwurf
        └─────┬──────┘            └─────────────┘
              │ aktivieren (Stichtag)
        ┌─────▼──────┐
        │   Aktiv    │
        └─────┬──────┘
              │ neue Version aktiviert
        ┌─────▼──────┐
        │  Abgelöst  │
        └────────────┘

        Sonderstatus: Verworfen (aus Entwurf heraus)
```

## Nachbedingungen
- Die neue Konfigurationsversion ist im System persistiert und historisiert (Hibernate Envers)
- Bei Aktivierung: Die Sparten-Registry liefert die neuen Konfigurationsdaten für Angebots- und Antragsprozesse (UC-01, UC-02)
- Vorherige aktive Version erhält den Status „Abgelöst" (bleibt für historische Berechnungen und Nachvollziehbarkeit erhalten)
- Alle Änderungen sind im Audit-Trail dokumentiert (Benutzer, Zeitstempel, Änderungsumfang)
- Simulationsergebnisse sind dem Entwurf zugeordnet und bleiben nach Aktivierung einsehbar

## Neue Kompetenzen (→ Kompetenz-System S8)

| Kompetenz | Beschreibung | Typische Rolle |
|-----------|-------------|----------------|
| `KONFIGURATION_LESEN` | Produktkonfiguration einsehen (bereits vorhanden) | Alle Innendienst-Mitarbeiter |
| `KONFIGURATION_BEARBEITEN` | Entwürfe erstellen, bearbeiten, einreichen | Produktmanager, Aktuar |
| `KONFIGURATION_SIMULIEREN` | Simulationen durchführen und Ergebnisse einsehen | Produktmanager, Aktuar |
| `KONFIGURATION_FREIGEBEN` | Versionen freigeben und aktivieren, Rollback | Abteilungsleiter, Lead-Aktuar |
| `KONFIGURATION_NOTFALL` | Notfallaktivierung ohne Simulation | System-Administrator, Abteilungsleiter |

## Akzeptanzkriterien
- [ ] Nur Benutzer mit `KONFIGURATION_BEARBEITEN` können Entwürfe erstellen und bearbeiten
- [ ] Neue Konfigurationsversionen werden als Kopie der aktuellen aktiven Version erstellt
- [ ] Alle Konfigurationsbereiche (Produkte, Deckungsbausteine, Tarifmerkmale, Abhängigkeiten, Aussteuerungsregeln) sind bearbeitbar
- [ ] Zirkuläre und widersprüchliche Produktabhängigkeiten werden erkannt und verhindert
- [ ] Simulation berechnet Beiträge mit Entwurfs-Parametern ohne Änderung der produktiven Konfiguration
- [ ] Simulationsergebnisse zeigen Alt/Neu-Vergleich mit absoluter und prozentualer Differenz
- [ ] Bestehende Verträge können anonymisiert als Simulationsbasis verwendet werden
- [ ] Freigabe erfordert 4-Augen-Prinzip (Ersteller ≠ Freigeber)
- [ ] Aktivierung erfolgt zum konfigurierten Gültig-ab-Datum (Scheduler) oder sofort
- [ ] Nach Aktivierung liefert die Sparten-Registry die neue Konfiguration für UC-01 und UC-02
- [ ] Rollback auf die vorherige aktive Version ist möglich
- [ ] Alle Änderungen sind vollständig über Hibernate Envers historisiert
- [ ] Batch-Import (CSV/JSON) erzeugt einen neuen Entwurf mit Import-Protokoll bei Fehlern
- [ ] Notfallaktivierung wird gesondert im Audit-Log protokolliert
- [ ] KFZ-Basisbeiträge und Scoringfaktoren sind innerhalb einer Konfigurationsversion bearbeitbar
- [ ] Simulation verwendet die Basisbeiträge und Scoringfaktoren der Entwurfs-Version
- [ ] Massenanpassung (prozentual) ist für Scoringfaktoren je Faktortyp möglich

## Wireframe / Skizze

```
┌──────────────────────────────────────────────────────────────────┐
│  Produktkonfiguration                          [Sparte: KFZ ▼]  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Versionen                     ┌─ Aktive Version: v2.3 (01.01.) │
│  ┌─────────────────────────┐   │  Geplant:        v2.4 (01.07.) │
│  │ ● v2.4  Entwurf    [✏] │   │                                 │
│  │ ○ v2.3  Aktiv      [👁] │   └─────────────────────────────────│
│  │ ○ v2.2  Abgelöst   [👁] │                                    │
│  └─────────────────────────┘                                     │
│                                                                  │
│  ┌─ Tabs ──────────────────────────────────────────────────────┐ │
│  │ Produkte │ Deckungsbausteine │ Tarifmerkmale │ Abhäng. │ ▾ │ │
│  ├──────────────────────────────────────────────────────────────┤ │
│  │                                                              │ │
│  │  Produkte der Version v2.4 (Entwurf)                        │ │
│  │  ┌─────────────┬──────────────┬──────────┬────────────────┐ │ │
│  │  │ Bezeichnung │ Produkttyp   │ Status   │ Gültig ab      │ │ │
│  │  ├─────────────┼──────────────┼──────────┼────────────────┤ │ │
│  │  │ KFZ-Haftpfl.│ HAUPTPRODUKT │ AKTIV    │ 01.01.2027     │ │ │
│  │  │ Teilkasko   │ ZUSATZBAUST. │ AKTIV    │ 01.01.2027     │ │ │
│  │  │ Vollkasko   │ ZUSATZBAUST. │ GEPLANT  │ 01.07.2027     │ │ │
│  │  └─────────────┴──────────────┴──────────┴────────────────┘ │ │
│  │                              [+ Produkt hinzufügen]          │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │ [💾 Speichern] [▶ Simulation] [📤 Einreichen] [🗑 Verwer.] │ │
│  └──────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│  Simulation – v2.4 (Entwurf)                                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Testfälle                         Statistik                     │
│  ┌─────────────────────────────┐   ┌──────────────────────────┐  │
│  │ ✅ PKW Golf Std.   +2,3 %  │   │ Ø Änderung:    +1,8 %   │  │
│  │ ✅ PKW BMW Premium +1,5 %  │   │ Min:           -0,5 %   │  │
│  │ ❌ Motorrad (Fehler)       │   │ Max:           +3,2 %   │  │
│  │ ✅ Wohnmobil       -0,5 %  │   │ Erfolgreich:   3 / 4    │  │
│  └─────────────────────────────┘   └──────────────────────────┘  │
│                                                                  │
│  Detail: PKW Golf Std.                                           │
│  ┌──────────────────┬──────────┬──────────┬──────────┐           │
│  │ Beitragsposition │ Alt (€)  │ Neu (€)  │ Diff (€) │           │
│  ├──────────────────┼──────────┼──────────┼──────────┤           │
│  │ Haftpflicht      │   245,00 │   250,00 │    +5,00 │           │
│  │ Teilkasko        │   120,00 │   124,50 │    +4,50 │           │
│  │ Gesamt           │   365,00 │   374,50 │    +9,50 │           │
│  └──────────────────┴──────────┴──────────┴──────────┘           │
│                                                                  │
│  [+ Testfall] [🔄 Erneut berechnen] [📄 Report exportieren]     │
└──────────────────────────────────────────────────────────────────┘
```

## Datenmodell-Erweiterung

> Die folgenden Entitäten ergänzen das bestehende Datenmodell (→ `07_datenmodell.md`).

### Konfigurationsversion (neu)

| Attribut | Typ | Pflicht | Beschreibung |
|----------|-----|---------|-------------|
| id | UUID | ✅ | Technischer Primärschlüssel |
| sparte_id | UUID (FK) | ✅ | Referenz auf Sparte |
| versionsnummer | String(20) | ✅ | Fachliche Kennung (z. B. „v2.4") |
| bezeichnung | String(200) | ✅ | Beschreibung (z. B. „Tarifanpassung Q3 2027") |
| status | Enum | ✅ | ENTWURF, EINGEREICHT, FREIGEGEBEN, AKTIV, ABGELOEST, ABGELEHNT, VERWORFEN |
| gueltig_ab | Date | ✅ | Geplanter Aktivierungszeitpunkt |
| erstellt_von | String(100) | ✅ | Ersteller (Benutzer-ID) |
| freigegeben_von | String(100) | ❌ | Freigeber (Benutzer-ID) |
| freigabe_zeitpunkt | Timestamp | ❌ | Zeitpunkt der Freigabe |
| ablehnungsgrund | String(2000) | ❌ | Begründung bei Ablehnung |
| kommentar | String(2000) | ❌ | Änderungskommentar |
| erstellt_am | Timestamp | ✅ | Erstellzeitpunkt |
| geaendert_am | Timestamp | ✅ | Letzte Änderung |

**Constraints:**
- `versionsnummer` UNIQUE pro sparte_id
- Max. 1 Version im Status `AKTIV` pro sparte_id zu einem Zeitpunkt
- `erstellt_von` ≠ `freigegeben_von` (4-Augen-Prinzip, GR-PK-03)

**Historisierung:** ✅ Hibernate Envers

### SimulationsLauf (neu)

| Attribut | Typ | Pflicht | Beschreibung |
|----------|-----|---------|-------------|
| id | UUID | ✅ | Technischer Primärschlüssel |
| version_id | UUID (FK) | ✅ | Referenz auf Konfigurationsversion |
| durchgefuehrt_von | String(100) | ✅ | Benutzer-ID |
| durchgefuehrt_am | Timestamp | ✅ | Zeitpunkt |
| anzahl_testfaelle | Integer | ✅ | Gesamtanzahl |
| anzahl_erfolgreich | Integer | ✅ | Erfolgreiche Berechnungen |
| ergebnis_zusammenfassung | JSONB | ✅ | Aggregiertes Ergebnis (Statistik) |
| testfaelle | JSONB | ✅ | Testfall-Eingabedaten und Einzelergebnisse |

**Historisierung:** ✅ (Immutable – kein Update, nur Insert)

## Offene Fragen
- Sollen spartenfremde Benutzer Konfigurationen anderer Sparten im Lesemodus einsehen können?
- Wie detailliert soll die Diff-Ansicht (Alt vs. Neu) sein – auf Feldebene oder nur strukturell?
- Soll es einen Genehmigungs-Workflow mit mehreren Stufen geben (z. B. 2 Freigeber bei besonders kritischen Änderungen)?
- Wie wird der Scheduler für die zeitgesteuerte Aktivierung technisch umgesetzt (Spring @Scheduled, Quartz, DB-basiert)?
- Sollen Simulationen auch für Bestandsverträge als Massensimulation möglich sein (z. B. „Was kostet die Tarifänderung für alle 10.000 KFZ-Verträge")?
- Gibt es Approval-Fristen (z. B. automatisches Zurücksetzen auf „Entwurf", wenn innerhalb von X Tagen nicht freigegeben)?
