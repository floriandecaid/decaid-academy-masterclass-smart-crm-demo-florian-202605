# Workflow 1 — Web Research & Smart Routing

> **Was er macht:** Empfängt jeden Webinar-Lead. Recherchiert dessen Firma live im Web (Revenue, News, strategische Themen). Erstellt — abhängig von der Watch-Time — entweder ein **Sales-Briefing-Doc** im Sales-Ordner (für Hot Leads) oder ein **Re-Engagement-Briefing-Doc** im Marketing-Ordner (für No-Shows).

**Setup-Zeit:** ~15 Minuten · **Werkzeuge:** Langdock (Web Search Tool aktiviert) + Google Drive

---

## Architektur

```
[Webhook]  →  [Web Research Agent]  →  [Condition: watch_time ≥ 30?]
                                              ↓                  ↓
                                          ja, hot           nein, cold
                                              ↓                  ↓
                                       [Sales-Briefing]  [Re-Engagement]
                                              ↓                  ↓
                                       [Doc → Sales-     [Doc → Marketing-
                                        Ordner]          Ordner]
```

**Der entscheidende Unterschied zu naiven Setups:** Die Recherche läuft für **jeden** Lead — auch für No-Shows. So kann auch das Re-Engagement-Briefing personalisiert sein, statt generischer Nurture-Standardware.

**Warum Google Docs statt E-Mail?** Docs landen sofort visuell auffindbar in einem geteilten Ordner, lassen sich formatieren (Tabellen, klickbare Quellen, Bilder), kommen nie in den Spam und sind Team-kollaborativ. Wenn du lieber Mail willst: tausch den letzten Node aus, der Rest bleibt identisch.

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

## Node 4a — Sales-Briefing-Doc (Pfad A, Hot)

**Integration:** Google Drive (Create Document) · **Trigger:** Watch-Time ≥ 30 Min

| Feld | Wert |
|---|---|
| **Drive-Ordner** | `<YOUR_SALES_FOLDER_ID>` (Ordner-ID aus Drive-URL: `drive.google.com/drive/folders/<ID>`) |
| **Doc-Name** | `🔥 HOT — {{webhook.body.lead.company_name}} ({{webhook.body.lead.webinar_watch_time_minutes}} Min) — {{webhook.body.lead.firstname}} {{webhook.body.lead.lastname}}` |
| **Format** | Markdown oder Plain Text (Drive konvertiert beim Anlegen) |

### Body

```markdown
# 🔥 Sales-Briefing: {{webhook.body.lead.company_name}}

**Lead:** {{webhook.body.lead.firstname}} {{webhook.body.lead.lastname}} — {{webhook.body.lead.jobtitle}}
**E-Mail:** {{webhook.body.lead.email}}
**Watch Time:** {{webhook.body.lead.webinar_watch_time_minutes}} Min (von 60)
**Erstellt:** {{webhook.received_at}}

---

## 🏢 Firma im Kontext (Live-Recherche)

| | |
|---|---|
| Größe | {{node_2.output.company_size}} |
| Geschätzter Jahresumsatz | {{node_2.output.estimated_revenue_eur}} € |
| Branche | {{node_2.output.industry}} |

## 📰 Was bei {{webhook.body.lead.company_name}} gerade läuft

{{node_2.output.research_summary}}

**Wichtigste strategische Initiative:**
{{node_2.output.top_initiative}}

## 💬 Empfohlener Eisbrecher für den Call

> "{{node_2.output.icebreaker}}"

## 🔗 Quellen

{{node_2.output.sources}}

---

*Generiert vom DECAID Web-Research-Agent.*
```

---

## Node 4b — Re-Engagement-Doc (Pfad B, Cold/No-Show)

**Integration:** Google Drive (Create Document) · **Trigger:** Watch-Time < 30 Min

> **Pädagogisch wichtig:** Dieses Doc landet im **Marketing-Ordner**, nicht im Sales-Ordner. Marketing entscheidet, ob/wie der Lead in eine Nurture-Sequenz eingebucht wird. Das Doc liefert die Hooks für eine personalisierte Wiederansprache.

| Feld | Wert |
|---|---|
| **Drive-Ordner** | `<YOUR_MARKETING_FOLDER_ID>` (Ordner-ID aus Drive-URL) |
| **Doc-Name** | `❄️ NO-SHOW — {{webhook.body.lead.company_name}} ({{webhook.body.lead.webinar_watch_time_minutes}} Min) — {{webhook.body.lead.firstname}} {{webhook.body.lead.lastname}}` |
| **Format** | Markdown oder Plain Text |

### Body

