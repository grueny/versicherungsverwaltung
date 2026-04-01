# UC-06: Vertragsstornierung durchführen

## Beschreibung
Ein Benutzer leitet die **Stornierung eines bestehenden Vertrags** ein. Aus dem Vertrag heraus wird ein **Stornierungsantrag** oder direkt eine **Stornierungsschwebe** erzeugt. Jede Stornierung hat einen **Stornierungsgrund** und einen **Veranlasser** (wer die Stornierung initiiert hat). Bei der Policierung der Stornierung wird ein abschließender Vertragsstand erzeugt, der Vertrag auf Status `STORNIERT` bzw. `GEKUENDIGT` gesetzt und die nachgelagerten Systeme (DataWarehouse S4, Provision S1, Konzerninkasso S2, Druck S5) versorgt.

Ein Änderungsangebot ist bei einer Stornierung **nicht** vorgesehen – Stornierungen werden direkt als Antrag oder Schwebe erfasst.

Dies ist ein **spartenübergreifender** Kernprozess. Spartenspezifische Besonderheiten (z. B. Ruheversicherung/eVB-Stornierung bei KFZ, Nachhaftung in der Haftpflicht) werden über die Extension Points / Hooks der Spartenkonfiguration eingebunden.

## Akteure
- **Primär:** Innendienst (Sachbearbeitung), Außendienst (Vertrieb)
- **Sekundär:** Kunde (via Kundenportal S7, schriftliche Kündigung), System (automatische Stornierung)

## Vorbedingungen
- Ein Vertrag im Status **`AKTIV`** oder **`RUHEND`** existiert
- Benutzer ist am System angemeldet und besitzt mindestens die Kompetenz **`VERTRAG_STORNIEREN`** (→ Kompetenz-System S8)
- Es existiert keine offene Stornierung für diesen Vertrag (kein offener Stornierungsantrag oder -schwebe)

## Auslöser
- Benutzer öffnet einen Vertrag und wählt „Vertrag stornieren"
- Kunde reicht eine schriftliche Kündigung ein (Erfassung durch Innendienst)
- Kunde beantragt eine Kündigung über das Kundenportal (S7)
- Systemereignis löst automatische Stornierung aus (z. B. Ablauf der Ruheversicherungsfrist bei KFZ, Nichtzahlung)

## Stornierungsgründe

| Grund-Code | Bezeichnung | Veranlasser | Frist/Bedingung | Beitragserstattung |
|-----------|-------------|-------------|-----------------|-------------------|
| `ORDENTLICHE_KUENDIGUNG` | Ordentliche Kündigung zum Ablauf | VN oder VU | Kündigungsfrist (z. B. 30 Tage vor Hauptfälligkeit) | Nein (läuft regulär aus) |
| `SONDERKUENDIGUNG_BEITRAG` | Sonderkündigung bei Beitragserhöhung | VN | 1 Monat nach Mitteilung der Erhöhung | Ja (ab Wirksamkeitsdatum) |
| `SONDERKUENDIGUNG_SCHADEN` | Sonderkündigung nach Schadenfall | VN oder VU | 1 Monat nach Schadenregulierung | Ja (ab Wirksamkeitsdatum) |
| `WIDERRUF` | Widerruf innerhalb der Widerrufsfrist | VN | 14 Tage nach Vertragsabschluss | Ja (vollständig) |
| `ANFECHTUNG` | Anfechtung wegen arglistiger Täuschung | VU | Unverzüglich nach Kenntnis | Nein (Vertrag war nichtig) |
| `RUECKTRITT` | Rücktritt wegen vorvertraglicher Anzeigepflichtverletzung | VU | Vor Fristablauf | Abhängig vom Einzelfall |
| `NICHTZAHLUNG` | Kündigung wegen Nichtzahlung | VU | Nach Mahnung + Nachfrist (§38 VVG) | Nein |
| `RUHEVERSICHERUNG_ABLAUF` | Automatische Beendigung nach Ablauf der Ruheversicherung | System | 18 Monate nach Beginn der Ruhephase | Nein |
| `STILLLEGUNG` | Stornierung wegen Fahrzeug-Stilllegung (KFZ) | System/Zulassungsstelle | Nach GDV-Meldung | Ja (ab Stilllegungsdatum) |
| `SONSTIGE` | Sonstige Gründe (mit freitextlicher Begründung) | VN oder VU | Einzelfallprüfung | Einzelfallprüfung |

