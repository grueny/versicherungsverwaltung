# UC-09: Rückwirkende Vertragsänderung und Stornierung durchführen

## Beschreibung
Ein Benutzer führt eine **rückwirkende Änderung** (Nachtrag) oder eine **rückwirkende Stornierung** an einem bestehenden Vertrag durch. Dabei wird ein **Wirksamkeitsdatum in der Vergangenheit** gewählt, das **vor** dem `gueltig_ab` eines oder mehrerer bereits policierter Vertragsstände liegt. Das System muss deshalb **bestehende Vertragsstände invalidieren**, die **Vertragshistorie neu aufbauen** und alle betroffenen **Schnittstellenmeldungen zurückbuchen** (Stornobuchungen an S4, S1, S2, S5) sowie die korrekten Neuberechnungen erneut melden.

Im Gegensatz zu einer vorwärts gerichteten Änderung (UC-05) oder Stornierung (UC-06) greift die rückwirkende Bewegung in die **historische Vertragsstandkette** ein und erzeugt eine **Berichtigungskette**: Storno der falschen Stände → Neuerzeugung der korrekten Stände → erneute Versorgung aller Schnittstellen.

Dies ist ein **spartenübergreifender** Kernprozess. Spartenspezifische Besonderheiten (z. B. SF-Klassen-Neuberechnung bei KFZ, rückwirkende eVB-Korrektur) werden über die Extension Points / Hooks der Spartenkonfiguration eingebunden.

## Akteure
- **Primär:** Innendienst (Sachbearbeitung Senior, Teamleitung)
- **Sekundär:** System (automatische Rückverarbeitung bei Korrekturen)

## Vorbedingungen
- Ein Vertrag im Status **`AKTIV`**, **`GEKUENDIGT`**, **`STORNIERT`** oder **`BEENDET`** existiert mit mindestens einem policierten Vertragsstand
- Benutzer ist am System angemeldet und besitzt die Kompetenz **`RUECKWIRKEND_AENDERN`** (für rückwirkende Änderung) oder **`RUECKWIRKEND_STORNIEREN`** (für rückwirkende Stornierung) (→ Kompetenz-System S8)
- Es existiert keine offene rückwirkende Bewegung für diesen Vertrag
- Das gewünschte Wirksamkeitsdatum liegt **nach** dem Vertragsbeginn

## Auslöser
- Benutzer öffnet einen Vertrag und wählt „Rückwirkende Änderung" oder „Rückwirkende Stornierung"
- Innendienst korrigiert einen fehlerhaften Vertragsstand, der bereits policiert wurde
- Gerichtliche Entscheidung oder behördliche Anordnung erfordert rückwirkende Korrektur
- Nachträgliche Erkenntnis einer vorvertraglichen Anzeigepflichtverletzung (rückwirkende Anfechtung)

## Bewegungstypen

| Bewegungstyp | Beschreibung | Basis-UC | Typischer Auslöser |
|-------------|-------------|----------|---------------------|
| `RUECKWIRKENDE_AENDERUNG` | Rückwirkende Vertragsänderung – ändert Vertragsdaten ab einem Datum in der Vergangenheit | UC-05 | Fehlerkorrektur, nachträgliche Tarifanpassung, rückwirkende Adressänderung |
| `RUECKWIRKENDE_STORNIERUNG` | Rückwirkender Storno – beendet den Vertrag ab einem Datum in der Vergangenheit | UC-06 | Rückwirkender Widerruf, Anfechtung, Rücktritt, behördliche Korrektur |

## Betroffene-Stände-Ermittlung (Kernkonzept)

Das zentrale Konzept der rückwirkenden Bewegung ist die **Impact-Analyse**: Welche bereits policierten Vertragsstände liegen zeitlich **nach** dem Wirksamkeitsdatum und müssen invalidiert und neu aufgebaut werden?

```
Beispiel: Rückwirkende Änderung zum 01.03.

Vertragsstand-Kette VORHER:
  VS-1: 01.01. – 31.03.  (Neugeschäft)
  VS-2: 01.04. – 30.06.  (Nachtrag 1)
  VS-3: 01.07. – NULL    (Nachtrag 2, aktuell)

Betroffene Stände (gueltig_ab ≥ 01.03.):
  → VS-1 wird geschnitten (gueltig_bis = 28.02.)
  → VS-2 wird invalidiert (komplett nach Wirksamkeitsdatum)
  → VS-3 wird invalidiert (komplett nach Wirksamkeitsdatum)

Vertragsstand-Kette NACHHER:
  VS-1: 01.01. – 28.02.  (Neugeschäft, gekürzt)
  VS-4: 01.03. – 31.03.  (Rückwirkende Änderung, NEU)        ← Neue Daten
  VS-5: 01.04. – 30.06.  (Nachtrag 1, NEU BERECHNET)          ← Nachlauf
  VS-6: 01.07. – NULL    (Nachtrag 2, NEU BERECHNET, aktuell)  ← Nachlauf
```

## Ablauf (Hauptszenario – Rückwirkende Änderung)

### Phase 1: Rückwirkende Bewegung initiieren
1. Benutzer öffnet den Vertrag und wählt **„Rückwirkende Änderung"** oder **„Rückwirkende Stornierung"**
2. System prüft die Kompetenz des Benutzers (`RUECKWIRKEND_AENDERN` bzw. `RUECKWIRKEND_STORNIEREN`) über S8
3. System prüft, ob bereits eine **offene rückwirkende Bewegung** für diesen Vertrag existiert:
   - Ja → Hinweis und Verweis auf die bestehende Bewegung
   - Nein → weiter
4. System zeigt das **Wirksamkeitsdatum-Formular** an:
   - Benutzer wählt das **Wirksamkeitsdatum** (in der Vergangenheit)
   - Benutzer erfasst den **Grund** (aus Grundkatalog + Freitext)
5. System validiert das Wirksamkeitsdatum:
   - ≥ Vertragsbeginn
   - < heute (sonst ist es keine rückwirkende Bewegung → Verweis auf UC-05/UC-06)
   - Innerhalb der konfigurierbaren **maximalen Rückwirkungsfrist** (z. B. max. 12 Monate)

