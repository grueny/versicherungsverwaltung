# UI-Anforderungen – Versicherungsverwaltung

> Technologiebasis: **Angular 19** + **Angular Material** (Material Design), **TypeScript**, **Angular Reactive Forms**, **Angular Signals**.
> Referenzen: → [11_technische_rahmenbedingungen.md](11_technische_rahmenbedingungen.md), → [06_nichtfunktionale_anforderungen.md](06_nichtfunktionale_anforderungen.md) (NFA-U01–U06), → [09_schnittstellen.md](09_schnittstellen.md)

---

## Allgemeine Design-Prinzipien

| Nr. | Prinzip | Details |
|-----|---------|---------|
| DP-01 | **Spartenübergreifende Konsistenz** | Navigation, Layouts und Interaktionsmuster sind über alle Sparten hinweg einheitlich (→ NFA-U06). Gleiche Aktionen führen in jeder Sparte zum gleichen Verhalten (Principle of Least Surprise). |
| DP-02 | **Design-System: Angular Material** | Alle Komponenten basieren auf Angular Material (Material Design 3). Kein Custom-CSS für Standardkomponenten – Angular Material Theming für Branding. |
| DP-03 | **Farbschema & Branding** | Primärfarbe: Unternehmensblau (konfigurierbar über Angular Material Custom Theme). Sekundärfarbe: Akzentfarbe für Aktionen/CTAs. Statusfarben: ✅ Grün (Erfolg), ⚠️ Gelb/Orange (Warnung), ❌ Rot (Fehler), ℹ️ Blau (Info). Kontrastverhältnisse ≥ 4,5:1 (→ NFA-U01, WCAG 2.1 AA). |
| DP-04 | **Typografie** | Roboto (Angular Material Default) als Systemschriftart. Hierarchie: H1 (Seitentitel) → H2 (Abschnittstitel) → H3 (Kartentitel) → Body. Mindestschriftgröße: 14px (Body), 12px (Labels/Hilfstexte). |
| DP-05 | **Sprache** | Alle UI-Texte, Fehlermeldungen, Tooltips und Hilfetexte in **Deutsch** (de_DE) (→ NFA-U04). i18n über Angular i18n-Framework vorbereitet für spätere Erweiterung. |
| DP-06 | **Formulargestaltung** | Angular Reactive Forms mit Inline-Validierung. Pflichtfelder mit `*` gekennzeichnet. Fehlermeldungen direkt unter dem Feld in Rot. Plausibilitätsergebnisse (Fehler / Warnungen / Hinweise) werden in Echtzeit angezeigt (→ UC-01, Schritt 9–10). |
| DP-07 | **Spartenspezifische Bereiche** | Spartenspezifische UI-Abschnitte werden in visuell abgesetzten Containern (Card/Panel) dargestellt, z. B. „KFZ-Daten". Lazy Loading pro Spartenmodul (→ NFA-P01). |
| DP-08 | **Statusvisualisierung** | Jeder Status wird durch **Farbe + Icon + Text** dargestellt (nicht nur Farbe – a11y). Status-Chips (Angular Material Chips) in Listenansichten. |
| DP-09 | **Aktionssichtbarkeit** | Aktionen (Buttons) werden nur angezeigt, wenn der Benutzer die erforderliche **Kompetenz** besitzt (→ S8). Nicht verfügbare Aktionen sind ausgeblendet (nicht ausgegraut), um die UI nicht zu überladen. |
| DP-10 | **Ladeindikator** | Bei API-Aufrufen > 300 ms wird ein Ladeindikator (Spinner/Progress Bar) angezeigt. Skeleton-Screens für initiale Seitenladungen. |

---

## Navigationsstruktur

> Hauptnavigation als vertikale Sidebar (Angular Material Sidenav), einklappbar. Breadcrumb-Leiste unter der Toolbar.

```
├── 📊 Dashboard
├── 👤 Kunden
│   ├── Kundenübersicht (Suche + Liste)
│   ├── Kunde anlegen
│   └── Kundendetails
│       ├── Stammdaten
│       ├── Angebote des Kunden
│       ├── Anträge des Kunden
│       ├── Verträge des Kunden
│       └── Schäden des Kunden
├── 📋 Angebote
│   ├── Angebotsübersicht (Suche + Liste)
│   └── Angebot erstellen / bearbeiten (→ UC-01)
│       └── [Spartenspezifisch: z. B. Fahrzeugartauswahl bei KFZ → UC-KFZ-00]
├── 📝 Anträge
│   ├── Antragsübersicht (Suche + Liste)
│   └── Antrag bearbeiten (→ UC-02)
├── ⏳ Schweben
│   ├── Schweben-Übersicht (Filter: offen, Team, Bearbeiter)
│   └── Schwebe-Details (Entscheidung: Freigeben / Ablehnen / Zurückweisen)
├── 📂 Verträge
│   ├── Vertragsübersicht (Suche + Liste)
│   └── Vertragsdetails
│       ├── Aktueller Vertragsstand
│       ├── Vertragsstände (Versionshistorie)
│       ├── Vorgänge
│       ├── Schäden
│       └── Änderungshistorie (Envers)
├── 🔔 Benachrichtigungen
│   └── Notification-Center (Schwebe-Zuweisungen, Wiedervorlagen, eVB-Ablauf etc.)
└── ⚙️ Verwaltung (nur mit Kompetenz KONFIGURATION_LESEN)
    ├── Sparten & Produkte (Lesezugriff)
    └── Aussteuerungsregeln (Lesezugriff)
```

