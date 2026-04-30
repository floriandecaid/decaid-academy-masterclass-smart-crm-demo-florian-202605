# Quick Start

Du hast eben die Masterclass gesehen. Jetzt willst du die Demo nachbauen. Hier sind die nächsten 30 Minuten.

---

## Was du bekommst

| Datei | Zweck |
|---|---|
| [`index.html`](../index.html) | Webinar-Landingpage im DECAID-Look |
| [`slides/`](../slides/index.html) | Reveal.js-Slides der Masterclass |
| [`colab/webinar_simulator.ipynb`](../colab/webinar_simulator.ipynb) | Python-Notebook, das Mock-Daten an deinen Langdock-Workflow feuert |
| [`materials/workflow1-lead-triage.html`](workflow1-lead-triage.html) | Lead-Triage + Deep-Research Workflow |
| [`materials/workflow2-battle-cards.html`](workflow2-battle-cards.html) | Battle-Card-Generator |
| [`materials/architecture.html`](architecture.html) | System-Übersicht |
| [`transcripts/fake_sales_calls/`](../transcripts/fake_sales_calls/) | 8 fiktive Sales-Call-Transkripte zum Testen |

---

## 5 Schritte zum eigenen Setup

### 1. Repo klonen oder forken

```bash
git clone https://github.com/<your-org>/decaid-academy-masterclass-smart-crm-demo-202605.git
cd decaid-academy-masterclass-smart-crm-demo-202605
```

### 2. Langdock Workflow 1 anlegen

1. Login bei [Langdock](https://app.langdock.com) → **New Workflow**
2. Nodes nach [`workflow1-lead-triage.md`](workflow1-lead-triage.md) konfigurieren
3. **Wichtig:** Web-Search-Tool im Node 4 aktivieren
4. Im Webhook-Node auf **"Copy URL"** klicken — die brauchst du gleich

### 3. Colab-Notebook konfigurieren

1. [`webinar_simulator.ipynb`](../colab/webinar_simulator.ipynb) in [Google Colab](https://colab.research.google.com) öffnen
2. Erste Zelle: `LANGDOCK_WEBHOOK_URL` einsetzen
3. Optional: `HUBSPOT_PAT` setzen (`pat-...`), wenn du in HubSpot syncen willst — für die Demo aber nicht nötig

### 4. Test-Run

Notebook von oben nach unten ausführen:

- Zelle 4 (Jürgen) → Du erwartest **keine** E-Mail (Score zu niedrig).
- Zelle 5 (Gabi) → Du erwartest in 10–40 Sek eine E-Mail mit Sales-Briefing.

### 5. Workflow 2 (Battle Cards)

Optional, aber lohnt sich:

1. Zweiten Langdock-Workflow nach [`workflow2-battle-cards.md`](workflow2-battle-cards.md) anlegen
2. Die 8 Transkripte aus `transcripts/fake_sales_calls/` als Payload reinwerfen
3. Battle Card landet im Postfach

---

## Häufige Stolperfallen

### "Webhook gibt 200 zurück, aber kein Mail"

Score < 50 → Lead landet im Nurture-Pfad. Bei Jürgen ist das gewollt. Bei Gabi check die Triage-Logik.

### "Web Search liefert leere Ergebnisse"

Web-Search-Tool im Node nicht aktiviert oder Firmenname zu generisch. Vollständigen offiziellen Namen einsetzen ("Gerolsteiner Brunnen GmbH & Co. KG", nicht "Gerolsteiner").

### "Mail kommt im Spam an"

Für die Demo egal. In Production: SPF/DKIM für Absenderdomain einrichten.

### "Colab kann meinen Webhook nicht erreichen"

Manche Corporate-Netzwerke blocken Outbound zu unbekannten Hosts. Probier's privat oder nimm die JS-Variante über [`index.html`](../index.html).

---

## Brauchst du Hilfe?

Schreib an **florian@decaid.studio** oder buch dir ein 15-Min-Slot über die DECAID-Website.
