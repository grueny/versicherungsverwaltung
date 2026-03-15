# Kontextabgrenzung – Versicherungsverwaltung

## Systemkontext

```
                    ┌───────────────┐   ┌──────────────────┐
                    │  Provisions-  │   │  Konzerninkasso  │
                    │    system     │   │   / Exkasso      │
                    └──────┬────────┘   └────────┬─────────┘
                           │                     │
┌──────────────┐    ┌──────▼─────────────────────▼──────┐    ┌──────────────┐
│   Schaden-   │◄──►│                                   │◄──►│  Datawarehouse│
│  verwaltung  │    │      VERSICHERUNGSVERWALTUNG       │    │              │
└──────────────┘    │                                   │    └──────────────┘
                    │   Anbahnung │ Bestand │ Wartung   │
┌──────────────┐    │   Historisierung │ Schnittstellen  │    ┌──────────────┐
│   Kunden-    │◄──►│                                   │──►│  Kompetenz-  │
│   portal     │    │                                   │◄──│    System    │
└──────────────┘    └──────▲────────────────────▲───────┘    └──────────────┘
                           │                    │
                    ┌──────┴────────┐    ┌──────┴───────┐    ┌──────────────┐
                    │  Innendienst  │    │   Vertrieb / │    │   Druck-     │
                    │  Sachbearbeit.│    │    Makler    │    │ schnittstelle│
                    └───────────────┘    └──────────────┘    └──────────────┘
```

## Externe Schnittstellen

| Nr. | Externes System | Richtung | Beschreibung | Protokoll/Format |
|-----|----------------|----------|-------------|-----------------|
| S1 | Provisionssystem | Ausgehend | Vertragsdaten für Provisionsberechnung liefern (Neugeschäft, Änderungen, Storno) | <!-- Noch zu klären --> |
| S2 | Konzerninkasso | Ausgehend | Beitragsforderungen und Zahlungspläne übergeben | <!-- Noch zu klären --> |
| S3 | Exkasso | Eingehend / Ausgehend | Auszahlungsdaten (z. B. Rückkaufswerte, Überschüsse) | <!-- Noch zu klären --> |
| S4 | Datawarehouse | Ausgehend | Bestandsdaten, Vertragsbewegungen und Kennzahlen liefern | <!-- Noch zu klären --> |
| S5 | Druckschnittstelle | Ausgehend | Druckaufträge für Policen, Nachträge, Kündigungsbestätigungen etc. | <!-- Noch zu klären --> |
| S6 | Schadenverwaltung | Bidirektional | Vertragsdaten bereitstellen; Schadeninfos empfangen (z. B. Schadenstatus) | <!-- Noch zu klären --> |
| S7 | Kundenportal | Bidirektional | Vertragsdaten zur Anzeige bereitstellen; Änderungswünsche empfangen | <!-- Noch zu klären --> |
| S8 | Kompetenz-System | Eingehend (Abfrage) | Prüfung, ob ein Benutzer eine bestimmte Kompetenz (Kompetenz-ID) besitzt; Rollenverwaltung erfolgt im Kompetenz-System | <!-- Noch zu klären --> |

## Systemgrenzen

### Innerhalb des Systems
- Vertriebsanbahnung (Angebote, Anträge)
- Vertragsabschluss und Policierung
- Bestandsführung und Vertragsverwaltung
- Vertragswartung (Nachträge, Änderungen, Anpassungen)
- Vertragsbeendigung (Kündigung, Storno, Ablauf)
- Revisionssichere Historisierung aller Vertragsänderungen
- Stammdatenverwaltung (vertragsbezogen)

### Außerhalb des Systems
- Provisionsberechnung und -abrechnung → **Provisionssystem**
- Beitragseinzug und Mahnwesen → **Konzerninkasso**
- Auszahlungen → **Exkasso**
- Reporting und Analysen → **Datawarehouse**
- Dokumentenerstellung und Versand → **Druckschnittstelle**
- Schadenaufnahme und -regulierung → **Schadenverwaltung**
- Self-Service für Endkunden → **Kundenportal**
- Rollen- und Berechtigungsverwaltung → **Kompetenz-System**

## Abhängigkeiten
- **Druckschnittstelle:** Policen und Nachträge können erst als finalisiert gelten, wenn der Druck bestätigt ist
- **Konzerninkasso:** Beitragsrelevante Vertragsänderungen müssen zeitnah übermittelt werden
- **Provisionssystem:** Neugeschäft und Änderungen müssen für korrekte Provisionsberechnung geliefert werden
- **Schadenverwaltung:** Aktiver Vertragsstatus und Deckungsinformationen müssen abrufbar sein
- **Kompetenz-System:** Jede berechtigungsrelevante Aktion erfordert eine Kompetenzabfrage; bei Nichtverfügbarkeit werden geschützte Funktionen blockiert
