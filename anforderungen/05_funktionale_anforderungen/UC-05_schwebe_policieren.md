# UC-05: Schwebe policieren – Antrag zum Vertrag machen

## Beschreibung
Ein freigegebener Antrag wird zum Vertrag. Dieser spartenübergreifende Use Case beschreibt den Prozess der **Policierung**, bei dem aus einem Antrag ein Vertrag mit Vertragsstand erzeugt wird.

Es gibt zwei Auslöser für die Policierung:

1. **Dunkelverarbeitung (automatisch):** Wird ein Antrag freigegeben und **nicht** ausgesteuert (→ UC-02, Pfad A), erfolgt die Policierung automatisch. Eine Schwebe wird angelegt und sofort erledigt.
2. **Manuelle Policierung durch Sachbearbeiter:** Wird ein Antrag bei der Freigabe **ausgesteuert** (→ UC-02, Pfad B), entsteht eine offene Schwebe. Ein Innendienst-Sachbearbeiter prüft die Schwebe und kann den Antrag manuell policieren, d. h. zum Vertrag machen.

Beim Policieren werden folgende Schritte durchlaufen:
- Erzeugung eines **Vorgangs** (Neugeschäft, Änderung oder Stornierung)
- Erzeugung eines **Vertragsstands** und ggf. eines neuen **Vertrags**
- Aufruf der Hooks `vor_Policierung` und `nach_Policierung` (spartenspezifisch)
- Aufruf der externen Schnittstellen **DataWarehouse (S4)**, **Provision (S1)**, **Konzerninkasso (S2)** und **Druck (S5)**
- Abschluss der Schwebe (Status → „Erledigt")

Dies ist ein **spartenübergreifender** Kernprozess. Spartenspezifische Besonderheiten (z. B. eVB-Meldung bei KFZ, SF-Klassen-Aktualisierung) werden über die Extension Points / Hooks der Spartenkonfiguration eingebunden.

## Akteure
- **Primär:** Innendienst (Sachbearbeiter) – bei manueller Policierung ausgesteuerter Schweben
- **Sekundär:** System (automatische Policierung bei Dunkelverarbeitung), Außendienst (Vertrieb) – als Informationsempfänger

## Vorbedingungen
- Ein Antrag im Status **„Freigegeben"** (Dunkelverarbeitung) oder **„Ausgesteuert"** (Innendienst-Bearbeitung) existiert
- Eine zugehörige Schwebe im Status **„Offen"** existiert (→ GR-A05 / UC-02, GR-06)
- Bei manueller Policierung: Sachbearbeiter ist am System angemeldet und besitzt die Kompetenz **„Schwebe policieren"** (SCHWEBE_POLICIEREN)
- Spartenspezifische Konfiguration (Produkte, Regeln, Hooks) ist aktiv
- Externe Schnittstellen (S1, S2, S4, S5) sind erreichbar oder Retry-Mechanismus ist aktiv

## Auslöser
- **Automatisch:** Spring Event `AntragFreigegebenEvent` bei Dunkelverarbeitung (UC-02, Pfad A)
- **Manuell:** Sachbearbeiter wählt in der Schweben-Übersicht die Funktion **„Schwebe policieren"** für eine offene Schwebe

## Ablauf (Hauptszenario)

### Phase 1: Schwebe öffnen und prüfen (nur bei manueller Policierung)
1. Sachbearbeiter öffnet die **Schweben-Übersicht** und sieht alle offenen Schweben seines Teams
2. Sachbearbeiter wählt eine Schwebe im Status **„Offen"** aus
3. System zeigt die Schwebe-Details an: zugehöriger Antrag, Aussteuerungsgrund, Antragsdaten, Partner, Produkte, berechneter Beitrag
4. Sachbearbeiter prüft die Antragsdaten und den Aussteuerungsgrund
5. System prüft, ob der Sachbearbeiter die Kompetenz **„Schwebe policieren"** (SCHWEBE_POLICIEREN) besitzt

### Phase 2: Policierung durchführen
6. Sachbearbeiter wählt die Funktion **„Policieren"** (manuell) bzw. das System startet die Policierung automatisch (Dunkelverarbeitung)
7. System ermittelt den **Vorgangstyp** anhand des Antragskontexts:
   - Kein bestehender Vertrag → **Neugeschäft**
   - Bestehender Vertrag vorhanden → **Änderung**
   - Antrag auf Beendigung → **Stornierung**
8. System erzeugt einen **Vorgang** mit dem ermittelten Vorgangstyp
9. → **Hook `vor_Policierung`** wird ausgelöst (spartenspezifische Prüfungen, z. B. SF-Klassen-Validierung bei KFZ)
10. System erzeugt den **Vertragsstand**:
    - Bei Neugeschäft: System erzeugt einen neuen **Vertrag** und den ersten Vertragsstand (Version 1)
    - Bei Änderung: System erzeugt einen neuen Vertragsstand (Version n+1) für den bestehenden Vertrag
    - Bei Stornierung: System erzeugt den finalen Vertragsstand mit Beendigungsdaten
11. Vertragsstand wird dem Vorgang zugeordnet (1:1-Zuordnung, → GR-V02)
12. Antragsstatus wechselt zu **„Freigegeben"** (falls vorher „Ausgesteuert")

