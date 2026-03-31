# Datenmodell – Versicherungsverwaltung

> Spartenübergreifendes Kerndatenmodell und spartenspezifische Erweiterung (KFZ).
> Technologiebasis: PostgreSQL 17, Spring Data JPA (Hibernate 6), Hibernate Envers (Historisierung), JSONB für spartenspezifische Flexibilität.
> Alle Entitäten verwenden UUID als Primärschlüssel. Fachliche IDs (z. B. Angebotsnummer) sind separat.

---

## Übersicht (ER-Diagramm – Spartenübergreifend)

```
                                    ┌─────────────────┐
                                    │    Benutzer      │
                                    │ (extern: S8)     │
                                    └────────┬────────┘
                                             │ erstellt / bearbeitet
                                             ▼
┌─────────────────┐  1          n  ┌─────────────────┐  1          n  ┌─────────────────┐
│     Partner     │───────────────►│     Angebot     │───────────────►│ AngebotProdukt  │
│ (Kunde/VN)      │                │                 │                │                 │
└────────┬────────┘                └────────┬────────┘                └────────┬────────┘
         │                                  │                                  │
         │ 1                          erzeugt│ 0..1                             │ n
         │                                  ▼                                  ▼
         │ n                       ┌─────────────────┐  1          n  ┌─────────────────┐
         ├────────────────────────►│     Antrag      │───────────────►│  AntragProdukt  │
         │                         └────────┬────────┘                └─────────────────┘
         │                                  │
         │                         freigeben│
         │                                  ▼
         │                         ┌─────────────────┐
         │                         │     Schwebe     │
         │                         └────────┬────────┘
         │                                  │
         │                         erledigen│ (bei Dunkelverarbeitung)
         │                                  ▼
         │ n                       ┌─────────────────┐
         ├────────────────────────►│     Vertrag     │
         │                         └────────┬────────┘
         │                                  │ 1
         │                                  │
         │                                  │ n
         │                         ┌────────▼────────┐
         │                         │    Vorgang      │
         │                         │ (Neu/Änd/Storno)│
         │                         └────────┬────────┘
         │                                  │ 1
         │                                  │
         │                                  │ n
         │                         ┌────────▼────────┐  1    n  ┌──────────────────────┐
         │                         │ Vertragsstand   │─────────►│VertragsstandProdukt  │
         │                         └─────────────────┘          └──────────────────────┘
         │
         │ n                       ┌─────────────────┐
         └────────────────────────►│     Schaden     │ (Referenz zu S6)
                                   └─────────────────┘
```

## Produktkonfiguration (ER-Diagramm)

```
┌─────────────────┐  1          n  ┌─────────────────┐
│     Sparte      │───────────────►│     Produkt     │
│  (Konfiguration)│                │  (Konfiguration) │
└─────────────────┘                └────────┬────────┘
                                            │ 1
                                   ┌────────┼────────────────────┐
                                   │ n      │ n                  │ n
                          ┌────────▼──────┐ │           ┌────────▼────────┐
                          │Deckungsbaust. │ │           │  Tarifmerkmal   │
                          └───────────────┘ │           └─────────────────┘
                                   ┌────────▼────────┐
                                   │ ProduktAbhäng.  │
                                   └─────────────────┘
```

---

## 1. Kerndatenmodell (Spartenübergreifend)

### 1.1 Partner

> Zentrale Entität für Versicherungsnehmer, Versicherte Personen und andere Beteiligte.

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | `a1b2c3d4-...` |
| partnernummer | String(20) | ✅ | Fachliche, eindeutige Kennung | `KD-2026-004711` |
| partnertyp | Enum | ✅ | Typ des Partners | `NATUERLICHE_PERSON`, `JURISTISCHE_PERSON` |
| anrede | Enum | ❌ | Anrede | `HERR`, `FRAU`, `DIVERS`, `FIRMA` |
| titel | String(50) | ❌ | Akademischer Titel | `Dr.` |
| vorname | String(100) | Bedingt | Pflicht bei natürlicher Person | `Max` |
| nachname | String(100) | ✅ | Nachname / Firmenname | `Mustermann` |
| geburtsdatum | Date | Bedingt | Pflicht bei natürlicher Person | `1985-03-15` |
| strasse | String(200) | ✅ | Straße | `Kolde-Ring` |
| hausnummer | String(20) | ✅ | Hausnummer | `21` |
| plz | String(5) | ✅ | Postleitzahl | `48151` |
| ort | String(100) | ✅ | Ort | `Münster` |
| land | String(2) | ✅ | ISO 3166-1 Alpha-2 Ländercode | `DE` |
| email | String(255) | ❌ | E-Mail-Adresse | `max@example.de` |
| telefon | String(30) | ❌ | Telefonnummer | `+49 251 702-0` |
| mobiltelefon | String(30) | ❌ | Mobilnummer | `+49 170 1234567` |
| iban | String(34) | ❌ | Bankverbindung | `DE89370400440532013000` |
| bic | String(11) | ❌ | BIC der Bank | `COBADEFFXXX` |
| steuernummer | String(20) | ❌ | Steuernummer (bei Gewerbe) | |
| erstellt_am | Timestamp | ✅ | Erstellzeitpunkt | `2026-01-15T10:30:00` |
| geaendert_am | Timestamp | ✅ | Letzte Änderung | `2026-03-19T14:22:00` |

**Constraints:**
- `partnernummer` UNIQUE
- `vorname` NOT NULL wenn `partnertyp = NATUERLICHE_PERSON`
- `geburtsdatum` NOT NULL wenn `partnertyp = NATUERLICHE_PERSON`

**Historisierung:** ✅ Hibernate Envers (alle Änderungen revisionssicher protokolliert)

---

### 1.2 Angebot

> Unverbindlicher Versicherungsvorschlag → UC-01.

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| angebotsnummer | String(20) | ✅ | Fachliche, eindeutige Kennung | `AG-2026-001234` |
| status | Enum | ✅ | Aktueller Status (→ Statusmodell) | `BERECHNET` |
| partner_id | UUID (FK) | ✅ | Referenz auf Partner (VN) | |
| sparte | Enum | ✅ | Versicherungssparte | `KFZ` |
| vertragsbeginn | Date | ✅ | Gewünschter Versicherungsbeginn | `2027-01-01` |
| laufzeit_monate | Integer | ✅ | Vertragslaufzeit in Monaten | `12` |
| zahlungsweise | Enum | ✅ | Beitragszahlweise | `JAEHRLICH` |
| berechneter_jahresbeitrag | BigDecimal(12,2) | ❌ | Berechneter Jahresbeitrag brutto (EUR) | `487.32` |
| angebot_behalten | Boolean | ❌ | Soll Angebot nach Beantragung erhalten bleiben? | `true` |
| gueltig_bis | Date | ❌ | Verfallsdatum des Angebots | `2026-06-19` |
| spartenspezifische_daten | JSONB | ❌ | Flexible spartenspezifische Attribute (→ Abschnitt 7) | `{"fahrzeugart": "PKW", ...}` |
| erstellt_von | String(100) | ✅ | Benutzer-ID des Erstellers | `ad-mueller` |
| erstellt_am | Timestamp | ✅ | Erstellzeitpunkt | |
| geaendert_am | Timestamp | ✅ | Letzte Änderung | |

**Constraints:**
- `angebotsnummer` UNIQUE
- `vertragsbeginn >= CURRENT_DATE` bei Neuanlage
- Status-Übergänge nur gemäß Statusmodell (→ 5.1)

**Historisierung:** ✅ Hibernate Envers

---

### 1.3 Antrag

> Verbindlicher Versicherungsantrag → UC-02. Wird aus Angebot erzeugt oder direkt erfasst.

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| antragsnummer | String(20) | ✅ | Fachliche, eindeutige Kennung | `AN-2026-005678` |
| status | Enum | ✅ | Aktueller Status (→ Statusmodell) | `OFFEN` |
| angebot_id | UUID (FK) | ❌ | Referenz auf zugrunde liegendes Angebot (optional) | |
| partner_id | UUID (FK) | ✅ | Referenz auf Partner (VN) | |
| sparte | Enum | ✅ | Versicherungssparte | `KFZ` |
| vertragsbeginn | Date | ✅ | Gewünschter Versicherungsbeginn | `2027-01-01` |
| laufzeit_monate | Integer | ✅ | Vertragslaufzeit in Monaten | `12` |
| zahlungsweise | Enum | ✅ | Beitragszahlweise | `JAEHRLICH` |
| berechneter_jahresbeitrag | BigDecimal(12,2) | ❌ | Berechneter Jahresbeitrag brutto (EUR) | `487.32` |
| ablehnungsgrund | Text | ❌ | Begründung bei Ablehnung (min. 10 Zeichen) | |
| vorgangstyp | Enum | ❌ | Automatisch ermittelt bei Freigabe | `NEUGESCHAEFT` |
| spartenspezifische_daten | JSONB | ❌ | Flexible spartenspezifische Attribute | |
| erstellt_von | String(100) | ✅ | Benutzer-ID des Erstellers | |
| erstellt_am | Timestamp | ✅ | Erstellzeitpunkt | |
| geaendert_am | Timestamp | ✅ | Letzte Änderung | |

**Constraints:**
- `antragsnummer` UNIQUE
- `ablehnungsgrund` NOT NULL und LENGTH ≥ 10 wenn `status = ABGELEHNT`

**Historisierung:** ✅ Hibernate Envers

---

### 1.4 Schwebe

> Offener Vorgang zwischen Antragsfreigabe und Vertragsstand-Erzeugung → UC-02.

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| schwebenummer | String(20) | ✅ | Fachliche, eindeutige Kennung | `SW-2026-000042` |
| status | Enum | ✅ | Aktueller Status | `OFFEN` |
| antrag_id | UUID (FK) | ✅ | Referenz auf den auslösenden Antrag | |
| aussteuerungsgrund | String(500) | ❌ | Grund der Aussteuerung (falls zutreffend) | `Deckungssumme > 10 Mio. EUR` |
| aussteuerungsregel_id | UUID (FK) | ❌ | Referenz auf die ausgelöste Aussteuerungsregel | |
| zustaendiges_team | String(100) | ❌ | Innendienst-Team (bei Aussteuerung) | `ID-KFZ-Sonder` |
| zustaendiger_bearbeiter | String(100) | ❌ | Zugewiesener Sachbearbeiter | `id-schmidt` |
| wiedervorlage_am | Date | ❌ | Datum für Wiedervorlage | `2026-04-01` |
| erstellt_am | Timestamp | ✅ | Erstellzeitpunkt | |
| erledigt_am | Timestamp | ❌ | Zeitpunkt der Erledigung / Schließung | |

**Constraints:**
- `schwebenummer` UNIQUE
- Immer genau eine Schwebe pro Antrag-Freigabe (GR-06)

