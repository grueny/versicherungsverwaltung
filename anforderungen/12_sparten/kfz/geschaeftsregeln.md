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

## Fahrzeugwechsel-Regeln (Neuvertrag-Modell)

> Regeln für den Fahrzeugwechsel als eigenständigen Geschäftsvorfall mit Neuvertrag-Anlage (→ UC-KFZ-04).  
> Der Fahrzeugwechsel wird **nicht** als Nachtrag auf dem bestehenden Vertrag abgebildet.

| GR-Nr. | Regel | Auswirkung | Validierung |
|--------|-------|-----------|-------------|
| GR-KFZ-FW01 | Ein Fahrzeugwechsel erzeugt immer einen **neuen Vertrag** – der Alt-Vertrag wird beendet | Saubere Vertragstrennung, keine komplexe Abhängigkeit | Alt-Vertrag Status → `GEKUENDIGT`, Neuvertrag → `AKTIV` |
| GR-KFZ-FW02 | Das Wechseldatum entspricht dem Zulassungsdatum des neuen Fahrzeugs (Standard) oder einem vom VN gewählten Datum | Vertragsbeginn Neu = Wechseldatum, Vertragsende Alt = Wechseldatum − 1 Tag | Wechseldatum ≥ heute, ≤ 12 Monate in Zukunft |
| GR-KFZ-FW03 | Die **SF-Klasse** (HP und VK) wird vollständig vom Alt-Vertrag auf den Neuvertrag übernommen (SfKlassenHistorie-Eintrag mit Grund `FAHRZEUGWECHSEL`) | Keine SF-Rückstufung durch den Wechsel | SfKlassenHistorie-Eintrag mit Referenz auf Alt-Vertrag |
| GR-KFZ-FW04 | Beitragserstattung für Alt-Vertrag erfolgt **pro rata temporis** ab dem Wechseldatum | Tagesgenaue Abrechnung, keine doppelte Prämie | Erstattungsbetrag = Restlaufzeit × Tagesbeitrag |
| GR-KFZ-FW05 | Der RabattSchutz wird **nicht automatisch** auf den Neuvertrag übernommen – erneute Buchung erforderlich | Vertragsgebundener Baustein, Neuprüfung SF-Bedingung ≥ SF4 | RabattSchutz-Voraussetzungen am Neuvertrag prüfen |
| GR-KFZ-FW06 | Aktive eVB-Nummern des Alt-Vertrags werden storniert, **außer** im Status `VERWENDET` | Keine ungültige eVB-Stornierung | eVB-Status vor Stornierung prüfen |
| GR-KFZ-FW07 | Bei Fahrzeugartenwechsel werden nicht anwendbare Zusatzbausteine automatisch entfernt | Benutzer wird informiert, welche Bausteine entfallen | Baustein-Kompatibilität mit neuer Fahrzeugart prüfen |
| GR-KFZ-FW08 | Alt- und Neuvertrag speichern weiche Referenzen aufeinander (`nachfolge_vertrag_id` / `ursprung_vertrag_id` im JSONB) | Nachvollziehbarkeit ohne FK-Constraint | JSONB-Felder in `spartenspezifische_daten` |
| GR-KFZ-FW09 | Bei Widerruf des Neuvertrags (14-Tage-Frist) wird der Alt-Vertrag automatisch reaktiviert, sofern nicht `ABGELAUFEN` | Kein versicherungsloser Zustand nach Widerruf | Alt-Vertrag-Status prüfen, ggf. Reaktivierung blockieren |
| GR-KFZ-FW10 | Die VK-SF-Klasse wird auch ohne aktive VK im Neuvertrag in der SfKlassenHistorie gespeichert (Status `RUHEND`) | SF-Klasse geht nicht verloren bei späterem VK-Abschluss | SfKlassenHistorie-Eintrag auch für ruhende VK-SF |

## Deckungsbezogene Regeln

| GR-Nr. | Regel | Auswirkung | Validierung |
|--------|-------|-----------|-------------|
| GR-KFZ-D01 | Deckungssummen HP: bis 100 Mio. EUR pauschal, bei Personenschäden bis 15 Mio. EUR pro geschädigte Person | Liegt deutlich über gesetzlichem Mindeststandard (7,5 Mio./1,22 Mio./50.000 EUR) | Deckungssummen gemäß LVM-Tarif setzen |
| GR-KFZ-D02 | Bei Ruheversicherung: Nur Haftpflichtschutz (kein Kasko) | Kaskoschutz ruht | Deckungsbausteine einschränken |
| GR-KFZ-D03 | GAP-Deckung bei Leasingfahrzeugen optional anbietbar | Differenz Wiederbeschaffungswert / Ablösewert | Leasingflag prüfen |

