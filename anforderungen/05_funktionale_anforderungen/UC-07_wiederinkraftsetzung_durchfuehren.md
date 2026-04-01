# UC-07: Wiederinkraftsetzung durchführen

## Beschreibung
Ein Benutzer setzt einen zuvor **stornierten, gekündigten oder beendeten Vertrag** wieder in Kraft. Die Wiederinkraftsetzung erzeugt einen **Wiederinkraftsetzungsantrag** oder direkt eine **Wiederinkraftsetzungsschwebe**. Jede Wiederinkraftsetzung hat einen **Grund** und kann mit fachlichen **Bedingungen** verknüpft sein (z. B. Nachzahlung ausstehender Beiträge, erneute Risikoprüfung). Bei der Policierung der Wiederinkraftsetzung wird ein neuer Vertragsstand erzeugt, der Vertrag auf Status `AKTIV` gesetzt und die nachgelagerten Systeme (DataWarehouse S4, Provision S1, Konzerninkasso S2, Druck S5) versorgt.

Ein Änderungsangebot ist bei einer Wiederinkraftsetzung **nicht** vorgesehen – Wiederinkraftsetzungen werden direkt als Antrag oder Schwebe erfasst.

Dies ist ein **spartenübergreifender** Kernprozess. Spartenspezifische Besonderheiten (z. B. eVB-Neuerstellung bei KFZ, erneute GDV-Meldung, Neubewertung der SF-Klasse) werden über die Extension Points / Hooks der Spartenkonfiguration eingebunden.

## Akteure
- **Primär:** Innendienst (Sachbearbeitung)
- **Sekundär:** Außendienst (Vertrieb), Kunde (via Kundenportal S7)

## Vorbedingungen
- Ein Vertrag im Status **`GEKUENDIGT`**, **`STORNIERT`** oder **`BEENDET`** existiert
- Der **Wiederinkraftsetzungszeitraum** ist nicht abgelaufen (konfigurierbar pro Sparte/Grund, z. B. max. 6 Monate nach Stornierungsdatum)
- Benutzer ist am System angemeldet und besitzt mindestens die Kompetenz **`VERTRAG_WIEDERINKRAFTSETZEN`** (→ Kompetenz-System S8)
- Es existiert keine offene Wiederinkraftsetzung für diesen Vertrag (kein offener Antrag oder -schwebe)

## Auslöser
- Benutzer öffnet einen stornierten Vertrag und wählt „Wiederinkraftsetzung starten"
- Kunde beantragt eine Wiederinkraftsetzung über das Kundenportal (S7) oder schriftlich
- Versicherungsnehmer zahlt ausstehende Beiträge nach Nichtzahlungs-Kündigung und beantragt erneuten Versicherungsschutz

## Wiederinkraftsetzungsgründe

| Grund-Code | Bezeichnung | Typischer Auslöser | Beitragsregelung | Risikoprüfung |
|-----------|-------------|---------------------|------------------|---------------|
| `KUENDIGUNG_ZURUECKNAHME` | Rücknahme der Kündigung (VN oder VU) | VN/VU zieht Kündigung zurück, bevor Vertragsende erreicht | Lückenloser Beitragsanspruch | Nein |
| `NACHZAHLUNG_NICHTZAHLUNG` | Nachzahlung nach Nichtzahlungs-Kündigung | VN zahlt ausstehende Beiträge + Mahnkosten | Nachzahlung aller Rückstände | Nein |
| `WIEDERHERSTELLUNG_STILLLEGUNG` | Wieder-Zulassung nach Stilllegung (KFZ) | Fahrzeug wird erneut zugelassen | Neuer Beitrag ab Zulassungsdatum | Ja (SF-Klasse prüfen) |
| `WIEDERHERSTELLUNG_RUHEVERSICHERUNG` | Wiederherstellung nach Ruheversicherung | Fahrzeug wird vor Ablauf der Ruhefrist wieder zugelassen | Neuer Beitrag ab Wiederherstellungsdatum | Ggf. SF-Klasse anpassen |
| `WIDERRUF_STORNIERUNG` | Widerruf der Stornierungsentscheidung | VU widerruft eigene Kündigungsentscheidung (z. B. Anfechtung wurde hinfällig) | Lückenloser Beitragsanspruch | Nein |
| `KULANZ` | Kulanz-Wiederinkraftsetzung | VU setzt Vertrag aus Kulanzgründen wieder in Kraft | Einzelfallentscheidung | Einzelfallentscheidung |
| `SONSTIGE` | Sonstige Gründe (mit Begründung) | Freitextliche Begründung erforderlich | Einzelfallentscheidung | Einzelfallentscheidung |

## Ablauf (Hauptszenario)

