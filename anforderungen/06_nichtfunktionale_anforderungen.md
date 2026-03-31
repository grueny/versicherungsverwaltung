# Nichtfunktionale Anforderungen – Versicherungsverwaltung

## Performance
| NFA-Nr. | Anforderung | Zielwert | Priorität |
|---------|------------|----------|-----------|
| NFA-P01 | Seitenladezeit | Initiale Ladezeit (Cold Start) ≤ 3 Sekunden; Navigation zwischen Ansichten (SPA-Routing) ≤ 1 Sekunde. Angular Lazy Loading pro Spartenmodul, um die initiale Ladezeit gering zu halten. | 🔴 Muss |
| NFA-P02 | API-Antwortzeit | Standard-CRUD-Operationen (Vertrag lesen/schreiben) ≤ 500 ms (95. Perzentil); Komplexe Operationen (Tarifberechnung, Plausibilitätsprüfung via Drools) ≤ 2 Sekunden (95. Perzentil); Suchanfragen im Bestand ≤ 1 Sekunde (95. Perzentil). Gemessen am Spring Boot Actuator Endpoint. | 🔴 Muss |
| NFA-P03 | Gleichzeitige Benutzer | Das System muss mindestens 200 gleichzeitig aktive Benutzer (Innendienst + Außendienst) ohne Degradation unterstützen. Lasttest-Ziel: 500 gleichzeitige Sessions bei akzeptabler Performance (≤ 2× der Normalwerte). | 🔴 Muss |
| NFA-P04 | Datenbankabfragen | Einzelne SQL-Queries ≤ 100 ms (95. Perzentil). Komplexe Reports/Aggregationen ≤ 5 Sekunden. PostgreSQL-Query-Performance wird über pg_stat_statements überwacht. | 🟡 Soll |
| NFA-P05 | Schnittstellenaufrufe | Aufrufe an externe Systeme (S1–S8) dürfen bei Timeout (≤ 5 Sekunden) den laufenden Geschäftsprozess nicht blockieren. Circuit-Breaker-Pattern (Resilience4j) für externe Abhängigkeiten. | 🔴 Muss |

## Sicherheit
| NFA-Nr. | Anforderung | Details | Priorität |
|---------|------------|---------|-----------|
| NFA-S01 | Authentifizierung | Token-basierte Authentifizierung via OAuth2 / JWT (vgl. Technische Rahmenbedingungen). Access-Token-Gültigkeit ≤ 15 Minuten, Refresh-Token ≤ 8 Stunden. Session-Timeout bei Inaktivität nach 30 Minuten. Multi-Faktor-Authentifizierung (MFA) für Innendienst-Zugang empfohlen. | 🔴 Muss |
| NFA-S02 | Autorisierung | Kompetenz-basierte Autorisierung über externes Kompetenz-System (S8). Jede geschützte Aktion erfordert eine Kompetenzprüfung (Kompetenz-ID). Bei Nichtverfügbarkeit des Kompetenz-Systems werden geschützte Funktionen blockiert (Fail-Closed). Autorisierungsergebnisse dürfen für ≤ 5 Minuten gecacht werden. | 🔴 Muss |
| NFA-S03 | Datenverschlüsselung | Transport: TLS 1.3 (mindestens TLS 1.2) für alle Verbindungen (Frontend ↔ Backend, Backend ↔ Datenbank, Backend ↔ externe Systeme). Ruhende Daten: PostgreSQL Transparent Data Encryption (TDE) oder Festplattenverschlüsselung. Sensible Felder (z. B. Bankdaten, Geburtsdatum) werden auf Anwendungsebene verschlüsselt gespeichert. | 🔴 Muss |
| NFA-S04 | Audit-Logging | Alle sicherheitsrelevanten Aktionen (Login, Logout, fehlgeschlagene Logins, Datenänderungen, Kompetenzprüfungen) werden revisionssicher protokolliert. Audit-Logs enthalten: Zeitstempel (UTC), Benutzer-ID, Aktion, betroffene Entität, Ergebnis (Erfolg/Fehler). Logs sind unveränderlich und werden getrennt von Anwendungslogs gespeichert. Aufbewahrung: mindestens 10 Jahre (vgl. NFA-C03). | 🔴 Muss |
| NFA-S05 | DSGVO-Konformität | Recht auf Auskunft: Export aller personenbezogenen Daten eines Kunden in maschinenlesbarem Format (JSON/CSV) innerhalb von 72 Stunden. Recht auf Löschung: Anonymisierung/Pseudonymisierung von Kundendaten nach Ablauf gesetzlicher Aufbewahrungsfristen. Einwilligungsverwaltung: Speicherung und Nachvollziehbarkeit von Einwilligungen. Datensparsamkeit: Nur für den Geschäftszweck notwendige Daten werden erhoben. | 🔴 Muss |
| NFA-S06 | Penetrationstests | Jährliche Penetrationstests durch externen Dienstleister gemäß DORA Art. 24–27 (Testen der digitalen operationalen Resilienz). OWASP Top 10 als Mindestprüfkatalog. Behebung kritischer Schwachstellen innerhalb von 72 Stunden. | 🔴 Muss |
| NFA-S07 | Eingabevalidierung | Serverseitige Validierung aller Eingaben (Spring Validation + Drools-Regeln). Schutz gegen SQL-Injection, XSS und CSRF. Content Security Policy (CSP) Header im Frontend. | 🔴 Muss |