## Prämienberechnung – Scoring-Faktor-Modell

> Basismodell für die KFZ-Prämienberechnung. Alle **Basisbeiträge** und **Scoringfaktoren** sind als Konfigurationsdaten in der Datenbank gespeichert und über **UC-03 (Produktkonfiguration und Kalkulation)** bearbeitbar.  
> Die hier angegebenen Werte dienen als **initiale Referenzdaten** und können über UC-03 jederzeit angepasst werden.

### Berechnungsformel

```
Jahresbeitrag(Produkt) = Basisbeitrag(Produkt, Fahrzeugart)
    × SF-Faktor(SF-Klasse)                    [nur HP, VK]
    × Typklassen-Faktor(Typklasse)
    × Regionalklassen-Faktor(Regionalklasse)
    × Fahrerkreis-Faktor(Fahrerkreis)
    × Fahrleistungs-Faktor(km-Stufe)
    × Alter-Faktor(Alter jüngster Fahrer)
    × Stellplatz-Faktor(Stellplatz)
    × Nutzungs-Faktor(Nutzungsart)
    × SB-Faktor(Selbstbeteiligung)             [nur TK, VK]

Gesamtjahresbeitrag_netto = Σ Jahresbeitrag(Produkt_i)    [für alle gewählten Produkte]

Versicherungssteuer = Gesamtjahresbeitrag_netto × 19 %

Gesamtjahresbeitrag_brutto = Gesamtjahresbeitrag_netto + Versicherungssteuer

Zahlbeitrag = Gesamtjahresbeitrag_brutto × Zahlungsweise-Faktor
```

| Berechnungsschritt | Beschreibung | Rundung |
|--------------------|-------------|---------|
| Einzelfaktor-Multiplikation | Basisbeitrag × Faktor₁ × Faktor₂ × … | Keine Zwischenrundung |
| Jahresbeitrag pro Produkt | Endwert der Multiplikation | 2 Nachkommastellen (kaufm.) |
| Gesamtjahresbeitrag netto | Summe aller Produktbeiträge | 2 Nachkommastellen |
| Versicherungssteuer | 19 % auf netto | 2 Nachkommastellen |
| Zahlbeitrag | Brutto × Zahlungsweise-Faktor | 2 Nachkommastellen |

> **Nachvollziehbarkeit:** Alle Zwischenergebnisse werden im JSONB-Feld `berechnungsdetails` (→ 07_datenmodell.md, Abschnitt 7.4) in `AngebotProdukt` und `AntragProdukt` persistiert.

### Basisbeiträge (GR-KFZ-PR06)

> Jährlicher Grundbeitrag in EUR je Produkt und Fahrzeugart. Dient als Ausgangswert, auf den alle Scoringfaktoren multiplikativ angewendet werden. Bearbeitbar über UC-03 → Phase 3f.

| Produkt | PKW | Motorrad | Wohnmobil | LKW bis 3,5t | Anhänger |
|---------|----:|--------:|---------:|-------------:|--------:|
| KFZ-HP  | 280 |     120 |      350 |          680 |       85 |
| KFZ-TK  | 100 |      70 |      130 |          200 |       40 |
| KFZ-VK  | 180 |     110 |      220 |          400 |       – |

> **Hinweis:** Vollkasko (VK) ist für Anhänger nicht verfügbar. Basisbeiträge werden je Konfigurationsversion versioniert (→ `kfz_basisbeitrag.gueltig_ab`).

### Scoringfaktor-Tabellen

> Alle Faktoren sind Multiplikatoren zum Basisbeitrag. Faktorwert `1.00` = neutral (kein Einfluss). Werte < 1.00 = Rabatt, Werte > 1.00 = Zuschlag.  
> Bearbeitbar über UC-03 → Phase 3f. Persistiert in Entity `KfzScoringfaktor` (→ 07_datenmodell.md).

#### SF-Faktor (GR-KFZ-PR07) – gilt für HP und VK

> Separate Faktortabellen je Produkt (HP / VK). Die SF-Klasse ist das einflussreichste Tarifmerkmal.

**HP-Faktoren:**

