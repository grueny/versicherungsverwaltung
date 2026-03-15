# Spartenübersicht – Versicherungsverwaltung

> Jede Sparte ist eine Konfiguration aus **Produkten**, **spartenspezifischen Prozessen** und **Plausibilitäten**.  
> Spartenübergreifende Anforderungen werden unter `05_funktionale_anforderungen/` und `08_geschaeftsregeln.md` erfasst.

## Architekturprinzip

```
┌─────────────────────────────────────────────────────────────┐
│              Spartenübergreifender Kern                      │
│  (Einheitliche Prozesse, UI-Patterns, Geschäftsregeln)      │
├─────────────┬─────────────┬─────────────┬───────────────────┤
│  Sparte A   │  Sparte B   │  Sparte C   │  Sparte ...       │
│ ┌─────────┐ │ ┌─────────┐ │ ┌─────────┐ │ ┌─────────┐      │
│ │Produkte │ │ │Produkte │ │ │Produkte │ │ │Produkte │      │
│ ├─────────┤ │ ├─────────┤ │ ├─────────┤ │ ├─────────┤      │
│ │Prozesse │ │ │Prozesse │ │ │Prozesse │ │ │Prozesse │      │
│ ├─────────┤ │ ├─────────┤ │ ├─────────┤ │ ├─────────┤      │
│ │Plausi-  │ │ │Plausi-  │ │ │Plausi-  │ │ │Plausi-  │      │
│ │bilitäten│ │ │bilitäten│ │ │bilitäten│ │ │bilitäten│      │
│ └─────────┘ │ └─────────┘ │ └─────────┘ │ └─────────┘      │
└─────────────┴─────────────┴─────────────┴───────────────────┘
```

## Registrierte Sparten

| Nr. | Sparte | Ordner | Status | Beschreibung |
|-----|--------|--------|--------|-------------|
| 1 | KFZ | `kfz/` | ✏️ In Erfassung | Kraftfahrzeugversicherung (HP, TK, VK, Schutzbrief etc.) |

> **Status:** 🔲 Geplant | ✏️ In Erfassung | ✅ Vollständig | 🔍 In Review

## Neue Sparte anlegen

1. Kopiere den Ordner `_template/` und benenne ihn nach der Sparte (z. B. `sach/`, `leben/`, `kfz/`)
2. Befülle die Dateien in der folgenden Reihenfolge:
   - `README.md` – Spartenübersicht und Besonderheiten
   - `produkte.md` – Produktdefinitionen, Merkmale, Tarife
   - `prozesse.md` – Abweichungen/Ergänzungen zum Kernprozess
   - `plausibilitaeten.md` – Spartenspezifische Validierungen
   - `geschaeftsregeln.md` – Spartenspezifische Geschäftsregeln
   - `use_cases/` – Spartenspezifische Use Cases
3. Trage die Sparte in die Tabelle oben ein

## Verzeichnisstruktur je Sparte

```
sparte_name/
├── README.md                 ← Überblick, Besonderheiten der Sparte
├── produkte.md               ← Produkte, Merkmale, Tarife, Deckungsbausteine
├── prozesse.md               ← Abweichungen vom Kernprozess
├── plausibilitaeten.md       ← Spartenspezifische Validierungen
├── geschaeftsregeln.md       ← Spartenspezifische Geschäftsregeln
└── use_cases/                ← Spartenspezifische Use Cases
    └── README.md
```
