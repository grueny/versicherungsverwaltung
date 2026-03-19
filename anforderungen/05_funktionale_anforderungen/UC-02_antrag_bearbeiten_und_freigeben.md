# UC-02: Antrag bearbeiten und freigeben

## Beschreibung
Der Benutzer bearbeitet einen bestehenden Antrag (z. B. erzeugt aus UC-01). Der Antrag kann mit weiteren Daten ergänzt, berechnet (Beitragskalkulation) und geprüft (Plausibilitäten) werden. Ist der Antrag vollständig und fehlerfrei, kann der Benutzer ihn freigeben.

Bei der **Freigabe** prüft das System anhand konfigurierbarer Aussteuerungsregeln, ob der Antrag zur manuellen Bearbeitung an den Innendienst **ausgesteuert** wird (z. B. bei hohen Deckungssummen, Sonderfällen, fehlenden Nachweisen). Unabhängig davon wird eine **Schwebe** angelegt, die den offenen Vorgang repräsentiert.

- **Ausgesteuert:** Der Antrag wird dem Innendienst zur manuellen Prüfung/Entscheidung vorgelegt. Die Schwebe bleibt offen, bis der Innendienst den Antrag freigibt oder ablehnt.
- **Nicht ausgesteuert:** Das System erzeugt automatisch einen **Vertragsstand** und schließt die Schwebe ab. Der Vertragsstand gehört zu einem **Vorgang** (z. B. Neugeschäft, Änderung, Stornierung).

Dies ist ein **spartenübergreifender** Kernprozess. Spartenspezifische Besonderheiten (z. B. eVB-Verfahren bei KFZ, SF-Klassen-Validierung) werden über die Extension Points / Hooks der Spartenkonfiguration eingebunden.

## Akteure
- **Primär:** Außendienst (Vertrieb), Innendienst (Sachbearbeitung)
- **Sekundär:** Kunde (via Kundenportal, über Schnittstelle S7)

## Vorbedingungen
- Ein Antrag im Status **„Offen"** existiert (z. B. erzeugt durch UC-01)
- Benutzer ist am System angemeldet und besitzt die Kompetenz „Antrag bearbeiten"
- Spartenspezifische Konfiguration (Produkte, Regeln, Hooks) ist aktiv

## Auslöser
- Benutzer öffnet einen bestehenden Antrag zur Bearbeitung
- Alternativ: System zeigt dem Innendienst einen ausgesteuerten Antrag in der Schweben-Übersicht an

## Ablauf (Hauptszenario)

### Phase 1: Antrag ergänzen
1. Benutzer öffnet den Antrag im Status **„Offen"**
2. System zeigt die vorhandenen Antragsdaten an (übernommen aus Angebot oder manuell erfasst)
3. Benutzer ergänzt oder ändert Daten (Produkte, Deckungsbausteine, vertragsspezifische Angaben)
4. → **Hook `vor_Antragsprüfung`** wird ausgelöst (spartenspezifische Zusatzerfassungen)
5. System führt bei jeder Eingabe **Plausibilitätsprüfungen** durch (spartenübergreifend + spartenspezifisch)

### Phase 2: Antrag berechnen
6. Benutzer löst die **Beitragsberechnung** aus (oder diese erfolgt automatisch bei vollständiger Datenlage)
7. System berechnet den Beitrag auf Basis der gewählten Produkte, Tarife und Merkmale
8. System zeigt den berechneten Beitrag und die Beitragsdetails an
9. Antragsstatus wechselt zu **„Berechnet"**

### Phase 3: Antrag prüfen
10. Benutzer löst die **Gesamtprüfung** des Antrags aus
11. System führt alle relevanten Plausibilitätsprüfungen durch (spartenübergreifend + spartenspezifisch)
12. System zeigt das Prüfergebnis an:
    - ✅ **Prüfung bestanden** → Antrag ist freigebbar, Status wechselt zu **„Geprüft"**
    - ❌ **Prüfung nicht bestanden** → Fehler werden angezeigt, Antrag bleibt im Status „Berechnet"