## Verfügbarkeit
| NFA-Nr. | Anforderung | Zielwert | Priorität |
|---------|------------|----------|-----------|
| NFA-V01 | Verfügbarkeit (SLA) | ≥ 99,5 % Verfügbarkeit während der Geschäftszeiten (Mo–Fr, 07:00–20:00 Uhr). ≥ 98,0 % Verfügbarkeit außerhalb der Geschäftszeiten. Gemessen pro Kalendermonat. Maximale ungeplante Ausfallzeit: ≤ 4 Stunden/Monat (innerhalb Geschäftszeiten). | 🔴 Muss |
| NFA-V02 | Geplante Wartungsfenster | Wartungsfenster: Samstag 22:00 – Sonntag 06:00 Uhr. Ankündigung mindestens 5 Werktage im Voraus. Maximal 2 geplante Wartungsfenster pro Monat. Zero-Downtime-Deployments für Minor-Releases angestrebt (Rolling Updates). | 🟡 Soll |
| NFA-V03 | Backup & Recovery | Vollbackup der PostgreSQL-Datenbank: täglich (00:00 Uhr). Inkrementelle Backups: alle 4 Stunden. Point-in-Time Recovery (PITR) über WAL-Archivierung. Recovery Time Objective (RTO): ≤ 4 Stunden. Recovery Point Objective (RPO): ≤ 1 Stunde (max. Datenverlust). Backup-Aufbewahrung: 30 Tage (täglich), 12 Monate (monatlich). Regelmäßige Restore-Tests: mindestens quartalsweise. | 🔴 Muss |
| NFA-V04 | Disaster Recovery | Dokumentierter Disaster-Recovery-Plan gemäß DORA Art. 5–16 (IKT-Risikomanagement). Jährlicher DR-Test. Georedundante Backup-Speicherung. Wiederanlaufplan mit definierten Verantwortlichkeiten und Eskalationsstufen. | 🔴 Muss |
| NFA-V05 | Incident Management | Klassifizierung von Störungen gemäß DORA Art. 17–23. Kritische Störungen (Totalausfall): Reaktionszeit ≤ 30 Minuten. Schwere Störungen (Teilausfall): Reaktionszeit ≤ 2 Stunden. Meldepflicht an BaFin bei schwerwiegenden IKT-Vorfällen. | 🔴 Muss |

