# Technische Rahmenbedingungen – Versicherungsverwaltung

## Technologie-Stack

### Frontend
- **Framework:** Angular 19
- **Sprache:** TypeScript
- **CSS/Styling:** Angular Material (Material Design Komponenten)
- **State Management:** Angular Signals (ngrx/signals)
- **Forms:** Angular Reactive Forms (eingebaute Validierung → ideal für Plausibilitäten in Echtzeit)
- **Build:** Angular CLI
- **API-Client:** Automatisch generiert aus OpenAPI 3.1 Spec

### Backend
- **Framework:** Spring Boot 3 (Spring 6)
- **Sprache:** Java 21 (LTS)
- **API-Stil:** REST + OpenAPI 3.1 (API-First-Entwicklung)
- **Regelengine:** Drools (KIE) – für spartenspezifische Plausibilitäten und Geschäftsregeln
- **Hook-Mechanismus:** Spring Events (ApplicationEventPublisher) – für Extension Points / Hooks
- **Modularisierung:** Spring Modulith – modularer Monolith, ein Modul pro Sparte
- **Audit/Historisierung:** Hibernate Envers – revisionssichere Historisierung aller Entitäten
- **Security:** Spring Security – Integration mit Kompetenz-System (S8)

### Datenbank
- **Typ:** Relational (RDBMS)
- **System:** PostgreSQL 17
- **ORM/Query Builder:** Spring Data JPA (Hibernate 6)
- **Historisierung:** Temporal Tables (SQL:2011, system-versioned) + Hibernate Envers
- **Flexible Spartendaten:** JSONB-Spalten für spartenspezifische Attribute
- **Skalierung:** Table Partitioning nach Sparte/Jahr für große Bestände

### Authentifizierung
- **Methode:** Token-basiert (JWT / OAuth2)
- **Provider:** Externes Kompetenz-System (S8) – Kompetenz-ID-Abfrage pro Aktion
- **Integration:** Spring Security Filter → Kompetenz-System-Abfrage bei geschützten Endpunkten

## Architektur

- **Architekturstil:** Modularer Monolith (Spring Modulith)
- **Begründung:** Für den Start kein Microservice-Overhead nötig. Spring Modulith erzwingt saubere Modulgrenzen (Kern + Sparten-Module) und erlaubt einen späteren Split in Microservices, falls nötig. Jede Sparte ist ein eigenständiges Modul mit definierten Interfaces zum Kern.

### Gesamtarchitektur

```
┌─────────────────────────────────────────────────────┐
│                    Frontend                          │
│              Angular 19 + Material                   │
│      Reactive Forms + Signals + Lazy Loading         │
└──────────────────────┬──────────────────────────────┘
                       │ REST / OpenAPI 3.1 (JSON)
┌──────────────────────▼──────────────────────────────┐
│           Backend (Spring Boot 3 / Java 21)          │
│  ┌─────────────┐  ┌──────────┐  ┌───────────────┐  │
│  │ Kernprozess  │  │  Drools  │  │ Spring Events │  │
│  │  (Services)  │  │ (Regeln) │  │   (Hooks)     │  │
│  └──────┬──────┘  └────┬─────┘  └──────┬────────┘  │
│         │              │               │            │
│  ┌──────▼──────────────▼───────────────▼────────┐  │
│  │         Sparten-Registry (Spring Modulith)    │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐   │  │
│  │  │ Modul:   │  │ Modul:   │  │ Modul:   │   │  │
│  │  │   KFZ    │  │   Sach   │  │  Leben   │   │  │
│  │  └──────────┘  └──────────┘  └──────────┘   │  │
│  └──────────────────────┬───────────────────────┘  │
│                         │                           │
│  ┌──────────────────────▼───────────────────────┐  │
│  │    Spring Data JPA (Hibernate 6 + Envers)     │  │
│  └──────────────────────┬───────────────────────┘  │
└─────────────────────────┼───────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────┐
│              PostgreSQL 17                           │
│   Temporal Tables │ JSONB │ Partitioning │ Envers    │
└─────────────────────────────────────────────────────┘
```

### Technologie-Zuordnung zu Architekturkonzepten

