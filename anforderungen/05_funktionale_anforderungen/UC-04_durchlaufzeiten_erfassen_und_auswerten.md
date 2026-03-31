# UC-04: Durchlaufzeiten erfassen und auswerten

## Beschreibung
Das System erfasst automatisch die **Durchlaufzeiten** aller Geschäftsvorfälle vom Zeitpunkt der Angebotserstellung bis zur Policierung. Jeder Statuswechsel innerhalb der Prozesskette (Angebot → Antrag → Schwebe → Vertragsstand) wird als **Meilenstein** mit Zeitstempel festgehalten. Auf Basis dieser Daten können Durchschnittswerte, Ausreißer und Engpässe identifiziert, als Kennzahlen (KPIs) dargestellt und als Reports exportiert werden.

Dies ist ein **spartenübergreifender** Querschnitts-Use-Case. Die Durchlaufzeiterfassung gilt automatisch für alle Sparten und Vorgangstypen (Neugeschäft, Nachtrag, Stornierung). Spartenspezifische Meilensteine (z. B. eVB-Erzeugung bei KFZ) können über das Hook-System ergänzt werden.

## Akteure
- **Primär:** Controller, Abteilungsleiter, Prozessmanager
- **Sekundär:** System (automatische Erfassung), DWH (S4, Datenexport)

## Vorbedingungen
- Benutzer ist am System angemeldet und besitzt die Kompetenz **`DURCHLAUFZEITEN_LESEN`** (→ Kompetenz-System S8)
- Mindestens ein abgeschlossener Geschäftsvorfall (Angebot → Policierung) existiert im System
- Die Prozessschritte UC-01 (Angebot) und UC-02 (Antrag) sind aktiv und erzeugen Zeitereignisse

## Auslöser

### Automatische Erfassung (kontinuierlich)
- Jeder Statuswechsel in Angebot, Antrag, Schwebe oder Vertragsstand löst automatisch einen Meilenstein-Eintrag aus (Event-basiert über Spring Events)

### Manuelle Auswertung
- Benutzer öffnet das Dashboard „Durchlaufzeiten" im Berichtswesen-Bereich

## Ablauf (Hauptszenario)

### Phase 1: Automatische Meilenstein-Erfassung (Laufzeit)
1. Bei jedem Statuswechsel einer prozessrelevanten Entität publiziert das System ein **`ProzessMeilensteinEvent`**
2. Der `DurchlaufzeitService` empfängt das Event und erzeugt einen **Meilenstein-Eintrag**:
   - Referenz auf den Geschäftsvorfall (Angebot → Antrag → Schwebe → Vertragsstand)
   - Meilenstein-Typ (z. B. `ANGEBOT_ERSTELLT`, `ANTRAG_FREIGEGEBEN`, `POLICIERT`)
   - Zeitstempel
   - Benutzer-ID (wer den Statuswechsel ausgelöst hat)
   - Optionale spartenspezifische Zusatzdaten
3. System berechnet automatisch die **Durchlaufzeit zwischen aufeinanderfolgenden Meilensteinen** und speichert diese als Differenz (in Minuten)

### Phase 2: Dashboard aufrufen und filtern
4. Benutzer navigiert zu „Berichtswesen → Durchlaufzeiten"
5. System prüft die Kompetenz `DURCHLAUFZEITEN_LESEN` über das Kompetenz-System (S8)
6. System zeigt eine Übersichtsseite mit vordefinierten **KPI-Kacheln**:
   - Ø Gesamtdurchlaufzeit (Angebotserstellung → Policierung)
   - Ø Durchlaufzeit je Prozessabschnitt (Angebot → Antrag, Antrag → Freigabe, Freigabe → Policierung)
   - Anzahl offener Vorgänge mit Überschreitung der SLA-Schwellenwerte
   - Trend (Vergleich zum Vormonat / Vorquartal)
7. Benutzer kann die Anzeige filtern nach:
   - **Zeitraum** (von–bis)
   - **Sparte** (alle / einzelne)
   - **Vorgangstyp** (Neugeschäft, Nachtrag, Stornierung)
   - **Bearbeiter / Team**
   - **Status** (nur abgeschlossene / inkl. laufende Vorgänge)