## Veranlasser

| Veranlasser-Code | Bezeichnung | Beschreibung |
|-----------------|-------------|-------------|
| `VERSICHERUNGSNEHMER` | Versicherungsnehmer (VN) | Kunde kündigt den Vertrag |
| `VERSICHERER` | Versicherer (VU) | Unternehmen kündigt den Vertrag |
| `SYSTEM` | System (automatisch) | Automatische Stornierung durch Systemereignis |
| `ZULASSUNGSSTELLE` | Zulassungsstelle (KFZ) | Stornierung aufgrund behördlicher Meldung |

## Ablauf (Hauptszenario)

### Phase 1: Stornierung initiieren
1. Benutzer öffnet den Vertrag und wählt die Funktion **„Vertrag stornieren"**
2. System prüft die Kompetenz des Benutzers über das Kompetenz-System (S8)
3. System prüft, ob der Vertrag im Status `AKTIV` oder `RUHEND` ist
4. System prüft, ob bereits eine **offene Stornierung** für diesen Vertrag existiert:
   - Ja → System zeigt Hinweis: „Es existiert bereits eine offene Stornierung. Bitte diese zuerst abschließen oder verwerfen."
   - Nein → weiter mit Schritt 5
5. System zeigt das **Stornierungsformular** mit folgenden Pflichtangaben:
   - **Stornierungsgrund** (Auswahl aus dem Stornierungsgrund-Katalog)
   - **Veranlasser** (VN, VU, System, Zulassungsstelle)
   - **Einstieg:** Stornierungsantrag oder Stornierungsschwebe
6. Benutzer wählt den Einstiegspfad:
   - **Stornierungsantrag**: Standardpfad – durchläuft Prüfung und Freigabe
   - **Stornierungsschwebe**: Direkter Innendienst-Einstieg – z. B. bei manuellem Eingang einer schriftlichen Kündigung

### Phase 2: Stornierungsdaten erfassen
7. System erzeugt das gewählte Stornierungsobjekt (Antrag oder Schwebe) im Status **„Entwurf"**
8. System zeigt die Vertragsdetails schreibgeschützt an (Produkte, Beiträge, Vertragsstand)
9. Benutzer erfasst die Stornierungsdaten:
   - **Stornierungsdatum** (= gewünschtes Vertragsende):
     - Bei ordentlicher Kündigung: nächster Hauptfälligkeitstermin (Vorschlag)
     - Bei Sonderkündigung: frei wählbar (unter Beachtung der Fristen)
     - Bei Widerruf: sofort bzw. rückwirkend ab Vertragsbeginn
   - **Eingang der Kündigung** (Datum des Kündigungsschreibens / der Willenserklärung)
   - **Begründung** (Freitext, Pflicht bei `SONSTIGE`, optional bei anderen Gründen)
   - **Nachweis-Referenz** (optional: Dokumenten-ID eines eingescannten Kündigungsschreibens)
10. → **Hook `vor_Kuendigung`** wird ausgelöst (spartenspezifische Prüfungen, z. B. Pflichtversicherung bei KFZ → Folgeversicherung prüfen)
11. System führt **Plausibilitätsprüfungen** durch:
    - Kündigungsfrist eingehalten? (abhängig vom Grund)
    - Stornierungsdatum plausibel? (z. B. nicht vor Vertragsbeginn)
    - Grund-spezifische Voraussetzungen erfüllt? (z. B. Schadenfall bei `SONDERKUENDIGUNG_SCHADEN`)
12. System zeigt Plausibilitätsfehler und -hinweise an

### Phase 3: Abrechnung berechnen
13. System berechnet die **Schlussabrechnung**:
    - Beitrag bis zum Stornierungsdatum (pro rata temporis)
    - Bereits gezahlte Beiträge im laufenden Versicherungsjahr
    - **Erstattungsbetrag** (falls Beiträge für den Zeitraum nach Stornierungsdatum bereits eingezogen wurden)
    - Oder **Nachforderung** (falls Beiträge für den Zeitraum bis Stornierungsdatum noch ausstehen)
