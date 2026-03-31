# Schnittstellen – Versicherungsverwaltung

> Vollständige Spezifikation aller internen REST-APIs und externen Systemschnittstellen.
> Technologiebasis: REST + OpenAPI 3.1 (API-First), JSON, JWT/OAuth2.
> Referenzen: → [04_kontextabgrenzung.md](04_kontextabgrenzung.md) (S1–S8), → [07_datenmodell.md](07_datenmodell.md), → [11_technische_rahmenbedingungen.md](11_technische_rahmenbedingungen.md)

---

## 1. API-Design-Grundsätze

| Grundsatz | Festlegung |
|-----------|------------|
| **Stil** | REST (Resource-oriented), OpenAPI 3.1 Spec als Single Source of Truth |
| **Entwicklungsansatz** | API-First – OpenAPI Spec wird vor der Implementierung geschrieben; Client-Code (Angular) und Server-Stubs (Spring) werden daraus generiert |
| **Datenformat** | JSON (`application/json`), UTF-8 |
| **Versionierung** | URL-basiert: `/api/v1/...`, `/api/v2/...` |
| **Namenskonvention** | Ressourcen: Plural, kebab-case (`/api/v1/angebote`, `/api/v1/angebot-produkte`) |
| **IDs** | UUID in Pfaden und Payloads; fachliche Nummern (z. B. `AG-2026-001234`) als Query-Parameter |
| **Datumsformat** | ISO 8601 (`yyyy-MM-dd` für Dates, `yyyy-MM-dd'T'HH:mm:ss'Z'` für Timestamps) |
| **Währung** | EUR, BigDecimal mit 2 Nachkommastellen als String in JSON (`"487.32"`) |
| **Pagination** | Offset-basiert: `?page=0&size=20&sort=erstellt_am,desc` |
| **Filtering** | Query-Parameter: `?status=OFFEN&sparte=KFZ&partner_id=uuid` |
| **HATEOAS** | Nein (nicht im MVP) |
| **Idempotenz** | PUT und DELETE sind idempotent; POST mit `Idempotency-Key`-Header für kritische Operationen |
| **Bulk-Operationen** | Nicht im MVP vorgesehen |
| **Content Negotiation** | `Accept: application/json` (einziger unterstützter Typ) |
| **Kompression** | gzip via `Accept-Encoding` |

### 1.1 HTTP-Methoden-Konvention

| Methode | Verwendung | Idempotent | Response-Code (Erfolg) |
|---------|-----------|------------|----------------------|
| `GET` | Ressource lesen / Liste abrufen | ✅ | 200 OK |
| `POST` | Ressource anlegen / Aktion auslösen | ❌ | 201 Created (Anlage) / 200 OK (Aktion) |
| `PUT` | Ressource vollständig aktualisieren | ✅ | 200 OK |
| `PATCH` | Ressource teilweise aktualisieren | ❌ | 200 OK |
| `DELETE` | Ressource löschen (soft-delete) | ✅ | 204 No Content |

### 1.2 Standard-Response-Hülle

Alle Listen-Endpunkte liefern ein einheitliches Pagination-Objekt:

```json
{
  "content": [ ... ],
  "page": {
    "number": 0,
    "size": 20,
    "totalElements": 142,
    "totalPages": 8
  },
  "sort": {
    "field": "erstellt_am",
    "direction": "DESC"
  }
}
```

### 1.3 Standard-Fehler-Response

Alle Fehler folgen dem RFC 7807 Problem-Details-Format:

```json
{
  "type": "https://api.versicherungsverwaltung.de/errors/validation",
  "title": "Validierungsfehler",
  "status": 422,
  "detail": "Der Antrag enthält 3 Plausibilitätsfehler.",
  "instance": "/api/v1/antraege/uuid-123/aktionen/pruefen",
  "timestamp": "2026-03-19T14:22:00Z",
  "traceId": "abc-def-123",
  "errors": [
    {
      "field": "vertragsbeginn",
      "code": "PL-001",
      "severity": "ERROR",
      "message": "Vertragsbeginn muss in der Zukunft liegen."
    }
  ]
}
```

| HTTP-Status | Verwendung |
|-------------|-----------|
| 400 Bad Request | Syntaktisch ungültiger Request (JSON-Fehler, fehlende Pflichtfelder) |
| 401 Unauthorized | Kein gültiger JWT-Token |
| 403 Forbidden | Kompetenz für diese Aktion nicht vorhanden |
| 404 Not Found | Ressource nicht gefunden |
| 409 Conflict | Statusübergang nicht erlaubt (z. B. Angebot im Status ENTWURF beantragen) |
| 422 Unprocessable Entity | Plausibilitätsfehler / fachliche Validierung fehlgeschlagen |
| 500 Internal Server Error | Unerwarteter Serverfehler |
| 502 Bad Gateway | Externe Schnittstelle nicht erreichbar (S1–S8) |
| 503 Service Unavailable | System überlastet / Wartung |

---

## 2. Interne APIs (REST-Endpunkte)

> Alle Endpunkte unter `/api/v1/`. Kompetenzprüfung erfolgt pro Endpunkt über das Kompetenz-System (S8).

### 2.1 Partner-API

> Lokaler Partner-Cache der Versicherungsverwaltung. Stammdaten werden vom Partner-System (S7) synchronisiert.

| Methode | Endpunkt | Beschreibung | Kompetenz |
|---------|---------|-------------|-----------|
| `GET` | `/api/v1/partner` | Partner suchen (Name, Nummer, PLZ) | `PARTNER_LESEN` |
| `GET` | `/api/v1/partner/{id}` | Partner-Details abrufen | `PARTNER_LESEN` |
| `POST` | `/api/v1/partner` | Neuen Partner anlegen | `PARTNER_ANLEGEN` |
| `PUT` | `/api/v1/partner/{id}` | Partnerdaten aktualisieren | `PARTNER_BEARBEITEN` |
| `GET` | `/api/v1/partner/{id}/vertraege` | Alle Verträge eines Partners | `VERTRAG_LESEN` |
| `GET` | `/api/v1/partner/{id}/angebote` | Alle Angebote eines Partners | `ANGEBOT_LESEN` |
| `GET` | `/api/v1/partner/{id}/antraege` | Alle Anträge eines Partners | `ANTRAG_LESEN` |

#### POST `/api/v1/partner` – Partner anlegen

**Request:**
```json
{
  "partnertyp": "NATUERLICHE_PERSON",
  "anrede": "HERR",
  "titel": null,
  "vorname": "Max",
  "nachname": "Mustermann",
  "geburtsdatum": "1985-03-15",
  "strasse": "Kolde-Ring",
  "hausnummer": "21",
  "plz": "48151",
  "ort": "Münster",
  "land": "DE",
  "email": "max@example.de",
  "telefon": "+49 251 702-0",
  "iban": "DE89370400440532013000",
  "bic": "COBADEFFXXX"
}
```

**Response (201 Created):**
```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "partnernummer": "KD-2026-004711",
  "partnertyp": "NATUERLICHE_PERSON",
  "anrede": "HERR",
  "vorname": "Max",
  "nachname": "Mustermann",
  "geburtsdatum": "1985-03-15",
  "strasse": "Kolde-Ring",
  "hausnummer": "21",
  "plz": "48151",
  "ort": "Münster",
  "land": "DE",
  "email": "max@example.de",
  "telefon": "+49 251 702-0",
  "iban": "DE89370400440532013000",
  "bic": "COBADEFFXXX",
  "erstellt_am": "2026-03-19T14:22:00Z",
  "geaendert_am": "2026-03-19T14:22:00Z"
}
```

---

### 2.2 Angebot-API (UC-01)

