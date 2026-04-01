# UC-KFZ-04: Fahrzeugwechsel durchführen (Neuvertrag)

## Beschreibung
Beim **Fahrzeugwechsel** wird der bestehende KFZ-Vertrag für das alte Fahrzeug beendet und ein **neuer Vertrag** für das neue Fahrzeug angelegt. Im Gegensatz zu einem klassischen Nachtrag (UC-05) werden Alt- und Neuvertrag als **eigenständige Verträge ohne komplexe Abhängigkeit** geführt.

Bestimmte Daten werden vom alten auf den neuen Vertrag **übernommen**:
- **SF-Klasse** (HP und VK) → Interner Transfer, ähnlich SFR-Transfer (UC-KFZ-21), jedoch automatisiert mit Änderungsgrund `FAHRZEUGWECHSEL`
- **Versicherungsnehmer** (Partner-Referenz)
- **Vertragsdaten**: Fahrerkreis, Nutzungsart, Stellplatz, Fahrleistung, Selbstbeteiligung, Zahlungsweise
- **Zusatzbausteine**: Schutzbrief, RabattSchutz, FahrerKasko, AuslandPlus (sofern auf neues Fahrzeug anwendbar)

Für das neue Fahrzeug wird eine **neue eVB-Nummer** erzeugt (→ UC-KFZ-02). Die eVB-Nummer des alten Vertrags wird beim GDV storniert, sofern sie nicht bereits verwendet wurde.

Am Ende des Prozesses existieren zwei unabhängige Verträge:
- **Alt-Vertrag:** Status `GEKUENDIGT` oder `ABGELAUFEN` mit Stornierungsgrund `FAHRZEUGWECHSEL`
- **Neu-Vertrag:** Status `AKTIV` mit neuer Vertragsnummer, neuem Fahrzeug und übernommener SF-Klasse

Optional kann der Versicherungsnehmer für das alte Fahrzeug eine **Ruheversicherung** beantragen (→ UC-KFZ-06), z. B. bei einer Übergangszeit, in der das alte Fahrzeug noch nicht verkauft/abgemeldet ist.

## Akteure
- **Primär:** Innendienst (Sachbearbeitung), Außendienst (Vertrieb)
- **Sekundär:** Versicherungsnehmer (VN) – beantragt den Fahrzeugwechsel
- **Extern:** GDV (eVB-Stornierung Alt, eVB-Erzeugung Neu), Zulassungsstelle (Zulassung Neu-Fahrzeug)

## Vorbedingungen
- Ein **aktiver** KFZ-Vertrag für das alte Fahrzeug existiert (Status `AKTIV`)
- Der Vertrag enthält mindestens das Produkt **KFZ-Haftpflicht (KFZ-HP)** (Pflichtversicherung)
- Der Benutzer besitzt die Kompetenz `VERTRAG_FAHRZEUGWECHSEL`
- Die Fahrzeugdaten des neuen Fahrzeugs sind bekannt (mindestens HSN/TSN oder FIN)

## Auslöser
- Versicherungsnehmer meldet den Fahrzeugwechsel (schriftlich, telefonisch, über Außendienst oder Self-Service-Portal)
- Innendienst-Sachbearbeiter oder Außendienst-Vermittler eröffnet den Fahrzeugwechsel im System

## Ablauf (Hauptszenario)

### Phase 1: Fahrzeugwechsel initiieren
1. Der Benutzer navigiert zur Vertragsdetailansicht des Alt-Vertrags und wählt die Aktion **„Fahrzeugwechsel"**
2. System prüft die Vorbedingungen:
   - Vertrag im Status `AKTIV`
   - Kein laufender Fahrzeugwechsel, VWB-Vorgang oder SFR-Transfer auf diesem Vertrag
   - Keine offene Schwebe, die den Vertrag sperrt
3. System zeigt eine Übersicht des Alt-Vertrags mit Fahrzeugdaten, aktueller SF-Klasse (HP + VK), Deckungsumfang, Beitrag und übernehmbaren Vertragsdaten
4. System erzeugt einen internen **Fahrzeugwechsel-Vorgang** im Status `ENTWURF`

### Phase 2: Neues Fahrzeug erfassen
5. Der Benutzer erfasst die Daten des neuen Fahrzeugs:
   - **HSN/TSN** (Herstellerschlüsselnummer / Typschlüsselnummer) – Pflicht
   - **FIN** (Fahrzeugidentnummer) – optional bei Neuwagen
   - **Erstzulassung** – Pflicht
   - **Amtliches Kennzeichen** – optional (ggf. noch nicht bekannt bei Neuwagen)
6. System ermittelt automatisch die Fahrzeugdaten aus dem GDV-Typklassenverzeichnis:
   - Hersteller, Modell, Leistung (kW/PS), Hubraum, Antriebsart
   - **Typklassen** (HP, TK, VK) → GR-KFZ-PR01
7. System ermittelt die **Regionalklasse** anhand der Halteradresse (PLZ des VN)
8. System prüft, ob sich die **Fahrzeugart** ändert (z. B. PKW → Wohnmobil) → Phase 2a

#### Phase 2a: Fahrzeugartenwechsel (bedingt)
9. Falls sich die Fahrzeugart ändert, informiert das System den Benutzer:
   - Geänderte Basisbeiträge je Fahrzeugart
   - Ggf. nicht mehr verfügbare Zusatzbausteine (z. B. Schutzbrief nur für PKW/Motorrad)
   - Ggf. zusätzlich verfügbare Bausteine
10. Der Benutzer bestätigt den Fahrzeugartenwechsel oder bricht ab

### Phase 3: Vertragsdaten übernehmen und anpassen
11. System übernimmt automatisch folgende Daten vom Alt-Vertrag als **Vorschlag**:
    - Versicherungsnehmer (Partner-ID)
    - **SF-Klasse HP** und **SF-Klasse VK** (sofern VK vorhanden)
    - Deckungsumfang (HP, TK, VK – sofern auf neues Fahrzeug anwendbar)
    - Fahrerkreis (Alter jüngster/ältester Fahrer, Fahreranzahl)
    - Nutzungsart (privat, gewerblich, Pendelfahrten)
    - Jährliche Fahrleistung (km/Jahr)
    - Stellplatz (Garage, Carport, Straße etc.)
    - Selbstbeteiligung TK und VK
    - Zahlungsweise (jährlich, halbjährlich, vierteljährlich, monatlich)
    - Zusatzbausteine (Schutzbrief, RabattSchutz, FahrerKasko, AuslandPlus)