### Phase 2: Impact-Analyse
6. System führt die **Impact-Analyse** durch und ermittelt:
   - **Betroffene Vertragsstände**: Alle Vertragsstände mit `gueltig_ab ≥ Wirksamkeitsdatum` oder die das Wirksamkeitsdatum umschließen
   - **Betroffene Vorgänge**: Zugehörige Vorgänge der betroffenen Vertragsstände
   - **Betroffene Schnittstellenmeldungen**: Alle an S4/S1/S2/S5 gemeldeten Bewegungen zu den betroffenen Vertragsständen
   - **Betroffene Inkasso-Forderungen**: Offene und bezahlte Forderungen im betroffenen Zeitraum
   - **Betroffener Zahlungsplan**: Forderungen, die storniert oder angepasst werden müssen
7. System zeigt die **Impact-Übersicht** an:
   - Anzahl betroffener Vertragsstände
   - Zeitraum (Wirksamkeitsdatum → heute)
   - Beitragsauswirkung (geschätzt)
   - Liste der zu stornierenden Schnittstellenmeldungen
   - Warnung bei hoher Anzahl betroffener Stände

### Phase 3: Rückwirkende Daten erfassen

#### Pfad A: Rückwirkende Änderung
8a. System erzeugt eine **Rückwirkende-Änderungsschwebe** im Status „Entwurf"
9a. System zeigt den **Vertragsstand zum Wirksamkeitsdatum** an (= der zu ändernde Stand vor dem Wirksamkeitsdatum)
10a. Benutzer erfasst die gewünschten Änderungen (analog UC-05, Phase 2):
    - Produkte, Deckungsbausteine, Tarifmerkmale, spartenspezifische Daten
11a. → **Hook `vor_Rueckwirkung`** wird ausgelöst (spartenspezifische Prüfungen)
12a. System berechnet den **neuen Beitrag** ab Wirksamkeitsdatum

#### Pfad B: Rückwirkende Stornierung
8b. System erzeugt eine **Rückwirkende-Stornierungsschwebe** im Status „Entwurf"
9b. System zeigt die Stornierungsdaten an (analog UC-06, Phase 2):
    - Stornierungsgrund und Veranlasser
    - Stornierungsdatum = Wirksamkeitsdatum
10b. → **Hook `vor_Rueckwirkung`** wird ausgelöst
11b. System berechnet die **rückwirkende Schlussabrechnung**

### Phase 4: Vorschau & Bestätigung (4-Augen-Prinzip)
13. System erstellt eine **Gesamtvorschau** der rückwirkenden Bewegung:

    **Stornobuchungen (Phase 5):**
    - Liste aller zu invalidierenden Vertragsstände mit Stornobuchungen an S4/S1/S2/S5
    - Bisher gemeldete Beiträge und Provisionen werden storniert

    **Neubuchungen (Phase 6):**
    - Neuer Vertragsstand ab Wirksamkeitsdatum (mit geänderten/stornierten Daten)
    - Neu berechnete Nachlauf-Vertragsstände (falls spätere Vorgänge bestanden)
    - Neue Beitrags-, Provisions-, DWH- und Druckmeldungen

    **Finanzielle Auswirkung:**
    - Beitragsdifferenz gesamt (alter Gesamtbeitrag vs. neuer Gesamtbeitrag über den betroffenen Zeitraum)
    - Erstattung oder Nachforderung an den VN
    - Provisionsrückforderung oder Nachzahlung an den Vermittler

14. System erzwingt das **4-Augen-Prinzip**:
    - Der Ersteller der rückwirkenden Bewegung kann diese nicht selbst freigeben
    - Ein zweiter Benutzer mit Kompetenz `RUECKWIRKEND_FREIGEBEN` muss die Bewegung bestätigen
15. Zweiter Benutzer prüft die Vorschau und wählt **„Rückwirkung freigeben"** oder **„Rückwirkung ablehnen"**

### Phase 5: Stornobuchungen (bestehende Stände invalidieren)
16. System startet die **Stornierungsphase** in einer Transaktion:
17. Für jeden betroffenen Vertragsstand (vom neuesten zum ältesten):
    a. Vertragsstand erhält den Status **`INVALIDIERT`** und das Flag `invalidiert_durch` = Rückwirkungs-ID
    b. Die `gueltig_bis`-Zeiten der Vorgänger-Stände werden **nicht** verändert – die invalidierten Stände bleiben als Audit-Trail erhalten
    c. Der zugehörige Vorgang erhält den Status `STORNIERT_RUECKWIRKEND`
18. System erzeugt **Stornobuchungen** an alle vier Schnittstellen:

| Schnittstelle | Storno-Aktion | Payload-Kern | Mechanismus |
|---------------|--------------|-------------|-------------|
| **S4 – DataWarehouse** | Bewegungs-Storno für jeden invalidierten Vertragsstand | Vertragsnummer, Storno-Referenz (Original-Bewegungs-ID), Stornogrund `RUECKWIRKENDE_KORREKTUR` | Kafka-Topic `vertrag.datawarehouse.bewegungen` (Ereignistyp `STORNO`) |
| **S1 – Provision** | Provisionsrückbuchung für jeden invalidierten Vertragsstand | Vertragsnummer, Storno-Referenz, Provisionsrückforderungsbetrag, Vermittler-ID | Kafka-Topic `vertrag.provision.ereignisse` (Ereignistyp `STORNO`) |
| **S2 – Konzerninkasso** | Forderungsstorno für betroffenen Zeitraum | Vertragsnummer, Stornozeitraum (ab Wirksamkeitsdatum), stornierte Forderungs-IDs, Erstattungsbetrag | Kafka-Topic `vertrag.inkasso.beitragsforderungen` (Ereignistyp `STORNO`) |
| **S5 – Druck** | Druckstorno (falls Dokumente noch nicht versendet) oder Berichtigungsdokument | Vertragsnummer, Storno-Referenz (Original-Druckauftrag-ID), Dokumenttyp `STORNO_BERICHTIGUNG` | REST `POST /api/v1/druck/auftraege` |

19. System protokolliert jede Stornobuchung im **Stornobuchungs-Protokoll** (Referenz auf Original-Meldung + Storno-Meldung)

### Phase 6: Neubuchungen (korrekte Stände erzeugen)
20. → **Hook `vor_Policierung`** wird ausgelöst
21. **Rückwirkender Vertragsstand erzeugen:**
    - Bei rückwirkender Änderung: Neuer Vertragsstand mit geänderten Daten, `gueltig_ab` = Wirksamkeitsdatum
    - Bei rückwirkender Stornierung: Abschließender Vertragsstand mit `gueltig_bis` = Wirksamkeitsdatum, Vertragsstatus → `STORNIERT`/`GEKUENDIGT`/`BEENDET`
