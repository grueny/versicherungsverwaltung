# Projektvision – Versicherungsverwaltung

## Projektname
Versicherungsverwaltung

## Vision

Ein zentrales, **spartenübergreifendes** Bestandsführungssystem zur Verwaltung von Versicherungspolicen über deren gesamten Lebenszyklus hinweg – von der Anbahnung im Vertrieb über den Vertragsabschluss und die laufende Vertragswartung bis hin zur Vertragsbeendigung. Sämtliche Veränderungen an Verträgen werden revisionssicher dokumentiert, sodass jederzeit ein lückenloser Änderungsverlauf nachvollzogen werden kann.

Das System verfolgt einen **spartenübergreifend einheitlichen Ansatz**: Unterschiedliche Versicherungssparten durchlaufen die gleichen bzw. ähnlichen Prozesse, um Synergieeffekte zu nutzen. Spartenübergreifende Aspekte (z. B. Navigation, Vertragsverwaltung, Kundenansicht) werden in der Benutzeroberfläche einheitlich gestaltet.

Eine **Sparte** ist hierbei als **Konfiguration** zu verstehen, bestehend aus:
- **Produkten** – den versicherbaren Produkten und deren Merkmalen/Tarifen
- **Spartenspezifischen Prozessen** – fachliche Abläufe, die über den generischen Kernprozess hinaus gelten
- **Plausibilitäten** – spartenspezifische Validierungen und Geschäftsregeln

Das System stellt einen generischen Prozessrahmen bereit, der durch die Spartenkonfiguration um produkt-, prozess- und plausibilitätsspezifische Aspekte ergänzt wird.

## Ziele

- Ganzheitliche Verwaltung von Versicherungspolicen über den gesamten Vertragslebenszyklus
- **Spartenübergreifend einheitliche Prozesse**, um Synergieeffekte bei Entwicklung, Schulung und Betrieb zu nutzen
- **Konsistente Benutzeroberfläche** über alle Sparten hinweg für spartenübergreifende Aspekte (Navigation, Vertragsübersichten, Kundenansicht)
- **Flexibel erweiterbar** für spartenspezifische Besonderheiten, ohne die einheitliche Grundstruktur zu brechen
- Unterstützung des Vertriebs bei der Anbahnung und dem Abschluss von Verträgen
- Effiziente Vertragswartung durch strukturierte Anpassungs- und Änderungsprozesse
- Revisionssichere Dokumentation aller Vertragsänderungen (vollständige Historisierung)
- Bereitstellung von Schnittstellen zu angrenzenden Systemen im Konzernumfeld

## Scope (Was gehört dazu?)

- **Spartenübergreifender Kern:** Einheitliche Prozesse und UI-Patterns, die für alle Sparten gelten
- **Spartenkonfiguration:** Jede Sparte wird als Konfiguration abgebildet, bestehend aus:
  - *Produkte:* Produktdefinitionen mit Merkmalen, Tarifen und Deckungsbausteinen
  - *Prozesse:* Spartenspezifische Prozessschritte und -abweichungen vom Kernprozess
  - *Plausibilitäten:* Spartenspezifische Validierungen, Prüfregeln und Geschäftslogik
  - → Dokumentation je Sparte unter `12_sparten/<spartenname>/`
- **Vertrieb / Anbahnung:** Angebotserstellung, Antragsverwaltung, Vertragsabschluss
- **Bestandsführung:** Policierung, Vertragsverwaltung, Vertragsauskünfte
- **Vertragswartung:** Vertragsänderungen (Nachträge), Adress-/Stammdatenänderungen, Beitragsanpassungen
- **Vertragsbeendigung:** Kündigungen, Stornierungen, Ablauf
- **Revisionssichere Historisierung:** Lückenlose Protokollierung aller Vertragsänderungen mit Zeitstempel und Benutzer
- **Schnittstellenbereitstellung:** Datenlieferung und -empfang an/von angrenzende(n) Systeme(n)
- **Datenimport / Migration:** Import von Bestandsdaten aus Alt-Systemen zur Übernahme in das neue System

## Nicht-Scope (Was gehört NICHT dazu?)

> Die folgenden Bereiche werden durch eigenständige Systeme abgedeckt und sind über Schnittstellen angebunden:

- **Provisionssystem** – Berechnung und Verwaltung von Vermittlerprovisionen
- **Konzerninkasso / Exkasso** – Beitragseinzug und Auszahlungen
- **Datawarehouse** – Reporting, Analysen und Datenauswertungen
- **Druckschnittstelle** – Erzeugung und Versand von Dokumenten und Briefen
- **Schadenverwaltung** – Schadenaufnahme, -bearbeitung und -regulierung
- **Kundenportal** – Self-Service-Bereich für Versicherungsnehmer
- **Kompetenz-System** – Verwaltung von Rollen und Berechtigungen (Kompetenzen); das Bestandsführungssystem fragt über eine Schnittstelle ab, ob ein Benutzer eine bestimmte Kompetenz (Kompetenz-ID) besitzt

