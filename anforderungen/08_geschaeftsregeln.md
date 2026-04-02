# Geschäftsregeln – Spartenübergreifend

> Fachliche Regeln, die **spartenübergreifend** im System durchgesetzt werden müssen.  
> **Spartenspezifische** Geschäftsregeln werden unter `12_sparten/<spartenname>/geschaeftsregeln.md` erfasst.  
> Jede Regel hat eine eindeutige ID zur Referenzierung in Use Cases.

## Vertragsbezogene Regeln

| GR-Nr. | Regel | Auswirkung | Validierung | Quelle |
|--------|-------|-----------|-------------|--------|
| GR-V01 | Ein Antrag übernimmt alle Daten des Angebots; nachträgliche Änderungen am Angebot wirken sich nicht auf den Antrag aus | Angebot und Antrag sind nach Beantragung entkoppelt | Snapshot bei Antragserzeugung; keine Referenz auf lebende Angebotsdaten | UC-01 (GR-03) |
| GR-V02 | Ein Vertragsstand gehört immer zu genau einem Vorgang (Neugeschäft, Änderung, Stornierung) | Vertragsstand kann nicht ohne Vorgangszuordnung erzeugt werden | 1:1-Zuordnung Vertragsstand → Vorgang prüfen | UC-02 (GR-08) |
| GR-V03 | Der Vorgangstyp wird automatisch aus dem Antragskontext ermittelt (neuer Vertrag = Neugeschäft, bestehender Vertrag = Änderung) | Vorgangstyp wird systemseitig gesetzt | Existiert Vertrag? Ja → Änderung, Nein → Neugeschäft | UC-02 (GR-09) |
| GR-V04 | Bei Ablehnung eines Antrags wird kein Vertragsstand erzeugt; die Schwebe wird geschlossen | Kein Vertrag bei Ablehnung | Antragsstatus = „Abgelehnt" → keine Vertragsstand-Erzeugung; Schwebe → „Geschlossen" | UC-02 (GR-11) |
| GR-V05 | Bei jeder Policierung wird die Vertragsbewegung an das DataWarehouse (S4) über Kafka-Topic `vertrag.datawarehouse.bewegungen` gemeldet | DataWarehouse erhält vollständige Bestandsdaten und Vertragsbewegungen | Kafka-Message muss Vertragsnummer, Sparte, Vorgangstyp, Beitrag und Zeitstempel enthalten | UC-05 (GR-12) |

## Kundenbezogene Regeln

| GR-Nr. | Regel | Auswirkung | Validierung | Quelle |
|--------|-------|-----------|-------------|--------|
| GR-K01 | | | | |
| GR-K02 | | | | |

## Schadenbezogene Regeln

| GR-Nr. | Regel | Auswirkung | Validierung | Quelle |
|--------|-------|-----------|-------------|--------|
| GR-S01 | | | | |
| GR-S02 | | | | |

## Prämienbezogene Regeln

