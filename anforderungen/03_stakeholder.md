# Stakeholder & Rollen – Versicherungsverwaltung

## Übersicht der Rollen

### Rolle 1: Außendienst (Vertrieb)
- **Beschreibung:** Vertriebsmitarbeiter im Außendienst, die Kunden vor Ort beraten und Verträge abschließen. Primäre Nutzergruppe für die Vertriebsanbahnung.
- **Hauptaufgaben:**
  - Angebote erstellen und Kunden präsentieren
  - Anträge erfassen und einreichen
  - Vertragsabschlüsse durchführen
  - Bestandsauskünfte für Kunden einholen
  - Fahrzeugdaten / spartenspezifische Daten erfassen
- **Berechtigungen:** Vertrag anlegen, Angebote erstellen, Vertragsauskünfte einsehen, eingeschränkte Vertragsänderungen (über Kompetenzmodell gesteuert)
- **Häufigkeit der Nutzung:** Täglich, mobil und vor Ort beim Kunden

### Rolle 2: Innendienst (Sachbearbeitung)
- **Beschreibung:** Sachbearbeiter im Innendienst, die Verträge verwalten, Sonderfälle bearbeiten und die Qualitätssicherung der Vertragsdaten übernehmen.
- **Hauptaufgaben:**
  - Vertragsverwaltung und Bestandsführung
  - Vertragsänderungen (Nachträge) bearbeiten
  - Sonderfälle und Ausnahmen behandeln (z. B. manuelle SF-Übernahme, Kulanzregelungen)
  - Kündigungen und Stornierungen verarbeiten
  - Datenbereinigung und Qualitätssicherung
  - Schnittstellen-Ergebnisse prüfen (Druck, Inkasso, Provision)
- **Berechtigungen:** Vollzugriff auf Vertragsverwaltung, erweiterte Berechtigungen für Sonderfälle (über Kompetenzmodell gesteuert)
- **Häufigkeit der Nutzung:** Täglich, am Büroarbeitsplatz

### Rolle 3: Kunde (via Kundenportal)
- **Beschreibung:** Versicherungsnehmer, die über das Kundenportal auf ihre Vertragsdaten zugreifen. Kein direkter Systemzugang – der Zugriff erfolgt ausschließlich über die Kundenportal-Schnittstelle (S7).
- **Hauptaufgaben:**
  - Eigene Verträge einsehen
  - Neue Verträge abschließen (Self-Service)
  - Vertragsänderungen beauftragen (z. B. Adressänderung, Fahrzeugwechsel, Bausteine hinzufügen/entfernen)
  - Dokumente und Policen herunterladen
- **Berechtigungen:** Nur eigene Verträge, nur über Kundenportal-Schnittstelle; keine direkte Systemanmeldung
- **Häufigkeit der Nutzung:** Gelegentlich bis regelmäßig

## Rollenmatrix (Berechtigungen)

> Berechtigungen werden über das externe **Kompetenz-System** gesteuert (Kompetenz-ID → ja/nein).  
> Die folgende Matrix zeigt die **fachliche Soll-Zuordnung**; die tatsächlichen Berechtigungen werden im Kompetenz-System konfiguriert.

| Funktion | Außendienst | Innendienst | Kunde (Portal) |
|----------|-------------|-------------|----------------|
| Angebot erstellen | ✅ | ✅ | ✅ |
| Antrag erfassen | ✅ | ✅ | ✅ |
| Vertrag anlegen / abschließen | ✅ | ✅ | ✅ |
| Vertrag einsehen | ✅ (eigene Kunden) | ✅ | 👁️ (eigene Verträge) |
| Vertrag ändern (Nachtrag) | ⚠️ (eingeschränkt) | ✅ | ⚠️ (eingeschränkt) |
| Sonderfälle bearbeiten | ❌ | ✅ | ❌ |
| Vertrag kündigen / stornieren | ❌ | ✅ | ⚠️ (Kündigungswunsch) |
| Stammdaten ändern | ⚠️ (eingeschränkt) | ✅ | ⚠️ (eigene Adresse) |
| Dokumente / Policen einsehen | ✅ (eigene Kunden) | ✅ | 👁️ (eigene Dokumente) |
| Import (Datenmigration) | ❌ | ✅ | ❌ |

> Legende: ✅ = Vollzugriff | 👁️ = Nur Lesen | ⚠️ = Eingeschränkt | ❌ = Kein Zugriff

## Externe Stakeholder
<!-- Systeme, Partner, Behörden etc. die mit dem System interagieren -->
- **Kundenportal** – Leitet Kundenaktionen (Einsicht, Abschluss, Änderungen) über Schnittstelle S7 an das System weiter
- **Zulassungsstellen** – Empfangen eVB-Nummern (KFZ-spezifisch)
- **GDV** – Liefert Typklassen, Regionalklassen, SF-Klassen-Informationen (KFZ-spezifisch)
- **BaFin** – Aufsichtsbehörde, regulatorische Vorgaben 