**Historisierung:** ✅ Hibernate Envers

---

### 1.5 Vertrag

> Versicherungsvertrag – entsteht durch erfolgreiche Policierung.

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| vertragsnummer | String(20) | ✅ | Fachliche, eindeutige Kennung | `VN-2026-100001` |
| status | Enum | ✅ | Aktueller Vertragsstatus | `AKTIV` |
| partner_id | UUID (FK) | ✅ | Referenz auf Partner (VN) | |
| sparte | Enum | ✅ | Versicherungssparte | `KFZ` |
| vertragsbeginn | Date | ✅ | Versicherungsbeginn | `2027-01-01` |
| vertragsende | Date | ❌ | Vertragsende (bei befristetem Vertrag / Kündigung) | `2028-01-01` |
| laufzeit_monate | Integer | ✅ | Vertragslaufzeit | `12` |
| hauptfaelligkeit | Date | ✅ | Stichtag der Hauptfälligkeit | `2027-01-01` |
| zahlungsweise | Enum | ✅ | Beitragszahlweise | `JAEHRLICH` |
| aktueller_jahresbeitrag | BigDecimal(12,2) | ✅ | Aktueller Jahresbeitrag brutto (EUR) | `487.32` |
| kuendigungsfrist_tage | Integer | ✅ | Kündigungsfrist in Tagen | `30` |
| spartenspezifische_daten | JSONB | ❌ | Flexible spartenspezifische Attribute | |
| erstellt_am | Timestamp | ✅ | Erstellzeitpunkt | |
| geaendert_am | Timestamp | ✅ | Letzte Änderung | |

**Constraints:**
- `vertragsnummer` UNIQUE
- Genau ein aktueller Vertragsstand pro Vertrag (der neueste gültige)

**Historisierung:** ✅ Hibernate Envers + PostgreSQL Temporal Tables (system-versioned)

---

### 1.6 Vorgang

> Fachlicher Geschäftsvorfall, dem ein Vertragsstand zugeordnet wird → UC-02.

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| vorgangsnummer | String(20) | ✅ | Fachliche, eindeutige Kennung | `VG-2026-000001` |
| vorgangstyp | Enum | ✅ | Art des Geschäftsvorfalls | `NEUGESCHAEFT` |
| vertrag_id | UUID (FK) | ✅ | Referenz auf den Vertrag | |
| antrag_id | UUID (FK) | ✅ | Referenz auf den auslösenden Antrag | |
| schwebe_id | UUID (FK) | ✅ | Referenz auf die zugehörige Schwebe | |
| beschreibung | String(500) | ❌ | Optionale Beschreibung | `Neugeschäft PKW VW Golf VIII` |
| erstellt_von | String(100) | ✅ | Benutzer-ID | |
| erstellt_am | Timestamp | ✅ | Erstellzeitpunkt | |

**Constraints:**
- `vorgangsnummer` UNIQUE
- `vorgangstyp` wird automatisch aus Antragskontext ermittelt (GR-09)

**Historisierung:** ✅ Hibernate Envers

---

### 1.7 Vertragsstand

> Konkreter, zu einem Zeitpunkt gültiger Zustand eines Vertrags. Revisionssicher → UC-02.

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| vertrag_id | UUID (FK) | ✅ | Referenz auf den Vertrag | |
| vorgang_id | UUID (FK) | ✅ | Referenz auf den auslösenden Vorgang | |
| version | Integer | ✅ | Versionsnummer (aufsteigend) | `1` |
| gueltig_ab | Date | ✅ | Beginn der Gültigkeit | `2027-01-01` |
| gueltig_bis | Date | ❌ | Ende der Gültigkeit (NULL = aktuell gültig) | |
| jahresbeitrag | BigDecimal(12,2) | ✅ | Jahresbeitrag brutto (EUR) zu diesem Stand | `487.32` |
| zahlungsweise | Enum | ✅ | Beitragszahlweise zu diesem Stand | `JAEHRLICH` |
| spartenspezifische_daten | JSONB | ❌ | Spartenspezifische Attribute zu diesem Stand | |
| erstellt_von | String(100) | ✅ | Benutzer-ID | |
| erstellt_am | Timestamp | ✅ | Erstellzeitpunkt (= Policierungszeitpunkt) | |

**Constraints:**
- Immer genau einem Vorgang zugeordnet (GR-08)
- Nur ein Vertragsstand mit `gueltig_bis = NULL` pro Vertrag (= aktueller Stand)

**Historisierung:** ✅ Implizit (jeder Vertragsstand ist eine Revision); zusätzlich Hibernate Envers

---

### 1.8 Schaden (Referenz)

> Schadendaten werden im externen System Schadenverwaltung (S6) geführt. Diese Entität hält nur die vertragsseitige Referenz.

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| schadennummer | String(20) | ✅ | Fachliche Kennung (aus Schadenverwaltung) | `SD-2026-000123` |
| vertrag_id | UUID (FK) | ✅ | Referenz auf den betroffenen Vertrag | |
| schadendatum | Date | ✅ | Datum des Schadenereignisses | `2026-05-10` |
| meldedatum | Date | ✅ | Datum der Schadenmeldung | `2026-05-11` |
| status | Enum | ✅ | Status aus Schadenverwaltung | `GEMELDET` |
| schadenart | String(100) | ❌ | Art des Schadens | `Auffahrunfall` |
| regulierungsbetrag | BigDecimal(12,2) | ❌ | Regulierter Betrag (aus S6) | `3250.00` |
| auswirkung_sf | Boolean | ❌ | Wirkt sich auf SF-Klasse aus? | `true` |
| erstellt_am | Timestamp | ✅ | Übernahme-Zeitpunkt | |
| geaendert_am | Timestamp | ✅ | Letzte Aktualisierung aus S6 | |

**Historisierung:** ✅ Hibernate Envers

---

## 2. Produktkonfiguration (Spartenübergreifend)

> Konfigurationsdaten, die über eine Admin-Oberfläche oder Datenimport gepflegt werden. Nicht vom Endbenutzer bearbeitbar.

### 2.1 Sparte (Konfiguration)

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| kuerzel | String(10) | ✅ | Eindeutiges Kürzel | `KFZ` |
| bezeichnung | String(100) | ✅ | Anzeigename | `Kraftfahrzeugversicherung` |
| status | Enum | ✅ | Aktiv / Inaktiv | `AKTIV` |
| sortierung | Integer | ✅ | Reihenfolge in der UI | `1` |
| beschreibung | Text | ❌ | Beschreibung | |
| erstellt_am | Timestamp | ✅ | | |
| geaendert_am | Timestamp | ✅ | | |

**Constraints:**
- `kuerzel` UNIQUE

---

### 2.2 Produkt (Konfiguration)

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| produkt_id | String(20) | ✅ | Fachliche Produkt-ID | `KFZ-HP` |
| sparte_id | UUID (FK) | ✅ | Referenz auf die Sparte | |
| bezeichnung | String(200) | ✅ | Produktname | `KFZ-Haftpflichtversicherung` |
| beschreibung | Text | ❌ | Ausführliche Beschreibung | |
| produkttyp | Enum | ✅ | Hauptprodukt / Zusatzbaustein / Kurzfristig | `HAUPTPRODUKT` |
| zielgruppe | Enum | ✅ | Privat / Gewerbe / Beide | `BEIDE` |
| pflichtprodukt | Boolean | ✅ | Gesetzlich vorgeschrieben? | `true` (HP) |
| mindestlaufzeit_monate | Integer | ✅ | Mindestlaufzeit | `12` |
| kuendigungsfrist_tage | Integer | ✅ | Kündigungsfrist | `30` |
| status | Enum | ✅ | Aktiv / Inaktiv / Geplant | `AKTIV` |
| gueltig_ab | Date | ✅ | Tarif gültig ab | `2026-01-01` |
| gueltig_bis | Date | ❌ | Tarif gültig bis (NULL = unbefristet) | |
| sortierung | Integer | ✅ | Reihenfolge | `1` |
| erstellt_am | Timestamp | ✅ | | |
| geaendert_am | Timestamp | ✅ | | |

**Constraints:**
- `produkt_id` UNIQUE
- Mindestens ein Pflichtprodukt pro Sparte (z. B. KFZ-HP)

---

### 2.3 Deckungsbaustein (Konfiguration)

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| produkt_id | UUID (FK) | ✅ | Referenz auf das Produkt | |
| bezeichnung | String(200) | ✅ | Name des Bausteins | `Personenschäden` |
| beschreibung | Text | ❌ | Beschreibung | |
| pflicht_optional | Enum | ✅ | Pflicht / Optional / Inklusive | `PFLICHT` |
| versicherungssumme_text | String(200) | ❌ | Beschreibung der VS | `Bis 15 Mio. EUR pro Person` |
| versicherungssumme_max | BigDecimal(15,2) | ❌ | Maximale Versicherungssumme (EUR) | `15000000.00` |
| sortierung | Integer | ✅ | Reihenfolge | `1` |
| erstellt_am | Timestamp | ✅ | | |

---

### 2.4 Tarifmerkmal (Konfiguration)

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| produkt_id | UUID (FK) | ✅ | Referenz auf das Produkt | |
| merkmal_name | String(100) | ✅ | Technischer Name | `sf_klasse` |
| bezeichnung | String(200) | ✅ | Anzeigename | `Schadenfreiheitsklasse` |
| datentyp | Enum | ✅ | Datentyp des Werts | `AUSWAHL` |
| wertebereich | JSONB | ✅ | Erlaubte Werte | `{"typ":"range","von":"SF½","bis":"SF35","zusatz":["M","S","0"]}` |
| einfluss_praemie | Enum | ✅ | Einfluss auf die Prämie | `HOCH` |
| pflicht | Boolean | ✅ | Pflichtfeld bei Erfassung? | `true` |
| beschreibung | Text | ❌ | Erläuterung | |
| sortierung | Integer | ✅ | Reihenfolge | `1` |
| erstellt_am | Timestamp | ✅ | | |

---

### 2.5 Produktabhaengigkeit (Konfiguration)

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| produkt_id | UUID (FK) | ✅ | Das abhängige Produkt | KFZ-TK |
| abhaengig_von_id | UUID (FK) | ✅ | Produkt, von dem es abhängt | KFZ-HP |
| abhaengigkeitstyp | Enum | ✅ | Art der Abhängigkeit | `ERFORDERT` |
| beschreibung | String(500) | ❌ | Erklärung der Regel | `Teilkasko erfordert Haftpflicht` |

**Abhängigkeitstypen:**

| Typ | Bedeutung | Beispiel KFZ |
|-----|-----------|-------------|
| `ERFORDERT` | Produkt A kann nur mit Produkt B abgeschlossen werden | KFZ-TK → KFZ-HP |
| `ENTHAELT` | Produkt A beinhaltet automatisch Produkt B | KFZ-VK ⊃ KFZ-TK |
| `AUSSCHLIESST` | Produkt A und B können nicht gleichzeitig gewählt werden | – |
| `VORAUSSETZUNG` | Produkt A erfordert bestimmte Bedingung | KFZ-RS → SF ≥ 4 |