| Methode | Endpunkt | Beschreibung | Kompetenz |
|---------|---------|-------------|-----------|
| `GET` | `/api/v1/angebote` | Angebote suchen/auflisten (Filter: status, sparte, partner_id) | `ANGEBOT_LESEN` |
| `GET` | `/api/v1/angebote/{id}` | Angebot-Details inkl. Produkte, Spartendaten | `ANGEBOT_LESEN` |
| `POST` | `/api/v1/angebote` | Neues Angebot anlegen → Status ENTWURF | `ANGEBOT_ANLEGEN` |
| `PUT` | `/api/v1/angebote/{id}` | Angebot aktualisieren (Daten ergänzen/ändern) | `ANGEBOT_BEARBEITEN` |
| `DELETE` | `/api/v1/angebote/{id}` | Angebot löschen (soft-delete → Status GELOESCHT) | `ANGEBOT_BEARBEITEN` |
| `POST` | `/api/v1/angebote/{id}/aktionen/berechnen` | Beitrag berechnen → Status BERECHNET | `ANGEBOT_BEARBEITEN` |
| `POST` | `/api/v1/angebote/{id}/aktionen/pruefen` | Plausibilitätsprüfung → Status GEPRUEFT | `ANGEBOT_BEARBEITEN` |
| `POST` | `/api/v1/angebote/{id}/aktionen/beantragen` | Antrag erzeugen → Status BEANTRAGT | `ANGEBOT_BEANTRAGEN` |
| `POST` | `/api/v1/angebote/{id}/aktionen/kopieren` | Angebot kopieren → neues Angebot im Status ENTWURF | `ANGEBOT_ANLEGEN` |
| `GET` | `/api/v1/angebote/{id}/produkte` | Gewählte Produkte des Angebots | `ANGEBOT_LESEN` |
| `PUT` | `/api/v1/angebote/{id}/produkte` | Produktauswahl aktualisieren | `ANGEBOT_BEARBEITEN` |
| `GET` | `/api/v1/angebote/{id}/pruefergebnis` | Letztes Prüfergebnis (Plausibilitäten) | `ANGEBOT_LESEN` |

#### POST `/api/v1/angebote` – Angebot anlegen

**Request:**
```json
{
  "partner_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "sparte": "KFZ",
  "vertragsbeginn": "2027-01-01",
  "laufzeit_monate": 12,
  "zahlungsweise": "JAEHRLICH"
}
```

**Response (201 Created):**
```json
{
  "id": "f1e2d3c4-b5a6-7890-fedc-ba0987654321",
  "angebotsnummer": "AG-2026-001234",
  "status": "ENTWURF",
  "partner_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "sparte": "KFZ",
  "vertragsbeginn": "2027-01-01",
  "laufzeit_monate": 12,
  "zahlungsweise": "JAEHRLICH",
  "berechneter_jahresbeitrag": null,
  "produkte": [],
  "spartenspezifische_daten": null,
  "erstellt_von": "ad-mueller",
  "erstellt_am": "2026-03-19T14:22:00Z",
  "geaendert_am": "2026-03-19T14:22:00Z"
}
```

#### POST `/api/v1/angebote/{id}/aktionen/berechnen` – Beitrag berechnen

**Request:** (kein Body – berechnet auf Basis der aktuellen Angebotsdaten)

**Response (200 OK):**
```json
{
  "id": "f1e2d3c4-b5a6-7890-fedc-ba0987654321",
  "status": "BERECHNET",
  "berechneter_jahresbeitrag": "487.32",
  "produkte": [
    {
      "produkt_id": "KFZ-HP",
      "bezeichnung": "KFZ-Haftpflichtversicherung",
      "beitrag": "312.00",
      "berechnungsdetails": {
        "grundbeitrag": "380.00",
        "sf_rabatt_betrag": "-95.00",
        "typklassen_zuschlag": "27.00",
        "endbeitrag_brutto": "312.00"
      }
    },
    {
      "produkt_id": "KFZ-TK",
      "bezeichnung": "Teilkaskoversicherung",
      "beitrag": "167.11",
      "berechnungsdetails": {
        "grundbeitrag": "210.00",
        "sf_rabatt_betrag": "-52.50",
        "selbstbeteiligung_rabatt": "-15.00",
        "endbeitrag_brutto": "167.11"
      }
    }
  ]
}
```

#### POST `/api/v1/angebote/{id}/aktionen/pruefen` – Plausibilitätsprüfung

**Response (200 OK – Prüfung bestanden):**
```json
{
  "id": "f1e2d3c4-b5a6-7890-fedc-ba0987654321",
  "status": "GEPRUEFT",
  "pruefergebnis": {
    "bestanden": true,
    "fehler": [],
    "warnungen": [
      {
        "code": "PL-KFZ-A05",
        "feld": "jaehrliche_fahrleistung_km",
        "severity": "WARNING",
        "message": "Jährliche Fahrleistung von 45.000 km ist überdurchschnittlich hoch."
      }
    ],
    "hinweise": [
      {
        "code": "PL-KFZ-L01",
        "severity": "INFO",
        "message": "LVM-RabattSchutz ist bei SF-Klasse ≥ 4 verfügbar."
      }
    ]
  }
}
```

**Response (422 – Prüfung nicht bestanden):**
```json
{
  "type": "https://api.versicherungsverwaltung.de/errors/plausibilitaet",
  "title": "Plausibilitätsprüfung nicht bestanden",
  "status": 422,
  "detail": "Das Angebot enthält 2 Fehler.",
  "errors": [
    {
      "code": "PL-KFZ-A03",
      "feld": "erstzulassung",
      "severity": "ERROR",
      "message": "Erstzulassung darf nicht in der Zukunft liegen."
    },
    {
      "code": "PL-KFZ-A08",
      "feld": "fin",
      "severity": "ERROR",
      "message": "FIN-Prüfziffer ist ungültig."
    }
  ]
}
```

#### POST `/api/v1/angebote/{id}/aktionen/beantragen` – Antrag erzeugen

**Request:**
```json
{
  "angebot_behalten": true
}
```

**Response (201 Created):**
```json
{
  "angebot_id": "f1e2d3c4-b5a6-7890-fedc-ba0987654321",
  "angebot_status": "BEANTRAGT",
  "antrag": {
    "id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
    "antragsnummer": "AN-2026-005678",
    "status": "OFFEN",
    "sparte": "KFZ",
    "berechneter_jahresbeitrag": "487.32"
  }
}
```

---

### 2.3 Antrag-API (UC-02)

| Methode | Endpunkt | Beschreibung | Kompetenz |
|---------|---------|-------------|-----------|
| `GET` | `/api/v1/antraege` | Anträge suchen/auflisten (Filter: status, sparte, partner_id) | `ANTRAG_LESEN` |
| `GET` | `/api/v1/antraege/{id}` | Antrag-Details inkl. Produkte, Spartendaten | `ANTRAG_LESEN` |
| `PUT` | `/api/v1/antraege/{id}` | Antrag aktualisieren (Daten ergänzen/ändern) | `ANTRAG_BEARBEITEN` |
| `POST` | `/api/v1/antraege/{id}/aktionen/berechnen` | Beitrag berechnen → Status BERECHNET | `ANTRAG_BEARBEITEN` |
| `POST` | `/api/v1/antraege/{id}/aktionen/pruefen` | Plausibilitätsprüfung → Status GEPRUEFT | `ANTRAG_BEARBEITEN` |
| `POST` | `/api/v1/antraege/{id}/aktionen/freigeben` | Freigabe → Schwebe + ggf. Vertragsstand | `ANTRAG_FREIGEBEN` |
| `POST` | `/api/v1/antraege/{id}/aktionen/stornieren` | Antrag stornieren → Status STORNIERT | `ANTRAG_BEARBEITEN` |
| `GET` | `/api/v1/antraege/{id}/produkte` | Gewählte Produkte des Antrags | `ANTRAG_LESEN` |
| `PUT` | `/api/v1/antraege/{id}/produkte` | Produktauswahl aktualisieren | `ANTRAG_BEARBEITEN` |
| `GET` | `/api/v1/antraege/{id}/pruefergebnis` | Letztes Prüfergebnis | `ANTRAG_LESEN` |
| `GET` | `/api/v1/antraege/{id}/schwebe` | Zugehörige Schwebe abrufen | `SCHWEBE_LESEN` |

#### POST `/api/v1/antraege/{id}/aktionen/freigeben` – Antrag freigeben

**Request:** (kein Body)

