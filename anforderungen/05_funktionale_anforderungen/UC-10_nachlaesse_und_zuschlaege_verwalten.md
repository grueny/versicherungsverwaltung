# UC-10: Nachlässe und Zuschläge verwalten

## Beschreibung
Benutzer können **Nachlässe** (Rabatte) und **Zuschläge** (Aufschläge) auf **Vertragsebene** und **Risikoebene** (= Produktebene) anlegen, bearbeiten und entfernen. Nachlässe und Zuschläge sind ab dem Status **Angebot** bis zur **Schwebe** bearbeitbar. Im policierten **Vertrag** werden sie nur noch angezeigt – eine Änderung ist dort ausschließlich über eine Vertragsänderung (→ UC-05) möglich.

Bei der **Beitragskalkulation** werden alle aktiven Nachlässe und Zuschläge automatisch berücksichtigt: Nachlässe reduzieren den berechneten Beitrag, Zuschläge erhöhen ihn. Die Berechnung erfolgt auf Basis des Nettobeitrags (nach Scoring-Faktor-Berechnung, vor Versicherungssteuer).

Zusätzlich bietet die **Zielbeitragsfunktion** die Möglichkeit, einen gewünschten **Gesamtbeitrag (brutto, p.a.)** vorzugeben. Das System ermittelt daraus automatisch den erforderlichen prozentualen Nachlass oder Zuschlag auf Vertragsebene, um den Zielbeitrag zu erreichen. So kann der Vertrieb dem Kunden direkt einen Wunschbeitrag anbieten, ohne den Nachlass manuell berechnen zu müssen.

Dies ist ein **spartenübergreifender** Kernprozess. Nachlässe und Zuschläge gelten für alle Sparten gleich. Spartenspezifische Nachlass- und Zuschlagsarten können über die Spartenkonfiguration ergänzt werden.

## Akteure
- **Primär:** Außendienst (Vertrieb), Innendienst (Sachbearbeitung)
- **Sekundär:** System (automatische Anwendung bei Kalkulation)

## Vorbedingungen
- Ein Angebot, Antrag oder eine Schwebe existiert in einem bearbeitbaren Status (Angebot: `ENTWURF`, `BERECHNET`; Antrag: `OFFEN`, `GEPRUEFT`; Schwebe: `OFFEN`)
- Benutzer ist am System angemeldet und besitzt die Kompetenz **`NACHLASS_ZUSCHLAG_BEARBEITEN`** (→ Kompetenz-System S8)
- Für Nachlässe, die konfigurierte Maximalwerte überschreiten, ist zusätzlich die Kompetenz **`NACHLASS_ZUSCHLAG_ERWEITERT`** erforderlich

## Auslöser
- Benutzer öffnet ein Angebot, einen Antrag oder eine Schwebe und navigiert zum Bereich „Nachlässe & Zuschläge"
- Benutzer wählt „Nachlass hinzufügen" oder „Zuschlag hinzufügen" im Kontext eines Angebots, Antrags oder einer Schwebe
- Benutzer wählt „Zielbeitrag vorgeben" im Kontext eines Angebots, Antrags oder einer Schwebe

## Ablauf (Hauptszenario)

### Phase 1: Nachlässe und Zuschläge einsehen
1. Benutzer öffnet ein Angebot, einen Antrag oder eine Schwebe
2. System zeigt den Bereich **„Nachlässe & Zuschläge"** mit allen bereits erfassten Einträgen an:
   - **Vertragsebene:** Nachlässe/Zuschläge, die auf den Gesamtbeitrag wirken
   - **Risikoebene (Produkt):** Nachlässe/Zuschläge, die auf den Beitrag eines einzelnen Produkts wirken
3. Für jeden Eintrag werden angezeigt: Art (Nachlass/Zuschlag), Bezeichnung, Ebene (Vertrag/Risiko), Bezugsprodukt (bei Risikoebene), Wert (prozentual oder absolut), Gültigkeitszeitraum, Begründung

### Phase 2: Nachlass oder Zuschlag hinzufügen
4. Benutzer wählt **„Nachlass hinzufügen"** oder **„Zuschlag hinzufügen"**
5. Benutzer wählt die **Ebene**:
   - **Vertragsebene:** Der Nachlass/Zuschlag wirkt auf den Gesamtnettobeitrag aller Produkte
   - **Risikoebene (Produkt):** Benutzer wählt das betroffene Produkt aus der Liste der im Angebot/Antrag enthaltenen Produkte