**KFZ-Produktabhängigkeiten (Referenzdaten):**

| Produkt | abhängig von | Typ | Regel |
|---------|-------------|-----|-------|
| KFZ-TK | KFZ-HP | ERFORDERT | Teilkasko nur mit Haftpflicht |
| KFZ-VK | KFZ-HP | ERFORDERT | Vollkasko nur mit Haftpflicht |
| KFZ-VK | KFZ-TK | ENTHAELT | Vollkasko beinhaltet Teilkaskoschutz |
| KFZ-SB | KFZ-HP | ERFORDERT | Schutzbrief nur als Zusatz zur HP |
| KFZ-FK | KFZ-HP | ERFORDERT | FahrerKasko nur als Zusatz zur HP |
| KFZ-RS | KFZ-HP | ERFORDERT | RabattSchutz nur mit HP oder VK |
| KFZ-AP | KFZ-HP | ERFORDERT | AuslandPlus nur als Zusatz zur HP |
| KFZ-KZ-ZF | KFZ-HP | ERFORDERT | Zusatzfahrer-Schutz setzt bestehende KFZ-Versicherung voraus |

---

### 2.6 Aussteuerungsregel (Konfiguration)

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| bezeichnung | String(200) | ✅ | Regelname | `Hohe Deckungssumme` |
| sparte | Enum | ❌ | Sparte (NULL = spartenübergreifend) | `KFZ` |
| bedingung_ausdruck | Text | ✅ | Regelausdruck (Drools DRL oder SpEL) | `antrag.beitrag > 50000` |
| schwere | Enum | ✅ | Immer ausgesteuert / Prüfung empfohlen | `IMMER` |
| zustaendiges_team | String(100) | ❌ | Standard-Zuordnung | `ID-KFZ-Sonder` |
| aktiv | Boolean | ✅ | Regel aktiv? | `true` |
| erstellt_am | Timestamp | ✅ | | |

---

## 3. Produkt-Zuordnungen (Geschäftsdaten)

> Welche Produkte zu einem Angebot, Antrag oder Vertragsstand gewählt wurden.

### 3.1 AngebotProdukt

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| angebot_id | UUID (FK) | ✅ | Referenz auf das Angebot | |
| produkt_id | UUID (FK) | ✅ | Referenz auf das konfigurierte Produkt | |
| beitrag | BigDecimal(12,2) | ❌ | Berechneter Beitrag für dieses Produkt | `312.00` |
| selbstbeteiligung | BigDecimal(10,2) | ❌ | Gewählte Selbstbeteiligung (EUR) | `150.00` |
| tarifmerkmale | JSONB | ❌ | Gewählte Tarifmerkmal-Werte | `{"sf_klasse":"SF5","fahrerkreis":"ALLE"}` |
| deckungsbausteine | JSONB | ❌ | Gewählte optionale Bausteine | `["erweiterter_wildschaden","tierbiss_folge"]` |
| berechnungsdetails | JSONB | ❌ | Zwischenergebnisse der Prämienberechnung (→ DM-06) | `{"grundbeitrag":380.00, ...}` |

> **Hinweis (DM-06):** `berechnungsdetails` speichert die vollständige Berechnungsherleitung, um bei Rückfragen oder Missverständnissen die Prämienberechnung nachvollziehen zu können. Beispielstruktur → Abschnitt 7.4.

### 3.2 AntragProdukt

> Identische Struktur wie AngebotProdukt, aber `antrag_id` statt `angebot_id`.

| Attribut | Typ | Pflicht | Beschreibung |
|----------|-----|---------|-------------|
| id | UUID | ✅ | Technischer Primärschlüssel |
| antrag_id | UUID (FK) | ✅ | Referenz auf den Antrag |
| produkt_id | UUID (FK) | ✅ | Referenz auf das konfigurierte Produkt |
| beitrag | BigDecimal(12,2) | ❌ | Berechneter Beitrag |
| selbstbeteiligung | BigDecimal(10,2) | ❌ | Gewählte Selbstbeteiligung |
| tarifmerkmale | JSONB | ❌ | Gewählte Tarifmerkmal-Werte |
| deckungsbausteine | JSONB | ❌ | Gewählte optionale Bausteine |
| berechnungsdetails | JSONB | ❌ | Zwischenergebnisse der Prämienberechnung (→ DM-06) |

### 3.3 VertragsstandProdukt

> Identische Struktur, aber `vertragsstand_id` statt `angebot_id`.

| Attribut | Typ | Pflicht | Beschreibung |
|----------|-----|---------|-------------|
| id | UUID | ✅ | Technischer Primärschlüssel |
| vertragsstand_id | UUID (FK) | ✅ | Referenz auf den Vertragsstand |
| produkt_id | UUID (FK) | ✅ | Referenz auf das konfigurierte Produkt |
| beitrag | BigDecimal(12,2) | ✅ | Policierter Beitrag |
| selbstbeteiligung | BigDecimal(10,2) | ❌ | Policierte Selbstbeteiligung |
| tarifmerkmale | JSONB | ❌ | Policierte Tarifmerkmal-Werte |
| deckungsbausteine | JSONB | ❌ | Policierte Deckungsbausteine |

---

## 4. KFZ-Spartendatenmodell

> Spartenspezifische Entitäten, die nur in der KFZ-Sparte existieren. Liegen als eigene Tabellen vor (nicht nur JSONB), da sie strukturiert abfragbar und validierbar sein müssen.

### KFZ-ER-Diagramm

```
                          ┌──────────────────┐
                          │  Angebot/Antrag/ │
                          │    Vertrag       │
                          └────────┬─────────┘
                                   │ 1
                                   │
                                   │ 0..1
                          ┌────────▼─────────┐
                          │    Fahrzeug      │
                          │ (KFZ-spezifisch) │
                          └────────┬─────────┘
                                   │ 1
                                   │
                                   │ 1
                          ┌────────▼─────────┐
                          │ KfzTarifierung   │
                          │ (Tarifmerkmale)  │
                          └──────────────────┘

┌──────────────────┐                          ┌──────────────────┐
│    EvbNummer     │                          │SfKlassenHistorie │
│ (Antrag/Vertrag) │                          │   (Vertrag)      │
└──────────────────┘                          └──────────────────┘
```

### 4.1 Fahrzeug

> Zentrale KFZ-Entität – an Angebot, Antrag oder Vertrag gebunden.

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| referenz_typ | Enum | ✅ | Zuordnung zu Angebot / Antrag / Vertrag | `VERTRAG` |
| referenz_id | UUID | ✅ | ID des zugehörigen Angebots / Antrags / Vertrags | |
| fahrzeugart | Enum | ✅ | Fahrzeugart (→ UC-KFZ-00) | `PKW` |
| fin | String(17) | Bedingt | Fahrgestellnummer (17-stellig) | `WVWZZZ3CZWE123456` |
| hsn | String(4) | ✅ | Herstellerschlüsselnummer | `0603` |
| tsn | String(3) | ✅ | Typschlüsselnummer | `BPM` |
| amtliches_kennzeichen | String(15) | Bedingt | Amtliches Kennzeichen | `MS-LV 1234` |
| erstzulassung | Date | ✅ | Datum der Erstzulassung | `2024-06-15` |
| hersteller | String(100) | ❌ | Herstellername (aus GDV-Daten) | `Volkswagen` |
| modell | String(100) | ❌ | Modellbezeichnung (aus GDV-Daten) | `Golf VIII 1.5 TSI` |
| leistung_kw | Integer | ❌ | Motorleistung in kW | `110` |
| hubraum_ccm | Integer | ❌ | Hubraum in ccm | `1498` |
| kraftstoffart | Enum | ❌ | Kraftstoffart | `BENZIN` |
| anzahl_sitzplaetze | Integer | ❌ | Sitzplätze | `5` |
| farbe | String(50) | ❌ | Fahrzeugfarbe | `Schwarz` |
| listenpreis | BigDecimal(12,2) | ❌ | Listenpreis Neufahrzeug (EUR) | `35990.00` |
| wertgutachten_vorhanden | Boolean | ❌ | Wertgutachten vorhanden? (Pflicht bei Oldtimer) | `false` |
| wertgutachten_wert | BigDecimal(12,2) | ❌ | Wert laut Gutachten (EUR) | |
| leasing | Boolean | ❌ | Leasingfahrzeug? | `false` |
| erstellt_am | Timestamp | ✅ | | |
| geaendert_am | Timestamp | ✅ | | |

**Constraints:**
- `fin` muss 17-stellig sein und Prüfziffer bestehen (PL-KFZ-A08)
- `amtliches_kennzeichen` muss deutschem Format entsprechen (PL-KFZ-A09)
- `erstzulassung <= CURRENT_DATE` (PL-KFZ-A03)
- `wertgutachten_vorhanden = true` Pflicht wenn `fahrzeugart = OLDTIMER` (GR-KFZ-FA04)

**Historisierung:** ✅ Hibernate Envers

---

### 4.2 KfzTarifierung

> Tarifmerkmale eines KFZ-Vertrags / -Angebots / -Antrags, die zur Prämienberechnung herangezogen werden.

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| fahrzeug_id | UUID (FK) | ✅ | Referenz auf das Fahrzeug | |
| sf_klasse_hp | String(5) | Bedingt | SF-Klasse Haftpflicht | `SF5` |
| sf_klasse_vk | String(5) | ❌ | SF-Klasse Vollkasko | `SF5` |
| typklasse_hp | Integer | ✅ | GDV-Typklasse Haftpflicht (10-25) | `15` |
| typklasse_tk | Integer | ❌ | GDV-Typklasse Teilkasko (10-33) | `20` |
| typklasse_vk | Integer | ❌ | GDV-Typklasse Vollkasko (10-34) | `17` |
| regionalklasse_hp | Integer | ✅ | GDV-Regionalklasse HP (1-12) | `4` |
| regionalklasse_tk | Integer | ❌ | GDV-Regionalklasse TK (1-16) | `3` |
| regionalklasse_vk | Integer | ❌ | GDV-Regionalklasse VK (1-9) | `2` |
| fahrerkreis | Enum | ✅ | Berechtigter Fahrerkreis | `ALLE` |
| alter_juengster_fahrer | Integer | ✅ | Alter des jüngsten Fahrers (17-99) | `25` |
| jaehrliche_fahrleistung_km | Integer | ✅ | Geschätzte Jahreskilometer | `15000` |
| stellplatz | Enum | ✅ | Üblicher Abstellort | `GARAGE` |
| nutzungsart | Enum | ✅ | Nutzungsart | `PRIVAT` |
| sb_tk | BigDecimal(10,2) | ❌ | Selbstbeteiligung Teilkasko (EUR) | `150.00` |
| sb_vk | BigDecimal(10,2) | ❌ | Selbstbeteiligung Vollkasko (EUR) | `300.00` |
| rabattschutz | Boolean | ❌ | LVM-RabattSchutz gebucht? | `false` |
| erstellt_am | Timestamp | ✅ | | |
| geaendert_am | Timestamp | ✅ | | |

