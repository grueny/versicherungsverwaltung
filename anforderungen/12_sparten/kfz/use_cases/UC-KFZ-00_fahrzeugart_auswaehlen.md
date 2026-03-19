# UC-KFZ-00: Fahrzeugart auswählen und Angebot eröffnen

## Beschreibung
Bevor ein KFZ-Angebot erstellt wird (→ spartenübergreifender UC-01), muss der Benutzer die **Fahrzeugart** auswählen. Die Fahrzeugart bestimmt, welche Produkte, Tarife, Erfassungsfelder und Plausibilitäten im nachfolgenden Angebotsprozess zur Verfügung stehen. Nach der Auswahl wird ein entsprechendes Angebot im spartenübergreifenden Kernprozess (UC-01) eröffnet, das bereits mit der gewählten Fahrzeugart vorbelegt ist.

Dieser Use Case wird über den **Hook `vor_Angebotserstellung`** in den spartenübergreifenden Kernprozess eingebunden. Er ist der erste KFZ-spezifische Schritt im Angebotsprozess.

## Akteure
- **Primär:** Außendienst (Vertrieb), Innendienst (Sachbearbeitung)
- **Sekundär:** Kunde (via Kundenportal, über Schnittstelle S7)

## Vorbedingungen
- Benutzer hat im spartenübergreifenden Angebotsprozess (UC-01) die Sparte **KFZ** ausgewählt
- Benutzer besitzt die Kompetenz „Angebot erstellen"

## Auslöser
- Hook `vor_Angebotserstellung` wird durch die Sparte KFZ ausgelöst (nach Spartenauswahl in UC-01, Phase 2, Schritt 6)

## Fahrzeugarten

| ID | Fahrzeugart | Beschreibung | Besonderheiten |
|----|-------------|-------------|----------------|
| FA-01 | **PKW** | Personenkraftwagen (auch SUV, Kombi, Cabrio etc.) | Standardprozess; Typklasse/Regionalklasse über GDV; SF-Klasse; alle Produkte (HP/TK/VK/Bausteine) verfügbar |
| FA-02 | **LKW** | Lastkraftwagen, Transporter, Nutzfahrzeuge | Abweichende Tarifierung; ggf. Gewichtsklassen; eingeschränkte Zusatzbausteine; gewerbliche Nutzung als Standard |
| FA-03 | **Oldtimer** | Fahrzeuge mit Oldtimer-Zulassung (H-Kennzeichen, mind. 30 Jahre) | Spezielle Oldtimer-Tarife; Wertgutachten erforderlich; eingeschränkte Fahrleistung; kein SF-System |
| FA-04 | **Versicherungskennzeichen** | Mopeds, Mofas, E-Scooter, Segways etc. (Saisonkennzeichen 1.3.–28.2.) | Keine Zulassung, sondern Versicherungskennzeichen; Saisonvertrag; vereinfachte Tarifierung; nur HP (+ ggf. TK) |
| FA-05 | **Motorrad / Kraftrad** | Motorräder, Leichtkrafträder | SF-Klasse; Saisonkennzeichen möglich; eigene Typklassen; eingeschränkte Zusatzbausteine |
| FA-06 | **Wohnmobil / Wohnwagen** | Wohnmobile (eigener Antrieb) und Wohnwagen (Anhänger) | Eigene Typklassen; Gewichts-/Längenklassen; ggf. Inhaltsversicherung; Campingzubehör |
| FA-07 | **Anhänger** | Anhänger ohne eigenen Antrieb (Transportanhänger, Bootstrailer etc.) | Nur HP; keine Kasko üblich; vereinfachte Tarifierung |
| FA-08 | **Sonderfahrzeug** | Land-/Forstwirtschaftliche Fahrzeuge, Baumaschinen, Gabelstapler etc. | Spezielle Tarife; ggf. keine Straßenzulassung; abweichende Deckungen |

## Ablauf (Hauptszenario)

1. System erkennt Sparte **KFZ** und löst den Hook `vor_Angebotserstellung` aus
2. System zeigt die **Auswahl der Fahrzeugarten** an (Kacheln oder Liste mit Symbolen und Kurzbeschreibung)
3. Benutzer wählt die gewünschte Fahrzeugart aus
4. System lädt die zur Fahrzeugart passende Konfiguration:
   - Verfügbare **Produkte** (z. B. PKW: HP/TK/VK + alle Bausteine; Anhänger: nur HP)
   - Relevante **Erfassungsfelder** (z. B. PKW: HSN/TSN/SF-Klasse; Oldtimer: Wertgutachten)
   - Zutreffende **Plausibilitäten** (z. B. Oldtimer: H-Kennzeichen Pflicht)
   - Angepasste **Tarifierung** (z. B. Versicherungskennzeichen: Saisonvertrag)
5. System eröffnet das Angebot (UC-01, Phase 2) mit der vorausgewählten Fahrzeugart
6. Das Angebot wird im Datenmodell mit der Fahrzeugart gekennzeichnet (Feld `fahrzeugart`)

## Alternativszenarien

- **A1: Fahrzeugart nachträglich ändern**
  Solange das Angebot im Status „Entwurf" ist und noch keine fahrzeugartspezifischen Daten erfasst wurden, kann die Fahrzeugart geändert werden. Dabei werden alle bereits geladenen Produkte und Felder zurückgesetzt. Das System warnt: „Alle bisherigen Eingaben gehen verloren. Fahrzeugart wirklich ändern?"

- **A2: Fahrzeugart ändern mit bereits erfassten Daten**
  Wurden bereits fahrzeugartspezifische Daten erfasst (z. B. HSN/TSN), wird die Änderung der Fahrzeugart blockiert. Der Benutzer muss das Angebot löschen und ein neues erstellen.

