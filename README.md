# Versicherungsverwaltung

> Spartenübergreifendes Bestandsführungssystem zur Verwaltung von Versicherungspolicen über den gesamten Vertragslebenszyklus.

## Vision

Ein zentrales System für Anbahnung, Vertragsabschluss, Bestandsführung, Vertragswartung und Vertragsbeendigung – mit revisionssicherer Historisierung aller Änderungen. Das System verfolgt einen **spartenübergreifend einheitlichen Ansatz**: Unterschiedliche Versicherungssparten durchlaufen die gleichen Prozesse, ergänzt durch eine **Spartenkonfiguration** (Produkte, Prozesse, Plausibilitäten).

**Initiale Sparte:** KFZ (basierend auf öffentlichen Daten der LVM Versicherung)

## Anforderungsdokumentation

Die Anforderungen sind im Ordner [`anforderungen/`](anforderungen/) strukturiert abgelegt – optimiert für die Verarbeitung durch einen AI-Agenten.

### Dokumentenstruktur

| Nr. | Dokument | Beschreibung | Status |
|-----|----------|-------------|--------|
| 01 | [Projektvision](anforderungen/01_projektvision.md) | Vision, Ziele, Scope, Rahmenbedingungen | ✏️ Teilweise |
| 02 | [Glossar](anforderungen/02_glossar.md) | Einheitliche Begriffsdefinitionen | ✏️ Teilweise |
| 03 | [Stakeholder](anforderungen/03_stakeholder.md) | Rollen und Verantwortlichkeiten | 🔲 Offen |
| 04 | [Kontextabgrenzung](anforderungen/04_kontextabgrenzung.md) | Systemgrenzen, Schnittstellen, Kontextdiagramm | ✅ Erfasst |
| 05 | [Funktionale Anforderungen](anforderungen/05_funktionale_anforderungen/) | Spartenübergreifende Use Cases und Epics | ✏️ Teilweise |
| 06 | [Nichtfunktionale Anforderungen](anforderungen/06_nichtfunktionale_anforderungen.md) | Performance, Sicherheit, Verfügbarkeit etc. | 🔲 Offen |
| 07 | [Datenmodell](anforderungen/07_datenmodell.md) | Entitäten, Relationen, Enumerationen | ✅ Vollständig |
| 08 | [Geschäftsregeln](anforderungen/08_geschaeftsregeln.md) | Spartenübergreifende Geschäftsregeln | 🔲 Offen |
| 09 | [Schnittstellen](anforderungen/09_schnittstellen.md) | Technische Schnittstellenspezifikationen | ✅ Vollständig |
| 10 | [UI-Anforderungen](anforderungen/10_ui_anforderungen.md) | Navigation, Layouts, Interaktionsmuster | 🔲 Offen |
| 11 | [Technische Rahmenbedingungen](anforderungen/11_technische_rahmenbedingungen.md) | Architektur, Stack, Spartenkonfiguration | ✅ Vollständig |
| 12 | [Sparten](anforderungen/12_sparten/) | Spartenspezifische Anforderungen | ✏️ Teilweise |

### Spartenstruktur

Jede Sparte ist eine Konfiguration bestehend aus Produkten, Prozessen und Plausibilitäten:

```
anforderungen/12_sparten/
├── _template/          ← Vorlage für neue Sparten
├── kfz/                ← ✏️ In Erfassung (LVM-Daten)
│   ├── README.md
│   ├── produkte.md
│   ├── prozesse.md
│   ├── plausibilitaeten.md
│   ├── geschaeftsregeln.md
│   └── use_cases/
└── README.md
```

## Architekturprinzip – Spartenkonfiguration

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

| Aspekt | Technik | Änderbar ohne Deployment |
|--------|---------|--------------------------|
| Produkte (Tarife, Deckungsbausteine) | Datenbank-Konfiguration | ✅ Ja |
| Plausibilitäten (Validierungen) | Regelengine / Strategy-Pattern | ✅ Teilweise |
| Prozessabweichungen | Extension Points / Hooks | ❌ Nein |
| Spartenbündelung | Sparten-Registry-Pattern | ❌ Nein |

## Externe Systeme (Nicht-Scope)

Das Bestandsführungssystem ist über Schnittstellen an folgende externe Systeme angebunden:

| System | Richtung |
|--------|----------|
| Provisionssystem | Ausgehend |
| Konzerninkasso / Exkasso | Ausgehend / Bidirektional |
| Datawarehouse | Ausgehend |
| Druckschnittstelle | Ausgehend |
| Schadenverwaltung | Bidirektional |
| Kundenportal | Bidirektional |
| Kompetenz-System | Eingehend (Abfrage) |

## Regulatorik

- **DORA** (Digital Operational Resilience Act, EU 2022/2554) – IKT-Risikomanagement, Vorfallmeldung, Resilienz-Tests

## Geklärte Entscheidungen

| Thema | Entscheidung |
|-------|-------------|
| Initiale Sparte | KFZ |
| Mandantenfähigkeit | Nein |
| Berechtigungen | Kompetenzmodell (externes Kompetenz-System, Abfrage per Kompetenz-ID) |
| Spartenkonfiguration | Hybrid: DB + Regelengine + Hooks + Registry |
| Datenmigration | Import aus Alt-Systemen vorgesehen |
| Regulatorik | DORA-konform |
| **Technologie-Stack** | **Java 21 + Spring Boot 3 + Angular 19 + PostgreSQL 17** |
| Regelengine | Drools (KIE) |
| Hook-Mechanismus | Spring ApplicationEvents |
| Architekturstil | Modularer Monolith (Spring Modulith) |
| Asynchrone Kommunikation | Apache Kafka (Transactional Outbox Pattern) |
| GDV-Anbindung | GDV REST API (Typklassen, Regionalklassen, eVB, Zulassung) |
| Contract-Tests | Spring Cloud Contract |
| **KFZ Sub-Module** | **4 Sub-Module: kern, evb, sfr, vkz (innerhalb Spring Modulith KFZ-Modul)** |