22. **Nachlauf-Vertragsstände neu berechnen** (nur bei rückwirkender Änderung):
    - Für jeden invalidierten Vertragsstand, der einen eigenständigen Vorgang hatte (z. B. Nachtrag, Hauptinkasso):
      a. System prüft, ob der ursprüngliche Vorgang noch relevant ist (z. B. Produktänderung)
      b. System berechnet den Vertragsstand **auf Basis des neuen Vorgänger-Standes** neu
      c. System erzeugt einen neuen Vertragsstand (Nachlauf) mit dem ursprünglichen Vorgangstyp
    - Die Nachlauf-Kette wird vom Wirksamkeitsdatum bis zum aktuellen Datum chronologisch aufgebaut
23. System erzeugt **Vorgänge** für jeden neuen Vertragsstand:
    - Rückwirkender Vertragsstand: Vorgangstyp `RUECKWIRKENDE_AENDERUNG` oder `RUECKWIRKENDE_STORNIERUNG`
    - Nachlauf-Vertragsstände: Vorgangstyp `NACHLAUF_NEUBERECHNUNG`
24. → **Hook `nach_Rueckwirkung`** wird ausgelöst (spartenspezifische Nachverarbeitung)

### Phase 7: Schnittstellen-Neubuchungen
25. System publiziert für jeden neuen Vertragsstand (rückwirkend + Nachlauf) die **Neubuchungen** an alle vier Schnittstellen:

| Schnittstelle | Neubuchungs-Aktion | Payload-Kern | Mechanismus |
|---------------|-------------------|-------------|-------------|
| **S4 – DataWarehouse** | Korrigierte Vertragsbewegungen (Neugeschäft-Korrektur, Nachtrag-Korrektur, etc.) | Vertragsnummer, Korrektur-Referenz (Rückwirkungs-ID), Vorgangstyp, neuer Beitrag, Gültigkeitszeitraum | Kafka-Topic `vertrag.datawarehouse.bewegungen` (Ereignistyp `KORREKTUR`) |
| **S1 – Provision** | Korrigierte Provisionsereignisse | Vertragsnummer, Korrektur-Referenz, neuer Beitrag, Provisionsanspruch, Vermittler-ID | Kafka-Topic `vertrag.provision.ereignisse` (Ereignistyp `KORREKTUR`) |
| **S2 – Konzerninkasso** | Korrigierter Zahlungsplan + Differenzforderung/-erstattung | Vertragsnummer, korrigierter Beitrag, Differenz (Erstattung/Nachforderung), neuer Zahlungsplan ab Wirksamkeitsdatum | Kafka-Topic `vertrag.inkasso.beitragsforderungen` (Ereignistyp `KORREKTUR`) |
| **S5 – Druck** | Berichtigungsdokument drucken | Vertragsnummer, Partner, Korrekturbeschreibung, alte + neue Vertragsdaten, Dokumenttyp `BERICHTIGUNG` | REST `POST /api/v1/druck/auftraege` |

26. System aktualisiert den **Vertrag**:
    - `aktueller_jahresbeitrag` = Jahresbeitrag des letzten gültigen Vertragsstands
    - Vertragsstatus ggf. anpassen (bei rückwirkender Stornierung)
27. System setzt die Schwebe auf **„Erledigt"**

### Phase 8: Abschluss
28. System erzeugt eine **Zusammenfassung** der rückwirkenden Bewegung:
    - Anzahl stornierter Vertragsstände
    - Anzahl neu erzeugter Vertragsstände
    - Finanzielle Differential (Erstattung/Nachforderung)
    - Schnittstellenstatus (alle Storno- und Neubuchungen)
29. System publiziert **`RueckwirkendeBewegungAbgeschlossenEvent`** (Spring Event)

## Stornobuchungs-Protokoll (Schnittstellenrückbuchung)

> Jede an eine Schnittstelle gemeldete Bewegung wird beim Storno referenziell verknüpft. Das Protokoll stellt sicher, dass jede Originalmeldung genau eine Stornomeldung erhält und jeder neue Vertragsstand eine Neubuchung auslöst.

| Attribut | Beschreibung |
|----------|-------------|
| rueckwirkungs_id | Referenz auf die rückwirkende Bewegung |
| original_meldung_id | ID der ursprünglichen Schnittstellenmeldung |
| storno_meldung_id | ID der Stornobuchung |
| neubuchung_meldung_id | ID der korrigierten Neubuchung |
| schnittstelle | S1, S2, S4 oder S5 |
| status | STORNO_GESENDET, STORNO_BESTAETIGT, NEUBUCHUNG_GESENDET, NEUBUCHUNG_BESTAETIGT, FEHLERHAFT |

## Alternativszenarien

- **A1: Rückwirkende Bewegung verwerfen**
  Der Ersteller oder der Freigebende kann die Bewegung verwerfen. Alle Vertragsstände bleiben unverändert. Status → „Verworfen".

- **A2: 4-Augen-Freigabe ablehnen**
  Der zweite Benutzer lehnt die rückwirkende Bewegung ab. System dokumentiert den Ablehnungsgrund. Der Ersteller kann die Bewegung anpassen und erneut zur Freigabe einreichen.

- **A3: Rückwirkende Änderung ohne Nachlauf**
  Das Wirksamkeitsdatum liegt nach dem letzten Vorgang, d. h. nur der aktuelle Vertragsstand ist betroffen. In diesem Fall wird kein Nachlauf benötigt – die Verarbeitung entspricht einer normalen rückwirkenden Änderung über UC-05 mit `NACHTRAG_RUECKWIRKEND`. System weist den Benutzer darauf hin.

- **A4: Rückwirkende Stornierung eines bereits stornierten Vertrags**
  Der Vertrag wurde bereits per UC-06 storniert, aber das Stornierungsdatum muss korrigiert werden (z. B. Kündigung war schon früher wirksam). System invalidiert den bestehenden Stornierungsvertragsstand und erzeugt einen neuen mit korrigiertem Datum.

- **A5: Rückwirkende Änderung mit Schadenfall im betroffenen Zeitraum**
  Im Zeitraum zwischen Wirksamkeitsdatum und heute liegt ein Schadenfall (S6). System zeigt eine Warnung und informiert die Schadenverwaltung über die rückwirkende Änderung (ggf. Auswirkung auf Deckung und Regulierung).