### Toolbar (Header)

```
┌────────────────────────────────────────────────────────────────┐
│ ☰  [Logo] Versicherungsverwaltung    🔍 Globale Suche    🔔 3  👤 ad-mueller ▾ │
└────────────────────────────────────────────────────────────────┘
```

| Element | Beschreibung |
|---------|-------------|
| ☰ Hamburger | Sidebar ein-/ausklappen |
| Logo + Titel | Branding, Klick → Dashboard |
| Globale Suche | Suche über Partner, Angebote, Anträge, Verträge nach Nummer oder Name |
| 🔔 Notifications | Badge mit Anzahl ungelesener Benachrichtigungen (→ NOT-01 bis NOT-05) |
| 👤 Benutzername | Dropdown: Profil, Abmelden |

### Breadcrumbs

```
Dashboard > Angebote > AG-2026-001234
```

- Breadcrumbs auf jeder Seite unterhalb der Toolbar
- Klickbar für Navigation zu übergeordneten Ebenen

---

## Screens / Seitenübersicht

### Screen 1: Dashboard

- **Zweck:** Startseite nach Login – Überblick über relevante Arbeitspakete und Kennzahlen
- **Kompetenz:** Alle angemeldeten Benutzer
- **Inhalte:**
  - **Kennzahlen-Kacheln** (Angular Material Cards):
    - Offene Angebote (Anzahl, aufgeteilt nach Status)
    - Offene Anträge (Anzahl, aufgeteilt nach Status)
    - Offene Schweben (Anzahl, ggf. nach Team)
    - Aktive Verträge (Gesamtanzahl)
  - **Meine offenen Aufgaben** (Tabelle, max. 10 Einträge):
    - Zugewiesene Schweben
    - Wiedervorlagen (fällig heute/überfällig)
    - Bald verfallende Angebote (→ NOT-05)
    - eVB-Ablauf-Warnungen (→ NOT-04)
  - **Zuletzt bearbeitet** (Tabelle, max. 5 Einträge):
    - Die letzten durch den Benutzer bearbeiteten Angebote, Anträge oder Verträge mit Zeitstempel
  - **Schnellzugriff-Kacheln**:
    - „Neues Angebot erstellen" (→ UC-01)
    - „Schweben-Übersicht" (nur Innendienst)
- **Aktionen:**
  - Klick auf Kennzahl → Navigiert zur jeweiligen gefilterten Liste
  - Klick auf Aufgabe → Navigiert zum Detail (Schwebe, Angebot etc.)
  - „Neues Angebot erstellen" → UC-01

### Screen 2: Kundenübersicht

- **Zweck:** Kunden suchen und auflisten
- **Kompetenz:** `PARTNER_LESEN`
- **Inhalte:**
  - **Suchbereich** (oberhalb der Tabelle):
    - Freitextsuche (Name, Partnernummer)
    - Erweiterte Filter: PLZ, Ort, Partnertyp
  - **Ergebnistabelle** (Angular Material Table mit Paginierung):
    - Spalten: Partnernummer, Name/Vorname, Geburtsdatum, PLZ/Ort, Partnertyp, Aktionen
    - Sortierbar nach allen Spalten
    - Paginierung: 20 Einträge pro Seite (Standard)
- **Aktionen:**
  - „Kunde anlegen" (Button oben rechts, Kompetenz `PARTNER_ANLEGEN`)
  - Zeile klicken → Kundendetails
  - Inline-Aktionen pro Zeile: „Neues Angebot" (→ UC-01)

### Screen 3: Kundendetails

- **Zweck:** 360°-Sicht auf einen Kunden mit allen zugehörigen Geschäftsobjekten
- **Kompetenz:** `PARTNER_LESEN`
- **Inhalte:**
  - **Header-Card**: Partnernummer, Name, Anrede, Adresse, Kontaktdaten
  - **Tab-Navigation** (Angular Material Tabs):
    - **Stammdaten**: Bearbeitbares Formular (Kompetenz `PARTNER_BEARBEITEN`)
    - **Angebote**: Tabelle aller Angebote des Kunden (→ API: GET /partner/{id}/angebote)
    - **Anträge**: Tabelle aller Anträge
    - **Verträge**: Tabelle aller Verträge mit Statusanzeige
    - **Schäden**: Tabelle aller Schadenreferenzen
    - **Historie**: Revisionssichere Änderungshistorie der Partnerdaten (→ Historisierungs-API)
- **Aktionen:**
  - „Bearbeiten" / „Speichern" (Stammdaten-Tab)
  - „Neues Angebot für diesen Kunden" (→ UC-01 mit vorausgewähltem Kunden)
  - Klick auf Zeile in Tabs → Navigation zum jeweiligen Detail

### Screen 4: Angebotsübersicht