6. Benutzer wählt die **Nachlassart / Zuschlagsart** aus der konfigurierten Liste (→ Spartenkonfiguration oder spartenübergreifende Konfiguration):
   - Beispiele: Bündelrabatt, Treuerabatt, Sondernachlass, Gefahrenerhöhung, Sonderzuschlag, Schadenverlauf
7. Benutzer gibt den **Wert** ein:
   - **Prozentualer Wert:** z. B. 10 % Nachlass → Wirkung auf den Nettobeitrag
   - **Absoluter Wert:** z. B. 50,00 EUR Nachlass → Abzug vom Nettobeitrag
8. Benutzer gibt optional einen **Gültigkeitszeitraum** an:
   - **Gültig ab:** Datum, ab dem der Nachlass/Zuschlag wirkt (Standard: Vertragsbeginn)
   - **Gültig bis:** Datum, bis zu dem der Nachlass/Zuschlag wirkt (optional; NULL = unbefristet)
9. Benutzer gibt eine **Begründung** ein (Pflicht, mindestens 10 Zeichen)
10. System führt **Plausibilitätsprüfungen** durch (→ Geschäftsregeln GR-P03 bis GR-P08):
    - Maximaler prozentualer Nachlass pro Ebene nicht überschritten?
    - Kumulierter Nachlass aller Einträge nicht unter Mindestbeitrag?
    - Keine widersprüchlichen Einträge (gleiche Art auf gleicher Ebene)?
    - Kompetenzprüfung bei Überschreitung konfigurierter Schwellwerte
11. Bei erfolgreicher Prüfung: Nachlass/Zuschlag wird gespeichert
12. System löst automatisch eine **Neuberechnung** des Beitrags aus (→ Phase 4)

### Phase 3: Nachlass oder Zuschlag bearbeiten / entfernen
13. Benutzer kann einen bestehenden Nachlass/Zuschlag **bearbeiten** (Wert, Gültigkeitszeitraum, Begründung ändern) oder **entfernen**
14. Bei Bearbeitung: Plausibilitätsprüfungen werden erneut durchgeführt (wie Phase 2, Schritte 10–11)
15. Bei Entfernung: System fragt „Nachlass/Zuschlag wirklich entfernen?" → Bei Bestätigung wird der Eintrag entfernt
16. System löst automatisch eine **Neuberechnung** des Beitrags aus

### Phase 3b: Zielbeitrag vorgeben (Beitragswunsch)
17. Benutzer wählt die Funktion **„Zielbeitrag vorgeben"**
18. System zeigt den aktuellen **berechneten Jahresbeitrag brutto** als Referenzwert an
19. Benutzer gibt den gewünschten **Zielbeitrag** (Jahresbeitrag brutto in EUR) ein
20. Benutzer gibt eine **Begründung** ein (Pflicht, mindestens 10 Zeichen)
21. System führt eine **Rückrechnung** durch:
    - Aus dem Zielbeitrag brutto wird der erforderliche Gesamtnettobeitrag ermittelt (Zielbeitrag ÷ (1 + Versicherungssteuersatz) ÷ Zahlungsweisefaktor)
    - Der Gesamtnettobeitrag vor Vertragsnachlass/-zuschlag wird aus den aktuellen Produktbeiträgen (nach bestehenden Risikonachlässen/-zuschlägen) ermittelt
    - Die Differenz zwischen Ist-Nettobeitrag und Ziel-Nettobeitrag bestimmt den erforderlichen prozentualen Nachlass oder Zuschlag auf Vertragsebene
    - **Zielbeitrag < aktueller Beitrag** → System erzeugt einen **Nachlass** auf Vertragsebene (Nachlassart: `ZIELBEITRAG_NACHLASS`)
    - **Zielbeitrag > aktueller Beitrag** → System erzeugt einen **Zuschlag** auf Vertragsebene (Zuschlagsart: `ZIELBEITRAG_ZUSCHLAG`)
    - **Zielbeitrag = aktueller Beitrag** → Kein Nachlass/Zuschlag erforderlich; System zeigt Hinweis: „Der Zielbeitrag entspricht dem aktuellen Beitrag."
22. System führt die Standard-**Plausibilitätsprüfungen** durch (→ GR-P05, GR-P06, GR-P11, GR-P12):
    - Ermittelter Nachlass überschreitet den konfigurierten Maximalwert? → Kompetenzprüfung oder Fehler
    - Zielbeitrag liegt unter dem Mindestbeitrag? → Fehler
    - Zielbeitrag ist negativ oder null? → Fehler