**Constraints:**
- `sf_klasse_hp` Pflicht wenn Produkt KFZ-HP gewählt
- `sf_klasse_vk` Pflicht wenn Produkt KFZ-VK gewählt
- `alter_juengster_fahrer >= 17` (PL-KFZ-A07)
- `jaehrliche_fahrleistung_km` zwischen 5.000 und 100.000 (PL-KFZ-A05, Warnung)
- `rabattschutz = true` nur erlaubt wenn SF-Klasse ≥ 4 (GR-KFZ-P04, PL-KFZ-L01)

**Historisierung:** ✅ Hibernate Envers

---

### 4.3 EvbNummer

> Elektronische Versicherungsbestätigung – KFZ-spezifischer Prozess.

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| evb_nummer | String(7) | ✅ | 7-stelliger alphanumerischer Code | `A1B2C3D` |
| antrag_id | UUID (FK) | ❌ | Referenz auf den Antrag | |
| vertrag_id | UUID (FK) | ❌ | Referenz auf den Vertrag | |
| partner_id | UUID (FK) | ✅ | Versicherungsnehmer (immer gesetzt) | |
| fahrzeug_id | UUID (FK) | ❌ | Fahrzeug (falls bei Erzeugung bekannt) | |
| deckungsumfang | Enum | ✅ | Gewählter Deckungsumfang | `HP_TK` |
| fahrzeugart | Enum | ✅ | Fahrzeugart (aus UC-KFZ-00) | `PKW` |
| status | Enum | ✅ | Status der eVB | `ERZEUGT` |
| erzeugt_am | Timestamp | ✅ | Erzeugungszeitpunkt | |
| gueltig_bis | Date | ✅ | Ablaufdatum (6 Monate nach Erzeugung) | `2026-09-19` |
| verwendet_am | Timestamp | ❌ | Zeitpunkt der Verwendung bei Zulassungsstelle | |
| storniert_am | Timestamp | ❌ | Stornierungszeitpunkt | |
| antragsanmahnung_id | UUID (FK) | ❌ | Verknüpfte Antragsanmahnung (bei eVB ohne Antrag + Zulassung) | |

**Constraints:**
- `evb_nummer` UNIQUE
- `partner_id` ist immer gesetzt (auch bei eVB ohne Antrag)
- `antrag_id` / `vertrag_id` können beide NULL sein (eVB ohne Antrag) oder genau eines gesetzt
- Automatische Stornierung nach 6 Monaten ohne Verwendung

---

### 4.4 SfKlassenHistorie

> Verlauf der Schadenfreiheitsklasse eines KFZ-Vertrags.

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| vertrag_id | UUID (FK) | ✅ | Referenz auf den KFZ-Vertrag | |
| sf_klasse_hp | String(5) | ✅ | SF-Klasse Haftpflicht | `SF5` |
| sf_klasse_vk | String(5) | ❌ | SF-Klasse Vollkasko | `SF5` |
| gueltig_ab | Date | ✅ | Beginn dieser SF-Klasse | `2027-01-01` |
| gueltig_bis | Date | ❌ | Ende dieser SF-Klasse (NULL = aktuell) | |
| aenderungsgrund | Enum | ✅ | Grund der Änderung | `JAEHRLICHE_HOCHSTUFUNG` |
| schaden_id | UUID (FK) | ❌ | Referenz auf den auslösenden Schaden (bei Rückstufung) | |
| rabattschutz_eingesetzt | Boolean | ❌ | Wurde RabattSchutz eingesetzt? | `false` |
| erstellt_am | Timestamp | ✅ | | |

---

### 4.5 VwbNachricht (NEU)

> Nachrichten des Versicherer-Wechsel-Branchenverfahrens (VWB) – SF-Anfragen und SF-Auskünfte zwischen Versicherern.

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|----------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| vwb_nummer | String(20) | ✅ | Fachliche VWB-Vorgangsnummer | `VWB-2026-000001` |
| vertrag_id | UUID (FK) | ❌ | Referenz auf den KFZ-Vertrag (bei Abgang) | |
| antrag_id | UUID (FK) | ❌ | Referenz auf den KFZ-Antrag (bei Zugang) | |
| richtung | Enum | ✅ | ZUGANG (wir fragen) oder ABGANG (wir antworten) | `ZUGANG` |
| status | Enum | ✅ | VWB-Status (→ VwbStatus) | `ANFRAGE_GESENDET` |
| partner_versicherer_id | String(50) | ✅ | Kennung des Vor-/Nachversicherers | `AXA` |
| partner_vertrag_nummer | String(50) | ❌ | Vertragsnummer beim Vor-/Nachversicherer | `AXA-KFZ-2024-12345` |
| angefragte_sf_klasse_hp | String(5) | ❌ | Angefragte SF-Klasse HP | `SF5` |
| angefragte_sf_klasse_vk | String(5) | ❌ | Angefragte SF-Klasse VK | `SF3` |
| bestaetigte_sf_klasse_hp | String(5) | ❌ | Bestätigte SF-Klasse HP (aus Auskunft) | `SF5` |
| bestaetigte_sf_klasse_vk | String(5) | ❌ | Bestätigte SF-Klasse VK (aus Auskunft) | `SF3` |
| schaeden_letzte_5_jahre | Integer | ❌ | Anzahl Schäden in den letzten 5 Jahren | `0` |
| stichtag | Date | ✅ | Stichtag der SF-Abfrage | `2026-12-31` |
| frist_bis | Date | ✅ | Antwortfrist (4 Wochen nach Anfrage) | `2027-01-28` |
| gesendet_am | Timestamp | ❌ | Zeitpunkt des Versands | |
| empfangen_am | Timestamp | ❌ | Zeitpunkt des Empfangs | |
| erstellt_am | Timestamp | ✅ | | |
| geaendert_am | Timestamp | ✅ | | |

**Constraints:**
- `vwb_nummer` UNIQUE
- Genau eine der Referenzen (`vertrag_id` bei Abgang, `antrag_id` bei Zugang) muss gesetzt sein
- `frist_bis = gesendet_am + 4 Wochen` (automatisch berechnet)

**Historisierung:** ✅ Hibernate Envers

---

### 4.6 VkzKennzeichen (NEU)

> Physisches Versicherungskennzeichen – Bestandsverwaltung (Kontingent, Zuweisung, Rücknahme).

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|----------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| kennzeichen_nummer | String(20) | ✅ | Aufgedruckte Kennzeichen-Nummer | `051-ABC` |
| farbe | Enum | ✅ | Kennzeichenfarbe der Saison | `SCHWARZ` |
| saison_jahr | Integer | ✅ | Versicherungsjahr (01.03.YYYY – 28.02.YYYY+1) | `2026` |
| gueltig_ab | Date | ✅ | Beginn der Gültigkeit | `2026-03-01` |
| gueltig_bis | Date | ✅ | Ende der Gültigkeit | `2027-02-28` |
| status | Enum | ✅ | Kennzeichen-Status (→ VkzKennzeichenStatus) | `AUF_LAGER` |
| vertrag_id | UUID (FK) | ❌ | Zugewiesener Vertrag (wenn ZUGEWIESEN) | |
| partner_id | UUID (FK) | ❌ | Versicherungsnehmer (wenn ZUGEWIESEN) | |
| kontingent_id | UUID (FK) | ✅ | Referenz auf Bestellkontingent | |
| zugewiesen_am | Timestamp | ❌ | Zeitpunkt der Zuweisung | |
| zurueckgenommen_am | Timestamp | ❌ | Zeitpunkt der Rücknahme | |
| vernichtet_am | Timestamp | ❌ | Zeitpunkt der Vernichtung | |
| erstellt_am | Timestamp | ✅ | | |
| geaendert_am | Timestamp | ✅ | | |

**Constraints:**
- `kennzeichen_nummer` + `saison_jahr` UNIQUE
- `vertrag_id` Pflicht wenn Status = `ZUGEWIESEN`
- `farbe` folgt 3-Jahres-Rhythmus: 2026=SCHWARZ, 2027=BLAU, 2028=GRUEN, 2029=SCHWARZ, …

**Historisierung:** ✅ Hibernate Envers

---

### 4.7 VkzKontingent (NEU)

> Bestellkontingent für Versicherungskennzeichen – Gruppierung einer Bestellung.

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|----------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| kontingent_nummer | String(20) | ✅ | Fachliche Bestellnummer | `VKZ-B-2026-001` |
| saison_jahr | Integer | ✅ | Für welche Saison bestellt | `2026` |
| farbe | Enum | ✅ | Kennzeichenfarbe | `SCHWARZ` |
| anzahl_bestellt | Integer | ✅ | Bestellte Menge | `5000` |
| anzahl_geliefert | Integer | ❌ | Gelieferte Menge | `5000` |
| bestellt_am | Date | ✅ | Bestelldatum | `2025-12-01` |
| geliefert_am | Date | ❌ | Lieferdatum | `2026-01-15` |
| erstellt_am | Timestamp | ✅ | | |
| geaendert_am | Timestamp | ✅ | | |

**Constraints:**
- `kontingent_nummer` UNIQUE

---

### 4.8 Antragsanmahnung (NEU)

> Aufforderung zur Nachholung eines Antrags, wenn eine eVB ohne Antrag ausgestellt und anschließend von der Zulassungsstelle verwendet wurde.

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|----------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| anmahnung_nummer | String(20) | ✅ | Fachliche Vorgangsnummer | `ANM-2026-000001` |
| evb_id | UUID (FK) | ✅ | Referenz auf die auslösende eVB-Nummer | |
| partner_id | UUID (FK) | ✅ | Versicherungsnehmer | |
| angebot_id | UUID (FK) | ❌ | Erzeugtes/zugeordnetes Angebot (wenn erledigt) | |
| antrag_id | UUID (FK) | ❌ | Erzeugter/zugeordneter Antrag (wenn erledigt) | |
| status | Enum | ✅ | Anmahnungsstatus (→ AntragsanmahnungStatus) | `OFFEN` |
| fahrzeugart | Enum | ✅ | Fahrzeugart aus eVB | `PKW` |
| deckungsumfang | Enum | ✅ | Deckungsumfang aus eVB | `HP_TK` |
| zulassung_kennzeichen | String(15) | ❌ | Amtliches Kennzeichen (aus GDV-Rückmeldung) | `MS-AB 1234` |
| zulassung_fin | String(17) | ❌ | FIN (aus GDV-Rückmeldung) | `WVWZZZ3CZWE123456` |
| zulassung_halter_name | String(200) | ❌ | Halter-Name (aus GDV-Rückmeldung) | `Mustermann, Max` |
| zulassung_erstzulassung | Date | ❌ | Datum der Erstzulassung | `2026-04-01` |
| zulassung_hsn | String(4) | ❌ | HSN (aus GDV-Rückmeldung, falls vorhanden) | `0603` |
| zulassung_tsn | String(3) | ❌ | TSN (aus GDV-Rückmeldung, falls vorhanden) | `BPM` |
| zulassung_datum | Date | ✅ | Datum der Zulassung (= Verwendung der eVB) | `2026-04-01` |
| frist_anmahnung | Date | ✅ | Anmahnungsfrist (14 Tage nach Zulassung) | `2026-04-15` |
| frist_eskalation | Date | ✅ | Eskalationsfrist (28 Tage nach Zulassung) | `2026-04-29` |
| stornierung_grund | String(500) | ❌ | Begründung bei Stornierung | |
| erstellt_am | Timestamp | ✅ | | |
| geaendert_am | Timestamp | ✅ | | |