**Response (200 OK – Dunkelverarbeitung / nicht ausgesteuert):**
```json
{
  "antrag_id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
  "antrag_status": "FREIGEGEBEN",
  "ausgesteuert": false,
  "schwebe": {
    "id": "c3d4e5f6-a7b8-9012-cdef-123456789012",
    "schwebenummer": "SW-2026-000042",
    "status": "ERLEDIGT"
  },
  "vorgang": {
    "id": "d4e5f6a7-b8c9-0123-defa-234567890123",
    "vorgangsnummer": "VG-2026-000001",
    "vorgangstyp": "NEUGESCHAEFT"
  },
  "vertragsstand": {
    "id": "e5f6a7b8-c9d0-1234-efab-345678901234",
    "vertrag_id": "f6a7b8c9-d0e1-2345-fabc-456789012345",
    "vertragsnummer": "VN-2026-100001",
    "version": 1,
    "jahresbeitrag": "487.32"
  }
}
```

**Response (200 OK – Ausgesteuert):**
```json
{
  "antrag_id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
  "antrag_status": "AUSGESTEUERT",
  "ausgesteuert": true,
  "aussteuerungsgrund": "Deckungssumme > 10 Mio. EUR",
  "aussteuerungsregel_id": "uuid-der-regel",
  "schwebe": {
    "id": "c3d4e5f6-a7b8-9012-cdef-123456789012",
    "schwebenummer": "SW-2026-000042",
    "status": "OFFEN",
    "zustaendiges_team": "ID-KFZ-Sonder"
  },
  "vorgang": null,
  "vertragsstand": null
}
```

---

### 2.4 Schwebe-API (UC-02 – Innendienst)

| Methode | Endpunkt | Beschreibung | Kompetenz |
|---------|---------|-------------|-----------|
| `GET` | `/api/v1/schweben` | Schweben-Übersicht (Filter: status, team, bearbeiter) | `SCHWEBE_LESEN` |
| `GET` | `/api/v1/schweben/{id}` | Schwebe-Details inkl. Antrag und Aussteuerungsgrund | `SCHWEBE_LESEN` |
| `POST` | `/api/v1/schweben/{id}/aktionen/freigeben` | Ausgesteuerten Antrag freigeben → Vertragsstand erzeugen | `SCHWEBE_ENTSCHEIDEN` |
| `POST` | `/api/v1/schweben/{id}/aktionen/ablehnen` | Antrag ablehnen | `SCHWEBE_ENTSCHEIDEN` |
| `POST` | `/api/v1/schweben/{id}/aktionen/zurueckweisen` | Antrag zurückweisen (zurück an Ersteller) | `SCHWEBE_ENTSCHEIDEN` |
| `PATCH` | `/api/v1/schweben/{id}` | Schwebe zuweisen (Bearbeiter, Wiedervorlage) | `SCHWEBE_BEARBEITEN` |

#### POST `/api/v1/schweben/{id}/aktionen/ablehnen`

**Request:**
```json
{
  "ablehnungsgrund": "Deckungssumme nicht darstellbar – Risikoprüfung negativ."
}
```

**Response (200 OK):**
```json
{
  "schwebe_id": "c3d4e5f6-a7b8-9012-cdef-123456789012",
  "schwebe_status": "GESCHLOSSEN",
  "antrag_id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
  "antrag_status": "ABGELEHNT",
  "ablehnungsgrund": "Deckungssumme nicht darstellbar – Risikoprüfung negativ."
}
```

---

### 2.5 Vertrag-API

| Methode | Endpunkt | Beschreibung | Kompetenz |
|---------|---------|-------------|-----------|
| `GET` | `/api/v1/vertraege` | Verträge suchen/auflisten (Filter: status, sparte, partner_id, vertragsnummer) | `VERTRAG_LESEN` |
| `GET` | `/api/v1/vertraege/{id}` | Vertrag-Details inkl. aktueller Vertragsstand | `VERTRAG_LESEN` |
| `GET` | `/api/v1/vertraege/{id}/vertragstaende` | Alle Vertragsstände (Versionshistorie) | `VERTRAG_LESEN` |
| `GET` | `/api/v1/vertraege/{id}/vertragstaende/{version}` | Bestimmter Vertragsstand | `VERTRAG_LESEN` |
| `GET` | `/api/v1/vertraege/{id}/vorgaenge` | Alle Vorgänge zum Vertrag | `VERTRAG_LESEN` |
| `GET` | `/api/v1/vertraege/{id}/schaeden` | Schadenreferenzen zum Vertrag | `VERTRAG_LESEN` |

#### GET `/api/v1/vertraege/{id}` – Vertrag-Details

**Response (200 OK):**
```json
{
  "id": "f6a7b8c9-d0e1-2345-fabc-456789012345",
  "vertragsnummer": "VN-2026-100001",
  "status": "AKTIV",
  "partner": {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "partnernummer": "KD-2026-004711",
    "vorname": "Max",
    "nachname": "Mustermann"
  },
  "sparte": "KFZ",
  "vertragsbeginn": "2027-01-01",
  "vertragsende": null,
  "laufzeit_monate": 12,
  "hauptfaelligkeit": "2027-01-01",
  "zahlungsweise": "JAEHRLICH",
  "aktueller_jahresbeitrag": "487.32",
  "kuendigungsfrist_tage": 30,
  "aktueller_vertragsstand": {
    "id": "e5f6a7b8-c9d0-1234-efab-345678901234",
    "version": 1,
    "gueltig_ab": "2027-01-01",
    "gueltig_bis": null,
    "jahresbeitrag": "487.32",
    "produkte": [
      {
        "produkt_id": "KFZ-HP",
        "bezeichnung": "KFZ-Haftpflichtversicherung",
        "beitrag": "312.00"
      },
      {
        "produkt_id": "KFZ-TK",
        "bezeichnung": "Teilkaskoversicherung",
        "beitrag": "167.11",
        "selbstbeteiligung": "150.00"
      }
    ]
  },
  "spartenspezifische_daten": {
    "fahrzeugart": "PKW",
    "fahrzeug_id": "uuid-fahrzeug"
  },
  "erstellt_am": "2026-03-19T14:22:00Z",
  "geaendert_am": "2026-03-19T14:22:00Z"
}
```

---

### 2.6 Produktkonfigurations-API (Lesezugriff)

> Konfigurationsdaten werden über Admin-UI (→ DM-offen) oder Datenimport gepflegt. Diese API bietet Lesezugriff für Frontend und Geschäftslogik.

| Methode | Endpunkt | Beschreibung | Kompetenz |
|---------|---------|-------------|-----------|
| `GET` | `/api/v1/sparten` | Alle aktiven Sparten | `KONFIGURATION_LESEN` |
| `GET` | `/api/v1/sparten/{kuerzel}/produkte` | Produkte einer Sparte | `KONFIGURATION_LESEN` |
| `GET` | `/api/v1/produkte/{id}` | Produkt-Details inkl. Deckungsbausteine und Tarifmerkmale | `KONFIGURATION_LESEN` |
| `GET` | `/api/v1/produkte/{id}/deckungsbausteine` | Deckungsbausteine eines Produkts | `KONFIGURATION_LESEN` |
| `GET` | `/api/v1/produkte/{id}/tarifmerkmale` | Tarifmerkmale eines Produkts | `KONFIGURATION_LESEN` |
| `GET` | `/api/v1/produkte/{id}/abhaengigkeiten` | Produktabhängigkeiten | `KONFIGURATION_LESEN` |
| `GET` | `/api/v1/sparten/{kuerzel}/aussteuerungsregeln` | Aussteuerungsregeln einer Sparte | `KONFIGURATION_LESEN` |

#### GET `/api/v1/sparten/KFZ/produkte` – KFZ-Produkte

**Response (200 OK):**
```json
{
  "content": [
    {
      "id": "uuid-kfz-hp",
      "produkt_id": "KFZ-HP",
      "bezeichnung": "KFZ-Haftpflichtversicherung",
      "produkttyp": "HAUPTPRODUKT",
      "pflichtprodukt": true,
      "zielgruppe": "BEIDE",
      "mindestlaufzeit_monate": 12,
      "status": "AKTIV",
      "gueltig_ab": "2026-01-01",
      "abhaengigkeiten": []
    },
    {
      "id": "uuid-kfz-tk",
      "produkt_id": "KFZ-TK",
      "bezeichnung": "Teilkaskoversicherung",
      "produkttyp": "HAUPTPRODUKT",
      "pflichtprodukt": false,
      "zielgruppe": "BEIDE",
      "mindestlaufzeit_monate": 12,
      "status": "AKTIV",
      "gueltig_ab": "2026-01-01",
      "abhaengigkeiten": [
        { "abhaengig_von": "KFZ-HP", "typ": "ERFORDERT" }
      ]
    }
  ],
  "page": { "number": 0, "size": 20, "totalElements": 11, "totalPages": 1 }
}
```

