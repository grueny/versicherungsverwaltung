# UC-11: Versicherungsgewerbetarif als Nachlass verwalten

## Beschreibung
Mitarbeitende der Versicherungswirtschaft (Innen- und Außendienst des eigenen Unternehmens sowie anderer Versicherungsunternehmen) erhalten auf Basis des **Versicherungsgewerbetarifs (VGT)** einen prozentualen Nachlass auf ihre persönlichen Versicherungsverträge. Der VGT-Nachlass wird als **spartenübergreifender, automatisierter Nachlass** auf **Vertragsebene** abgebildet und nutzt das bestehende Nachlass-/Zuschlagssystem (→ UC-10).

Der Prozess umfasst die **Erfassung und Prüfung der VGT-Berechtigung**, die **automatische Vergabe des VGT-Nachlasses** bei der Angebotserstellung sowie die **jährliche Überprüfung der Berechtigung** zum Hauptfälligkeitstermin. Endet die Berechtigung (z. B. durch Ausscheiden aus der Versicherungswirtschaft), wird der VGT-Nachlass bei der nächsten Beitragsberechnung automatisch entfernt.

Dies ist ein **spartenübergreifender** Prozess. Der VGT-Nachlass gilt einheitlich für alle Sparten. Die Nachlasshöhe ist konfigurierbar und kann pro Sparte differenziert werden (→ UC-03).

## Akteure
- **Primär:** Außendienst (Vertrieb), Innendienst (Sachbearbeitung)
- **Sekundär:** System (automatische VGT-Vergabe und -Prüfung), Kompetenz-System (S8)

## Vorbedingungen
- Ein Angebot, Antrag oder eine Schwebe existiert in einem bearbeitbaren Status (Angebot: `ENTWURF`, `BERECHNET`; Antrag: `OFFEN`, `GEPRUEFT`; Schwebe: `OFFEN`)
- Benutzer ist am System angemeldet und besitzt die Kompetenz **`VGT_NACHLASS_BEARBEITEN`** (→ Kompetenz-System S8)
- Der Partner (Versicherungsnehmer) ist erfasst und sein Partnerprofil ist zugänglich

## Auslöser
- Benutzer markiert einen Partner als VGT-berechtigt und gibt den Arbeitgebernachweis ein
- System erkennt bei der Angebotserstellung, dass der VN als VGT-berechtigt hinterlegt ist, und schlägt den VGT-Nachlass automatisch vor
- Jährlicher Batch-Lauf zum Hauptfälligkeitstermin prüft die VGT-Berechtigung bestehender Verträge

## Ablauf (Hauptszenario)

### Phase 1: VGT-Berechtigung am Partner erfassen
1. Benutzer öffnet das Partnerprofil des Versicherungsnehmers
2. Benutzer aktiviert das Merkmal **„VGT-berechtigt"** und erfasst die folgenden Daten:
   - **Arbeitgeber:** Name des Versicherungsunternehmens oder Vermittlerbetriebs
   - **Beschäftigungsverhältnis:** Enum (`ANGESTELLT`, `AUSZUBILDEND`, `RUHEND`, `PENSIONIERT`)
   - **Nachweis-Referenz:** Referenz auf den VGT-Nachweis (z. B. Bescheinigung des Arbeitgebers, Mitgliedsausweis BWV/BVK)
   - **VGT gültig ab:** Datum, ab dem die VGT-Berechtigung gilt (Standard: heute)
   - **VGT gültig bis:** Optionales Enddatum der Berechtigung (NULL = unbefristet, z. B. bei Pensionierung)
3. System speichert die VGT-Berechtigung am Partner und protokolliert die Änderung (Hibernate Envers)

### Phase 2: Automatische VGT-Nachlassvergabe bei Angebotserstellung
4. Benutzer erstellt ein neues Angebot (→ UC-01) für einen VGT-berechtigten Partner
5. System erkennt die VGT-Berechtigung und prüft:
   - Ist die VGT-Berechtigung zum Vertragsbeginn gültig (`vgt_gueltig_ab ≤ Vertragsbeginn` und `vgt_gueltig_bis IS NULL OR vgt_gueltig_bis ≥ Vertragsbeginn`)?
   - Ist die VGT-Nachlasshöhe für die gewählte Sparte konfiguriert?