14. System zeigt die Abrechnungsübersicht:
    - Vertragslaufzeit (Beginn → Stornierungsdatum)
    - Beitrag für den gültigen Zeitraum
    - Bereits bezahlter Betrag
    - Differenz (Erstattung oder Nachforderung)

### Phase 4: Stornierung freigeben

#### Pfad A: Einstieg über Stornierungsantrag
15a. Benutzer wählt **„Stornierung prüfen"**
16a. System führt die Gesamtprüfung durch (Plausibilitäten, Fristen, Spartenhooks)
17a. Prüfung bestanden → Status wechselt zu **„Geprüft"**
18a. Benutzer wählt **„Stornierung freigeben"**
19a. System prüft die Kompetenz `ANTRAG_FREIGEBEN`
20a. System ermittelt den Vorgangstyp → `STORNIERUNG`
21a. System legt eine **Schwebe** an (→ GR-A05)
22a. System prüft **Aussteuerungsregeln**:
    - **Nicht ausgesteuert** → weiter mit Phase 5 (Policierung)
    - **Ausgesteuert** → Schwebe bleibt offen, Innendienst bearbeitet (analog UC-02, Pfad B)

#### Pfad B: Einstieg über Stornierungsschwebe (direkt)
15b. Stornierungsschwebe wurde direkt erzeugt → Innendienst-Sachbearbeiter prüft die Stornierung
16b. Sachbearbeiter wählt **„Stornierung policieren"** (erfordert `SCHWEBE_POLICIEREN`)
17b. Weiter mit Phase 5

### Phase 5: Policierung (Stornierung wirksam)
23. → **Hook `vor_Policierung`** wird ausgelöst (spartenspezifische Prüfungen)
24. System erzeugt einen **abschließenden Vertragsstand** (Version n+1) mit:
    - `gueltig_ab` = aktueller Vertragsstand.gueltig_ab
    - `gueltig_bis` = Stornierungsdatum
    - `jahresbeitrag` = 0,00 (Vertrag beendet)
    - Alle bisherigen Produkte und Daten bleiben als historischer Snapshot erhalten
25. Aktueller Vertragsstand erhält `gueltig_bis` = Stornierungsdatum − 1 Tag
26. System erzeugt einen **Vorgang** mit Vorgangstyp `STORNIERUNG`
27. System setzt den **Vertragsstatus** auf:
    - `GEKUENDIGT` bei ordentlicher Kündigung / Sonderkündigung
    - `STORNIERT` bei Widerruf / Anfechtung / Rücktritt
    - `BEENDET` bei Ablauf (Ruheversicherung, befristeter Vertrag)
28. System setzt `vertragsende` auf das Stornierungsdatum
29. → **Hook `nach_Kuendigung`** wird ausgelöst (spartenspezifische Aktionen, z. B. eVB-Stornierung, Ruheversicherung anbieten bei KFZ)

### Phase 6: Nachgelagerte Systeme versorgen
30. System publiziert **`VertragStorniertEvent`** (Spring Event) → löst folgende Schnittstellenaufrufe aus:

| Schnittstelle | Aktion | Payload-Kern | Mechanismus |
|---------------|--------|-------------|-------------|
| **S4 – DataWarehouse** | Vertragsbewegung `STORNIERUNG` melden | Vertragsnummer, Sparte, Stornierungsgrund, Veranlasser, Stornierungsdatum, letzter Beitrag | Kafka-Topic `vertrag.datawarehouse.bewegungen` |
| **S1 – Provision** | Stornierungsereignis melden | Vertragsnummer, Sparte, Stornierungsgrund, Stornierungsdatum, Vermittler-ID, Stornoabzug | Kafka-Topic `vertrag.provision.ereignisse` (Ereignistyp `STORNIERUNG`) |
| **S2 – Konzerninkasso** | Beitragsforderung beenden + Schlussabrechnung | Vertragsnummer, Stornierungsdatum, Erstattungsbetrag oder Nachforderung, Zahlungsplan-Beendigung | Kafka-Topic `vertrag.inkasso.beitragsforderungen` |
| **S5 – Druck** | Kündigungsbestätigung drucken | Vertragsnummer, Partner, Stornierungsgrund, Stornierungsdatum, Dokumenttyp `KUENDIGUNGSBESTAETIGUNG` | REST `POST /api/v1/druck/auftraege` |

