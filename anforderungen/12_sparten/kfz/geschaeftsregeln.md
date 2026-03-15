# Geschäftsregeln – Sparte: KFZ (basierend auf LVM Versicherung)

> Spartenspezifische Geschäftsregeln, die nur in der KFZ-Sparte gelten.  
> Quelle: Öffentlich zugängliche Produktinformationen der LVM Versicherung (lvm.de, Stand März 2026).  
> Spartenübergreifende Regeln sind in `anforderungen/08_geschaeftsregeln.md` erfasst.  
> Jede Regel hat eine eindeutige ID nach dem Schema `GR-KFZ-XXX`.

## Vertragsbezogene Regeln

| GR-Nr. | Regel | Auswirkung | Validierung |
|--------|-------|-----------|-------------|
| GR-KFZ-V01 | Hauptfälligkeit ist der 01.01., Stichtagskündigung bis 30.11. | Vertragsablauf und Kündigungsfrist | Fälligkeitsdatum prüfen |
| GR-KFZ-V02 | Ein Vertrag ist immer an genau ein Fahrzeug gebunden | Fahrzeugdaten sind Pflicht | FIN/HSN/TSN vorhanden |
| GR-KFZ-V03 | KFZ-Haftpflicht ist Pflichtversicherung – darf nicht ohne Nachfolgeversicherung oder Abmeldung enden | Kündigung nur mit Nachweis | Abmelde- oder Folgeversicherungsnachweis |
| GR-KFZ-V04 | Vertragsbeginn bei Neuzulassung = Zulassungsdatum | Automatische Übernahme | Zulassungsdatum als Beginn setzen |

## Produktbezogene Regeln

| GR-Nr. | Regel | Auswirkung | Validierung |
|--------|-------|-----------|-------------|
| GR-KFZ-P01 | Vollkasko enthält automatisch Teilkaskoschutz | Keine separate TK nötig bei VK | Bei VK-Abschluss TK-Deckung automatisch einschließen |
| GR-KFZ-P02 | Kaskoversicherung (TK/VK) setzt KFZ-Haftpflicht voraus | Produkt-Abhängigkeit | HP muss im Vertrag enthalten sein |
| GR-KFZ-P03 | Schutzbrief, Insassenunfall und Mallorca-Police sind Zusatzprodukte zur HP | Nur als Ergänzung abschließbar | HP muss im Vertrag enthalten sein |
| GR-KFZ-P04 | LVM-RabattSchutz ist optional zur HP oder VK buchbar ab SF-Klasse 4 | Verhindert SF-Rückstufung bei 1 Schaden/Jahr, einmal pro Versicherungsjahr einsetzbar | SF-Klasse ≥ 4 prüfen, Schadenanzahl im VJ prüfen |

## LVM-spezifische Regeln

| GR-Nr. | Regel | Auswirkung | Validierung |
|--------|-------|-----------|-------------|
| GR-KFZ-L01 | Update-Garantie: Bedingungsverbesserungen werden automatisch in bestehende Verträge übernommen | Keine manuelle Anpassung durch VN nötig | Automatische Übernahme bei Bedingungsaktualisierung |
| GR-KFZ-L02 | Gelegentliche Fahrer ab 23 Jahre sind auch außerhalb des festgelegten Fahrerkreises berechtigt | Kein Deckungsverlust bei gelegentlicher Nutzung durch Dritte ab 23 | Alter des gelegentlichen Fahrers ≥ 23 |
| GR-KFZ-L03 | Mallorca-Police ist in der HP inklusive enthalten | Kein separater Baustein nötig für Mietfahrzeuge im Ausland | Automatisch aktiv bei HP-Abschluss |
| GR-KFZ-L04 | Eigenschadendeckung ist in der HP inklusive enthalten | Schäden an eigenen Gebäuden/Gegenständen durch eigenes KFZ sind mitversichert | Automatisch aktiv bei HP-Abschluss |
| GR-KFZ-L05 | SchadenService 24/7 – Schadensmeldung rund um die Uhr möglich | Keine Einschränkung bei Meldezeiten | Verfügbarkeit sicherstellen |
| GR-KFZ-L06 | Freie Werkstattwahl bei Kaskoschäden | VN kann Werkstatt frei wählen, alternativ LVM-SchadenService (Partnerwerkstatt mit Zusatzleistungen) | Werkstattwahl dokumentieren |
| GR-KFZ-L07 | Kurzfristige KFZ-Versicherungen (Zusatzfahrer, Mietwagen, Probefahrt, Carsharing) laufen automatisch ab ohne Kündigung | Kein Kündigungsprozess erforderlich | Ablaufdatum prüfen, automatische Beendigung |

## Prämienbezogene Regeln

| GR-Nr. | Regel | Auswirkung | Validierung |
|--------|-------|-----------|-------------|
| GR-KFZ-PR01 | Prämienberechnung basiert auf: SF-Klasse × Typklasse × Regionalklasse × Tarifmerkmale | Automatische Berechnung | Alle Tarifmerkmale müssen erfasst sein |
| GR-KFZ-PR02 | SF-Klasse steigt jährlich um 1 Stufe bei Schadenfreiheit | Automatische Hochstufung zum Fälligkeitstermin | Schadenhistorie prüfen |
| GR-KFZ-PR03 | Rückstufung nach Schadenfall gemäß SF-Rückstufungstabelle | Beitragserhöhung | SF-Tabelle anwenden |
| GR-KFZ-PR04 | Bei Rabattschutz: Erster Schaden pro Versicherungsjahr führt nicht zur Rückstufung | Keine Beitragserhöhung | Rabattschutz-Flag und Schadenanzahl prüfen |
| GR-KFZ-PR05 | Beitragserhöhung durch geänderte Typklassen (GDV-Aktualisierung) löst Sonderkündigungsrecht aus | VN darf sonderkündigen | Beitragsvergleich Alt/Neu |

## Deckungsbezogene Regeln

| GR-Nr. | Regel | Auswirkung | Validierung |
|--------|-------|-----------|-------------|
| GR-KFZ-D01 | Deckungssummen HP: bis 100 Mio. EUR pauschal, bei Personenschäden bis 15 Mio. EUR pro geschädigte Person | Liegt deutlich über gesetzlichem Mindeststandard (7,5 Mio./1,22 Mio./50.000 EUR) | Deckungssummen gemäß LVM-Tarif setzen |
| GR-KFZ-D02 | Bei Ruheversicherung: Nur Haftpflichtschutz (kein Kasko) | Kaskoschutz ruht | Deckungsbausteine einschränken |
| GR-KFZ-D03 | GAP-Deckung bei Leasingfahrzeugen optional anbietbar | Differenz Wiederbeschaffungswert / Ablösewert | Leasingflag prüfen |

## SF-Klassen-Tabelle (Referenz)

| Aktuelle SF-Klasse | Rückstufung nach 1 Schaden | Rückstufung nach 2 Schäden |
|--------------------|---------------------------|---------------------------|
| SF35 – SF20 | SF10 | SF4 |
| SF19 – SF10 | SF5 | SF1 |
| SF9 – SF5 | SF2 | SF0 |
| SF4 – SF1 | SF0 | M |
| SF½ | 0 | M |
| 0 | M | M |

> Hinweis: Die tatsächliche Rückstufungstabelle ist tarifabhängig und kann je nach Produkt abweichen.