## Skalierbarkeit
| NFA-Nr. | Anforderung | Details | Priorität |
|---------|------------|---------|-----------|
| NFA-SK01 | Datenvolumen | Das System muss initial ≥ 500.000 Verträge verwalten können und auf ≥ 5.000.000 Verträge skalierbar sein. PostgreSQL Table Partitioning nach Sparte/Jahr für große Bestände (vgl. Technische Rahmenbedingungen). JSONB-Spalten für spartenspezifische Attribute dürfen die Query-Performance nicht wesentlich beeinträchtigen (GIN-Indizes). Historisierungsdaten (Envers + Temporal Tables) müssen effizient archivierbar sein. | 🔴 Muss |
| NFA-SK02 | Horizontale Skalierung | Der modulare Monolith (Spring Modulith) muss als stateless Application-Server konzipiert sein, sodass mehrere Instanzen hinter einem Load Balancer betrieben werden können. Session-Daten werden nicht im Anwendungsserver gehalten (JWT-basiert). Datenbankverbindungen über Connection Pooling (HikariCP). Bei Bedarf späterer Split in Microservices möglich (vgl. Architekturentscheidung). | 🟡 Soll |
| NFA-SK03 | Datenarchivierung | Verträge, die seit ≥ 10 Jahren beendet sind und deren Aufbewahrungsfristen abgelaufen sind, können in ein Archiv überführt werden, um die Produktivdatenbank schlank zu halten. Archivierte Daten bleiben über einen separaten Zugriffspfad lesbar. | 🟢 Kann |

## Benutzbarkeit (Usability)
| NFA-Nr. | Anforderung | Details | Priorität |
|---------|------------|---------|-----------|
| NFA-U01 | Barrierefreiheit | Konformität mit WCAG 2.1 Level AA. Tastaturnavigation für alle Kernfunktionen. Ausreichende Kontrastverhältnisse (≥ 4,5:1). Angular Material bietet von Haus aus ARIA-Unterstützung, die konsequent genutzt werden muss. Screenreader-Kompatibilität für alle Formulare und Tabellen. | 🟡 Soll |
| NFA-U02 | Responsive Design | Optimiert für Desktop-Nutzung (Innendienst: ≥ 1280×720 px). Tablet-fähig für Außendienst (≥ 768 px Breite, z. B. iPad). Smartphone-Nutzung nicht im Scope (Endkunden nutzen das Kundenportal). Angular Material Responsive Grid/Layout wird verwendet. | 🔴 Muss |
| NFA-U03 | Browser-Unterstützung | Aktuelle Versionen (letzte 2 Major-Releases) von: Google Chrome, Microsoft Edge (Chromium), Mozilla Firefox. Safari-Unterstützung optional (🟢 Kann). Internet Explorer wird **nicht** unterstützt. Automatisierte Browser-Tests über die CI/CD-Pipeline (z. B. Playwright/Cypress). | 🔴 Muss |
| NFA-U04 | Sprache(n) | Primärsprache: Deutsch (de_DE). Alle UI-Texte, Fehlermeldungen und Hilfetexte in deutscher Sprache. Internationalisierung (i18n) via Angular i18n-Framework vorbereitet, sodass eine spätere Erweiterung um weitere Sprachen möglich ist. API-Fehlermeldungen: Deutsch (mit maschinenlesbarem Error-Code). | 🔴 Muss |
| NFA-U05 | Einarbeitungszeit | Neue Sachbearbeiter sollen die Kernfunktionen (Vertrag suchen, anzeigen, Nachtrag erstellen) nach ≤ 2 Tagen Einarbeitung eigenständig bedienen können. Kontextsensitive Hilfe und Tooltips in der Anwendung. | 🟡 Soll |
| NFA-U06 | Konsistenz | Spartenübergreifend einheitliche Navigation, Layouts und Interaktionsmuster (vgl. Projektvision). Einheitliches Design-System basierend auf Angular Material. Gleiche Aktionen führen in allen Sparten zu gleichem Verhalten (Principle of Least Surprise). | 🔴 Muss |

