# Workflow 1 — Web Research & Smart Routing

> **Was er macht:** Empfängt jeden Webinar-Lead. Recherchiert dessen Firma live im Web (Revenue, News, strategische Themen). Schickt — abhängig von der Watch-Time — entweder ein **Sales-Briefing** an Sales (für Hot Leads) oder eine **personalisierte Re-Engagement-Mail** an Marketing (für No-Shows).

**Setup-Zeit:** ~15 Minuten · **Werkzeuge:** Langdock (Web Search Tool aktiviert) + ein Mail-Account

---

## Architektur

```
[Webhook]  →  [Web Research Agent]  →  [Condition: watch_time ≥ 30?]
                                              ↓                  ↓
                                          ja, hot           nein, cold
                                              ↓                  ↓
                                       [Sales-Briefing]  [Re-Engagement]
                                              ↓                  ↓
                                          [Mail Sales]    [Mail Marketing]
```

**Der entscheidende Unterschied zu naiven Setups:** Die Recherche läuft für **jeden** Lead — auch für No-Shows. So kann auch die Re-Engagement-Mail personalisiert sein, statt generischer Nurture-Standardware.

---

## Was die Webhook-Payload **nicht** enthält

Webinar-Plattformen (Demio, Livestorm, Zoom Webinars) liefern dir nur:

- Vorname, Nachname, E-Mail
- Firma, Position
- Watch-Time

**Sie liefern dir nicht:** Jahresumsatz, Branche, aktuelle News, strategische Themen. Das alles ergänzt die KI im Web-Research-Node.

---

## Node 1 — Webhook Trigger

| Setting | Wert |
|---|---|
| **Type** | Webhook |
| **Method** | POST |
| **Erwarteter Body** | JSON (siehe unten) |

**Erwartete Payload-Struktur:**

```json
{
  "event": "webinar_engagement_update",
  "lead": {
    "firstname": "Gabi",
    "lastname": "Schneider",
    "email": "g.schneider@gerolsteiner.example",
    "company_name": "Gerolsteiner Brunnen GmbH & Co. KG",
    "jobtitle": "VP of RevOps",
    "webinar_watch_time_minutes": 58
  }
}
```

> **Tipp:** Klick im Webhook-Node auf **"Copy URL"** und füg sie in `colab/webinar_simulator.ipynb` ein (Variable `LANGDOCK_WEBHOOK_URL`).

---

## Node 2 — Web Research Agent

**Modell:** Claude Sonnet (gut bei langen Web-Outputs) · **Tool aktiviert:** ✅ Web Search · **Output-Format:** JSON

### Input

```
{{webhook.body.lead.company_name}}
```

### System Prompt

```text
Du bist ein B2B-Sales-Researcher. Deine Aufgabe: Innerhalb von 90 Sekunden
herausfinden, wer diese Firma ist und was sie strategisch beschäftigt.

Vorgehen:

1. Nutze das Web Search Tool, um die 3 aktuellsten relevanten Quellen zur Firma
   zu finden: Pressemitteilungen, Quartalszahlen, Personalwechsel auf C-Level,
   neue Produktlaunches, Akquisitionen.
2. Schätze den Jahresumsatz auf Basis öffentlicher Daten (Bundesanzeiger,
   Konzernberichte, Branchenpublikationen). Wenn unklar → "unbekannt".
3. Klassifiziere die Firmengröße: small (<5M €), mid (5–50M €),
   enterprise (>50M €).
4. Ignoriere Wikipedia-Übersichten älter als 12 Monate, Klatschpresse,
   Stellenanzeigen.
5. Identifiziere die WICHTIGSTE strategische Initiative der Firma.

Antworte AUSSCHLIESSLICH mit gültigem JSON:

{
  "company_size": "<small|mid|enterprise>",
  "estimated_revenue_eur": <int oder null>,
  "industry": "<Branche in 2-3 Wörtern>",
  "research_summary": "<3-5 Sätze über die aktuellen strategischen Themen, auf Deutsch>",
  "top_initiative": "<EIN strategisches Vorhaben, in einem Satz>",
  "icebreaker": "<Wortwörtlicher Satz, mit dem ein Sales-Rep den Call beginnen könnte>",
  "sources": ["<URL 1>", "<URL 2>", "<URL 3>"]
}
```