**Constraints:**
- `anmahnung_nummer` UNIQUE
- `evb_id` UNIQUE (eine eVB erzeugt max. eine Antragsanmahnung)
- `angebot_id` / `antrag_id` werden gesetzt, wenn Anmahnung erledigt wird
- Fristen werden beim Erstellen automatisch berechnet

**Historisierung:** ✅ Hibernate Envers

---

### 4.9 KfzBasisbeitrag (KFZ-Konfiguration)

> Jährlicher Basisbeitrag je Produkt und Fahrzeugart. Ausgangswert für das Scoring-Faktor-Modell (→ kfz/geschaeftsregeln.md). Verwaltung über UC-03 (Produktkonfiguration und Kalkulation).

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|--------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| produkt_id | UUID (FK) | ✅ | Referenz auf Produkt (KFZ-HP, KFZ-TK, KFZ-VK) | |
| fahrzeugart | Enum | ✅ | Fahrzeugart (→ 5.13) | `PKW` |
| basisbeitrag | BigDecimal(10,2) | ✅ | Jahresbasisbeitrag in EUR | `280.00` |
| gueltig_ab | Date | ✅ | Beginn der Gültigkeit | `2026-01-01` |
| gueltig_bis | Date | ❌ | Ende der Gültigkeit (NULL = unbefristet) | |
| erstellt_am | Timestamp | ✅ | | |
| geaendert_am | Timestamp | ✅ | | |

**Constraints:**
- `UNIQUE(produkt_id, fahrzeugart, gueltig_ab)` – pro Produkt und Fahrzeugart darf es zu einem Stichtag nur einen Basisbeitrag geben
- `basisbeitrag > 0`
- `gueltig_bis IS NULL OR gueltig_bis > gueltig_ab`

**Historisierung:** ✅ Hibernate Envers

---

### 4.10 KfzScoringfaktor (KFZ-Konfiguration)

> Multiplikative Scoringfaktoren für die KFZ-Prämienberechnung. Jede Zeile ordnet einem Tarifmerkmal-Wert einen Faktor zu. Verwaltung über UC-03 (Produktkonfiguration und Kalkulation).

| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|--------|
| id | UUID | ✅ | Technischer Primärschlüssel | |
| faktor_typ | Enum | ✅ | Kategorie des Scoringfaktors (→ 5.17) | `SF_KLASSE` |
| produkt_id | UUID (FK) | ❌ | Produktbezug (NULL = produktübergreifend) | |
| faktor_schluessel | String(50) | ✅ | Schlüsselwert des Tarifmerkmals | `SF5` |
| faktor_wert | BigDecimal(6,4) | ✅ | Multiplikator (1.0000 = neutral) | `0.5500` |
| gueltig_ab | Date | ✅ | Beginn der Gültigkeit | `2026-01-01` |
| gueltig_bis | Date | ❌ | Ende der Gültigkeit (NULL = unbefristet) | |
| erstellt_am | Timestamp | ✅ | | |
| geaendert_am | Timestamp | ✅ | | |

**Constraints:**
- `UNIQUE(faktor_typ, produkt_id, faktor_schluessel, gueltig_ab)` – pro Typ, Produkt und Schlüssel max. ein Faktor pro Stichtag
- `faktor_wert > 0` (kein negativer oder Null-Faktor)
- `produkt_id` ist NULL bei produktübergreifenden Faktoren (Fahrerkreis, Fahrleistung, Alter, Stellplatz, Nutzungsart)
- `produkt_id` ist gesetzt bei produktspezifischen Faktoren (SF_KLASSE, TYPKLASSE, REGIONALKLASSE, SELBSTBETEILIGUNG)

**Historisierung:** ✅ Hibernate Envers

---

## 5. Enumerationen / Wertelisten

### 5.1 Angebotsstatus

> Statusmodell aus UC-01.

| Wert | Beschreibung | Folgestatus |
|------|-------------|-------------|
| `ENTWURF` | Angebot angelegt, noch nicht berechnet | BERECHNET, GELOESCHT |
| `BERECHNET` | Beitrag wurde berechnet | GEPRUEFT, GELOESCHT |
| `GEPRUEFT` | Plausibilitätsprüfung bestanden | BEANTRAGT, GELOESCHT |
| `BEANTRAGT` | Antrag wurde erzeugt (Angebot beibehalten) | – (Endzustand) |
| `GELOESCHT` | Angebot gelöscht oder verfallen | – (Endzustand) |

### 5.2 Antragsstatus

> Statusmodell aus UC-02.

| Wert | Beschreibung | Folgestatus |
|------|-------------|-------------|
| `OFFEN` | Antrag angelegt / zurückgewiesen | BERECHNET, STORNIERT |
| `BERECHNET` | Beitrag wurde berechnet | GEPRUEFT, STORNIERT |
| `GEPRUEFT` | Plausibilitätsprüfung bestanden | FREIGEGEBEN, AUSGESTEUERT |
| `FREIGEGEBEN` | Antrag freigegeben (Dunkelverarbeitung) | – (Endzustand) |
| `AUSGESTEUERT` | Antrag an Innendienst zur Prüfung | FREIGEGEBEN, ABGELEHNT, OFFEN (zurückgewiesen) |
| `ABGELEHNT` | Antrag vom Innendienst abgelehnt | – (Endzustand) |
| `STORNIERT` | Antrag storniert | – (Endzustand) |

### 5.3 Schwebestatus

| Wert | Beschreibung | Folgestatus |
|------|-------------|-------------|
| `OFFEN` | Schwebe angelegt, Vorgang läuft | ERLEDIGT, GESCHLOSSEN |
| `ERLEDIGT` | Vertragsstand wurde erzeugt | – (Endzustand) |
| `GESCHLOSSEN` | Antrag abgelehnt / storniert, kein Vertragsstand | – (Endzustand) |

### 5.4 Vertragsstatus

| Wert | Beschreibung | Folgestatus |
|------|-------------|-------------|
| `AKTIV` | Vertrag läuft | RUHEND, GEKUENDIGT |
| `RUHEND` | Ruheversicherung (z. B. bei Fahrzeugabmeldung KFZ) | AKTIV, ABGELAUFEN |
| `GEKUENDIGT` | Kündigung eingegangen, läuft noch bis Ende | ABGELAUFEN |
| `ABGELAUFEN` | Vertrag beendet | – (Endzustand) |

### 5.5 Vorgangstyp

| Wert | Beschreibung | Ermittlung (GR-09) |
|------|-------------|-------------------|
| `NEUGESCHAEFT` | Neuer Vertrag | Kein bestehender Vertrag vorhanden |
| `AENDERUNG` | Nachtrag / Änderung | Bestehender Vertrag vorhanden |
| `STORNIERUNG` | Stornierung / Kündigung | Vertrag wird beendet |

### 5.6 Schadenstatus

| Wert | Beschreibung |
|------|-------------|
| `GEMELDET` | Schaden gemeldet, noch nicht bearbeitet |
| `IN_BEARBEITUNG` | Schaden wird reguliert |
| `REGULIERT` | Schaden reguliert, Betrag ausgezahlt |
| `ABGELEHNT` | Schadenregulierung abgelehnt |
| `ABGESCHLOSSEN` | Schadenfall abgeschlossen |

### 5.7 EvbStatus

| Wert | Beschreibung |
|------|-------------|
| `ERZEUGT` | eVB-Nummer generiert |
| `GEMELDET` | An GDV-System gemeldet |
| `VERWENDET` | Bei Zulassungsstelle verwendet |
| `STORNIERT` | Storniert (nicht verwendet oder Antrag storniert) |

### 5.8 VwbStatus (NEU)

| Wert | Beschreibung |
|------|-------------|
| `ANFRAGE_GESENDET` | SF-Anfrage an Vorversicherer gesendet |
| `ANFRAGE_EMPFANGEN` | SF-Anfrage von Nachversicherer empfangen |
| `AUSKUNFT_EMPFANGEN` | Antwort des Vorversicherers empfangen |
| `AUSKUNFT_GESENDET` | Antwort an Nachversicherer gesendet |
| `ABWEICHUNG_PRUEFEN` | SF-Klasse weicht ab → manuelle Prüfung |
| `FRIST_ABGELAUFEN` | Vorversicherer hat nicht innerhalb 4 Wochen geantwortet |
| `UEBERNOMMEN` | SF-Klasse übernommen und in Tarifierung eingepflegt |
| `KORRIGIERT` | SF-Klasse nach Prüfung korrigiert |
| `ABGESCHLOSSEN` | VWB-Vorgang abgeschlossen |

### 5.9 VkzKennzeichenStatus (NEU)

| Wert | Beschreibung |
|------|-------------|
| `BESTELLT` | Kennzeichen beim Hersteller bestellt |
| `AUF_LAGER` | Im Bestand, verfügbar zur Zuweisung |
| `ZUGEWIESEN` | Einem Vertrag zugeordnet, in Nutzung |
| `ABGELAUFEN` | Gültigkeitszeitraum abgelaufen |
| `ZURUECKGENOMMEN` | Physisch zurückgegeben |
| `VERNICHTET` | Endgültig aus dem Umlauf |
| `VERLOREN` | Nicht zurückgegeben, Wiedervorlage |

### 5.10 VkzFarbe (NEU)

| Wert | Beschreibung | Saison-Beispiele |
|------|-------------|------------------|
| `SCHWARZ` | Schwarzes Kennzeichen | 2026, 2029, 2032, … |
| `BLAU` | Blaues Kennzeichen | 2027, 2030, 2033, … |
| `GRUEN` | Grünes Kennzeichen | 2028, 2031, 2034, … |

### 5.11 AntragsanmahnungStatus (NEU)

