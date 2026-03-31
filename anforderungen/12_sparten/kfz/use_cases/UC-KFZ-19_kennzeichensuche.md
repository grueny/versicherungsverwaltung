# UC-KFZ-19: Kennzeichensuche – Angebote, Anträge und Verträge über amtliches Kennzeichen finden

## Beschreibung
Der Benutzer sucht anhand eines **amtlichen Kennzeichens** nach Angeboten, Anträgen und Verträgen, in denen dieses Kennzeichen hinterlegt ist. Das System durchsucht dabei alle KFZ-Fahrzeugdaten und liefert eine konsolidierte Trefferliste, die zeigt, **wo** das Kennzeichen aktuell oder historisch verwendet wird.

Da ein Kennzeichen im Laufe der Zeit in verschiedenen Geschäftsobjekten vorkommen kann (z. B. altes Angebot, laufender Vertrag, abgelaufener Vertragsstand nach Fahrzeugwechsel), werden alle Fundstellen aufgelistet und nach **Aktualität** klassifiziert:

- **Aktive Treffer:** Kennzeichen ist im aktuell gültigen Vertragsstand, in einem offenen Antrag oder in einem nicht-abgeschlossenen Angebot enthalten.
- **Historische Treffer:** Kennzeichen war in einem früheren (nicht mehr gültigen) Vertragsstand enthalten (z. B. vor einem Fahrzeugwechsel) oder in einem gelöschten/stornierten/abgelehnten Geschäftsobjekt.

Dieser Use Case ist ein **KFZ-spartenspezifischer Suchprozess**, der die spartenübergreifende globale Suche um eine kennzeichenbasierte Recherche ergänzt. Er ist besonders relevant für den Innendienst (Bestandsrecherche, Schadenauskunft, Doppelversicherungsprüfung) und den Außendienst (schnelle Vertragsidentifikation beim Kunden).

## Akteure
- **Primär:** Innendienst (Sachbearbeitung), Außendienst (Vertrieb)
- **Sekundär:** –

## Vorbedingungen
- Benutzer ist am System angemeldet und besitzt mindestens eine der Kompetenzen `ANGEBOT_LESEN`, `ANTRAG_LESEN` oder `VERTRAG_LESEN`
- Mindestens ein KFZ-Angebot, -Antrag oder -Vertrag mit Fahrzeugdaten existiert im System

## Auslöser
- Benutzer gibt ein amtliches Kennzeichen in das Suchfeld der KFZ-Kennzeichensuche ein
- Alternativ: Benutzer nutzt die globale Suche mit einem Wert im Kennzeichen-Format

## Ablauf (Hauptszenario)

### Phase 1: Kennzeichen eingeben
1. Benutzer navigiert zur **KFZ-Kennzeichensuche** (über KFZ-Menü oder globale Suche)
2. System zeigt das Suchfeld für amtliche Kennzeichen an
3. Benutzer gibt das Kennzeichen ein (vollständig oder als Teilsuche mit Platzhalter)
4. System validiert das Eingabeformat (→ Plausibilität: deutsches Kennzeichenformat)

### Phase 2: Suche durchführen
5. System durchsucht alle Fahrzeug-Entitäten (`fahrzeug.amtliches_kennzeichen`) in:
   - **Angeboten** (Fahrzeug referenziert über `referenz_typ = ANGEBOT`)
   - **Anträgen** (Fahrzeug referenziert über `referenz_typ = ANTRAG`)
   - **Verträgen** (Fahrzeug referenziert über `referenz_typ = VERTRAG`)
6. System durchsucht zusätzlich die **Vertragsstand-Historie** (Hibernate Envers / Temporal Tables), um Kennzeichen in **nicht mehr aktuellen Vertragsständen** zu identifizieren
7. System prüft für jeden Treffer den **Aktualitätsstatus** (aktiv vs. historisch)