## Zielgruppe
- **Außendienst (Vertrieb):** Vertriebsmitarbeiter, die Kunden vor Ort beraten, Angebote erstellen und Verträge abschließen
- **Innendienst (Sachbearbeitung):** Sachbearbeiter, die Verträge verwalten, Sonderfälle bearbeiten und die Qualitätssicherung übernehmen
- **Kunden (via Kundenportal):** Versicherungsnehmer, die über das Kundenportal Verträge einsehen, abschließen und Änderungen beauftragen (indirekter Zugriff über Schnittstelle S7)

## Erfolgskriterien
<!-- Woran messen wir, ob das Projekt erfolgreich ist? -->
- 
- 

## Rahmenbedingungen
<!-- Budget, Zeitrahmen, regulatorische Vorgaben etc. -->
- **DORA-Konformität:** Das System muss den Anforderungen des Digital Operational Resilience Act (EU-Verordnung 2022/2554) entsprechen. Dies betrifft insbesondere:
  - IKT-Risikomanagement (Art. 5–16): Sicherstellung der operativen Resilienz aller IT-Systeme
  - Meldung IKT-bezogener Vorfälle (Art. 17–23): Fähigkeit zur Erkennung, Klassifizierung und Meldung von Störungen
  - Testen der digitalen operationalen Resilienz (Art. 24–27): Regelmäßige Tests inkl. Penetrationstests (TLPT)
  - IKT-Drittparteirisiko (Art. 28–44): Überwachung und Steuerung von Risiken aus externen IT-Dienstleistern
  - Informationsaustausch (Art. 45): Fähigkeit zum Austausch von Bedrohungsinformationen

## Offene Fragen

- ~~Welche Versicherungssparten sollen initial unterstützt werden und in welcher Reihenfolge werden weitere Sparten angebunden (Leben, Sach, Kranken, KFZ, Haftpflicht, …)?~~ → **Geklärt:** Initial wird mit der Sparte **KFZ** begonnen. Reihenfolge weiterer Sparten noch offen.
- Welche Aspekte sind spartenübergreifend einheitlich und welche werden über die Spartenkonfiguration gesteuert? (→ Abgrenzung Kern vs. Konfiguration)
- ~~Wie wird die Spartenkonfiguration technisch umgesetzt (Datenbank-Konfiguration, DSL, Regelengine, Plugin-Architektur)?~~ → **Geklärt:** Hybrid-Ansatz – Produkte in **Datenbank-Konfiguration**, Plausibilitäten über **Regelengine/Strategy-Pattern**, Prozessabweichungen über **Extension Points/Hooks** im Kernprozess, zusammengeführt durch ein **Sparten-Registry-Pattern**. Details in `11_technische_rahmenbedingungen.md`.
- ~~Werden Produkte, Prozessabweichungen und Plausibilitäten über ein gemeinsames Konfigurationsmodell verwaltet oder getrennt?~~ → **Geklärt:** Getrennt nach Aspekt (Produkte in DB, Regeln in Engine, Prozesse als Code), aber pro Sparte über eine zentrale **Sparten-Registry** zusammengeführt.
- ~~Wie sieht die Datenübernahme aus bestehenden Systemen aus (Migration)?~~ → **Geklärt:** Für die Migration von Alt-Systemen muss ein **Import** vorgesehen werden. Detaillierte Anforderungen an Importformate, -prozesse und -validierungen sind noch zu spezifizieren.
- Welche konkreten Datenformate und Protokolle nutzen die Schnittstellen?
- Gibt es regulatorische Anforderungen (z. B. BaFin, IDD, DSGVO), die besonders berücksichtigt werden müssen? → **Teilweise geklärt:** Das System muss die Vorgaben von **DORA** (Digital Operational Resilience Act, EU-Verordnung 2022/2554) einhalten. Weitere regulatorische Anforderungen (BaFin, IDD, DSGVO) noch offen.
- ~~Wie ist das Berechtigungskonzept aufgebaut (Rollen, Mandanten)?~~ → **Geklärt:** Berechtigungen werden über das **Kompetenzmodell** abgebildet. Das System fragt per Schnittstelle (Kompetenz-ID) ab, ob ein Benutzer eine bestimmte Kompetenz besitzt. Die Rollenverwaltung erfolgt im externen Kompetenz-System.
- ~~Soll das System mandantenfähig sein (z. B. für mehrere Konzerngesellschaften)?~~ → **Geklärt:** Nein, das System muss **nicht mandantenfähig** sein.