| Wert | Beschreibung | Folgestatus |
|------|-------------|-------------|
| `OFFEN` | Antragsanmahnung erzeugt, Bearbeitung steht aus | ERLEDIGT, UEBERFAELLIG, STORNIERT |
| `UEBERFAELLIG` | 14-Tage-Frist abgelaufen, noch nicht bearbeitet | ERLEDIGT, ESKALIERT, STORNIERT |
| `ESKALIERT` | 28-Tage-Frist abgelaufen, Teamleiter muss handeln | ERLEDIGT, STORNIERT |
| `ERLEDIGT` | Antrag wurde erstellt oder zugeordnet | – (Endzustand) |
| `STORNIERT` | Manuell storniert (kein Antrag nötig) | – (Endzustand) |

### 5.12 EvbDeckungsumfang (NEU)

| Wert | Beschreibung |
|------|-------------|
| `HP` | Nur Haftpflicht |
| `HP_TK` | Haftpflicht + Teilkasko |
| `HP_VK` | Haftpflicht + Vollkasko |

### 5.13 Fahrzeugart (KFZ)

> Aus UC-KFZ-00.

| Wert | ID | Beschreibung | Verfügbare Produkte |
|------|-----|-------------|-------------------|
| `PKW` | FA-01 | Personenkraftwagen | HP, TK, VK + alle Bausteine |
| `LKW` | FA-02 | Lastkraftwagen / Nutzfahrzeuge | HP, TK, VK (eingeschränkt) |
| `OLDTIMER` | FA-03 | Fahrzeuge mit H-Kennzeichen (≥ 30 Jahre) | HP, TK, VK (Spezialtarif) |
| `VERSICHERUNGSKENNZEICHEN` | FA-04 | Mopeds, Mofas, E-Scooter | Nur HP (+ ggf. TK) |
| `MOTORRAD` | FA-05 | Motorräder, Leichtkrafträder | HP, TK, VK (eigene Typklassen) |
| `WOHNMOBIL` | FA-06 | Wohnmobile und Wohnwagen | HP, TK, VK (eigene Typklassen) |
| `ANHAENGER` | FA-07 | Anhänger ohne eigenen Antrieb | Nur HP |
| `SONDERFAHRZEUG` | FA-08 | Land-/Forstwirtschaft, Baumaschinen | Spezialtarif |

### 5.14 Sparte

| Wert | Beschreibung |
|------|-------------|
| `KFZ` | Kraftfahrzeugversicherung (initial) |
| _weitere Sparten werden bei Bedarf ergänzt_ | |

### 5.15 Zahlungsweise

| Wert | Beschreibung | Aufschlag |
|------|-------------|-----------|
| `JAEHRLICH` | Einmal pro Jahr | 0% |
| `HALBJAEHRLICH` | Zweimal pro Jahr | ~3% |
| `VIERTELJAEHRLICH` | Viermal pro Jahr | ~5% |
| `MONATLICH` | Monatlich | ~5% |

### 5.16 Weitere Enumerationen

| Enum | Werte | Verwendet in |
|------|-------|-------------|
| `Partnertyp` | NATUERLICHE_PERSON, JURISTISCHE_PERSON | Partner |
| `Anrede` | HERR, FRAU, DIVERS, FIRMA | Partner |
| `Produkttyp` | HAUPTPRODUKT, ZUSATZBAUSTEIN, KURZFRISTIG | Produkt |
| `Zielgruppe` | PRIVAT, GEWERBE, BEIDE | Produkt |
| `DeckungArt` | PFLICHT, OPTIONAL, INKLUSIVE | Deckungsbaustein |
| `PraemienEinfluss` | HOCH, MITTEL, GERING | Tarifmerkmal |
| `Datentyp` | GANZZAHL, DEZIMAL, AUSWAHL, TEXT, DATUM, BOOLEAN | Tarifmerkmal |
| `AbhaengigkeitsTyp` | ERFORDERT, ENTHAELT, AUSSCHLIESST, VORAUSSETZUNG | Produktabhaengigkeit |
| `AussteuerungSchwere` | IMMER, EMPFOHLEN | Aussteuerungsregel |
| `Fahrerkreis` | ALLE, VN_PARTNER, NUR_VN | KfzTarifierung |
| `Stellplatz` | GARAGE, CARPORT, STRASSE | KfzTarifierung |
| `Nutzungsart` | PRIVAT, GESCHAEFTLICH, GEMISCHT | KfzTarifierung |
| `ScoringfaktorTyp` | SF_KLASSE, TYPKLASSE, REGIONALKLASSE, FAHRERKREIS, FAHRLEISTUNG, ALTER_JUENGSTER_FAHRER, STELLPLATZ, NUTZUNGSART, SELBSTBETEILIGUNG | KfzScoringfaktor |
| `Kraftstoffart` | BENZIN, DIESEL, ELEKTRO, HYBRID_BENZIN, HYBRID_DIESEL, GAS, WASSERSTOFF | Fahrzeug |
| `SfAenderungsgrund` | JAEHRLICHE_HOCHSTUFUNG, RUECKSTUFUNG_SCHADEN, UEBERNAHME, KORREKTUR | SfKlassenHistorie |

### 5.17 ScoringfaktorTyp (NEU)

> Kategorisiert die Scoringfaktoren in `KfzScoringfaktor`. Bestimmt, welches Tarifmerkmal der Faktor abbildet und ob er produktspezifisch oder produktübergreifend ist.

| Wert | Beschreibung | Produktbezug | Schlüssel-Beispiele |
|------|-------------|:------------:|---------------------|
| `SF_KLASSE` | Schadenfreiheitsklasse (HP / VK separat) | pro Produkt | `SF35`, `SF5`, `SF½`, `0`, `M` |
| `TYPKLASSE` | GDV-Typklasseneinstufung (HP / TK / VK separat) | pro Produkt | `10`, `15`, `20`, `25`, `33` |
| `REGIONALKLASSE` | GDV-Regionalklasse (HP / TK / VK separat) | pro Produkt | `1`, `6`, `12`, `16` |
| `FAHRERKREIS` | Berechtigter Fahrerkreis | übergreifend | `NUR_VN`, `VN_PARTNER`, `ALLE` |
| `FAHRLEISTUNG` | Jährliche Fahrleistung in km-Stufen | übergreifend | `BIS_6000`, `6001_9000`, `9001_12000`, … |
| `ALTER_JUENGSTER_FAHRER` | Altersgruppe des jüngsten Fahrers | übergreifend | `17_20`, `21_22`, `23_24`, `25_29`, `30_49`, … |
| `STELLPLATZ` | Üblicher Abstellort | übergreifend | `GARAGE`, `CARPORT`, `STRASSE` |
| `NUTZUNGSART` | Art der Fahrzeugnutzung | übergreifend | `PRIVAT`, `GEMISCHT`, `GESCHAEFTLICH` |
| `SELBSTBETEILIGUNG` | Selbstbeteiligung in EUR (TK / VK separat) | pro Produkt | `0`, `150`, `300`, `500`, `1000` |

---

## 6. Beziehungen (Zusammenfassung)

| Von | Zu | Kardinalität | FK in | Beschreibung |
|-----|-----|-------------|-------|-------------|
| Partner | Angebot | 1:n | Angebot.partner_id | Ein Partner kann mehrere Angebote haben |
| Partner | Antrag | 1:n | Antrag.partner_id | Ein Partner kann mehrere Anträge haben |
| Partner | Vertrag | 1:n | Vertrag.partner_id | Ein Partner kann mehrere Verträge haben |
| Partner | Schaden | 1:n | (über Vertrag) | Indirekt über Vertrag |
| Angebot | Antrag | 1:0..1 | Antrag.angebot_id | Ein Angebot kann in max. einen Antrag überführt werden |
| Angebot | AngebotProdukt | 1:n | AngebotProdukt.angebot_id | Gewählte Produkte im Angebot |
| Antrag | Schwebe | 1:1 | Schwebe.antrag_id | Bei Freigabe genau eine Schwebe (GR-06) |
| Antrag | AntragProdukt | 1:n | AntragProdukt.antrag_id | Gewählte Produkte im Antrag |
| Antrag | Vorgang | 1:1 | Vorgang.antrag_id | Ein Antrag erzeugt einen Vorgang |
| Vertrag | Vorgang | 1:n | Vorgang.vertrag_id | Ein Vertrag hat mehrere Vorgänge (Neugeschäft, Änderungen) |
| Vertrag | Vertragsstand | 1:n | Vertragsstand.vertrag_id | Versionshistorie des Vertrags |
| Vertrag | Schaden | 1:n | Schaden.vertrag_id | Schadenfälle zum Vertrag |
| Vorgang | Vertragsstand | 1:n | Vertragsstand.vorgang_id | Ein Vorgang erzeugt mindestens einen Vertragsstand |
| Schwebe | Aussteuerungsregel | n:0..1 | Schwebe.aussteuerungsregel_id | Optionale Referenz auf die ausgelöste Regel |
| Sparte | Produkt | 1:n | Produkt.sparte_id | Produkte pro Sparte |
| Produkt | Deckungsbaustein | 1:n | Deckungsbaustein.produkt_id | Bausteine pro Produkt |
| Produkt | Tarifmerkmal | 1:n | Tarifmerkmal.produkt_id | Merkmale pro Produkt |
| Produkt | Produktabhaengigkeit | 1:n | Produktabhaengigkeit.produkt_id | Abhängigkeiten |
| **KFZ:** Fahrzeug | KfzTarifierung | 1:1 | KfzTarifierung.fahrzeug_id | Tarifmerkmale pro Fahrzeug |
| **KFZ:** Produkt | KfzBasisbeitrag | 1:n | KfzBasisbeitrag.produkt_id | Basisbeiträge pro Produkt und Fahrzeugart |
| **KFZ:** Produkt | KfzScoringfaktor | 1:n | KfzScoringfaktor.produkt_id | Produktspezifische Scoringfaktoren (NULL = übergreifend) |
| **KFZ:** Vertrag | SfKlassenHistorie | 1:n | SfKlassenHistorie.vertrag_id | SF-Verlauf |
| **KFZ:** Antrag/Vertrag | EvbNummer | 1:0..n | EvbNummer.antrag_id / vertrag_id | eVB-Nummern |
| **KFZ:** Partner | EvbNummer | 1:n | EvbNummer.partner_id | eVBs eines Partners (auch ohne Antrag) |
| **KFZ:** EvbNummer | Antragsanmahnung | 1:0..1 | Antragsanmahnung.evb_id | eVB löst Anmahnung aus |
| **KFZ:** Partner | Antragsanmahnung | 1:n | Antragsanmahnung.partner_id | Anmahnungen eines Partners |
| **KFZ:** Antragsanmahnung | Angebot | 1:0..1 | Antragsanmahnung.angebot_id | Erzeugtes Angebot |
| **KFZ:** Antragsanmahnung | Antrag | 1:0..1 | Antragsanmahnung.antrag_id | Erzeugter/zugeordneter Antrag |
| **KFZ:** SfKlassenHistorie | Schaden | n:0..1 | SfKlassenHistorie.schaden_id | Auslösender Schaden |
| **KFZ:** Antrag | VwbNachricht | 1:0..1 | VwbNachricht.antrag_id | VWB-Zugang (SF-Übernahme) |
| **KFZ:** Vertrag | VwbNachricht | 1:0..n | VwbNachricht.vertrag_id | VWB-Abgang (SF-Auskunft) |
| **KFZ:** Vertrag | VkzKennzeichen | 1:0..n | VkzKennzeichen.vertrag_id | Zugewiesene Kennzeichen (pro Saison) |
| **KFZ:** VkzKontingent | VkzKennzeichen | 1:n | VkzKennzeichen.kontingent_id | Kennzeichen eines Kontingents |