6. Bei gültiger Berechtigung: System erzeugt **automatisch** einen Nachlass auf **Vertragsebene** mit:
   - Art: `NACHLASS`
   - Ebene: `VERTRAG`
   - Nachlassart: `VGT_NACHLASS`
   - Werttyp: `PROZENTUAL`
   - Wert: Konfigurierter VGT-Nachlasssatz der Sparte (Standard: 15 %)
   - Gültig ab: Vertragsbeginn
   - Gültig bis: `vgt_gueltig_bis` des Partners (falls befristet) oder NULL (unbefristet)
   - Begründung: `Versicherungsgewerbetarif – {Arbeitgeber}, {Beschäftigungsverhältnis}` (automatisch generiert)
7. System zeigt dem Benutzer einen **Hinweis**: „VGT-Nachlass von {X} % wurde automatisch vergeben (Versicherungsgewerbetarif)."
8. System löst eine **Neuberechnung** des Beitrags aus (→ UC-10, Phase 4)

### Phase 3: Manuelle VGT-Nachlassvergabe
9. Alternativ kann der Benutzer den VGT-Nachlass manuell über den Bereich „Nachlässe & Zuschläge" (→ UC-10) hinzufügen, indem er die Nachlassart `VGT_NACHLASS` auswählt
10. System prüft, ob der Partner als VGT-berechtigt hinterlegt ist:
    - **Ja:** Nachlasshöhe wird aus der Konfiguration vorbelegt; Benutzer kann sie innerhalb der konfigurierten Grenzen anpassen
    - **Nein:** System zeigt Fehler: „Der Versicherungsnehmer ist nicht als VGT-berechtigt hinterlegt. Bitte erfassen Sie zunächst die VGT-Berechtigung im Partnerprofil."
11. System führt die Standard-Plausibilitätsprüfungen durch (→ UC-10, GR-P03 bis GR-P08) und prüft zusätzlich die VGT-spezifischen Regeln (→ GR-P13, GR-P14)
12. Bei erfolgreicher Prüfung: VGT-Nachlass wird gespeichert
13. System löst eine **Neuberechnung** des Beitrags aus

### Phase 4: Jährliche VGT-Berechtigungsprüfung (Batch)
14. Zum **Hauptfälligkeitstermin** jedes Vertrags führt das System automatisch eine VGT-Berechtigungsprüfung durch:
    - Ist der VGT-Nachlass im aktuellen Vertragsstand vorhanden?
    - Ist die VGT-Berechtigung des Partners noch gültig (`vgt_gueltig_bis IS NULL OR vgt_gueltig_bis ≥ nächster Versicherungsbeginn`)?
15. **Berechtigung weiterhin gültig:** Keine Änderung. VGT-Nachlass bleibt bestehen.
16. **Berechtigung abgelaufen oder entzogen:**
    - System entfernt den VGT-Nachlass automatisch (über Vertragsänderung / Nachtrag → UC-05)
    - System erzeugt eine **Wiedervorlage** für den zuständigen Sachbearbeiter: „VGT-Berechtigung für Vertrag {Vertragsnummer} ist abgelaufen. VGT-Nachlass wurde entfernt. Bitte den Kunden informieren."
    - System löst eine **Neuberechnung** des Beitrags aus

### Phase 5: VGT-Nachlass im Vertrag anzeigen
17. Nach Policierung werden VGT-Nachlässe im Vertragsstand angezeigt – mit dem Hinweis „Versicherungsgewerbetarif"
18. Im policierten Vertrag ist der VGT-Nachlass **schreibgeschützt** – eine Änderung ist nur über eine Vertragsänderung (→ UC-05) möglich

## Alternativszenarien

- **A1: VGT-Nachlass bei Vertragsänderung (Nachtrag) hinzufügen oder entfernen**
  Im Rahmen einer Vertragsänderung (→ UC-05) kann der VGT-Nachlass nachträglich hinzugefügt werden (z. B. wenn der VN während der Vertragslaufzeit in die Versicherungswirtschaft eintritt) oder entfernt werden (z. B. bei Branchenwechsel). Der Ablauf entspricht dem Hauptszenario, bezogen auf das Änderungsobjekt.

- **A2: Partner hat bereits einen VGT-Nachlass auf einem anderen Vertrag**
  Das System prüft nicht, ob der Partner bereits auf einem anderen Vertrag einen VGT-Nachlass erhält. Der VGT-Nachlass ist ein vertragsbezogener Nachlass und kann auf jedem Vertrag des Partners vergeben werden, solange die VGT-Berechtigung gültig ist.