### Phase 3: Schnittstellenaufrufe
13. System veröffentlicht das Spring Event **`VertragsstandErzeugtEvent`**
14. **DataWarehouse (S4):** Vertragsbewegung wird als Kafka-Message an Topic `vertrag.datawarehouse.bewegungen` gesendet (Bestandsdaten, Vertragsbewegung, Kennzahlen)
15. **Provision (S1):** Vertragsereignis wird als Kafka-Message an Topic `vertrag.provision.ereignisse` gesendet (Vertragsdaten für Provisionsberechnung)
16. **Konzerninkasso (S2):** Beitragsforderung wird als Kafka-Message an Topic `vertrag.inkasso.beitragsforderungen` gesendet (Zahlungsplan, Beitragsdaten)
17. **Druck (S5):** Druckauftrag wird per REST-Call an `POST /api/v1/druck/auftraege` ausgelöst:
    - Neugeschäft → Dokumenttyp **POLICE**
    - Änderung → Dokumenttyp **NACHTRAG**
    - Stornierung → Dokumenttyp **KUENDIGUNGSBESTAETIGUNG**

### Phase 4: Schwebe abschließen
18. → **Hook `nach_Policierung`** wird ausgelöst (spartenspezifische Aktionen, z. B. eVB-Meldung an GDV bei KFZ)
19. Schwebe-Status wechselt zu **„Erledigt"**
20. System protokolliert den Abschluss der Policierung (Benutzer, Zeitstempel, erzeugte Artefakte)
21. Bei manueller Policierung: System zeigt dem Sachbearbeiter eine Bestätigung mit Vertragsnummer und Vertragsstand-ID an

## Alternativszenarien

- **A1: Sachbearbeiter lehnt den Antrag ab (statt zu policieren)**
  Der Sachbearbeiter entscheidet, dass der Antrag nicht policiert werden soll. → Antragsstatus wechselt zu „Abgelehnt", Schwebe wird auf Status „Geschlossen" gesetzt. Es wird kein Vertragsstand erzeugt (→ GR-V04). Ein Ablehnungsgrund muss angegeben werden (mindestens 10 Zeichen).

- **A2: Sachbearbeiter weist den Antrag zurück**
  Der Sachbearbeiter stellt Mängel im Antrag fest. → Antrag geht zurück an den Ersteller, Antragsstatus wird auf „Offen" zurückgesetzt. Die Schwebe bleibt „Offen". Der Ersteller kann den Antrag nachbearbeiten und erneut freigeben (→ UC-02).