### Phase 1: Wiederinkraftsetzung initiieren
1. Benutzer öffnet den stornierten Vertrag und wählt die Funktion **„Wiederinkraftsetzung starten"**
2. System prüft die Kompetenz des Benutzers über das Kompetenz-System (S8)
3. System prüft, ob der Vertrag im Status `GEKUENDIGT`, `STORNIERT` oder `BEENDET` ist
4. System prüft den **Wiederinkraftsetzungszeitraum**:
   - Stornierungsdatum + konfigurierte Frist (je Sparte/Grund) ≥ heute?
   - Ja → weiter mit Schritt 5
   - Nein → System zeigt Hinweis: „Die Frist für eine Wiederinkraftsetzung ist abgelaufen. Bitte legen Sie einen neuen Vertrag an (Neugeschäft)."
5. System prüft, ob bereits eine **offene Wiederinkraftsetzung** für diesen Vertrag existiert:
   - Ja → System zeigt Hinweis: „Es existiert bereits eine offene Wiederinkraftsetzung. Bitte diese zuerst abschließen oder verwerfen."
   - Nein → weiter mit Schritt 6
6. System zeigt das **Wiederinkraftsetzungsformular** mit folgenden Pflichtangaben:
   - **Wiederinkraftsetzungsgrund** (Auswahl aus dem Grund-Katalog)
   - **Einstieg:** Wiederinkraftsetzungsantrag oder Wiederinkraftsetzungsschwebe
7. Benutzer wählt den Einstiegspfad:
   - **Wiederinkraftsetzungsantrag**: Standardpfad – durchläuft Prüfung und Freigabe
   - **Wiederinkraftsetzungsschwebe**: Direkter Innendienst-Einstieg – z. B. bei manueller Bearbeitung eines Kundenantrags

### Phase 2: Wiederinkraftsetzungsdaten erfassen
8. System erzeugt das gewählte Wiederinkraftsetzungsobjekt (Antrag oder Schwebe) im Status **„Entwurf"**
9. System zeigt die **letzten Vertragsdetails** (Produkte, Beiträge, letzter Vertragsstand vor Stornierung) an
10. System zeigt die **Stornierungsinformationen** an:
    - Stornierungsgrund (aus UC-06)
    - Stornierungsdatum / Vertragsende
    - Veranlasser der Stornierung
    - Schlussabrechnung (Erstattung/Nachforderung)
11. Benutzer erfasst die Wiederinkraftsetzungsdaten:
    - **Wiederinkraftsetzungsdatum** (= neuer Gültig-ab):
      - Bei Rücknahme: ursprüngliches Stornierungsdatum (lückenlos)
      - Bei Nachzahlung: Datum der vollständigen Zahlung oder vereinbartes Datum
      - Bei Wieder-Zulassung: Zulassungsdatum
      - Frühestens: Stornierungsdatum, spätestens: heute + konfigurierbare Kulanzfrist
    - **Vertragsdaten-Übernahme**: Standardmäßig werden alle Daten des letzten Vertragsstands vor Stornierung übernommen
    - **Änderungen** (optional): Benutzer kann bei der Wiederinkraftsetzung gleichzeitig Vertragsdaten anpassen (bearbeitbare Felder analog UC-05, abhängig von Spartenkonfiguration und Kompetenz)
    - **Begründung** (Freitext, Pflicht bei `KULANZ` und `SONSTIGE`)
    - **Nachweis-Referenz** (optional: z. B. Zahlungsnachweis, Zulassungsbescheinigung)
12. → **Hook `vor_Wiederinkraftsetzung`** wird ausgelöst (spartenspezifische Prüfungen, z. B. SF-Klasse Neubewertung bei KFZ, eVB-Prüfung)

### Phase 3: Beitragsberechnung und Nachberechnung
13. System prüft, ob eine **Nachberechnung** erforderlich ist:
    - **Lückenloser Übergang** (z. B. Rücknahme Kündigung): Beitrag für die Lücke zwischen Stornierungsdatum und Wiederinkraftsetzungsdatum wird nachberechnet
    - **Neuer Versicherungsbeginn** (z. B. Wieder-Zulassung): Beitrag ab Wiederinkraftsetzungsdatum neu berechnen
    - **Geänderter Vertrag**: Falls Vertragsdaten angepasst wurden, Neuberechnung auf Basis der neuen Daten
14. System berechnet den **neuen Beitrag**:
    - Jahresbeitrag auf Basis der übernommenen bzw. angepassten Vertragsdaten
    - Ggf. Nachforderung für den Lückenzeitraum (pro rata temporis)
    - Ggf. Verrechnung mit offenen Erstattungen aus der Stornierung
15. System zeigt die **Beitragsübersicht**:
    - Neuer Jahresbeitrag
    - Nachforderung für Lückenzeitraum (falls zutreffend)
    - Verrechnung mit offener Erstattung (falls vorhanden)
    - Zahlungsbetrag (Summe)