---

### 2.7 KFZ-Sparten-API

> Spartenspezifische Endpunkte, die nur im Kontext der KFZ-Sparte existieren.

| Methode | Endpunkt | Beschreibung | Kompetenz |
|---------|---------|-------------|-----------|
| `GET` | `/api/v1/kfz/fahrzeuge/{id}` | Fahrzeugdaten abrufen | `ANGEBOT_LESEN` |
| `PUT` | `/api/v1/kfz/fahrzeuge/{id}` | Fahrzeugdaten aktualisieren | `ANGEBOT_BEARBEITEN` |
| `GET` | `/api/v1/kfz/fahrzeuge/{id}/tarifierung` | Tarifierungsdaten zum Fahrzeug | `ANGEBOT_LESEN` |
| `PUT` | `/api/v1/kfz/fahrzeuge/{id}/tarifierung` | Tarifierungsdaten aktualisieren | `ANGEBOT_BEARBEITEN` |
| `GET` | `/api/v1/kfz/typklassen` | Typklassen-Lookup (Query: hsn, tsn) | `KONFIGURATION_LESEN` |
| `GET` | `/api/v1/kfz/regionalklassen` | Regionalklassen-Lookup (Query: plz) | `KONFIGURATION_LESEN` |
| `GET` | `/api/v1/kfz/evb-nummern` | eVB-Nummern suchen (Filter: status, antrag_id, vertrag_id) | `ANTRAG_LESEN` |
| `GET` | `/api/v1/kfz/evb-nummern/{id}` | eVB-Details | `ANTRAG_LESEN` |
| `GET` | `/api/v1/kfz/vertraege/{vertrag_id}/sf-historie` | SF-Klassen-Verlauf eines Vertrags | `VERTRAG_LESEN` |
| `GET` | `/api/v1/kfz/fahrzeugarten` | Verfügbare Fahrzeugarten mit Produktmatrix | `KONFIGURATION_LESEN` |
| | | **VWB-Verfahren (Sub-Modul: sfr)** | |
| `GET` | `/api/v1/kfz/vwb-nachrichten` | VWB-Vorgänge suchen (Filter: status, richtung, partner) | `VERTRAG_LESEN` |
| `GET` | `/api/v1/kfz/vwb-nachrichten/{id}` | VWB-Vorgang-Details | `VERTRAG_LESEN` |
| `POST` | `/api/v1/kfz/vwb-nachrichten` | VWB-Anfrage manuell auslösen (Zugang) | `VERTRAG_BEARBEITEN` |
| `PUT` | `/api/v1/kfz/vwb-nachrichten/{id}/aktionen/uebernehmen` | SF-Klasse aus VWB-Auskunft übernehmen | `VERTRAG_BEARBEITEN` |
| `PUT` | `/api/v1/kfz/vwb-nachrichten/{id}/aktionen/korrigieren` | SF-Klasse manuell korrigieren | `VERTRAG_BEARBEITEN` |
| | | **Versicherungskennzeichen (Sub-Modul: vkz)** | |
| `GET` | `/api/v1/kfz/vkz-kennzeichen` | Kennzeichen suchen (Filter: status, saison_jahr, vertrag_id) | `VERTRAG_LESEN` |
| `GET` | `/api/v1/kfz/vkz-kennzeichen/{id}` | Kennzeichen-Details | `VERTRAG_LESEN` |
| `POST` | `/api/v1/kfz/vkz-kennzeichen/{id}/aktionen/zuweisen` | Kennzeichen einem Vertrag zuweisen | `VERTRAG_BEARBEITEN` |
| `POST` | `/api/v1/kfz/vkz-kennzeichen/{id}/aktionen/zuruecknehmen` | Kennzeichen zurücknehmen | `VERTRAG_BEARBEITEN` |
| `GET` | `/api/v1/kfz/vkz-kontingente` | Kontingente anzeigen (Filter: saison_jahr) | `KONFIGURATION_LESEN` |
| `POST` | `/api/v1/kfz/vkz-kontingente` | Neues Kontingent erfassen (Wareneingang) | `KONFIGURATION_BEARBEITEN` |
| `GET` | `/api/v1/kfz/vkz-kontingente/{id}/kennzeichen` | Kennzeichen eines Kontingents | `KONFIGURATION_LESEN` |

#### GET `/api/v1/kfz/typklassen?hsn=0603&tsn=BPM`

**Response (200 OK):**
```json
{
  "hsn": "0603",
  "tsn": "BPM",
  "hersteller": "Volkswagen",
  "modell": "Golf VIII 1.5 TSI",
  "leistung_kw": 110,
  "hubraum_ccm": 1498,
  "typklasse_hp": 15,
  "typklasse_tk": 20,
  "typklasse_vk": 17
}
```

#### GET `/api/v1/kfz/regionalklassen?plz=48151`

**Response (200 OK):**
```json
{
  "plz": "48151",
  "ort": "Münster",
  "zulassungsbezirk": "MS",
  "regionalklasse_hp": 4,
  "regionalklasse_tk": 3,
  "regionalklasse_vk": 2
}
```

---

### 2.8 Historisierungs-API

> Revisionssichere Änderungshistorie über Hibernate Envers.

| Methode | Endpunkt | Beschreibung | Kompetenz |
|---------|---------|-------------|-----------|
| `GET` | `/api/v1/historie/{entitaet}/{id}` | Revisionen einer Entität (z. B. `/historie/partner/uuid`) | `HISTORIE_LESEN` |
| `GET` | `/api/v1/historie/{entitaet}/{id}/revisionen/{rev}` | Bestimmte Revision einer Entität | `HISTORIE_LESEN` |
| `GET` | `/api/v1/historie/{entitaet}/{id}/diff/{rev1}/{rev2}` | Diff zwischen zwei Revisionen | `HISTORIE_LESEN` |

#### GET `/api/v1/historie/partner/{id}`

**Response (200 OK):**
```json
{
  "entitaet": "partner",
  "entitaet_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "revisionen": [
    {
      "revision": 1,
      "revisionType": "INSERT",
      "timestamp": "2026-01-15T10:30:00Z",
      "benutzer": "ad-mueller"
    },
    {
      "revision": 5,
      "revisionType": "UPDATE",
      "timestamp": "2026-03-19T14:22:00Z",
      "benutzer": "id-schmidt",
      "geaenderte_felder": ["email", "telefon"]
    }
  ]
}
```

---

### 2.9 Nummernkreis-API (intern)

> Technische API zur Vergabe fachlicher Nummern. Wird nur intern von Services genutzt (nicht vom Frontend).

| Methode | Endpunkt | Beschreibung |
|---------|---------|-------------|
| `POST` | `/api/v1/intern/nummernkreise/{entitaet}/naechste` | Nächste Nummer für Entität (z. B. `angebot`) vergeben |

**Response (200 OK):**
```json
{
  "entitaet": "angebot",
  "jahr": 2026,
  "nummer": "AG-2026-001235"
}
```

---

## 3. Externe Schnittstellen

> Anbindung an die 8 externen Systeme (S1–S8) aus der [Kontextabgrenzung](04_kontextabgrenzung.md).

### 3.1 Übersicht

