# Architektur — System-Übersicht

> Drei Bausteine. Kein Backend. Keine Server. Du brauchst nur einen Langdock-Account, einen Google-Drive-Ordner und ~30 Minuten Zeit.

---

## Big Picture

```
┌──────────────────┐    ┌────────────────────────────┐    ┌──────────────────┐
│  Webinar-Page    │    │   Langdock Workflow        │    │  Google Drive    │
│  (GitHub Pages)  │───▶│   Web Research → Routing   │───▶│  Briefing-Doc    │
│  Webhook-POST    │    │   → Doc-Generation         │    │  im Ordner       │
└──────────────────┘    └────────────────────────────┘    └──────────────────┘
```

Die Landingpage triggert den Webhook direkt aus dem Browser. Langdock recherchiert die Firma live im Web und legt — abhängig von der Watch-Time — ein passendes Briefing-Doc in einem geteilten Drive-Ordner an.

> **Optional:** Statt Browser-Trigger kannst du jeden anderen HTTP-Client nutzen — Postman, curl, dein CRM-Webflow. Im Repo liegt zusätzlich ein [Colab-Notebook](../colab/webinar_simulator.ipynb), falls du die Pipeline aus einem Python-Kontext anstoßen willst.

---

## Komponenten im Detail

### 1. Frontend — `index.html`

| Aspekt | Detail |
|---|---|
| **Hosting** | GitHub Pages (kostenlos, HTTPS, statisch) |
| **Stack** | Plain HTML + Vanilla JS, keine Build-Pipeline |
| **Funktion** | Registrierungsformular + zwei Demo-Trigger-Buttons (Jürgen / Gabi) |
| **Datenfluss** | `fetch(LANGDOCK_WEBHOOK_URL, {method: 'POST', body: persona})` |

**Warum kein Framework?** Weil's keinen Bedarf gibt. Eine Landingpage in Vanilla JS lädt in 100ms, hat keine Build-Schritte, und du kannst sie in jedem Editor anpassen.

---

### 2. Langdock Workflow

Das ist der "intelligente" Teil. Zwei separate Workflows:

#### Workflow 1 — Web Research & Smart Routing (live während der Masterclass)

```
[Webhook] → [Web Research] → [Condition] → [Sales-Briefing-Doc]
                                ↓
                          [Re-Engagement-Doc]
```

**Recherchiert jeden Lead** (Revenue, News, strategische Themen). Routet basierend auf Watch-Time: Hot → Sales-Briefing-Doc im Sales-Ordner. Cold/No-Show → Re-Engagement-Doc im Marketing-Ordner. Kein Lead fällt durchs Raster.

→ Details: [`workflow1-lead-triage.html`](workflow1-lead-triage.html)

#### Workflow 2 — Battle Cards (Bonus-Demo)

```
[Webhook] → [Extract Objections] → [Synthesize] → [Doc → Sales-Ordner]
```

Wertet 8–10 Sales-Call-Transkripte aus und legt eine wöchentliche Battle Card als Doc ab.

→ Details: [`workflow2-battle-cards.html`](workflow2-battle-cards.html)

---

### 3. Google Drive — Output-Layer

| Aspekt | Detail |
|---|---|
| **Setup** | Zwei Ordner empfohlen: `Sales – Hot Leads` und `Marketing – Re-Engagement` (plus optional `Battle Cards`) |
| **Warum getrennt?** | Sales soll nur Hot-Briefings sehen, Marketing entscheidet über Re-Engagement separat |
| **Format** | Google Docs mit Markdown-Rendering — Tabellen, Quellen-Links, formatierte Briefings |
| **Auffindbarkeit** | "Sortieren nach: Zuletzt geändert" → neue Briefings poppen oben rein |

**Warum Drive statt Mail?**
- Kein Spam-Risiko
- Sofort visuell auffindbar im Ordner statt einzelne Mails
- Format-reich (Tabellen, Quellen-Links, Bilder)
- Team-kollaborativ (Kommentare, Tasks zuweisen)
- Kein OAuth-Drama mit Refresh-Tokens

---

## Was bewusst **nicht** Teil der Demo ist

| Komponente | Warum nicht |
|---|---|
| **HubSpot-Sync** | Wir wollen keine Mock-Personas in einem produktiven CRM. In Prod relevant, in Demo nicht. |
| **Slack-Notifications** | Reduktion auf das Wesentliche. Jeder kann's später ergänzen. |
| **Persistente Datenbank** | Für die Demo unnötig. In Prod könntest du Webhooks zusätzlich in BigQuery loggen. |
| **Auth/User-Management** | Statische Page, kein Login. |

---

## Tech-Stack-Zusammenfassung

| Layer | Tool | Kostenpunkt |
|---|---|---|
| Frontend | GitHub Pages + Vanilla HTML/CSS/JS | $0 |
| Workflow-Engine | Langdock | ab ~$30/Monat |
| Web Search | Langdock-integriert (Bing/Google API) | inkludiert |
| Output | Google Drive (Workspace oder Free) | $0 (existing account) |
| **Gesamt** | — | **~$30/Monat** |

---

## Skalierungs-Pfad

So sieht's aus, wenn du das Setup in Production schiebst:

1. **Frontend**: GitHub Pages → Vercel/Netlify mit eigener Domain (oder direkter Webhook-Trigger aus deinem echten Webinar-Tool).
2. **Trigger**: Direkter Sync aus deinem CRM/Webinar-Tool (HubSpot Workflow → Webhook, Demio Webhook, Zoom Webhook).
3. **Langdock**: Bleibt. Skaliert von 100 zu 100k Leads/Monat ohne Code-Änderung.
4. **Output**: Drive bleibt fürs Team-Briefing — zusätzlich Slack-Notifications oder direkter HubSpot-Lifecycle-Update.
5. **Logging**: Jeden Workflow-Run in BigQuery/Snowflake mitschreiben für Konversionsanalyse.

Die Architektur ändert sich nicht. Nur die Komponenten werden professionalisiert.