31. System setzt die Schwebe auf Status **„Erledigt"**
32. System bestätigt die erfolgreiche Stornierung und zeigt die Zusammenfassung an

## Alternativszenarien

- **A1: Stornierung verwerfen**
  Der Benutzer kann eine offene Stornierung verwerfen. System fragt: „Stornierung wirklich verwerfen?" Bei Bestätigung wird das Stornierungsobjekt auf Status **„Verworfen"** gesetzt. Der Vertrag bleibt unverändert im Status `AKTIV`.

- **A2: Stornierung über Kundenportal (S7)**
  Der Kunde beantragt eine Kündigung über das Kundenportal. Die Stornierung wird als Stornierungsantrag im Status „Offen" ins System übernommen und **immer ausgesteuert** (Innendienst-Prüfung bei Kundenportal-Kündigungen).

- **A3: Automatische Stornierung (Systemereignis)**
  Das System löst automatisch eine Stornierung aus (z. B. Ablauf der 18-Monats-Ruheversicherungsfrist, Nichtzahlung nach Mahnung). System erzeugt direkt eine Stornierungsschwebe mit Veranlasser `SYSTEM` und policiert automatisch (Dunkelverarbeitung).

- **A4: Stornierung ablehnen (Innendienst)**
  Der Innendienst lehnt eine ausgesteuerte Stornierung ab (z. B. Kündigungsfrist nicht eingehalten, fehlende Nachweise). Die Schwebe wird geschlossen, der Vertrag bleibt aktiv. Der Ablehnungsgrund wird dokumentiert.

- **A5: Stornierung zurückweisen (Nachbearbeitung)**
  Der Innendienst weist die Stornierung zurück (z. B. fehlendes Kündigungsschreiben). Status geht zurück auf „Offen", Ersteller kann nachbessern.

- **A6: Widerruf der Stornierung**
  Vor der Policierung kann der Benutzer / der Kunde die Stornierung zurücknehmen. Das Stornierungsobjekt wird verworfen, der Vertrag bleibt unverändert.

- **A7: Teilstornierung (einzelne Produkte)**
  Soll nicht der gesamte Vertrag, sondern nur einzelne Produkte / Bausteine entfernt werden, ist dies eine **Vertragsänderung** (→ UC-05), keine Stornierung. System weist den Benutzer darauf hin.

- **A8: Nachversicherungsschutz / Nachhaftung**
  Bei bestimmten Sparten (z. B. Haftpflicht) besteht nach Stornierung ein zeitlich begrenzter Nachversicherungsschutz. Dies wird über den Hook `nach_Kuendigung` spartenspezifisch abgebildet.

## Fehlerfälle

- **F1: Vertrag nicht im Status AKTIV oder RUHEND**
  Der Vertrag ist bereits gekündigt, storniert oder beendet. → System zeigt Fehlermeldung: „Dieser Vertrag kann nicht storniert werden (Status: [Status])."

- **F2: Offene Stornierung existiert**
  Für den Vertrag existiert bereits ein offener Stornierungsantrag oder eine offene Stornierungsschwebe. → System zeigt Hinweis und Link zur bestehenden Stornierung.

- **F3: Kündigungsfrist nicht eingehalten**
  Bei ordentlicher Kündigung ist die Kündigungsfrist nicht eingehalten (Stornierungsdatum zu früh). → System zeigt den frühestmöglichen Kündigungstermin an und bietet an, das Datum entsprechend zu korrigieren.

- **F4: Kompetenz nicht ausreichend**
  Benutzer besitzt nicht die Kompetenz `VERTRAG_STORNIEREN`. → System verweigert den Zugriff mit Fehlermeldung.

- **F5: Pflichtversicherung ohne Folgeversicherung (KFZ)**
  Bei KFZ-Haftpflicht prüft der Hook `vor_Kuendigung`, ob eine Folgeversicherung beim selben oder einem anderen Versicherer besteht. → Warnung, keine Blockierung (gesetzliche Pflicht liegt beim VN).