- **A3: VGT-Berechtigung wird nachträglich am Partner entfernt**
  Wird die VGT-Berechtigung am Partner gelöscht oder auf „ungültig" gesetzt, zeigt das System bei der nächsten Bearbeitung eines betroffenen Angebots/Antrags einen Warnhinweis: „Die VGT-Berechtigung des Versicherungsnehmers ist nicht mehr gültig. Der VGT-Nachlass wird bei der nächsten Beitragsberechnung nicht mehr berücksichtigt." Bei bestehenden Verträgen greift die jährliche Batch-Prüfung (→ Phase 4).

- **A4: VGT-Nachlasshöhe wird in der Konfiguration geändert**
  Wird der konfigurierte VGT-Nachlasssatz über UC-03 geändert, wirkt die Änderung nur auf **neue** Angebote und Anträge. Bestehende Verträge behalten den zum Zeitpunkt der Policierung gültigen VGT-Nachlasssatz bei. Eine Anpassung bestehender Verträge erfolgt nur über eine explizite Vertragsänderung (→ UC-05) oder einen Massenänderungslauf.

- **A5: VGT-Nachlass zusammen mit anderen Nachlässen**
  Der VGT-Nachlass ist mit anderen Nachlässen (z. B. Bündelrabatt, Treuerabatt) kombinierbar. Die Kumulierung unterliegt den allgemeinen Plausibilitätsprüfungen (GR-P05, GR-P06). Der VGT-Nachlass wird in der Berechnungsreihenfolge wie ein regulärer Vertragsnachlass (prozentual) behandelt (→ GR-P07).

- **A6: Pensionierte Mitarbeitende**
  Pensionierte Mitarbeitende der Versicherungswirtschaft behalten ihre VGT-Berechtigung in der Regel bei. Das Beschäftigungsverhältnis wird auf `PENSIONIERT` gesetzt; die VGT-Berechtigung bleibt unbefristet gültig (`vgt_gueltig_bis = NULL`).

## Fehlerfälle

- **F1: Partner ist nicht VGT-berechtigt**
  Benutzer versucht, einen VGT-Nachlass (Nachlassart `VGT_NACHLASS`) zu vergeben, aber der Partner ist nicht als VGT-berechtigt hinterlegt. → System zeigt Fehler: „Der Versicherungsnehmer ist nicht als VGT-berechtigt hinterlegt. Bitte erfassen Sie zunächst die VGT-Berechtigung im Partnerprofil."

- **F2: VGT-Berechtigung zum Vertragsbeginn nicht gültig**
  Die VGT-Berechtigung des Partners ist zum Vertragsbeginn noch nicht oder nicht mehr gültig. → System zeigt Fehler: „Die VGT-Berechtigung ist zum Vertragsbeginn ({Datum}) nicht gültig. VGT-Nachlass kann nicht vergeben werden."

- **F3: VGT-Nachlass bereits vorhanden**
  Es existiert bereits ein Nachlass mit Nachlassart `VGT_NACHLASS` auf dem Angebot/Antrag/Vertragsstand. → System zeigt Fehler: „Es ist bereits ein VGT-Nachlass vorhanden. Pro Vertrag ist nur ein VGT-Nachlass zulässig."

- **F4: Kumulierte Nachlässe unterschreiten Mindestbeitrag**
  Der VGT-Nachlass in Kombination mit anderen Nachlässen führt dazu, dass der Mindestbeitrag unterschritten wird. → System zeigt Fehler: „Der Gesamtbeitrag darf nicht unter den Mindestbeitrag von {X} EUR fallen." (analog UC-10, F2)

- **F5: VGT-Nachlasssatz nicht konfiguriert**
  Für die gewählte Sparte ist kein VGT-Nachlasssatz konfiguriert. → System zeigt Fehler: „Für die Sparte {Sparte} ist kein Versicherungsgewerbetarif konfiguriert. Bitte wenden Sie sich an die Fachadministration."

- **F6: Kompetenz nicht vorhanden**
  Benutzer besitzt nicht die Kompetenz `VGT_NACHLASS_BEARBEITEN`. → System zeigt Hinweis: „Keine Berechtigung für die Bearbeitung des Versicherungsgewerbetarifs."

- **F7: Pflichtdaten der VGT-Berechtigung fehlen**
  Benutzer versucht, die VGT-Berechtigung am Partner zu speichern, ohne alle Pflichtfelder auszufüllen. → System zeigt Fehler: „Bitte füllen Sie alle Pflichtfelder der VGT-Berechtigung aus (Arbeitgeber, Beschäftigungsverhältnis, Nachweis-Referenz)."