- **A6: Teilweise Schnittstellenrückbuchung fehlgeschlagen**
  Eine oder mehrere Stornobuchungen an S1/S2/S4/S5 schlagen fehl. System policiert die neuen Vertragsstände trotzdem (→ GR-A11) und markiert die fehlgeschlagenen Stornobuchungen zur Wiedervorlage. Dead-Letter-Topics und Retry-Mechanismen greifen.

- **A7: Rückwirkende Bewegung über Versicherungsjahr-Grenze**
  Das Wirksamkeitsdatum liegt in einem vorherigen Versicherungsjahr. System muss ggf. Hauptinkasso-Vertragsstände invalidieren und neu berechnen. Besondere Komplexität bei Zahlungsplan-Korrektur.

- **A8: Kettenrückwirkung (rückwirkende Bewegung auf eine bereits rückwirkende Bewegung)**
  Auf einen Vertrag wird eine zweite rückwirkende Bewegung angewendet, die einen Zeitraum betrifft, der bereits durch eine vorherige Rückwirkung korrigiert wurde. System behandelt die Nachlauf-Stände der vorherigen Rückwirkung wie normale Vertragsstände und invalidiert sie erneut.

## Fehlerfälle

- **F1: Wirksamkeitsdatum vor Vertragsbeginn**
  Das gewählte Wirksamkeitsdatum liegt vor dem Vertragsbeginn. → System zeigt Fehlermeldung: „Das Wirksamkeitsdatum darf nicht vor dem Vertragsbeginn ([Datum]) liegen."

- **F2: Maximale Rückwirkungsfrist überschritten**
  Das Wirksamkeitsdatum liegt weiter als die konfigurierbare Frist in der Vergangenheit (Standard: 12 Monate). → System zeigt Fehlermeldung. Übersteuerung nur mit Kompetenz `RUECKWIRKUNG_UNBEGRENZT`.

- **F3: Kompetenz nicht ausreichend**
  Benutzer besitzt nicht die erforderliche Kompetenz (`RUECKWIRKEND_AENDERN`, `RUECKWIRKEND_STORNIEREN` oder `RUECKWIRKEND_FREIGEBEN`). → Zugriff verweigert.

- **F4: Offene rückwirkende Bewegung existiert**
  Für den Vertrag existiert bereits eine offene rückwirkende Bewegung. → System zeigt Hinweis und Link.

- **F5: Offene vorwärtsgerichtete Änderung/Stornierung existiert**
  Für den Vertrag existiert eine offene Änderung (UC-05) oder Stornierung (UC-06). → System warnt: „Es existiert eine offene Bewegung. Diese muss zuerst abgeschlossen oder verworfen werden."

- **F6: Nachlauf-Neuberechnung fehlgeschlagen**
  Ein Nachlauf-Vertragsstand kann nicht neu berechnet werden (z. B. Produkt zwischenzeitlich entfallen). → System markiert den betroffenen Nachlauf-Stand als `MANUELL_PRUEFEN`. Innendienst muss manuell eingreifen.

- **F7: Stornobuchung fehlgeschlagen**
  Eine Schnittstellenstornierung (S1/S2/S4/S5) schlägt fehl. → System policiert trotzdem (→ GR-A11). Fehlgeschlagene Stornos werden über Transactional Outbox + DLT nachgeliefert. Wiedervorlage für den Innendienst.

- **F8: Inkonsistente Vertragsstand-Kette**
  Die Impact-Analyse ergibt eine inkonsistente Kette (z. B. Lücken in den Gültigkeitszeiträumen). → System bricht ab und meldet: „Die Vertragshistorie ist inkonsistent. Bitte wenden Sie sich an den technischen Support."

- **F9: Schadenfall im betroffenen Zeitraum**
  Im betroffenen Zeitraum liegt ein gemeldeter Schadenfall. → System zeigt Warnung (kein Abbruch). Der Benutzer muss bestätigen, dass die rückwirkende Bewegung trotz des Schadenfalls durchgeführt werden soll.

- **F10: 4-Augen-Prinzip verletzt**
  Ersteller und Freigebender sind dieselbe Person. → System lehnt die Freigabe ab: „Eine rückwirkende Bewegung muss von einer zweiten Person freigegeben werden."

## Geschäftsregeln

| Regel-ID | Beschreibung | Quelle |
|----------|-------------|--------|
| GR-V02 | Ein Vertragsstand gehört immer zu genau einem Vorgang – auch invalidierte Stände behalten ihre Vorgangszuordnung | UC-02, UC-09 |
| GR-V05 | Bei jeder Policierung wird die Vertragsbewegung an das DataWarehouse (S4) gemeldet – bei Rückwirkung zusätzlich Stornobuchungen | UC-05, UC-09 |
| GR-A08 | Alle vier Schnittstellen werden bestückt – bei Rückwirkung sowohl mit Storno- als auch mit Neubuchungen | UC-05, UC-09 |
| GR-A09 | Jeder neue Vertragsstand (rückwirkend + Nachlauf) wird genau einem Vorgang zugeordnet | UC-02, UC-09 |
| GR-A11 | Schnittstellenfehler bei Storno- oder Neubuchungen führen nicht zum Abbruch – Retry-Mechanismen greifen | UC-05, UC-09 |
| GR-RW-01 | Rückwirkende Bewegungen erfordern das **4-Augen-Prinzip**: Ersteller und Freigebender müssen verschiedene Personen sein | UC-09 |
| GR-RW-02 | Invalidierte Vertragsstände werden **nicht physisch gelöscht**, sondern mit Status `INVALIDIERT` + Referenz auf die Rückwirkungs-ID markiert (Audit-Trail) | UC-09 |
| GR-RW-03 | Das Wirksamkeitsdatum muss nach dem Vertragsbeginn und innerhalb der **konfigurierbaren maximalen Rückwirkungsfrist** liegen (Standard: 12 Monate, konfigurierbar pro Sparte) | UC-09 |
| GR-RW-04 | Für jeden invalidierten Vertragsstand werden **Stornobuchungen** an alle betroffenen Schnittstellen gesendet (S4, S1, S2, S5), die die Originalmeldung referenzieren | UC-09 |
| GR-RW-05 | Für jeden neuen Vertragsstand (rückwirkend + Nachlauf) werden **Neubuchungen** an alle vier Schnittstellen gesendet | UC-09 |
| GR-RW-06 | Die Stornobuchungen werden vor den Neubuchungen verarbeitet – die Reihenfolge ist durch **geordnete Kafka-Messages** (Vertragsnummer als Key) sichergestellt | UC-09 |
| GR-RW-07 | Nachlauf-Vertragsstände werden auf Basis des jeweils neuen Vorgänger-Standes **chronologisch** neu berechnet (Kettenberechnung) | UC-09 |
| GR-RW-08 | Pro Vertrag darf maximal eine offene rückwirkende Bewegung existieren | UC-09 |
| GR-RW-09 | Die finanzielle Differenz (Beitrag alt vs. Beitrag neu über den gesamten betroffenen Zeitraum) wird als **Sammelforderung/-erstattung** an das Konzerninkasso (S2) gemeldet | UC-09 |
| GR-RW-10 | Im betroffenen Zeitraum liegende Schadenfälle führen zu einer **Warnung** (kein Abbruch), die der Benutzer bestätigen muss. Die Schadenverwaltung (S6) wird über die rückwirkende Änderung informiert | UC-09 |
| GR-RW-11 | Die Kompetenz `RUECKWIRKUNG_UNBEGRENZT` erlaubt eine Rückwirkung über die Standard-Frist hinaus (z. B. bei gerichtlicher Anordnung) | UC-09 |
| GR-RW-12 | Alle Storno- und Neubuchungen werden im **Stornobuchungs-Protokoll** mit Referenz auf Original-Meldung, Storno-Meldung und Neubuchungs-Meldung dokumentiert | UC-09 |