- **A3: Sachbearbeiter bearbeitet Antragsdaten vor der Policierung**
  Der Sachbearbeiter kann einzelne Antragsdaten korrigieren oder ergänzen, bevor er die Policierung auslöst (z. B. fehlende Nachweise nachtragen, Deckungssummen anpassen). Die Änderung wird revisionssicher protokolliert.

- **A4: Teilweise Schnittstellenfehler**
  Nicht alle Schnittstellen sind erreichbar. → Der Vertragsstand wird trotzdem erzeugt, die Schwebe bleibt jedoch im Status „Offen" mit Hinweis auf fehlende Schnittstellenbestätigungen. Eine Wiedervorlage wird automatisch angelegt (→ F2).

- **A5: Wiedervorlage bearbeiten**
  Sachbearbeiter bearbeitet eine Schwebe, die aufgrund eines früheren Schnittstellenfehlers auf Wiedervorlage steht. → System versucht die fehlgeschlagenen Schnittstellenaufrufe erneut. Bei Erfolg wird die Schwebe erledigt.

## Fehlerfälle

- **F1: Kompetenz „Schwebe policieren" nicht vorhanden**
  Sachbearbeiter besitzt nicht die erforderliche Kompetenz. → System zeigt Hinweis an: „Keine Berechtigung für diese Aktion." Die Schwebe kann an einen berechtigten Sachbearbeiter weitergeleitet werden.

- **F2: Schnittstellenfehler bei Policierung**
  Eine oder mehrere Schnittstellen (S1, S2, S4, S5) sind nicht erreichbar. → Der Vertragsstand wird erzeugt (Transaktion wird nicht zurückgerollt). Die betroffenen Schnittstellenaufrufe werden im Transactional-Outbox-Pattern nachgeliefert (Kafka-Retry, REST-Retry). Bei dauerhaftem Fehler: Dead-Letter-Topic bzw. Wiedervorlage in der Schwebe-Übersicht. Benachrichtigung NOT-03 wird an System-Admin ausgelöst.

- **F3: Hook `vor_Policierung` schlägt fehl**
  Ein spartenspezifischer Hook liefert einen Fehler (z. B. ungültige SF-Klasse bei KFZ). → Policierung wird abgebrochen, Schwebe bleibt „Offen". Fehler wird dem Sachbearbeiter angezeigt (bei manueller Policierung) oder in der Schwebe protokolliert (bei Dunkelverarbeitung; Schwebe wird dem Innendienst zugeordnet).

- **F4: Vertrag/Vertragsstand kann nicht erzeugt werden**
  Technischer Fehler bei der Vertragsstand-Erzeugung (z. B. Datenbank-Fehler). → Policierung wird abgebrochen, Schwebe bleibt „Offen". Fehler wird protokolliert, Wiedervorlage wird angelegt.

- **F5: Vorgangstyp kann nicht ermittelt werden**
  Der Antragskontext ist nicht eindeutig (z. B. fehlende Vertragszuordnung). → Policierung wird abgebrochen, Schwebe bleibt „Offen" mit Hinweis zur manuellen Klärung.

## Geschäftsregeln
- GR-12: Bei der Policierung werden immer alle vier Schnittstellen aufgerufen: DataWarehouse (S4), Provision (S1), Konzerninkasso (S2) und Druck (S5)
- GR-13: Die Policierung erzeugt immer genau einen Vertragsstand, der genau einem Vorgang zugeordnet ist (→ GR-V02)
- GR-14: Bei Neugeschäft wird zusätzlich ein neuer Vertrag angelegt; bei Änderung/Stornierung wird der bestehende Vertrag fortgeführt
- GR-15: Schnittstellenfehler bei der Policierung führen nicht zum Abbruch der Vertragsstand-Erzeugung; fehlgeschlagene Aufrufe werden über Retry-Mechanismen (Transactional Outbox, Dead-Letter-Topics) nachgeliefert
- GR-16: Ein Sachbearbeiter kann eine offene Schwebe nur policieren, wenn er die Kompetenz „Schwebe policieren" (SCHWEBE_POLICIEREN) besitzt
- GR-17: Bei manueller Policierung muss der Sachbearbeiter den Aussteuerungsgrund geprüft und ggf. die Antragsdaten korrigiert haben, bevor die Policierung ausgelöst werden kann
- GR-05: Ein Antrag kann nur freigegeben werden, wenn er den Status „Geprüft" hat (→ UC-02)
- GR-06: Bei jeder Freigabe wird eine Schwebe angelegt (→ UC-02)
- GR-V02: Ein Vertragsstand gehört immer zu genau einem Vorgang
- GR-V03: Der Vorgangstyp wird automatisch aus dem Antragskontext ermittelt
- GR-V04: Bei Ablehnung wird kein Vertragsstand erzeugt; die Schwebe wird geschlossen