12. Der Benutzer kann alle übernommenen Daten **individuell anpassen**, z. B.:
    - Deckungsumfang ändern (z. B. von VK auf TK umstellen)
    - Fahrleistung korrigieren
    - Selbstbeteiligung anpassen
    - Zusatzbausteine hinzufügen oder entfernen
13. System validiert die Plausibilität der Vertragsdaten (→ plausibilitaeten.md, PL-KFZ-N01 ff.)

### Phase 4: Prämienberechnung und Vorschau
14. System führt eine **vollständige Prämienberechnung** für den neuen Vertrag durch:
    - Basisbeitrag (Fahrzeugart × Deckung) × Scoringfaktoren (→ kfz/geschaeftsregeln.md, Scoring-Faktor-Modell)
    - SF-Faktor (übernommene SF-Klasse), Typklasse-Faktor (neues Fahrzeug), Regionalklasse-Faktor, etc.
15. System zeigt eine **Vergleichsansicht**:
    - Alt-Vertrag: Fahrzeug, Beitrag, SF-Klasse, Deckung
    - Neu-Vertrag: Fahrzeug, Beitrag, SF-Klasse, Deckung
    - Beitragsdifferenz (Alt vs. Neu)
    - Erstattung/Nachzahlung für den Alt-Vertrag (pro rata temporis bis Wechseldatum)
16. System berechnet den **Stichtag** für den Fahrzeugwechsel:
    - Standard: Zulassungsdatum des neuen Fahrzeugs (→ GR-KFZ-FW02)
    - Alternativ: Vom Benutzer gewähltes Wechseldatum (z. B. Liefertermin Neuwagen)

### Phase 5: Fahrzeugwechsel bestätigen und durchführen
17. Der Benutzer bestätigt den Fahrzeugwechsel
18. System führt folgende Aktionen **transaktional** aus:

**Alt-Vertrag beenden:**
19. System erzeugt einen **Vorgang** vom Typ `STORNIERUNG` auf dem Alt-Vertrag mit Beschreibung `Fahrzeugwechsel – Ablösung durch {Neu-Vertragsnummer}`
20. System erzeugt einen neuen **Vertragsstand** (nächste Version) mit:
    - `gueltig_bis` = Wechseldatum (Tag vor Beginn des Neuvertrags)
    - Stornierungsgrund: `FAHRZEUGWECHSEL`
21. System setzt den Alt-Vertrag auf Status `GEKUENDIGT` (→ läuft bis Wechseldatum, dann `ABGELAUFEN`)
22. Aktive **eVB-Nummer** des Alt-Vertrags wird beim GDV storniert (falls nicht bereits bei Zulassungsstelle verwendet → GR-KFZ-FW06)

**Neuvertrag anlegen:**
23. System erzeugt einen neuen **Antrag** mit Antragsart `NEUGESCHAEFT` und den erfassten Vertragsdaten
24. System erzeugt eine **Schwebe** zur Antragsprüfung (bei Dunkelverarbeitung automatisch erledigt → UC-02, GR-A08)
25. System erzeugt einen neuen **Vertrag** mit:
    - Neue Vertragsnummer
    - Vertragsbeginn = Wechseldatum
    - Partner-ID (VN) = Partner-ID des Alt-Vertrags
    - `spartenspezifische_daten.ursprung_vertrag_id` = ID des Alt-Vertrags (weiche Referenz, nur für Nachvollziehbarkeit)
26. System erzeugt einen **Vorgang** vom Typ `NEUGESCHAEFT` mit Beschreibung `Fahrzeugwechsel – Nachfolger von {Alt-Vertragsnummer}`
27. System erzeugt den initialen **Vertragsstand** (Version 1) mit den erfassten/übernommenen Daten
28. System erzeugt einen Eintrag in der **SfKlassenHistorie** des Neuvertrags mit Änderungsgrund `FAHRZEUGWECHSEL` und Referenz auf den Alt-Vertrag

**eVB-Nummer erzeugen:**
29. System erzeugt eine neue **eVB-Nummer** für den Neuvertrag und meldet diese beim GDV (→ UC-KFZ-02)

### Phase 6: Folgeprozesse
30. **S2 (Inkasso):** 
    - Alt-Vertrag: Beitragserstattung pro rata temporis ab Wechseldatum → Kafka-Topic `vertrag.inkasso.beitragsforderungen`
    - Neu-Vertrag: Neue Beitragsforderung ab Wechseldatum → Kafka-Topic `vertrag.inkasso.beitragsforderungen`
31. **S1 (Provision):**
    - Alt-Vertrag: Provisionsereignis `STORNO` → Kafka-Topic `vertrag.provision.ereignisse`
    - Neu-Vertrag: Provisionsereignis `NEUGESCHAEFT` → Kafka-Topic `vertrag.provision.ereignisse`
32. **S5 (Druck):**
    - Alt-Vertrag: Stornierungsdokument (Dokumenttyp `KUENDIGUNG_FAHRZEUGWECHSEL`) mit Hinweis auf Neuvertrag
    - Neu-Vertrag: Versicherungsschein (Dokumenttyp `VERSICHERUNGSSCHEIN`) mit Hinweis auf Fahrzeugwechsel und übernommener SF-Klasse
33. System publiziert die Events `AltVertragBeendetEvent` und `NeuVertragAngelegtEvent` mit Querverweisen

## Alternativszenarien

- **A1: Übergangszeit – Ruheversicherung für Alt-Fahrzeug**
  Das neue Fahrzeug ist bereits zugelassen, aber das alte Fahrzeug soll noch eine kurze Zeit versichert bleiben (z. B. noch nicht verkauft). → Der Alt-Vertrag wird nicht sofort beendet, sondern in eine **Ruheversicherung** (→ UC-KFZ-06) überführt. Die Ruheversicherung bietet beitragsfreien HP-Schutz für max. 18 Monate. Die SF-Klasse des Neuvertrags wird trotzdem sofort aus der aktuellen SF-Klasse des Alt-Vertrags übernommen (Snapshot bei Wechsel). Beide Verträge laufen parallel, bis die Ruheversicherung endet.

