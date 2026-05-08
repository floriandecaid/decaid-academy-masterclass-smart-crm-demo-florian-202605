# Workflow 2 — Battle Cards aus Sales-Calls

> **Was er macht:** Du wirfst 8–10 Sales-Call-Transkripte rein. Die KI extrahiert wiederkehrende Einwände + ICP-Merkmale und legt eine fertige Battle Card als Google Doc in einem geteilten Ordner ab — wöchentlich oder on-demand.

**Setup-Zeit:** ~10 Minuten · **Werkzeuge:** Langdock + Google Drive

---

## Architektur

```
[Webhook]  →  [Extract Agent]  →  [Synthesize Agent]  →  [Create Document]  →  [Update Document]
   ↑                                                          legt leeres Doc       schreibt Markdown
8–10 Transkripte                                              im Drive-Ordner an    ins Doc rein
als Array
```

**5 Nodes** — gleiche Drive-Pattern wie Workflow 1 (Create gibt `documentId` zurück, Update schreibt Body rein). Synthesize gibt zwei Felder zurück: `doc_title` und `doc_body`.

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

**Modell:** GPT-4 oder Claude Sonnet · **Output-Format:** JSON mit `doc_title` + `doc_body` (Markdown im Body)

Wie bei Larry: zwei Felder zurück, getrennt verwendbar von Create und Update Document.

### System Prompt

```text
Du bist ein Senior-Sales-Coach. Du bekommst eine strukturierte Auswertung von
~10 Sales Calls der letzten Woche und schreibst daraus eine Battle Card, die
Reps am Montagmorgen in 3 Minuten lesen können.

# Output-Contract

Du gibst ein einziges JSON-Objekt zurück mit genau zwei Feldern:

{
  "doc_title": "<String, siehe Title-Template unten>",
  "doc_body":  "<String, vollständiger Markdown-Body, siehe Body-Template unten>"
}

- Keine Markdown-Code-Fences um das JSON.
- Keine extra Felder, kein Fließtext davor/danach.
- doc_body ist Markdown — Drive rendert die Formatierung.

# Title-Template

`📋 Battle Card — Woche {week}` (z.B. `📋 Battle Card — Woche 2026-W22`)

# Body-Template (Markdown im doc_body)

# 📋 Battle Card — Woche {week}

Basierend auf {N} Sales-Calls der Vorwoche.

## 🎯 Top 3 wiederkehrende Einwände

Für jeden Einwand:
- **Der Einwand:** wortwörtlich oder paraphrasiert
- **Wie oft kam er?** ({X} von {N} Calls)
- **Was hat funktioniert?** Beispiel aus einem won-Deal
- **Was floppt?** Beispiel aus einem lost-Deal

## 🟢 ICP-Signale, die heiße Deals zeigen

Liste der 3–5 stärksten Signale, die in won-Deals immer wieder vorkamen.

## 🔴 Anti-Signale — wenn du das hörst, qualifiziere ab

Liste der 2–3 Signale, die zuverlässig in lost-Deals vorkamen.

## 📞 Empfohlener Diskussions-Eröffner für nächste Woche

Ein konkreter Eröffnungssatz, basierend auf dem Muster der besten Calls.

---

# Wichtige Hinweise

- Schreibe auf Deutsch.
- Sei konkret, keine Floskeln.
- Wenn die Datenbasis dünn ist, sag es ehrlich im Body.
- Body maximal 600 Wörter.
- Niemals geschweifte Klammern im Output stehen lassen — alle {Platzhalter}
  durch echte Werte ersetzen ({week}, {N}, {X}).
```

### Input-Mapping

```
{{node_2.output.extractions}}
```

---

## Node 4 — Create Document

**Integration:** Google Drive (Create Document)

Legt ein leeres Doc im Drive-Ordner an. Titel kommt vom Synthesize Agent.

| Feld | Wert |
|---|---|
| **Drive-Ordner** | `<YOUR_DRIVE_FOLDER_ID>` (gleicher Demo-Ordner wie Workflow 1, oder ein eigener `Battle Cards`-Unterordner) |
| **Doc-Titel** | `{{node_3.output.doc_title}}` |
| **Output** | `documentId` für Node 5 |

---

## Node 5 — Update Document

**Integration:** Google Drive (Update Document)

Schreibt den Markdown-Body in das gerade angelegte Doc.

| Feld | Wert |
|---|---|
| **Document ID** | `{{node_4.output.documentId}}` |
| **Body** | `{{node_3.output.doc_body}}` |
| **Format** | Markdown (Drive rendert Headings, Bullets, Quotes) |

> **Tipp:** Battle Cards funktionieren super als fortlaufende Sammlung. Lege einen Ordner `Battle Cards` an, sortiere nach "Zuletzt geändert" — neue Cards sind oben, alte historisierst du.

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
- **Slack-Post** mit Drive-Link zum frischen Battle-Card-Doc.
- **E-Mail-Versand parallel**: wenn dein Team eher in der Inbox lebt, hänge zusätzlich einen Send-Email-Node an — Mail mit Drive-Link drauf.
- **Vergleich Woche-zu-Woche**: Trends in Einwänden tracken.
- **Persona-spezifisch**: Battle Cards getrennt für SMB/MidMarket/Enterprise.