## Daten (Ein-/Ausgabe)

### Eingabedaten
| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| Schwebe-ID | UUID | ✅ | Schwebe muss im Status „Offen" sein | Kennung der zu policierenden Schwebe |
| Policierungs-Entscheidung | Enum | ✅ (manuell) | Policieren / Ablehnen / Zurückweisen | Entscheidung des Sachbearbeiters |
| Ablehnungsgrund | Text | ✅ (bei Ablehnung) | Mindestens 10 Zeichen | Begründung der Ablehnung |
| Korrigierte Antragsdaten | Variabel | ❌ | Plausibilitätsprüfungen | Optionale Korrekturen durch den Sachbearbeiter |
| Sachbearbeiter-Kommentar | Text | ❌ | Max. 1000 Zeichen | Optionale Notiz zur Policierung |

### Ausgabedaten
| Feld | Typ | Beschreibung |
|------|-----|-------------|
| Vertragsnummer | String (eindeutig) | Fachliche Kennung des erzeugten/aktualisierten Vertrags |
| Vertragsstand-ID | UUID | Kennung des erzeugten Vertragsstands |
| Vertragsstand-Version | Integer | Versionsnummer des Vertragsstands (1 bei Neugeschäft, n+1 bei Änderung) |
| Vorgangs-ID | UUID | Kennung des erzeugten Vorgangs |
| Vorgangstyp | Enum | Neugeschäft / Änderung / Stornierung |
| Schwebe-Status | Enum | Erledigt (bei erfolgreicher Policierung) / Offen (bei Fehler) / Geschlossen (bei Ablehnung) |
| Schnittstellenstatus | Liste | Status der vier Schnittstellenaufrufe (S1, S2, S4, S5): Erfolgreich / Fehlgeschlagen / Ausstehend |
| Druckauftrags-ID | String | Kennung des ausgelösten Druckauftrags (S5) |

## Prozessübersicht: Policierung

```
                              ┌──────────────────┐
                              │  Schwebe (Offen)  │
                              │  + Antrag         │
                              └────────┬─────────┘
                                       │
                          ┌────────────┼────────────┐
                          │            │            │
                     policieren    ablehnen    zurückweisen
                          │            │            │
                          ▼            ▼            ▼
                 ┌────────────┐ ┌──────────┐ ┌──────────┐
                 │ Policierung│ │Abgelehnt │ │  Offen   │
                 │ starten    │ │(Schwebe  │ │(Antrag   │
                 └─────┬──────┘ │geschl.)  │ │zurück)   │
                       │        └──────────┘ └──────────┘
                       ▼
              ┌─────────────────┐
              │ Vorgangstyp     │
              │ ermitteln       │
              └────────┬────────┘
                       │
              ┌────────┼──────────────┐
              │        │              │
          Neugeschäft Änderung   Stornierung
              │        │              │
              ▼        ▼              ▼
        ┌──────────────────────────────────┐
        │ Hook: vor_Policierung            │
        └──────────────┬───────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────┐
        │ Vertragsstand erzeugen           │
        │ (+ ggf. neuer Vertrag)           │
        └──────────────┬───────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────┐
        │ VertragsstandErzeugtEvent        │
        └──────────────┬───────────────────┘
                       │
          ┌────────────┼────────────┬───────────┐
          │            │            │           │
          ▼            ▼            ▼           ▼
    ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
    │ S4: Data-│ │ S1: Pro- │ │ S2: Kon- │ │ S5: Druck│
    │ Warehouse│ │ vision   │ │ zernin-  │ │          │
    │ (Kafka)  │ │ (Kafka)  │ │ kasso    │ │ (REST)   │
    │          │ │          │ │ (Kafka)  │ │          │
    └──────────┘ └──────────┘ └──────────┘ └──────────┘
                       │
                       ▼
        ┌──────────────────────────────────┐
        │ Hook: nach_Policierung           │
        └──────────────┬───────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────┐
        │ Schwebe → „Erledigt"             │
        │ Policierung abgeschlossen        │
        └──────────────────────────────────┘
```