| Nr. | System | Richtung | Protokoll | Format | Auslöser | Priorität MVP |
|-----|--------|----------|-----------|--------|----------|--------------|
| S1 | Provisionssystem | Ausgehend | Kafka | JSON | Vertragsstand erzeugt, Vertrag geändert/storniert | 🟡 Stub |
| S2 | Konzerninkasso | Ausgehend | Kafka | JSON | Vertragsstand erzeugt, Beitrag geändert | 🟡 Stub |
| S3 | Exkasso | Bidirektional | Kafka / REST | JSON | Rückkaufswerte, Erstattungen | 🟢 Nicht MVP |
| S4 | Datawarehouse | Ausgehend | Kafka | JSON | Nightly Batch oder Event-basiert | 🟢 Nicht MVP |
| S5 | Druckschnittstelle | Ausgehend | REST | JSON | Vertragsstand erzeugt (Police), Nachtrag, Kündigung | 🟡 Stub |
| S6 | Schadenverwaltung | Bidirektional | REST | JSON | Schaden gemeldet (eingehend), Vertragsdaten (ausgehend) | 🟡 Stub |
| S7 | Partner-System / Kundenportal | Bidirektional | REST | JSON | Partnerdaten synchronisieren, Versandwege abfragen | 🔴 MVP |
| S8 | Kompetenz-System | Eingehend (Abfrage) | REST | JSON | Jede berechtigungsrelevante Aktion | 🔴 MVP |

### 3.2 S1 – Provisionssystem (Ausgehend)

> Vertragsdaten für Provisionsberechnung liefern.

**Auslöser:** Spring Event `VertragsstandErzeugtEvent`, `VertragGeaendertEvent`, `VertragStorniertEvent`

**Kafka-Topic:** `vertrag.provision.ereignisse`

**Mechanismus:** Der Versicherungsverwaltung-Service published eine Kafka-Message; das Provisionssystem konsumiert das Topic.

**Message-Payload:**
```json
{
  "ereignistyp": "NEUGESCHAEFT",
  "vertragsnummer": "VN-2026-100001",
  "sparte": "KFZ",
  "partner": {
    "partnernummer": "KD-2026-004711",
    "nachname": "Mustermann"
  },
  "vertragsbeginn": "2027-01-01",
  "jahresbeitrag": "487.32",
  "produkte": [
    { "produkt_id": "KFZ-HP", "beitrag": "312.00" },
    { "produkt_id": "KFZ-TK", "beitrag": "167.11" }
  ],
  "vermittler_id": null,
  "ereignis_datum": "2026-03-19T14:22:00Z"
}
```

**Kafka-Key:** `vertragsnummer` (Partitionierung nach Vertrag für geordnete Verarbeitung)

**Fehlerbehandlung:** Kafka-Consumer-Retry (3 Versuche, exponentieller Backoff). Bei dauerhaftem Fehler: Dead-Letter-Topic `vertrag.provision.ereignisse.dlt` + Wiedervorlage in Schwebe-Übersicht.

---

### 3.3 S2 – Konzerninkasso (Ausgehend)

> Beitragsforderungen und Zahlungspläne übergeben.

**Auslöser:** Spring Event `VertragsstandErzeugtEvent`, `BeitragGeaendertEvent`

**Kafka-Topic:** `vertrag.inkasso.beitragsforderungen`

**Mechanismus:** Der Versicherungsverwaltung-Service published eine Kafka-Message; das Konzerninkasso konsumiert das Topic.

**Message-Payload:**
```json
{
  "vertragsnummer": "VN-2026-100001",
  "partner": {
    "partnernummer": "KD-2026-004711",
    "iban": "DE89370400440532013000",
    "bic": "COBADEFFXXX"
  },
  "jahresbeitrag": "487.32",
  "zahlungsweise": "JAEHRLICH",
  "faelligkeit_ab": "2027-01-01",
  "zahlungsplan": [
    { "faellig_am": "2027-01-01", "betrag": "487.32" }
  ]
}
```

**Kafka-Key:** `vertragsnummer`