23. Bei erfolgreicher Prüfung: Der automatisch ermittelte Nachlass/Zuschlag wird als neuer Eintrag auf Vertragsebene gespeichert. Ein bestehender `ZIELBEITRAG_NACHLASS` oder `ZIELBEITRAG_ZUSCHLAG` wird dabei **ersetzt** (nicht kumuliert)
24. System löst eine **Neuberechnung** des Beitrags aus; der berechnete Jahresbeitrag brutto muss dem eingegebenen Zielbeitrag entsprechen (Rundungsdifferenz ≤ 0,01 EUR zulässig)

### Phase 4: Kalkulation mit Nachlässen und Zuschlägen
25. System berechnet den Beitrag unter Berücksichtigung aller aktiven Nachlässe und Zuschläge:
    - **Risikoebene (Produkt):** Für jedes Produkt wird der Nettobeitrag (Basisbeitrag × Scoring-Faktoren) um die produktspezifischen Nachlässe/Zuschläge angepasst
    - **Vertragsebene:** Auf die Summe aller Produktnettobeiträge (nach Risikonachlässen) werden die Vertragsnachlässe/-zuschläge angewendet (einschließlich Zielbeitrags-Nachlass/-Zuschlag)
    - **Reihenfolge:** Prozentuale Nachlässe/Zuschläge werden vor absoluten angewendet
26. System zeigt die angepassten Beitragsdetails an, einschließlich der Aufschlüsselung aller Nachlässe und Zuschläge
27. Die Nachlass-/Zuschlagsinformationen werden in den `berechnungsdetails` (JSONB) der jeweiligen Produkt-Zuordnung (AngebotProdukt / AntragProdukt) persistiert

### Phase 5: Anzeige im Vertrag (nach Policierung)
28. Nach erfolgreicher Policierung (→ UC-05 Schwebe policieren) werden die Nachlässe und Zuschläge im **Vertragsstand** übernommen und angezeigt
29. Im policierten Vertrag sind Nachlässe und Zuschläge **schreibgeschützt** – eine Änderung ist nur über eine Vertragsänderung (→ UC-05 Nachtrag) möglich

## Alternativszenarien

- **A1: Nachlass/Zuschlag bei Vertragsänderung (Nachtrag) hinzufügen oder ändern**
  Im Rahmen einer Vertragsänderung (→ UC-05) kann der Benutzer Nachlässe und Zuschläge am Änderungsobjekt (Änderungsangebot, Änderungsantrag, Änderungsschwebe) bearbeiten. Der Ablauf ist identisch zum Hauptszenario, allerdings bezogen auf das Änderungsobjekt. Bei Policierung des Nachtrags werden die geänderten Nachlässe/Zuschläge im neuen Vertragsstand übernommen.

- **A2: Befristeter Nachlass läuft automatisch aus**
  Ein Nachlass mit `gueltig_bis`-Datum wird nach Ablauf automatisch bei der nächsten Beitragsberechnung (z. B. zum Hauptfälligkeitstermin) nicht mehr berücksichtigt. Das System protokolliert den Ablauf und informiert den Sachbearbeiter über die Beitragsänderung.

- **A3: Nachlass/Zuschlag bei Risikoentfernung**
  Wird ein Produkt (Risiko) aus dem Angebot/Antrag entfernt, werden alle zugehörigen Nachlässe und Zuschläge auf Risikoebene automatisch mit entfernt. System zeigt einen Hinweis: „Produktspezifische Nachlässe/Zuschläge wurden entfernt."

- **A4: Nachlass/Zuschlag kopieren bei Angebot → Antrag**
  Bei der Überführung eines Angebots in einen Antrag (→ UC-01) werden alle Nachlässe und Zuschläge in den Antrag übernommen (Snapshot-Prinzip, analog GR-V01). Im Antrag sind sie weiterhin bearbeitbar.

- **A5: Zielbeitrag bei bestehendem manuellen Vertragsnachlass**
  Existieren auf Vertragsebene bereits manuelle Nachlässe oder Zuschläge (die nicht aus einem Zielbeitrag resultieren), werden diese bei der Zielbeitragsermittlung **beibehalten**. Der Zielbeitrags-Nachlass/-Zuschlag wird **zusätzlich** zu den bestehenden manuellen Einträgen berechnet. Das System berücksichtigt die bestehenden manuellen Vertragsnachlässe bei der Rückrechnung, sodass der Zielbeitrag exakt getroffen wird.