- **Zweck:** Angebote suchen, filtern und verwalten
- **Kompetenz:** `ANGEBOT_LESEN`
- **Inhalte:**
  - **Filterbereich**:
    - Status (Chips/Dropdown: Entwurf, Berechnet, Geprüft, Beantragt, Gelöscht)
    - Sparte (Dropdown)
    - Zeitraum (Erstellt von–bis)
    - Partnernummer / Angebotsnummer (Freitext)
  - **Ergebnistabelle**:
    - Spalten: Angebotsnummer, Kunde (Name + Nr.), Sparte, Status (Chip), Jahresbeitrag, Erstellt am, Erstellt von
    - Sortierbar, paginiert (20/Seite)
- **Aktionen:**
  - „Neues Angebot" (→ UC-01)
  - Zeile klicken → Angebot bearbeiten
  - Inline: Kopieren (→ A3 UC-01), Löschen (→ A2 UC-01)

### Screen 5: Angebot erstellen / bearbeiten (→ UC-01)

- **Zweck:** Zentraler Arbeitsscreen für den Angebotsprozess (Phasen 1–5 aus UC-01)
- **Kompetenz:** `ANGEBOT_ANLEGEN` / `ANGEBOT_BEARBEITEN` / `ANGEBOT_BEANTRAGEN`
- **Layout:** Mehrstufiges Formular mit Fortschrittsanzeige (Stepper oder Tab-basiert)
- **Inhalte:**
  - **Header**: Angebotsnummer, Status (farbiger Chip), Kunde (mit Link zur Kundendetails)
  - **Abschnitt 1 – Kundendaten**: Kundenauswahl (Bestandskunde suchen oder Neuerfassung)
  - **Abschnitt 2 – Spartenauswahl**: Auswahl der Sparte (Dropdown/Kacheln)
  - **Abschnitt 3 – Spartenspezifisch** (dynamisch geladen per Lazy Loading):
    - KFZ: Fahrzeugartauswahl (→ UC-KFZ-00 Kachelansicht), Fahrzeugdaten (HSN/TSN, Kennzeichen, FIN), Tarifmerkmale (SF-Klasse, Fahrerkreis, Fahrleistung, Stellplatz etc.)
  - **Abschnitt 4 – Produktauswahl**: Verfügbare Produkte der Sparte als Checkbox-Liste mit Beschreibung, Deckungsbausteinen und Abhängigkeitshinweisen
  - **Abschnitt 5 – Vertragsdaten**: Vertragsbeginn, Laufzeit, Zahlungsweise, Selbstbeteiligung
  - **Abschnitt 6 – Prüfergebnis**: Panel mit Plausibilitätsergebnissen (Fehler ❌, Warnungen ⚠️, Hinweise ℹ️) – erscheint nach Prüfung
  - **Abschnitt 7 – Beitragsübersicht**: Berechneter Jahresbeitrag mit Aufschlüsselung pro Produkt/Baustein – erscheint nach Berechnung
- **Aktionen-Leiste** (fixiert am unteren Rand / Sticky Footer):
  - `[Speichern]` – jederzeit verfügbar (→ A1 UC-01)
  - `[Berechnen]` – verfügbar ab vollständiger Datenlage
  - `[Prüfen]` – verfügbar nach Berechnung
  - `[Beantragen]` – nur bei Status „Geprüft" (→ GR-A01); öffnet Bestätigungsdialog „Angebot behalten?"
  - `[Drucken / Exportieren]` – nach Berechnung (→ A4 UC-01, S5)
  - `[Löschen]` – nur vor Beantragung (→ A2 UC-01)
- **Validierung:**
  - Inline-Validierung bei Eingabe (Angular Reactive Forms Validators)
  - Plausibilitätsprüfungen in Echtzeit (spartenübergreifend + spartenspezifisch aus Regelengine)
  - Fehlermeldungen unter dem jeweiligen Feld

**Wireframe** (vgl. UC-01):

```
┌─────────────────────────────────────────────────────────┐
│  Angebot #AG-2026-001234          Status: ⬤ Berechnet  │
├─────────────────────────────────────────────────────────┤
│  Kunde: Max Mustermann (KD-4711)         [Ändern]      │
│  Sparte: KFZ                                           │
├─────────────────────────────────────────────────────────┤
│  ┌─── Spartenspezifisch (KFZ) ──────────────────────┐  │
│  │ Fahrzeugart: PKW                                 │  │
│  │ Fahrzeug: VW Golf VIII (HSN/TSN: 0603/BPM)      │  │
│  │ Typklasse HP: 15 | TK: 20 | VK: 17              │  │
│  │ SF-Klasse: SF 5   Fahrerkreis: ab 25 Jahre       │  │
│  └──────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────┤
│  Produkte:                                              │
│  ☑ KFZ-Haftpflicht          ☑ Teilkasko                │
│  ☐ Vollkasko                ☑ LVM-Schutzbrief           │
├─────────────────────────────────────────────────────────┤
│  Vertragsbeginn: 01.01.2027    Laufzeit: 1 Jahr        │
│  Zahlweise: Jährlich          SB Teilkasko: 150 €      │
├─────────────────────────────────────────────────────────┤
│  Berechneter Jahresbeitrag:            € 487,32        │
│  ├─ KFZ-Haftpflicht:                  € 312,00        │
│  ├─ Teilkasko:                         € 167,11        │
│  └─ LVM-Schutzbrief:                  €   8,21        │
├─────────────────────────────────────────────────────────┤
│  ⚠ Hinweis: Gelegentliche Fahrer ab 23 mitversichert  │
├─────────────────────────────────────────────────────────┤
│  [Speichern]  [Berechnen]  [Prüfen]  [Beantragen]     │
└─────────────────────────────────────────────────────────┘
```

