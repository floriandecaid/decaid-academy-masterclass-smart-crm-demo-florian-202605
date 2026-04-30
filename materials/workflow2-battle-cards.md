# Workflow 2 — Battle Cards aus Sales-Calls

> **Was er macht:** Du wirfst 8–10 Sales-Call-Transkripte rein. Die KI extrahiert wiederkehrende Einwände + ICP-Merkmale und schickt deinem Team eine fertige Battle Card per E-Mail — wöchentlich oder on-demand.

**Setup-Zeit:** ~10 Minuten · **Werkzeuge:** Langdock + Mail-Account

---

## Architektur

```
[Webhook]  →  [Extract Agent]  →  [Synthesize Agent]  →  [Email]
   ↑
8–10 Transkripte als Array
```

---

## Node 1 — Webhook Trigger

**Erwartete Payload:**

```json
{
  "event": "weekly_battle_card_generation",
  "week": "2026-W22",
  "transcripts": [
    {
      "deal_stage": "discovery",
      "outcome": "lost",
      "industry": "Manufacturing",
      "company_size": "200-500",
      "transcript": "Sales: Hey Frau Becker, danke für die Zeit. Lass uns über..."
    },
    {
      "deal_stage": "demo",
      "outcome": "won",
      "industry": "SaaS",
      "company_size": "50-200",
      "transcript": "Sales: Du hast erwähnt, dass euer aktueller Stack..."
    }
    // ... weitere 6–8 Einträge
  ]
}
```

> Die Transkripte liegen im Repo unter `transcripts/fake_sales_calls/` als einzelne `.txt`-Dateien. Du kannst sie in Langdock einlesen oder via Webhook posten.

---

## Node 2 — Extract Agent (Objections + ICP)

**Modell:** Claude Sonnet (für längere Kontexte besser) · **Output-Format:** JSON

### System Prompt

```text
Du bist ein Sales-Enablement-Analyst. Du bekommst eine Liste von
Sales-Call-Transkripten. Deine Aufgabe: Aus jedem einzelnen Transkript
strukturierte Daten extrahieren.

Für JEDES Transkript extrahiere:

1. Top 1–3 Einwände, die der Prospect geäußert hat (wortwörtliche Zitate
   bevorzugt, sonst paraphrasiert).
2. Entscheidende ICP-Signale (Firmengröße, Branche, Pain Point, aktueller Stack,
   Buying Intent).
3. Wenn outcome=won: was hat den Deal gekippt? Welche Antwort/Demo/Aussage?
4. Wenn outcome=lost: was war der finale Showstopper?

Antworte AUSSCHLIESSLICH mit gültigem JSON:

{
  "extractions": [
    {
      "industry": "<aus Input>",
      "outcome": "<won|lost|open>",
      "objections": ["<Einwand 1>", "<Einwand 2>"],
      "icp_signals": ["<Signal 1>", "<Signal 2>"],
      "tipping_point": "<was hat es gekippt, in einem Satz>"
    }
  ]
}
```

### Input-Mapping

```
{{webhook.body.transcripts}}
```

---

## Node 3 — Synthesize Agent (Battle Card)

**Modell:** GPT-4 oder Claude Sonnet · **Output-Format:** Markdown (kein JSON, da direkt in Mail)

### System Prompt

```text
Du bist ein Senior-Sales-Coach. Du bekommst eine strukturierte Auswertung von
~10 Sales Calls der letzten Woche und schreibst daraus eine Battle Card, die
Reps am Montagmorgen in 3 Minuten lesen können.

Format (strikt, Markdown):

# Battle Card — Woche {{webhook.body.week}}

## 🎯 Top 3 wiederkehrende Einwände

Für jeden Einwand:
- **Der Einwand:** wortwörtlich oder paraphrasiert
- **Wie oft kam er?** (X von Y Calls)
- **Was hat funktioniert?** (Beispiel aus einem won-Deal)
- **Was floppt?** (Beispiel aus einem lost-Deal)

## 🟢 ICP-Signale, die heiße Deals zeigen

Liste der 3–5 stärksten Signale, die in won-Deals immer wieder vorkamen.

## 🔴 Anti-Signale — wenn du das hörst, qualifiziere ab

Liste der 2–3 Signale, die zuverlässig in lost-Deals vorkamen.

## 📞 Empfohlener Diskussions-Eröffner für nächste Woche

Ein konkreter Eröffnungssatz, basierend auf dem Muster der besten Calls.

WICHTIG:
- Schreibe auf Deutsch.
- Sei konkret, keine Floskeln.
- Wenn die Datenbasis dünn ist, sag es ehrlich.
- Maximal 600 Wörter.
```

### Input-Mapping

```
{{node_2.output.extractions}}
```

---

## Node 4 — Send Email an Sales

| Feld | Wert |
|---|---|
| **To** | `<YOUR_SALES_INBOX>` (z.B. `sales@your-company.com`) |
| **Subject** | `📋 Battle Card — Woche {{webhook.body.week}}` |
| **Body** | `{{node_3.output}}` (Markdown direkt) |

> **Tipp:** Wenn dein Mail-Provider Markdown nicht rendert, hänge `marked.js` o.ä. davor, oder lass Node 3 direkt HTML ausgeben.

---

## Test-Daten

Im Repo liegen 8 fiktive Sales-Call-Transkripte unter:

```
transcripts/fake_sales_calls/
├── 01-saas-won-discovery.txt
├── 02-manufacturing-lost-pricing.txt
├── 03-fintech-won-demo.txt
├── 04-retail-lost-integration.txt
├── 05-healthcare-open-followup.txt
├── 06-logistics-won-procurement.txt
├── 07-mediacorp-lost-changemgmt.txt
└── 08-energy-won-csuite.txt
```

Jede Datei enthält ein vollständiges, fiktives Transkript mit klar markierten Einwänden — perfekt zum Testen.

---

## Erweiterungen

- **Cron-Trigger** statt Webhook: jeden Montag 7:00 Uhr automatisch.
- **Slack-Post** zusätzlich zur Mail.
- **Vergleich Woche-zu-Woche**: Trends in Einwänden tracken.
- **Persona-spezifisch**: Battle Cards getrennt für SMB/MidMarket/Enterprise.