- **A6: Zielbeitrag erneut anpassen**
  Der Benutzer kann den Zielbeitrag jederzeit erneut über „Zielbeitrag vorgeben" anpassen. Der bestehende `ZIELBEITRAG_NACHLASS` oder `ZIELBEITRAG_ZUSCHLAG` wird dabei durch den neu berechneten Wert ersetzt. Wird der Zielbeitrags-Nachlass/-Zuschlag manuell entfernt, kehrt der Beitrag zum kalkulierten Wert (ohne Zielbeitragsanpassung) zurück.

- **A7: Zielbeitrag mit Neuberechnung nach Datenänderung**
  Werden nach der Zielbeitragsvorgabe Vertrags- oder Risikodaten geändert (z. B. Produkt hinzugefügt, Tarifmerkmale geändert), ändert sich der Basisbeitrag. Der bestehende Zielbeitrags-Nachlass/-Zuschlag bleibt erhalten, aber der tatsächliche Jahresbeitrag weicht dann vom ursprünglichen Zielbeitrag ab. System zeigt einen **Warnhinweis**: „Der Zielbeitrag von {X} EUR wird aufgrund geänderter Vertragsdaten nicht mehr exakt erreicht. Aktueller Beitrag: {Y} EUR. Bitte Zielbeitrag erneut anpassen." Der Benutzer kann den Zielbeitrag erneut vorgeben (→ Phase 3b).

## Fehlerfälle

- **F1: Maximaler Nachlass überschritten**
  Der konfigurierte Maximalnachlass (prozentual oder absolut) wird überschritten. → System zeigt Fehler: „Der maximale Nachlass von {X} % / {X} EUR für diese Ebene wurde überschritten." Speicherung wird blockiert, sofern der Benutzer nicht die Kompetenz `NACHLASS_ZUSCHLAG_ERWEITERT` besitzt.

- **F2: Kumulierter Nachlass unterschreitet Mindestbeitrag**
  Die Summe aller Nachlässe führt dazu, dass der Beitrag unter den konfigurierten Mindestbeitrag fällt. → System zeigt Fehler: „Der Gesamtbeitrag darf nicht unter den Mindestbeitrag von {X} EUR fallen."

- **F3: Kompetenz nicht vorhanden**
  Benutzer besitzt nicht die erforderliche Kompetenz. → System zeigt Hinweis: „Keine Berechtigung für diese Aktion."

- **F4: Bearbeitung im nicht-bearbeitbaren Status**
  Benutzer versucht, Nachlässe/Zuschläge in einem policierten Vertrag direkt zu bearbeiten. → System zeigt Hinweis: „Nachlässe und Zuschläge können im Vertrag nur über eine Vertragsänderung bearbeitet werden."

- **F5: Pflichtfeld Begründung fehlt**
  Benutzer gibt keine oder eine zu kurze Begründung ein. → System zeigt Fehler: „Bitte geben Sie eine Begründung ein (mindestens 10 Zeichen)."

- **F6: Zielbeitrag unter Mindestbeitrag**
  Der eingegebene Zielbeitrag liegt unter dem konfigurierten Mindestbeitrag (brutto). → System zeigt Fehler: „Der Zielbeitrag von {X} EUR liegt unter dem zulässigen Mindestbeitrag von {Y} EUR."

- **F7: Zielbeitrag erfordert unzulässig hohen Nachlass**
  Der zum Erreichen des Zielbeitrags erforderliche Nachlass überschreitet den konfigurierten Maximalwert und der Benutzer besitzt nicht die Kompetenz `NACHLASS_ZUSCHLAG_ERWEITERT`. → System zeigt Fehler: „Der erforderliche Nachlass von {X} % überschreitet den zulässigen Maximalwert von {Y} %. Bitte einen höheren Zielbeitrag eingeben oder die erweiterte Kompetenz anfordern."

- **F8: Zielbeitrag negativ oder null**
  Benutzer gibt einen Zielbeitrag ≤ 0 EUR ein. → System zeigt Fehler: „Der Zielbeitrag muss größer als 0 EUR sein."

- **F9: Kein kalkulierter Beitrag vorhanden**
  Es wurde noch keine Beitragsberechnung durchgeführt (kein aktueller Nettobeitrag vorhanden). → System zeigt Fehler: „Bitte führen Sie zuerst eine Beitragsberechnung durch, bevor Sie einen Zielbeitrag vorgeben."