### Phase 4: Antrag freigeben
13. Benutzer wählt die Funktion **„Antrag freigeben"**
14. System prüft, ob der Benutzer die Kompetenz „Antrag freigeben" besitzt
15. System ermittelt den **Vorgangstyp** (Neugeschäft, Änderung, Stornierung) anhand des Antragskontexts
16. System legt eine **Schwebe** an, die den offenen Vorgang repräsentiert
17. System prüft anhand der **Aussteuerungsregeln**, ob der Antrag ausgesteuert wird:

#### Pfad A: Nicht ausgesteuert (Dunkelverarbeitung)
18a. Antragsstatus wechselt zu **„Freigegeben"**
19a. System erzeugt einen **Vertragsstand** und ordnet ihn dem Vorgang zu
20a. → **Hook `vor_Policierung`** wird ausgelöst (spartenspezifische Prüfungen)
21a. → **Hook `nach_Policierung`** wird ausgelöst (spartenspezifische Aktionen, z. B. eVB-Meldung)
22a. Schwebe wird auf Status **„Erledigt"** gesetzt
23a. Druckauftrag wird über Druckschnittstelle (S5) ausgelöst (Police/Nachtrag)
24a. Beitragsdaten werden an Konzerninkasso (S2) übermittelt
25a. Vertragsdaten werden an Provisionssystem (S1) übermittelt

#### Pfad B: Ausgesteuert (Innendienst-Bearbeitung)
18b. Antragsstatus wechselt zu **„Ausgesteuert"**
19b. Schwebe bleibt im Status **„Offen"** mit Verweis auf den Aussteuerungsgrund
20b. System ordnet die Schwebe dem zuständigen Innendienst-Team zu (basierend auf Sparte, Aussteuerungsgrund, ggf. Kompetenz)
21b. Innendienst-Sachbearbeiter öffnet die Schwebe und bearbeitet den Antrag
22b. Sachbearbeiter kann den Antrag:
    - **Freigeben** → weiter mit Schritt 18a (Dunkelverarbeitung)
    - **Ablehnen** → Antragsstatus wechselt zu „Abgelehnt", Schwebe wird geschlossen
    - **Zurückweisen** → Antrag geht zurück an den Ersteller zur Nachbearbeitung, Status wird auf „Offen" zurückgesetzt

## Alternativszenarien

- **A1: Antrag zwischenspeichern**
  Der Benutzer kann den Antrag jederzeit im aktuellen Status speichern und später weiterbearbeiten.

- **A2: Antrag stornieren**
  Der Benutzer kann einen Antrag stornieren, solange noch kein Vertragsstand erzeugt wurde. Der Antragsstatus wechselt zu „Storniert", eine eventuell vorhandene Schwebe wird geschlossen.

- **A3: Antrag an Innendienst eskalieren**
  Der Außendienst kann einen Antrag manuell an den Innendienst eskalieren, auch wenn keine automatische Aussteuerung greift (z. B. bei Rückfragen).

- **A4: Freigabe über Kundenportal**
  Der Kunde gibt den Antrag über das Kundenportal frei. Es gelten ggf. strengere Aussteuerungsregeln (z. B. immer ausgesteuert bei hohen Deckungssummen).

## Fehlerfälle

- **F1: Plausibilitätsfehler bei Prüfung**
  Der Antrag enthält Fehler. → System zeigt alle Fehler an, Freigabe wird blockiert.

- **F2: Kompetenz „Antrag freigeben" nicht vorhanden**
  Benutzer besitzt nicht die erforderliche Kompetenz. → System zeigt Hinweis an. Antrag kann an berechtigten Benutzer weitergeleitet werden.

- **F3: Schwebe kann nicht angelegt werden**
  Technischer Fehler bei Schwebe-Erzeugung. → Freigabe wird abgebrochen, Antrag bleibt im Status „Geprüft". Fehler wird protokolliert.

- **F4: Schnittstellenfehler bei Vertragsstand-Erzeugung**
  Druckschnittstelle, Inkasso oder Provision sind nicht erreichbar. → Vertragsstand wird erzeugt, Schwebe bleibt offen mit Hinweis auf fehlende Schnittstellenbestätigung. Wiedervorlage wird angelegt.