### User Prompt

```text
Firma: {{webhook.body.lead.company_name}}

Recherchiere und antworte im JSON-Format.
```

> **Wichtig:** Die Recherche läuft für **alle** Leads — auch für No-Shows. Das ist Absicht. Die Re-Engagement-Mail soll genauso personalisiert sein wie das Sales-Briefing.

---

## Node 3 — Condition Router

| Bedingung | Pfad |
|---|---|
| `{{webhook.body.lead.webinar_watch_time_minutes}} >= 30` | **Pfad A (Hot)** → Sales-Briefing |
| sonst | **Pfad B (Cold/No-Show)** → Re-Engagement |

> **Anti-Stolperdraht:** Falls Langdock den Watch-Time-Wert als String reinbekommt, vergleiche mit `"30"` als String oder konvertiere via `parseInt()`.

> **Warum nur Watch-Time entscheidet:** Wir wollen die Logik bewusst einfach halten. Firmengröße/Revenue können später als sekundäre Sortierung im Sales-Inbox-Filter gemacht werden.

---

## Node 4a — Sales-Briefing (Pfad A, Hot)

**Integration:** Gmail / Outlook · **Trigger:** Watch-Time ≥ 30 Min

| Feld | Wert |
|---|---|
| **To** | `<YOUR_SALES_INBOX>` (z.B. `sales@your-company.com`) |
| **From** | `<YOUR_SENDER_ADDRESS>` (OAuth-verbundener Absender) |
| **Subject** | `🔥 HOT LEAD [{{webhook.body.lead.webinar_watch_time_minutes}} Min] — {{webhook.body.lead.company_name}}` |

### Body

```text
Hey Sales-Team,

{{webhook.body.lead.firstname}} {{webhook.body.lead.lastname}}
({{webhook.body.lead.jobtitle}} bei {{webhook.body.lead.company_name}})
hat das Webinar fast komplett geschaut: {{webhook.body.lead.webinar_watch_time_minutes}} Min.

────────────────────────────────────────────
🏢 Was wir über die Firma wissen (Live-Recherche):

Größe: {{node_2.output.company_size}}
Geschätzter Jahresumsatz: {{node_2.output.estimated_revenue_eur}} €
Branche: {{node_2.output.industry}}
────────────────────────────────────────────

📰 Was bei {{webhook.body.lead.company_name}} gerade läuft:

{{node_2.output.research_summary}}

→ Wichtigste strategische Initiative:
   {{node_2.output.top_initiative}}

────────────────────────────────────────────
💬 Empfohlener Eisbrecher für den Call:

   "{{node_2.output.icebreaker}}"

────────────────────────────────────────────
🔗 Quellen:
   {{node_2.output.sources}}

— DECAID Web-Research-Agent
```

---

## Node 4b — Re-Engagement-Mail (Pfad B, Cold/No-Show)

**Integration:** Gmail / Outlook · **Trigger:** Watch-Time < 30 Min

> **Pädagogisch wichtig:** Diese Mail geht an dein **Marketing-Team**, nicht an Sales. Marketing entscheidet, ob/wie der Lead in eine Nurture-Sequenz eingebucht wird. Die Mail liefert die Hooks für eine personalisierte Wiederansprache.

| Feld | Wert |
|---|---|
| **To** | `<YOUR_MARKETING_INBOX>` (z.B. `marketing@your-company.com`) |
| **Subject** | `❄️ NO-SHOW Follow-up — {{webhook.body.lead.company_name}} ({{webhook.body.lead.webinar_watch_time_minutes}} Min)` |

### Body