## Wartbarkeit
| NFA-Nr. | Anforderung | Details | Priorität |
|---------|------------|---------|-----------|
| NFA-W01 | Code-Qualität | Google Java Style Guide + Checkstyle (Backend), Prettier + ESLint (Frontend). Statische Code-Analyse via SonarQube: keine Critical/Blocker Issues im Release-Branch. Maximale zyklomatische Komplexität pro Methode: ≤ 15. Code Reviews via Pull Requests mit mindestens 1 Reviewer. Architectural Fitness Functions über ArchUnit (Einhaltung der Modulgrenzen Spring Modulith). | 🔴 Muss |
| NFA-W02 | Dokumentation | JavaDoc für alle öffentlichen APIs und Services. OpenAPI 3.1 Spezifikation für alle REST-Endpunkte (automatisch generiert via springdoc-openapi). Architecture Decision Records (ADRs) für alle wesentlichen Architekturentscheidungen. Fachliche Dokumentation der Spartenkonfiguration (Produkte, Regeln, Hooks) unter `12_sparten/<spartenname>/`. README pro Modul im Quellcode. | 🔴 Muss |
| NFA-W03 | Testabdeckung | Mindest-Testabdeckung (Line Coverage): ≥ 80 % (Kern und Sparten jeweils). Branch Coverage: ≥ 70 %. Drools-Regeln: 100 % der definierten Regeln durch Unit-Tests abgedeckt. Consumer-Driven Contracts: Alle Kern ↔ Sparte Schnittstellen durch Contract-Tests gesichert. Sparten-Testkit: Alle Sparten liefern das generische Testkit (Happy Paths). | 🔴 Muss |
| NFA-W04 | Modularität | Saubere Modulgrenzen gemäß Spring Modulith. Keine zyklischen Abhängigkeiten zwischen Modulen. Sparten-Module dürfen nur über definierte Schnittstellen (Registry, Hooks, Events) mit dem Kern kommunizieren. Neue Sparten müssen ohne Änderung am Kern integrierbar sein. | 🔴 Muss |
| NFA-W05 | Deployment-Frequenz | Ziel: ≥ 1 Release pro Monat (nach initialer Stabilisierung). Hotfixes innerhalb von 24 Stunden deploybar. CI/CD-Pipeline von Commit bis Staging ≤ 30 Minuten (inkl. aller Tests). | 🟡 Soll |