| SF-Klasse | Beitragssatz (%) | Faktor |
|-----------|----------------:|-------:|
| SF35 | 25 | 0,2500 |
| SF30 | 27 | 0,2700 |
| SF25 | 29 | 0,2900 |
| SF20 | 32 | 0,3200 |
| SF15 | 37 | 0,3700 |
| SF10 | 42 | 0,4200 |
| SF8 | 47 | 0,4700 |
| SF6 | 52 | 0,5200 |
| SF5 | 55 | 0,5500 |
| SF4 | 60 | 0,6000 |
| SF3 | 65 | 0,6500 |
| SF2 | 72 | 0,7200 |
| SF1 | 80 | 0,8000 |
| SF½ | 100 | 1,0000 |
| 0 | 130 | 1,3000 |
| M | 240 | 2,4000 |

**VK-Faktoren:**

| SF-Klasse | Beitragssatz (%) | Faktor |
|-----------|----------------:|-------:|
| SF35 | 25 | 0,2500 |
| SF30 | 27 | 0,2700 |
| SF25 | 30 | 0,3000 |
| SF20 | 33 | 0,3300 |
| SF15 | 38 | 0,3800 |
| SF10 | 44 | 0,4400 |
| SF8 | 49 | 0,4900 |
| SF6 | 54 | 0,5400 |
| SF5 | 58 | 0,5800 |
| SF4 | 63 | 0,6300 |
| SF3 | 68 | 0,6800 |
| SF2 | 75 | 0,7500 |
| SF1 | 85 | 0,8500 |
| SF½ | 100 | 1,0000 |
| 0 | 125 | 1,2500 |
| M | 200 | 2,0000 |

> **TK:** Die Teilkaskoversicherung kennt keine SF-Klasse. Der SF-Faktor entfällt (implizit 1,0000).

#### Typklassen-Faktor (GR-KFZ-PR08)

> Die Typklasse wird vom GDV zentral vergeben (→ IMP-01). Separate Typklassenbereiche je Produkt: HP (10–25), TK (10–33), VK (10–34). Faktor-Referenzwert = Typklasse 15.

| Typklasse | HP-Faktor | TK-Faktor | VK-Faktor |
|-----------|----------:|----------:|----------:|
| 10 | 0,7000 | 0,6000 | 0,6500 |
| 11 | 0,7600 | 0,6800 | 0,7100 |
| 12 | 0,8200 | 0,7600 | 0,7700 |
| 13 | 0,8800 | 0,8400 | 0,8300 |
| 14 | 0,9400 | 0,9200 | 0,9100 |
| 15 | 1,0000 | 1,0000 | 1,0000 |
| 16 | 1,0800 | 1,0800 | 1,0900 |
| 17 | 1,1600 | 1,1600 | 1,1800 |
| 18 | 1,2500 | 1,2500 | 1,2800 |
| 19 | 1,3500 | 1,3500 | 1,3900 |
| 20 | 1,4500 | 1,4500 | 1,5000 |
| 21 | 1,5700 | 1,5700 | 1,6200 |
| 22 | 1,7000 | 1,7000 | 1,7500 |
| 23 | 1,8500 | 1,8500 | 1,9000 |
| 24 | 2,0000 | 2,0000 | 2,0500 |
| 25 | 2,1500 | 2,1500 | 2,2000 |
| 26–33 | – | 2,3000–3,5000 | 2,3500–3,6000 |
| 34 | – | – | 3,8000 |

> Typklassen > 25 kommen nur bei TK und VK vor. In der Datenbank wird jede Typklasse einzeln als Zeile in `KfzScoringfaktor` gespeichert.

#### Regionalklassen-Faktor (GR-KFZ-PR09)

> Die Regionalklasse wird vom GDV zentral vergeben (→ IMP-02). Separate Regionalklassenbereiche: HP (1–12), TK (1–16), VK (1–9). Faktor-Referenzwert = mittlere Klasse.

| Regionalklasse | HP-Faktor | TK-Faktor | VK-Faktor |
|----------------|----------:|----------:|----------:|
| 1 | 0,8000 | 0,7500 | 0,8000 |
| 2 | 0,8500 | 0,8000 | 0,8500 |
| 3 | 0,9000 | 0,8500 | 0,9000 |
| 4 | 0,9500 | 0,9000 | 0,9500 |
| 5 | 0,9800 | 0,9500 | 0,9800 |
| 6 | 1,0000 | 1,0000 | 1,0000 |
| 7 | 1,0500 | 1,0500 | 1,0500 |
| 8 | 1,1000 | 1,1000 | 1,1000 |
| 9 | 1,1500 | 1,1500 | 1,1500 |
| 10 | 1,2000 | 1,2000 | – |
| 11 | 1,2800 | 1,2500 | – |
| 12 | 1,3500 | 1,3000 | – |
| 13–16 | – | 1,3500–1,5000 | – |

