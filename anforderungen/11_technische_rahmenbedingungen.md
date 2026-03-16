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
- **Testing-Strategie:** → siehe Abschnitt „Teststrategie" weiter unten

## Teststrategie – Kern vs. Sparten

> **Ziel:** Änderungen am spartenübergreifenden Kern dürfen keine unkontrollierten Testaufwände in den Sparten verursachen. Breaking Changes müssen erkannt werden, bevor sie die Sparten erreichen.

### Testpyramide

```
         ╱  E2E-Tests (wenige)  ╲         ← Pro Sparte: nur kritische Pfade
        ╱   je Sparte ~5-10      ╲
       ╱───────────────────────────╲
      ╱  Integrationstests          ╲     ← Contract-Tests Kern ↔ Sparte
     ╱   Kern ↔ Hook-Schnittstelle   ╲
    ╱─────────────────────────────────╲
   ╱  Unit-Tests (viele)               ╲  ← Kern und Sparte GETRENNT testbar
  ╱   Kern: ~80% | Sparte: ~80%        ╲
 ╱───────────────────────────────────────╲
```

| Ebene | Was wird getestet | Verantwortung | Auslöser |
|-------|-------------------|---------------|----------|
| **Unit-Tests Kern** | Generischer Prozess mit Mock-Hooks | Kern-Team | Jeder Kern-Commit |
| **Unit-Tests Sparte** | Hooks/Regeln mit Mock-Kerndaten | Sparten-Team | Jeder Sparten-Commit |
| **Contract-Tests** | Schnittstelle Kern ↔ Sparte (Consumer-Driven) | Kern-Team führt aus, Sparte definiert Contracts | Jeder Kern-Commit |
| **Sparten-Regressionstests** | Generisches Testkit pro Sparte (Happy Paths) | Sparte schreibt einmal, Kern führt aus | Jeder Kern-Commit |
| **E2E-Tests** | Kritische End-to-End-Pfade je Sparte | Sparten-Team | Release-Branch |

### Generisches Sparten-Testkit

Jede Sparte stellt ein standardisiertes Testkit bereit, das bei jedem Kern-Build automatisch ausgeführt wird:

| Testfall | Prüft | Automatisiert |
|----------|-------|---------------|
| Vertrag anlegen (Happy Path) | Kern + Hooks | ✅ |
| Vertrag ändern (Nachtrag) | Kern + Hooks | ✅ |
| Vertrag kündigen | Kern + Hooks | ✅ |
| Plausibilitäten greifen | Regelengine | ✅ |
| Produktkombinationen gültig | DB-Konfiguration | ✅ |

### CI/CD-Pipeline bei Kern-Änderungen

```
Kern-Commit
    │
    ├── 1. Kern-Unit-Tests              ← Schnell (~2 Min)
    ├── 2. Consumer-Contract-Tests      ← Schnell (~1 Min)
    │      "Liefert der Kern noch alles, was jede Sparte braucht?"
    ├── 3. Sparten-Regressionstests     ← Parallel pro Sparte (~5 Min)
    │      ├── KFZ-Testkit
    │      ├── Sach-Testkit
    │      └── Leben-Testkit
    └── 4. E2E nur bei Release-Branch   ← Nur wenn nötig (~20 Min)
```

## Schutz vor Breaking Changes – Architekturentscheidung

> **Problem:** Änderungen am Kernprozess können Hook-Interfaces brechen und damit alle Sparten gleichzeitig betreffen.  
> **Entscheidung:** Drei gestaffelte Schutzschichten verhindern, dass Breaking Changes unkontrolliert die Sparten erreichen.

### Schicht 1: Consumer-Driven Contracts

Jede Sparte definiert, was sie vom Kern **braucht** (nicht was der Kern liefert):

```
Sparte KFZ definiert:
  "Ich brauche von vorPolicierung:
   - vertrag.sparte == 'KFZ'
   - vertrag.produkte nicht leer
   - kunde.geburtsdatum vorhanden"

Sparte Sach definiert:
  "Ich brauche von vorPolicierung:
   - vertrag.sparte == 'SACH'
   - vertrag.versicherungsort vorhanden"
```

Der Kern sammelt alle Consumer-Contracts und prüft sie bei jedem Build:

```
Kern-CI:
  ├── Consumer-Contract KFZ:   "Liefere ich noch alles?" ✅
  ├── Consumer-Contract Sach:  "Liefere ich noch alles?" ✅
  └── Consumer-Contract Leben: "Liefere ich noch alles?" ❌ BRICHT
      → Kern-Änderung wird BLOCKIERT, bevor sie ins Release kommt
```