- **F6: Schnittstellenfehler bei Policierung**
  Eine oder mehrere nachgelagerte Schnittstellen sind nicht erreichbar. → Stornierung wird trotzdem policiert (→ GR-A11). Retry-Mechanismen liefern nach.

- **F7: Widerrufsfrist überschritten**
  Benutzer wählt Stornierungsgrund `WIDERRUF`, aber die 14-Tage-Widerrufsfrist ist bereits abgelaufen. → System zeigt Fehlermeldung und schlägt `ORDENTLICHE_KUENDIGUNG` als Alternative vor.

## Geschäftsregeln

| Regel-ID | Beschreibung | Quelle |
|----------|-------------|--------|
| GR-V02 | Ein Vertragsstand gehört immer zu genau einem Vorgang – hier Vorgangstyp `STORNIERUNG` | UC-02, UC-06 |
| GR-V05 | Bei jeder Policierung wird die Vertragsbewegung an das DataWarehouse (S4) gemeldet | UC-05, UC-06 |
| GR-A05 | Bei jeder Freigabe wird eine Schwebe angelegt | UC-02, UC-06 |
| GR-A06 | Aussteuerungsregeln bestimmen, ob der Stornierungsantrag dem Innendienst vorgelegt wird | UC-02, UC-06 |
| GR-A08 | Bei der Policierung werden alle vier Schnittstellen bestückt: S4, S1, S2, S5 | UC-05, UC-06 |
| GR-A09 | Die Policierung erzeugt genau einen Vertragsstand, der genau einem Vorgang zugeordnet ist | UC-02, UC-06 |
| GR-A10 | Bei Stornierung wird der bestehende Vertrag fortgeführt (neuer abschließender Vertragsstand) | UC-05, UC-06 |
| GR-A11 | Schnittstellenfehler führen nicht zum Abbruch – Retry-Mechanismen liefern nach | UC-05, UC-06 |
| GR-A12 | Manuelle Policierung einer Schwebe erfordert `SCHWEBE_POLICIEREN` | UC-02, UC-06 |
| GR-ST-01 | Jede Stornierung erfordert einen **Stornierungsgrund** und einen **Veranlasser** | UC-06 |
| GR-ST-02 | Bei einer Stornierung kann **kein** Änderungsangebot erzeugt werden – nur Stornierungsantrag oder Stornierungsschwebe | UC-06 |
| GR-ST-03 | Die Kündigungsfrist wird anhand des Stornierungsgrunds und der Vertragskonditionen validiert (Kündigungsfrist-Tage aus Vertrag) | UC-06 |
| GR-ST-04 | Bei ordentlicher Kündigung muss die Kündigung vor Ablauf der Kündigungsfrist eingegangen sein (Eingangsdatum ≤ Hauptfälligkeit − Kündigungsfrist) | UC-06 |
| GR-ST-05 | Ein Widerruf ist nur innerhalb von 14 Tagen nach Vertragsabschluss möglich | UC-06 |
| GR-ST-06 | Pro Vertrag darf maximal eine offene Stornierung (Antrag oder Schwebe) existieren | UC-06 |
| GR-ST-07 | Stornierungen über das Kundenportal (S7) werden immer an den Innendienst ausgesteuert | UC-06 |
| GR-ST-08 | Die Schlussabrechnung (Erstattung oder Nachforderung) wird pro rata temporis berechnet: (Stornierungsdatum − letzter Beitragszeitraum-Beginn) / Gesamttage × Jahresbeitrag | UC-06 |
| GR-ST-09 | Nach Policierung der Stornierung wechselt der Vertragsstatus zu `GEKUENDIGT`, `STORNIERT` oder `BEENDET` – abhängig vom Stornierungsgrund | UC-06 |
| GR-ST-10 | Der Vertrag erhält ein `vertragsende`-Datum, das dem Stornierungsdatum entspricht | UC-06 |

## Daten (Ein-/Ausgabe)

