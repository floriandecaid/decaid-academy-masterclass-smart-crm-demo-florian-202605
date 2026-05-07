# Smarter CRM & KI-Automatisierung — Masterclass-Materialien

Du hast die Masterclass besucht. Hier ist alles, was du brauchst, um den gezeigten Workflow in deinem eigenen Setup nachzubauen.

> **In 30 Minuten** hast du eine Pipeline am Laufen, die jeden Webinar-Lead live im Web recherchiert und ein Briefing-Doc in deinem Drive ablegt — Hot Leads bekommen ein Sales-Briefing mit Eisbrecher (🔥-Präfix), No-Shows einen personalisierten Re-Engagement-Vorschlag fürs Marketing (❄️-Präfix).

**👉 Direkt loslegen:** [Quick-Start-Anleitung öffnen](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/materials/quick-start.html)

---

## Was du hier findest

| Material | Was du damit machst |
|---|---|
| [Landingpage](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/) | Webinar-Landingpage mit zwei Demo-Buttons, mit denen du beide Personas an deinen Workflow feuerst. |
| [Slides der Masterclass](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/slides/) | Foliendeck — wenn du das Konzept intern weitergeben willst. |
| [Quick-Start-Anleitung](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/materials/quick-start.html) | Schritt-für-Schritt-Anleitung. Lies das zuerst. |
| [Workflow 1 — Web Research & Smart Routing](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/materials/workflow1-lead-triage.html) | Alle Nodes, alle Prompts, alle Doc-Templates copy-paste-ready. |
| [Workflow 2 — Battle Cards](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/materials/workflow2-battle-cards.html) | Bonus-Workflow: aus Sales-Call-Transkripten automatisch wöchentliche Battle Cards generieren. |
| [Architektur-Übersicht](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/materials/architecture.html) | Big-Picture: wie alle Komponenten zusammenspielen, Tech-Stack, Skalierungs-Pfad. |
| [`transcripts/fake_sales_calls/`](transcripts/fake_sales_calls/) | 8 fiktive Sales-Call-Transkripte (4 won, 3 lost, 1 open) als Testdaten für Workflow 2. |
| [`colab/webinar_simulator.ipynb`](colab/webinar_simulator.ipynb) | **Optional** — Python-Notebook für Google Colab. Alternative zum Browser-Trigger, falls du die Pipeline aus einem Python-Kontext anstoßen willst. |

---

## Reihenfolge

1. **[Quick Start lesen](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/materials/quick-start.html)** — der schnellste Weg zur ersten funktionierenden Pipeline.
2. **Langdock-Account + Google-Drive-Integration** — falls du noch keinen Langdock-Account hast, brauchst du einen Plan mit aktiviertem Web-Search-Tool (typisch: Pro oder höher). Drive verbindest du in Langdock per OAuth.
3. **Drive-Ordner anlegen** — ein gemeinsamer für die Demo (z.B. `DECAID Lead Briefings`); Hot/Cold-Trennung kannst du später einbauen.
4. **[Workflow 1 nachbauen](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/materials/workflow1-lead-triage.html)** — Setup-Zeit: 10 Minuten. Vier Nodes: Webhook → Larry → Create Doc → Update Doc. Larry routet im Prompt selbst zwischen Hot- und Cold-Briefing.
5. **Über die Landingpage testen** — Demo-Buttons feuern beide Personas an den Workflow. Beide Docs landen in deinem Drive.
6. **[Workflow 2 dazuholen](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/materials/workflow2-battle-cards.html)**, wenn du Sales-Call-Transkripte hast — optional, aber wenn du Gong, Modjo oder ähnliches im Einsatz hast, lohnen sich weitere 10 Minuten Setup.

---

## Was du brauchst

| Komponente | Wofür |
|---|---|
| **Langdock-Account** mit Web-Search-Tool | Workflow-Engine, recherchiert Firmen live im Netz |
| **Google-Drive** (Workspace oder Free) | Output-Layer: hier landen die Briefing-Docs |
| **(Optional) GitHub-Account** | Falls du die Landingpage bei dir hosten willst |

---

## Architektur in einer Zeile

```
[Webhook]  →  [Larry the Lead Researcher]  →  [Create Doc]  →  [Update Doc]
                  Web-Search aktiviert,
                  routet selbst (Hot/Cold)
```

Larry ist ein einziger Agent mit Web-Search-Zugriff, der **jeden** Lead recherchiert (Revenue, News, strategische Themen) und auf Basis der Watch-Time selbst entscheidet, ob er ein Sales-Briefing oder ein Re-Engagement-Briefing schreibt. Das Ergebnis landet als Google Doc im geteilten Ordner. Kein Lead fällt durchs Raster.

Volle Erklärung in der [Architektur-Übersicht](https://floriandecaid.github.io/decaid-academy-masterclass-smart-crm-demo-florian-202605/materials/architecture.html).

---

## Lizenz

Frei nutzbar. Du darfst die Workflows, Prompts und Templates in deinen eigenen Projekten einsetzen, anpassen und weiterverteilen — ohne Rückfrage.