---

## 7. JSONB-Strukturen (Spartenspezifische Flexibilität)

> Neben den dedizierten KFZ-Tabellen (Fahrzeug, KfzTarifierung) werden `spartenspezifische_daten` als JSONB in Angebot, Antrag, Vertrag und Vertragsstand gespeichert. Dies ermöglicht Flexibilität für künftige Sparten ohne Schema-Änderung.

### 7.1 JSONB-Schema KFZ (Angebot/Antrag)

```json
{
  "fahrzeugart": "PKW",
  "fahrzeug_id": "uuid-referenz",
  "evb_nummer": "A1B2C3D",
  "antragsanmahnung_id": null,
  "saisonkennzeichen": false,
  "saison_von": null,
  "saison_bis": null,
  "ruheversicherung": false,
  "kurzfristig": false,
  "kurzfristig_typ": null,
  "kurzfristig_laufzeit_tage": null
}
```

### 7.2 JSONB-Schema KFZ (Vertragsstand)

```json
{
  "fahrzeugart": "PKW",
  "fahrzeug_id": "uuid-referenz",
  "sf_klasse_hp": "SF5",
  "sf_klasse_vk": "SF5",
  "typklasse_hp": 15,
  "typklasse_tk": 20,
  "typklasse_vk": 17,
  "regionalklasse_hp": 4,
  "rabattschutz_aktiv": false,
  "ruheversicherung": false
}
```

### 7.4 JSONB-Schema Berechnungsdetails (Angebot/Antrag → DM-06)

> Zwischenergebnisse der Prämienberechnung zur Nachvollziehbarkeit bei Rückfragen. Bildet das Scoring-Faktor-Modell (→ kfz/geschaeftsregeln.md) vollständig ab.

```json
{
  "berechnungsdatum": "2026-03-19T14:22:00",
  "tarifversion": "KFZ-2026-Q1",
  "produkte": [
    {
      "produkt_id": "KFZ-HP",
      "basisbeitrag": 280.00,
      "fahrzeugart": "PKW",
      "faktoren": [
        { "typ": "SF_KLASSE", "schluessel": "SF5", "faktor": 0.5500 },
        { "typ": "TYPKLASSE", "schluessel": "15", "faktor": 1.0000 },
        { "typ": "REGIONALKLASSE", "schluessel": "6", "faktor": 1.0000 },
        { "typ": "FAHRERKREIS", "schluessel": "ALLE", "faktor": 1.1000 },
        { "typ": "FAHRLEISTUNG", "schluessel": "12001_15000", "faktor": 1.0000 },
        { "typ": "ALTER_JUENGSTER_FAHRER", "schluessel": "30_49", "faktor": 1.0000 },
        { "typ": "STELLPLATZ", "schluessel": "GARAGE", "faktor": 0.9000 },
        { "typ": "NUTZUNGSART", "schluessel": "PRIVAT", "faktor": 1.0000 }
      ],
      "jahresbeitrag_netto": 152.46
    },
    {
      "produkt_id": "KFZ-TK",
      "basisbeitrag": 100.00,
      "fahrzeugart": "PKW",
      "faktoren": [
        { "typ": "TYPKLASSE", "schluessel": "20", "faktor": 1.4500 },
        { "typ": "REGIONALKLASSE", "schluessel": "6", "faktor": 1.0000 },
        { "typ": "FAHRERKREIS", "schluessel": "ALLE", "faktor": 1.1000 },
        { "typ": "FAHRLEISTUNG", "schluessel": "12001_15000", "faktor": 1.0000 },
        { "typ": "ALTER_JUENGSTER_FAHRER", "schluessel": "30_49", "faktor": 1.0000 },
        { "typ": "STELLPLATZ", "schluessel": "GARAGE", "faktor": 0.9000 },
        { "typ": "NUTZUNGSART", "schluessel": "PRIVAT", "faktor": 1.0000 },
        { "typ": "SELBSTBETEILIGUNG", "schluessel": "150", "faktor": 1.0000 }
      ],
      "jahresbeitrag_netto": 143.55
    }
  ],
  "gesamtjahresbeitrag_netto": 296.01,
  "versicherungssteuer_prozent": 19.0,
  "versicherungssteuer_betrag": 56.24,
  "gesamtjahresbeitrag_brutto": 352.25,
  "zahlungsweise": "JAEHRLICH",
  "zahlungsweise_faktor": 1.0000,
  "zahlbeitrag": 352.25
}
```

> **Hinweis:** Im `VertragsstandProdukt` werden Berechnungsdetails **nicht** persistiert – dort gilt nur der finale policierte Beitrag. Die Zwischenergebnisse bleiben über das Angebot/Antrag dauerhaft abrufbar.

---

### 7.3 Design-Prinzip: Tabelle vs. JSONB

| Kriterium | Eigene Tabelle | JSONB-Feld |
|-----------|---------------|------------|
| Abfragbar über SQL | ✅ Direkt (WHERE, JOIN) | ⚠️ Mit JSON-Operatoren (`->`, `->>`, `@>`) |
| Referenzielle Integrität | ✅ FK-Constraints | ❌ Nur Application-Level |
| Schema-Validierung | ✅ DB-Level | ⚠️ Application-Level (JSON Schema) |
| Flexibilität für neue Sparten | ❌ DDL-Änderung nötig | ✅ Ohne Schema-Migration |
| Performance bei großen Datenmengen | ✅ Index auf Spalten | ⚠️ GIN-Index auf JSONB |

**Entscheidung:** KFZ als initiale Sparte bekommt **eigene Tabellen** (Fahrzeug, KfzTarifierung, EvbNummer, SfKlassenHistorie, VwbNachricht, VkzKennzeichen, VkzKontingent, Antragsanmahnung) für optimale Abfragbarkeit. JSONB-Felder dienen als **Ergänzung** für unkritische Daten und als **Standard für künftige Sparten** bis diese eigene Tabellen rechtfertigen.

---

## 8. Historisierung

### 8.1 Strategie

| Schicht | Technologie | Zweck | Betroffene Entitäten |
|---------|-------------|-------|---------------------|
| **Application Level** | Hibernate Envers | Revisionssichere Änderungshistorie mit Benutzer + Zeitstempel | Alle Kern-Entitäten (Partner, Angebot, Antrag, Vertrag, etc.) |
| **Database Level** | PostgreSQL Temporal Tables (SQL:2011) | System-versionierte Zeitreisen-Abfragen | Vertrag, Vertragsstand |
| **Fachliche Versionierung** | Vertragsstand-Kette | Fachliche Versionshistorie eines Vertrags | Vertragsstand (version = 1, 2, 3, ...) |

### 8.2 Envers-Konfiguration (Audit-Tabellen)

Für jede mit `@Audited` annotierte Entität erzeugt Hibernate Envers eine `_AUD`-Tabelle:

```
partner       → partner_aud
angebot       → angebot_aud
antrag        → antrag_aud
vertrag       → vertrag_aud
vertragsstand → vertragsstand_aud
schwebe       → schwebe_aud
vorgang       → vorgang_aud
fahrzeug      → fahrzeug_aud      (KFZ)
kfz_tarifierung → kfz_tarifierung_aud (KFZ)
vwb_nachricht → vwb_nachricht_aud  (KFZ)
vkz_kennzeichen → vkz_kennzeichen_aud (KFZ)
antragsanmahnung → antragsanmahnung_aud (KFZ)
kfz_basisbeitrag → kfz_basisbeitrag_aud (KFZ-Konfiguration)
kfz_scoringfaktor → kfz_scoringfaktor_aud (KFZ-Konfiguration)
```

Jeder Audit-Eintrag enthält:

| Feld | Typ | Beschreibung |
|------|-----|-------------|
| `REV` | Integer | Revisionsnummer (global aufsteigend) |
| `REVTYPE` | Integer | 0 = INSERT, 1 = UPDATE, 2 = DELETE |
| `revision_timestamp` | Timestamp | Zeitpunkt der Änderung |
| `revision_user` | String | Benutzer-ID der Änderung |

### 8.3 Temporal Tables (PostgreSQL)

```sql
-- Beispiel: Vertrag als system-versionierte Temporal Table
CREATE TABLE vertrag (
    id UUID PRIMARY KEY,
    -- ... alle Spalten ...
    sys_period TSTZRANGE NOT NULL DEFAULT tstzrange(current_timestamp, NULL)
);

CREATE TABLE vertrag_history (LIKE vertrag);

CREATE TRIGGER versioning_trigger
    BEFORE INSERT OR UPDATE OR DELETE ON vertrag
    FOR EACH ROW EXECUTE PROCEDURE versioning('sys_period', 'vertrag_history', true);
```

> Ermöglicht Abfragen wie: „Wie sah der Vertrag am 01.01.2027 aus?" → `SELECT * FROM vertrag_history WHERE id = ? AND sys_period @> '2027-01-01'::timestamptz`

---

## 9. Indizes und Partitionierung

### 9.1 Wichtige Indizes