16. System führt **Plausibilitätsprüfungen** durch:
    - Wiederinkraftsetzungsdatum plausibel?
    - Frist nicht abgelaufen?
    - Grund-spezifische Voraussetzungen erfüllt? (z. B. Nachzahlung vollständig bei `NACHZAHLUNG_NICHTZAHLUNG`)
    - Spartenspezifische Plausibilitäten (über Hook)
17. System zeigt Plausibilitätsfehler und -hinweise an

### Phase 4: Wiederinkraftsetzung freigeben

#### Pfad A: Einstieg über Wiederinkraftsetzungsantrag
18a. Benutzer wählt **„Wiederinkraftsetzung prüfen"**
19a. System führt die Gesamtprüfung durch (Plausibilitäten, Fristen, Spartenhooks)
20a. Prüfung bestanden → Status wechselt zu **„Geprüft"**
21a. Benutzer wählt **„Wiederinkraftsetzung freigeben"**
22a. System prüft die Kompetenz `ANTRAG_FREIGEBEN`
23a. System ermittelt den Vorgangstyp → `WIEDERINKRAFTSETZUNG`
24a. System legt eine **Schwebe** an (→ GR-A05)
25a. System prüft **Aussteuerungsregeln**:
    - **Nicht ausgesteuert** → weiter mit Phase 5 (Policierung)
    - **Ausgesteuert** → Schwebe bleibt offen, Innendienst bearbeitet (analog UC-02, Pfad B)

#### Pfad B: Einstieg über Wiederinkraftsetzungsschwebe (direkt)
18b. Wiederinkraftsetzungsschwebe wurde direkt erzeugt → Innendienst-Sachbearbeiter prüft die Wiederinkraftsetzung
19b. Sachbearbeiter wählt **„Wiederinkraftsetzung policieren"** (erfordert `SCHWEBE_POLICIEREN`)
20b. Weiter mit Phase 5

### Phase 5: Policierung (Vertrag wieder aktiv)
26. → **Hook `vor_Policierung`** wird ausgelöst (spartenspezifische Prüfungen, z. B. SF-Klasse validieren bei KFZ)
27. System erzeugt einen **neuen Vertragsstand** (Version n+1) am bestehenden Vertrag:
    - `gueltig_ab` = Wiederinkraftsetzungsdatum
    - `gueltig_bis` = NULL (unbefristet bis nächste Hauptfälligkeit/Änderung)
    - Jahresbeitrag = neu berechneter Beitrag
    - Vertragsdaten = übernommene bzw. angepasste Daten des letzten Vertragsstands vor Stornierung
28. Abschließender Vertragsstand der Stornierung bleibt unverändert (historischer Snapshot)
29. System erzeugt einen **Vorgang** mit Vorgangstyp `WIEDERINKRAFTSETZUNG`
30. System setzt den **Vertragsstatus** auf `AKTIV`
31. System löscht das `vertragsende`-Datum (oder setzt es auf die nächste reguläre Hauptfälligkeit)
32. → **Hook `nach_Wiederinkraftsetzung`** wird ausgelöst (spartenspezifische Aktionen, z. B. GDV-Wiederzulassungsmeldung bei KFZ, eVB-Neuerstellung)

### Phase 6: Nachgelagerte Systeme versorgen
33. System publiziert **`VertragWiederinkraftgesetztEvent`** (Spring Event) → löst folgende Schnittstellenaufrufe aus:

| Schnittstelle | Aktion | Payload-Kern | Mechanismus |
|---------------|--------|-------------|-------------|
| **S4 – DataWarehouse** | Vertragsbewegung `WIEDERINKRAFTSETZUNG` melden | Vertragsnummer, Sparte, Wiederinkraftsetzungsgrund, Wiederinkraftsetzungsdatum, neuer Beitrag | Kafka-Topic `vertrag.datawarehouse.bewegungen` |
| **S1 – Provision** | Wiederinkraftsetzungsereignis melden | Vertragsnummer, Sparte, Wiederinkraftsetzungsdatum, Vermittler-ID, neuer Beitrag | Kafka-Topic `vertrag.provision.ereignisse` (Ereignistyp `WIEDERINKRAFTSETZUNG`) |
| **S2 – Konzerninkasso** | Neue Beitragsforderung einstellen + Nachforderung für Lücke | Vertragsnummer, Wiederinkraftsetzungsdatum, Jahresbeitrag, Zahlungsweise, Nachforderungsbetrag | Kafka-Topic `vertrag.inkasso.beitragsforderungen` |
| **S5 – Druck** | Bestätigung der Wiederinkraftsetzung drucken | Vertragsnummer, Partner, Wiederinkraftsetzungsgrund, Wiederinkraftsetzungsdatum, Dokumenttyp `WIEDERINKRAFTSETZUNGSBESTAETIGUNG` | REST `POST /api/v1/druck/auftraege` |

34. System setzt die Schwebe auf Status **„Erledigt"**
35. System bestätigt die erfolgreiche Wiederinkraftsetzung und zeigt die Zusammenfassung an