## Schnittstellendetails

### S4 – DataWarehouse (Ausgehend, Kafka)
| Aspekt | Details |
|--------|---------|
| **Kafka-Topic** | `vertrag.datawarehouse.bewegungen` |
| **Kafka-Key** | `vertragsnummer` |
| **Auslöser** | `VertragsstandErzeugtEvent` |
| **Inhalt** | Vertragsbewegung: Vertragsnummer, Sparte, Partner, Vorgangstyp, Vertragsstand-Version, Jahresbeitrag, Produkte, Zeitstempel |
| **Fehlerbehandlung** | Transactional Outbox → Kafka-Retry → Dead-Letter-Topic `vertrag.datawarehouse.bewegungen.dlt` |

### S1 – Provisionssystem (Ausgehend, Kafka)
| Aspekt | Details |
|--------|---------|
| **Kafka-Topic** | `vertrag.provision.ereignisse` |
| **Kafka-Key** | `vertragsnummer` |
| **Auslöser** | `VertragsstandErzeugtEvent` |
| **Inhalt** | Ereignistyp (NEUGESCHAEFT/AENDERUNG/STORNIERUNG), Vertragsnummer, Sparte, Partner, Jahresbeitrag, Produkte, Vermittler-ID |
| **Fehlerbehandlung** | Kafka-Retry (3 Versuche, exponentieller Backoff) → Dead-Letter-Topic `vertrag.provision.ereignisse.dlt` + Wiedervorlage |

### S2 – Konzerninkasso (Ausgehend, Kafka)
| Aspekt | Details |
|--------|---------|
| **Kafka-Topic** | `vertrag.inkasso.beitragsforderungen` |
| **Kafka-Key** | `vertragsnummer` |
| **Auslöser** | `VertragsstandErzeugtEvent` |
| **Inhalt** | Vertragsnummer, Partner (inkl. IBAN/BIC), Jahresbeitrag, Zahlungsweise, Fälligkeit, Zahlungsplan |
| **Fehlerbehandlung** | Consumer-Retry (3 Versuche) → Dead-Letter-Topic `vertrag.inkasso.beitragsforderungen.dlt` |

### S5 – Druckschnittstelle (Ausgehend, REST)
| Aspekt | Details |
|--------|---------|
| **Endpunkt** | `POST /api/v1/druck/auftraege` |
| **Auslöser** | `VertragsstandErzeugtEvent` |
| **Dokumenttyp** | POLICE (Neugeschäft), NACHTRAG (Änderung), KUENDIGUNGSBESTAETIGUNG (Stornierung) |
| **Inhalt** | Dokumenttyp, Vertragsnummer, Partner (Adresse), Vertragsstand-Version, Sparte, Produkte, Versandweg, Priorität |
| **Fehlerbehandlung** | REST-Retry (3 Versuche) → Wiedervorlage in Schwebe-Übersicht, Benachrichtigung NOT-03 |