## Geschäftsregeln

| Regel-ID | Beschreibung | Quelle |
|----------|-------------|--------|
| GR-P03 | Nachlässe und Zuschläge können auf Vertragsebene (Gesamtbeitrag) und auf Risikoebene (Einzelprodukt) vergeben werden. Beide Ebenen werden bei der Kalkulation berücksichtigt | UC-10 |
| GR-P04 | Nachlässe und Zuschläge sind ab Angebot (Status ENTWURF) bis Schwebe (Status OFFEN) bearbeitbar. Im policierten Vertrag werden sie nur angezeigt; Änderungen erfordern eine Vertragsänderung (UC-05) | UC-10 |
| GR-P05 | Der maximale prozentuale Nachlass pro Ebene ist konfigurierbar (Standard: 30 % auf Vertragsebene, 50 % auf Risikoebene). Überschreitung erfordert die Kompetenz `NACHLASS_ZUSCHLAG_ERWEITERT` | UC-10 |
| GR-P06 | Die Summe aller Nachlässe darf nicht dazu führen, dass der berechnete Gesamtbeitrag unter einen konfigurierten Mindestbeitrag (Standard: 1,00 EUR netto p.a.) fällt | UC-10 |
| GR-P07 | Bei der Kalkulation werden Nachlässe und Zuschläge in folgender Reihenfolge angewendet: (1) Risikoebene prozentual, (2) Risikoebene absolut, (3) Vertragsebene prozentual, (4) Vertragsebene absolut – jeweils vor Versicherungssteuer und Zahlungsweiseaufschlag | UC-10 |
| GR-P08 | Jeder Nachlass und Zuschlag erfordert eine Begründung (Pflichtfeld, mindestens 10 Zeichen). Alle Änderungen an Nachlässen und Zuschlägen werden revisionssicher historisiert (Hibernate Envers) | UC-10 |
| GR-P09 | Bei der Überführung Angebot → Antrag (UC-01) und Antrag → Schwebe (UC-02) werden alle Nachlässe und Zuschläge als Snapshot übernommen | UC-10 |
| GR-P10 | Befristete Nachlässe (mit `gueltig_bis`-Datum) werden nach Ablauf bei der nächsten Beitragsberechnung automatisch nicht mehr berücksichtigt | UC-10 |
| GR-P11 | Über die Zielbeitragsfunktion kann ein gewünschter Jahresbeitrag brutto vorgegeben werden. Das System ermittelt automatisch den erforderlichen prozentualen Nachlass oder Zuschlag auf Vertragsebene (Nachlassart `ZIELBEITRAG_NACHLASS` bzw. `ZIELBEITRAG_ZUSCHLAG`). Ein bestehender Zielbeitrags-Nachlass/-Zuschlag wird bei erneuter Vorgabe ersetzt, nicht kumuliert | UC-10 |
| GR-P12 | Die Zielbeitragsermittlung unterliegt denselben Plausibilitätsprüfungen wie manuell erfasste Nachlässe (GR-P05, GR-P06). Der Zielbeitrag muss > 0 EUR und ≥ Mindestbeitrag (brutto) sein. Der ermittelte Nachlass muss innerhalb der konfigurierten Maximalwerte liegen oder die erweiterte Kompetenz `NACHLASS_ZUSCHLAG_ERWEITERT` erfordern | UC-10 |

## Daten (Ein-/Ausgabe)

