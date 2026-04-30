# DECAID Academy — Masterclass: Smarter CRM & KI-Automatisierung

> Live-Masterclass am **26. Mai 2026** · 60 Minuten · Florian Langer
> Plug-and-Play-Materialien, mit denen Marketing- & Sales-Profis den Sprung von ChatGPT-Spielereien zu skalierbaren Revenue-Workflows schaffen.

---

## Was ist das hier?

Das öffentliche Repo zur Masterclass. Du findest hier:

- 🎬 **Landingpage** ([`index.html`](index.html)) — DECAID-Look, Webinar-Registrierung, Live-Demo-Trigger.
- 🎞 **Slides** ([`slides/`](slides/index.html)) — Reveal.js, dieselbe Optik wie die Landingpage.
- 🐍 **Colab Notebook** ([`colab/webinar_simulator.ipynb`](colab/webinar_simulator.ipynb)) — Mock-Personas an deinen Langdock-Webhook feuern.
- 📋 **Workflow-Specs** ([`materials/`](materials/quick-start.html)) — Prompts, Mappings, E-Mail-Templates copy-paste-ready.
- 🗂 **Fake Sales-Transkripte** ([`transcripts/fake_sales_calls/`](transcripts/fake_sales_calls/)) — 8 fiktive Calls (4 won, 3 lost, 1 open) für Workflow 2.

---

## Quick Start

```bash
git clone https://github.com/<your-org>/decaid-academy-masterclass-smart-crm-demo-florian-202605.git
cd decaid-academy-masterclass-smart-crm-demo-florian-202605
open index.html   # macOS — oder im Browser deiner Wahl
```

Vollständige Anleitung: [`materials/quick-start.html`](materials/quick-start.html)

---

## Architektur (TL;DR)

```
[Landingpage / Colab]  →  [Langdock Workflow]  →  [Sales-Inbox]
     (POST JSON)              (Triage + Web Search)      (Briefing-Mail)
```

Vier Bausteine. Kein Backend. Kein Server. ~$30/Monat in Production.

→ Details: [`materials/architecture.html`](materials/architecture.html)

---

## Live-Demo — Was während der Masterclass passiert

1. **Min 0–5** — Live-Sign-Up auf der Landingpage (Jürgen + Gabi).
2. **Min 5–15** — Walkthrough Colab + Webhook-Setup.
3. **Min 15–30** — Deep Dive Workflow 1 (Triage + Live-Web-Search).
4. **Min 30–40** — Bonus-Workflow 2 (Battle Cards aus Sales-Calls).
5. **Min 40–60** — Q&A + Materialien-Übergabe.

---

## Stack

| Layer | Tool |
|---|---|
| Hosting | GitHub Pages |
| Frontend | Vanilla HTML + CSS + JS (DECAID-Tokens) |
| Slides | Reveal.js 5.x |
| Trigger | Google Colab + Python stdlib |
| Workflow-Engine | [Langdock](https://langdock.com) (mit Web-Search-Tool) |
| Mail | Gmail / Outlook OAuth |

---

## Lizenz & Nutzung

Alle Materialien stehen **DECAID-Academy-Teilnehmern** zur freien Nutzung in eigenen Projekten zur Verfügung. Bitte den DECAID-Hinweis in der Footer-Section beibehalten, wenn du die Landingpage 1:1 übernimmst.

---

## Kontakt

Florian Langer · [florian@decaid.studio](mailto:florian@decaid.studio) · [decaid.ai](https://decaid.ai)