## Alternativszenarien

- **A1: Wiederinkraftsetzung verwerfen**
  Der Benutzer kann eine offene Wiederinkraftsetzung verwerfen. System fragt: „Wiederinkraftsetzung wirklich verwerfen?" Bei Bestätigung wird das Objekt auf Status **„Verworfen"** gesetzt. Der Vertrag bleibt unverändert im stornierten Status.

- **A2: Wiederinkraftsetzung über Kundenportal (S7)**
  Der Kunde beantragt eine Wiederinkraftsetzung über das Kundenportal. Die Wiederinkraftsetzung wird als Antrag im Status „Offen" ins System übernommen und **immer ausgesteuert** (Innendienst-Prüfung bei Kundenportal-Anfragen).

- **A3: Wiederinkraftsetzung mit Vertragsänderung**
  Der Benutzer möchte bei der Wiederinkraftsetzung gleichzeitig Vertragsdaten anpassen (z. B. neues Fahrzeug bei KFZ, Deckungserweiterung). System bietet die Möglichkeit, bearbeitbare Felder analog UC-05 zu ändern. Die Beitragsberechnung erfolgt auf Basis der geänderten Daten.

- **A4: Wiederinkraftsetzung ablehnen (Innendienst)**
  Der Innendienst lehnt eine ausgesteuerte Wiederinkraftsetzung ab (z. B. Nachzahlung nicht geleistet, Risikoprüfung negativ). Die Schwebe wird geschlossen, der Vertrag bleibt im stornierten Status. Der Ablehnungsgrund wird dokumentiert.

- **A5: Wiederinkraftsetzung zurückweisen (Nachbearbeitung)**
  Der Innendienst weist die Wiederinkraftsetzung zurück (z. B. fehlende Nachweise, unvollständige Zahlung). Status geht zurück auf „Offen", Ersteller kann nachbessern.

- **A6: Wiederinkraftsetzung mit Bedingungen**
  Der Innendienst genehmigt die Wiederinkraftsetzung unter Bedingungen (z. B. Nachzahlung innerhalb von 14 Tagen, erneute Gesundheitsprüfung bei Lebensversicherung). System dokumentiert die Bedingungen und überwacht deren Erfüllung.

- **A7: Frist abgelaufen – Neugeschäft erforderlich**
  Der Wiederinkraftsetzungszeitraum ist abgelaufen. System verweist auf das Neugeschäft (UC-01). Die historischen Vertragsdaten können als Vorlage für ein neues Angebot übernommen werden.

- **A8: Risikoprüfung erforderlich**
  Bei bestimmten Gründen (z. B. Wieder-Zulassung, Kulanz) ist eine erneute Risikoprüfung notwendig. Das System löst diese über den Hook `vor_Wiederinkraftsetzung` spartenspezifisch aus (z. B. SF-Klasse Neuberechnung, Gesundheitsprüfung).

## Fehlerfälle

- **F1: Vertrag nicht im Status GEKUENDIGT, STORNIERT oder BEENDET**
  Der Vertrag ist aktiv oder ruhend. → System zeigt Fehlermeldung: „Dieser Vertrag ist aktiv und kann nicht wiederinkraftgesetzt werden."

- **F2: Wiederinkraftsetzungsfrist abgelaufen**
  Die konfigurierbare Frist seit Stornierungsdatum ist überschritten. → System zeigt Fehlermeldung mit Stornierungsdatum und abgelaufener Frist. Weist auf Neugeschäft (UC-01) hin.

- **F3: Offene Wiederinkraftsetzung existiert**
  Für den Vertrag existiert bereits ein offener Wiederinkraftsetzungsantrag oder eine offene Schwebe. → System zeigt Hinweis und Link zur bestehenden Wiederinkraftsetzung.

- **F4: Kompetenz nicht ausreichend**
  Benutzer besitzt nicht die Kompetenz `VERTRAG_WIEDERINKRAFTSETZEN`. → System verweigert den Zugriff mit Fehlermeldung.

- **F5: Ausstehende Beiträge bei Nichtzahlungs-Kündigung**
  Bei Grund `NACHZAHLUNG_NICHTZAHLUNG` sind die ausstehenden Beiträge + Mahnkosten noch nicht vollständig bezahlt. → System zeigt den offenen Betrag an und blockiert die Freigabe bis zum Zahlungseingang.

- **F6: Risikoprüfung negativ**
  Die über den Spartenhook ausgelöste Risikoprüfung ergibt eine Ablehnung (z. B. SF-Klasse inakzeptabel). → System zeigt den Ablehnungsgrund an. Benutzer kann manuell entscheiden (mit entsprechender Kompetenz `WIEDERINKRAFTSETZUNG_KULANZ`).

- **F7: Schnittstellenfehler bei Policierung**
  Eine oder mehrere nachgelagerte Schnittstellen sind nicht erreichbar. → Wiederinkraftsetzung wird trotzdem policiert (→ GR-A11). Retry-Mechanismen liefern nach.