## Geschäftsregeln
- GR-05: Ein Antrag kann nur freigegeben werden, wenn er den Status „Geprüft" hat
- GR-06: Bei jeder Freigabe wird eine Schwebe angelegt, unabhängig davon, ob der Antrag ausgesteuert wird
- GR-07: Aussteuerungsregeln sind konfigurierbar (pro Sparte und spartenübergreifend) und bestimmen, ob ein Antrag dem Innendienst vorgelegt wird
- GR-08: Ein Vertragsstand gehört immer zu genau einem Vorgang (Neugeschäft, Änderung, Stornierung)
- GR-09: Der Vorgangstyp wird automatisch aus dem Antragskontext ermittelt (neuer Vertrag = Neugeschäft, bestehender Vertrag = Änderung)
- GR-10: Ein ausgesteuerter Antrag kann vom Innendienst freigegeben, abgelehnt oder zurückgewiesen werden
- GR-11: Bei Ablehnung eines Antrags wird keine Vertragsstand erzeugt; die Schwebe wird geschlossen

## Daten (Ein-/Ausgabe)

### Eingabedaten
| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| Antragsdaten | Variabel | ✅ | Plausibilitätsprüfungen | Alle vertragsrelevanten Daten (aus Angebot übernommen oder ergänzt) |
| Spartenspezifische Daten | Variabel | Spartenabhängig | Über Spartenplausibilitäten | z. B. Fahrzeugdaten, SF-Klasse bei KFZ |
| Freigabe-Entscheidung (Innendienst) | Enum | ✅ (bei Aussteuerung) | Freigeben / Ablehnen / Zurückweisen | Entscheidung des Sachbearbeiters |
| Ablehnungsgrund | Text | ✅ (bei Ablehnung) | Mindestens 10 Zeichen | Begründung der Ablehnung |

### Ausgabedaten
| Feld | Typ | Beschreibung |
|------|-----|-------------|
| Antragsstatus | Enum | Offen → Berechnet → Geprüft → Freigegeben / Ausgesteuert / Abgelehnt / Storniert |
| Schwebe-ID | String (eindeutig) | Kennung der erzeugten Schwebe |
| Schwebe-Status | Enum | Offen → Erledigt / Geschlossen |
| Aussteuerungsgrund | Text / Code | Grund der Aussteuerung (falls zutreffend) |
| Vorgangs-ID | String (eindeutig) | Kennung des Vorgangs (Neugeschäft, Änderung, Stornierung) |
| Vorgangstyp | Enum | Neugeschäft / Änderung / Stornierung |
| Vertragsstand-ID | String (eindeutig) | Kennung des erzeugten Vertragsstands (falls nicht ausgesteuert) |

## Statusmodell Antrag

```
  ┌──────────┐    berechnen     ┌──────────┐    Prüfung       ┌──────────┐
  │  Offen   │─────────────────►│Berechnet │───bestanden─────►│ Geprüft  │
  └──────────┘                   └──────────┘                  └────┬─────┘
       ▲                              │                             │
       │ zurück-                      │ stornieren           freigeben
       │ weisen                       ▼                             │
       │                         ┌──────────┐               ┌──────┴──────┐
       │                         │Storniert │               │ Aussteuerung│
       │                         └──────────┘               │   prüfen    │
       │                                                    └──┬───────┬──┘
       │                                                       │       │
       │         ┌──────────────┐                    nicht      │       │  ausge-
       └─────────│ Ausgesteuert │◄──────────────── steuert ────┘       │  steuert
                 └──────┬───────┘                                      │
                        │                                              │
              ┌─────────┼──────────┐                            ┌──────▼──────┐
              │         │          │                            │ Freigegeben │
         freigeben  ablehnen  zurück-                           └──────┬──────┘
              │         │     weisen                                   │
              ▼         ▼                                        erzeugt
        ┌──────────┐ ┌──────────┐                                     │
        │Freige-   │ │Abgelehnt │                              ┌──────▼──────┐
        │geben     │ └──────────┘                              │ Vertrags-   │
        └────┬─────┘                                           │   stand     │
             │                                                 └─────────────┘
             │ erzeugt
             ▼
       ┌─────────────┐
       │ Vertrags-   │
       │   stand     │
       └─────────────┘
```