## Compliance & Regulatorik
| NFA-Nr. | Anforderung | Details | Priorität |
|---------|------------|---------|-----------|
| NFA-C01 | Gesetzliche Vorgaben | **DORA** (EU-Verordnung 2022/2554): Vollständige Konformität erforderlich – IKT-Risikomanagement (Art. 5–16), Meldung IKT-bezogener Vorfälle (Art. 17–23), Resilienz-Tests (Art. 24–27), Drittparteirisiko (Art. 28–44), Informationsaustausch (Art. 45). **DSGVO** (EU-Verordnung 2016/679): Datenschutz-Grundverordnung, insb. Art. 15 (Auskunftsrecht), Art. 17 (Recht auf Löschung), Art. 25 (Privacy by Design), Art. 30 (Verzeichnis von Verarbeitungstätigkeiten), Art. 33/34 (Meldepflicht bei Datenpannen). **VVG** (Versicherungsvertragsgesetz): Beachtung der Vorgaben zu Informationspflichten, Widerrufsrecht und Vertragsabwicklung. **IDD** (Insurance Distribution Directive, EU 2016/97): Beratungsdokumentation und Transparenzanforderungen bei Vertrieb. | 🔴 Muss |
| NFA-C02 | Branchenstandards | **GDV-Datensätze:** Unterstützung der GDV-Branchendatenformate für den Datenaustausch (soweit für die Schnittstellen S1–S8 relevant). **BiPRO-Normen:** Prüfung der Anwendbarkeit von BiPRO-Normen für standardisierte Geschäftsprozesse und Datenmodelle (insb. für Makler-Schnittstelle). **ISO 27001:** Informationssicherheitsmanagement als Orientierungsrahmen. **OWASP Top 10:** Berücksichtigung der aktuellen OWASP Top 10 als Mindeststandard für Anwendungssicherheit. | 🟡 Soll |
| NFA-C03 | Aufbewahrungsfristen | Vertragsdaten: 10 Jahre nach Vertragsbeendigung (§ 257 HGB, § 147 AO). Buchhaltungsrelevante Daten (Beiträge, Zahlungen): 10 Jahre. Audit-Logs: 10 Jahre. Kundenkommunikation (Beratungsprotokolle): 5 Jahre nach letztem Kontakt. Personenbezogene Daten: Löschung/Anonymisierung nach Ablauf der jeweiligen Aufbewahrungsfrist (DSGVO Art. 17). Revisionssichere Historisierung (Hibernate Envers + Temporal Tables) stellt die lückenlose Nachvollziehbarkeit sicher. | 🔴 Muss |
| NFA-C04 | Revisionssicherheit | Alle Vertragsänderungen werden revisionssicher mit Zeitstempel (UTC), Benutzer-ID und Änderungsgrund protokolliert. Historisierung über Hibernate Envers (Anwendungsebene) + Temporal Tables (Datenbankebene). Daten dürfen nachträglich nicht verändert oder gelöscht werden (Append-Only für Audit-relevante Tabellen). | 🔴 Muss |
| NFA-C05 | BaFin-Anforderungen | Einhaltung der **MaGo** (Mindestanforderungen an die Geschäftsorganisation) und **VAIT** (Versicherungsaufsichtliche Anforderungen an die IT). Dokumentation des IT-Risikomanagements. Ordnungsgemäße IT-Governance und Informationssicherheitsmanagement. | 🔴 Muss |

## Betrieb & Monitoring
| NFA-Nr. | Anforderung | Details | Priorität |
|---------|------------|---------|-----------|
| NFA-B01 | Health Checks | Spring Boot Actuator Health Endpoint für Liveness- und Readiness-Probes. Health Checks für: Datenbankverbindung, Kompetenz-System (S8), Regelengine (Drools), Festplattenplatz. Endpunkte: `/actuator/health/liveness`, `/actuator/health/readiness`. | 🔴 Muss |
| NFA-B02 | Metriken | Micrometer + Prometheus-kompatible Metriken: Request-Rate, Latenz (Histogramm), Error-Rate, JVM-Metriken (Heap, GC, Threads), Datenbankverbindungspool (HikariCP), Drools Rule-Execution-Time. Dashboards in Grafana (oder vergleichbar) für Echtzeit-Monitoring. | 🟡 Soll |
| NFA-B03 | Logging | Strukturiertes JSON-Logging (SLF4J + Logback) in Produktion. Correlation-ID pro Request (über alle Module und Schnittstellen hinweg). Log-Level: ERROR, WARN für Alerting; INFO für Geschäftsprozesse; DEBUG nur in Nicht-Produktion. Zentrale Log-Aggregation (ELK-Stack, Loki oder vergleichbar). | 🔴 Muss |
| NFA-B04 | Alerting | Automatische Alerts bei: Verfügbarkeit < SLA-Zielwert, API-Antwortzeit > 5 Sekunden (p95), Error-Rate > 1 %, Datenbankverbindungen erschöpft, Festplattenplatz < 20 %. Benachrichtigung über E-Mail und/oder Messaging-Tool (noch zu definieren). | 🟡 Soll |
| NFA-B05 | Tracing | Distributed Tracing via Micrometer Tracing über alle Module hinweg. Trace-ID wird an externe Schnittstellen (S1–S8) propagiert (soweit unterstützt). Ermöglicht End-to-End-Nachverfolgung von Geschäftsprozessen (z. B. „Angebot erstellen" über Kern → Hook → Regelengine → DB). | 🟡 Soll |

> **Priorität:** 🔴 Muss | 🟡 Soll | 🟢 Kann