```text
Hey Marketing-Team,

{{webhook.body.lead.firstname}} {{webhook.body.lead.lastname}}
({{webhook.body.lead.jobtitle}} bei {{webhook.body.lead.company_name}})
hat sich angemeldet, aber das Webinar nur {{webhook.body.lead.webinar_watch_time_minutes}} Min geschaut.

→ Standard-Nurture wäre Verschwendung. Hier sind die Hooks, mit denen wir
  eine personalisierte Re-Engagement-Mail schreiben können.

────────────────────────────────────────────
🏢 Firma im Kontext:

Größe: {{node_2.output.company_size}}
Geschätzter Jahresumsatz: {{node_2.output.estimated_revenue_eur}} €
Branche: {{node_2.output.industry}}

📰 Was bei {{webhook.body.lead.company_name}} aktuell läuft:

{{node_2.output.research_summary}}

→ Strategisches Top-Thema:
   {{node_2.output.top_initiative}}
────────────────────────────────────────────

💌 Vorschlag für die Re-Engagement-Mail:

Subject: "Schade, dass es nicht geklappt hat — kurzer Gedanke zu {{node_2.output.top_initiative}}"

Body (Vorschlag, anpassen):

  Hi {{webhook.body.lead.firstname}},

  schade, dass es zeitlich beim Webinar gestern nicht geklappt hat.

  Ich habe gesehen, dass ihr bei {{webhook.body.lead.company_name}} gerade
  bei {{node_2.output.top_initiative}} unterwegs seid — und genau dafür
  hatten wir im Webinar einen Workflow gezeigt, der euch in der
  Umsetzung Wochen sparen würde.

  Wenn du 5 Minuten hast, schick ich dir die 3 Kern-Slides plus den
  Workflow-Mitschnitt. Reicht zum Einordnen, ob's für euch relevant ist.

  Beste Grüße,
  [Dein Name]

────────────────────────────────────────────
🔗 Quellen für die Recherche:
   {{node_2.output.sources}}

— DECAID Web-Research-Agent
```

> **Optional erweiterbar:** Statt manuell zu versenden, kannst du einen weiteren Agent-Node davor schalten, der die Re-Engagement-Mail komplett ausschreibt (statt Vorschlag). Für die Masterclass lassen wir's beim Vorschlag — Marketing hat das letzte Wort.

---

## Test-Checklist

Bevor du live gehst:

- [ ] Webhook-URL kopiert und in `colab/webinar_simulator.ipynb` eingetragen
- [ ] Web-Search-Tool im Workflow-Settings aktiviert
- [ ] Mail-Account verbunden (Gmail/Outlook OAuth)
- [ ] **Test mit Jürgen** (3 Min Watch) → Re-Engagement-Mail kommt
- [ ] **Test mit Gabi** (58 Min Watch) → Sales-Briefing-Mail kommt
- [ ] Beide Mails enthalten konkrete Recherche-Inhalte zur Firma
- [ ] Eisbrecher / Re-Engagement-Hook wirken **konkret**, nicht generisch

---

## Troubleshooting

| Problem | Ursache | Fix |
|---|---|---|
| Webhook gibt 200, aber keine Mail | Mail-Node nicht verbunden oder OAuth abgelaufen | Re-Auth Mail-Integration |
| Web Search liefert leere Quellen | Firmenname zu generisch | Vollständigen offiziellen Namen einsetzen |
| Beide Mails kommen | Condition feuert nicht | Watch-Time-String vs. Int prüfen |
| Mail im Spam | SPF/DKIM nicht gesetzt | Für Demo egal, in Prod relevant |
| `estimated_revenue_eur` ist `null` | Nicht-börsennotiertes Privatunternehmen | OK — Mail-Template muss `null` graceful handhaben |

---

## Erweiterungen für Production

- **HubSpot-Update**: parallel zur E-Mail einen HubSpot-Lifecycle-Update schreiben (Hot → Sales-Owner, Cold → Marketing-Nurture-Liste).
- **Slack-Alert**: bei Hot-Path zusätzlich in `#sales-hot-leads` posten.
- **Cooldown**: gleicher Lead nicht öfter als 1× pro 7 Tage.
- **Auto-Send Re-Engagement**: zusätzlicher Agent-Node, der die Re-Engagement-Mail vollständig schreibt und versendet (statt nur Vorschlag).
- **Logging**: Output von Node 2 in einer Tabelle ablegen für spätere Konversionsanalyse.