## Daten (Ein-/Ausgabe)

### Eingabedaten

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| vertrag_id | UUID | ✅ | Existierender Vertrag | Betroffener Vertrag |
| bewegungstyp | Enum | ✅ | RUECKWIRKENDE_AENDERUNG, RUECKWIRKENDE_STORNIERUNG | Art der rückwirkenden Bewegung |
| wirksamkeitsdatum | Date | ✅ | > Vertragsbeginn; < heute; innerhalb Rückwirkungsfrist | Datum, ab dem die Änderung/Stornierung wirksam wird |
| grund | String(500) | ✅ | Min. 10 Zeichen | Begründung der Rückwirkung |
| grund_code | Enum | ✅ | FEHLERKORREKTUR, GERICHTLICHE_ANORDNUNG, ANFECHTUNG, ANZEIGEPFLICHTVERLETZUNG, SONSTIGE | Kategorisierter Grund |
| stornierungsgrund | Enum | Bedingt | Pflicht bei RUECKWIRKENDE_STORNIERUNG | Stornierungsgrund (analog UC-06) |
| veranlasser | Enum | Bedingt | Pflicht bei RUECKWIRKENDE_STORNIERUNG | Veranlasser der Stornierung |
| geaenderte_felder | JSONB | Bedingt | Pflicht bei RUECKWIRKENDE_AENDERUNG | Geänderte Vertragsdaten |

### Ausgabedaten

| Feld | Typ | Beschreibung |
|------|-----|-------------|
| rueckwirkungs_id | UUID | ID der rückwirkenden Bewegung |
| bewegungstyp | Enum | RUECKWIRKENDE_AENDERUNG oder RUECKWIRKENDE_STORNIERUNG |
| status | Enum | Aktueller Status der Bewegung |
| wirksamkeitsdatum | Date | Wirksames Datum der Rückwirkung |
| anzahl_invalidierte_stande | Integer | Anzahl invalidierter Vertragsstände |
| anzahl_neue_stande | Integer | Anzahl neu erzeugter Vertragsstände (rückwirkend + Nachlauf) |
| beitrag_differenz_gesamt | BigDecimal(12,2) | Beitragsdifferenz über den gesamten betroffenen Zeitraum |
| erstattung_nachforderung | BigDecimal(12,2) | Erstattung (positiv) oder Nachforderung (negativ) an VN |
| provision_differenz | BigDecimal(12,2) | Provisionsdifferenz (Rückforderung oder Nachzahlung) |
| stornobuchungen | List | Liste aller Stornobuchungen (pro Schnittstelle) |
| neubuchungen | List | Liste aller Neubuchungen (pro Schnittstelle) |
| neue_vertragsstand_ids | List<UUID> | IDs aller neu erzeugten Vertragsstände |
| freigegeben_von | String | Benutzer-ID des Freigebenden (4-Augen) |

## API-Endpunkte

### Rückwirkende Bewegung initiieren und bearbeiten

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| POST | `/api/v1/vertraege/{vertrag_id}/rueckwirkungen` | `RUECKWIRKEND_AENDERN` / `RUECKWIRKEND_STORNIEREN` | Rückwirkende Bewegung starten |
| GET | `/api/v1/vertraege/{vertrag_id}/rueckwirkungen` | `VERTRAG_LESEN` | Rückwirkende Bewegungen des Vertrags auflisten |
| GET | `/api/v1/vertraege/{vertrag_id}/rueckwirkungen/{id}` | `VERTRAG_LESEN` | Rückwirkungs-Details inkl. Impact-Analyse |
| PUT | `/api/v1/vertraege/{vertrag_id}/rueckwirkungen/{id}` | `RUECKWIRKEND_AENDERN` / `RUECKWIRKEND_STORNIEREN` | Rückwirkungs-Daten aktualisieren |
| DELETE | `/api/v1/vertraege/{vertrag_id}/rueckwirkungen/{id}` | `RUECKWIRKEND_AENDERN` / `RUECKWIRKEND_STORNIEREN` | Rückwirkung verwerfen (logisch) |

### Impact-Analyse und Vorschau

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| GET | `/api/v1/vertraege/{vertrag_id}/rueckwirkungen/{id}/impact` | `RUECKWIRKEND_AENDERN` / `RUECKWIRKEND_STORNIEREN` | Impact-Analyse: betroffene Stände, Schnittstellenmeldungen, Beitragsdifferenz |
| GET | `/api/v1/vertraege/{vertrag_id}/rueckwirkungen/{id}/vorschau` | `RUECKWIRKEND_AENDERN` / `RUECKWIRKEND_STORNIEREN` | Vollständige Vorschau (Stornobuchungen + Neubuchungen + finanzielle Auswirkung) |