## Nachbedingungen
- Ein **Vertragsstand** wurde erzeugt und einem **Vorgang** zugeordnet
- Bei Neugeschäft: Ein neuer **Vertrag** im Status „Aktiv" wurde erzeugt
- Bei Änderung: Der bestehende Vertrag hat einen neuen Vertragsstand (aktuelle Version)
- Die **Schwebe** wurde auf Status **„Erledigt"** gesetzt
- Der **Antrag** befindet sich im Status **„Freigegeben"**
- **DataWarehouse (S4)** hat die Vertragsbewegung erhalten (oder Retry ist aktiv)
- **Provisionssystem (S1)** hat das Vertragsereignis erhalten (oder Retry ist aktiv)
- **Konzerninkasso (S2)** hat die Beitragsforderung erhalten (oder Retry ist aktiv)
- **Druckschnittstelle (S5)** hat den Druckauftrag (Police/Nachtrag/Kündigungsbestätigung) erhalten (oder Retry ist aktiv)
- Spartenspezifische Hooks (`vor_Policierung`, `nach_Policierung`) wurden ausgeführt
- Alle Statuswechsel und Aktionen sind **revisionssicher protokolliert** (Benutzer, Zeitstempel, Hibernate Envers)

## Akzeptanzkriterien
- [ ] Eine offene Schwebe kann durch einen berechtigten Sachbearbeiter policiert werden
- [ ] Bei Dunkelverarbeitung (UC-02, Pfad A) erfolgt die Policierung automatisch ohne manuelle Interaktion
- [ ] Bei der Policierung wird immer ein Vertragsstand erzeugt und genau einem Vorgang zugeordnet
- [ ] Der Vorgangstyp (Neugeschäft/Änderung/Stornierung) wird korrekt aus dem Antragskontext ermittelt
- [ ] Bei Neugeschäft wird ein neuer Vertrag angelegt; bei Änderung wird ein neuer Vertragsstand an den bestehenden Vertrag angehängt
- [ ] Alle vier Schnittstellen werden bei der Policierung bedient: DataWarehouse (S4), Provision (S1), Konzerninkasso (S2), Druck (S5)
- [ ] DataWarehouse (S4) erhält die Vertragsbewegung über Kafka-Topic `vertrag.datawarehouse.bewegungen`
- [ ] Provisionssystem (S1) erhält das Vertragsereignis über Kafka-Topic `vertrag.provision.ereignisse`
- [ ] Konzerninkasso (S2) erhält die Beitragsforderung über Kafka-Topic `vertrag.inkasso.beitragsforderungen`
- [ ] Druckschnittstelle (S5) erhält den Druckauftrag per REST-Call mit korrektem Dokumenttyp (Police/Nachtrag/Kündigungsbestätigung)
- [ ] Schnittstellenfehler führen nicht zum Abbruch der Vertragsstand-Erzeugung (Transactional Outbox, Retry, Dead-Letter-Topic)
- [ ] Spartenspezifische Hooks (`vor_Policierung`, `nach_Policierung`) werden korrekt ausgeführt
- [ ] Bei Fehler im Hook `vor_Policierung` wird die Policierung abgebrochen und die Schwebe bleibt offen
- [ ] Sachbearbeiter kann alternativ ablehnen (→ kein Vertragsstand, Schwebe geschlossen) oder zurückweisen (→ Antrag zurück an Ersteller)
- [ ] Sachbearbeiter kann Antragsdaten vor der Policierung korrigieren
- [ ] Alle Statuswechsel (Antrag, Schwebe, Vertrag) werden revisionssicher historisiert (Hibernate Envers)
- [ ] Der Prozess funktioniert identisch über alle konfigurierten Sparten hinweg

## Wireframe / Skizze