- **F8: Vertragsänderung nicht kompatibel**
  Benutzer hat bei der Wiederinkraftsetzung Vertragsdaten geändert, die eine Plausibilitätsprüfung nicht bestehen (z. B. ungültiges Fahrzeug bei KFZ). → System zeigt Plausibilitätsfehler analog UC-05 an.

## Geschäftsregeln

| Regel-ID | Beschreibung | Quelle |
|----------|-------------|--------|
| GR-V02 | Ein Vertragsstand gehört immer zu genau einem Vorgang – hier Vorgangstyp `WIEDERINKRAFTSETZUNG` | UC-02, UC-07 |
| GR-V05 | Bei jeder Policierung wird die Vertragsbewegung an das DataWarehouse (S4) gemeldet | UC-05, UC-07 |
| GR-A05 | Bei jeder Freigabe wird eine Schwebe angelegt | UC-02, UC-07 |
| GR-A06 | Aussteuerungsregeln bestimmen, ob der Wiederinkraftsetzungsantrag dem Innendienst vorgelegt wird | UC-02, UC-07 |
| GR-A08 | Bei der Policierung werden alle vier Schnittstellen bestückt: S4, S1, S2, S5 | UC-05, UC-07 |
| GR-A09 | Die Policierung erzeugt genau einen Vertragsstand, der genau einem Vorgang zugeordnet ist | UC-02, UC-07 |
| GR-A10 | Bei Wiederinkraftsetzung wird der bestehende Vertrag fortgeführt (neuer Vertragsstand am selben Vertrag) | UC-05, UC-07 |
| GR-A11 | Schnittstellenfehler führen nicht zum Abbruch – Retry-Mechanismen liefern nach | UC-05, UC-07 |
| GR-A12 | Manuelle Policierung einer Schwebe erfordert `SCHWEBE_POLICIEREN` | UC-02, UC-07 |
| GR-WI-01 | Jede Wiederinkraftsetzung erfordert einen **Wiederinkraftsetzungsgrund** aus dem Grund-Katalog | UC-07 |
| GR-WI-02 | Bei einer Wiederinkraftsetzung kann **kein** Änderungsangebot erzeugt werden – nur Antrag oder Schwebe | UC-07 |
| GR-WI-03 | Eine Wiederinkraftsetzung ist nur innerhalb der **konfigurierbaren Frist** nach Stornierungsdatum möglich (Standard: 6 Monate, konfigurierbar pro Sparte und Stornierungsgrund) | UC-07 |
| GR-WI-04 | Pro Vertrag darf maximal eine offene Wiederinkraftsetzung (Antrag oder Schwebe) existieren | UC-07 |
| GR-WI-05 | Bei `NACHZAHLUNG_NICHTZAHLUNG` müssen **alle ausstehenden Beiträge und Mahnkosten** beglichen sein, bevor die Wiederinkraftsetzung freigegeben werden kann | UC-07 |
| GR-WI-06 | Wiederinkraftsetzungen über das Kundenportal (S7) werden **immer** an den Innendienst ausgesteuert | UC-07 |
| GR-WI-07 | Bei lückenloser Wiederinkraftsetzung (z. B. Rücknahme Kündigung) wird der Beitrag für den Lückenzeitraum pro rata temporis nachberechnet | UC-07 |
| GR-WI-08 | Die Wiederinkraftsetzung setzt den Vertragsstatus **immer** auf `AKTIV`, unabhängig vom vorherigen Status (`GEKUENDIGT`, `STORNIERT`, `BEENDET`) | UC-07 |
| GR-WI-09 | Bei der Policierung wird das `vertragsende`-Datum entfernt oder auf die nächste reguläre Hauptfälligkeit gesetzt | UC-07 |
| GR-WI-10 | Spartenspezifische Risikoprüfungen bei Wiederinkraftsetzung werden über den Hook `vor_Wiederinkraftsetzung` durchgeführt; das Ergebnis kann die Wiederinkraftsetzung blockieren oder mit Auflagen versehen | UC-07 |

## Daten (Ein-/Ausgabe)

### Eingabedaten

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| vertrag_id | UUID | ✅ | Existierender Vertrag im Status GEKUENDIGT/STORNIERT/BEENDET | Wiederinkraftzusetzender Vertrag |
| einstiegstyp | Enum | ✅ | WIEDERINKRAFTSETZUNGSANTRAG, WIEDERINKRAFTSETZUNGSSCHWEBE | Einstiegspfad |
| wiederinkraftsetzungsgrund | Enum | ✅ | Gültiger Wert aus Grund-Katalog | Grund der Wiederinkraftsetzung |
| wiederinkraftsetzungsdatum | Date | ✅ | ≥ Stornierungsdatum; ≤ heute + Kulanzfrist | Neuer Gültig-ab des Vertragsstands |
| begruendung | String(2000) | Bedingt | Pflicht bei Grund `KULANZ` und `SONSTIGE`; min. 10 Zeichen | Freitextliche Begründung |
| nachweis_referenz | String(100) | ❌ | – | Dokumenten-ID eines Nachweises (z. B. Zahlungsbeleg, Zulassungsbescheinigung) |
| vertragsdaten_aenderungen | Object | ❌ | Analog UC-05 Feldberechtigungen | Optionale Vertragsanpassungen bei Wiederinkraftsetzung |