### Prüfung und Freigabe (4-Augen)

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| POST | `/api/v1/vertraege/{vertrag_id}/rueckwirkungen/{id}/aktionen/pruefen` | `RUECKWIRKEND_AENDERN` / `RUECKWIRKEND_STORNIEREN` | Gesamtprüfung (Plausibilitäten, Impact, Spartenhooks) |
| POST | `/api/v1/vertraege/{vertrag_id}/rueckwirkungen/{id}/aktionen/freigeben` | `RUECKWIRKEND_FREIGEBEN` | 4-Augen-Freigabe (muss andere Person als Ersteller sein) |
| POST | `/api/v1/vertraege/{vertrag_id}/rueckwirkungen/{id}/aktionen/ablehnen` | `RUECKWIRKEND_FREIGEBEN` | 4-Augen-Ablehnung mit Begründung |

### Stornobuchungs-Protokoll

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| GET | `/api/v1/vertraege/{vertrag_id}/rueckwirkungen/{id}/stornobuchungen` | `VERTRAG_LESEN` | Stornobuchungs-Protokoll (Original → Storno → Neubuchung) |

## Events

| Event | Auslöser | Payload | Konsument |
|-------|----------|---------|-----------|
| `RueckwirkungGestartetEvent` | Rückwirkende Bewegung initiiert | vertrag_id, bewegungstyp, wirksamkeitsdatum, benutzer_id | Durchlaufzeiten (UC-04), Audit-Log |
| `RueckwirkungFreigegebenEvent` | 4-Augen-Freigabe erteilt | vertrag_id, rueckwirkungs_id, freigegeben_von | Audit-Log |
| `VertragsstandInvalidiertEvent` | Vertragsstand invalidiert | vertrag_id, vertragsstand_id, invalidiert_durch, rueckwirkungs_id | S1 (Storno), S2 (Storno), S4 (Storno), S5 (Storno) |
| `RueckwirkenderVertragsstandErzeugtEvent` | Neuer Vertragsstand (rückwirkend oder Nachlauf) erzeugt | vertrag_id, vertragsstand_id, vorgangstyp, gueltig_ab, jahresbeitrag, korrektur_referenz | S1 (Neubuchung), S2 (Neubuchung), S4 (Neubuchung), S5 (Neubuchung) |
| `RueckwirkendeBewegungAbgeschlossenEvent` | Gesamte rückwirkende Bewegung abgeschlossen | vertrag_id, rueckwirkungs_id, anzahl_storniert, anzahl_neu, beitragsdifferenz | UC-04 (Meilenstein), Monitoring, Berichtswesen |

## Statusmodell – Rückwirkende Bewegung

```
  ┌──────────┐
  │ Entwurf  │
  └────┬─────┘
       │
    prüfen
       │
  ┌────▼─────┐
  │ Geprüft  │
  └────┬─────┘
       │
  zur Freigabe   ──────────────────────┐
  einreichen                            │
       │                                │
  ┌────▼────────────┐                   │
  │ Warte auf       │                   │
  │ 4-Augen-Freig.  │                   │
  └───┬─────────┬───┘                   │
      │         │                       │
  freigeben  ablehnen                   │
      │         │                       │
      │    ┌────▼──────┐                │
      │    │ Abgelehnt │                │
      │    │ (Nachbes.) │               │
      │    └────┬──────┘                │
      │         │ erneut einreichen     │
      │         └───────────────────────┘
      │
  ┌───▼────────────┐
  │ Freigegeben    │
  │ (Verarbeitung) │
  └───┬────────┬───┘
      │        │
  erfolg-   teilweise
  reich     fehlerhaft
      │        │
  ┌───▼─────┐ ┌▼───────────────┐
  │Abgeschl.│ │Abgeschl. mit   │
  │         │ │Wiedervorlage   │
  └─────────┘ └────────────────┘

  Sonderstatus: Verworfen (aus Entwurf, Geprüft, Abgelehnt)
```

## Statusmodell – Vertragsstand (Erweiterung)

```
  Bisherige Status:
    GUELTIG (gueltig_bis = NULL → aktueller Stand)
    ABGESCHLOSSEN (gueltig_bis ≠ NULL → historischer Stand)

  Neuer Status durch UC-09:
    INVALIDIERT (durch rückwirkende Bewegung überschrieben)
      → invalidiert_durch: UUID (Referenz auf Rückwirkungs-ID)
      → invalidiert_am: Timestamp
      → Bleibt als Audit-Trail erhalten, wird aber in der
        aktiven Vertragsstandkette nicht mehr berücksichtigt
```

## Nachbedingungen

### Bei rückwirkender Änderung
- Alle betroffenen Vertragsstände ab Wirksamkeitsdatum sind **invalidiert** (Audit-Trail erhalten)
- Ein **neuer rückwirkender Vertragsstand** mit geänderten Daten wurde erzeugt
- Alle **Nachlauf-Vertragsstände** wurden chronologisch neu berechnet und erzeugt
- **Stornobuchungen** an S4/S1/S2/S5 für jeden invalidierten Stand wurden gesendet
- **Neubuchungen** an S4/S1/S2/S5 für jeden neuen Stand wurden gesendet
- Die **Beitragsdifferenz** für den gesamten betroffenen Zeitraum wurde an S2 gemeldet
- Der **Vertrag** wurde mit dem aktuellen Jahresbeitrag aktualisiert
- Das **4-Augen-Prinzip** wurde eingehalten
- Alle Storno- und Neubuchungen sind im **Stornobuchungs-Protokoll** dokumentiert
- Durchlaufzeit-Meilensteine (UC-04) wurden erzeugt

### Bei rückwirkender Stornierung
- Alle Vertragsstände ab Wirksamkeitsdatum sind **invalidiert**
- Ein **abschließender Vertragsstand** mit `gueltig_bis` = Wirksamkeitsdatum wurde erzeugt
- Der Vertragsstatus wurde auf `STORNIERT`/`GEKUENDIGT`/`BEENDET` gesetzt
- **Stornobuchungen** an S4/S1/S2/S5 wurden gesendet
- **Abschluss-Neubuchung** (Schlussabrechnung) an S4/S1/S2/S5 wurde gesendet
- Die **Gesamt-Erstattung/-Nachforderung** wurde an S2 gemeldet

## Neue Kompetenzen (→ Kompetenz-System S8)

