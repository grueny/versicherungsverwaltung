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