### Eingabedaten

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| vertrag_id | UUID | ✅ | Existierender Vertrag im Status AKTIV/RUHEND | Zu stornierender Vertrag |
| einstiegstyp | Enum | ✅ | STORNIERUNGSANTRAG, STORNIERUNGSSCHWEBE | Einstiegspfad |
| stornierungsgrund | Enum | ✅ | Gültiger Wert aus Stornierungsgrund-Katalog | Grund der Stornierung |
| veranlasser | Enum | ✅ | VERSICHERUNGSNEHMER, VERSICHERER, SYSTEM, ZULASSUNGSSTELLE | Wer die Stornierung veranlasst |
| stornierungsdatum | Date | ✅ | ≥ heute (Ausnahme: Widerruf); ≤ nächste Hauptfälligkeit (bei ordentl. Kündigung) | Gewünschtes Vertragsende |
| eingang_kuendigung | Date | ✅ | ≤ heute | Eingangsdatum der Kündigung / Willenserklärung |
| begruendung | String(2000) | Bedingt | Pflicht bei Grund `SONSTIGE`; min. 10 Zeichen | Freitextliche Begründung |
| nachweis_referenz | String(100) | ❌ | – | Dokumenten-ID eines eingescannten Nachweises |

### Ausgabedaten

| Feld | Typ | Beschreibung |
|------|-----|-------------|
| stornierungsobjekt_id | UUID | ID des Stornierungsantrags / -schwebe |
| stornierungsobjekt_typ | Enum | STORNIERUNGSANTRAG oder STORNIERUNGSSCHWEBE |
| status | Enum | Aktueller Status |
| stornierungsdatum | Date | Wirksames Vertragsende |
| erstattungsbetrag | BigDecimal(12,2) | Erstattung an den VN (positiv) oder Nachforderung (negativ) |
| beitrag_bis_stornierung | BigDecimal(12,2) | Beitrag für den gültigen Zeitraum |
| bereits_bezahlt | BigDecimal(12,2) | Im laufenden Zeitraum gezahlte Beiträge |
| neuer_vertragsstatus | Enum | GEKUENDIGT, STORNIERT oder BEENDET |
| vertragsstand_id | UUID | ID des abschließenden Vertragsstands (nach Policierung) |
| vorgang_id | UUID | ID des erzeugten Vorgangs (Typ STORNIERUNG) |

## API-Endpunkte