- **A2: Gleichzeitiger Produktwechsel**
  Der Versicherungsnehmer möchte beim Fahrzeugwechsel gleichzeitig den Deckungsumfang ändern (z. B. von HP+VK auf nur HP+TK, weil das neue Fahrzeug älter ist). → Dies wird in Phase 3 (Schritt 12) abgebildet: Der Benutzer passt den Deckungsumfang des Neuvertrags individuell an. Die VK-SF-Klasse wird trotzdem in der SfKlassenHistorie dokumentiert (ruhend bis ggf. spätere VK-Aktivierung).

- **A3: Wechseldatum in der Zukunft**
  Das neue Fahrzeug wird erst zu einem zukünftigen Zeitpunkt geliefert/zugelassen (z. B. Neuwagen-Bestellung). → Der Fahrzeugwechsel wird mit Wechseldatum in der Zukunft erfasst und im Status `GEPLANT` gespeichert. Ein **Scheduler-Job** führt den Wechsel zum Stichtag automatisch aus. Der Alt-Vertrag bleibt bis dahin `AKTIV`. Bis zum Stichtag kann der Wechsel storniert werden.

- **A4: Fahrzeugwechsel ohne bekanntes Kennzeichen**
  Das neue Kennzeichen ist zum Zeitpunkt des Wechsels noch nicht bekannt (z. B. noch nicht zugelassen). → Der Neuvertrag wird ohne amtliches Kennzeichen angelegt. Das Kennzeichen wird nachträglich ergänzt, sobald es bekannt ist (Nachtrag via UC-05). Die eVB-Nummer wird trotzdem sofort erzeugt und kann bei der späteren Zulassung verwendet werden.

- **A5: Fahrzeugwechsel über Self-Service-Portal**
  Der VN startet den Fahrzeugwechsel selbständig über das Kundenportal. → Das Portal erfasst die Fahrzeugdaten (HSN/TSN), zeigt die Beitragsvorschau und leitet den Antrag zur internen Prüfung weiter. Eine Schwebe wird erzeugt, und der Innendienst bestätigt oder korrigiert den Wechsel. Dunkelverarbeitung ist möglich, wenn alle Plausibilitäten bestanden werden.

- **A6: Fahrzeugwechsel durch Außendienst**
  Der Außendienst erfasst den Fahrzeugwechsel im Außendienst-System und reicht den Antrag ein. → Der Ablauf ist identisch, jedoch mit eingeschränkten Kompetenzen (z. B. keine Sondernachlässe ohne Innendienst-Freigabe).

## Fehlerfälle

- **F1: HSN/TSN nicht im Typklassenverzeichnis**
  Das neue Fahrzeug wird anhand der HSN/TSN nicht im GDV-Typklassenverzeichnis gefunden. → System zeigt Fehler: „Das Fahrzeug mit HSN {hsn} / TSN {tsn} ist nicht im Typklassenverzeichnis bekannt." Manuelle Erfassung der Fahrzeugdaten und Typklassen durch den Sachbearbeiter ist möglich (Kompetenz `KFZ_MANUELL_ERFASSEN`).

- **F2: Alt-Vertrag nicht aktiv**
  Der Alt-Vertrag ist nicht im Status `AKTIV`. → System zeigt Fehler: „Der Vertrag {Vertragsnummer} ist im Status {Status} und kann nicht für einen Fahrzeugwechsel verwendet werden."

- **F3: Laufender VWB-Vorgang oder SFR-Transfer**
  Für den Alt-Vertrag läuft ein VWB-Verfahren (SF-Klasse noch nicht bestätigt) oder ein SF-Transfer. → System zeigt Warnung: „Für den Vertrag {Vertragsnummer} läuft ein {Vorgangstyp}. Der Fahrzeugwechsel kann erst nach Abschluss durchgeführt werden." Sachbearbeiter kann mit Kompetenz `FAHRZEUGWECHSEL_OVERRIDE` den Wechsel auf Basis der vorläufigen SF-Klasse trotzdem durchführen.

- **F4: eVB-Erzeugung fehlgeschlagen**
  Die eVB-Nummer für den Neuvertrag kann nicht erzeugt werden (GDV-Schnittstelle nicht erreichbar). → Der Fahrzeugwechsel wird trotzdem intern angelegt. Die eVB-Erzeugung wird in die Outbox-Tabelle geschrieben und asynchron nachgeholt. System zeigt Hinweis: „Die eVB-Nummer konnte nicht sofort erzeugt werden und wird automatisch nachgereicht."

- **F5: Prämienberechnung fehlgeschlagen**
  Die Prämie für den Neuvertrag kann nicht berechnet werden (z. B. fehlende Typklasse im Tarif). → System zeigt Fehler: „Die Prämie konnte nicht berechnet werden: {Detail}." Der Fahrzeugwechsel kann nicht abgeschlossen werden, bis das Tarifproblem behoben ist.

- **F6: Kompetenz fehlt**
  Der Benutzer hat nicht die Kompetenz `VERTRAG_FAHRZEUGWECHSEL`. → HTTP 403 Forbidden.

- **F7: Doppelter Fahrzeugwechsel**
  Für den Alt-Vertrag existiert bereits ein aktiver Fahrzeugwechsel-Vorgang (Status `ENTWURF` oder `GEPLANT`). → System zeigt Fehler: „Für den Vertrag {Vertragsnummer} liegt bereits ein Fahrzeugwechsel-Vorgang vor ({Vorgangsnummer})." Der bestehende Vorgang muss zuerst abgeschlossen oder storniert werden.

## Geschäftsregeln