### Phase 3: Ergebnis anzeigen
8. System zeigt die **konsolidierte Trefferliste** an, aufgeteilt in zwei Bereiche:

   **Bereich 1 – Aktive Treffer** (zuerst angezeigt):
   - Angebote im Status Entwurf, Berechnet, Geprüft oder Beantragt
   - Anträge im Status Offen, Berechnet, Geprüft, Freigegeben oder Ausgesteuert
   - Verträge mit dem Kennzeichen im **aktuell gültigen Vertragsstand** (Status Aktiv, Ruhend oder Gekündigt)

   **Bereich 2 – Historische Treffer** (visuell abgesetzt, z. B. grau/kursiv):
   - Angebote im Status Gelöscht
   - Anträge im Status Abgelehnt oder Storniert
   - Verträge, bei denen das Kennzeichen in einem **früheren Vertragsstand** enthalten war (z. B. vor einem Fahrzeugwechsel → UC-KFZ-04), aber im aktuellen Stand nicht mehr
   - Verträge im Status Abgelaufen

9. Jeder Treffer zeigt folgende Informationen:
   - **Typ-Icon** (Angebot / Antrag / Vertrag)
   - **Fachliche Nummer** (Angebots-, Antrags- oder Vertragsnummer)
   - **Kunde** (Name + Partnernummer)
   - **Status** (als farbiger Chip)
   - **Kennzeichen** (hervorgehoben)
   - **Fahrzeug** (Hersteller + Modell, falls vorhanden)
   - **Aktualität** (Badge: „Aktuell" grün / „Historisch" grau)
   - **Vertragsstand-Version** (bei Verträgen: welcher Stand das Kennzeichen enthält)
   - **Gültigkeitszeitraum** (bei historischen Vertragsständen: gültig von–bis)

10. Benutzer kann einen Treffer auswählen und zum jeweiligen Detail navigieren