### Stornierung initiieren und bearbeiten

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| POST | `/api/v1/vertraege/{vertrag_id}/stornierung` | `VERTRAG_STORNIEREN` | Stornierung starten (erzeugt Stornierungsantrag oder -schwebe) |
| GET | `/api/v1/vertraege/{vertrag_id}/stornierung` | `VERTRAG_LESEN` | Aktuelle / historische Stornierungen des Vertrags abrufen |
| GET | `/api/v1/vertraege/{vertrag_id}/stornierung/{id}` | `VERTRAG_LESEN` | Stornierungsdetails abrufen |
| PUT | `/api/v1/vertraege/{vertrag_id}/stornierung/{id}` | `VERTRAG_STORNIEREN` | Stornierungsdaten aktualisieren |
| DELETE | `/api/v1/vertraege/{vertrag_id}/stornierung/{id}` | `VERTRAG_STORNIEREN` | Stornierung verwerfen (logisch → Status „Verworfen") |

### Abrechnung und Prüfung

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| GET | `/api/v1/vertraege/{vertrag_id}/stornierung/{id}/abrechnung` | `VERTRAG_LESEN` | Schlussabrechnung (Erstattung/Nachforderung) berechnen und anzeigen |
| POST | `/api/v1/vertraege/{vertrag_id}/stornierung/{id}/aktionen/pruefen` | `VERTRAG_STORNIEREN` | Gesamtprüfung (Plausibilitäten, Fristen) |

### Freigabe und Policierung

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| POST | `/api/v1/vertraege/{vertrag_id}/stornierung/{id}/aktionen/freigeben` | `ANTRAG_FREIGEBEN` | Stornierungsantrag freigeben → Schwebe + ggf. Policierung |
| POST | `/api/v1/vertraege/{vertrag_id}/stornierung/{id}/aktionen/policieren` | `SCHWEBE_POLICIEREN` | Stornierungsschwebe manuell policieren |

### Metadaten

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| GET | `/api/v1/stornierungsgruende` | `VERTRAG_LESEN` | Stornierungsgrund-Katalog abrufen |
| GET | `/api/v1/vertraege/{vertrag_id}/kuendigungstermine` | `VERTRAG_LESEN` | Nächstmögliche Kündigungstermine je Grund berechnen |

## Events

| Event | Auslöser | Payload | Konsument |
|-------|----------|---------|-----------|
| `StornierungGestartetEvent` | Stornierung initiiert | vertrag_id, stornierungsgrund, veranlasser, benutzer_id | Durchlaufzeiten (UC-04), Audit-Log |
| `StornierungFreigegebenEvent` | Stornierungsantrag freigegeben | vertrag_id, antrag_id, ausgesteuert | Schweben-Dashboard, Innendienst |
| `VertragStorniertEvent` | Stornierung policiert | vertrag_id, vertragsstand_id, stornierungsgrund, veranlasser, stornierungsdatum, erstattungsbetrag | S1 (Provision), S2 (Inkasso), S4 (DWH), S5 (Druck) |

## Statusmodell – Stornierungsobjekt

```
                    ┌──────────┐
                    │ Entwurf  │
                    └────┬─────┘
                         │
              ┌──────────┴──────────┐
              │                     │
        ┌─────▼──────┐       ┌─────▼──────────┐
        │  Stornier.- │       │  Stornier.-    │
        │  Antrag     │       │  Schwebe       │
        │  (Offen)    │       │  (Offen)       │
        └─────┬──────┘       └───────┬────────┘
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
                 │  beendet)   │
                 └─────────────┘

    Sonderstatus: Verworfen (aus jedem Status, außer Policiert)
```

## Nachbedingungen
- Ein **abschließender Vertragsstand** (Version n+1) wurde am bestehenden Vertrag erzeugt
- Der Vertragsstatus wurde auf `GEKUENDIGT`, `STORNIERT` oder `BEENDET` gesetzt
- Das Feld `vertragsende` wurde auf das Stornierungsdatum gesetzt
- Ein **Vorgang** mit Typ `STORNIERUNG` wurde erzeugt und dem Vertragsstand zugeordnet
- Die **Schwebe** wurde auf „Erledigt" gesetzt
- **DataWarehouse (S4)** hat die Stornierungsbewegung erhalten (Kafka)
- **Provision (S1)** hat das Stornierungsereignis erhalten (Kafka)
- **Konzerninkasso (S2)** hat die Schlussabrechnung / Zahlungsplan-Beendigung erhalten (Kafka)
- **Druckschnittstelle (S5)** hat den Druckauftrag für die Kündigungsbestätigung erhalten (REST)
- Spartenspezifische Hooks (`vor_Kuendigung`, `nach_Kuendigung`) wurden ausgeführt
- Alle Statuswechsel sind revisionssicher protokolliert
- Durchlaufzeit-Meilensteine (UC-04) wurden erzeugt

## Neue Kompetenzen (→ Kompetenz-System S8)

| Kompetenz | Beschreibung | Typische Rolle |
|-----------|-------------|----------------|
| `VERTRAG_STORNIEREN` | Stornierung initiieren und bearbeiten | Innendienst, Außendienst |
| `STORNIERUNG_SONDERKUENDIGUNG` | Sonderkündigungen erfassen (erweiterte Gründe) | Innendienst |
| `STORNIERUNG_VU` | Stornierung als Versicherer-Veranlasser durchführen (Anfechtung, Rücktritt) | Innendienst-Senior, Rechtsabteilung |

## Akzeptanzkriterien
- [ ] Aus einem Vertrag im Status AKTIV oder RUHEND kann eine Stornierung gestartet werden
- [ ] Nur Stornierungsantrag oder Stornierungsschwebe als Einstieg möglich – kein Änderungsangebot
- [ ] Stornierungsgrund (Pflicht) und Veranlasser (Pflicht) werden erfasst
- [ ] Kündigungsfrist wird anhand des Stornierungsgrunds und der Vertragskonditionen validiert
- [ ] Widerruf ist nur innerhalb von 14 Tagen nach Vertragsabschluss möglich
- [ ] Schlussabrechnung (Erstattung/Nachforderung) wird pro rata temporis berechnet
- [ ] Stornierungsantrag durchläuft Prüfung, Freigabe und Aussteuerung analog UC-02
- [ ] Stornierungsschwebe kann vom Innendienst direkt policiert werden
- [ ] Bei Policierung wird ein abschließender Vertragsstand erzeugt und der Vertrag beendet
- [ ] Vertragsstatus wechselt korrekt zu GEKUENDIGT, STORNIERT oder BEENDET je nach Grund
- [ ] Alle vier Schnittstellen werden bei Policierung bedient: S4 (DWH), S1 (Provision), S2 (Inkasso), S5 (Druck)
- [ ] Pro Vertrag ist maximal eine offene Stornierung gleichzeitig möglich
- [ ] Stornierungen über das Kundenportal werden immer ausgesteuert
- [ ] Spartenspezifische Hooks (`vor_Kuendigung`, `nach_Kuendigung`) werden ausgelöst
- [ ] Durchlaufzeit-Meilensteine (UC-04) werden korrekt erzeugt

## Wireframe / Skizze

```
┌──────────────────────────────────────────────────────────────────────┐
│  Vertragsstornierung – VN-2026-100001         Status: ⬤ Entwurf    │
├──────────────────────────────────────────────────────────────────────┤
│  Vertrag: VN-2026-100001 │ Sparte: KFZ │ VN: Max Mustermann        │
│  Vertragsbeginn: 01.01.2027 │ Hauptfälligkeit: 01.01.              │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Stornierungsdaten                                                   │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │ Stornierungsgrund:  [Ordentliche Kündigung zum Ablauf    ▼]  │    │
│  │ Veranlasser:        [Versicherungsnehmer (VN)            ▼]  │    │
│  │ Stornierungsdatum:  [01.01.2028 📅]  (= nächste HF)         │    │
│  │ Eingang Kündigung:  [15.10.2027 📅]                          │    │
│  │ Begründung:         [____________________________________]   │    │
│  │ Nachweis:           [📎 Dokument hochladen]                  │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ✅ Kündigungsfrist eingehalten (30 Tage vor 01.01.2028 = 02.12.)   │
│                                                                      │
│  ┌── Schlussabrechnung ────────────────────────────────────────┐    │
│  │                                                              │    │
│  │  Vertragslaufzeit:          01.01.2027 – 01.01.2028         │    │
│  │  Jahresbeitrag:             487,32 €                        │    │
│  │  Beitrag für gültigen Zr.:  487,32 €  (12/12 Monate)       │    │
│  │  Bereits bezahlt:           487,32 €                        │    │
│  │  ─────────────────────────────────────────                  │    │
│  │  Erstattung / Nachford.:      0,00 €  (kein Restbetrag)    │    │
│  │                                                              │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌── Vertragsüberblick (schreibgeschützt) ─────────────────────┐    │
│  │  Produkte: KFZ-HP (312,00 €) │ KFZ-TK (167,11 €)          │    │
│  │  Fahrzeug: VW Golf VIII │ Kennz.: MS-LV 1234               │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Einstieg: ○ Stornierungsantrag  ○ Stornierungsschwebe             │
│                                                                      │
│  [💾 Speichern]  [✔ Prüfen]  [📤 Freigeben]  [🗑 Verwerfen]      │
└──────────────────────────────────────────────────────────────────────┘
```

## Offene Fragen
- Sollen Stornierungsgründe per Admin-UI erweiterbar sein, oder ist der Katalog fest im Code?
- Wie wird bei einer Versicherer-Kündigung (VU) der Kunde benachrichtigt – nur über Druck (S5) oder auch per E-Mail?
- Gibt es eine Sperrfrist nach einer abgelehnten Stornierung, bevor eine neue Stornierung für denselben Vertrag eingeleitet werden kann?
- Soll bei Sonderkündigung nach Schadenfall automatisch die Schadenreferenz aus S6 verknüpft werden?
- Wie wird mit laufenden Änderungen (UC-05) umgegangen, wenn gleichzeitig eine Stornierung eingeleitet wird?
- Gibt es einen Nachweis-Workflow, bei dem ein eingescanntes Kündigungsschreiben vom Innendienst geprüft und bestätigt werden muss?
