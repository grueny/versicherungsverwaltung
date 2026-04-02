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
| Angebot | Ein unverbindlicher Versicherungsvorschlag mit Produkten, Deckungsbausteinen und berechnetem Beitrag. Kann geprüft und in einen Antrag überführt werden (UC-01). | Offerte | Angebot #AG-2026-001234 |
| Antrag | Ein verbindlicher Versicherungsantrag, der aus einem Angebot erzeugt oder direkt erfasst wird. Kann bearbeitet, berechnet, geprüft und freigegeben werden (UC-02). Bei Freigabe wird eine Schwebe angelegt und ggf. ein Vertragsstand erzeugt. | Versicherungsantrag | Antrag #AN-2026-005678 |
| Schwebe | Ein offener Vorgang im System, der bei der Freigabe eines Antrags erzeugt wird. Repräsentiert den Zustand zwischen Antragsfreigabe und Vertragsstand-Erzeugung. Bei Aussteuerung bleibt die Schwebe offen, bis der Innendienst entscheidet. | Offener Posten, Wiedervorlage | Schwebe zur Innendienst-Prüfung eines KFZ-Antrags |
| Aussteuerung | Die automatische oder regelbasierte Weiterleitung eines Antrags an den Innendienst zur manuellen Prüfung. Aussteuerungsregeln sind konfigurierbar (pro Sparte und spartenübergreifend). | Innendienst-Aussteuerung, Dunkelverarbeitungs-Abbruch | Aussteuerung wegen Deckungssumme > 10 Mio. EUR |
| Vorgang | Ein fachlicher Geschäftsvorfall, dem ein oder mehrere Vertragsstände zugeordnet sind. Klassifiziert die Art der Vertragsveränderung. | Geschäftsvorfall | Neugeschäft, Änderung, Stornierung |
| Vertragsstand | Ein konkreter, zu einem bestimmten Zeitpunkt gültiger Zustand eines Vertrags. Wird bei der Freigabe eines Antrags erzeugt und einem Vorgang zugeordnet. Jeder Vertragsstand ist revisionssicher historisiert. | Vertragsversion, Vertragsbild | Vertragsstand nach Fahrzeugwechsel (Vorgang: Änderung) |
| Tarif | | | |
| Versicherungsgewerbetarif (VGT) | Ein spartenübergreifender Sondernachlass für Mitarbeitende der Versicherungswirtschaft (eigenes Unternehmen und andere Versicherungsunternehmen). Wird als prozentualer Nachlass auf Vertragsebene (Nachlassart `VGT_NACHLASS`) abgebildet und automatisch bei der Angebotserstellung vergeben, sofern der Versicherungsnehmer als VGT-berechtigt hinterlegt ist (UC-11). | Gewerbetarif, Mitarbeiterrabatt, Branchentarif | VGT-Nachlass 15 % auf Gesamtbeitrag |
| Nachlass | Ein Rabatt oder Abschlag auf den Versicherungsbeitrag, der auf Vertragsebene (Gesamtbeitrag) oder Risikoebene (Einzelprodukt) gewährt werden kann. Nachlässe können prozentual oder als Absolutbetrag angegeben werden und werden bei der Beitragskalkulation berücksichtigt (UC-10). | Rabatt, Abschlag, Discount | Bündelrabatt 5 % auf Gesamtbeitrag |
| Zuschlag | Ein Aufschlag auf den Versicherungsbeitrag, der auf Vertragsebene oder Risikoebene erhoben werden kann. Zuschläge können prozentual oder als Absolutbetrag angegeben werden und werden bei der Beitragskalkulation berücksichtigt (UC-10). | Aufschlag, Surcharge, Risikozuschlag | Gefahrenerhöhung 15 % auf KFZ-Haftpflicht |
| Leistung | | | |
| Selbstbeteiligung | | | |
| Laufzeit | | | |
| Kündigung | | | |
| Makler | | | |

<!-- Weitere Begriffe hier ergänzen -->