| GR-Nr. | Regel | Auswirkung |
|--------|-------|-----------|
| GR-KFZ-FW01 | Ein Fahrzeugwechsel erzeugt immer einen **neuen Vertrag** – der Alt-Vertrag wird beendet, der Neuvertrag ist unabhängig | Kein Nachtrag auf dem Alt-Vertrag, saubere Vertragstrennung |
| GR-KFZ-FW02 | Das **Wechseldatum** entspricht dem Zulassungsdatum des neuen Fahrzeugs (Standard) oder einem vom VN gewählten Datum | Vertragsbeginn Neu = Wechseldatum, Vertragsende Alt = Wechseldatum − 1 Tag |
| GR-KFZ-FW03 | Die **SF-Klasse** (HP und VK) wird vom Alt-Vertrag auf den Neuvertrag übernommen (Änderungsgrund `FAHRZEUGWECHSEL` in SfKlassenHistorie) | Keine SF-Rückstufung durch den Wechsel, kontinuierliche SF-Historie |
| GR-KFZ-FW04 | Bei Beitragserstattung für den Alt-Vertrag gilt **pro rata temporis** ab dem Wechseldatum | Tagesgenaue Abrechnung, keine doppelte Prämie für Überlappungstage |
| GR-KFZ-FW05 | Der **RabattSchutz** (LVM-RabattSchutz) wird nicht automatisch auf den Neuvertrag übernommen, muss explizit gebucht werden | Neubuchung erforderlich, da vertragsgebunden und ggf. SF-Klasse-Bedingung (≥ SF4) prüfbar |
| GR-KFZ-FW06 | Aktive eVB-Nummern des Alt-Vertrags werden beim GDV storniert – **außer** die eVB wurde bereits bei der Zulassungsstelle verwendet (Status `VERWENDET`) | Vermeidung ungültiger eVB-Stornierungen |
| GR-KFZ-FW07 | Bei Fahrzeugartenwechsel (z. B. PKW → Wohnmobil) werden nicht anwendbare Zusatzbausteine automatisch entfernt und der Benutzer wird informiert | Keine ungültigen Produktkombinationen im Neuvertrag |
| GR-KFZ-FW08 | Der Alt-Vertrag speichert im JSONB `spartenspezifische_daten` eine Referenz auf den Neuvertrag (`nachfolge_vertrag_id`) und umgekehrt (`ursprung_vertrag_id`) – als **weiche Referenz** ohne FK-Constraint | Nachvollziehbarkeit ohne feste Kopplung |
| GR-KFZ-FW09 | Wird der Neuvertrag innerhalb von **14 Tagen** widerrufen (Widerrufsrecht), wird der Alt-Vertrag automatisch reaktiviert (Status `AKTIV`), sofern er nicht bereits `ABGELAUFEN` ist | Schutz des VN – kein versicherungsloser Zustand nach Widerruf |
| GR-KFZ-FW10 | Die **VK-SF-Klasse** wird auch dann in der SfKlassenHistorie des Neuvertrags gespeichert, wenn der Neuvertrag keine VK enthält (Status `RUHEND`) | SF-Klasse geht nicht verloren bei späterem VK-Abschluss |

## Daten (Ein-/Ausgabe)

### Eingabedaten

| Feld | Typ | Pflicht | Validierung | Beschreibung |
|------|-----|---------|-------------|-------------|
| alt_vertrag_id | UUID | ✅ | Aktiver KFZ-Vertrag mit HP | Vertrag für das bisherige Fahrzeug |
| wechseldatum | Date (ISO 8601) | ✅ | ≥ heute, ≤ 12 Monate in Zukunft | Stichtag des Fahrzeugwechsels |
| neues_fahrzeug.hsn | String(4) | ✅ | 4-stellig, im Typklassenverzeichnis | Herstellerschlüsselnummer |
| neues_fahrzeug.tsn | String(3) | ✅ | 3-stellig, im Typklassenverzeichnis | Typschlüsselnummer |
| neues_fahrzeug.fin | String(17) | ⬜ | 17-stellig, Prüfziffer | Fahrzeugidentnummer (optional bei Neuzulassung) |
| neues_fahrzeug.erstzulassung | Date (ISO 8601) | ✅ | ≤ heute (+ 12 Monate bei geplanter Zulassung) | Erstzulassungsdatum |
| neues_fahrzeug.amtliches_kennzeichen | String | ⬜ | Gültiges DE-Kennzeichenformat | Amtliches Kennzeichen (optional, kann nachgereicht werden) |
| deckungsumfang | Enum | ✅ | `HP`, `HP_TK`, `HP_VK` | Gewünschter Versicherungsumfang |
| fahrerkreis.juengster_fahrer_alter | Integer | ✅ | 17–99 | Alter des jüngsten Fahrers |
| fahrerkreis.aeltester_fahrer_alter | Integer | ⬜ | 17–99, ≥ jüngster | Alter des ältesten Fahrers |
| fahrerkreis.anzahl_fahrer | Enum | ✅ | `NUR_VN`, `VN_UND_PARTNER`, `BESTIMMTE_PERSONEN`, `BELIEBIGE_PERSONEN` | Fahrerkreis-Definition |
| nutzungsart | Enum | ✅ | `PRIVAT`, `GEWERBLICH`, `PENDELFAHRTEN`, `PRIVAT_UND_PENDEL` | Nutzungsart des Fahrzeugs |
| jaehrliche_fahrleistung_km | Integer | ✅ | 1.000–100.000 | Erwartete jährliche Fahrleistung |
| stellplatz | Enum | ✅ | `GARAGE`, `CARPORT`, `TIEFGARAGE`, `STELLPLATZ`, `STRASSE` | Üblicher Abstellort |
| selbstbeteiligung_tk | BigDecimal | ⬜ | 0 / 150 / 300 / 500 / 1000 | Selbstbeteiligung Teilkasko (EUR) |
| selbstbeteiligung_vk | BigDecimal | ⬜ | 150 / 300 / 500 / 1000 / 2500 | Selbstbeteiligung Vollkasko (EUR) |
| zahlungsweise | Enum | ✅ | `JAEHRLICH`, `HALBJAEHRLICH`, `VIERTELJAEHRLICH`, `MONATLICH` | Beitragszahlweise |
| zusatzbausteine | Array\<Enum\> | ⬜ | Gültige Baustein-IDs | z. B. `SCHUTZBRIEF`, `RABATTSCHUTZ`, `FAHRERKASKO`, `AUSLANDPLUS` |
| ruheversicherung_alt | Boolean | ⬜ | – | `true` = Alt-Fahrzeug in Ruheversicherung überführen statt Stornierung |
| bemerkung | String(1000) | ⬜ | Max. 1000 Zeichen | Freitext-Bemerkung / Aktennotiz |

### Ausgabedaten