| Architekturkonzept | Technologie | Begründung |
|--------------------|-------------|------------|
| Sparten-Registry | Spring Modulith Module | Ein Modul pro Sparte, klare Grenzen, DI-basierte Registrierung |
| Extension Points / Hooks | Spring ApplicationEvents | Entkoppelt, asynchron möglich, testbar mit @EventListener |
| Regelengine (Plausibilitäten) | Drools (KIE) | DRL-Regeln fachlich lesbar, Hot-Deployment ohne Restart |
| Produktkonfiguration (DB) | Spring Data JPA + JSONB | Fachbereich pflegt Tarife/Produkte ohne Deployment |
| Revisionssichere Historisierung | Hibernate Envers + Temporal Tables | Lückenlose Änderungshistorie mit Benutzer + Zeitstempel |
| Consumer-Driven Contracts | Spring Cloud Contract | Contract-Tests im Kern-Build, Stubs für Sparten-Tests |
| Kompetenz-Integration | Spring Security + RestTemplate | Kompetenz-Abfrage per REST an externes System (S8) |
| API-Dokumentation | springdoc-openapi (OpenAPI 3.1) | Automatische Spec-Generierung, Client-Codegen für Angular |
| Formular-Validierung (Frontend) | Angular Reactive Forms | Synchrone + asynchrone Validatoren, 1:1 Mapping zu Plausibilitäten |

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

### KFZ-Sparte: Sub-Modul-Architektur

> **Entscheidung:** Die KFZ-Sparte wird in vier fachliche Sub-Module aufgeteilt, um die hohe Komplexität (eVB-Verwaltung, SFR/VWB-Verfahren, Versicherungskennzeichen) beherrschbar zu halten. Alle Sub-Module bleiben innerhalb des KFZ Spring-Modulith-Moduls.

```
kfz/
├── kern/           ← Fahrzeug, Tarifierung, KFZ-spezifische Hooks
│   ├── FahrzeugService
│   ├── KfzTarifierungService
│   ├── KfzHookHandler          (vor_Angebotserstellung, vor_Policierung etc.)
│   └── KfzProduktValidator
│
├── evb/            ← eVB-Erzeugung, GDV-Meldung, Stornierung, Ablauf
│   ├── EvbService              (Erzeugung, Ablauf, Stornierung)
│   ├── EvbGdvClient            (GDV REST API: Meldung/Storno)
│   ├── EvbAblaufScheduler      (6-Monats-Timer, NOT-04)
│   └── EvbHookHandler          (nach_Antragserstellung → eVB erzeugen)
│
├── sfr/            ← SF-Klassen, VWB-Verfahren, Rückstufung, Hochstufung
│   ├── SfKlassenService        (Berechnung, Hoch-/Rückstufung)
│   ├── SfRueckstufungEngine    (Tabelle anwenden, Rabattschutz prüfen)
│   ├── VwbService              (VWB-Verfahren: Ein-/Ausgang)
│   ├── VwbGdvClient            (GDV REST API: SF-Anfrage/Meldung)
│   └── VwbHookHandler          (vor_Policierung → SF validieren/VWB prüfen)
│
└── vkz/            ← Versicherungskennzeichen-Verwaltung
    ├── VkzBestandService       (Kontingente, Lager, Bestellung)
    ├── VkzZuweisungService     (Zuweisung an Vertrag, Rücknahme)
    ├── VkzSaisonService        (01.03.–Ende Feb. Zyklusverwaltung)
    └── VkzHookHandler          (nach_Policierung → VKZ zuweisen)
```

#### Begründung

| Kriterium | kern | evb | sfr | vkz |
|-----------|------|-----|-----|-----|
| **Eigener Lebenszyklus** | Fahrzeugdaten ändern sich selten | eVB: ERZEUGT→GEMELDET→VERWENDET→STORNIERT | SF: jährliche Hochstufung, Rückstufung bei Schaden; VWB: eigener Nachrichtenaustausch | VKZ: Saison 01.03.–28.02., jährliche Erneuerung |
| **Eigene GDV-Schnittstelle** | Typklassen/Regionalklassen | eVB-Meldung/Storno | VWB-Nachrichten (SF-Anfrage/Auskunft) | – (interne Verwaltung) |
| **Fachliche Komplexität** | Mittel | Mittel (Fristen, GDV-Meldung) | Hoch (Tabellen, Rabattschutz, VWB-Prozess) | Mittel (Kontingente, Saisonlogik) |
| **Getrennt testbar** | ✅ | ✅ | ✅ | ✅ |

#### Spring-Modulith-Packages

```
de.versicherungsverwaltung.kfz.kern.*      → @ApplicationModule
de.versicherungsverwaltung.kfz.evb.*       → @ApplicationModule
de.versicherungsverwaltung.kfz.sfr.*       → @ApplicationModule
de.versicherungsverwaltung.kfz.vkz.*       → @ApplicationModule
```