**Fehlerbehandlung:** Consumer-Retry (3 Versuche). Dead-Letter-Topic: `vertrag.inkasso.beitragsforderungen.dlt`
```

---

### 3.4 S5 – Druckschnittstelle (Ausgehend)

> Druckaufträge für Policen, Nachträge, Kündigungsbestätigungen.

**Auslöser:** Event `VERTRAGSSTAND_ERZEUGT`, `VERTRAG_GEKUENDIGT`

**Endpunkt:** `POST /api/v1/druck/auftraege`

**Payload:**
```json
{
  "dokumenttyp": "POLICE",
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
  "vertragsstand_version": 1,
  "sparte": "KFZ",
  "produkte": [
    { "produkt_id": "KFZ-HP", "bezeichnung": "KFZ-Haftpflichtversicherung", "beitrag": "312.00" }
  ],
  "versandweg": "BRIEF",
  "prioritaet": "NORMAL"
}
```

**Dokumenttypen:**

| Typ | Auslöser | Beschreibung |
|-----|---------|-------------|
| `POLICE` | Vertragsstand erzeugt (Neugeschäft) | Versicherungsschein |
| `NACHTRAG` | Vertragsstand erzeugt (Änderung) | Nachtrag zum Versicherungsschein |
| `KUENDIGUNGSBESTAETIGUNG` | Vertrag gekündigt | Bestätigung der Kündigung |
| `ANGEBOT_DOKUMENT` | Angebot drucken (A4 in UC-01) | Angebotsausdruck für den Kunden |
| `EVB_BESTAETIGUNG` | eVB-Nummer erzeugt (KFZ) | eVB-Bestätigung |

---

### 3.5 S6 – Schadenverwaltung (Bidirektional)

> Vertragsdaten bereitstellen, Schadeninfos empfangen.

#### Ausgehend: Vertragsdaten für Schadenprüfung

**Endpunkt (bereitgestellt von uns):** `GET /api/v1/vertraege/{vertragsnummer}/deckungsauskunft`

**Response:**
```json
{
  "vertragsnummer": "VN-2026-100001",
  "status": "AKTIV",
  "sparte": "KFZ",
  "versicherungsnehmer": "Max Mustermann",
  "vertragsbeginn": "2027-01-01",
  "produkte": [
    {
      "produkt_id": "KFZ-HP",
      "deckungssumme_max": "100000000.00",
      "selbstbeteiligung": "0.00"
    },
    {
      "produkt_id": "KFZ-TK",
      "deckungssumme_max": "35990.00",
      "selbstbeteiligung": "150.00"
    }
  ],
  "fahrzeug": {
    "amtliches_kennzeichen": "MS-LV 1234",
    "hersteller": "Volkswagen",
    "modell": "Golf VIII 1.5 TSI",
    "fin": "WVWZZZ3CZWE123456"
  }
}
```

#### Eingehend: Schadenmeldung empfangen

**Endpunkt (bereitgestellt von uns):** `POST /api/v1/schaeden/meldungen`

**Payload (von Schadenverwaltung gesendet):**
```json
{
  "schadennummer": "SD-2026-000123",
  "vertragsnummer": "VN-2026-100001",
  "schadendatum": "2026-05-10",
  "meldedatum": "2026-05-11",
  "schadenart": "Auffahrunfall",
  "status": "GEMELDET",
  "auswirkung_sf": true
}
```

**Statusaktualisierung:** `PUT /api/v1/schaeden/meldungen/{schadennummer}`

```json
{
  "status": "REGULIERT",
  "regulierungsbetrag": "3250.00"
}
```

---

### 3.6 S7 – Partner-System / Kundenportal (Bidirektional)

> Partnerdaten synchronisieren, Versandwege abfragen (→ DM-01, DM-04).

#### Partnerdaten synchronisieren (eingehend)

**Endpunkt (bereitgestellt von uns):** `PUT /api/v1/partner/{partnernummer}/sync`

**Payload (vom Partner-System gesendet):**
```json
{
  "partnernummer": "KD-2026-004711",
  "vorname": "Max",
  "nachname": "Mustermann",
  "strasse": "Kolde-Ring",
  "hausnummer": "21",
  "plz": "48151",
  "ort": "Münster",
  "land": "DE",
  "email": "max.mustermann@example.de",
  "telefon": "+49 251 702-0",
  "geaendert_am": "2026-03-19T14:22:00Z"
}
```

#### Versandwege abfragen (ausgehend)

**Endpunkt (aufgerufen bei Partner-System):** `GET /partner-api/v1/partner/{partnernummer}/versandwege`

**Response (vom Partner-System):**
```json
{
  "partnernummer": "KD-2026-004711",
  "versandwege": [
    { "dokumenttyp": "POLICE", "kanal": "BRIEF", "adresse": "Kolde-Ring 21, 48151 Münster" },
    { "dokumenttyp": "RECHNUNG", "kanal": "EMAIL", "adresse": "max.mustermann@example.de" },
    { "dokumenttyp": "SCHADENINFO", "kanal": "BRIEF", "adresse": "Kolde-Ring 21, 48151 Münster" }
  ]
}
```

---

### 3.7 S8 – Kompetenz-System (Eingehend / Abfrage)

> Kompetenzprüfung bei jeder berechtigungsrelevanten Aktion. Wird durch Spring Security Filter automatisch ausgelöst.

**Endpunkt (aufgerufen bei Kompetenz-System):** `GET /kompetenz-api/v1/benutzer/{benutzer_id}/kompetenzen/{kompetenz_id}`

**Response (vom Kompetenz-System):**
```json
{
  "benutzer_id": "ad-mueller",
  "kompetenz_id": "ANGEBOT_BEANTRAGEN",
  "vorhanden": true,
  "gueltig_bis": null
}
```

**Bulk-Abfrage (Performance-Optimierung):** `POST /kompetenz-api/v1/benutzer/{benutzer_id}/kompetenzen/pruefen`

```json
{
  "kompetenz_ids": ["ANGEBOT_LESEN", "ANGEBOT_BEARBEITEN", "ANGEBOT_BEANTRAGEN"]
}
```

**Response:**
```json
{
  "benutzer_id": "ad-mueller",
  "ergebnis": {
    "ANGEBOT_LESEN": true,
    "ANGEBOT_BEARBEITEN": true,
    "ANGEBOT_BEANTRAGEN": true
  }
}
```

#### Kompetenzliste (MVP)

| Kompetenz-ID | Beschreibung | Typische Rolle |
|-------------|-------------|---------------|
| `PARTNER_LESEN` | Partnerdaten einsehen | AD, ID |
| `PARTNER_ANLEGEN` | Neuen Partner anlegen | AD, ID |
| `PARTNER_BEARBEITEN` | Partnerdaten ändern | AD, ID |
| `ANGEBOT_LESEN` | Angebote einsehen | AD, ID |
| `ANGEBOT_ANLEGEN` | Neues Angebot erstellen | AD, ID |
| `ANGEBOT_BEARBEITEN` | Angebot bearbeiten/berechnen/prüfen | AD, ID |
| `ANGEBOT_BEANTRAGEN` | Angebot in Antrag überführen | AD, ID |
| `ANTRAG_LESEN` | Anträge einsehen | AD, ID |
| `ANTRAG_BEARBEITEN` | Antrag bearbeiten/berechnen/prüfen | AD, ID |
| `ANTRAG_FREIGEBEN` | Antrag freigeben (→ Schwebe) | AD, ID |
| `SCHWEBE_LESEN` | Schweben-Übersicht einsehen | ID |
| `SCHWEBE_BEARBEITEN` | Schwebe zuweisen / Wiedervorlage setzen | ID |
| `SCHWEBE_ENTSCHEIDEN` | Ausgesteuerten Antrag freigeben/ablehnen/zurückweisen | ID |
| `VERTRAG_LESEN` | Verträge und Vertragsstände einsehen | AD, ID |
| `KONFIGURATION_LESEN` | Produkte, Sparten, Regeln einsehen | AD, ID |
| `HISTORIE_LESEN` | Änderungshistorie einsehen | ID |

> **AD** = Außendienst, **ID** = Innendienst

---

### 3.8 Kafka-Topics-Übersicht

> Apache Kafka wird für die gesamte asynchrone Kommunikation mit externen Systemen eingesetzt. Spring Kafka (Spring Boot Starter) als Client-Bibliothek.

| Topic | Producer | Consumer | Key | Beschreibung |
|-------|---------|---------|-----|--------------|
| `vertrag.provision.ereignisse` | Versicherungsverwaltung | Provisionssystem (S1) | `vertragsnummer` | Vertragsereignisse für Provisionsberechnung |
| `vertrag.inkasso.beitragsforderungen` | Versicherungsverwaltung | Konzerninkasso (S2) | `vertragsnummer` | Beitragsforderungen und Zahlungspläne |
| `vertrag.exkasso.erstattungen` | Exkasso (S3) | Versicherungsverwaltung | `vertragsnummer` | Erstattungen und Rückkaufswerte |
| `vertrag.datawarehouse.bewegungen` | Versicherungsverwaltung | Datawarehouse (S4) | `vertragsnummer` | Vertragsbewegungen, Bestandsdaten |
| `vertrag.provision.ereignisse.dlt` | Kafka (auto) | Monitoring / Retry-Service | – | Dead-Letter für S1-Fehler |
| `vertrag.inkasso.beitragsforderungen.dlt` | Kafka (auto) | Monitoring / Retry-Service | – | Dead-Letter für S2-Fehler |

#### Kafka-Konfiguration (MVP)

| Aspekt | Festlegung |
|--------|------------|
| **Serialisierung** | JSON (Jackson) mit Schema-Header |
| **Partitionierung** | Nach `vertragsnummer` (geordnete Verarbeitung pro Vertrag) |
| **Retention** | 7 Tage (konfigurierbar) |
| **Replikation** | `replication-factor: 3`, `min.insync.replicas: 2` |
| **Consumer-Gruppen** | Je eine pro Zielsystem (z. B. `cg-provisionssystem`, `cg-konzerninkasso`) |
| **Idempotenz** | Producer: `enable.idempotence=true`; Consumer: Deduplizierung über `ereignis_id` |
| **Exactly-Once** | Transactional Outbox Pattern (DB-Transaktion + Outbox-Tabelle → Kafka Connector/Poller) |
| **Dead-Letter-Topics** | `.dlt`-Suffix, manuelle Wiederverarbeitung über Admin-API |
| **Monitoring** | Kafka-Metriken via Micrometer → Prometheus/Grafana |

#### Transactional Outbox Pattern

```
┌──────────────────────────────────────────────────┐
│  Versicherungsverwaltung                         │
│                                                  │
│  1. Business-Transaktion                         │
│     ┌────────────────────┐                       │
│     │ BEGIN TRANSACTION  │                       │
│     │  UPDATE vertrag    │                       │
│     │  INSERT outbox_msg │  ← gleiche Transaktion│
│     │ COMMIT             │                       │
│     └────────┬───────────┘                       │
│              │                                   │
│  2. Outbox-Poller (async)                        │
│     ┌────────▼───────────┐                       │
│     │ SELECT FROM outbox │                       │
│     │ WHERE sent = false │                       │
│     └────────┬───────────┘                       │
│              │                                   │
│  3. Kafka Publish                                │
│     ┌────────▼───────────┐    ┌───────────────┐  │
│     │ kafkaTemplate.send │───►│  Kafka Broker │  │
│     └────────┬───────────┘    └───────────────┘  │
│              │                                   │
│  4. Mark as sent                                 │
│     ┌────────▼───────────┐                       │
│     │ UPDATE outbox      │                       │
│     │ SET sent = true    │                       │
│     └────────────────────┘                       │
└──────────────────────────────────────────────────┘
```

---

### 3.9 GDV REST API (Externe Datenquelle)

> Anbindung an die neue REST-API des Gesamtverbands der Deutschen Versicherungswirtschaft (GDV) für Fahrzeug-, Typklassen-, Regionalklassen- und eVB-Daten.

**Base-URL:** `https://api.gdv.de/v1/` (konfigurierbar per Spring-Property)

**Authentifizierung:** API-Key oder OAuth2 Client-Credentials (je nach GDV-Vertrag)

#### Genutzte GDV-Endpunkte

| Endpunkt (GDV) | Methode | Beschreibung | Interner Consumer |
|----------------|---------|-------------|-------------------|
| `/typklassen?hsn={hsn}&tsn={tsn}` | `GET` | Typklassen HP/TK/VK für Fahrzeugtyp | KFZ-Sparten-API (2.7) |
| `/regionalklassen?plz={plz}` | `GET` | Regionalklassen HP/TK/VK für Zulassungsbezirk | KFZ-Sparten-API (2.7) |
| `/fahrzeuge?fin={fin}` | `GET` | Fahrzeugdaten anhand FIN | KFZ-Hook (Fahrzeugdaten) |
| `/evb/meldungen` | `POST` | eVB-Nummer beim GDV melden | KFZ-Hook (eVB-Erzeugung) |
| `/evb/meldungen/{evb_nr}/storno` | `POST` | eVB-Stornierung melden | KFZ-Prozess (eVB-Stornierung) |
| `/zulassung/abmeldungen` | `GET` | Fahrzeug-Abmeldungen abrufen (Polling) | KFZ-Service (Ruheversicherung) |

