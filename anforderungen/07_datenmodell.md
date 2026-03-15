# Datenmodell – Versicherungsverwaltung

## Übersicht (ER-Diagramm)

```
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│    Kunde     │1    n │   Vertrag    │1    n │   Schaden    │
│──────────────│───────│──────────────│───────│──────────────│
│ id           │       │ id           │       │ id           │
│ ...          │       │ ...          │       │ ...          │
└──────────────┘       └──────┬───────┘       └──────────────┘
                              │1
                              │
                              │n
                       ┌──────┴───────┐
                       │   Prämie     │
                       │──────────────│
                       │ id           │
                       │ ...          │
                       └──────────────┘
```

## Entitäten

### Kunde
| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Eindeutige Kennung | |
| | | | | |

### Vertrag
| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Eindeutige Kennung | |
| | | | | |

### Schaden
| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Eindeutige Kennung | |
| | | | | |

### Prämie
| Attribut | Typ | Pflicht | Beschreibung | Beispiel |
|----------|-----|---------|-------------|---------|
| id | UUID | ✅ | Eindeutige Kennung | |
| | | | | |

<!-- Weitere Entitäten nach Bedarf ergänzen -->

## Beziehungen

| Von | Zu | Kardinalität | Beschreibung |
|-----|-----|-------------|-------------|
| Kunde | Vertrag | 1:n | Ein Kunde kann mehrere Verträge haben |
| Vertrag | Schaden | 1:n | Zu einem Vertrag können mehrere Schäden gehören |
| Vertrag | Prämie | 1:n | Ein Vertrag hat mehrere Prämienzahlungen |

## Enumerationen / Wertelisten

### Vertragsstatus
| Wert | Beschreibung |
|------|-------------|
| ENTWURF | |
| AKTIV | |
| RUHEND | |
| GEKUENDIGT | |
| ABGELAUFEN | |

### Schadenstatus
| Wert | Beschreibung |
|------|-------------|
| GEMELDET | |
| IN_BEARBEITUNG | |
| GENEHMIGT | |
| ABGELEHNT | |
| ABGESCHLOSSEN | |

### Sparte
| Wert | Beschreibung |
|------|-------------|
| | |

<!-- Weitere Enums nach Bedarf ergänzen -->

## Historisierung
<!-- Welche Daten müssen historisiert werden? Wie wird Historisierung umgesetzt? -->
- 