| GR-Nr. | Regel | Auswirkung | Validierung | Quelle |
|--------|-------|-----------|-------------|--------|
| GR-P01 | Die Prämienberechnung einer Sparte erfolgt über ein konfigurierbares Scoring-Faktor-Modell: Basisbeitrag × Faktor₁ × … × Faktorₙ = Nettobeitrag. Basisbeiträge und Faktoren sind über UC-03 versioniert pflegbar | Änderungen an Tarifen erfordern kein Deployment; Simulation und 4-Augen-Freigabe über UC-03 | Alle benötigten Faktoren müssen für die gewählte Produktkombination vorhanden sein; fehlende Faktoren führen zu Berechnungsfehler | UC-03, kfz/geschaeftsregeln.md |
| GR-P02 | Versicherungssteuer (19 %) wird auf den Gesamtjahresbeitrag netto (Summe aller Produkte) aufgeschlagen. Der Zahlungsweise-Aufschlag wird auf den Bruttobeitrag angewendet | Steuerbetrag und Zahlungsbeitrag werden als Einzelposten in den Berechnungsdetails ausgewiesen | Versicherungssteuersatz und Zahlungsweise-Faktoren müssen konfiguriert sein | UC-01, UC-03 |
| GR-P03 | Nachlässe und Zuschläge können auf Vertragsebene (Gesamtbeitrag) und auf Risikoebene (Einzelprodukt) vergeben werden. Beide Ebenen werden bei der Kalkulation berücksichtigt | Beitrag wird um Nachlässe reduziert bzw. um Zuschläge erhöht | Ebene (VERTRAG/RISIKO) und Art (NACHLASS/ZUSCHLAG) müssen angegeben sein | UC-10 |
| GR-P04 | Nachlässe und Zuschläge sind ab Angebot (Status ENTWURF) bis Schwebe (Status OFFEN) bearbeitbar. Im policierten Vertrag werden sie nur angezeigt | Bearbeitung nur in vorvertraglichen Stadien möglich; Änderung im Vertrag nur über Nachtrag (UC-05) | Statusprüfung des übergeordneten Objekts vor Bearbeitung | UC-10 |
| GR-P05 | Der maximale prozentuale Nachlass pro Ebene ist konfigurierbar (Standard: 30 % auf Vertragsebene, 50 % auf Risikoebene). Überschreitung erfordert die Kompetenz `NACHLASS_ZUSCHLAG_ERWEITERT` | Kompetenzprüfung bei Nachlassvergabe über Schwellwert | Nachlasshöhe gegen konfigurierten Maximalwert prüfen; bei Überschreitung Kompetenzabfrage gegen S8 | UC-10 |
| GR-P06 | Die Summe aller Nachlässe darf nicht dazu führen, dass der berechnete Gesamtbeitrag unter einen konfigurierten Mindestbeitrag (Standard: 1,00 EUR netto p.a.) fällt | Mindestbeitrag wird systemseitig durchgesetzt | Gesamtbeitrag nach Abzug aller Nachlässe ≥ Mindestbeitrag | UC-10 |
| GR-P07 | Reihenfolge der Nachlass-/Zuschlagsanwendung: (1) Risikoebene prozentual, (2) Risikoebene absolut, (3) Vertragsebene prozentual, (4) Vertragsebene absolut – jeweils vor Versicherungssteuer und Zahlungsweiseaufschlag | Deterministische Berechnungsreihenfolge | Kalkulation muss die definierte Reihenfolge einhalten | UC-10 |
| GR-P08 | Jeder Nachlass und Zuschlag erfordert eine Begründung (Pflichtfeld, ≥ 10 Zeichen). Alle Änderungen werden revisionssicher historisiert | Nachvollziehbarkeit und Audit-Trail | Begründung NOT NULL und LENGTH ≥ 10; Hibernate Envers aktiv | UC-10 |
| GR-P09 | Bei der Überführung Angebot → Antrag (UC-01) und Antrag → Schwebe (UC-02) werden alle Nachlässe und Zuschläge als Snapshot übernommen | Vollständige Datenübernahme über die Prozesskette | Nachlässe/Zuschläge müssen beim Kopieren mitübernommen werden | UC-10 |
| GR-P10 | Befristete Nachlässe (mit gültig_bis-Datum) werden nach Ablauf bei der nächsten Beitragsberechnung automatisch nicht mehr berücksichtigt | Automatischer Ablauf befristeter Nachlässe | gueltig_bis < Berechnungsstichtag → Nachlass nicht anwenden | UC-10 |
| GR-P11 | Über die Zielbeitragsfunktion kann ein gewünschter Jahresbeitrag brutto vorgegeben werden. Das System ermittelt automatisch den erforderlichen prozentualen Nachlass oder Zuschlag auf Vertragsebene (Nachlassart `ZIELBEITRAG_NACHLASS` bzw. `ZIELBEITRAG_ZUSCHLAG`). Ein bestehender Zielbeitrags-Nachlass/-Zuschlag wird bei erneuter Vorgabe ersetzt, nicht kumuliert | Automatische Rückrechnung des erforderlichen Nachlasses/Zuschlags aus Zielbeitrag | Zielbeitrag > 0 EUR; Zielbeitrag ≥ Mindestbeitrag (brutto); Rückrechnung: Ziel-Netto = Zielbeitrag ÷ (1 + VStSatz) ÷ Zahlweisefaktor; Nachlass = (Ist-Netto − Ziel-Netto) ÷ Ist-Netto × 100 | UC-10 |
| GR-P12 | Die Zielbeitragsermittlung unterliegt denselben Plausibilitätsprüfungen wie manuell erfasste Nachlässe (GR-P05, GR-P06). Der ermittelte Nachlass/Zuschlag muss innerhalb der konfigurierten Maximalwerte liegen; Überschreitung erfordert `NACHLASS_ZUSCHLAG_ERWEITERT` | Konsistente Validierung für manuelle und automatische Nachlässe | Ermittelter Nachlasswert gegen konfiguriertes Maximum prüfen; Gesamtbeitrag nach Abzug ≥ Mindestbeitrag | UC-10 |