## Geschäftsregeln

| Regel-ID | Beschreibung | Quelle |
|----------|-------------|--------|
| GR-P13 | Der Versicherungsgewerbetarif (VGT) wird als prozentualer Nachlass auf Vertragsebene (Nachlassart `VGT_NACHLASS`) abgebildet. Die Nachlasshöhe ist pro Sparte konfigurierbar (Standard: 15 %). Pro Angebot/Antrag/Vertragsstand ist maximal ein VGT-Nachlass zulässig | UC-11 |
| GR-P14 | Die VGT-Berechtigung wird am Partner gespeichert und umfasst: Arbeitgeber, Beschäftigungsverhältnis (ANGESTELLT, AUSZUBILDEND, RUHEND, PENSIONIERT), Nachweis-Referenz und Gültigkeitszeitraum. Die Berechtigung wird jährlich zum Hauptfälligkeitstermin des Vertrags automatisch geprüft. Bei Ablauf wird der VGT-Nachlass entfernt und eine Wiedervorlage erzeugt | UC-11 |
| GR-A13 | Bei der Angebotserstellung für einen VGT-berechtigten Partner wird der VGT-Nachlass automatisch vergeben, sofern die Berechtigung zum Vertragsbeginn gültig ist und die Nachlasshöhe für die Sparte konfiguriert ist. Der Benutzer wird per Hinweis über die automatische Vergabe informiert | UC-11 |

## Daten (Ein-/Ausgabe)

### Eingabedaten – VGT-Berechtigung (am Partner)

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| VGT-berechtigt | Boolean | ✅ | – | Ist der Partner VGT-berechtigt? |
| Arbeitgeber | String(200) | ✅ | Mindestens 3 Zeichen | Name des Versicherungsunternehmens oder Vermittlerbetriebs |
| Beschäftigungsverhältnis | Enum | ✅ | `ANGESTELLT`, `AUSZUBILDEND`, `RUHEND`, `PENSIONIERT` | Art des Beschäftigungsverhältnisses |
| Nachweis-Referenz | String(200) | ✅ | Mindestens 5 Zeichen | Referenz auf den VGT-Nachweis (z. B. Bescheinigung, Mitgliedsausweis) |
| VGT gültig ab | Date | ✅ | ≥ Partner-Erfassungsdatum | Beginn der VGT-Berechtigung |
| VGT gültig bis | Date | ❌ | > VGT gültig ab (falls angegeben) | Ende der VGT-Berechtigung (NULL = unbefristet) |

### Eingabedaten – VGT-Nachlass (am Angebot/Antrag)

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| Nachlassart | Enum | ✅ | `VGT_NACHLASS` (vorbelegt) | Fachliche Kategorisierung |
| Wert | BigDecimal(10,4) | ✅ | > 0; ≤ konfiguriertes VGT-Maximum | Prozentualer VGT-Nachlasssatz (Standard aus Konfiguration) |
| Begründung | String(500) | ✅ | Mindestens 10 Zeichen (wird bei automatischer Vergabe systemseitig befüllt) | Fachliche Begründung |

### Ausgabedaten

| Feld | Typ | Beschreibung |
|------|-----|-------------|
| VGT-Nachlass-ID | UUID | Systemseitig vergebene eindeutige Kennung |
| Nachlassart | String | `VGT_NACHLASS` |
| Wert | BigDecimal | Prozentualer Nachlasssatz |
| Berechneter Effekt | BigDecimal | Auswirkung auf den Beitrag in EUR |
| VGT-Berechtigungsstatus | String | Gültig / Abgelaufen / Nicht vorhanden |
| Arbeitgeber | String | Name des Arbeitgebers (aus Partnerprofil) |
| Beschäftigungsverhältnis | String | Art der Beschäftigung |
| Nächste VGT-Prüfung | Date | Datum der nächsten jährlichen Berechtigungsprüfung |

## Kalkulationsbeispiel