### Phase 3: Detailanalyse
8. Benutzer wählt einen Prozessabschnitt zur Detailanalyse aus (z. B. „Antrag → Freigabe")
9. System zeigt eine **Häufigkeitsverteilung** (Histogramm) der Durchlaufzeiten
10. System zeigt die **Top-10-Ausreißer** (längste Durchlaufzeiten) mit Verlinkung zum jeweiligen Vorgang
11. Benutzer kann einen einzelnen Vorgang anklicken und die **komplette Meilenstein-Chronologie** einsehen:
    - Tabellarische Darstellung aller Meilensteine mit Zeitstempel und Dauer
    - Visuelle Darstellung als horizontaler Timeline-Balken
12. Bei ausgesteuerten Vorgängen (Schwebe) wird die **Liegezeit im Innendienst** gesondert ausgewiesen

### Phase 4: SLA-Überwachung
13. System prüft alle laufenden Vorgänge gegen konfigurierbare **SLA-Schwellenwerte** (z. B. max. 5 Tage Angebot → Policierung für Neugeschäft)
14. Vorgänge, die einen Schwellenwert überschreiten, werden in einer **Warnliste** hervorgehoben
15. Bei Überschreitung kann das System optional eine **Benachrichtigung** (NOT-XX) an den zuständigen Teamleiter senden
16. SLA-Schwellenwerte sind pro Sparte und Vorgangstyp konfigurierbar (→ Admin-Bereich)

### Phase 5: Export und Weiterleitung
17. Benutzer kann die aktuelle Ansicht als **Report exportieren** (CSV, PDF über Druckschnittstelle S5)
18. Aggregierte Durchlaufzeitdaten werden periodisch an das **DWH (S4)** übermittelt (Kafka-Event oder Batch-Export)

## Meilenstein-Katalog

> Standardmäßig erfasste Meilensteine entlang der Prozesskette UC-01 / UC-02.

| Meilenstein-Typ | Auslösende Entität | Statuswechsel | Phase |
|------------------|--------------------|---------------|-------|
| `ANGEBOT_ERSTELLT` | Angebot | → Entwurf | Anbahnung |
| `ANGEBOT_BERECHNET` | Angebot | → Berechnet | Anbahnung |
| `ANGEBOT_GEPRUEFT` | Angebot | → Geprüft | Anbahnung |
| `ANGEBOT_BEANTRAGT` | Angebot | → Beantragt | Anbahnung |
| `ANTRAG_ERSTELLT` | Antrag | → Offen | Antragsprüfung |
| `ANTRAG_BERECHNET` | Antrag | → Berechnet | Antragsprüfung |
| `ANTRAG_GEPRUEFT` | Antrag | → Geprüft | Antragsprüfung |
| `ANTRAG_FREIGEGEBEN` | Antrag | → Freigegeben | Antragsprüfung |
| `ANTRAG_AUSGESTEUERT` | Antrag | → Ausgesteuert | Innendienst |
| `SCHWEBE_ERSTELLT` | Schwebe | → Offen | Innendienst |
| `SCHWEBE_ERLEDIGT` | Schwebe | → Erledigt | Innendienst |
| `POLICIERT` | Vertragsstand | Vertragsstand erstellt | Policierung |

**Erweiterbarkeit:** Sparten können über den Hook-Mechanismus eigene Meilensteine registrieren (z. B. `EVB_ERZEUGT`, `SF_VALIDIERT`, `VKZ_ZUGEWIESEN` bei KFZ).

## Durchlaufzeit-Abschnitte (berechnete KPIs)

| KPI | Start-Meilenstein | End-Meilenstein | Beschreibung |
|-----|-------------------|-----------------|-------------|
| **Angebotsdauer** | `ANGEBOT_ERSTELLT` | `ANGEBOT_BEANTRAGT` | Zeit von Angebotserstellung bis Beantragung |
| **Antragsprüfdauer** | `ANTRAG_ERSTELLT` | `ANTRAG_FREIGEGEBEN` | Zeit von Antragseröffnung bis Freigabe |
| **Aussteuerungsdauer** | `ANTRAG_AUSGESTEUERT` | `SCHWEBE_ERLEDIGT` | Liegezeit eines ausgesteuerten Antrags im Innendienst |
| **Policierungsdauer** | `ANTRAG_FREIGEGEBEN` | `POLICIERT` | Zeit von Freigabe bis Vertragsstand-Erzeugung |
| **Gesamtdurchlaufzeit** | `ANGEBOT_ERSTELLT` | `POLICIERT` | Ende-zu-Ende-Durchlaufzeit |
| **Nettodurchlaufzeit** | `ANGEBOT_ERSTELLT` | `POLICIERT` | Gesamtdurchlaufzeit minus Liegezeit bei Aussteuerung |

## Alternativszenarien

- **A1: Vorgang ohne Angebot (Direktantrag)**
  Ein Antrag kann ohne vorheriges Angebot erstellt werden. In diesem Fall beginnt die Durchlaufzeitmessung erst beim Meilenstein `ANTRAG_ERSTELLT`. Die Angebotsphase entfällt.

- **A2: Abgebrochener Vorgang**
  Ein Angebot oder Antrag wird verworfen / abgelehnt. Der Vorgang wird mit dem letzten erreichten Meilenstein abgeschlossen. Die Teil-Durchlaufzeiten bis zum Abbruch werden in die Statistik aufgenommen (separat markiert als „abgebrochen").

- **A3: Konfiguration der SLA-Schwellenwerte**
  Ein Benutzer mit Kompetenz `DURCHLAUFZEITEN_KONFIGURIEREN` kann die Schwellenwerte im Admin-Bereich pflegen (→ pro Sparte, Vorgangstyp und Prozessabschnitt).

- **A4: Automatische Benachrichtigung bei SLA-Überschreitung**
  Das System prüft periodisch (Scheduler, z. B. stündlich) alle laufenden Vorgänge gegen die SLA-Schwellenwerte. Bei Überschreitung wird eine Benachrichtigung erzeugt und dem zuständigen Teamleiter zugestellt.

- **A5: Historische Vergleichsanalyse**
  Benutzer kann zwei Zeiträume gegenüberstellen (z. B. Q1 2026 vs. Q1 2027) und die Entwicklung der Durchlaufzeiten im Vergleich betrachten.

## Fehlerfälle

- **F1: Fehlende Berechtigung**
  Benutzer besitzt nicht die Kompetenz `DURCHLAUFZEITEN_LESEN`. → System zeigt Fehlermeldung: „Sie verfügen nicht über die erforderliche Berechtigung." Zugriff wird verweigert.

- **F2: Keine Daten im gewählten Zeitraum**
  Für den gewählten Filter existieren keine Vorgänge. → System zeigt eine Info-Meldung: „Für die gewählten Filterkriterien liegen keine Daten vor."

- **F3: Meilenstein-Event geht verloren**
  Ein Spring Event wird nicht korrekt verarbeitet (z. B. bei Systemfehler). → System schreibt den Fehler ins Error-Log. Ein Batch-Job (täglich) prüft auf fehlende Meilensteine und erzeugt diese nachträglich aus den `erstellt_am` / `geaendert_am`-Zeitstempeln der Quellentitäten (Fallback).

- **F4: DWH-Export fehlgeschlagen**
  Die Übermittlung an das DWH (S4) schlägt fehl. → System wiederholt den Export mit Retry-Strategie (3 Versuche, exponentielles Backoff). Bei dauerhaftem Fehler: Alert an Administrator.

## Geschäftsregeln

| Regel-ID | Beschreibung | Quelle |
|----------|-------------|--------|
| GR-DZ-01 | Jeder Statuswechsel in Angebot, Antrag, Schwebe und Vertragsstand löst automatisch einen Meilenstein-Eintrag aus | UC-04 |
| GR-DZ-02 | Meilensteine sind unveränderlich (immutable) – einmal erzeugt, können sie nicht gelöscht oder verändert werden | UC-04 |
| GR-DZ-03 | Die Gesamtdurchlaufzeit wird erst bei Erreichen des Meilensteins `POLICIERT` als abgeschlossen berechnet | UC-04 |
| GR-DZ-04 | Nur Benutzer mit Kompetenz `DURCHLAUFZEITEN_LESEN` dürfen Auswertungen einsehen | UC-04 |
| GR-DZ-05 | Nur Benutzer mit Kompetenz `DURCHLAUFZEITEN_KONFIGURIEREN` dürfen SLA-Schwellenwerte pflegen | UC-04 |
| GR-DZ-06 | SLA-Schwellenwerte sind pro Sparte und Vorgangstyp separat konfigurierbar | UC-04 |
| GR-DZ-07 | Abgebrochene Vorgänge (verworfene Angebote, abgelehnte Anträge) werden separat ausgewiesen und nicht in die Hauptstatistik eingerechnet | UC-04 |
| GR-DZ-08 | Die Nettodurchlaufzeit schließt Liegezeiten bei Aussteuerung (Innendienst-Bearbeitung) aus | UC-04 |
| GR-DZ-09 | Ein fehlender Meilenstein wird durch den Batch-Fallback spätestens am Folgetag nacherzeugt | UC-04 |

## Daten (Ein-/Ausgabe)

### Eingabedaten – Meilenstein (automatisch erzeugt)

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| referenz_typ | Enum | ✅ | ANGEBOT, ANTRAG, SCHWEBE, VERTRAGSSTAND | Art der auslösenden Entität |
| referenz_id | UUID | ✅ | Existierende Entität | ID der auslösenden Entität |
| vorgang_kette_id | UUID | ✅ | – | Klammer-ID, die Angebot → Antrag → Schwebe → Vertrag verbindet |
| meilenstein_typ | Enum | ✅ | Gültiger Typ aus Katalog | z. B. `ANGEBOT_ERSTELLT` |
| zeitstempel | Timestamp | ✅ | – | Zeitpunkt des Statuswechsels |
| benutzer_id | String(100) | ✅ | – | Auslösender Benutzer |
| sparte | Enum | ✅ | Existierende Sparte | Sparte des Vorgangs |
| vorgangstyp | Enum | ❌ | NEUGESCHAEFT, NACHTRAG, STORNIERUNG | Art des Geschäftsvorfalls (falls bekannt) |
| zusatzdaten | JSONB | ❌ | Valides JSON | Spartenspezifische Zusatzinformationen |

### Eingabedaten – Dashboard-Filter

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| zeitraum_von | Date | ❌ | ≤ zeitraum_bis | Beginn des Auswertungszeitraums |
| zeitraum_bis | Date | ❌ | ≥ zeitraum_von | Ende des Auswertungszeitraums |
| sparte | Enum | ❌ | Existierende Sparte | Filter auf Sparte |
| vorgangstyp | Enum | ❌ | Gültiger Typ | Filter auf Vorgangstyp |
| bearbeiter_id | String(100) | ❌ | – | Filter auf Bearbeiter |
| team | String(100) | ❌ | – | Filter auf Team |
| nur_abgeschlossene | Boolean | ❌ | – | Nur abgeschlossene Vorgänge (Standard: true) |

### Eingabedaten – SLA-Schwellenwert (Konfiguration)

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| sparte | Enum | ✅ | Existierende Sparte | Gültig für welche Sparte |
| vorgangstyp | Enum | ✅ | Gültiger Typ | Gültig für welchen Vorgangstyp |
| prozessabschnitt | Enum | ✅ | Gültiger KPI-Name | Welcher Durchlaufzeit-Abschnitt |
| schwellenwert_minuten | Integer | ✅ | > 0 | Maximale Durchlaufzeit in Minuten |
| warnschwelle_minuten | Integer | ❌ | < schwellenwert | Vorstufe: Warnung ab dieser Dauer |
| benachrichtigung_aktiv | Boolean | ✅ | – | Automatische Benachrichtigung bei Überschreitung? |

### Ausgabedaten – KPI-Übersicht

| Feld | Typ | Beschreibung |
|------|-----|-------------|
| prozessabschnitt | Enum | Name des Durchlaufzeit-Abschnitts |
| anzahl_vorgaenge | Integer | Anzahl der Vorgänge im Zeitraum |
| durchschnitt_minuten | BigDecimal(10,2) | Durchschnittliche Durchlaufzeit |
| median_minuten | BigDecimal(10,2) | Median der Durchlaufzeit |
| p90_minuten | BigDecimal(10,2) | 90. Perzentil |
| p99_minuten | BigDecimal(10,2) | 99. Perzentil |
| min_minuten | BigDecimal(10,2) | Kürzeste Durchlaufzeit |
| max_minuten | BigDecimal(10,2) | Längste Durchlaufzeit |
| sla_ueberschreitungen | Integer | Anzahl der SLA-Überschreitungen |
| trend_prozent | BigDecimal(5,2) | Veränderung zum Vormonat (%) |

### Ausgabedaten – Meilenstein-Chronologie (Einzelvorgang)

| Feld | Typ | Beschreibung |
|------|-----|-------------|
| meilenstein_typ | Enum | Typ des Meilensteins |
| zeitstempel | Timestamp | Zeitpunkt des Meilensteins |
| benutzer_id | String | Auslösender Benutzer |
| dauer_seit_vorgaenger_minuten | BigDecimal(10,2) | Dauer seit dem vorherigen Meilenstein |
| kumulierte_dauer_minuten | BigDecimal(10,2) | Kumulierte Dauer seit dem ersten Meilenstein |

## API-Endpunkte

### Meilensteine (intern, automatisch befüllt)

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| GET | `/api/v1/durchlaufzeiten/meilensteine` | `DURCHLAUFZEITEN_LESEN` | Meilensteine nach Vorgangskette auflisten |
| GET | `/api/v1/durchlaufzeiten/meilensteine/{vorgang_kette_id}` | `DURCHLAUFZEITEN_LESEN` | Chronologie eines einzelnen Vorgangs abrufen |

### KPI-Auswertungen

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| GET | `/api/v1/durchlaufzeiten/kpis` | `DURCHLAUFZEITEN_LESEN` | KPI-Übersicht mit Filtermöglichkeiten |
| GET | `/api/v1/durchlaufzeiten/kpis/{prozessabschnitt}/verteilung` | `DURCHLAUFZEITEN_LESEN` | Häufigkeitsverteilung für einen Prozessabschnitt |
| GET | `/api/v1/durchlaufzeiten/kpis/{prozessabschnitt}/ausreisser` | `DURCHLAUFZEITEN_LESEN` | Top-N Ausreißer (längste Durchlaufzeiten) |
| GET | `/api/v1/durchlaufzeiten/kpis/trend` | `DURCHLAUFZEITEN_LESEN` | Trend-Vergleich über Zeiträume |

### SLA-Überwachung

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| GET | `/api/v1/durchlaufzeiten/sla/warnliste` | `DURCHLAUFZEITEN_LESEN` | Laufende Vorgänge mit SLA-Überschreitung / Warnung |
| GET | `/api/v1/durchlaufzeiten/sla/schwellenwerte` | `DURCHLAUFZEITEN_LESEN` | Konfigurierte Schwellenwerte anzeigen |
| PUT | `/api/v1/durchlaufzeiten/sla/schwellenwerte/{id}` | `DURCHLAUFZEITEN_KONFIGURIEREN` | Schwellenwert aktualisieren |
| POST | `/api/v1/durchlaufzeiten/sla/schwellenwerte` | `DURCHLAUFZEITEN_KONFIGURIEREN` | Neuen Schwellenwert anlegen |
| DELETE | `/api/v1/durchlaufzeiten/sla/schwellenwerte/{id}` | `DURCHLAUFZEITEN_KONFIGURIEREN` | Schwellenwert entfernen |

### Export

| Methode | Pfad | Kompetenz | Beschreibung |
|---------|------|-----------|-------------|
| POST | `/api/v1/durchlaufzeiten/export` | `DURCHLAUFZEITEN_LESEN` | Report exportieren (CSV / PDF über Druckschnittstelle S5) |

## Events

| Event | Auslöser | Payload | Konsument |
|-------|----------|---------|-----------|
| `ProzessMeilensteinEvent` | Statuswechsel in Angebot / Antrag / Schwebe / Vertragsstand | referenz_typ, referenz_id, meilenstein_typ, zeitstempel, benutzer_id, sparte | DurchlaufzeitService |
| `SlaUeberschreitungEvent` | Schwellenwert überschritten | vorgang_kette_id, prozessabschnitt, schwellenwert, aktuelle_dauer | Benachrichtigungs-Service, Dashboard |
| `DurchlaufzeitExportEvent` | Batch-Export an DWH | zeitraum, anzahl_datensaetze | DWH-Schnittstelle (S4) |

## Datenmodell-Erweiterung

> Die folgenden Entitäten ergänzen das bestehende Datenmodell (→ `07_datenmodell.md`).

### ProzessMeilenstein (neu)

| Attribut | Typ | Pflicht | Beschreibung |
|----------|-----|---------|-------------|
| id | UUID | ✅ | Technischer Primärschlüssel |
| vorgang_kette_id | UUID | ✅ | Verbindet Angebot → Antrag → Schwebe → Vertragsstand zu einer Kette |
| referenz_typ | Enum | ✅ | ANGEBOT, ANTRAG, SCHWEBE, VERTRAGSSTAND |
| referenz_id | UUID | ✅ | ID der auslösenden Entität |
| meilenstein_typ | Enum | ✅ | Typ aus dem Meilenstein-Katalog |
| zeitstempel | Timestamp | ✅ | Zeitpunkt des Statuswechsels |
| benutzer_id | String(100) | ✅ | Auslösender Benutzer |
| sparte | Enum | ✅ | Sparte des Vorgangs |
| vorgangstyp | Enum | ❌ | NEUGESCHAEFT, NACHTRAG, STORNIERUNG |
| dauer_seit_vorgaenger_min | BigDecimal(10,2) | ❌ | Berechnete Dauer seit vorherigem Meilenstein (Minuten) |
| zusatzdaten | JSONB | ❌ | Spartenspezifische Zusatzinformationen |
| erstellt_am | Timestamp | ✅ | Erstellzeitpunkt des Eintrags |

**Constraints:**
- `vorgang_kette_id` + `meilenstein_typ` UNIQUE (kein doppelter Meilenstein pro Kette)
- Immutable: kein UPDATE/DELETE (→ GR-DZ-02)
- Index auf `vorgang_kette_id`, `sparte`, `zeitstempel`

**Partitionierung:** Range-Partitionierung nach `zeitstempel` (monatlich) für effiziente Abfragen und Retention

### SlaSchwellenwert (neu)

| Attribut | Typ | Pflicht | Beschreibung |
|----------|-----|---------|-------------|
| id | UUID | ✅ | Technischer Primärschlüssel |
| sparte | Enum | ✅ | Gültig für welche Sparte |
| vorgangstyp | Enum | ✅ | Gültig für welchen Vorgangstyp |
| prozessabschnitt | Enum | ✅ | Welcher Durchlaufzeit-KPI |
| schwellenwert_minuten | Integer | ✅ | Maximale Durchlaufzeit |
| warnschwelle_minuten | Integer | ❌ | Vorstufe: Warnung ab dieser Dauer |
| benachrichtigung_aktiv | Boolean | ✅ | Automatische Benachrichtigung? |
| erstellt_am | Timestamp | ✅ | Erstellzeitpunkt |
| geaendert_am | Timestamp | ✅ | Letzte Änderung |

**Constraints:**
- `sparte` + `vorgangstyp` + `prozessabschnitt` UNIQUE
- `warnschwelle_minuten < schwellenwert_minuten`

**Historisierung:** ✅ Hibernate Envers

### VorgangKette (neu – Verknüpfungstabelle)

| Attribut | Typ | Pflicht | Beschreibung |
|----------|-----|---------|-------------|
| id | UUID | ✅ | Technischer Primärschlüssel (= vorgang_kette_id) |
| angebot_id | UUID (FK) | ❌ | Referenz auf Angebot (NULL bei Direktantrag) |
| antrag_id | UUID (FK) | ✅ | Referenz auf Antrag |
| schwebe_id | UUID (FK) | ❌ | Referenz auf Schwebe |
| vertrag_id | UUID (FK) | ❌ | Referenz auf Vertrag (nach Policierung) |
| vertragsstand_id | UUID (FK) | ❌ | Referenz auf Vertragsstand (nach Policierung) |
| sparte | Enum | ✅ | Sparte |
| vorgangstyp | Enum | ❌ | NEUGESCHAEFT, NACHTRAG, STORNIERUNG |
| status | Enum | ✅ | LAUFEND, ABGESCHLOSSEN, ABGEBROCHEN |
| gestartet_am | Timestamp | ✅ | Zeitpunkt des ersten Meilensteins |
| abgeschlossen_am | Timestamp | ❌ | Zeitpunkt des letzten Meilensteins (bei Abschluss) |
| gesamtdauer_minuten | BigDecimal(10,2) | ❌ | Berechnete Gesamtdurchlaufzeit |
| nettodauer_minuten | BigDecimal(10,2) | ❌ | Gesamtdauer minus Liegezeit Aussteuerung |

**Constraints:**
- Mindestens `antrag_id` muss gesetzt sein
- Index auf `sparte`, `status`, `gestartet_am`

## Neue Kompetenzen (→ Kompetenz-System S8)

| Kompetenz | Beschreibung | Typische Rolle |
|-----------|-------------|----------------|
| `DURCHLAUFZEITEN_LESEN` | Durchlaufzeit-Dashboard und Auswertungen einsehen | Controller, Abteilungsleiter, Prozessmanager |
| `DURCHLAUFZEITEN_KONFIGURIEREN` | SLA-Schwellenwerte pflegen, Benachrichtigungsregeln verwalten | Prozessmanager, System-Administrator |

## Nachbedingungen
- Jeder Statuswechsel in der Prozesskette ist als unveränderlicher Meilenstein persistiert
- Die Vorgangskette verbindet Angebot → Antrag → Schwebe → Vertragsstand lückenlos
- KPI-Berechnung (Durchschnitt, Median, Perzentile, Trend) ist für jeden Filterkontext abrufbar
- SLA-Überschreitungen werden in Echtzeit erkannt und optional gemeldet
- Aggregierte Daten stehen dem DWH (S4) für weiterführende Analysen zur Verfügung

## Akzeptanzkriterien
- [ ] Jeder Statuswechsel in Angebot, Antrag, Schwebe und Vertragsstand erzeugt automatisch einen Meilenstein-Eintrag
- [ ] Meilensteine sind unveränderlich (kein UPDATE/DELETE möglich)
- [ ] Dashboard zeigt Ø Gesamtdurchlaufzeit, Ø je Prozessabschnitt, Trend und SLA-Überschreitungen
- [ ] Filter nach Zeitraum, Sparte, Vorgangstyp, Bearbeiter und Team funktionieren korrekt
- [ ] Häufigkeitsverteilung und Top-10-Ausreißer sind je Prozessabschnitt abrufbar
- [ ] Einzelvorgang zeigt komplette Meilenstein-Chronologie mit Dauer zwischen Meilensteinen
- [ ] Nettodurchlaufzeit wird korrekt berechnet (Gesamtdauer minus Aussteuerungszeit)
- [ ] SLA-Schwellenwerte sind pro Sparte und Vorgangstyp konfigurierbar
- [ ] Bei SLA-Überschreitung wird (optional) eine Benachrichtigung an den Teamleiter erzeugt
- [ ] Fehlende Meilensteine werden durch Batch-Fallback spätestens am Folgetag nacherzeugt
- [ ] Export als CSV und PDF funktioniert
- [ ] Spartenspezifische Meilensteine können über den Hook-Mechanismus ergänzt werden

## Wireframe / Skizze

```
┌────────────────────────────────────────────────────────────────────┐
│  Durchlaufzeiten – Dashboard                                      │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Filter: [Zeitraum ▼] [Sparte: Alle ▼] [Typ: Alle ▼] [🔍 Filtern]│
│                                                                    │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌───────────┐ │
│  │ Ø Gesamt     │ │ Ø Angebot→   │ │ Ø Antrag→    │ │ SLA-      │ │
│  │  3,2 Tage    │ │   Antrag     │ │   Policierung│ │ Verletzung│ │
│  │ ▼ -8% vs. VM │ │  1,1 Tage    │ │  2,1 Tage    │ │   12      │ │
│  └──────────────┘ └──────────────┘ └──────────────┘ └───────────┘ │
│                                                                    │
│  Durchlaufzeiten nach Prozessabschnitt                             │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Abschnitt           │  Ø     │ Median│ P90   │ SLA   │ ▲▼  │  │
│  ├─────────────────────┼────────┼───────┼───────┼───────┼──────┤  │
│  │ Angebotsdauer       │ 0,8 T  │ 0,5 T │ 1,5 T │ 2,0 T │  ✅ │  │
│  │ Antragsprüfdauer    │ 1,2 T  │ 0,9 T │ 2,8 T │ 3,0 T │  ✅ │  │
│  │ Aussteuerungsdauer  │ 3,1 T  │ 2,5 T │ 6,0 T │ 5,0 T │  ⚠️ │  │
│  │ Policierungsdauer   │ 0,1 T  │ 0,0 T │ 0,2 T │ 1,0 T │  ✅ │  │
│  │ Gesamtdurchlaufzeit │ 3,2 T  │ 2,4 T │ 6,5 T │ 7,0 T │  ✅ │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                         [📄 Export CSV] [📄 PDF]   │
│                                                                    │
│  Top-10 Ausreißer (Gesamtdurchlaufzeit)                           │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │ 1. AN-2026-004711  │ KFZ │ 14,2 Tage │ Ausgesteuert │  [→]  │ │
│  │ 2. AN-2026-003289  │ KFZ │ 11,8 Tage │ Ausgesteuert │  [→]  │ │
│  │ 3. AN-2026-005102  │ HR  │  9,5 Tage │ Abgeschlossen│  [→]  │ │
│  └───────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────┐
│  Meilenstein-Chronologie – AN-2026-004711                         │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ●────────●────●────●────────────────●────●─────●──────●           │
│  Ang.     Ber. Gepr.Bean.            Ausst.     Erl.   Pol.       │
│  erstellt                            5,3 T             0,1 T      │
│  14:22    14:35 14:38 14:42          +5,3 Tage  +3,1T  +0,1T      │
│                                                                    │
│  ┌──────────────────┬──────────┬──────────┬──────────────────────┐ │
│  │ Meilenstein      │ Zeitpkt. │ Δ Dauer  │ Bearbeiter           │ │
│  ├──────────────────┼──────────┼──────────┼──────────────────────┤ │
│  │ ANGEBOT_ERSTELLT │ 01.03.   │ –        │ ad-mueller           │ │
│  │ ANGEBOT_BERECHN. │ 01.03.   │ 13 min   │ ad-mueller           │ │
│  │ ANGEBOT_GEPRUEFT │ 01.03.   │  3 min   │ ad-mueller           │ │
│  │ ANGEBOT_BEANTR.  │ 01.03.   │  4 min   │ ad-mueller           │ │
│  │ ANTRAG_ERSTELLT  │ 01.03.   │  0 min   │ System               │ │
│  │ ANTRAG_AUSGEST.  │ 01.03.   │  2 min   │ System               │ │
│  │ SCHWEBE_ERSTELLT │ 01.03.   │  0 min   │ System               │ │
│  │ SCHWEBE_ERLEDIGT │ 06.03.   │ 7.632 min│ id-schmidt           │ │
│  │ ANTRAG_FREIGEG.  │ 06.03.   │  5 min   │ id-schmidt           │ │
│  │ POLICIERT        │ 06.03.   │ 12 min   │ System               │ │
│  └──────────────────┴──────────┴──────────┴──────────────────────┘ │
│                                                                    │
│  Gesamt: 14,2 Tage │ Netto (ohne Aussteuerung): 8,9 Tage         │
└────────────────────────────────────────────────────────────────────┘
```

## Technische Hinweise
- **Event-basierte Erfassung:** Die Meilensteine werden über Spring Events erfasst – die bestehenden Statuswechsel in UC-01/UC-02 publizieren bereits Events, die um den `ProzessMeilensteinEvent`-Listener erweitert werden
- **Partitionierung:** Die Tabelle `prozess_meilenstein` wird nach `zeitstempel` monatlich partitioniert (PostgreSQL Range Partitioning), um effiziente Abfragen über große Zeiträume zu ermöglichen
- **Materialized Views:** Für die KPI-Berechnung (Durchschnitt, Median, Perzentile) können PostgreSQL Materialized Views genutzt werden, die periodisch (z. B. stündlich) aktualisiert werden
- **Retention:** Meilensteine älter als 5 Jahre können nach Abstimmung mit dem DWH archiviert werden (Partitionen können effizient entfernt werden)
- **VorgangKette-Erzeugung:** Die `vorgang_kette_id` wird beim ersten Meilenstein (`ANGEBOT_ERSTELLT` bzw. `ANTRAG_ERSTELLT`) erzeugt und über die gesamte Prozesskette weitergereicht (im Angebot/Antrag als zusätzliches Feld oder über eine separate Verknüpfungstabelle)

## Offene Fragen
- Sollen Durchlaufzeiten auch für Bestandsprozesse (Nachtrag, Kündigung) gemessen werden, die keinen Angebots-Schritt haben?
- Wie wird die `vorgang_kette_id` technisch am besten über die Entitäten (Angebot → Antrag → Schwebe → Vertragsstand) transportiert – als Feld in jeder Entität oder ausschließlich über die VorgangKette-Tabelle?
- Sollen die SLA-Schwellenwerte auch saisonale Anpassungen unterstützen (z. B. höhere Schwellenwerte im November für KFZ-Saisonwechsel)?
- Welche Granularität soll der DWH-Export haben – aggregiert pro Tag/Woche oder jeden einzelnen Meilenstein?
- Soll das Dashboard auch eine **Echtzeitansicht** (Live-Aktualisierung per WebSocket) unterstützen?