- **A3: Unbekannte Fahrzeugart**
  Falls die gewünschte Fahrzeugart nicht in der Auswahl vorhanden ist, kann der Benutzer „Sonstige" wählen oder den Innendienst kontaktieren. Bei „Sonstige" wird ein generisches KFZ-Angebot mit manueller Tarifierung eröffnet.

## Fehlerfälle

- **F1: Keine Produkte für Fahrzeugart konfiguriert**
  Die gewählte Fahrzeugart hat keine aktiven Produkte in der Spartenkonfiguration. → System zeigt Hinweis: „Für diese Fahrzeugart sind aktuell keine Produkte verfügbar."

- **F2: Kompetenz fehlt für bestimmte Fahrzeugart**
  Bestimmte Fahrzeugarten (z. B. Sonderfahrzeug) erfordern eine spezielle Kompetenz. → Nicht berechtigte Fahrzeugarten werden ausgegraut mit Hinweis angezeigt.

## Geschäftsregeln
- GR-KFZ-FA01: Jedes KFZ-Angebot muss genau eine Fahrzeugart zugeordnet haben
- GR-KFZ-FA02: Die Fahrzeugart bestimmt die verfügbaren Produkte, Erfassungsfelder und Plausibilitäten
- GR-KFZ-FA03: Die Fahrzeugart kann nur im Status „Entwurf" und ohne erfasste fahrzeugartspezifische Daten geändert werden
- GR-KFZ-FA04: Für Oldtimer (FA-03) ist ein Wertgutachten Pflicht
- GR-KFZ-FA05: Für Versicherungskennzeichen (FA-04) gilt ein fester Saisonzeitraum (01.03.–28.02.)

## Daten (Ein-/Ausgabe)

### Eingabedaten
| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| Fahrzeugart | Enum (FA-01 bis FA-08) | ✅ | Muss aktiv konfiguriert sein | Vom Benutzer gewählte Fahrzeugart |

### Ausgabedaten
| Feld | Typ | Beschreibung |
|------|-----|-------------|
| Verfügbare Produkte | Liste | Produkte, die für die Fahrzeugart buchbar sind |
| Erfassungsfelder | Liste | Felder, die für die Fahrzeugart relevant sind |
| Plausibilitätsregeln | Liste | Für die Fahrzeugart geltende Validierungen |
| Tarifierungsparameter | Konfiguration | Fahrzeugartspezifische Tarifmerkmale |

## Nachbedingungen
- Das Angebot ist mit der gewählten Fahrzeugart vorbelegt
- Produkte, Erfassungsfelder und Plausibilitäten sind auf die Fahrzeugart abgestimmt
- Der spartenübergreifende Angebotsprozess (UC-01) setzt mit Phase 2 fort

## Akzeptanzkriterien
- [ ] Bei Auswahl der Sparte KFZ wird vor der Angebotserstellung die Fahrzeugartauswahl angezeigt
- [ ] Alle konfigurierten Fahrzeugarten werden mit Bezeichnung und Kurzbeschreibung angezeigt
- [ ] Nach Auswahl werden nur die für die Fahrzeugart gültigen Produkte angeboten
- [ ] Fahrzeugartspezifische Erfassungsfelder werden korrekt angezeigt (z. B. Wertgutachten bei Oldtimer)
- [ ] Die Fahrzeugart wird im Angebot gespeichert und ist im weiteren Prozess verfügbar
- [ ] Eine Änderung der Fahrzeugart ist nur unter den definierten Bedingungen möglich (A1/A2)
- [ ] Nicht verfügbare Fahrzeugarten werden korrekt behandelt (F1/F2)

## Wireframe / Skizze

```
┌─────────────────────────────────────────────────────────┐
│  Neues KFZ-Angebot – Fahrzeugart wählen                │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │  🚗 PKW     │  │  🚛 LKW     │  │  🏎️ Oldtimer│    │
│  │             │  │             │  │             │    │
│  │ PKW, SUV,   │  │ Transport,  │  │ H-Kennz.,  │    │
│  │ Kombi, ...  │  │ Nutzfzg.    │  │ mind. 30 J. │    │
│  └─────────────┘  └─────────────┘  └─────────────┘    │
│                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │  🛵 Vers.-  │  │  🏍️ Motor-  │  │  🏕️ Wohn-   │    │
│  │  Kennzeichen│  │    rad      │  │ mobil/Wagen │    │
│  │             │  │             │  │             │    │
│  │ Mofa, Moped,│  │ Motorrad,   │  │ Wohnmobil,  │    │
│  │ E-Scooter   │  │ Leichtkrad  │  │ Wohnwagen   │    │
│  └─────────────┘  └─────────────┘  └─────────────┘    │
│                                                         │
│  ┌─────────────┐  ┌─────────────┐                      │
│  │  🚜 Sonder- │  │  🚛 Anhänger│                      │
│  │  fahrzeug   │  │             │                      │
│  │             │  │ Transport,  │                      │
│  │ Baumaschin.,│  │ Bootstrail. │                      │
│  │ Landwirtsch.│  │             │                      │
│  └─────────────┘  └─────────────┘                      │
│                                                         │
│                                          [Abbrechen]    │
└─────────────────────────────────────────────────────────┘
```

## Offene Fragen
- Sollen weitere Fahrzeugarten ergänzt werden (z. B. Taxi, Selbstfahrervermietfahrzeug)?
- Gibt es Fahrzeugarten, die nur vom Innendienst bearbeitet werden dürfen?
- Soll die Fahrzeugartauswahl die häufigsten Arten hervorheben (z. B. PKW größer darstellen)?