### Ausgabedaten

| Feld | Typ | Beschreibung |
|------|-----|-------------|
| wiederinkraftsetzung_id | UUID | ID des Wiederinkraftsetzungsantrags / -schwebe |
| wiederinkraftsetzung_typ | Enum | WIEDERINKRAFTSETZUNGSANTRAG oder WIEDERINKRAFTSETZUNGSSCHWEBE |
| status | Enum | Aktueller Status |
| wiederinkraftsetzungsdatum | Date | Neuer Gültig-ab des Vertragsstands |
| neuer_jahresbeitrag | BigDecimal(12,2) | Neu berechneter Jahresbeitrag |
| nachforderung_luecke | BigDecimal(12,2) | Nachforderung für Lückenzeitraum (0,00 wenn kein Lückenzeitraum) |
| verrechnung_erstattung | BigDecimal(12,2) | Verrechnung mit offener Erstattung aus Stornierung |
| zahlungsbetrag | BigDecimal(12,2) | Gesamtbetrag (Nachforderung − Verrechnung) |
| neuer_vertragsstatus | Enum | Immer `AKTIV` |
| vertragsstand_id | UUID | ID des neuen Vertragsstands (nach Policierung) |
| vorgang_id | UUID | ID des erzeugten Vorgangs (Typ WIEDERINKRAFTSETZUNG) |

## API-Endpunkte