## Statusmodell Schwebe

```
  ┌──────────┐    Vertragsstand erzeugt    ┌──────────┐
  │  Offen   │────────────────────────────►│ Erledigt │
  └──────────┘                             └──────────┘
       │
       │ Antrag abgelehnt / storniert
       ▼
  ┌───────────┐
  │Geschlossen│
  └───────────┘
```

## Zusammenspiel: Antrag → Schwebe → Vorgang → Vertragsstand

```
┌──────────────┐      erzeugt       ┌──────────────┐
│   Antrag     │───────────────────►│   Schwebe    │
│   (Geprüft)  │                    │   (Offen)    │
└──────┬───────┘                    └──────┬───────┘
       │                                   │
       │ freigeben                         │ abschließen
       ▼                                   ▼
┌──────────────┐                    ┌──────────────┐
│   Vorgang    │◄───── gehört zu ──│ Vertragsstand│
│   (z.B.      │                    │              │
│  Neugeschäft)│                    │              │
└──────────────┘                    └──────────────┘
```

## Nachbedingungen
- Eine Schwebe wurde angelegt und ist entweder offen (bei Aussteuerung) oder erledigt (bei Dunkelverarbeitung)
- Bei Dunkelverarbeitung: Ein Vertragsstand wurde erzeugt und einem Vorgang zugeordnet
- Druckauftrag, Inkasso-Meldung und Provisionsmeldung wurden ausgelöst (bei erfolgreicher Vertragsstand-Erzeugung)
- Spartenspezifische Hooks (`vor_Policierung`, `nach_Policierung`) wurden ausgeführt
- Alle Statuswechsel sind revisionssicher protokolliert (Benutzer, Zeitstempel)

## Akzeptanzkriterien
- [ ] Ein Antrag im Status „Offen" kann mit Daten ergänzt, berechnet und geprüft werden
- [ ] Ein geprüfter Antrag kann freigegeben werden; dabei wird immer eine Schwebe angelegt
- [ ] Aussteuerungsregeln werden korrekt angewendet (konfigurierbar pro Sparte und spartenübergreifend)
- [ ] Bei Aussteuerung: Antrag erscheint in der Schweben-Übersicht des Innendienstes mit Aussteuerungsgrund
- [ ] Bei Nicht-Aussteuerung: Vertragsstand wird automatisch erzeugt und dem korrekten Vorgang zugeordnet
- [ ] Der Vorgangstyp (Neugeschäft/Änderung/Stornierung) wird korrekt aus dem Kontext ermittelt
- [ ] Innendienst kann ausgesteuerte Anträge freigeben, ablehnen oder zurückweisen
- [ ] Bei Ablehnung wird kein Vertragsstand erzeugt und die Schwebe geschlossen
- [ ] Bei Zurückweisung geht der Antrag zurück an den Ersteller im Status „Offen"
- [ ] Spartenspezifische Hooks (`vor_Policierung`, `nach_Policierung`) werden bei Vertragsstand-Erzeugung ausgelöst
- [ ] Schnittstellen (Druck S5, Inkasso S2, Provision S1) werden bei Vertragsstand-Erzeugung bedient
- [ ] Alle Statuswechsel (Antrag + Schwebe) werden revisionssicher historisiert

## Offene Fragen
- Welche konkreten Aussteuerungsregeln gelten spartenübergreifend, welche sind spartenspezifisch?
- Gibt es eine SLA für die Bearbeitung ausgesteuerter Anträge im Innendienst?
- Soll der Außendienst den Bearbeitungsstand einer Schwebe einsehen können?
- Wie wird mit Schweben umgegangen, die längere Zeit nicht bearbeitet werden (Eskalation)?
- Gibt es eine automatische Wiedervorlage bei fehlgeschlagenen Schnittstellenaufrufen?