| Kompetenz | Beschreibung | Typische Rolle |
|-----------|-------------|----------------|
| `RUECKWIRKEND_AENDERN` | Rückwirkende Vertragsänderung initiieren und bearbeiten | Innendienst-Senior |
| `RUECKWIRKEND_STORNIEREN` | Rückwirkende Stornierung initiieren und bearbeiten | Innendienst-Senior, Rechtsabteilung |
| `RUECKWIRKEND_FREIGEBEN` | 4-Augen-Freigabe für rückwirkende Bewegungen erteilen | Teamleitung, Bereichsleitung |
| `RUECKWIRKUNG_UNBEGRENZT` | Rückwirkung über die Standard-Frist hinaus (z. B. gerichtliche Anordnung) | Rechtsabteilung, Bereichsleitung |

## Akzeptanzkriterien
- [ ] Eine rückwirkende Änderung oder Stornierung kann mit einem Wirksamkeitsdatum in der Vergangenheit gestartet werden
- [ ] Die Impact-Analyse ermittelt korrekt alle betroffenen Vertragsstände, Vorgänge und Schnittstellenmeldungen
- [ ] Die Vorschau zeigt Stornobuchungen, Neubuchungen und finanzielle Auswirkung vollständig an
- [ ] Das 4-Augen-Prinzip wird erzwungen (Ersteller ≠ Freigebender)
- [ ] Bestehende Vertragsstände werden als `INVALIDIERT` markiert (nicht gelöscht – Audit-Trail)
- [ ] Für jeden invalidierten Vertragsstand werden Stornobuchungen an S4, S1, S2, S5 gesendet
- [ ] Neue Vertragsstände (rückwirkend + Nachlauf) werden korrekt erzeugt und chronologisch berechnet
- [ ] Für jeden neuen Vertragsstand werden Neubuchungen an S4, S1, S2, S5 gesendet
- [ ] Stornobuchungen werden VOR Neubuchungen verarbeitet (Reihenfolge garantiert)
- [ ] Die Gesamt-Beitragsdifferenz wird als Sammelforderung/-erstattung an S2 gemeldet
- [ ] Schadenfälle im betroffenen Zeitraum lösen eine Warnung aus (kein Abbruch)
- [ ] Die maximale Rückwirkungsfrist wird validiert (Übersteuerung mit `RUECKWIRKUNG_UNBEGRENZT`)
- [ ] Pro Vertrag ist maximal eine offene rückwirkende Bewegung möglich
- [ ] Schnittstellenfehler bei Storno-/Neubuchungen führen nicht zum Abbruch – Retry greift
- [ ] Alle Storno- und Neubuchungen sind im Stornobuchungs-Protokoll dokumentiert
- [ ] Spartenspezifische Hooks (`vor_Rueckwirkung`, `nach_Rueckwirkung`) werden ausgelöst
- [ ] Durchlaufzeit-Meilensteine (UC-04) werden korrekt erzeugt

## Wireframe / Skizze

### Impact-Analyse & Vorschau

```
┌──────────────────────────────────────────────────────────────────────┐
│  Rückwirkende Änderung – VN-2026-100001     Status: ⬤ Geprüft      │
├──────────────────────────────────────────────────────────────────────┤
│  Vertrag: VN-2026-100001 │ Sparte: KFZ │ VN: Max Mustermann        │
│  Wirksamkeitsdatum: 01.03.2027          │ Grund: Fehlerkorrektur    │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌── Impact-Analyse ────────────────────────────────────────────┐   │
│  │                                                               │   │
│  │  ⚠ Betroffene Vertragsstände:  3 von 4                      │   │
│  │  ⚠ Betroffener Zeitraum:       01.03.2027 – heute           │   │
│  │                                                               │   │
│  │  Vertragsstand-Kette:                                         │   │
│  │  ✅ VS-1: 01.01. – 28.02.  Neugeschäft     → bleibt (gekürzt)│   │
│  │  ❌ VS-2: 01.03. – 31.05.  Nachtrag 1      → INVALIDIERT     │   │
│  │  ❌ VS-3: 01.06. – 30.09.  Nachtrag 2      → INVALIDIERT     │   │
│  │  ❌ VS-4: 01.10. – (aktiv)  Hauptinkasso   → INVALIDIERT     │   │
│  │                                                               │   │
│  └───────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌── Stornobuchungen ──────────────────────────────────────────┐    │
│  │  • S4 (DWH): 3× Bewegungs-Storno                            │    │
│  │  • S1 (Provision): 3× Provisionsrückbuchung                  │    │
│  │  • S2 (Inkasso): Forderungsstorno ab 01.03.2027              │    │
│  │  • S5 (Druck): 2× Druckstorno + 1× Berichtigungsdokument    │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌── Neubuchungen ─────────────────────────────────────────────┐    │
│  │  • VS-NEU-1: 01.03. – 31.05.  Rückwirkende Änd.  512,44 €  │    │
│  │  • VS-NEU-2: 01.06. – 30.09.  Nachlauf (Ntr. 2)  512,44 €  │    │
│  │  • VS-NEU-3: 01.10. – (aktiv) Nachlauf (HI)      527,80 €  │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌── Finanzielle Auswirkung ───────────────────────────────────┐    │
│  │  Beitrag alt (01.03.–heute):          1.218,30 €             │    │
│  │  Beitrag neu (01.03.–heute):          1.289,56 €             │    │
│  │  Differenz:                              +71,26 €            │    │
│  │  → Nachforderung an VN:                  71,26 €             │    │
│  │  → Provisionsnachzahlung Vermittler:      3,56 €            │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ⚠ 4-Augen-Prinzip: Freigabe durch andere Person erforderlich      │
│  Erstellt von: id-mueller │ Freigabe ausstehend                     │
│                                                                      │
│  [💾 Speichern]  [✔ Prüfen]  [📤 Zur Freigabe einreichen]         │
└──────────────────────────────────────────────────────────────────────┘
```

### 4-Augen-Freigabe (Sicht Teamleitung)