### Screen 5a: Fahrzeugartauswahl (KFZ – UC-KFZ-00)

- **Zweck:** Auswahl der Fahrzeugart vor der KFZ-Angebotserstellung (Hook `vor_Angebotserstellung`)
- **Kompetenz:** `ANGEBOT_ANLEGEN`
- **Darstellung:** Vollflächige Kacheln (Angular Material Cards) mit Icon, Titel und Kurzbeschreibung
- **Inhalte:**
  - 8 Fahrzeugarten (FA-01 bis FA-08) als Kacheln in einem Grid (3 Spalten Desktop, 2 Spalten Tablet)
  - Nicht verfügbare Fahrzeugarten ausgegraut mit Tooltip (→ F1, F2 UC-KFZ-00)
- **Aktionen:**
  - Kachel klicken → Fahrzeugart auswählen, weiter zu Angebotserstellung
  - `[Abbrechen]` → zurück zur Angebotsübersicht

**Wireframe** (vgl. UC-KFZ-00):

```
┌─────────────────────────────────────────────────────────┐
│  Neues KFZ-Angebot – Fahrzeugart wählen                │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │  🚗 PKW     │  │  🚛 LKW     │  │  🏎️ Oldtimer│    │
│  │ PKW, SUV,   │  │ Transport,  │  │ H-Kennz.,  │    │
│  │ Kombi, ...  │  │ Nutzfzg.    │  │ mind. 30 J. │    │
│  └─────────────┘  └─────────────┘  └─────────────┘    │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │  🛵 Vers.-  │  │  🏍️ Motor-  │  │  🏕️ Wohn-   │    │
│  │  Kennzeichen│  │    rad      │  │ mobil/Wagen │    │
│  └─────────────┘  └─────────────┘  └─────────────┘    │
│  ┌─────────────┐  ┌─────────────┐                      │
│  │  🚜 Sonder- │  │  🚛 Anhänger│                      │
│  │  fahrzeug   │  │             │                      │
│  └─────────────┘  └─────────────┘                      │
│                                          [Abbrechen]    │
└─────────────────────────────────────────────────────────┘
```

### Screen 6: Antragsübersicht

- **Zweck:** Anträge suchen, filtern und verwalten
- **Kompetenz:** `ANTRAG_LESEN`
- **Inhalte:**
  - **Filterbereich**: Status (Offen, Berechnet, Geprüft, Freigegeben, Ausgesteuert, Abgelehnt, Storniert), Sparte, Zeitraum, Partner/Antragsnummer
  - **Ergebnistabelle**:
    - Spalten: Antragsnummer, Angebotsnummer (Link), Kunde, Sparte, Status (Chip), Jahresbeitrag, Vorgangstyp, Erstellt am
    - Ausgesteuerte Anträge mit auffälligem Status-Chip (z. B. Orange)
- **Aktionen:**
  - Zeile klicken → Antrag bearbeiten
  - Quick-Filter-Chips: „Meine offenen Anträge", „Ausgesteuert"

### Screen 7: Antrag bearbeiten (→ UC-02)

- **Zweck:** Antragsdaten ergänzen, berechnen, prüfen und freigeben
- **Kompetenz:** `ANTRAG_BEARBEITEN` / `ANTRAG_FREIGEBEN`
- **Layout:** Analog zu Screen 5 (Angebot), aber mit zusätzlichen Aktionen
- **Inhalte:**
  - Identisch zu Screen 5, aber mit Antragsnummer und Antragsstatus
  - Zusätzlich: Referenz auf zugrunde liegendes Angebot (Link)
  - Bei Status „Ausgesteuert": Aussteuerungsgrund und Innendienst-Entscheidungsbereich
- **Aktionen-Leiste:**
  - `[Speichern]`, `[Berechnen]`, `[Prüfen]` – wie Angebot
  - `[Freigeben]` – nur bei Status „Geprüft" (→ GR-A04)
  - `[Stornieren]` – solange kein Vertragsstand erzeugt (→ A2 UC-02)
  - `[An Innendienst eskalieren]` – für manuellen Innendienst-Zugang (→ A3 UC-02)

### Screen 8: Schweben-Übersicht (Innendienst)

- **Zweck:** Zentrale Arbeitsvorratsliste für den Innendienst – alle offenen Schweben
- **Kompetenz:** `SCHWEBE_LESEN`
- **Inhalte:**
  - **Filterbereich**: Status (Offen / Erledigt / Geschlossen), Zuständiges Team, Bearbeiter, Sparte, Fälligkeitsdatum
  - **Ergebnistabelle**:
    - Spalten: Schwebenummer, Antragsnummer (Link), Kunde, Sparte, Aussteuerungsgrund, Zuständiges Team, Bearbeiter, Erstellt am, Wiedervorlage
    - Farbliche Hervorhebung: Überfällige Wiedervorlagen in Rot
    - Sortierung Standard: Älteste zuerst