Die Sub-Module kommunizieren untereinander über **Spring Events** (z. B. `SfRueckstufungEvent` aus sfr → evb prüft eVB-Status). Direkte Methoden-Aufrufe erfolgen nur über definierte **Internal APIs** innerhalb des KFZ-Moduls.

---

### Vorteile dieses Ansatzes

- **Produkte in DB** → Fachbereich kann Tarife/Deckungsbausteine pflegen, ohne Release-Zyklus
- **Plausibilitäten als Regeln** → Fachlich lesbar, änderbar, aber trotzdem testbar
- **Prozessabweichungen als Code** → Zu komplex für reine Konfiguration, müssen zuverlässig getestet werden
- **Registry** → Saubere Registrierung pro Sparte, Kernprozess kennt keine Sparten-Details
- **Neue Sparte hinzufügen** → Produkte in DB anlegen + Regeln registrieren + Hooks implementieren + in Registry eintragen

## Deployment & Infrastruktur
- **Hosting:** Noch zu entscheiden (On-Premise / Cloud / Hybrid)
- **Container:** Docker (Dockerfile für Backend + Frontend), Docker Compose für lokale Entwicklung
- **CI/CD:** Noch zu entscheiden (Jenkins / GitLab CI / GitHub Actions)
- **Build-Tools:** Maven (Backend), Angular CLI / npm (Frontend)
- **Environments:** Development | Staging | Production
- **Artifact:** Spring Boot Fat-JAR (Backend), nginx-served Static Files (Frontend)

## Entwicklungsstandards
- **Versionierung:** Git
- **Branching-Strategie:** Git Flow (main, develop, feature/*, release/*, hotfix/*)
- **Code-Style:** Google Java Style Guide + Checkstyle, Prettier + ESLint (Angular)
- **Testing-Strategie:** → siehe Abschnitt „Teststrategie" weiter unten
- **Code Reviews:** Pull/Merge Requests mit mindestens 1 Reviewer
- **Dokumentation:** JavaDoc (öffentliche APIs), OpenAPI Spec (REST), ADRs (Architecture Decision Records)

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
- **Logging:** SLF4J + Logback (Spring Boot Default), strukturiertes JSON-Logging für Produktion
- **Monitoring:** Spring Boot Actuator (Health, Metrics, Info) + Micrometer (Prometheus-kompatibel)
- **Error Tracking:** Zentrale Exception-Handler (@ControllerAdvice), Correlation-IDs pro Request
- **Tracing:** Micrometer Tracing (ehemals Spring Cloud Sleuth) – Request-Tracing über Module hinweg

## Sonstiges
- **Zeitzone:** Europe/Berlin (CET/CEST)
- **Währung:** EUR (€), Beträge in Cent (long) oder BigDecimal mit 2 Nachkommastellen
- **Zeichenkodierung:** UTF-8
- **Datumsformat:** dd.MM.yyyy (Anzeige), ISO 8601 / yyyy-MM-dd (API/DB)
- **Locale:** de_DE
- **Versionierung API:** URL-basiert (/api/v1/, /api/v2/)

## Offene Entscheidungen
<!-- Technische Entscheidungen, die noch getroffen werden müssen -->
- [x] Spartenkonfiguration: Hybrid-Ansatz (DB + Regelengine + Hooks + Registry) → siehe oben
- [x] Teststrategie: Testpyramide mit Consumer-Driven Contracts + generischem Sparten-Testkit → siehe oben
- [x] Schutz vor Breaking Changes: 3-Schichten-Modell (Contracts + Parallelbetrieb + Adapter) → siehe oben
- [x] Hook-Parameter: Erweiterbare Kontextobjekte statt einzelner Parameter → siehe oben
- [x] Technologie-Stack: Java 21 + Spring Boot 3 + Angular 19 + PostgreSQL 17 → siehe Technologie-Stack
- [x] Regelengine: Drools (KIE) → fachlich lesbare DRL-Regeln, Hot-Deployment
- [x] Hook-Mechanismus: Spring ApplicationEvents (ApplicationEventPublisher) → entkoppelt, asynchron möglich
- [x] Contract-Test-Framework: Spring Cloud Contract → nahtlose Spring-Boot-Integration, Stub-Generierung
- [ ] Administrations-UI für Produkt-/Tarifpflege durch Fachbereich
- [ ] Grace Period für deprecated Interfaces festlegen (Anzahl Releases)