| Tabelle | Index | Spalte(n) | Typ | Begründung |
|---------|-------|-----------|-----|-----------|
| partner | idx_partner_nummer | partnernummer | UNIQUE B-Tree | Suche nach Partnernummer |
| partner | idx_partner_name | nachname, vorname | B-Tree | Namenssuche |
| angebot | idx_angebot_nummer | angebotsnummer | UNIQUE B-Tree | Suche nach Angebotsnummer |
| angebot | idx_angebot_partner | partner_id | B-Tree | Angebote eines Partners |
| angebot | idx_angebot_status | status | B-Tree | Filterung nach Status |
| antrag | idx_antrag_nummer | antragsnummer | UNIQUE B-Tree | Suche nach Antragsnummer |
| antrag | idx_antrag_partner | partner_id | B-Tree | Anträge eines Partners |
| antrag | idx_antrag_status | status | B-Tree | Filterung / Schweben-Übersicht |
| vertrag | idx_vertrag_nummer | vertragsnummer | UNIQUE B-Tree | Suche nach Vertragsnummer |
| vertrag | idx_vertrag_partner | partner_id | B-Tree | Verträge eines Partners |
| vertrag | idx_vertrag_sparte | sparte | B-Tree | Filterung nach Sparte |
| schwebe | idx_schwebe_status | status | B-Tree | Schweben-Übersicht (offene) |
| schwebe | idx_schwebe_team | zustaendiges_team | B-Tree | Zuordnung Innendienst |
| fahrzeug | idx_fahrzeug_fin | fin | UNIQUE B-Tree | FIN-Suche |
| fahrzeug | idx_fahrzeug_kennzeichen | amtliches_kennzeichen | B-Tree | Kennzeichen-Suche |
| fahrzeug | idx_fahrzeug_hsn_tsn | hsn, tsn | B-Tree | Typklassen-Lookup |
| evb_nummer | idx_evb_nummer | evb_nummer | UNIQUE B-Tree | eVB-Suche |
| vwb_nachricht | idx_vwb_nummer | vwb_nummer | UNIQUE B-Tree | VWB-Suche |
| vwb_nachricht | idx_vwb_status | status | B-Tree | Offene VWB-Vorgänge |
| vwb_nachricht | idx_vwb_frist | frist_bis | B-Tree | Fristüberwachung |
| vkz_kennzeichen | idx_vkz_nummer_saison | kennzeichen_nummer, saison_jahr | UNIQUE B-Tree | Kennzeichen-Suche |
| vkz_kennzeichen | idx_vkz_status | status | B-Tree | Verfügbare Kennzeichen |
| vkz_kennzeichen | idx_vkz_vertrag | vertrag_id | B-Tree | Kennzeichen eines Vertrags |
| vkz_kontingent | idx_vkz_kontingent_nr | kontingent_nummer | UNIQUE B-Tree | Kontingent-Suche |
| antragsanmahnung | idx_anm_nummer | anmahnung_nummer | UNIQUE B-Tree | Anmahnungssuche |
| antragsanmahnung | idx_anm_status | status | B-Tree | Offene Anmahnungen |
| antragsanmahnung | idx_anm_frist | frist_anmahnung | B-Tree | Fristüberwachung |
| antragsanmahnung | idx_anm_evb | evb_id | UNIQUE B-Tree | Anmahnung zur eVB |
| antragsanmahnung | idx_anm_partner | partner_id | B-Tree | Anmahnungen eines Partners |
| kfz_basisbeitrag | idx_bb_produkt_fza | produkt_id, fahrzeugart, gueltig_ab | UNIQUE B-Tree | Basisbeitrag-Lookup |
| kfz_scoringfaktor | idx_sf_typ_prod_key | faktor_typ, produkt_id, faktor_schluessel, gueltig_ab | UNIQUE B-Tree | Faktor-Lookup |
| kfz_scoringfaktor | idx_sf_typ_prod | faktor_typ, produkt_id | B-Tree | Alle Faktoren eines Typs/Produkts |
| angebot | idx_angebot_spezifisch | spartenspezifische_daten | GIN | JSONB-Abfragen |

### 9.2 Partitionierung

| Tabelle | Partitionierungsstrategie | Partitionsschlüssel | Begründung |
|---------|--------------------------|--------------------|-----------| 
| vertrag | RANGE | sparte + Erstellungsjahr | Große Bestände nach Sparte und Jahr aufteilen |
| vertragsstand | RANGE | Erstellungsjahr | Historische Stände wachsen kontinuierlich |
| partner_aud | RANGE | revision_timestamp (Jahr) | Audit-Tabellen wachsen schnell |
| vertrag_history | RANGE | sys_period (Jahr) | Temporal-History wächst kontinuierlich |

---

## 10. Datenfluss: Angebot → Vertrag

> Zusammenfassung des Datenflusses über den gesamten Lebenszyklus.

```
┌─────────────────────────────────────────────────────────────────────┐
│                          PARTNER                                    │
│                       (Stammdaten)                                  │
└──────────────┬────────────────────────┬─────────────────────────────┘
               │                        │
          1:n  │                   1:n  │
               ▼                        ▼
┌──────────────────────┐  erzeugt  ┌──────────────────────┐
│       ANGEBOT        │──────────►│       ANTRAG         │
│ + AngebotProdukt     │  (UC-01)  │ + AntragProdukt      │
│ + Fahrzeug (KFZ)     │           │ + Fahrzeug (KFZ)     │
│ + KfzTarifierung     │           │ + KfzTarifierung     │
└──────────────────────┘           └──────────┬───────────┘
                                              │ freigeben (UC-02)
                                              ▼
                                   ┌──────────────────────┐
                                   │       SCHWEBE        │
                                   └──────────┬───────────┘
                                              │
                                    ┌─────────┴──────────┐
                                    │                    │
                              nicht ausgest.        ausgesteuert
                                    │                    │
                                    ▼                    ▼
                          ┌──────────────┐    ┌──────────────────┐
                          │   VORGANG    │    │  Innendienst     │
                          │ (Neugesch.)  │    │  prüft           │
                          └──────┬───────┘    └────────┬─────────┘
                                 │                     │
                                 ▼               freigeben/ablehnen
                          ┌──────────────┐             │
                          │   VERTRAG    │◄────────────┘
                          └──────┬───────┘
                                 │
                                 ▼
                          ┌──────────────────────┐
                          │   VERTRAGSSTAND      │
                          │ + VertragsstandProd.  │    ──► S1 (Provision)
                          │ + Fahrzeug (KFZ)     │    ──► S2 (Inkasso)
                          │ + KfzTarifierung     │    ──► S5 (Druck)
                          └──────────────────────┘
```

---

## 11. Nummernkreise

| Entität | Format | Beispiel | Bemerkung |
|---------|--------|---------|-----------|
| Partner | `KD-{YYYY}-{NNNNNN}` | `KD-2026-004711` | Fortlaufend pro Jahr |
| Angebot | `AG-{YYYY}-{NNNNNN}` | `AG-2026-001234` | Fortlaufend pro Jahr |
| Antrag | `AN-{YYYY}-{NNNNNN}` | `AN-2026-005678` | Fortlaufend pro Jahr |
| Vertrag | `VN-{YYYY}-{NNNNNN}` | `VN-2026-100001` | Fortlaufend pro Jahr |
| Vorgang | `VG-{YYYY}-{NNNNNN}` | `VG-2026-000001` | Fortlaufend pro Jahr |
| Schwebe | `SW-{YYYY}-{NNNNNN}` | `SW-2026-000042` | Fortlaufend pro Jahr |
| Schaden | `SD-{YYYY}-{NNNNNN}` | `SD-2026-000123` | Aus Schadenverwaltung (S6) |
| eVB-Nummer | `{7-stellig alphanum.}` | `A1B2C3D` | GDV-konform, systemgeneriert |
| VWB-Vorgang | `VWB-{YYYY}-{NNNNNN}` | `VWB-2026-000001` | Fortlaufend pro Jahr |
| VKZ-Kontingent | `VKZ-B-{YYYY}-{NNN}` | `VKZ-B-2026-001` | Fortlaufend pro Jahr |
| Antragsanmahnung | `ANM-{YYYY}-{NNNNNN}` | `ANM-2026-000001` | Fortlaufend pro Jahr |

> Nummernkreise werden in einer separaten Tabelle `nummernkreis` verwaltet (Sequenz pro Entität + Jahr) und sind **threadsafe** über `SELECT ... FOR UPDATE`.

---

## Offene Fragen

| Nr. | Frage | Kontext |
|-----|-------|---------|
| ~~DM-01~~ | ✅ **Entschieden:** Kontakt- und Versandwege je Dokumententyp werden **nicht** im Kerndatenmodell abgebildet. Diese Daten werden in einer separaten Schnittstelle des Partner-Systems (S7) gespeichert und verwaltet. Die Versicherungsverwaltung konsumiert diese bei Bedarf per API-Aufruf. | Partner / S7 |
| DM-02 | Werden Zweitversicherungsnehmer oder weitere beteiligte Personen am Vertrag benötigt? | Partner / Vertrag |
| DM-03 | Soll ein Dokumentenarchiv (Referenzen auf gescannte Unterlagen, Wertgutachten, SF-Nachweise) im Modell abgebildet werden? | Allgemein |
| ~~DM-09~~ | ✅ **Entschieden:** KFZ-Prämienberechnung erfolgt über ein **multiplikatives Scoring-Faktor-Modell** (Basisbeitrag × Faktor₁ × … × Faktorₙ). Basisbeiträge und Scoringfaktoren werden als dedizierte Tabellen (`KfzBasisbeitrag`, `KfzScoringfaktor`) in der DB gespeichert und über UC-03 versioniert verwaltet. Die Formel und alle Faktor-Tabellen sind in `12_sparten/kfz/geschaeftsregeln.md` dokumentiert. | KFZ / Prämie |
| ~~DM-04~~ | ✅ **Entschieden:** Die Historisierung der Partnerdaten (inkl. Adressdaten) wird vollständig im **Partner-System (S7)** verwaltet. Die Versicherungsverwaltung hält nur den jeweils aktuellen Stand des Partners vor und zieht diesen bei Bedarf per API-Aufruf. Eine eigene Adress-Entität oder Adresshistorie im Kerndatenmodell ist **nicht** erforderlich. Die lokale Partner-Entität wird bei fachlichem Bedarf (z. B. Angebotserstellung, Policierung) synchronisiert. Hibernate Envers protokolliert weiterhin Änderungen am lokalen Datenbestand für die Revisionssicherheit innerhalb der Versicherungsverwaltung. | Partner / S7 |
| DM-05 | Gibt es ein Konzept für Vermittler/Makler-Zuordnung am Vertrag? | Vertrag (→ 03_stakeholder) |
| ~~DM-06~~ | ✅ **Entschieden:** Prämienberechnungs-Zwischenergebnisse werden in Angeboten und Anträgen **mitgespeichert**, um bei Missverständnissen oder Unklarheiten nachvollziehbar darauf zurückgreifen zu können. Die Persistierung erfolgt als JSONB-Feld `berechnungsdetails` in `AngebotProdukt` und `AntragProdukt` (→ Abschnitt 3). Inhalt: Einzelposten der Prämienberechnung (Grundbeitrag, Zu-/Abschläge, SF-Rabatt, Zahlungsweiseaufschlag, etc.). Im `VertragsstandProdukt` wird nur der finale policierte Beitrag gespeichert. | Angebot / Antrag |
| DM-07 | Welche konkreten Aussteuerungsregeln gelten für die initiale KFZ-Sparte? | Aussteuerungsregel |
| DM-08 | Wie wird mit Saisonkennzeichen (KFZ) umgegangen – eigene Vertragslaufzeitlogik? | Vertrag / KFZ |
