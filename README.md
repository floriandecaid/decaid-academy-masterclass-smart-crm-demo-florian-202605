# Smarter CRM & KI-Automatisierung — Masterclass-Materialien

Du hast die Masterclass besucht. Hier ist alles, was du brauchst, um den gezeigten Workflow in deinem eigenen Setup nachzubauen.

> **In 30 Minuten** hast du eine Pipeline am Laufen, die jeden Webinar-Lead live im Web recherchiert, Hot Leads als Sales-Briefing an dein Sales-Team schickt und No-Shows mit personalisierten Re-Engagement-Hooks ans Marketing weitergibt.

---

## Was du hier findest

| Datei | Was du damit machst |
|---|---|
| [`index.html`](index.html) | Webinar-Landingpage. Direkt lokal öffnen oder bei dir hosten. Enthält zwei Demo-Buttons, mit denen du beide Personas an deinen Workflow feuerst. |
| [`slides/`](slides/index.html) | Reveal.js-Foliendeck der Masterclass — wenn du das Konzept intern weitergeben willst. |
| [`colab/webinar_simulator.ipynb`](colab/webinar_simulator.ipynb) | Python-Notebook für Google Colab. Zwei Mock-Personas, ein Webhook-POST. Damit testest du den Workflow ohne echte Webinar-Plattform. |
| [`materials/quick-start.html`](materials/quick-start.html) | Schritt-für-Schritt-Anleitung. Lies das zuerst. |
| [`materials/workflow1-lead-triage.html`](materials/workflow1-lead-triage.html) | Vollständige Spec für Workflow 1 (Web Research & Smart Routing): alle Nodes, alle Prompts, alle E-Mail-Templates copy-paste-ready. |
| [`materials/workflow2-battle-cards.html`](materials/workflow2-battle-cards.html) | Bonus-Workflow: aus Sales-Call-Transkripten automatisch wöchentliche Battle Cards generieren. |
| [`materials/architecture.html`](materials/architecture.html) | Big-Picture-Übersicht: wie alle Komponenten zusammenspielen, Tech-Stack, Skalierungs-Pfad. |
| [`transcripts/fake_sales_calls/`](transcripts/fake_sales_calls/) | 8 fiktive Sales-Call-Transkripte (4 won, 3 lost, 1 open) als Testdaten für Workflow 2. |

---

## Reihenfolge

1. **[Quick Start](materials/quick-start.html) lesen** — der schnellste Weg zur ersten funktionierenden Pipeline.
2. **Langdock-Account holen** — falls du noch keinen hast, brauchst du einen Plan mit aktiviertem Web-Search-Tool (typisch: Pro oder höher).
3. **Workflow 1 nachbauen** — folge der [Spec](materials/workflow1-lead-triage.html). Setup-Zeit: 15 Minuten. Output: zwei E-Mail-Pfade, die personalisiert sind.
4. **Mit dem Colab-Notebook testen** — zwei Personas, zwei Mails, beide kommen automatisch.
5. **Workflow 2 dazuholen, wenn du Sales-Call-Transkripte hast** — der Battle-Card-Workflow ist optional, aber wenn du Gong, Modjo oder ähnliches im Einsatz hast, lohnt sich der zweite Workflow nach 10 weiteren Minuten Setup.

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

Volle Erklärung in [`materials/architecture.html`](materials/architecture.html).

---

## Lizenz

Frei nutzbar. Du darfst die Workflows, Prompts und Templates in deinen eigenen Projekten einsetzen, anpassen und weiterverteilen — ohne Rückfrage.