| Feld | Typ | Beschreibung |
|------|-----|-------------|
| fahrzeugwechsel_id | UUID | Eindeutige ID des Fahrzeugwechsel-Vorgangs |
| status | Enum | Status des Wechsels (`DURCHGEFUEHRT`, `GEPLANT`, `GENEHMIGUNG_ERFORDERLICH`) |
| wechseldatum | Date | Stichtag des Fahrzeugwechsels |
| alt_vertrag.vertragsnummer | String | Vertragsnummer des alten Vertrags |
| alt_vertrag.status | Enum | Neuer Status (`GEKUENDIGT`) |
| alt_vertrag.vertragsende | Date | Ende-Datum (= Wechseldatum − 1 Tag) |
| alt_vertrag.sf_klasse_hp | String | SF-Klasse HP zum Zeitpunkt des Wechsels |
| alt_vertrag.sf_klasse_vk | String | SF-Klasse VK zum Zeitpunkt des Wechsels |
| alt_vertrag.beitragserstattung_betrag | BigDecimal | Pro-rata-temporis-Erstattung |
| alt_vertrag.vorgang_id | UUID | ID des Stornierungsvorgangs |
| alt_vertrag.druckauftrag_id | UUID | ID des Kündigungsdokuments |
| neu_vertrag.vertrag_id | UUID | ID des neuen Vertrags |
| neu_vertrag.vertragsnummer | String | Neue Vertragsnummer |
| neu_vertrag.status | Enum | Status (`AKTIV`) |
| neu_vertrag.vertragsbeginn | Date | Vertragsbeginn (= Wechseldatum) |
| neu_vertrag.sf_klasse_hp | String | Übernommene SF-Klasse HP |
| neu_vertrag.sf_klasse_vk | String | Übernommene SF-Klasse VK (ggf. ruhend) |
| neu_vertrag.jahresbeitrag | BigDecimal | Neuer Jahresbeitrag |
| neu_vertrag.vorgang_id | UUID | ID des Neugeschäft-Vorgangs |
| neu_vertrag.evb_nummer | String | Neue eVB-Nummer |
| neu_vertrag.druckauftrag_id | UUID | ID des Versicherungsscheins |
| beitragsdifferenz | BigDecimal | Differenz Alt-Beitrag vs. Neu-Beitrag (jährlich) |

## API-Endpunkte

> Ergänzung zur KFZ-Sparten-API (→ 09_schnittstellen.md, Abschnitt 2.7)

| Methode | Endpunkt | Beschreibung | Kompetenz |
|---------|---------|-------------|-----------|
| `POST` | `/api/v1/kfz/fahrzeugwechsel/vorschau` | Vorschau berechnen (ohne Durchführung) | `VERTRAG_LESEN` |
| `POST` | `/api/v1/kfz/fahrzeugwechsel` | Fahrzeugwechsel durchführen | `VERTRAG_FAHRZEUGWECHSEL` |
| `GET` | `/api/v1/kfz/fahrzeugwechsel` | Fahrzeugwechsel suchen (Filter: status, vertrag_id, partner_id) | `VERTRAG_LESEN` |
| `GET` | `/api/v1/kfz/fahrzeugwechsel/{id}` | Fahrzeugwechsel-Details | `VERTRAG_LESEN` |
| `DELETE` | `/api/v1/kfz/fahrzeugwechsel/{id}` | Geplanten Fahrzeugwechsel stornieren | `VERTRAG_FAHRZEUGWECHSEL` |

### POST `/api/v1/kfz/fahrzeugwechsel/vorschau` – Beitragsvorschau

**Request:**
```json
{
  "alt_vertrag_id": "f6a7b8c9-d0e1-2345-fabc-456789012345",
  "wechseldatum": "2026-07-01",
  "neues_fahrzeug": {
    "hsn": "0603",
    "tsn": "BNJ",
    "fin": "WVWZZZ3CZWE654321",
    "erstzulassung": "2026-06-15",
    "amtliches_kennzeichen": "MS-NF 4567"
  },
  "deckungsumfang": "HP_VK",
  "fahrerkreis": {
    "juengster_fahrer_alter": 35,
    "aeltester_fahrer_alter": 55,
    "anzahl_fahrer": "VN_UND_PARTNER"
  },
  "nutzungsart": "PRIVAT_UND_PENDEL",
  "jaehrliche_fahrleistung_km": 15000,
  "stellplatz": "GARAGE",
  "selbstbeteiligung_tk": 150,
  "selbstbeteiligung_vk": 500,
  "zahlungsweise": "JAEHRLICH",
  "zusatzbausteine": ["SCHUTZBRIEF", "RABATTSCHUTZ"]
}
```

**Response (200 OK):**
```json
{
  "alt_vertrag": {
    "vertrag_id": "f6a7b8c9-d0e1-2345-fabc-456789012345",
    "vertragsnummer": "VN-2026-100001",
    "partner_name": "Max Mustermann",
    "fahrzeug": "MS-LV 1234 | VW Golf VIII 1.5 TSI",
    "sf_klasse_hp": "SF12",
    "sf_klasse_vk": "SF8",
    "jahresbeitrag": "487.32",
    "beitragserstattung": {
      "betrag": "243.66",
      "zeitraum_ab": "2026-07-01",
      "zeitraum_bis": "2026-12-31"
    }
  },
  "neu_vertrag": {
    "fahrzeug": "MS-NF 4567 | VW Tiguan 2.0 TDI",
    "fahrzeugart": "PKW",
    "typklasse_hp": 16,
    "typklasse_tk": 20,
    "typklasse_vk": 22,
    "regionalklasse_hp": 4,
    "regionalklasse_tk": 3,
    "regionalklasse_vk": 2,
    "sf_klasse_hp": "SF12",
    "sf_klasse_vk": "SF8",
    "jahresbeitrag": "612.80",
    "berechnungsdetails": {
      "basisbeitrag_hp": 245.00,
      "basisbeitrag_tk": 89.00,
      "basisbeitrag_vk": 178.00,
      "sf_faktor_hp": 0.34,
      "sf_faktor_vk": 0.41,
      "typklasse_faktor_hp": 1.15,
      "typklasse_faktor_vk": 1.28,
      "regionalklasse_faktor_hp": 1.05,
      "zahlungsweise_faktor": 1.00
    }
  },
  "beitragsdifferenz": "+125.48",
  "wechseldatum": "2026-07-01",
  "uebernommene_daten": {
    "sf_klasse": true,
    "fahrerkreis": true,
    "zusatzbausteine": ["SCHUTZBRIEF", "RABATTSCHUTZ"],
    "nicht_uebernommen": [],
    "warnungen": []
  }
}
```

