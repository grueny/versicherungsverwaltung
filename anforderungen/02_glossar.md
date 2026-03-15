# Glossar – Versicherungsverwaltung

> Einheitliche Definition aller fachlichen Begriffe. Diese Begriffe werden konsistent in Code, Datenbank und UI verwendet.

| Begriff | Definition | Synonyme | Beispiel |
|---------|-----------|----------|---------|
| Versicherungsnehmer | | | |
| Vertrag | | | |
| Police | | | |
| Prämie | | | |
| Schaden | | | |
| Schadenfall | | | |
| Deckung | | | |
| Sparte | Eine Konfiguration bestehend aus Produkten, spartenspezifischen Prozessen und Plausibilitäten. Definiert, wie der generische Kernprozess für einen Versicherungszweig konkret ausgeprägt wird. | Versicherungszweig, Versicherungssparte | Sach, Leben, Kranken, KFZ |
| Produkt | Ein versicherbares Angebot innerhalb einer Sparte, definiert durch Merkmale, Tarife und Deckungsbausteine. | Versicherungsprodukt | Hausratversicherung Komfort, KFZ-Haftpflicht Basis |
| Plausibilität | Eine spartenspezifische Validierung oder Geschäftsregel, die bei der Erfassung oder Änderung von Vertragsdaten geprüft wird. | Prüfregel, Validierung | Mindestalter für KFZ: 18 Jahre |
| Spartenkonfiguration | Die Gesamtheit aller Produkte, Prozessabweichungen und Plausibilitäten, die eine Sparte definieren. | Spartendefinition | Konfiguration der Sparte „Sach" |
| Kompetenz | Eine einzelne, abfragbare Berechtigung, die einem Benutzer zugeordnet ist. Wird über eine eindeutige Kompetenz-ID identifiziert und vom externen Kompetenz-System verwaltet. | Berechtigung, Permission | Kompetenz „Vertrag_anlegen", „Nachtrag_freigeben" |
| Kompetenzmodell | Das Berechtigungskonzept des Systems. Berechtigungen werden nicht im Bestandsführungssystem verwaltet, sondern über eine Schnittstelle zum externen Kompetenz-System abgefragt (Kompetenz-ID → ja/nein). Rollen und Zuordnungen werden ausschließlich im Kompetenz-System gepflegt. | Berechtigungsmodell | System fragt: „Hat Benutzer X die Kompetenz KFZ_Vertrag_policieren?" |
| DORA | Digital Operational Resilience Act (EU-Verordnung 2022/2554). Regulatorischer Rahmen für die digitale operationale Resilienz von Finanzunternehmen (inkl. Versicherungen). Stellt Anforderungen an IKT-Risikomanagement, Vorfallmeldung, Resilienz-Tests, Drittparteirisiko und Informationsaustausch. | Digital Operational Resilience Act | DORA-konforme Protokollierung von IKT-Vorfällen |
| Tarif | | | |
| Leistung | | | |
| Selbstbeteiligung | | | |
| Laufzeit | | | |
| Kündigung | | | |
| Makler | | | |

<!-- Weitere Begriffe hier ergänzen -->