## Übergreifende Regeln

| GR-Nr. | Regel | Auswirkung | Validierung | Quelle |
|--------|-------|-----------|-------------|--------|
| GR-A01 | Ein Angebot kann nur beantragt werden, wenn es den Status „Geprüft" hat | Funktion „Beantragen" ist nur bei Status „Geprüft" aktiv | Angebotsstatus = „Geprüft" | UC-01 (GR-01) |
| GR-A02 | Bei der Beantragung entscheidet der Benutzer, ob das Angebot erhalten bleibt oder gelöscht wird | Angebot wird beibehalten (Status „Beantragt") oder gelöscht | Pflichtauswahl „Angebot behalten" (Ja/Nein) bei Beantragung | UC-01 (GR-02) |
| GR-A03 | Ein Angebot ohne Aktivität kann nach einem konfigurierbaren Zeitraum automatisch verfallen | Angebot wechselt automatisch in Status „Verfallen" | Letzte Änderung + konfigurierbarer Zeitraum < heute | UC-01 (GR-04) |
| GR-A04 | Ein Antrag kann nur freigegeben werden, wenn er den Status „Geprüft" hat | Funktion „Freigeben" ist nur bei Status „Geprüft" aktiv | Antragsstatus = „Geprüft" | UC-02 (GR-05) |
| GR-A05 | Bei jeder Freigabe wird eine Schwebe angelegt, unabhängig davon, ob der Antrag ausgesteuert wird | Schwebe dokumentiert den offenen Vorgang | Freigabe → Schwebe muss erzeugt werden | UC-02 (GR-06) |
| GR-A06 | Aussteuerungsregeln sind konfigurierbar (pro Sparte und spartenübergreifend) und bestimmen, ob ein Antrag dem Innendienst vorgelegt wird | Antrag wird ausgesteuert oder dunkelverarbeitet | Regelengine prüft Aussteuerungskriterien | UC-02 (GR-07) |
| GR-A07 | Ein ausgesteuerter Antrag kann vom Innendienst freigegeben, abgelehnt oder zurückgewiesen werden | Drei Entscheidungsoptionen bei ausgesteuerten Anträgen | Innendienst-Kompetenz + Pflichtauswahl Entscheidung | UC-02 (GR-10) |
| GR-A08 | Bei der Policierung werden immer alle vier Schnittstellen aufgerufen: DataWarehouse (S4), Provision (S1), Konzerninkasso (S2) und Druck (S5) | Vollständige Benachrichtigung aller nachgelagerten Systeme | Alle vier Schnittstellenaufrufe müssen protokolliert werden (erfolgreich, fehlgeschlagen oder ausstehend) | UC-05 (GR-12) |
| GR-A09 | Die Policierung erzeugt immer genau einen Vertragsstand, der genau einem Vorgang zugeordnet ist | 1:1-Zuordnung Vertragsstand → Vorgang | Vertragsstand-Erzeugung + Vorgangszuordnung in einer Transaktion | UC-05 (GR-13), UC-02 (GR-08) |
| GR-A10 | Bei Neugeschäft wird ein neuer Vertrag angelegt; bei Änderung/Stornierung wird der bestehende Vertrag fortgeführt | Korrekte Vertragserzeugung bzw. -fortschreibung | Vorgangstyp bestimmt, ob neuer Vertrag oder neuer Vertragsstand an bestehendem Vertrag | UC-05 (GR-14) |
| GR-A11 | Schnittstellenfehler bei der Policierung führen nicht zum Abbruch der Vertragsstand-Erzeugung; fehlgeschlagene Aufrufe werden über Retry-Mechanismen nachgeliefert | Vertragsstand wird auch bei Schnittstellenfehlern erzeugt | Transactional Outbox Pattern; Dead-Letter-Topics; Wiedervorlage bei dauerhaftem Fehler | UC-05 (GR-15) |
| GR-A12 | Ein Sachbearbeiter kann eine offene Schwebe nur policieren, wenn er die Kompetenz „Schwebe policieren" (SCHWEBE_POLICIEREN) besitzt | Berechtigungsprüfung vor manueller Policierung | Kompetenzabfrage gegen S8 (Kompetenz-System) | UC-05 (GR-16) |