### POST `/api/v1/kfz/fahrzeugwechsel` – Fahrzeugwechsel durchführen

**Request:**
```json
{
  "alt_vertrag_id": "f6a7b8c9-d0e1-2345-fabc-456789012345",
  "wechseldatum": "2026-07-01",
  "neues_fahrzeug": {
    "hsn": "0603",
    "tsn": "BNJ",
    "fin": "WVWZZZ3CZWE654321",
    "erstzulassung": "2026-06-15",
    "amtliches_kennzeichen": "MS-NF 4567"
  },
  "deckungsumfang": "HP_VK",
  "fahrerkreis": {
    "juengster_fahrer_alter": 35,
    "aeltester_fahrer_alter": 55,
    "anzahl_fahrer": "VN_UND_PARTNER"
  },
  "nutzungsart": "PRIVAT_UND_PENDEL",
  "jaehrliche_fahrleistung_km": 15000,
  "stellplatz": "GARAGE",
  "selbstbeteiligung_tk": 150,
  "selbstbeteiligung_vk": 500,
  "zahlungsweise": "JAEHRLICH",
  "zusatzbausteine": ["SCHUTZBRIEF", "RABATTSCHUTZ"],
  "ruheversicherung_alt": false,
  "bemerkung": "Fahrzeugwechsel VW Golf auf VW Tiguan, Neuwagen-Lieferung am 15.06."
}
```

**Response (200 OK – Fahrzeugwechsel durchgeführt):**
```json
{
  "fahrzeugwechsel_id": "a2b3c4d5-e6f7-8901-abcd-234567890123",
  "status": "DURCHGEFUEHRT",
  "wechseldatum": "2026-07-01",
  "alt_vertrag": {
    "vertrag_id": "f6a7b8c9-d0e1-2345-fabc-456789012345",
    "vertragsnummer": "VN-2026-100001",
    "status": "GEKUENDIGT",
    "vertragsende": "2026-06-30",
    "sf_klasse_hp": "SF12",
    "sf_klasse_vk": "SF8",
    "beitragserstattung": {
      "betrag": "243.66",
      "zeitraum_ab": "2026-07-01",
      "zeitraum_bis": "2026-12-31"
    },
    "vorgang": {
      "id": "d4e5f6a7-b8c9-0123-defa-234567890123",
      "vorgangsnummer": "VG-2026-000060",
      "vorgangstyp": "STORNIERUNG"
    },
    "druckauftrag_id": "uuid-druck-alt-001"
  },
  "neu_vertrag": {
    "vertrag_id": "b3c4d5e6-f7a8-9012-bcde-345678901234",
    "vertragsnummer": "VN-2026-100042",
    "status": "AKTIV",
    "vertragsbeginn": "2026-07-01",
    "fahrzeug": {
      "amtliches_kennzeichen": "MS-NF 4567",
      "hersteller": "Volkswagen",
      "modell": "Tiguan 2.0 TDI",
      "fin": "WVWZZZ3CZWE654321",
      "hsn": "0603",
      "tsn": "BNJ"
    },
    "sf_klasse_hp": "SF12",
    "sf_klasse_vk": "SF8",
    "jahresbeitrag": "612.80",
    "evb_nummer": "A123B456C",
    "vorgang": {
      "id": "e5f6a7b8-c9d0-1234-efab-345678901234",
      "vorgangsnummer": "VG-2026-000061",
      "vorgangstyp": "NEUGESCHAEFT"
    },
    "druckauftrag_id": "uuid-druck-neu-001"
  },
  "beitragsdifferenz": "+125.48",
  "erstellt_von": "id-schmidt",
  "erstellt_am": "2026-06-20T14:30:00Z"
}
```

**Response (200 OK – Geplanter Fahrzeugwechsel):**
```json
{
  "fahrzeugwechsel_id": "a2b3c4d5-e6f7-8901-abcd-234567890123",
  "status": "GEPLANT",
  "wechseldatum": "2026-09-01",
  "geplante_ausfuehrung": "2026-09-01T00:00:00Z",
  "alt_vertrag": {
    "vertrag_id": "f6a7b8c9-d0e1-2345-fabc-456789012345",
    "vertragsnummer": "VN-2026-100001",
    "status": "AKTIV",
    "hinweis": "Vertrag bleibt bis zum Wechseldatum aktiv."
  },
  "neu_vertrag_vorschau": {
    "jahresbeitrag": "612.80",
    "sf_klasse_hp": "SF12",
    "sf_klasse_vk": "SF8"
  },
  "stornierbar_bis": "2026-08-31T23:59:59Z"
}
```

**Response (409 Conflict – Vertrag nicht bereit):**
```json
{
  "type": "https://api.versicherungsverwaltung.de/errors/fahrzeugwechsel-conflict",
  "title": "Fahrzeugwechsel nicht möglich",
  "status": 409,
  "detail": "Für den Vertrag VN-2026-100001 läuft ein VWB-Verfahren. Der Fahrzeugwechsel kann erst nach Abschluss durchgeführt werden.",
  "errors": [
    {
      "code": "FW-E03",
      "field": "alt_vertrag_id",
      "message": "Laufender VWB-Vorgang auf dem Alt-Vertrag."
    }
  ]
}
```

## Fahrzeugwechsel-Statusmodell

| Status | Beschreibung | Folgestatus |
|--------|-------------|-------------|
| `ENTWURF` | Fahrzeugwechsel angelegt, Daten werden erfasst | DURCHGEFUEHRT, GEPLANT, ABGEBROCHEN |
| `GEPLANT` | Wechseldatum liegt in der Zukunft, Scheduler wartet | DURCHGEFUEHRT, ABGEBROCHEN |
| `DURCHGEFUEHRT` | Fahrzeugwechsel erfolgreich durchgeführt | – (Endzustand) |
| `ABGEBROCHEN` | Fahrzeugwechsel vor Durchführung storniert | – (Endzustand) |
| `WIDERRUFEN` | Neuvertrag widerrufen, Alt-Vertrag reaktiviert (→ GR-KFZ-FW09) | – (Endzustand) |