```markdown
# ❄️ Re-Engagement-Briefing: {{webhook.body.lead.company_name}}

**Lead:** {{webhook.body.lead.firstname}} {{webhook.body.lead.lastname}} — {{webhook.body.lead.jobtitle}}
**E-Mail:** {{webhook.body.lead.email}}
**Watch Time:** {{webhook.body.lead.webinar_watch_time_minutes}} Min (No-Show / abgebrochen)
**Erstellt:** {{webhook.received_at}}

> Standard-Nurture wäre Verschwendung. Hier sind die Hooks, mit denen wir eine personalisierte Re-Engagement-Mail schreiben können.

---

## 🏢 Firma im Kontext

| | |
|---|---|
| Größe | {{node_2.output.company_size}} |
| Geschätzter Jahresumsatz | {{node_2.output.estimated_revenue_eur}} € |
| Branche | {{node_2.output.industry}} |

## 📰 Was bei {{webhook.body.lead.company_name}} aktuell läuft

{{node_2.output.research_summary}}

**Strategisches Top-Thema:**
{{node_2.output.top_initiative}}

## 💌 Vorschlag für die Re-Engagement-Mail

**Subject:** Schade, dass es nicht geklappt hat — kurzer Gedanke zu {{node_2.output.top_initiative}}

**Body (Vorschlag, anpassen):**

> Hi {{webhook.body.lead.firstname}},
>
> schade, dass es zeitlich beim Webinar gestern nicht geklappt hat.
>
> Ich habe gesehen, dass ihr bei {{webhook.body.lead.company_name}} gerade bei **{{node_2.output.top_initiative}}** unterwegs seid — und genau dafür hatten wir im Webinar einen Workflow gezeigt, der euch in der Umsetzung Wochen sparen würde.
>
> Wenn du 5 Minuten hast, schick ich dir die 3 Kern-Slides plus den Workflow-Mitschnitt. Reicht zum Einordnen, ob's für euch relevant ist.
>
> Beste Grüße,
> [Dein Name]

## 🔗 Quellen für die Recherche

{{node_2.output.sources}}

---

*Generiert vom DECAID Web-Research-Agent.*
```

> **Optional erweiterbar:** Statt manuell zu versenden, kannst du einen weiteren Agent-Node davor schalten, der die Re-Engagement-Mail komplett ausschreibt (statt Vorschlag). Für die Masterclass lassen wir's beim Vorschlag — Marketing hat das letzte Wort.

---

## Test-Checklist

Bevor du live gehst:

- [ ] Webhook-URL kopiert und in `colab/webinar_simulator.ipynb` eingetragen
- [ ] Web-Search-Tool im Workflow-Settings aktiviert
- [ ] Google-Drive-Integration verbunden (OAuth)
- [ ] Sales-Ordner und Marketing-Ordner in Drive angelegt, IDs in beiden Doc-Nodes eingetragen
- [ ] **Test mit Jürgen** (3 Min Watch) → Re-Engagement-Doc landet im Marketing-Ordner
- [ ] **Test mit Gabi** (58 Min Watch) → Sales-Briefing-Doc landet im Sales-Ordner
- [ ] Beide Docs enthalten konkrete Recherche-Inhalte zur Firma
- [ ] Eisbrecher / Re-Engagement-Hook wirken **konkret**, nicht generisch

---

## Troubleshooting

| Problem | Ursache | Fix |
|---|---|---|
| Webhook gibt 200, aber kein Doc erscheint | Drive-Node nicht verbunden oder OAuth abgelaufen | Re-Auth Drive-Integration |
| Doc landet im falschen Ordner | Ordner-ID vertauscht | IDs aus Drive-URL nochmal kopieren |
| Web Search liefert leere Quellen | Firmenname zu generisch | Vollständigen offiziellen Namen einsetzen |
| Beide Docs werden erstellt | Condition feuert nicht | Watch-Time-String vs. Int prüfen |
| `estimated_revenue_eur` ist `null` | Nicht-börsennotiertes Privatunternehmen | OK — Template muss `null` graceful handhaben |
| Doc-Inhalt ist Plain-Text statt formatiert | Drive-Node speichert raw | Doc-Format auf "Markdown" stellen oder Node nutzen, der Markdown rendert |

---

## Erweiterungen für Production

- **E-Mail-Versand parallel**: Wenn dein Team eher in der Inbox lebt, hänge zusätzlich einen Send-Email-Node an die jeweiligen Pfade — Doc plus Mail mit Link drauf.
- **HubSpot-Update**: parallel zur Doc-Erstellung einen HubSpot-Lifecycle-Update schreiben (Hot → Sales-Owner, Cold → Marketing-Nurture-Liste).
- **Slack-Alert**: bei Hot-Path zusätzlich in `#sales-hot-leads` posten — mit Drive-Link zum Briefing-Doc.
- **Cooldown**: gleicher Lead nicht öfter als 1× pro 7 Tage.
- **Auto-Send Re-Engagement**: zusätzlicher Agent-Node, der die Re-Engagement-Mail vollständig schreibt und versendet (statt nur Vorschlag im Doc).
- **Logging**: Output von Node 2 zusätzlich in einem Sheet ablegen für spätere Konversionsanalyse.