```
┌─────────────────────────────────────────────────────────────────────┐
│  Schweben-Übersicht                              [Filter] [Suche]  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌────────┬──────────┬───────────┬──────────────────┬────────────┐ │
│  │Schwebe │ Sparte   │ Antrag    │ Aussteuerungsgrund│ Zugewiesen │ │
│  ├────────┼──────────┼───────────┼──────────────────┼────────────┤ │
│  │SW-0042 │ KFZ      │ AN-10023  │ Hohe Deckung     │ Team A     │ │
│  │SW-0043 │ Sach     │ AN-10024  │ Fehlender Nachweis│ Team B    │ │
│  │►SW-0044│ KFZ      │ AN-10025  │ Sonderfall       │ Müller, K. │ │
│  └────────┴──────────┴───────────┴──────────────────┴────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  Schwebe #SW-0044                           Status: ⬤ Offen        │
├─────────────────────────────────────────────────────────────────────┤
│  Antrag: AN-10025          Sparte: KFZ          Erstellt: 01.04.26 │
│  Aussteuerungsgrund: Sonderfall – manuelle Prüfung erforderlich    │
│  Zugewiesen an: Müller, Klaus (Innendienst Team A)                 │
├─────────────────────────────────────────────────────────────────────┤
│  Kunde: Max Mustermann (KD-4711)                                   │
│  Produkte: KFZ-Haftpflicht, Teilkasko                              │
│  Vertragsbeginn: 01.01.2027    Laufzeit: 12 Monate                 │
│  Jahresbeitrag: € 487,32                                           │
├─────────────────────────────────────────────────────────────────────┤
│  ┌─── Spartenspezifisch (KFZ) ──────────────────────────────────┐  │
│  │ Fahrzeug: VW Golf VIII (HSN/TSN: 0603/BPM)                  │  │
│  │ SF-Klasse: SF 5   Fahrerkreis: ab 25 Jahre                  │  │
│  │ eVB-Nummer: A1B2C3D   Status: ERZEUGT                       │  │
│  └──────────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────────┤
│  Kommentar: ___________________________________________________    │
│                                                                     │
│  [Policieren]    [Ablehnen]    [Zurückweisen]    [Antrag bearbeiten]│
└─────────────────────────────────────────────────────────────────────┘

          │ Klick auf „Policieren"
          ▼

┌─────────────────────────────────────────────────────────────────────┐
│  ✅ Policierung erfolgreich                                        │
├─────────────────────────────────────────────────────────────────────┤
│  Vertragsnummer:     VN-2026-100042                                │
│  Vertragsstand:      Version 1 (Neugeschäft)                      │
│  Vorgangs-ID:        VG-2026-005001                                │
│  Schwebe:            SW-0044 → Erledigt                            │
├─────────────────────────────────────────────────────────────────────┤
│  Schnittstellen:                                                    │
│  ✅ DataWarehouse (S4)  – Vertragsbewegung gesendet                │
│  ✅ Provision (S1)      – Vertragsereignis gesendet                │
│  ✅ Konzerninkasso (S2) – Beitragsforderung gesendet               │
│  ✅ Druck (S5)          – Police-Druckauftrag ausgelöst            │
├─────────────────────────────────────────────────────────────────────┤
│  [Vertrag öffnen]    [Zurück zur Übersicht]                        │
└─────────────────────────────────────────────────────────────────────┘
```

## Offene Fragen
- Soll die DataWarehouse-Meldung (S4) bei der Policierung immer ereignisbasiert erfolgen oder nur im Nightly Batch? (Aktuell: ereignisbasiert bei Policierung)
- Darf ein Sachbearbeiter die Policierung rückgängig machen (Storno eines frisch policierten Vertragsstands)?
- Welche Felder darf der Sachbearbeiter bei der manuellen Korrektur vor Policierung ändern (alle Antragsfelder oder nur bestimmte)?
- Soll die Schweben-Übersicht eine Priorisierung/Sortierung nach Alter oder Dringlichkeit bieten?
- Gibt es eine maximale Bearbeitungszeit (SLA) für offene Schweben, nach der eine automatische Eskalation erfolgt?
- Wie wird mit der DataWarehouse-Schnittstelle (S4) im MVP umgegangen, wenn diese als „Nicht MVP" priorisiert ist? (Stub oder Deaktivierung?)