## Events

| Event | Auslöser | Consumer | Beschreibung |
|-------|---------|---------|-------------|
| `FahrzeugwechselInitiiertEvent` | Phase 1, Schritt 4 | Logging | Fahrzeugwechsel wurde als Entwurf angelegt |
| `AltVertragBeendetEvent` | Phase 5, Schritt 21 | S1, S2, S5 | Alt-Vertrag beendet wegen Fahrzeugwechsel |
| `NeuVertragAngelegtEvent` | Phase 5, Schritt 25 | S1, S2, S5 | Neuvertrag aus Fahrzeugwechsel angelegt |
| `EvbStornoEvent` | Phase 5, Schritt 22 | GDV-eVB-Service | eVB des Alt-Vertrags storniert |
| `EvbErzeugtEvent` | Phase 5, Schritt 29 | GDV-eVB-Service | Neue eVB für Neuvertrag erzeugt |
| `FahrzeugwechselAbgeschlossenEvent` | Phase 6 abgeschlossen | Logging, Reporting | Gesamtvorgang inkl. Folgeprozesse abgeschlossen |
| `FahrzeugwechselGeplantEvent` | A3: Wechseldatum in Zukunft | Scheduler | Geplanter Wechsel wartet auf Stichtag |
| `FahrzeugwechselWiderrufenEvent` | GR-KFZ-FW09: Widerruf | Storno-Service | Neuvertrag widerrufen, Alt-Vertrag reaktiviert |

## SfKlassenHistorie-Einträge (Beispiel)

Nach einem Fahrzeugwechsel von VN-2026-100001 (SF12 HP / SF8 VK) → VN-2026-100042:

**Alt-Vertrag (VN-2026-100001) – kein neuer SfKlassenHistorie-Eintrag, da Vertrag endet.**

**Neu-Vertrag (VN-2026-100042):**
```json
{
  "vertrag_id": "b3c4d5e6-f7a8-9012-bcde-345678901234",
  "sf_klasse_hp": "SF12",
  "sf_klasse_vk": "SF8",
  "gueltig_ab": "2026-07-01",
  "aenderungsgrund": "FAHRZEUGWECHSEL",
  "referenz_vertrag_id": "f6a7b8c9-d0e1-2345-fabc-456789012345",
  "bemerkung": "SF-Klasse übernommen von Alt-Vertrag VN-2026-100001 im Rahmen eines Fahrzeugwechsels",
  "erstellt_von": "id-schmidt",
  "erstellt_am": "2026-06-20T14:30:00Z"
}
```

## Wireframe / Skizze

### Fahrzeugwechsel – Vergleichsansicht (Phase 4)

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  Fahrzeugwechsel – Vorschau                                                  │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Wechseldatum: [ 01.07.2026  📅 ]                                           │
│                                                                              │
│  ┌─── Alt-Vertrag ─────────────────┐  ┌─── Neu-Vertrag ─────────────────┐   │
│  │ VN-2026-100001                  │  │ (wird angelegt)                  │   │
│  │ MS-LV 1234 | VW Golf VIII      │  │ MS-NF 4567 | VW Tiguan 2.0 TDI  │   │
│  │                                 │  │                                  │   │
│  │ SF-Klasse HP: SF12              │  │ SF-Klasse HP: SF12  ✅ übernom.  │   │
│  │ SF-Klasse VK: SF8               │  │ SF-Klasse VK: SF8   ✅ übernom.  │   │
│  │                                 │  │                                  │   │
│  │ Deckung: HP + VK                │  │ Deckung: HP + VK                │   │
│  │ Jahresbeitrag: 487,32 €         │  │ Jahresbeitrag: 612,80 €         │   │
│  │                                 │  │                                  │   │
│  │ → Status: GEKÜNDIGT             │  │ → Status: AKTIV                  │   │
│  │ → Ende: 30.06.2026              │  │ → Beginn: 01.07.2026            │   │
│  │ → Erstattung: 243,66 €          │  │                                  │   │
│  └─────────────────────────────────┘  └──────────────────────────────────┘   │
│                                                                              │
│  Beitragsdifferenz (jährlich): + 125,48 €                                    │
│                                                                              │
│  ☐ Ruheversicherung für Alt-Fahrzeug aktivieren (→ UC-KFZ-06)               │
│                                                                              │
│                        [ Abbrechen ]  [ Fahrzeugwechsel durchführen ]        │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

### Bestätigungsdialog

```
┌───────────────────────────────────────────────────────────┐
│  ⚠️  Fahrzeugwechsel bestätigen                           │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  Alt-Vertrag: VN-2026-100001 (MS-LV 1234)                │
│  → wird zum 30.06.2026 beendet                            │
│  → Erstattung: 243,66 €                                   │
│                                                           │
│  Neu-Vertrag: (wird angelegt)                             │
│  → Fahrzeug: MS-NF 4567 | VW Tiguan 2.0 TDI              │
│  → Beginn: 01.07.2026                                     │
│  → Jahresbeitrag: 612,80 €                                │
│  → SF-Klasse: SF12 (HP) / SF8 (VK) übernommen            │
│                                                           │
│  Der Alt-Vertrag wird unwiderruflich beendet.             │
│  Sie erhalten ein Kündigungs- und ein Policierungsdoku-   │
│  ment.                                                    │
│                                                           │
│            [ Abbrechen ]  [ Bestätigen ]                  │
└───────────────────────────────────────────────────────────┘
```

## Sequenzdiagramm

