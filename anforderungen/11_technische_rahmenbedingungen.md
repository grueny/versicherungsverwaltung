# Technische Rahmenbedingungen – Versicherungsverwaltung

## Technologie-Stack

### Frontend
- **Framework:** 
- **Sprache:** 
- **CSS/Styling:** 
- **State Management:** 

### Backend
- **Framework:** 
- **Sprache:** 
- **API-Stil:** 

### Datenbank
- **Typ:** 
- **System:** 
- **ORM/Query Builder:** 

### Authentifizierung
- **Methode:** 
- **Provider:** 

## Architektur
<!-- Monolith, Microservices, Serverless etc. -->
- **Architekturstil:** 
- **Begründung:** 

## Spartenkonfiguration – Architekturentscheidung

> **Entscheidung:** Hybrid-Ansatz – jeder Konfigurationsaspekt wird mit der am besten geeigneten Technik umgesetzt, gebündelt über eine zentrale Sparten-Registry.

### Übersicht

```
┌─────────────────────────────────────────────────┐
│              Generischer Kernprozess             │
│  Anbahnung → Antrag → Policierung → Wartung → … │
│       ▼            ▼           ▼          ▼      │
│    [Hook]       [Hook]      [Hook]     [Hook]    │
└──────┬────────────┬───────────┬──────────┬───────┘
       │            │           │          │
┌──────▼────────────▼───────────▼──────────▼───────┐
│            Sparten-Registry                       │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐          │
│  │  KFZ    │  │  Sach   │  │  Leben  │   ...    │
│  │─────────│  │─────────│  │─────────│          │
│  │Produkte │  │Produkte │  │Produkte │  ← DB    │
│  │Regeln   │  │Regeln   │  │Regeln   │  ← Engine│
│  │Hooks    │  │Hooks    │  │Hooks    │  ← Code  │
│  └─────────┘  └─────────┘  └─────────┘          │
└──────────────────────────────────────────────────┘
```

### Konfigurationsaspekte im Detail

| Aspekt | Technik | Änderbarkeit | Begründung |
|--------|---------|-------------|------------|
| **Produkte** (Merkmale, Tarife, Deckungsbausteine) | Datenbank-Konfiguration | Ohne Deployment (Laufzeit) | Ändern sich häufig; Fachbereich soll Tarife/Deckungsbausteine selbst pflegen können |
| **Plausibilitäten** (Validierungen, Prüfregeln, Geschäftsregeln) | Regelengine / Strategy-Pattern | Regelengine: ohne Deployment; Strategy: mit Deployment | Fachlich komplex, aber klar abgrenzbar; Regeln je Sparte registrierbar und testbar |
| **Prozessabweichungen** (spartenspezifische Schritte) | Extension Points / Hooks im Kernprozess | Mit Deployment | Zu komplex für reine Konfiguration; Kernprozess definiert Erweiterungspunkte (`vor_Policierung`, `nach_Antrag` etc.), Sparten registrieren Handler |
| **Spartenkonfiguration gesamt** | Sparten-Registry-Pattern | Mit Deployment (neue Sparte) | Zentrale Registratur, die Produkte, Regeln und Hooks pro Sparte bündelt |

### Kernprozess – Extension Points (Hooks)

Der generische Kernprozess definiert feste Erweiterungspunkte, an denen sich Sparten einklinken können:

| Hook-Zeitpunkt | Beschreibung | Beispiel KFZ |
|----------------|-------------|-------------|
| `vor_Angebotserstellung` | Spartenspezifische Vorabprüfungen | Fahrzeugdaten erfassen, Typklasse ermitteln |
| `nach_Antragserstellung` | Zusätzliche Schritte nach Antrag | eVB-Nummer erzeugen |
| `vor_Policierung` | Prüfungen vor Vertragsabschluss | SF-Klasse validieren, Pflichtversicherung prüfen |
| `nach_Policierung` | Aktionen nach Vertragsabschluss | eVB-Nummer an Zulassungsstelle melden |
| `vor_Nachtrag` | Prüfungen vor Vertragsänderung | Fahrzeugwechsel-Plausibilitäten |
| `nach_Nachtrag` | Aktionen nach Vertragsänderung | Neue eVB bei Fahrzeugwechsel |
| `vor_Kuendigung` | Prüfungen vor Beendigung | Pflichtversicherung: Folgeversicherung prüfen |
| `nach_Kuendigung` | Aktionen nach Beendigung | eVB-Stornierung, Ruheversicherung anbieten |

### Sparten-Registry

Die Sparten-Registry ist die zentrale Komponente, die alle Konfigurationsaspekte einer Sparte zusammenführt:

```
SpartenRegistry
├── registriere(spartenId, konfiguration)
├── getProdukte(spartenId) → Produkt[]          ← aus DB
├── getRegeln(spartenId) → Regel[]              ← aus Regelengine
├── getHooks(spartenId, hookId) → Handler[]     ← aus Code
└── getSpartenkonfiguration(spartenId) → Konfig ← gebündelt
```

### Vorteile dieses Ansatzes

- **Produkte in DB** → Fachbereich kann Tarife/Deckungsbausteine pflegen, ohne Release-Zyklus
- **Plausibilitäten als Regeln** → Fachlich lesbar, änderbar, aber trotzdem testbar
- **Prozessabweichungen als Code** → Zu komplex für reine Konfiguration, müssen zuverlässig getestet werden
- **Registry** → Saubere Registrierung pro Sparte, Kernprozess kennt keine Sparten-Details
- **Neue Sparte hinzufügen** → Produkte in DB anlegen + Regeln registrieren + Hooks implementieren + in Registry eintragen

## Deployment & Infrastruktur
- **Hosting:** 
- **Container:** 
- **CI/CD:** 
- **Environments:** Development | Staging | Production

## Entwicklungsstandards
- **Versionierung:** Git
- **Branching-Strategie:** 
- **Code-Style:** 
- **Testing-Strategie:** 
  - Unit-Tests: 
  - Integration-Tests: 
  - E2E-Tests: 

## Monitoring & Logging
- **Logging:** 
- **Monitoring:** 
- **Error Tracking:** 

## Sonstiges
- **Zeitzone:** 
- **Währung:** 
- **Zeichenkodierung:** UTF-8
- **Datumsformat:** 

## Offene Entscheidungen
<!-- Technische Entscheidungen, die noch getroffen werden müssen -->
- [x] Spartenkonfiguration: Hybrid-Ansatz (DB + Regelengine + Hooks + Registry) → siehe oben
- [ ] Konkrete Regelengine auswählen (z. B. Drools, Easy Rules, eigene Implementierung)
- [ ] Hook-Mechanismus: Framework-spezifische Events vs. eigenes Observer-Pattern
- [ ] Administrations-UI für Produkt-/Tarifpflege durch Fachbereich
