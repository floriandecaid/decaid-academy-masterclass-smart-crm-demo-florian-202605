# Smarter CRM & KI-Automatisierung — Masterclass-Materialien

Du hast die Masterclass besucht. Hier ist alles, was du brauchst, um den gezeigten Workflow in deinem eigenen Setup nachzubauen.

> **In 30 Minuten** hast du eine Pipeline am Laufen, die jeden Webinar-Lead live im Web recherchiert, Hot Leads als Sales-Briefing an dein Sales-Team schickt und No-Shows mit personalisierten Re-Engagement-Hooks ans Marketing weitergibt.

**👉 Direkt loslegen:** [Quick-Start-Anleitung öffnen](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/materials/quick-start.html)

---

## Was du hier findest

| Material | Was du damit machst |
|---|---|
| [Landingpage](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/) | Webinar-Landingpage mit zwei Demo-Buttons, mit denen du beide Personas an deinen Workflow feuerst. |
| [Slides der Masterclass](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/slides/) | Foliendeck — wenn du das Konzept intern weitergeben willst. |
| [Quick-Start-Anleitung](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/materials/quick-start.html) | Schritt-für-Schritt-Anleitung. Lies das zuerst. |
| [Workflow 1 — Web Research & Smart Routing](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/materials/workflow1-lead-triage.html) | Alle Nodes, alle Prompts, alle E-Mail-Templates copy-paste-ready. |
| [Workflow 2 — Battle Cards](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/materials/workflow2-battle-cards.html) | Bonus-Workflow: aus Sales-Call-Transkripten automatisch wöchentliche Battle Cards generieren. |
| [Architektur-Übersicht](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/materials/architecture.html) | Big-Picture: wie alle Komponenten zusammenspielen, Tech-Stack, Skalierungs-Pfad. |
| [`colab/webinar_simulator.ipynb`](colab/webinar_simulator.ipynb) | Python-Notebook für Google Colab. Zwei Mock-Personas, ein Webhook-POST. Damit testest du den Workflow ohne echte Webinar-Plattform. |
| [`transcripts/fake_sales_calls/`](transcripts/fake_sales_calls/) | 8 fiktive Sales-Call-Transkripte (4 won, 3 lost, 1 open) als Testdaten für Workflow 2. |

---

## Reihenfolge

1. **[Quick Start lesen](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/materials/quick-start.html)** — der schnellste Weg zur ersten funktionierenden Pipeline.
2. **Langdock-Account holen** — falls du noch keinen hast, brauchst du einen Plan mit aktiviertem Web-Search-Tool (typisch: Pro oder höher).
3. **[Workflow 1 nachbauen](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/materials/workflow1-lead-triage.html)** — Setup-Zeit: 15 Minuten. Output: zwei E-Mail-Pfade, die personalisiert sind.
4. **Mit dem Colab-Notebook testen** — zwei Personas, zwei Mails, beide kommen automatisch.
5. **[Workflow 2 dazuholen](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/materials/workflow2-battle-cards.html)**, wenn du Sales-Call-Transkripte hast — optional, aber wenn du Gong, Modjo oder ähnliches im Einsatz hast, lohnen sich weitere 10 Minuten Setup.

---

## Was du brauchst

| Komponente | Wofür |
|---|---|
| **Langdock-Account** mit Web-Search-Tool | Workflow-Engine, recherchiert Firmen live im Netz |
| **Mail-Account (Gmail oder Outlook)** | Versand der Sales-Briefing- und Re-Engagement-Mails |
| **Google-Account für Colab** | Mock-Personas an den Webhook feuern (alternativ lokal mit Python) |
| **(Optional) GitHub-Account** | Falls du die Landingpage bei dir hosten willst |

---

## Architektur in einer Zeile

```
[Webhook]  →  [Web Research Agent]  →  [Watch-Time ≥ 30 Min?]  →  Sales-Briefing  |  Re-Engagement
```

Die KI recherchiert **jeden** Lead — Revenue, News, strategische Themen. Erst danach entscheidet die Watch-Time, ob das Sales-Team einen Call vorbereiten soll oder Marketing eine personalisierte Wiederansprache plant. Kein Lead fällt durchs Raster.

Volle Erklärung in der [Architektur-Übersicht](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/materials/architecture.html).

---

## Lizenz

Frei nutzbar. Du darfst die Workflows, Prompts und Templates in deinen eigenen Projekten einsetzen, anpassen und weiterverteilen — ohne Rückfrage.