```
Partner: Max Mustermann (VGT-berechtigt, Arbeitgeber: LVM Versicherung, angestellt)
Sparte: KFZ
VGT-Nachlasssatz (konfiguriert): 15 %

Produkt KFZ-HP:
  Basisbeitrag:                        280,00 EUR
  × Scoring-Faktoren (SF5, Typ15, …):   × 0,5446
  = Nettobeitrag Produkt:              152,49 EUR

Produkt KFZ-TK:
  Basisbeitrag:                        100,00 EUR
  × Scoring-Faktoren (Typ20, …):        × 1,4355
  = Nettobeitrag Produkt:              143,55 EUR

Summe Nettobeiträge:                   296,04 EUR
  − VGT-Nachlass 15 % (Gewerbetarif):  − 44,41 EUR
  = Gesamtnettobeitrag:                251,63 EUR

  + Versicherungssteuer 19 %:          + 47,81 EUR
  = Jahresbeitrag brutto:              299,44 EUR
  × Zahlungsweisefaktor (jährlich):      × 1,0000
  = Zahlbeitrag:                       299,44 EUR
```

### Kalkulationsbeispiel mit Kombination VGT + Risikonachlass

```
Produkt KFZ-HP:
  Nettobeitrag Produkt:               152,49 EUR
  − Risikonachlass 10 % (Treuerabatt): − 15,25 EUR
  = Nettobeitrag nach Risikonachlass: 137,24 EUR

Produkt KFZ-TK:
  Nettobeitrag Produkt:               143,55 EUR
  (kein Risikonachlass)

Summe Nettobeiträge (nach Risiko):    280,79 EUR
  − VGT-Nachlass 15 % (Gewerbetarif):  − 42,12 EUR
  = Gesamtnettobeitrag:               238,67 EUR

  + Versicherungssteuer 19 %:         + 45,35 EUR
  = Jahresbeitrag brutto:             284,02 EUR
  × Zahlungsweisefaktor (jährlich):     × 1,0000
  = Zahlbeitrag:                      284,02 EUR
```

> **Hinweis:** Der VGT-Nachlass wird in der Berechnungsreihenfolge (GR-P07) als prozentualer Vertragsnachlass behandelt und vor absoluten Vertragsnachlässen angewendet.

## Nachbedingungen
- Die VGT-Berechtigung ist am Partner gespeichert und revisionssicher protokolliert
- Der VGT-Nachlass ist im Angebot/Antrag/Vertragsstand als Nachlass auf Vertragsebene (Nachlassart `VGT_NACHLASS`) gespeichert
- Der Beitrag ist unter Berücksichtigung des VGT-Nachlasses korrekt berechnet
- Bei Policierung: Der VGT-Nachlass ist im Vertragsstand übernommen und schreibgeschützt
- Bei abgelaufener VGT-Berechtigung: Der VGT-Nachlass ist entfernt und eine Wiedervorlage ist erzeugt

## Akzeptanzkriterien
- [ ] Die VGT-Berechtigung kann am Partner mit Arbeitgeber, Beschäftigungsverhältnis, Nachweis-Referenz und Gültigkeitszeitraum erfasst werden
- [ ] Bei Angebotserstellung für einen VGT-berechtigten Partner wird der VGT-Nachlass automatisch auf Vertragsebene vergeben
- [ ] Der Benutzer wird per Hinweis über die automatische VGT-Nachlassvergabe informiert
- [ ] Der VGT-Nachlasssatz ist pro Sparte konfigurierbar (über UC-03)
- [ ] Pro Angebot/Antrag/Vertragsstand ist maximal ein VGT-Nachlass zulässig
- [ ] Ein VGT-Nachlass kann nur vergeben werden, wenn der Partner als VGT-berechtigt hinterlegt ist
- [ ] Die VGT-Berechtigung wird zum Vertragsbeginn auf Gültigkeit geprüft
- [ ] Der VGT-Nachlass wird bei der Beitragskalkulation korrekt als prozentualer Vertragsnachlass berücksichtigt
- [ ] Der VGT-Nachlass ist mit anderen Nachlässen (Bündelrabatt, Treuerabatt etc.) kombinierbar unter Beachtung von GR-P05 und GR-P06
- [ ] Die jährliche Batch-Prüfung erkennt abgelaufene VGT-Berechtigungen und entfernt den VGT-Nachlass automatisch
- [ ] Bei Entfernung des VGT-Nachlasses durch den Batch-Lauf wird eine Wiedervorlage für den Sachbearbeiter erzeugt
- [ ] Die Kompetenz `VGT_NACHLASS_BEARBEITEN` wird korrekt geprüft
- [ ] Alle Änderungen an VGT-Berechtigungen und VGT-Nachlässen werden revisionssicher historisiert (Hibernate Envers)
- [ ] Die Funktion ist spartenübergreifend verfügbar und für alle konfigurierten Sparten nutzbar
- [ ] Pensionierte Mitarbeitende behalten ihre VGT-Berechtigung bei