#### Fahrerkreis-Faktor (GR-KFZ-PR10) – produktübergreifend

| Fahrerkreis | Faktor | Beschreibung |
|-------------|-------:|-------------|
| NUR_VN | 0,8500 | Nur Versicherungsnehmer fährt |
| VN_PARTNER | 0,9500 | VN und Partner |
| ALLE | 1,1000 | Unbeschränkter Fahrerkreis |

#### Fahrleistungs-Faktor (GR-KFZ-PR11) – produktübergreifend

| Jährliche Fahrleistung | Faktor | Stufe |
|------------------------|-------:|-------|
| bis 6.000 km | 0,7500 | Wenigfahrer |
| 6.001 – 9.000 km | 0,8500 | Gering |
| 9.001 – 12.000 km | 0,9500 | Unterdurchschnittlich |
| 12.001 – 15.000 km | 1,0000 | Durchschnitt |
| 15.001 – 20.000 km | 1,0800 | Überdurchschnittlich |
| 20.001 – 25.000 km | 1,1500 | Vielfahrer |
| 25.001 – 30.000 km | 1,2500 | Vielfahrer+ |
| über 30.000 km | 1,3500 | Berufspendler/Gewerbe |

#### Alter-Faktor (GR-KFZ-PR12) – produktübergreifend

> Alter des jüngsten berechtigten Fahrers. Junge Fahrer sind statistisch risikoreich.

| Alter jüngster Fahrer | Faktor |
|------------------------|-------:|
| 17 – 20 | 1,8500 |
| 21 – 22 | 1,5500 |
| 23 – 24 | 1,3000 |
| 25 – 29 | 1,1000 |
| 30 – 49 | 1,0000 |
| 50 – 64 | 1,0000 |
| 65 – 69 | 1,0500 |
| ab 70 | 1,1500 |

#### Stellplatz-Faktor (GR-KFZ-PR13) – produktübergreifend

| Stellplatz | Faktor |
|------------|-------:|
| GARAGE | 0,9000 |
| CARPORT | 0,9500 |
| STRASSE | 1,0500 |

#### Nutzungsart-Faktor (GR-KFZ-PR14) – produktübergreifend

| Nutzungsart | Faktor |
|-------------|-------:|
| PRIVAT | 1,0000 |
| GEMISCHT | 1,1000 |
| GESCHAEFTLICH | 1,2000 |

#### Selbstbeteiligungs-Faktor (GR-KFZ-PR15) – nur TK / VK

> Die Selbstbeteiligung beeinflusst den Kasko-Beitrag. Höhere SB = niedrigerer Beitrag. Separate Faktortabellen für TK und VK.

**TK-Faktoren:**

| Selbstbeteiligung | Faktor |
|-------------------|-------:|
| 0 EUR | 1,1500 |
| 150 EUR | 1,0000 |
| 300 EUR | 0,8500 |
| 500 EUR | 0,7500 |

**VK-Faktoren:**

| Selbstbeteiligung | Faktor |
|-------------------|-------:|
| 0 EUR | 1,2000 |
| 150 EUR | 1,0500 |
| 300 EUR | 1,0000 |
| 500 EUR | 0,8500 |
| 1.000 EUR | 0,7000 |

#### Zahlungsweise-Faktor (GR-KFZ-PR16) – spartenübergreifend

> Bereits in `08_geschaeftsregeln.md` als Enum `Zahlungsweise` definiert (→ Abschnitt 5.15). Aufschläge werden nicht als Scoringfaktor in `KfzScoringfaktor` gespeichert, sondern als separater Berechnungsschritt.

| Zahlungsweise | Faktor |
|---------------|-------:|
| JAEHRLICH | 1,0000 |
| HALBJAEHRLICH | 1,0300 |
| VIERTELJAEHRLICH | 1,0500 |
| MONATLICH | 1,0500 |

### Berechnungsbeispiel

> PKW, HP + TK, SF5, Typklasse HP 15 / TK 20, Regionalklasse HP 6 / TK 6, Fahrerkreis ALLE, 15.000 km, Alter 35, Garage, Privat, SB TK 150, jährliche Zahlung.

**HP-Beitrag:**
```
280,00 (Basisbeitrag PKW-HP)
× 0,5500  (SF5 HP)
× 1,0000  (Typklasse 15)
× 1,0000  (Regionalklasse 6)
× 1,1000  (Fahrerkreis ALLE)
× 1,0000  (15.000 km)
× 1,0000  (Alter 35)
× 0,9000  (Garage)
× 1,0000  (Privat)
────────────────────────
= 152,46 EUR (netto HP-Jahresbeitrag)
```