#### GDV-Response Typklassen (Beispiel)

**Request:** `GET https://api.gdv.de/v1/typklassen?hsn=0603&tsn=BPM`

**Response:**
```json
{
  "hsn": "0603",
  "tsn": "BPM",
  "hersteller": "Volkswagen",
  "modell": "Golf VIII 1.5 TSI",
  "leistung_kw": 110,
  "hubraum_ccm": 1498,
  "kraftstoffart": "BENZIN",
  "typklasse_hp": 15,
  "typklasse_tk": 20,
  "typklasse_vk": 17,
  "gueltig_ab": "2025-10-01"
}
```

#### GDV eVB-Meldung (Beispiel)

**Request:** `POST https://api.gdv.de/v1/evb/meldungen`

```json
{
  "evb_nummer": "A12B34C",
  "versicherer_id": "LVM",
  "versicherungsnehmer": {
    "name": "Mustermann",
    "vorname": "Max"
  },
  "fahrzeug": {
    "hsn": "0603",
    "tsn": "BPM",
    "fin": "WVWZZZ3CZWE123456"
  },
  "deckung": {
    "hp": true,
    "tk": true,
    "vk": false
  },
  "gueltig_ab": "2027-01-01",
  "gueltig_bis": "2027-07-01"
}
```

**Response (201 Created):**
```json
{
  "evb_nummer": "A12B34C",
  "status": "GEMELDET",
  "meldezeitpunkt": "2026-03-19T14:22:00Z",
  "referenz_id": "GDV-2026-XYZ789"
}
```

#### GDV VWB – SF-Anfrage senden (Beispiel)

**Request:** `POST https://api.gdv.de/v1/vwb/sf-anfragen`

```json
{
  "anfragender_versicherer": "LVM",
  "vorversicherer_id": "AXA",
  "vorvertrag_nummer": "AXA-KFZ-2024-12345",
  "versicherungsnehmer": {
    "name": "Mustermann",
    "vorname": "Max",
    "geburtsdatum": "1985-03-15"
  },
  "angefragte_sf_klasse_hp": "SF5",
  "angefragte_sf_klasse_vk": "SF3",
  "stichtag": "2026-12-31"
}
```

**Response (202 Accepted):**
```json
{
  "referenz_id": "VWB-GDV-2026-000001",
  "status": "ANFRAGE_WEITERGELEITET",
  "frist_bis": "2027-01-28"
}
```

#### GDV VWB – SF-Auskunft empfangen (Webhook/Polling)

**Eingehende Nachricht (Webhook):**
```json
{
  "referenz_id": "VWB-GDV-2026-000001",
  "nachrichtentyp": "SF_AUSKUNFT",
  "vorversicherer_id": "AXA",
  "bestaetigte_sf_klasse_hp": "SF5",
  "bestaetigte_sf_klasse_vk": "SF3",
  "schaeden_letzte_5_jahre": 0,
  "vertrag_beginn": "2019-01-01",
  "vertrag_ende": "2026-12-31",
  "kuendigungsgrund": "VN_KUENDIGUNG"
}
```

#### GDV-Integration – Technische Details

| Aspekt | Festlegung |
|--------|------------|
| **Client** | Spring WebClient (reaktiv) mit Resilience4j Circuit Breaker |
| **Caching** | Typklassen/Regionalklassen lokal cachen (Redis/Caffeine, TTL: 24h) – jährliche Aktualisierung per Scheduler |
| **Retry** | 3 Versuche mit exponentiellem Backoff (1s, 2s, 4s) |
| **Circuit Breaker** | Open nach 5 Fehlern in 60s; Half-Open nach 30s |
| **Fallback** | Bei GDV-Nichtverfügbarkeit: lokaler Cache (Typklassen/Regionalklassen), eVB-Meldung in Outbox für späteren Retry |
| **Timeout** | Connect: 5s, Read: 10s |
| **Logging** | Jeder GDV-Aufruf wird mit Correlation-ID und Response-Zeit geloggt |

---

## 4. Externe Schnittstellen – Importe

| Nr. | Quelle | Format | Protokoll | Häufigkeit | Beschreibung |
|-----|--------|--------|-----------|-----------|-------------|
| IMP-01 | GDV-Typklassenverzeichnis | JSON | REST (GDV-API → 3.9) | Jährlich (Oktober) + On-Demand-Lookup | Typklassen HP/TK/VK pro HSN/TSN |
| IMP-02 | GDV-Regionalklassenverzeichnis | JSON | REST (GDV-API → 3.9) | Jährlich (Oktober) + On-Demand-Lookup | Regionalklassen HP/TK/VK pro PLZ/Bezirk |
| IMP-03 | Schadenverwaltung (S6) | JSON | REST (Push) | Ereignisbasiert | Schadenmeldungen und Statusänderungen (→ 3.5) |
| IMP-04 | Partner-System (S7) | JSON | REST (Push) | Ereignisbasiert | Partnerdaten-Synchronisation (→ 3.6) |
| IMP-05 | GDV-Zulassungsstelle | JSON | REST (GDV-API → 3.9) | Polling (stündlich) | Fahrzeug-Abmeldungen (→ KFZ-Ruheversicherung) |
| IMP-06 | GDV-VWB (SF-Auskünfte) | JSON | REST Webhook (GDV-API → 3.9) | Ereignisbasiert | SF-Auskünfte von Vorversicherern (→ VWB-Verfahren) |
| IMP-07 | Alt-System (Migration) | CSV / JSON | Batch-Import | Einmalig (Go-Live) | Bestandsdaten-Migration aus Vorsystemen |

## 5. Externe Schnittstellen – Exporte

| Nr. | Ziel | Format | Protokoll | Häufigkeit | Beschreibung |
|-----|------|--------|-----------|-----------|-------------|
| EXP-01 | Provisionssystem (S1) | JSON | Kafka (→ 3.8) | Ereignisbasiert | Vertragsereignisse (→ 3.2) |
| EXP-02 | Konzerninkasso (S2) | JSON | Kafka (→ 3.8) | Ereignisbasiert | Beitragsforderungen (→ 3.3) |
| EXP-03 | Druckschnittstelle (S5) | JSON | REST | Ereignisbasiert | Druckaufträge (→ 3.4) |
| EXP-04 | Datawarehouse (S4) | JSON | Kafka (→ 3.8) | Nightly + Ereignisbasiert | Bestandsdaten, Vertragsbewegungen, Kennzahlen |
| EXP-05 | GDV-eVB-System | JSON | REST (GDV-API → 3.9) | Ereignisbasiert | eVB-Meldungen (KFZ-spezifisch) |
| EXP-06 | GDV-VWB-System | JSON | REST (GDV-API → 3.9) | Ereignisbasiert | VWB-SF-Anfragen und -Auskünfte (KFZ-spezifisch) |
| EXP-07 | Druckschnittstelle (S5) | JSON | REST | Jährlich (VKZ-Saison) | VKZ-Versand an Versicherungsnehmer |

---

## 6. Events (Interne Kommunikation)

> Spring ApplicationEvents für entkoppelte Verarbeitung innerhalb des Modularen Monolithen.