## Wireframe / Skizze

### VGT-Berechtigung im Partnerprofil

```
┌───────────────────────────────────────────────────────────────────┐
│  Partner: Max Mustermann (KD-2026-004711)                        │
├───────────────────────────────────────────────────────────────────┤
│  … (Adressdaten, Kontakt) …                                     │
├───────────────────────────────────────────────────────────────────┤
│  Versicherungsgewerbetarif (VGT)                                 │
│  ┌───────────────────────────────────────────────────────────┐   │
│  │  ☑ VGT-berechtigt                                        │   │
│  │                                                           │   │
│  │  Arbeitgeber:              [LVM Versicherung           ]  │   │
│  │  Beschäftigungsverhältnis: [▼ Angestellt               ]  │   │
│  │  Nachweis-Referenz:        [Bescheinigung HR-2026-0042 ]  │   │
│  │  Gültig ab:                [01.03.2020                 ]  │   │
│  │  Gültig bis:               [                           ]  │   │
│  │                            (leer = unbefristet)           │   │
│  └───────────────────────────────────────────────────────────┘   │
│  [Speichern]                                                     │
└───────────────────────────────────────────────────────────────────┘
```

### VGT-Nachlass im Angebot (automatisch vergeben)

```
┌───────────────────────────────────────────────────────────────────┐
│  Angebot #AG-2026-001234          Status: ⬤ Berechnet            │
├───────────────────────────────────────────────────────────────────┤
│  ℹ️ VGT-Nachlass von 15 % wurde automatisch vergeben             │
│     (Versicherungsgewerbetarif – LVM Versicherung, Angestellt)   │
├───────────────────────────────────────────────────────────────────┤
│  Nachlässe & Zuschläge                                           │
│  ┌───────────────────────────────────────────────────────────┐   │
│  │  Vertragsebene                                            │   │
│  │  ┌─────────────────────────────────────────────────────┐  │   │
│  │  │ 🏢 Gewerbetarif (VGT-Nachlass)  −15,00 % │ ✏️  🗑️ │  │   │
│  │  │   Gültig: 01.01.2027 – unbefristet                 │  │   │
│  │  │   Begründung: Versicherungsgewerbetarif –           │  │   │
│  │  │               LVM Versicherung, Angestellt          │  │   │
│  │  │   Effekt: −44,41 EUR                               │  │   │
│  │  └─────────────────────────────────────────────────────┘  │   │
│  │  [+ Nachlass hinzufügen]  [+ Zuschlag hinzufügen]        │   │
│  └───────────────────────────────────────────────────────────┘   │
├───────────────────────────────────────────────────────────────────┤
│  Berechneter Jahresbeitrag:                     € 299,44         │
│  ├─ KFZ-Haftpflicht:         € 152,49            = € 152,49     │
│  ├─ KFZ-Teilkasko:           € 143,55            = € 143,55     │
│  ├─ Summe Netto:                                  € 296,04      │
│  ├─ VGT-Nachlass (Gewerbetarif −15 %):             −€ 44,41     │
│  ├─ Gesamtnetto:                                  € 251,63      │
│  ├─ Versicherungssteuer 19 %:                      +€ 47,81     │
│  └─ Zahlbeitrag (jährlich):                       € 299,44      │
└───────────────────────────────────────────────────────────────────┘
```

## Offene Fragen
- Soll die VGT-Nachlasshöhe pro Sparte unterschiedlich sein, oder ein einheitlicher Satz über alle Sparten?
- Wie wird der VGT-Nachweis konkret erbracht (Digitaler Upload, manuelle Prüfung durch Innendienst)?
- Soll eine automatische Schnittstelle zu einem Arbeitgeber-/Branchenverzeichnis eingebunden werden, um die VGT-Berechtigung zu validieren?
- Ist der VGT-Nachlass auch für Vermittler (Makler, Agenten) verfügbar, oder nur für Angestellte von Versicherungsunternehmen?
- Sollen Familienangehörige (z. B. Ehepartner, Kinder) ebenfalls VGT-berechtigt sein?
- Muss der VGT-Nachlass mit dem Zielbeitragsmodell (UC-10, Phase 3b) interagieren, oder gilt er unabhängig davon?
- Gibt es eine Übergangsfrist nach Ausscheiden aus der Versicherungswirtschaft, in der der VGT-Nachlass noch gewährt wird?
