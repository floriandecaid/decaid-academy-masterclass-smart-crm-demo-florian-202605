# Architektur — System-Übersicht

> Vier Bausteine. Kein Backend. Keine Server. Du brauchst nur einen Langdock-Account und 60 Minuten Zeit.

---

## Big Picture

```
┌──────────────────┐    ┌──────────────────┐    ┌────────────────────────────┐    ┌──────────────┐
│  Webinar-Page    │    │  Daten-Sync      │    │   Langdock Workflow        │    │  Sales-Inbox │
│  (GitHub Pages)  │───▶│  Colab oder JS   │───▶│   Triage → Search → Mail   │───▶│  florian+... │
│                  │    │  Webhook-POST    │    │                            │    │              │
└──────────────────┘    └──────────────────┘    └────────────────────────────┘    └──────────────┘
```

Du kannst entweder die Landingpage **direkt** als Frontend benutzen (`index.html` mit JS-Webhook), oder das Colab-Notebook nehmen, um Daten gezielt für die Live-Demo zu triggern. Beide Wege landen am selben Webhook.

---

## Komponenten im Detail

### 1. Frontend — `index.html`

| Aspekt | Detail |
|---|---|
| **Hosting** | GitHub Pages (kostenlos, HTTPS, statisch) |
| **Stack** | Plain HTML + Vanilla JS, keine Build-Pipeline |
| **Look** | DECAID-Branding (Primary `#4A3AFF`, Satoshi/Lato) |
| **Funktion** | Registrierungsformular + zwei Demo-Trigger-Buttons (Jürgen / Gabi) |
| **Datenfluss** | `fetch(LANGDOCK_WEBHOOK_URL, {method: 'POST', body: persona})` |

**Warum kein Framework?** Weil's keinen Bedarf gibt. Eine Landingpage in Vanilla JS lädt in 100ms, hat keine Build-Schritte, und Teilnehmer können sie in jedem Editor ihrer Wahl anpassen.

---

### 2. Daten-Sync — `colab/webinar_simulator.ipynb`

| Aspekt | Detail |
|---|---|
| **Plattform** | Google Colab (kostenlos, läuft im Browser) |
| **Sprache** | Python 3 (nur stdlib, keine pip installs) |
| **Funktion** | Konfiguration + Mock-Personas + Webhook-POST |

**Warum Colab statt Standalone-Script?**
- Teilnehmer müssen nichts installieren.
- Code ist mit Markdown-Erklärungen verzahnt (selbsterklärend).
- Live-Demo-fähig: jede Zelle einzeln ausführbar.

---

### 3. Langdock Workflow

Das ist der "intelligente" Teil. Zwei separate Workflows:

#### Workflow 1 — Web Research & Smart Routing (live während der Masterclass)

```
[Webhook] → [Web Research] → [Condition] → [Sales-Briefing-Mail]
                                ↓
                          [Re-Engagement-Mail]
```

**Recherchiert jeden Lead** (Revenue, News, strategische Themen). Routet basierend auf Watch-Time: Hot → Sales-Briefing für Sales. Cold/No-Show → Re-Engagement-Hooks für Marketing. Kein Lead fällt durchs Raster.

→ Details: [`workflow1-lead-triage.html`](workflow1-lead-triage.html)

#### Workflow 2 — Battle Cards (Bonus-Demo)

```
[Webhook] → [Extract Objections] → [Synthesize] → [Mail]
```

Wertet 8–10 Sales-Call-Transkripte aus und generiert eine wöchentliche Battle Card.

→ Details: [`workflow2-battle-cards.html`](workflow2-battle-cards.html)

---

### 4. Sales-Inbox

| Aspekt | Detail |
|---|---|
| **Adresse-Setup** | Zwei Inboxen empfohlen: `<YOUR_SALES_INBOX>` für Hot-Path, `<YOUR_MARKETING_INBOX>` für Cold-Path |
| **Warum getrennt?** | Sales soll nur Hot Leads sehen, Marketing entscheidet über Re-Engagement-Sequenzen separat |
| **Content** | Markdown oder HTML, generiert von Langdock |

---

## Was bewusst **nicht** Teil der Demo ist

| Komponente | Warum nicht |
|---|---|
| **HubSpot-Sync** | Wir wollen keine Mock-Personas in einem produktiven CRM. In Prod relevant, in Demo nicht. |
| **Slack-Notifications** | Reduktion auf das Wesentliche. Jeder kann's später ergänzen. |
| **Persistente Datenbank** | Für die Demo unnötig. In Prod könntest du Webhooks zusätzlich loggen. |
| **Auth/User-Management** | Statische Page, kein Login. |

---

## Tech-Stack-Zusammenfassung

| Layer | Tool | Kostenpunkt |
|---|---|---|
| Frontend | GitHub Pages + Vanilla HTML/CSS/JS | $0 |
| Trigger | Google Colab + Python stdlib | $0 |
| Workflow-Engine | Langdock | ab ~$30/Monat |
| Web Search | Langdock-integriert (Bing/Google API) | inkludiert |
| Mail | Gmail/Outlook OAuth | $0 (existing account) |
| **Gesamt** | — | **~$30/Monat** |

---

## Skalierungs-Pfad

So sieht's aus, wenn du das Setup in Production schiebst:

1. **Frontend**: GitHub Pages → Vercel/Netlify mit eigener Domain.
2. **Trigger**: Statt Colab direkter Sync aus deinem CRM (HubSpot Workflow → Webhook).
3. **Langdock**: Bleibt. Skaliert von 100 zu 100k Leads/Monat ohne Code-Änderung.
4. **Mail**: Statt Gmail-OAuth → Postmark/Sendgrid mit eigener Domain.
5. **Logging**: Jede Triage in BigQuery/Snowflake mitschreiben für Konversionsanalyse.

Die Architektur ändert sich nicht. Nur die Komponenten werden professionalisiert.