| Event | Auslöser | Consumer | Beschreibung |
|-------|---------|---------|-------------|
| `AngebotErstelltEvent` | POST /angebote | KFZ-Hook (Fahrzeugdaten-Vorbereitung) | Neues Angebot angelegt |
| `AngebotBerechnetEvent` | POST /angebote/{id}/aktionen/berechnen | Logging, Analytics | Beitrag berechnet |
| `AntragErstelltEvent` | POST /angebote/{id}/aktionen/beantragen | KFZ-Hook (eVB-Erzeugung) | Antrag aus Angebot erzeugt |
| `AntragFreigegebenEvent` | POST /antraege/{id}/aktionen/freigeben | Policierung, Hooks | Antrag freigegeben |
| `AntragAusgesteuertEvent` | POST /antraege/{id}/aktionen/freigeben | Schwebe-Service | Antrag an Innendienst |
| `VertragsstandErzeugtEvent` | Policierung | S1 (Provision), S2 (Inkasso), S5 (Druck) | Neuer Vertragsstand policiert |
| `VertragGekuendigtEvent` | Kündigung | S2 (Inkasso), S5 (Druck) | Vertrag gekündigt |
| `SchadenGemeldetEvent` | POST /schaeden/meldungen | KFZ-SF-Service | Schaden von S6 empfangen |
| `PartnerAktualisiertEvent` | PUT /partner/{nr}/sync | Cache-Invalidierung | Partnerdaten von S7 aktualisiert |
| `VwbAnfrageGesendetEvent` | VWB-Zugang: SF-Anfrage | VWB-Fristüberwachung, Logging | VWB-Anfrage an Vorversicherer gesendet |
| `VwbAuskunftEmpfangenEvent` | GDV-VWB-Webhook | SF-Übernahme, VWB-Service | SF-Auskunft vom Vorversicherer empfangen |
| `VwbAbgangBeantwortetEvent` | VWB-Abgang beantwortet | Logging, Archivierung | SF-Auskunft an Nachversicherer gesendet |
| `VwbFristAbgelaufenEvent` | Scheduler (4-Wochen-Frist) | VWB-Service, Schwebe | Vorversicherer hat nicht geantwortet |
| `VkzZugewiesenEvent` | Policierung VKZ-Vertrag | S5 (Druck/Versand), Logging | Kennzeichen einem Vertrag zugewiesen |
| `VkzZurueckgenommenEvent` | Kündigung/Ablauf | Bestandsführung, Logging | Kennzeichen zurückgenommen |
| `VkzSaisonwechselEvent` | Scheduler (01.03.) | VKZ-Zuweisungen, S5 (Druck), S2 (Inkasso) | Jährlicher Kennzeichen-Saisonwechsel |

---

## 7. Benachrichtigungen

| Nr. | Typ | Auslöser | Empfänger | Kanal | Beschreibung |
|-----|-----|---------|-----------|-------|-------------|
| NOT-01 | Schwebe-Zuweisung | Antrag ausgesteuert | Innendienst-Team | UI-Notification | Neuer ausgesteuerter Antrag in der Schweben-Übersicht |
| NOT-02 | Wiedervorlage fällig | Wiedervorlage-Datum erreicht | Sachbearbeiter | UI-Notification | Schwebe muss bearbeitet werden |
| NOT-03 | Schnittstellenfehler | S1/S2/S5 nicht erreichbar | System-Admin | UI-Notification + Log | Retry fehlgeschlagen, manueller Eingriff nötig |
| NOT-04 | eVB-Ablauf (KFZ) | 5 Monate seit eVB-Erzeugung | Sachbearbeiter | UI-Notification | eVB-Nummer droht zu verfallen |
| NOT-05 | Angebot verfällt | Gültigkeitsdatum erreicht | Ersteller | UI-Notification | Angebot wird bald automatisch gelöscht |
| NOT-06 | VWB-Frist läuft ab (KFZ) | 3 Wochen nach VWB-Anfrage | Sachbearbeiter | UI-Notification | Vorversicherer hat noch nicht geantwortet, Frist droht abzulaufen |
| NOT-07 | VWB-Abweichung (KFZ) | SF-Klasse aus Auskunft weicht ab | Sachbearbeiter | UI-Notification | Manuelle Prüfung der SF-Klasse erforderlich |
| NOT-08 | VKZ-Kontingent niedrig | Bestand < 10% der Saison-Kontingente | System-Admin | UI-Notification | Nachbestellung Versicherungskennzeichen nötig |
| NOT-09 | VKZ-Saisonwechsel (KFZ) | 4 Wochen vor 01.03. | Sachbearbeiter | UI-Notification | Saisonwechsel steht bevor, neue Kennzeichen vorbereiten |

> **Hinweis:** Externe Benachrichtigungen an Kunden (E-Mail, Brief) laufen über die Druckschnittstelle (S5) und das Partner-System (S7, Versandwege). Das Bestandsführungssystem erzeugt keine direkten E-Mails/SMS.

---

## 8. Authentifizierung & Autorisierung

### 8.1 Authentifizierung

| Aspekt | Festlegung |
|--------|------------|
| **Methode** | JWT (JSON Web Token) über OAuth2 / OpenID Connect |
| **Token-Übergabe** | `Authorization: Bearer <jwt-token>` Header |
| **Token-Aussteller** | Externes Identity-System (IdP) – nicht Teil des Bestandsführungssystems |
| **Token-Inhalt** | `sub` (Benutzer-ID), `name`, `exp`, `iat`, Custom-Claims |
| **Token-Validierung** | Spring Security – Signatur, Ablaufzeit, Aussteller |
| **Session** | Stateless (kein Server-seitiger Session-State) |

### 8.2 Autorisierung (Kompetenzprüfung)

```
┌─────────┐     Request + JWT     ┌──────────────────────┐
│  Client  │─────────────────────►│  Spring Security     │
│ (Angular)│                      │  Filter Chain        │
└─────────┘                      └──────────┬───────────┘
                                             │
                                    1. JWT validieren
                                             │
                                    2. Benutzer-ID extrahieren
                                             │
                                    3. Kompetenz-Abfrage
                                             ▼
                                   ┌──────────────────────┐
                                   │  Kompetenz-System    │
                                   │  (S8)                │
                                   │  GET /kompetenzen/   │
                                   │  {benutzer}/{komp}   │
                                   └──────────┬───────────┘
                                             │
                                    4. vorhanden: true/false
                                             │
                                    ┌────────┴────────┐
                                    │                 │
                                   true             false
                                    │                 │
                              Request              403
                              weiterleiten         Forbidden
```

### 8.3 Kompetenz-Caching

| Aspekt | Festlegung |
|--------|------------|
| **Cache-Strategie** | Kompetenzen pro Benutzer für 5 Minuten im lokalen Cache (Caffeine) |
| **Cache-Invalidierung** | Bei explizitem Logout oder nach TTL (5 Min) |
| **Fallback** | Bei S8-Nichtverfügbarkeit: geschützte Funktionen blockiert (Fail-Closed) |

### 8.4 API-Sicherheit (weitere Maßnahmen)

| Maßnahme | Beschreibung |
|----------|-------------|
| **CORS** | Nur erlaubte Origins (Frontend-Domain) |
| **Rate Limiting** | 100 Requests/Minute pro Benutzer (konfigurierbar) |
| **Input Validation** | Jakarta Bean Validation auf allen DTOs |
| **SQL Injection** | Durch JPA/Hibernate parametrisierte Queries |
| **XSS** | Content-Security-Policy Header, Input-Sanitization |
| **CSRF** | Nicht relevant (Stateless JWT, kein Cookie-basiertes Auth) |
| **HTTPS** | Erzwungen für alle Endpunkte (TLS 1.3) |
| **Audit-Logging** | Jeder schreibende API-Aufruf wird mit Benutzer-ID und Timestamp geloggt |
| **Request-Tracing** | Correlation-ID (`X-Correlation-Id`) in jedem Request/Response |

---

## 9. Offene Fragen

| Nr. | Frage | Kontext |
|-----|-------|---------|
| API-01 | Soll GraphQL als Alternative zu REST für komplexe Abfragen (z. B. Dashboard) bereitgestellt werden? | Performance / Flexibilität |
| ~~API-02~~ | ~~Message-Queue-System~~ → **Entschieden: Apache Kafka** (→ 3.8) | ✅ Geklärt |
| ~~API-03~~ | ~~GDV-Netzanbindung~~ → **Entschieden: GDV REST API** (→ 3.9) | ✅ Geklärt |
| API-04 | Soll ein API-Gateway (z. B. Spring Cloud Gateway) vor die REST-APIs geschaltet werden? | Infrastruktur |
| API-05 | Wie werden Webhook-Callbacks für asynchrone Schnittstellenergebnisse (z. B. Druckbestätigung) realisiert? | S5 / Events |
| API-06 | Soll ein Retry-Dashboard für fehlgeschlagene Schnittstellenaufrufe im MVP enthalten sein? | Betrieb | 