### Wiederinkraftsetzung initiieren und bearbeiten

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| POST | `/api/v1/vertraege/{vertrag_id}/wiederinkraftsetzung` | `VERTRAG_WIEDERINKRAFTSETZEN` | Wiederinkraftsetzung starten (erzeugt Antrag oder Schwebe) |
| GET | `/api/v1/vertraege/{vertrag_id}/wiederinkraftsetzung` | `VERTRAG_LESEN` | Aktuelle / historische Wiederinkraftsetzungen des Vertrags abrufen |
| GET | `/api/v1/vertraege/{vertrag_id}/wiederinkraftsetzung/{id}` | `VERTRAG_LESEN` | Wiederinkraftsetzungsdetails abrufen |
| PUT | `/api/v1/vertraege/{vertrag_id}/wiederinkraftsetzung/{id}` | `VERTRAG_WIEDERINKRAFTSETZEN` | Wiederinkraftsetzungsdaten aktualisieren |
| DELETE | `/api/v1/vertraege/{vertrag_id}/wiederinkraftsetzung/{id}` | `VERTRAG_WIEDERINKRAFTSETZEN` | Wiederinkraftsetzung verwerfen (logisch → Status „Verworfen") |

### Beitragsberechnung und Prüfung

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| GET | `/api/v1/vertraege/{vertrag_id}/wiederinkraftsetzung/{id}/beitragsberechnung` | `VERTRAG_LESEN` | Beitragsübersicht (Jahresbeitrag, Nachforderung, Verrechnung) |
| POST | `/api/v1/vertraege/{vertrag_id}/wiederinkraftsetzung/{id}/aktionen/pruefen` | `VERTRAG_WIEDERINKRAFTSETZEN` | Gesamtprüfung (Plausibilitäten, Fristen, Spartenhooks) |

### Freigabe und Policierung

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| POST | `/api/v1/vertraege/{vertrag_id}/wiederinkraftsetzung/{id}/aktionen/freigeben` | `ANTRAG_FREIGEBEN` | Wiederinkraftsetzungsantrag freigeben → Schwebe + ggf. Policierung |
| POST | `/api/v1/vertraege/{vertrag_id}/wiederinkraftsetzung/{id}/aktionen/policieren` | `SCHWEBE_POLICIEREN` | Wiederinkraftsetzungsschwebe manuell policieren |

### Metadaten

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| GET | `/api/v1/wiederinkraftsetzungsgruende` | `VERTRAG_LESEN` | Wiederinkraftsetzungsgrund-Katalog abrufen |
| GET | `/api/v1/vertraege/{vertrag_id}/wiederinkraftsetzung/fristpruefung` | `VERTRAG_LESEN` | Fristprüfung: ist Wiederinkraftsetzung noch fristgerecht möglich? |

## Events

| Event | Auslöser | Payload | Konsument |
|-------|----------|---------|-----------|
| `WiederinkraftsetzungGestartetEvent` | Wiederinkraftsetzung initiiert | vertrag_id, wiederinkraftsetzungsgrund, benutzer_id | Durchlaufzeiten (UC-04), Audit-Log |
| `WiederinkraftsetzungFreigegebenEvent` | Wiederinkraftsetzungsantrag freigegeben | vertrag_id, antrag_id, ausgesteuert | Schweben-Dashboard, Innendienst |
| `VertragWiederinkraftgesetztEvent` | Wiederinkraftsetzung policiert | vertrag_id, vertragsstand_id, wiederinkraftsetzungsgrund, wiederinkraftsetzungsdatum, neuer_jahresbeitrag, nachforderung_luecke | S1 (Provision), S2 (Inkasso), S4 (DWH), S5 (Druck) |

## Statusmodell – Wiederinkraftsetzungsobjekt

```
                    ┌──────────┐
                    │ Entwurf  │
                    └────┬─────┘
                         │
              ┌──────────┴──────────┐
              │                     │
        ┌─────▼──────┐       ┌─────▼──────────┐
        │  WIK-      │       │  WIK-          │
        │  Antrag    │       │  Schwebe        │
        │  (Offen)   │       │  (Offen)        │
        └─────┬──────┘       └───────┬─────────┘
              │                      │
         prüfen                      │
              │                      │
        ┌─────▼──────┐               │
        │  Geprüft   │               │
        └─────┬──────┘               │
              │                      │
         freigeben              policieren
              │                      │
        ┌─────▼──────┐               │
        │ Aussteuer- │               │
        │ ungsprüfung│               │
        └──┬──────┬──┘               │
           │      │                  │
     nicht │      │ ausge-           │
     ausgest.     │ steuert          │
           │      │                  │
           │  ┌───▼────────┐         │
           │  │Ausgesteuert│         │
           │  └───┬────┬───┘         │
           │      │    │             │
           │  freig. ableh.          │
           │      │    │             │
           └──┬───┘  ┌─▼──────┐     │
              │      │Abgelehn.│     │
              │      └─────────┘     │
              │                      │
              └────────┬─────────────┘
                       │
                  policieren
                       │
                 ┌─────▼──────┐
                 │  Policiert  │
                 │ (Vertrag    │
                 │  AKTIV)     │
                 └─────────────┘

    Sonderstatus: Verworfen (aus jedem Status, außer Policiert)
```

## Nachbedingungen
- Ein **neuer Vertragsstand** (Version n+1) wurde am bestehenden Vertrag erzeugt mit `gueltig_ab` = Wiederinkraftsetzungsdatum
- Der Vertragsstatus wurde auf **`AKTIV`** gesetzt
- Das Feld `vertragsende` wurde entfernt oder auf die nächste reguläre Hauptfälligkeit gesetzt
- Ein **Vorgang** mit Typ `WIEDERINKRAFTSETZUNG` wurde erzeugt und dem Vertragsstand zugeordnet
- Die **Schwebe** wurde auf „Erledigt" gesetzt
- **DataWarehouse (S4)** hat die Wiederinkraftsetzungsbewegung erhalten (Kafka)
- **Provision (S1)** hat das Wiederinkraftsetzungsereignis erhalten (Kafka)
- **Konzerninkasso (S2)** hat die neue Beitragsforderung und ggf. Nachforderung erhalten (Kafka)
- **Druckschnittstelle (S5)** hat den Druckauftrag für die Wiederinkraftsetzungsbestätigung erhalten (REST)
- Spartenspezifische Hooks (`vor_Wiederinkraftsetzung`, `nach_Wiederinkraftsetzung`) wurden ausgeführt
- Alle Statuswechsel sind revisionssicher protokolliert
- Durchlaufzeit-Meilensteine (UC-04) wurden erzeugt

## Neue Kompetenzen (→ Kompetenz-System S8)

| Kompetenz | Beschreibung | Typische Rolle |
|-----------|-------------|----------------|
| `VERTRAG_WIEDERINKRAFTSETZEN` | Wiederinkraftsetzung initiieren und bearbeiten | Innendienst, Außendienst (eingeschränkt) |
| `WIEDERINKRAFTSETZUNG_KULANZ` | Kulanz-Wiederinkraftsetzung durchführen (erweiterte Gründe, Risikoprüfung übersteuern) | Innendienst-Senior, Teamleitung |
| `WIEDERINKRAFTSETZUNG_RUECKWIRKEND` | Rückwirkende Wiederinkraftsetzung vor dem Stornierungsdatum genehmigen | Innendienst-Senior |

## Akzeptanzkriterien
- [ ] Aus einem Vertrag im Status GEKUENDIGT, STORNIERT oder BEENDET kann eine Wiederinkraftsetzung gestartet werden
- [ ] Nur Wiederinkraftsetzungsantrag oder -schwebe als Einstieg möglich – kein Änderungsangebot
- [ ] Wiederinkraftsetzungsgrund (Pflicht) wird aus dem Grund-Katalog ausgewählt
- [ ] Die konfigurierbare Wiederinkraftsetzungsfrist wird pro Sparte/Grund validiert
- [ ] Bei Nachzahlung-Nichtzahlung müssen alle ausstehenden Beiträge beglichen sein
- [ ] Beitragsberechnung berücksichtigt Lückenzeitraum und offene Erstattungen
- [ ] Wiederinkraftsetzungsantrag durchläuft Prüfung, Freigabe und Aussteuerung analog UC-02
- [ ] Wiederinkraftsetzungsschwebe kann vom Innendienst direkt policiert werden
- [ ] Bei Policierung wird ein neuer Vertragsstand erzeugt und der Vertrag auf AKTIV gesetzt
- [ ] Alle vier Schnittstellen werden bei Policierung bedient: S4 (DWH), S1 (Provision), S2 (Inkasso), S5 (Druck)
- [ ] Pro Vertrag ist maximal eine offene Wiederinkraftsetzung gleichzeitig möglich
- [ ] Wiederinkraftsetzungen über das Kundenportal werden immer ausgesteuert
- [ ] Spartenspezifische Hooks (`vor_Wiederinkraftsetzung`, `nach_Wiederinkraftsetzung`) werden ausgelöst
- [ ] Vertragsdaten können optional bei der Wiederinkraftsetzung angepasst werden (analog UC-05 Feldberechtigungen)
- [ ] Durchlaufzeit-Meilensteine (UC-04) werden korrekt erzeugt

## Wireframe / Skizze

```
┌──────────────────────────────────────────────────────────────────────┐
│  Wiederinkraftsetzung – VN-2026-100001       Status: ⬤ Entwurf     │
├──────────────────────────────────────────────────────────────────────┤
│  Vertrag: VN-2026-100001 │ Sparte: KFZ │ VN: Max Mustermann        │
│  Vertragsbeginn: 01.01.2027 │ Vertragsende: 01.07.2027 (GEKÜNDIGT) │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌── Stornierungsinformationen (schreibgeschützt) ─────────────┐    │
│  │ Stornierungsgrund: Ordentliche Kündigung zum Ablauf          │    │
│  │ Veranlasser: Versicherungsnehmer (VN)                        │    │
│  │ Stornierungsdatum: 01.07.2027                                │    │
│  │ Schlussabrechnung: Erstattung 243,66 €                       │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Wiederinkraftsetzungsdaten                                          │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │ WIK-Grund:         [Rücknahme der Kündigung (VN/VU)      ▼]  │    │
│  │ WIK-Datum:         [01.07.2027 📅]  (= lückenlos)            │    │
│  │ Begründung:        [____________________________________]    │    │
│  │ Nachweis:          [📎 Dokument hochladen]                   │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌── Beitragsberechnung ────────────────────────────────────────┐   │
│  │                                                               │   │
│  │  Neuer Jahresbeitrag:        487,32 €                        │   │
│  │  Nachforderung Lücke:          0,00 €  (lückenlos)           │   │
│  │  Verrechnung Erstattung:    −243,66 €                        │   │
│  │  ─────────────────────────────────────────                   │   │
│  │  Zahlungsbetrag:            −243,66 €  (Erstattung verfällt) │   │
│  │                                                               │   │
│  └───────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌── Vertragsüberblick ◻ Daten anpassen ──────────────────────┐    │
│  │  Produkte: KFZ-HP (312,00 €) │ KFZ-TK (167,11 €)          │    │
│  │  Fahrzeug: VW Golf VIII │ Kennz.: MS-LV 1234               │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Einstieg: ○ WIK-Antrag  ○ WIK-Schwebe                             │
│                                                                      │
│  [💾 Speichern]  [✔ Prüfen]  [📤 Freigeben]  [🗑 Verwerfen]      │
└──────────────────────────────────────────────────────────────────────┘
```

## Offene Fragen
- Soll bei einer Wiederinkraftsetzung nach Nichtzahlung eine automatische Prüfung im Inkasso-System (S2) stattfinden, ob die ausstehenden Beiträge tatsächlich eingegangen sind?
- Gibt es eine maximale Anzahl von Wiederinkraftsetzungen pro Vertrag oder eine Sperrfrist nach wiederholter Stornierung/Wiederinkraftsetzung?
- Soll bei KFZ-Wiederinkraftsetzung automatisch eine eVB über das Partner-System (S7) generiert werden?
- Wie wird bei einer lückenlosen Wiederinkraftsetzung der Versicherungsschutz für den Lückenzeitraum rückwirkend sichergestellt (z. B. Schaden im Lückenzeitraum)?
- Können die Wiederinkraftsetzungsgründe per Admin-UI erweiterbar sein oder ist der Katalog fest?
- Soll der Außendienst Wiederinkraftsetzungen initiieren dürfen oder nur der Innendienst?