### Eingabedaten

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| Art | Enum | ✅ | `NACHLASS` oder `ZUSCHLAG` | Handelt es sich um einen Nachlass oder Zuschlag? |
| Ebene | Enum | ✅ | `VERTRAG` oder `RISIKO` | Wirkt auf Gesamtbeitrag oder auf einzelnes Produkt? |
| Produkt | Referenz (UUID) | Bedingt | Pflicht bei Ebene = RISIKO; Produkt muss im Angebot/Antrag enthalten sein | Betroffenes Produkt bei Risikonachlass/-zuschlag |
| Nachlassart / Zuschlagsart | Enum | ✅ | Muss in der konfigurierten Liste der Sparte oder spartenübergreifend vorhanden sein | Fachliche Kategorisierung (z. B. Bündelrabatt, Treuerabatt, Gefahrenerhöhung) |
| Werttyp | Enum | ✅ | `PROZENTUAL` oder `ABSOLUT` | Art der Wertangabe |
| Wert | BigDecimal(10,2) | ✅ | > 0; bei PROZENTUAL: ≤ konfiguriertes Maximum; bei ABSOLUT: ≤ konfiguriertes Maximum | Höhe des Nachlasses/Zuschlags |
| Gültig ab | Date | ❌ | ≥ Vertragsbeginn | Ab wann wirkt der Nachlass/Zuschlag (Standard: Vertragsbeginn) |
| Gültig bis | Date | ❌ | > Gültig ab (falls angegeben) | Bis wann wirkt der Nachlass/Zuschlag (NULL = unbefristet) |
| Begründung | String(500) | ✅ | Mindestens 10 Zeichen | Fachliche Begründung für den Nachlass/Zuschlag |
| Zielbeitrag | BigDecimal(12,2) | ❌ | > 0; ≥ Mindestbeitrag (brutto); nur bei Funktion „Zielbeitrag vorgeben" | Gewünschter Jahresbeitrag brutto in EUR (→ Phase 3b) |

### Ausgabedaten