- **Aktionen:**
  - Zeile klicken → Schwebe-Detail
  - Drag & Drop oder Dropdown: Schwebe einem Bearbeiter zuweisen (`SCHWEBE_BEARBEITEN`)

### Screen 9: Schwebe-Detail / Entscheidung

- **Zweck:** Ausgesteuerten Antrag prüfen und Entscheidung treffen
- **Kompetenz:** `SCHWEBE_ENTSCHEIDEN`
- **Layout:** Split-Ansicht – Antragsdaten links, Entscheidungsbereich rechts
- **Inhalte:**
  - **Linke Seite**: Vollständige Antragsdaten (readonly) inkl. Produkte, Beitrag, Prüfergebnis, spartenspezifische Daten
  - **Rechte Seite (Sidebar)**:
    - Aussteuerungsgrund (hervorgehoben)
    - Schwebe-Metadaten (Schwebenummer, Erstellt am, zuständiges Team, Bearbeiter)
    - Wiedervorlage-Datum (editierbar)
    - Entscheidungsbereich:
      - `[Freigeben]` – grün; erzeugt Vertragsstand (→ UC-02, Pfad A)
      - `[Ablehnen]` – rot; öffnet Ablehnungsgrund-Textarea (min. 10 Zeichen)
      - `[Zurückweisen]` – gelb; Antrag geht zurück an Ersteller (Status → „Offen")
- **Aktionen:**
  - Entscheidung auswählen → Bestätigungsdialog → Ausführung

### Screen 10: Vertragsübersicht

- **Zweck:** Verträge suchen, filtern und einsehen
- **Kompetenz:** `VERTRAG_LESEN`
- **Inhalte:**
  - **Suchbereich**: Vertragsnummer, Partnernummer/Name, Sparte, Status (Aktiv, Ruhend, Gekündigt, Abgelaufen)
  - **Ergebnistabelle**:
    - Spalten: Vertragsnummer, Kunde, Sparte, Status (Chip), Vertragsbeginn, Vertragsende, Jahresbeitrag, Zahlungsweise
- **Aktionen:**
  - Zeile klicken → Vertragsdetails

### Screen 11: Vertragsdetails

- **Zweck:** Vollständige Vertragseinsicht mit Versionshistorie und Vorgängen
- **Kompetenz:** `VERTRAG_LESEN`
- **Layout:** Header-Card + Tab-Navigation
- **Inhalte:**
  - **Header-Card**: Vertragsnummer, Status (Chip), Kunde (Link), Sparte, Vertragsbeginn/Ende, Jahresbeitrag
  - **Tabs**:
    - **Aktueller Stand**: Produkte mit Beiträgen, Deckungsbausteine, spartenspezifische Daten (KFZ: Fahrzeug, Tarifierung, SF-Klasse)
    - **Vertragsstände**: Tabelle aller Versionen (Version, Gültig ab/bis, Jahresbeitrag, Vorgang). Klick → Detailansicht eines Standes mit Diff zur Vorversion
    - **Vorgänge**: Tabelle aller Vorgänge (Typ, Datum, Auslöser). Links zu Antrag und Schwebe
    - **Schäden**: Schadenreferenzen aus S6 (Schadennummer, Datum, Art, Status, Regulierungsbetrag, SF-Auswirkung)
    - **SF-Historie** (KFZ): Verlauf der Schadenfreiheitsklasse als Tabelle und optionale Timeline-Visualisierung
    - **Änderungshistorie**: Revisionen aus Hibernate Envers (→ Historisierungs-API)
- **Aktionen:**
  - „Nachtrag erstellen" (→ zukünftiger UC, Kompetenz `ANTRAG_ANLEGEN`)
  - „Kündigung einleiten" (→ zukünftiger UC)
  - „Druckauftrag" (→ S5)

### Screen 12: Benachrichtigungs-Center

- **Zweck:** Zentrale Ansicht aller Benachrichtigungen des Benutzers
- **Kompetenz:** Alle angemeldeten Benutzer
- **Inhalte:**
  - Liste aller Benachrichtigungen (→ NOT-01 bis NOT-05), sortiert nach Datum (neueste zuerst)
  - Jede Benachrichtigung: Icon (nach Typ), Titel, Beschreibung, Zeitstempel, Gelesen/Ungelesen
  - Filter: Alle / Ungelesen / Typ (Schwebe, Wiedervorlage, eVB-Ablauf, Angebotsverfall, Fehler)
- **Aktionen:**
  - Klick → Navigiert zum betroffenen Objekt (Schwebe, Angebot etc.)
  - „Als gelesen markieren" (einzeln oder alle)

### Screen 13: Globale Suche

- **Zweck:** Schnelle Suche über alle Geschäftsobjekte anhand fachlicher Nummern oder Namen
- **Kompetenz:** Basierend auf den jeweiligen Lese-Kompetenzen
- **Darstellung:** Overlay/Modal, ausgelöst über Toolbar oder Tastenkürzel (Strg+K)
- **Inhalte:**
  - Suchfeld mit Autovervollständigung
  - Kategorisierte Ergebnisse: Partner, Angebote, Anträge, Verträge
  - Jedes Ergebnis zeigt: Typ-Icon, Nummer, Name/Bezeichnung, Status
- **Aktionen:**
  - Ergebnis klicken → Navigiert zur Detailansicht

---

## Bestätigungsdialoge (Modals)

> Für kritische Aktionen werden Bestätigungsdialoge (Angular Material Dialog) verwendet.

| Aktion | Dialog-Inhalt | Optionen |
|--------|---------------|----------|
| Angebot beantragen (UC-01, Schritt 19) | „Soll das Angebot nach der Überführung erhalten bleiben?" | `[Ja, behalten]` / `[Nein, löschen]` / `[Abbrechen]` |
| Angebot löschen (UC-01, A2) | „Angebot wirklich löschen? Diese Aktion kann nicht rückgängig gemacht werden." | `[Löschen]` (rot) / `[Abbrechen]` |
| Fahrzeugart ändern (UC-KFZ-00, A1) | „Alle bisherigen Eingaben gehen verloren. Fahrzeugart wirklich ändern?" | `[Ändern]` / `[Abbrechen]` |
| Antrag stornieren (UC-02, A2) | „Antrag wirklich stornieren?" | `[Stornieren]` (rot) / `[Abbrechen]` |
| Antrag ablehnen (UC-02, Pfad B) | „Bitte Ablehnungsgrund angeben:" + Textarea (min. 10 Zeichen) | `[Ablehnen]` (rot) / `[Abbrechen]` |
| Antrag zurückweisen (UC-02, Pfad B) | „Antrag wird an den Ersteller zurückgegeben." | `[Zurückweisen]` / `[Abbrechen]` |

---

## Plausibilitäts- und Prüfergebnisanzeige

> Plausibilitätsergebnisse werden einheitlich in einem **Ergebnis-Panel** dargestellt (Angular Material Expansion Panel oder Alert-Leiste).

### Schweregrade

| Severity | Icon | Farbe | Verhalten |
|----------|------|-------|-----------|
| `ERROR` | ❌ | Rot (`warn`) | Blockiert Beantragung/Freigabe; Inline am Feld + gesammelt im Ergebnis-Panel |
| `WARNING` | ⚠️ | Orange/Gelb (`accent`) | Warnung, blockiert nicht; Inline am Feld + gesammelt im Ergebnis-Panel |
| `INFO` | ℹ️ | Blau (`primary`) | Hinweis, nur im Ergebnis-Panel |

### Darstellung

```
┌─── Prüfergebnis ──────────────────────────────────────────────┐
│ ❌ 2 Fehler  ⚠️ 1 Warnung  ℹ️ 1 Hinweis                      │
├───────────────────────────────────────────────────────────────┤
│ ❌ PL-KFZ-A03: Erstzulassung darf nicht in der Zukunft liegen │
│    → Feld: Erstzulassung                          [Zum Feld] │
│ ❌ PL-KFZ-A08: FIN-Prüfziffer ist ungültig                    │
│    → Feld: FIN                                    [Zum Feld] │
│ ⚠️ PL-KFZ-A05: Fahrleistung von 45.000 km ist überdurchschn. │
│ ℹ️ PL-KFZ-L01: LVM-RabattSchutz bei SF ≥ 4 verfügbar         │
└───────────────────────────────────────────────────────────────┘
```

- Klick auf `[Zum Feld]` scrollt zum betroffenen Formularfeld und fokussiert es
- Inline-Fehlermeldungen erscheinen direkt unter dem Feld (Angular Material `mat-error`)

---

## Statusanzeige (Chips)

> Einheitliche Statusdarstellung in allen Listenansichten und Detail-Headern.

### Angebotsstatus

| Status | Farbe | Icon |
|--------|-------|------|
| Entwurf | Grau | ✏️ |
| Berechnet | Blau | 🧮 |
| Geprüft | Grün | ✅ |
| Beantragt | Dunkelgrün | 📝 |
| Gelöscht | Rot (durchgestrichen) | 🗑️ |

### Antragsstatus

| Status | Farbe | Icon |
|--------|-------|------|
| Offen | Grau | 📂 |
| Berechnet | Blau | 🧮 |
| Geprüft | Grün | ✅ |
| Freigegeben | Dunkelgrün | ✔️ |
| Ausgesteuert | Orange | ⏳ |
| Abgelehnt | Rot | ❌ |
| Storniert | Dunkelrot | 🚫 |

### Schwebe-Status

| Status | Farbe | Icon |
|--------|-------|------|
| Offen | Orange | ⏳ |
| Erledigt | Grün | ✔️ |
| Geschlossen | Grau | 🔒 |

### Vertragsstatus

| Status | Farbe | Icon |
|--------|-------|------|
| Aktiv | Grün | 📄 |
| Ruhend | Gelb | ⏸️ |
| Gekündigt | Orange | ⚠️ |
| Abgelaufen | Grau | 📁 |

---

## Komponenten & Patterns

> Wiederverwendbare UI-Komponenten auf Basis von Angular Material.

| Komponente | Angular Material Basis | Verwendung | Details |
|------------|----------------------|------------|---------|
| **Datentabelle** | `mat-table` + `mat-paginator` + `mat-sort` | Alle Listenansichten | Sortierung, Filterung, Paginierung (20/Seite Standard), Spaltenauswahl. Leerer Zustand: „Keine Einträge gefunden" mit Illustration. |
| **Suchfeld mit Autocomplete** | `mat-autocomplete` | Globale Suche, Kundensuche | Debounce 300ms, min. 3 Zeichen, Ergebnisse kategorisiert |
| **Formular mit Inline-Validierung** | `mat-form-field` + `mat-error` | Alle Erfassungsmasken | Angular Reactive Forms, Pflichtfelder mit `*`, Fehler unter dem Feld |
| **Status-Chip** | `mat-chip` | Tabellen, Detail-Header | Farbe + Icon + Text (a11y-konform) |
| **Bestätigungsdialog** | `mat-dialog` | Kritische Aktionen | Titel, Beschreibung, Primär-/Sekundäraktion, Schließen-X |
| **Toast-Benachrichtigungen** | `mat-snack-bar` | Erfolgs-/Fehlermeldungen | Position: unten rechts; Dauer: 5s (Erfolg), persistent (Fehler) |
| **Card / Panel** | `mat-card` | Dashboard-Kacheln, Detailbereiche | Schatteneffekt, Titel, Content, Actions |
| **Tab-Navigation** | `mat-tab-group` | Kundendetails, Vertragsdetails | Lazy-Loaded Tabs für Performance |
| **Expansion-Panel** | `mat-expansion-panel` | Prüfergebnisse, Berechnungsdetails | Aufklappbar, Zusammenfassung im Header |
| **Stepper** | `mat-stepper` | Angebot erstellen (optional) | Fortschrittsanzeige über die Phasen |
| **Sidebar-Navigation** | `mat-sidenav` | Hauptnavigation | Einklappbar, Icons + Text, aktiver Eintrag hervorgehoben |
| **Skeleton-Screen** | Custom (Angular Material) | Initiale Seitenladung | Platzhalter-Elemente bis Daten geladen (→ NFA-P01) |
| **Ladeindikator** | `mat-progress-bar` / `mat-spinner` | API-Aufrufe | Oben in der Seite (Progress Bar) oder zentral (Spinner) |
| **Badge** | `mat-badge` | Notification-Bell | Anzahl ungelesener Benachrichtigungen |
| **Tooltip** | `mat-tooltip` | Überall | Kontextsensitive Hilfe (→ NFA-U05); max. 2 Zeilen |
| **Leerer Zustand** | Custom | Listen ohne Ergebnisse | Illustration + Text + CTA-Button (z. B. „Noch keine Angebote. Jetzt erstellen.") |

---

## Responsiveness

> Desktop-first (Innendienst), Tablet-fähig (Außendienst). Smartphone nicht im Scope (→ NFA-U02).

| Gerät | Min. Breite | Verhalten | Zielgruppe |
|-------|------------|-----------|------------|
| Desktop | ≥ 1280 px | Volle Darstellung: Sidebar ausgeklappt, Tabellen mehrspaltig, Split-Ansichten (z. B. Schwebe-Detail) | Innendienst |
| Tablet | ≥ 768 px | Sidebar eingeklappt (Icon-Modus), Tabellen mit horizontalem Scroll oder reduziertem Spaltenset, Dialoge vollflächig, kein Split-View | Außendienst (iPad) |
| Mobil | < 768 px | **Nicht unterstützt** – Redirect/Hinweis auf Kundenportal oder Desktop-Nutzung | Kunden nutzen Kundenportal (S7) |

### Breakpoints (Angular Material)

| Breakpoint | Wert | CSS-Klasse |
|-----------|------|-----------|
| Handset | < 600 px | Nicht unterstützt |
| Tablet Portrait | 600–839 px | Reduziertes Layout |
| Tablet Landscape | 840–1279 px | Standard-Layout ohne Sidebar |
| Desktop | ≥ 1280 px | Volles Layout |

### Layout-Regeln

| Element | Desktop (≥ 1280) | Tablet (≥ 768) |
|---------|-----------------|----------------|
| Sidebar | Ausgeklappt (240 px) | Eingeklappt (Icons, 60 px) |
| Formularfelder | 2–3 Spalten Grid | 1–2 Spalten Grid |
| Tabellen | Alle Spalten sichtbar | Priorisierte Spalten, Rest im Detail |
| Dialoge | Zentriert (max. 600 px breit) | Vollflächig (bottom sheet) |
| Kacheln (Fahrzeugart) | 3 pro Reihe | 2 pro Reihe |
| Split-Ansicht (Schwebe) | 60% / 40% | Gestapelt (volle Breite) |

---

## Barrierefreiheit (a11y)

> Konformität mit **WCAG 2.1 Level AA** (→ NFA-U01). Angular Material liefert ARIA-Unterstützung von Haus aus – diese wird konsequent genutzt und bei Custom-Komponenten ergänzt.

| Nr. | Anforderung | Umsetzung |
|-----|-------------|-----------|
| A11Y-01 | **Tastaturnavigation** | Alle Kernfunktionen per Tastatur bedienbar (Tab, Enter, Escape, Pfeiltasten). Fokus-Reihenfolge logisch. Sichtbarer Fokus-Ring (`:focus-visible`). Skip-to-Content-Link. |
| A11Y-02 | **Kontrastverhältnisse** | Text: ≥ 4,5:1. Große Texte (≥ 18pt): ≥ 3:1. UI-Elemente und Grafiken: ≥ 3:1. Validierung über automatisierte Tools (Lighthouse, axe). |
| A11Y-03 | **ARIA-Labels** | Alle interaktiven Elemente haben `aria-label` oder `aria-labelledby`. Dynamische Inhalte nutzen `aria-live` Regions (z. B. Plausibilitätsfehler, Toast-Meldungen). |
| A11Y-04 | **Screenreader** | Formulare: `mat-label` + `mat-error` korrekt verknüpft. Tabellen: `mat-header-cell` semantisch korrekt. Status-Chips: Farbe wird ergänzt durch Icon + Text (nicht nur Farbe). Dialoge: Fokus-Trap + `aria-modal`. |
| A11Y-05 | **Fehlermeldungen** | Fehlermeldungen werden sofort im Screenreader vorgelesen (`aria-live="assertive"`). Verknüpfung Feld → Fehler über `aria-describedby`. |
| A11Y-06 | **Bilder und Icons** | Dekorative Icons: `aria-hidden="true"`. Funktionale Icons: `aria-label` mit Beschreibung. Keine Information nur durch Farbe vermittelt. |
| A11Y-07 | **Zoom und Skalierung** | Seite bleibt bis 200% Zoom funktional ohne horizontalen Scroll (Desktop). Keine fixen Pixelwerte für Schriftgrößen (rem-basiert). |
| A11Y-08 | **Automatisierte Tests** | axe-core Integration in CI/CD-Pipeline. Lighthouse Accessibility Score ≥ 90 als Quality Gate. |

---

## Tastaturkürzel

| Kürzel | Aktion | Kontext |
|--------|--------|---------|
| `Strg + K` | Globale Suche öffnen | Überall |
| `Strg + S` | Speichern | Formular-Screens |
| `Strg + N` | Neues Angebot erstellen | Überall |
| `Escape` | Dialog/Overlay schließen | Dialoge, Suche |
| `Tab` / `Shift+Tab` | Zum nächsten/vorherigen Feld | Formulare |
| `Enter` | Primäraktion des fokussierten Elements | Überall |

---

## Fehlerbehandlung (UI)

| Fehlertyp | Darstellung | Beispiel |
|-----------|-------------|---------|
| **Inline-Validierungsfehler** | `mat-error` unter dem Feld (rot) | „Vertragsbeginn muss in der Zukunft liegen." |
| **Plausibilitätsfehler** | Ergebnis-Panel + Inline | → Plausibilitäts- und Prüfergebnisanzeige (oben) |
| **HTTP 403 Forbidden** | Toast (Warnung) + Button deaktiviert | „Keine Berechtigung für diese Aktion." |
| **HTTP 404 Not Found** | Fehlerseite mit Illustration | „Das gesuchte Angebot wurde nicht gefunden." + Link zurück |
| **HTTP 409 Conflict** | Toast (Fehler) | „Statusübergang nicht erlaubt: Angebot ist nicht im Status ‚Geprüft'." |
| **HTTP 422 Unprocessable** | Ergebnis-Panel mit Fehlerdetails | Plausibilitätsfehler aus Backend |
| **HTTP 500 Server Error** | Toast (persistent, Fehler) | „Unerwarteter Fehler. Bitte versuchen Sie es erneut oder kontaktieren Sie den Support." + Trace-ID |
| **HTTP 502/503** | Banner oben (persistent) | „Ein externes System ist derzeit nicht erreichbar. Einige Funktionen sind eingeschränkt." |
| **Netzwerkfehler** | Banner oben | „Keine Verbindung zum Server. Bitte prüfen Sie Ihre Internetverbindung." |

---

## Offene Fragen

| Nr. | Frage | Kontext |
|-----|-------|---------|
| UI-01 | Soll ein Dark Mode angeboten werden? | Design-Prinzipien |
| UI-02 | Wird eine Favoriten- oder Lesezeichen-Funktion für häufig genutzte Verträge/Kunden benötigt? | Navigation |
| UI-03 | Soll die Fahrzeugartauswahl (KFZ) die häufigsten Fahrzeugarten hervorheben (z. B. PKW größer)? | UC-KFZ-00 |
| UI-04 | Wird eine Drag-&-Drop-Funktion für die Schwebe-Zuweisung an Bearbeiter benötigt (Kanban-Board)? | Schweben-Übersicht |
| UI-05 | Soll ein Dashboard für Außendienst und Innendienst unterschiedlich konfiguriert sein? | Dashboard |
| UI-06 | Wird eine kontextsensitive Online-Hilfe (Help-Center/Sidebar) benötigt oder reichen Tooltips? | NFA-U05 |
| UI-07 | Sollen Tabellenspalten vom Benutzer individuell ein-/ausblendbar und umsortierbar sein? | Usability | 