```
┌──────────────────────────────────────────────────────────────────────┐
│  Rückwirkung freigeben – VN-2026-100001  Status: ⬤ Warte auf 4A    │
├──────────────────────────────────────────────────────────────────────┤
│  Erstellt von: id-mueller │ Am: 15.11.2027 │ Typ: Rückw. Änderung  │
│  Grund: Fehlerkorrektur – Fahrerkreis war seit 01.03. falsch erfasst│
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Zusammenfassung:                                                    │
│  • 3 Vertragsstände werden invalidiert                               │
│  • 3 neue Vertragsstände werden erzeugt                              │
│  • 12 Schnittstellenbuchungen (6 Storno + 6 Neubuchungen)           │
│  • Nachforderung an VN: 71,26 €                                     │
│  • Provisionsnachzahlung: 3,56 €                                    │
│                                                                      │
│  [📋 Vorschau anzeigen]                                             │
│                                                                      │
│  [✅ Rückwirkung freigeben]  [❌ Rückwirkung ablehnen]              │
│  Ablehnungsgrund: [____________________________________]            │
└──────────────────────────────────────────────────────────────────────┘
```

## Datenmodell-Erweiterungen

### Rückwirkende Bewegung

| Attribut | Typ | Pflicht | Beschreibung |
|----------|-----|---------|-------------|
| id | UUID | ✅ | Technischer Primärschlüssel |
| vertrag_id | UUID (FK) | ✅ | Referenz auf den Vertrag |
| bewegungstyp | Enum | ✅ | RUECKWIRKENDE_AENDERUNG, RUECKWIRKENDE_STORNIERUNG |
| status | Enum | ✅ | ENTWURF, GEPRUEFT, WARTE_AUF_FREIGABE, ABGELEHNT, FREIGEGEBEN, ABGESCHLOSSEN, ABGESCHLOSSEN_MIT_WIEDERVORLAGE, VERWORFEN |
| wirksamkeitsdatum | Date | ✅ | Wirksamkeitsdatum der Rückwirkung |
| grund_code | Enum | ✅ | FEHLERKORREKTUR, GERICHTLICHE_ANORDNUNG, ANFECHTUNG, ANZEIGEPFLICHTVERLETZUNG, SONSTIGE |
| grund_text | String(500) | ✅ | Freitext-Begründung |
| geaenderte_felder | JSONB | ❌ | Geänderte Daten (bei Änderung) |
| stornierungsgrund | Enum | ❌ | Stornierungsgrund (bei Stornierung) |
| veranlasser | Enum | ❌ | Veranlasser (bei Stornierung) |
| erstellt_von | String(100) | ✅ | Benutzer-ID des Erstellers |
| freigegeben_von | String(100) | ❌ | Benutzer-ID des Freigebenden (4-Augen) |
| ablehnungsgrund | String(500) | ❌ | Grund der Ablehnung |
| anzahl_invalidierte_stande | Integer | ❌ | Ergebnis: Invalidierte Stände |
| anzahl_neue_stande | Integer | ❌ | Ergebnis: Neu erzeugte Stände |
| beitrag_differenz_gesamt | BigDecimal(12,2) | ❌ | Ergebnis: Beitragsdifferenz |
| erstellt_am | Timestamp | ✅ | Erstellzeitpunkt |
| freigegeben_am | Timestamp | ❌ | Freigabezeitpunkt |
| abgeschlossen_am | Timestamp | ❌ | Abschlusszeitpunkt |

**Constraints:**
- UNIQUE(vertrag_id) WHERE status IN ('ENTWURF', 'GEPRUEFT', 'WARTE_AUF_FREIGABE', 'FREIGEGEBEN') – max. eine offene Bewegung
- `erstellt_von ≠ freigegeben_von` (4-Augen-Prinzip)

### Vertragsstand (Erweiterung)

| Attribut | Typ | Pflicht | Beschreibung |
|----------|-----|---------|-------------|
| invalidiert | Boolean | ✅ | Standard: false. True bei Invalidierung durch Rückwirkung |
| invalidiert_durch | UUID (FK) | ❌ | Referenz auf die Rückwirkungs-ID |
| invalidiert_am | Timestamp | ❌ | Zeitpunkt der Invalidierung |
| nachlauf_von | UUID (FK) | ❌ | Bei Nachlauf-Ständen: Referenz auf die Rückwirkungs-ID |

### Stornobuchungs-Protokoll

| Attribut | Typ | Pflicht | Beschreibung |
|----------|-----|---------|-------------|
| id | UUID | ✅ | Technischer Primärschlüssel |
| rueckwirkungs_id | UUID (FK) | ✅ | Referenz auf die rückwirkende Bewegung |
| schnittstelle | Enum | ✅ | S1, S2, S4, S5 |
| original_vertragsstand_id | UUID (FK) | ✅ | Invalidierter Vertragsstand |
| original_meldung_id | UUID | ✅ | ID der Original-Schnittstellenmeldung |
| storno_meldung_id | UUID | ❌ | ID der Stornobuchung |
| neuer_vertragsstand_id | UUID (FK) | ❌ | Neuer Vertragsstand (Nachlauf/Korrektur) |
| neubuchung_meldung_id | UUID | ❌ | ID der Neubuchung |
| status | Enum | ✅ | STORNO_AUSSTEHEND, STORNO_GESENDET, STORNO_BESTAETIGT, NEUBUCHUNG_AUSSTEHEND, NEUBUCHUNG_GESENDET, NEUBUCHUNG_BESTAETIGT, FEHLERHAFT |
| erstellt_am | Timestamp | ✅ | Erstellzeitpunkt |
| geaendert_am | Timestamp | ✅ | Letzte Statusänderung |

## Offene Fragen
- Soll die Nachlauf-Neuberechnung hintereinander geschaltete Vorgänge automatisch zusammenführen oder jeden Vorgang einzeln neu berechnen?
- Wie wird mit Hauptinkasso-Vertragsständen umgegangen, deren HFT im rückwirkenden Zeitraum liegt – Zahlungsplan komplett neu aufbauen?
- Soll bei einer rückwirkenden Stornierung die Schadenverwaltung (S6) automatisch benachrichtigt werden, oder muss der Innendienst dies manuell anstoßen?
- Gibt es eine Obergrenze für die Anzahl betroffener Vertragsstände, ab der eine automatische Verarbeitung verweigert wird (z. B. > 20 Stände)?
- Soll das Stornobuchungs-Protokoll auch Bestätigungen (ACKs) von den Konsumenten (S1, S2, S4) entgegennehmen oder reicht die Send-and-Forget-Semantik mit DLT?
- Wie wird die Provisionsdifferenz bei rückwirkender Änderung behandelt – als Einmalbuchung oder als Korrekturserie?
- Soll es eine Vorschau-/Simulationsfunktion geben, die die Impact-Analyse zeigt, ohne die Bewegung tatsächlich zu starten?