| Feld | Typ | Beschreibung |
|------|-----|-------------|
| Nachlass-/Zuschlag-ID | UUID | Systemseitig vergebene eindeutige Kennung |
| Art | Enum | NACHLASS oder ZUSCHLAG |
| Ebene | Enum | VERTRAG oder RISIKO |
| Bezugsprodukt | String | Produktbezeichnung (bei Risikoebene) |
| Nachlassart / Zuschlagsart | String | Fachliche Bezeichnung |
| Werttyp | Enum | PROZENTUAL oder ABSOLUT |
| Wert | BigDecimal | Höhe des Nachlasses/Zuschlags |
| Berechneter Effekt | BigDecimal | Auswirkung auf den Beitrag in EUR |
| Gültig ab | Date | Beginn der Wirksamkeit |
| Gültig bis | Date | Ende der Wirksamkeit (oder „unbefristet") |
| Begründung | String | Fachliche Begründung |
| Beitrag vor Nachlass/Zuschlag | BigDecimal | Beitrag ohne diesen Eintrag |
| Beitrag nach Nachlass/Zuschlag | BigDecimal | Beitrag mit diesem Eintrag |
| Zielbeitrag (Vorgabe) | BigDecimal | Eingegebener Zielbeitrag (falls über Zielbeitragsfunktion erzeugt) |
| Ermittelter Zielbeitrags-Nachlass/-Zuschlag | BigDecimal (%) | Vom System berechneter prozentualer Wert zur Zielerreichung |

## Kalkulationsbeispiel

```
Produkt KFZ-HP:
  Basisbeitrag:                        280,00 EUR
  × Scoring-Faktoren (SF5, Typ15, …):   × 0,5446
  = Nettobeitrag Produkt:              152,49 EUR
  − Risikonachlass 10 % (Treuerabatt): − 15,25 EUR
  = Nettobeitrag nach Risikonachlass:  137,24 EUR

Produkt KFZ-TK:
  Basisbeitrag:                        100,00 EUR
  × Scoring-Faktoren (Typ20, …):        × 1,4355
  = Nettobeitrag Produkt:              143,55 EUR
  (kein Risikonachlass/-zuschlag)

Summe Nettobeiträge (nach Risiko):     280,79 EUR
  − Vertragsnachlass 5 % (Bündelrabatt): − 14,04 EUR
  = Gesamtnettobeitrag:                266,75 EUR

  + Versicherungssteuer 19 %:          + 50,68 EUR
  = Jahresbeitrag brutto:              317,43 EUR
  × Zahlungsweisefaktor (jährlich):      × 1,0000
  = Zahlbeitrag:                       317,43 EUR
```

### Kalkulationsbeispiel Zielbeitrag

```
Ausgangslage (ohne Vertragsnachlass, mit Risikonachlass):
  Summe Nettobeiträge (nach Risiko):     280,79 EUR
  Aktueller Jahresbeitrag brutto:        334,14 EUR
  (= 280,79 × 1,19 = 334,14 EUR)

Benutzer gibt Zielbeitrag vor:           300,00 EUR brutto

Rückrechnung:
  Ziel-Nettobeitrag = 300,00 ÷ 1,19     = 252,10 EUR
  Ist-Nettobeitrag (nach Risiko):          280,79 EUR
  Differenz:                              − 28,69 EUR
  Erforderlicher Nachlass:
    28,69 ÷ 280,79                        = 10,22 %

System erzeugt:
  Vertragsnachlass (ZIELBEITRAG_NACHLASS): −10,22 % auf Vertragsebene

Neuberechnung:
  Summe Nettobeiträge (nach Risiko):     280,79 EUR
  − Vertragsnachlass 10,22 %:           − 28,70 EUR
  = Gesamtnettobeitrag:                  252,09 EUR
  + Versicherungssteuer 19 %:           + 47,90 EUR
  = Jahresbeitrag brutto:               299,99 EUR
    (Rundungsdifferenz ≤ 0,01 EUR → OK)
  × Zahlungsweisefaktor (jährlich):       × 1,0000
  = Zahlbeitrag:                         299,99 EUR
```

## Nachbedingungen
- Nachlass/Zuschlag ist im Angebot, Antrag oder der Schwebe gespeichert und in der Beitragsberechnung berücksichtigt
- Bei Zielbeitragsvorgabe: Der ermittelte Nachlass/Zuschlag auf Vertragsebene ist gespeichert, der berechnete Jahresbeitrag entspricht dem Zielbeitrag (Rundungsdifferenz ≤ 0,01 EUR)
- Die Beitragsdetails enthalten die Aufschlüsselung aller Nachlässe und Zuschläge
- Alle Änderungen sind revisionssicher protokolliert (Benutzer, Zeitstempel)
- Nach Policierung: Nachlässe/Zuschläge sind im Vertragsstand übernommen und schreibgeschützt

## Akzeptanzkriterien
- [ ] Nachlässe und Zuschläge können auf Vertragsebene (Gesamtbeitrag) und Risikoebene (Einzelprodukt) angelegt werden
- [ ] Nachlässe und Zuschläge können prozentual (%) oder absolut (EUR) erfasst werden
- [ ] Die Bearbeitung ist in Angebot, Antrag und Schwebe möglich; im Vertrag sind Einträge nur lesbar
- [ ] Bei jeder Änderung eines Nachlasses/Zuschlags wird automatisch eine Neuberechnung des Beitrags ausgelöst
- [ ] Die Beitragsberechnung berücksichtigt alle aktiven Nachlässe und Zuschläge korrekt in der definierten Reihenfolge (GR-P07)
- [ ] Der maximale Nachlass (konfigurierbar pro Ebene) wird geprüft; Überschreitung erfordert erweiterte Kompetenz
- [ ] Der Mindestbeitrag wird nicht unterschritten
- [ ] Eine Begründung (≥ 10 Zeichen) ist Pflicht für jeden Nachlass/Zuschlag
- [ ] Befristete Nachlässe werden nach Ablauf nicht mehr in die Berechnung einbezogen
- [ ] Bei Überführung Angebot → Antrag und Antrag → Schwebe werden Nachlässe und Zuschläge vollständig übernommen
- [ ] Im Vertrag werden Nachlässe und Zuschläge in der Vertragsübersicht angezeigt
- [ ] Alle Änderungen an Nachlässen und Zuschlägen werden revisionssicher historisiert (Hibernate Envers)
- [ ] Die Funktion ist spartenübergreifend verfügbar und für alle konfigurierten Sparten nutzbar
- [ ] Der Kompetenzcheck (`NACHLASS_ZUSCHLAG_BEARBEITEN`, `NACHLASS_ZUSCHLAG_ERWEITERT`) wird korrekt durchgeführt
- [ ] Über „Zielbeitrag vorgeben" kann ein gewünschter Jahresbeitrag brutto eingegeben werden; das System ermittelt automatisch den erforderlichen prozentualen Nachlass oder Zuschlag auf Vertragsebene
- [ ] Bei erneuter Zielbeitragsvorgabe wird der bestehende Zielbeitrags-Nachlass/-Zuschlag ersetzt (nicht kumuliert)
- [ ] Der Zielbeitrag unterliegt denselben Plausibilitätsprüfungen (Mindestbeitrag, Maximalwert) wie manuelle Nachlässe
- [ ] Bei Datenänderungen nach Zielbeitragsvorgabe wird ein Warnhinweis angezeigt, dass der Zielbeitrag nicht mehr exakt erreicht wird
- [ ] Die Zielbeitragsfunktion funktioniert spartenübergreifend für alle konfigurierten Sparten

## Wireframe / Skizze

```
┌───────────────────────────────────────────────────────────────────┐
│  Angebot #AG-2026-001234          Status: ⬤ Berechnet            │
├───────────────────────────────────────────────────────────────────┤
│  … (Kundendaten, Produkte, Tarifmerkmale) …                     │
├───────────────────────────────────────────────────────────────────┤
│  Nachlässe & Zuschläge                                           │
│  ┌───────────────────────────────────────────────────────────┐   │
│  │  Vertragsebene                                            │   │
│  │  ┌─────────────────────────────────────────────────────┐  │   │
│  │  │ ▼ Bündelrabatt (Nachlass)       −5,00 %  │ ✏️  🗑️ │  │   │
│  │  │   Gültig: 01.01.2027 – unbefristet                 │  │   │
│  │  │   Begründung: Kunde hat KFZ + Hausrat gebündelt     │  │   │
│  │  │   Effekt: −14,04 EUR                               │  │   │
│  │  └─────────────────────────────────────────────────────┘  │   │
│  │  [+ Nachlass hinzufügen]  [+ Zuschlag hinzufügen]        │   │
│  ├───────────────────────────────────────────────────────────┤   │
│  │  Risikoebene                                              │   │
│  │  ┌─────────────────────────────────────────────────────┐  │   │
│  │  │ KFZ-Haftpflicht                                     │  │   │
│  │  │  ▼ Treuerabatt (Nachlass)      −10,00 %  │ ✏️  🗑️ │  │   │
│  │  │    Gültig: 01.01.2027 – 31.12.2027 (befristet)     │  │   │
│  │  │    Begründung: Langjähriger Bestandskunde seit 2015 │  │   │
│  │  │    Effekt: −15,25 EUR                               │  │   │
│  │  └─────────────────────────────────────────────────────┘  │   │
│  │  ┌─────────────────────────────────────────────────────┐  │   │
│  │  │ KFZ-Teilkasko                                       │  │   │
│  │  │  (keine Nachlässe/Zuschläge)                        │  │   │
│  │  └─────────────────────────────────────────────────────┘  │   │
│  │  [+ Nachlass hinzufügen]  [+ Zuschlag hinzufügen]        │   │
│  │  [🎯 Zielbeitrag vorgeben]                               │   │
│  └───────────────────────────────────────────────────────────┘   │
├───────────────────────────────────────────────────────────────────┤
│  Berechneter Jahresbeitrag:                     € 317,43         │
│  ├─ KFZ-Haftpflicht:         € 152,49 → −15,25 = € 137,24      │
│  ├─ KFZ-Teilkasko:           € 143,55            = € 143,55     │
│  ├─ Summe Netto (nach Risiko):                    € 280,79      │
│  ├─ Vertragsnachlass (Bündelrabatt −5 %):          −€ 14,04     │
│  ├─ Gesamtnetto:                                  € 266,75      │
│  ├─ Versicherungssteuer 19 %:                      +€ 50,68     │
│  └─ Zahlbeitrag (jährlich):                       € 317,43      │
├───────────────────────────────────────────────────────────────────┤
│  [Speichern]  [Berechnen]  [Prüfen]  [Beantragen]               │
└───────────────────────────────────────────────────────────────────┘
```

## Offene Fragen
- Sollen spartenspezifische Nachlassarten über die Spartenkonfiguration (UC-03) pflegbar sein, oder genügt eine globale Konfiguration?
- Gibt es Nachlässe, die automatisch vergeben werden (z. B. Bündelrabatt bei Mehrfachverträgen eines Kunden)?
- Sollen Zuschläge auch automatisch systemseitig vergeben werden können (z. B. bei negativem Schadenverlauf)?
- Welcher maximale Nachlass soll für die erweiterte Kompetenz (`NACHLASS_ZUSCHLAG_ERWEITERT`) gelten?
- Soll ein Genehmigungsworkflow (4-Augen-Prinzip) für Nachlässe oberhalb bestimmter Schwellwerte eingeführt werden?
- Soll die Zielbeitragsfunktion auch auf Risikoebene (einzelnes Produkt) verfügbar sein, oder ausschließlich auf Vertragsebene?
- Soll der Zielbeitrag als Netto- oder Bruttobetrag eingegeben werden? (Aktuell: brutto – konsistent mit der Kundenwahrnehmung)
- Soll bei Datenänderungen nach Zielbeitragsvorgabe der Zielbeitrags-Nachlass automatisch angepasst werden, oder nur ein Warnhinweis angezeigt werden? (Aktuell: Warnhinweis)