**TK-Beitrag:**
```
100,00 (Basisbeitrag PKW-TK)
× 1,0000  (kein SF-Faktor bei TK)
× 1,4500  (Typklasse 20 TK)
× 1,0000  (Regionalklasse 6)
× 1,1000  (Fahrerkreis ALLE)
× 1,0000  (15.000 km)
× 1,0000  (Alter 35)
× 0,9000  (Garage)
× 1,0000  (Privat)
× 1,0000  (SB 150)
────────────────────────
= 143,55 EUR (netto TK-Jahresbeitrag)
```

**Gesamt:**
```
Netto:    152,46 + 143,55 = 296,01 EUR
Steuer:   296,01 × 19 %   =  56,24 EUR
Brutto:   296,01 + 56,24  = 352,25 EUR
Zahlung:  352,25 × 1,0000 = 352,25 EUR (jährlich)
```

### Regelzusammenfassung – Prämien-Scoring

| GR-Nr. | Regel | Konfigurierbar via UC-03 | Datenbank-Entity |
|--------|-------|:-----------------------:|-----------------|
| GR-KFZ-PR06 | Basisbeiträge je Produkt × Fahrzeugart definieren den Ausgangswert der Prämienberechnung | ✅ | `KfzBasisbeitrag` |
| GR-KFZ-PR07 | SF-Faktoren (HP / VK separat) bestimmen den größten Einzeleinfluss auf den Beitrag | ✅ | `KfzScoringfaktor` (Typ: `SF_KLASSE`) |
| GR-KFZ-PR08 | Typklassen-Faktoren werden pro Produkt (HP / TK / VK) je Typklasse definiert | ✅ | `KfzScoringfaktor` (Typ: `TYPKLASSE`) |
| GR-KFZ-PR09 | Regionalklassen-Faktoren werden pro Produkt (HP / TK / VK) je Regionalklasse definiert | ✅ | `KfzScoringfaktor` (Typ: `REGIONALKLASSE`) |
| GR-KFZ-PR10 | Fahrerkreis-Faktor gilt produktübergreifend | ✅ | `KfzScoringfaktor` (Typ: `FAHRERKREIS`) |
| GR-KFZ-PR11 | Fahrleistungs-Faktor in gestaffelten km-Stufen | ✅ | `KfzScoringfaktor` (Typ: `FAHRLEISTUNG`) |
| GR-KFZ-PR12 | Alter-Faktor nach Altersgruppen des jüngsten Fahrers | ✅ | `KfzScoringfaktor` (Typ: `ALTER_JUENGSTER_FAHRER`) |
| GR-KFZ-PR13 | Stellplatz-Faktor (Garage / Carport / Straße) | ✅ | `KfzScoringfaktor` (Typ: `STELLPLATZ`) |
| GR-KFZ-PR14 | Nutzungsart-Faktor (Privat / Gemischt / Geschäftlich) | ✅ | `KfzScoringfaktor` (Typ: `NUTZUNGSART`) |
| GR-KFZ-PR15 | Selbstbeteiligungs-Faktor (TK / VK separat) | ✅ | `KfzScoringfaktor` (Typ: `SELBSTBETEILIGUNG`) |
| GR-KFZ-PR16 | Zahlungsweise-Faktor wird als letzter Schritt auf den Bruttobeitrag angewendet | ✅ | Enum `Zahlungsweise` (spartenübergreifend) |

### Design-Entscheidungen

| Entscheidung | Begründung |
|-------------|-----------|
| Multiplikatives Scoring-Modell (Faktorenkette) | Einfach erweiterbar; neue Faktoren werden als zusätzliche Multiplikatoren eingefügt, ohne bestehende Logik zu ändern |
| Keine Zwischenrundung | Vermeidung von Rundungsabweichungen; Rundung erfolgt erst beim Endergebnis pro Produkt |
| Separate SF-Faktoren für HP / VK | HP und VK haben unterschiedliche Schadenverläufe; TK hat keinen SF-Faktor |
| Produktübergreifende Faktoren für Fahrerkreis, Fahrleistung, Alter, Stellplatz, Nutzung | Vereinfacht die Konfiguration; bei Bedarf kann UC-03 diese später produktdifferenziert gestalten |
| Basisbeiträge nach Fahrzeugart differenziert | Fahrzeugart hat fundamentalen Einfluss auf das Grundrisiko; separate Basisbeiträge statt Fahrzeugart-Faktor |
| Alle Werte in DB statt Hardcoded | Anpassungen durch Aktuare über UC-03 ohne Deployment möglich |

---

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