→ Breaking Changes werden **im Kern erkannt, bevor sie die Sparten erreichen**.

### Schicht 2: Parallelbetrieb alter und neuer Interfaces

Bei bewussten Breaking Changes laufen altes und neues Interface parallel:

```
Release N:     HookV1 ✅ aktiv       HookV2 ✅ aktiv (neu)
Release N+1:   HookV1 ⚠️ deprecated  HookV2 ✅ aktiv
Release N+2:   HookV1 ❌ entfernt    HookV2 ✅ aktiv
```

- Altes Interface läuft mindestens **1 Release** parallel weiter (Grace Period)
- Sparten migrieren im eigenen Tempo innerhalb der Grace Period
- Sparten-Testkits laufen gegen **ihr** Interface – solange es lebt, brechen sie nicht

### Schicht 3: Adapter-Schicht pro Sparte

Sparten implementieren Hooks nicht direkt, sondern über einen Adapter:

```
Kern (v2)                Adapter                 Sparte KFZ
┌──────────┐     ┌─────────────────┐     ┌──────────────┐
│ HookAPI  │────►│ KfzHookAdapter  │────►│ KFZ-Logik    │
│ v2       │     │ übersetzt v2→v1 │     │ kennt nur v1 │
└──────────┘     └─────────────────┘     └──────────────┘
```

- Bei einem Breaking Change muss nur der **Adapter** angepasst werden (1 Klasse pro Sparte)
- Die gesamte Sparten-Logik + alle Sparten-Tests bleiben unverändert
- Das **Kern-Team liefert bei jedem Breaking Change die Adapter-Migration mit**
- Die Sparte entscheidet selbst, wann sie den Adapter aktualisiert oder direkt auf v2 geht

### Hook-Parameter: Erweiterbare Kontextobjekte

Um Breaking Changes an Hook-Signaturen zu vermeiden, werden Parameter als erweiterbare Kontextobjekte übergeben:

```
// NICHT: Einzelne Parameter (Änderung = Breaking Change)
void vorPolicierung(Vertrag v, Kunde k);

// RICHTIG: Erweiterbares Kontextobjekt
void vorPolicierung(PolicierungKontext ctx);

PolicierungKontext {
    Vertrag vertrag;          // bestehend
    Kunde kunde;              // bestehend
    @Optional
    ProduktAuswahl auswahl;   // NEU – optional, bricht nichts
}
```

→ Neue Felder sind optional → bestehende Sparten-Implementierungen bleiben kompatibel.

### Zusammenfassung: Schutzschichten

```
                    ┌─────────────────────────┐
                    │  Consumer-Driven         │  ← Erkennt Breaking Changes
                    │  Contracts im Kern-Build │     VOR dem Release
                    └────────────┬────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │  Parallelbetrieb v1+v2  │  ← Gibt Sparten Zeit
                    │  (Grace Period)          │     zur Migration
                    └────────────┬────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │  Adapter-Schicht         │  ← Fängt den Rest ab,
                    │  pro Sparte              │     Sparten-Logik stabil
                    └─────────────────────────┘
```

| Schicht | Schützt vor | Verantwortung |
|---------|-------------|---------------|
| Consumer-Contracts | Unbeabsichtigte Breaking Changes | Kern-Team (CI blockiert) |
| Parallelbetrieb v1+v2 | Zeitdruck bei Migration | Kern-Team (Grace Period einhalten) |
| Adapter-Schicht | Bewusste Breaking Changes | Kern-Team liefert Adapter, Sparte übernimmt wenn bereit |

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
- [x] Teststrategie: Testpyramide mit Consumer-Driven Contracts + generischem Sparten-Testkit → siehe oben
- [x] Schutz vor Breaking Changes: 3-Schichten-Modell (Contracts + Parallelbetrieb + Adapter) → siehe oben
- [x] Hook-Parameter: Erweiterbare Kontextobjekte statt einzelner Parameter → siehe oben
- [ ] Konkrete Regelengine auswählen (z. B. Drools, Easy Rules, eigene Implementierung)
- [ ] Hook-Mechanismus: Framework-spezifische Events vs. eigenes Observer-Pattern
- [ ] Administrations-UI für Produkt-/Tarifpflege durch Fachbereich
- [ ] Contract-Test-Framework auswählen (z. B. Pact, Spring Cloud Contract, eigene Implementierung)
- [ ] Grace Period für deprecated Interfaces festlegen (Anzahl Releases)