### Phase 4: Navigation zum Detail
11. Benutzer klickt auf einen Treffer
12. System navigiert zur Detailansicht des gewählten Geschäftsobjekts:
    - Angebot → Screen 5 (Angebot bearbeiten, → UC-01)
    - Antrag → Screen 7 (Antrag bearbeiten, → UC-02)
    - Vertrag → Screen 11 (Vertragsdetails), bei historischen Treffern direkt zum betreffenden Vertragsstand (Tab „Vertragsstände" mit Vorauswahl der Version)

## Alternativszenarien

- **A1: Teilsuche / Platzhaltersuche**
  Der Benutzer gibt nur einen Teil des Kennzeichens ein (z. B. „MS-LV" ohne Nummer). Das System führt eine Präfix-/Teilstringsuche durch und zeigt alle Treffer mit passendem Kennzeichenanteil an. Es wird ein Hinweis angezeigt: „Teilsuche – es werden alle Kennzeichen angezeigt, die ‚MS-LV' enthalten."

- **A2: Suche über globale Suche**
  Gibt der Benutzer in der globalen Suche (Strg+K) ein Kennzeichen ein, erkennt das System das Kennzeichenformat und bietet neben den Standard-Ergebnissen (Partner, Nummern) einen zusätzlichen Bereich „KFZ-Kennzeichensuche" an. Klick darauf öffnet die vollständige Kennzeichensuche mit vorausgefülltem Suchfeld.

- **A3: Mehrfachtreffer – gleiches Kennzeichen in Angebot UND Vertrag**
  Ein Kennzeichen kann gleichzeitig in einem Angebot (z. B. Tarifwechsel-Angebot) und im laufenden Vertrag vorhanden sein. Beide Treffer werden angezeigt, der Vertrag als „Aktuell", das Angebot mit seinem jeweiligen Status.

- **A4: Kennzeichen in mehreren historischen Vertragsständen**
  Wurde ein Fahrzeug mehrfach ab- und wieder zugemeldet (z. B. Ruheversicherung → Reaktivierung → erneuter Wechsel), kann das Kennzeichen in mehreren Vertragsständen desselben Vertrags auftauchen. Jeder betroffene Vertragsstand wird als eigener Treffer mit Version und Gültigkeitszeitraum angezeigt.

- **A5: Ergebnis filtern**
  Der Benutzer kann die Trefferliste nach Typ (Angebot / Antrag / Vertrag), Aktualität (Aktuell / Historisch) und Status filtern, um bei vielen Treffern die Übersicht zu behalten.

## Fehlerfälle

- **F1: Kein Treffer**
  Das Kennzeichen ist in keinem Angebot, Antrag oder Vertrag hinterlegt. → System zeigt: „Kein Ergebnis. Für das Kennzeichen ‹MS-XX 1234› wurden keine Angebote, Anträge oder Verträge gefunden." mit der Möglichkeit, die Suche anzupassen.

- **F2: Ungültiges Kennzeichenformat**
  Die Eingabe entspricht keinem deutschen Kennzeichenformat. → System zeigt Inline-Validierungsfehler: „Bitte geben Sie ein gültiges Kennzeichen ein (z. B. MS-LV 1234)." Die Suche wird nicht ausgelöst. Bei Teilsuche (≥ 2 Zeichen) wird trotzdem gesucht, mit Hinweis.

- **F3: Kompetenz fehlt für bestimmte Treffertypen**
  Der Benutzer besitzt z. B. `VERTRAG_LESEN`, aber nicht `ANGEBOT_LESEN`. → Es werden nur die Treffer angezeigt, für die der Benutzer die entsprechende Lesekompetenz besitzt. Treffer ohne Kompetenz werden nicht angezeigt (nicht ausgegraut), um keine Informationen preiszugeben.

- **F4: Sehr viele Treffer bei Teilsuche**
  Eine Teilsuche liefert mehr als 100 Treffer. → System zeigt die ersten 100 Treffer und den Hinweis: „Es wurden mehr als 100 Treffer gefunden. Bitte grenzen Sie die Suche ein." Paginierung ist verfügbar.

- **F5: Historiensuche dauert zu lange**
  Die Suche in den Audit-Tabellen (Envers) dauert bei großem Bestand zu lange (> 2 Sekunden). → System zeigt zuerst die aktiven Treffer (schnelle Suche in Produktiv-Tabellen) und lädt die historischen Treffer asynchron nach. Ladeindikator: „Historische Treffer werden geladen…"

## Geschäftsregeln
- GR-KFZ-KS01: Die Kennzeichensuche durchsucht Angebote, Anträge und Verträge der Sparte KFZ
- GR-KFZ-KS02: Kennzeichen in nicht mehr gültigen Vertragsständen werden als „Historisch" gekennzeichnet und visuell abgesetzt dargestellt
- GR-KFZ-KS03: Die Trefferliste ist nach Aktualität sortiert: aktive Treffer vor historischen Treffern
- GR-KFZ-KS04: Treffer werden nur angezeigt, wenn der Benutzer die jeweilige Lesekompetenz besitzt (Angebot, Antrag oder Vertrag)
- GR-KFZ-KS05: Bei historischen Vertragsständen wird der Gültigkeitszeitraum (gültig_ab / gültig_bis) und die Version des Vertragsstands angezeigt
- GR-KFZ-KS06: Die Suche unterstützt Teilsuche (Präfix/Teilstring) ab mindestens 2 Zeichen

## Daten (Ein-/Ausgabe)

### Eingabedaten
| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| Kennzeichen | String(15) | ✅ | Deutsches Kennzeichenformat (Regex) oder Teilstring (≥ 2 Zeichen) | Amtliches Kennzeichen oder Teil davon |

### Ausgabedaten
| Feld | Typ | Beschreibung |
|------|-----|-------------|
| Trefferliste | Liste | Konsolidierte Liste aller Fundstellen |
| Treffer.typ | Enum | `ANGEBOT` / `ANTRAG` / `VERTRAG` |
| Treffer.fachliche_nummer | String | Angebots-, Antrags- oder Vertragsnummer |
| Treffer.partner | Objekt | Partnernummer + Name des Kunden |
| Treffer.status | Enum | Status des jeweiligen Geschäftsobjekts |
| Treffer.kennzeichen | String | Das gefundene amtliche Kennzeichen (vollständig) |
| Treffer.fahrzeug | Objekt | Hersteller, Modell (sofern vorhanden) |
| Treffer.aktualitaet | Enum | `AKTUELL` / `HISTORISCH` |
| Treffer.vertragsstand_version | Integer | Version des Vertragsstands (bei Vertragstreffern) |
| Treffer.gueltig_ab | Date | Beginn der Gültigkeit des Vertragsstands (bei historischen) |
| Treffer.gueltig_bis | Date | Ende der Gültigkeit des Vertragsstands (bei historischen) |
| Treffer.historisch_grund | String | Grund für den historischen Status (z. B. „Fahrzeugwechsel", „Vertrag abgelaufen") |
| Anzahl_aktiv | Integer | Anzahl aktiver Treffer |
| Anzahl_historisch | Integer | Anzahl historischer Treffer |

## API-Endpunkt

> Ergänzung zur KFZ-Sparten-API (→ [09_schnittstellen.md](../../../09_schnittstellen.md), Abschnitt 2.7)

| Methode | Endpunkt | Beschreibung | Kompetenz |
|---------|---------|-------------|-----------|
| `GET` | `/api/v1/kfz/kennzeichensuche` | Kennzeichensuche über Angebote, Anträge und Verträge | `ANGEBOT_LESEN` und/oder `ANTRAG_LESEN` und/oder `VERTRAG_LESEN` |

**Query-Parameter:**

| Parameter | Typ | Pflicht | Beschreibung |
|-----------|-----|---------|-------------|
| `kennzeichen` | String | ✅ | Vollständiges Kennzeichen oder Teilstring (≥ 2 Zeichen) |
| `include_historisch` | Boolean | ❌ | Historische Treffer einschließen (Default: `true`) |
| `typ` | Enum | ❌ | Filter: `ANGEBOT`, `ANTRAG`, `VERTRAG` (kommagetrennt für mehrere) |
| `page` | Integer | ❌ | Seitennummer (Default: `0`) |
| `size` | Integer | ❌ | Seitengröße (Default: `20`, Max: `100`) |

**Response (200 OK):**
```json
{
  "kennzeichen_gesucht": "MS-LV 1234",
  "anzahl_aktiv": 2,
  "anzahl_historisch": 1,
  "content": [
    {
      "typ": "VERTRAG",
      "fachliche_nummer": "VN-2026-100001",
      "partner": {
        "partnernummer": "KD-2026-004711",
        "name": "Max Mustermann"
      },
      "status": "AKTIV",
      "kennzeichen": "MS-LV 1234",
      "fahrzeug": {
        "hersteller": "Volkswagen",
        "modell": "Golf VIII 1.5 TSI",
        "hsn": "0603",
        "tsn": "BPM"
      },
      "aktualitaet": "AKTUELL",
      "vertragsstand_version": 2,
      "gueltig_ab": "2027-01-01",
      "gueltig_bis": null,
      "historisch_grund": null
    },
    {
      "typ": "ANGEBOT",
      "fachliche_nummer": "AG-2026-001567",
      "partner": {
        "partnernummer": "KD-2026-004711",
        "name": "Max Mustermann"
      },
      "status": "BERECHNET",
      "kennzeichen": "MS-LV 1234",
      "fahrzeug": {
        "hersteller": "Volkswagen",
        "modell": "Golf VIII 1.5 TSI",
        "hsn": "0603",
        "tsn": "BPM"
      },
      "aktualitaet": "AKTUELL",
      "vertragsstand_version": null,
      "gueltig_ab": null,
      "gueltig_bis": null,
      "historisch_grund": null
    },
    {
      "typ": "VERTRAG",
      "fachliche_nummer": "VN-2026-100001",
      "partner": {
        "partnernummer": "KD-2026-004711",
        "name": "Max Mustermann"
      },
      "status": "AKTIV",
      "kennzeichen": "MS-LV 1234",
      "fahrzeug": {
        "hersteller": "Volkswagen",
        "modell": "Golf VII 1.4 TSI",
        "hsn": "0603",
        "tsn": "AKZ"
      },
      "aktualitaet": "HISTORISCH",
      "vertragsstand_version": 1,
      "gueltig_ab": "2026-01-01",
      "gueltig_bis": "2026-12-31",
      "historisch_grund": "Fahrzeugwechsel (UC-KFZ-04)"
    }
  ],
  "page": {
    "number": 0,
    "size": 20,
    "totalElements": 3,
    "totalPages": 1
  }
}
```

## Datenbank-Abfragestrategie

### Aktive Treffer (schnelle Suche)
```sql
-- Angebote
SELECT a.*, f.amtliches_kennzeichen, f.hersteller, f.modell
FROM angebot a
JOIN fahrzeug f ON f.referenz_id = a.id AND f.referenz_typ = 'ANGEBOT'
WHERE f.amtliches_kennzeichen ILIKE :kennzeichen_pattern
  AND a.sparte = 'KFZ'
  AND a.status NOT IN ('GELOESCHT');

-- Anträge
SELECT an.*, f.amtliches_kennzeichen, f.hersteller, f.modell
FROM antrag an
JOIN fahrzeug f ON f.referenz_id = an.id AND f.referenz_typ = 'ANTRAG'
WHERE f.amtliches_kennzeichen ILIKE :kennzeichen_pattern
  AND an.sparte = 'KFZ'
  AND an.status NOT IN ('ABGELEHNT', 'STORNIERT');

-- Verträge (aktueller Stand)
SELECT v.*, f.amtliches_kennzeichen, f.hersteller, f.modell
FROM vertrag v
JOIN fahrzeug f ON f.referenz_id = v.id AND f.referenz_typ = 'VERTRAG'
WHERE f.amtliches_kennzeichen ILIKE :kennzeichen_pattern
  AND v.sparte = 'KFZ';
```

### Historische Treffer (asynchrone Nachladung)
```sql
-- Historische Vertragsstände über Envers Audit-Tabellen
SELECT v.vertragsnummer, vs_aud.version, vs_aud.gueltig_ab, vs_aud.gueltig_bis,
       f_aud.amtliches_kennzeichen, f_aud.hersteller, f_aud.modell
FROM fahrzeug_aud f_aud
JOIN vertragsstand_aud vs_aud ON vs_aud.REV = f_aud.REV
JOIN vertrag v ON v.id = vs_aud.vertrag_id
WHERE f_aud.amtliches_kennzeichen ILIKE :kennzeichen_pattern
  AND f_aud.referenz_typ = 'VERTRAG'
  AND vs_aud.gueltig_bis IS NOT NULL  -- nur abgelöste Stände
  AND v.sparte = 'KFZ';

-- Gelöschte Angebote, abgelehnte/stornierte Anträge
SELECT a.*, f.amtliches_kennzeichen, f.hersteller, f.modell
FROM angebot a
JOIN fahrzeug f ON f.referenz_id = a.id AND f.referenz_typ = 'ANGEBOT'
WHERE f.amtliches_kennzeichen ILIKE :kennzeichen_pattern
  AND a.sparte = 'KFZ'
  AND a.status IN ('GELOESCHT');
-- (analog für Anträge mit Status ABGELEHNT, STORNIERT)
```

> **Performance-Hinweis:** Der Index `idx_fahrzeug_kennzeichen` (B-Tree auf `amtliches_kennzeichen`) unterstützt die Suche in den Produktivtabellen. Für die Audit-Tabellen (`fahrzeug_aud`) wird ein zusätzlicher Index empfohlen: `idx_fahrzeug_aud_kennzeichen` auf `amtliches_kennzeichen`.

## Nachbedingungen
- Die Trefferliste wird dem Benutzer angezeigt, aufgeteilt in aktive und historische Treffer
- Historische Treffer sind visuell als solche erkennbar (Badge, Farbgebung, Gültigkeitszeitraum)
- Der Benutzer kann von jedem Treffer direkt zur Detailansicht navigieren
- Es werden nur Treffer angezeigt, für die der Benutzer die jeweilige Lesekompetenz besitzt

## Akzeptanzkriterien
- [ ] Die Suche nach einem vollständigen Kennzeichen liefert alle Angebote, Anträge und Verträge, die dieses Kennzeichen enthalten
- [ ] Treffer in nicht mehr gültigen Vertragsständen werden als „Historisch" markiert und visuell abgesetzt
- [ ] Historische Treffer zeigen Version und Gültigkeitszeitraum des betroffenen Vertragsstands an
- [ ] Aktive Treffer werden vor historischen Treffern angezeigt
- [ ] Teilsuche (z. B. „MS-LV") liefert alle Kennzeichen, die den Suchbegriff enthalten
- [ ] Bei fehlender Lesekompetenz für einen Treffertyp werden die entsprechenden Treffer ausgeblendet
- [ ] Die Navigation vom Treffer zur Detailansicht funktioniert korrekt (Angebot, Antrag, Vertrag)
- [ ] Bei historischen Vertragstreffern wird der korrekte Vertragsstand vorausgewählt
- [ ] Die Suche in den Produktivtabellen (aktive Treffer) ist performant (≤ 1 Sekunde, → NFA-P02)
- [ ] Historische Treffer werden asynchron nachgeladen, ohne die Anzeige der aktiven Treffer zu verzögern
- [ ] Bei > 100 Treffern wird paginiert und ein Hinweis zur Eingrenzung angezeigt
- [ ] Ein ungültiges Kennzeichenformat wird mit einer Fehlermeldung quittiert

## Wireframe / Skizze

```
┌─────────────────────────────────────────────────────────────────┐
│  KFZ-Kennzeichensuche                                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  🔍 Amtliches Kennzeichen: [ MS-LV 1234          ] [Suchen]   │
│                                                                 │
│  3 Treffer gefunden (2 aktuell, 1 historisch)                  │
│                                                                 │
│  ── Aktuelle Treffer ──────────────────────────────────────     │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ 📂 Vertrag VN-2026-100001    ⬤ Aktiv    🟢 Aktuell    │   │
│  │ Kunde: Max Mustermann (KD-2026-004711)                  │   │
│  │ 🚗 VW Golf VIII 1.5 TSI    Kennzeichen: MS-LV 1234    │   │
│  │ Vertragsstand: Version 2 (gültig ab 01.01.2027)        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ 📋 Angebot AG-2026-001567    ⬤ Berechnet  🟢 Aktuell  │   │
│  │ Kunde: Max Mustermann (KD-2026-004711)                  │   │
│  │ 🚗 VW Golf VIII 1.5 TSI    Kennzeichen: MS-LV 1234    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ── Historische Treffer ───────────────────────────────────     │
│                                                                 │
│  ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐   │
│  │ 📂 Vertrag VN-2026-100001    ⬤ Aktiv    ⬜ Historisch  │   │
│  │ Kunde: Max Mustermann (KD-2026-004711)                  │   │
│  │ 🚗 VW Golf VII 1.4 TSI     Kennzeichen: MS-LV 1234    │   │
│  │ Vertragsstand: Version 1 (01.01.2026 – 31.12.2026)     │   │
│  │ Grund: Fahrzeugwechsel                                  │   │
│  └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘   │
│                                                                 │
│  Filter: [Alle ▾] [Aktuell ▾] [Status ▾]                      │
└─────────────────────────────────────────────────────────────────┘

Legende:
  ──────  = Aktiver Treffer (durchgezogener Rahmen, weiß)
  ─ ─ ─  = Historischer Treffer (gestrichelter Rahmen, grau hinterlegt)
```

## Offene Fragen
- Soll die Kennzeichensuche auch ausländische Kennzeichen unterstützen (z. B. für Grenzregionen)?
- Soll ein Suchverlauf der zuletzt gesuchten Kennzeichen angezeigt werden?
- Sollen die historischen Treffer standardmäßig eingeklappt sein (Expansion Panel) oder direkt sichtbar?
- Wird ein eigener Index auf `fahrzeug_aud.amtliches_kennzeichen` für performante Historiensuche benötigt, oder reicht der bestehende Audit-Mechanismus?
- Soll die Kennzeichensuche in die Schadenverwaltung (S6) integriert werden (z. B. Deckungsauskunft per Kennzeichen)?