```
Benutzer      UI              KFZ-Service       Vertrag-Service     S2 (Inkasso)    S5 (Druck)   GDV
   │           │                   │                  │                  │               │          │
   │ Aktion    │                   │                  │                  │               │          │
   │ "Fzg-     │                   │                  │                  │               │          │
   │  wechsel" │                   │                  │                  │               │          │
   │──────────►│  POST /vorschau   │                  │                  │               │          │
   │           │──────────────────►│                  │                  │               │          │
   │           │                   │  Alt-Vertrag     │                  │               │          │
   │           │                   │  laden           │                  │               │          │
   │           │                   │─────────────────►│                  │               │          │
   │           │                   │  Typklasse       │                  │               │          │
   │           │                   │  ermitteln (GDV) │                  │               │          │
   │           │                   │─────────────────────────────────────────────────────────────►│
   │           │                   │  Prämie berechn. │                  │               │          │
   │           │                   │──┐               │                  │               │          │
   │           │                   │  │               │                  │               │          │
   │           │  Vorschau         │◄─┘               │                  │               │          │
   │           │◄──────────────────│                  │                  │               │          │
   │ Vorschau  │                   │                  │                  │               │          │
   │◄──────────│                   │                  │                  │               │          │
   │           │                   │                  │                  │               │          │
   │ Bestätig. │                   │                  │                  │               │          │
   │──────────►│  POST /fzg-wechs. │                  │                  │               │          │
   │           │──────────────────►│                  │                  │               │          │
   │           │                   │  Alt-Vertrag     │                  │               │          │
   │           │                   │  beenden         │                  │               │          │
   │           │                   │─────────────────►│                  │               │          │
   │           │                   │  Neuvertrag      │                  │               │          │
   │           │                   │  anlegen         │                  │               │          │
   │           │                   │─────────────────►│                  │               │          │
   │           │                   │  eVB erzeugen    │                  │               │          │
   │           │                   │─────────────────────────────────────────────────────────────►│
   │           │                   │  eVB Alt storn.  │                  │               │          │
   │           │                   │─────────────────────────────────────────────────────────────►│
   │           │                   │                  │ AltVertragBeendetEvent           │          │
   │           │                   │                  │─────────────────►│               │          │
   │           │                   │                  │ NeuVertragAngelegtEvent          │          │
   │           │                   │                  │─────────────────►│               │          │
   │           │                   │                  │──────────────────────────────────►│         │
   │           │                   │                  │  Druck: Kündigung + Police       │          │
   │           │  Ergebnis         │                  │                  │               │          │
   │           │◄──────────────────│                  │                  │               │          │
   │ Ergebnis  │                   │                  │                  │               │          │
   │◄──────────│                   │                  │                  │               │          │
```

## Nachbedingungen
- Der Alt-Vertrag ist im Status `GEKUENDIGT` (bzw. `ABGELAUFEN` ab Wechseldatum)
- Ein Stornierungsvorgang und ein neuer Vertragsstand (mit `gueltig_bis`) sind auf dem Alt-Vertrag angelegt
- Ein neuer Vertrag mit eigenem Antrag, Vorgang und Vertragsstand existiert
- Die SF-Klasse (HP + VK) ist vollständig auf den Neuvertrag übertragen und in der SfKlassenHistorie dokumentiert
- Die eVB-Nummer des Alt-Vertrags ist beim GDV storniert (sofern Status ≠ `VERWENDET`)
- Eine neue eVB-Nummer ist für den Neuvertrag erzeugt und beim GDV gemeldet
- Beitragserstattung (Alt) und Beitragsforderung (Neu) sind an S2 (Inkasso) übermittelt
- Provisionsereignisse sind an S1 publiziert (STORNO für Alt, NEUGESCHAEFT für Neu)
- Druckdokumente sind an S5 übermittelt (Kündigung für Alt, Versicherungsschein für Neu)
- Beide Verträge referenzieren einander via `spartenspezifische_daten` (weiche Referenz)

## Akzeptanzkriterien
- [ ] Der Benutzer kann über die Vertragsdetailansicht die Aktion „Fahrzeugwechsel" auslösen
- [ ] Das System zeigt eine Vergleichsansicht mit Alt-Vertrag und geplantem Neuvertrag (inkl. Beitragsvorschau)
- [ ] Vertragsdaten (SF-Klasse, Fahrerkreis, Nutzungsart, Stellplatz etc.) werden als Vorschlag übernommen und sind änderbar
- [ ] Die SF-Klasse (HP + VK) wird an den Neuvertrag übergeben und in der SfKlassenHistorie dokumentiert (Änderungsgrund `FAHRZEUGWECHSEL`)
- [ ] Der Alt-Vertrag wird zum Wechseldatum beendet (Status `GEKUENDIGT`, Stornierungsgrund `FAHRZEUGWECHSEL`)
- [ ] Ein neuer Vertrag mit neuer Vertragsnummer wird angelegt (Status `AKTIV`, Vorgangstyp `NEUGESCHAEFT`)
- [ ] Pro-rata-temporis-Erstattung für den Alt-Vertrag wird korrekt berechnet und an S2 übermittelt
- [ ] Eine neue eVB-Nummer wird für den Neuvertrag erzeugt
- [ ] Die eVB-Nummer des Alt-Vertrags wird storniert (sofern Status ≠ `VERWENDET`)
- [ ] Druckdokumente werden für beide Verträge erzeugt (Kündigung + Versicherungsschein)
- [ ] Beide Verträge enthalten eine weiche Referenz aufeinander (`ursprung_vertrag_id` / `nachfolge_vertrag_id`)
- [ ] Bei Wechseldatum in der Zukunft wird der Wechsel als `GEPLANT` gespeichert und per Scheduler ausgeführt
- [ ] Ein geplanter Fahrzeugwechsel kann vor dem Stichtag storniert werden
- [ ] Bei Widerruf des Neuvertrags (innerhalb 14 Tagen) wird der Alt-Vertrag reaktiviert
- [ ] Optional: Ruheversicherung für das Alt-Fahrzeug kann direkt im Fahrzeugwechsel-Prozess aktiviert werden (→ UC-KFZ-06)
- [ ] Bei Fahrzeugartenwechsel werden nicht anwendbare Zusatzbausteine automatisch entfernt und der Benutzer informiert

## Offene Fragen

| Nr. | Frage | Kontext |
|-----|-------|---------|
| UC-KFZ-04-01 | Soll bei einem Fahrzeugwechsel automatisch eine Plausibilitätsprüfung gegen Betrug erfolgen (z. B. auffällig häufige Fahrzeugwechsel innerhalb kurzer Zeit)? | Schadenverhütung / Compliance |
| UC-KFZ-04-02 | Muss das System prüfen, ob der VN für das neue Fahrzeug eine ausreichende Fahrerlaubnis besitzt (z. B. Klasse A für Motorrad)? | Deckungsschutz / Haftung |
| UC-KFZ-04-03 | Soll die Beitragserstattung des Alt-Vertrags automatisch mit der ersten Beitragsforderung des Neuvertrags verrechnet werden? | Zahlungsverkehr / UX |
