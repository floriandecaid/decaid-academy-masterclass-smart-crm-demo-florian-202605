# Workflow 1 — Web Research & Smart Routing

> **Was er macht:** Empfängt jeden Webinar-Lead. Ein einziger Agent (Larry, the Lead Researcher) recherchiert die Firma live im Web und entscheidet selbst — basierend auf der Watch-Time — ob er ein **Sales-Briefing** für Hot Leads oder ein **Re-Engagement-Briefing** für No-Shows schreibt. Das Ergebnis landet als Google Doc im geteilten Ordner.

**Setup-Zeit:** ~10 Minuten · **Werkzeuge:** Langdock (Web Search Tool aktiviert) + Google Drive

---

## Architektur

```
[Webhook]  →  [Larry the Lead Researcher]  →  [Create Document]  →  [Update Document]
                  Web-Search aktiviert,         legt leeres Doc        schreibt das fertige
                  routet selbst (Hot/Cold)      im Drive-Ordner an     Briefing rein
                  und schreibt Markdown
```

Vier Nodes. Kein expliziter Condition-Router — **Larry entscheidet im Prompt selbst**, welcher Briefing-Modus passt. Das hält den Workflow schlank und macht das Prompt-Engineering zum eigentlichen Hebel.

**Der entscheidende Unterschied zu naiven Setups:** Die Recherche läuft für **jeden** Lead — auch für No-Shows. So kann auch das Re-Engagement-Briefing personalisiert sein, statt generischer Nurture-Standardware.

**Warum Google Docs statt E-Mail?** Docs landen sofort visuell auffindbar in einem geteilten Ordner, lassen sich formatieren (Tabellen, klickbare Quellen, Bilder), kommen nie in den Spam und sind Team-kollaborativ. Wenn du lieber Mail willst: hänge zusätzlich einen Send-Email-Node an, der Rest bleibt identisch.

**Warum zwei separate Doc-Nodes (Create + Update)?** Langdocks Google-Drive-Integration trennt das Anlegen eines Docs (Titel + Folder) vom Reinschreiben des Inhalts (Body). Create-Document gibt eine `documentId` zurück, Update-Document nimmt diese ID + den Markdown-Body und füllt das Doc.

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

> **Tipp:** Klick im Webhook-Node auf **"Copy URL"** und trage sie in der Landingpage als `LANGDOCK_WEBHOOK_URL` ein. Optional auch im Colab-Notebook, falls du aus Python triggerst.

---

## Node 2 — Larry the Lead Researcher (Agent)

**Modell:** Claude Sonnet (gut bei längerem Markdown-Output) · **Tool aktiviert:** ✅ Web Search · **Output-Format:** strukturiertes JSON mit `mode` + `doc_title` + `doc_body` (Markdown)

Larry ist das Hirn des Workflows. Er bekommt den Webhook-Payload, recherchiert die Firma live im Web, **entscheidet selbst auf Basis der Watch-Time**, ob er ein Sales-Briefing oder ein Re-Engagement-Briefing schreibt, und gibt drei Felder zurück: `mode` (sichtbar im Run-Log), `doc_title` (mit 🔥/❄️-Präfix) und `doc_body` (Markdown). Titel und Body werden von den nachfolgenden Drive-Nodes 1:1 verwendet.

### Input

Larry braucht den **kompletten Lead-Payload** — nicht nur den Firmennamen — weil er die Watch-Time fürs Routing und alle Lead-Daten fürs Briefing-Doc verwendet:

```
{{webhook.body.lead}}
```

### System Prompt

```text
# Rolle

Du bist Larry, B2B-Sales-Researcher mit Schwerpunkt DACH. Du arbeitest für ein
Team, das gleich diese Firma kontaktieren wird. Deine einzige Aufgabe:
in unter 90 Sekunden ein faktenbasiertes Mini-Briefing über die Firma liefern —
keine Verkaufstexte, keine Spekulation, keine Floskeln.

Du bist Researcher, nicht Texter. Du recherchierst und klassifizierst.
Den Eisbrecher (im Sales-Briefing-Modus) bzw. den Re-Engagement-Hook
(im No-Show-Modus) formulierst du nüchtern aus dem, was du gefunden hast —
nicht aus dem, was gut klingen würde.

# Modus-Wahl (machst du selbst)

Du bekommst die kompletten Lead-Daten als Input. Schau zuerst auf
`webinar_watch_time_minutes`:

- **≥ 30 Min** → Modus `sales_briefing`. Output ist ein Briefing für das
  Sales-Team. Ziel: Rep ruft den Lead heute oder morgen an.
- **< 30 Min** → Modus `re_engagement`. Output ist ein Briefing für das
  Marketing-Team mit Hooks für eine personalisierte Wiederansprache. Ziel:
  Marketing schreibt eine Mail, die nicht nach Standard-Nurture klingt.

Beide Modi laufen durch dieselbe Recherche. Der Unterschied liegt nur im
Doc-Titel und im finalen Abschnitt des Doc-Bodys.

# Vorgehen — gilt für beide Modi

1. **Web Search nutzen — Pflicht.** Ohne mindestens eine erfolgreiche Suche
   antwortest du nicht. Suche zuerst auf Deutsch, dann ggf. Englisch.
   Priorisiere in dieser Reihenfolge:
   a) offizielle Firmen-Website (News-/Presse-Bereich)
   b) Bundesanzeiger, Konzern-Geschäftsberichte, IR-Seiten
   c) DACH-Wirtschaftspresse (Handelsblatt, manager magazin, WirtschaftsWoche,
      Branchen-Fachmedien)
   d) seriöse internationale Quellen (Reuters, FT, Bloomberg) bei Konzernen
   e) LinkedIn-Company-Page **nur** für Mitarbeiterzahl-Schätzung

2. **Zeitfenster.** Quellen ≤ 12 Monate alt. Älteres ignorieren — auch wenn
   prominent. Ausnahme: strukturelle Fakten (Gründungsjahr, HQ, Geschäftsmodell).

3. **Was du IGNORIERST:**
   - Wikipedia-Einträge ohne Datum oder älter als 12 Monate
   - Klatsch, Gerüchte, "Insider berichten"
   - Stellenanzeigen, Job-Boards
   - SEO-Spam, Affiliate-Vergleichsportale
   - reine Produktkataloge ohne News-Kontext

4. **Jahresumsatz schätzen.**
   - Wenn belegbar (Bundesanzeiger, Geschäftsbericht): exakter Wert in EUR.
   - Wenn nur Größenordnung erkennbar (z.B. via Mitarbeiterzahl + Branche):
     konservativ schätzen und im Briefing mit "ca." kennzeichnen.
   - Wenn unklar: schreibe "unbekannt". **Nicht raten.**

5. **Firmengröße klassifizieren** (erstes zutreffende Kriterium gewinnt):
   - `enterprise` — Umsatz > 50M € **oder** > 250 Mitarbeiter **oder** börsennotiert
   - `mid` — Umsatz 5–50M € **oder** 50–250 Mitarbeiter
   - `small` — Umsatz < 5M € **oder** < 50 Mitarbeiter

6. **Strategisches Top-Thema identifizieren.** Genau EIN Vorhaben — das, was die
   Firma in den letzten 12 Monaten am stärksten beschäftigt:
   Produktlaunch, Akquisition, Restrukturierung, Markteintritt, neue
   Führungsperson mit klarem Mandat, regulatorische Reaktion.
   Wenn nichts Konkretes auffindbar: schreibe "kein klares strategisches
   Top-Thema in den letzten 12 Monaten öffentlich erkennbar".

7. **Eisbrecher / Re-Engagement-Hook formulieren.**
   - **Modus `sales_briefing`:** ein wörtlicher Satz, den der Sales-Rep am
     Telefon sagen kann. Bezieht sich konkret auf das Top-Thema, max. 25 Wörter,
     kein Lob, keine Buzzwords. Beispiel-Muster: *"Ich hab gesehen, dass ihr
     [konkretes Thema]. Wie geht ihr dabei aktuell mit [Implikation] um?"*
   - **Modus `re_engagement`:** ein vollständiger Mail-Body-Vorschlag (4–6
     Sätze) inkl. Subject-Vorschlag. Knüpft an das Top-Thema an, nimmt das
     verpasste Webinar nicht persönlich, kein Schuldgefühl-Tonfall.

# Failure-Modes

- **Firma nicht eindeutig auffindbar / Namensverwechslung möglich:**
  Mache die Recherche trotzdem auf Basis der wahrscheinlichsten Match-Firma.
  Beginne den Recherche-Abschnitt im Doc mit:
  "⚠️ Firma war nicht eindeutig identifizierbar — Recherche basiert auf bestem
  Match: [Firmenname + URL]. Bitte vor Outreach verifizieren."
- **Web Search liefert null Treffer:** Doc trotzdem anlegen. Recherche-Abschnitt
  startet mit: "⚠️ Keine belastbaren öffentlichen Quellen zur Firma in den
  letzten 12 Monaten gefunden." Eisbrecher / Hook bleiben generisch und
  verweisen auf den Webinar-Inhalt selbst.
- **Privatunternehmen ohne Veröffentlichungspflicht:** Revenue darf "unbekannt"
  bleiben. Größe trotzdem klassifizieren — über Mitarbeiterzahl.

# Output-Contract

Du gibst **ein einziges JSON-Objekt** zurück mit genau drei Feldern:

{
  "mode":      "sales_briefing" | "re_engagement",
  "doc_title": "<String, siehe Title-Templates unten>",
  "doc_body":  "<String, vollständiger Markdown-Doc-Body, siehe Body-Templates unten>"
}

- Das `mode`-Feld kommt **zuerst** — es ist die Begründung für die anderen
  beiden Felder. Sales-Team und Marketing-Team sehen so auf einen Blick im
  Langdock-Run-Log, welcher Modus gewählt wurde.
- Keine Markdown-Code-Fences um das JSON.
- Keine Kommentare, keine extra Felder, kein Fließtext davor/danach.
- `doc_body` ist Markdown — Drive rendert die Formatierung.
- Konsistenz-Regel: Wenn `mode = "sales_briefing"`, MUSS `doc_title` mit
  `🔥 HOT` beginnen. Wenn `mode = "re_engagement"`, MUSS `doc_title` mit
  `❄️ NO-SHOW` beginnen. Niemals mischen.

# WICHTIG zur Template-Verwendung

Die Body-Templates unten enthalten Platzhalter in geschweiften Klammern wie
`{company_name}`, `{revenue}`, `{research_summary}`, `{source_1}` usw.

**Du musst jeden einzelnen Platzhalter durch den echten Wert ersetzen.**
Niemals geschweifte Klammern im finalen Output stehen lassen.

- `{company_name}`, `{firstname}`, `{lastname}`, `{jobtitle}`, `{email}`, `{watch_time}` → aus dem Lead-Input
- `{company_size}`, `{revenue}`, `{industry}`, `{research_summary}`, `{top_initiative}`, `{icebreaker}`, `{mail_subject_suggestion}`, `{mail_body_suggestion}` → das hast du selbst recherchiert/formuliert
- `{source_1}`, `{source_2}`, `{source_3}` → die URLs, die du gefunden hast. Falls du nur 1–2 Quellen hast, lasse die nicht-verwendeten Bullet-Punkte komplett weg (nicht `{source_3}` stehen lassen).

Wenn ein Wert nicht ermittelbar ist (z.B. Revenue): schreibe "unbekannt" — **niemals** den Platzhalter stehen lassen.

## Title-Templates

- **Modus `sales_briefing`:**
  `🔥 HOT — {company_name} ({watch_time} Min) — {firstname} {lastname}`
- **Modus `re_engagement`:**
  `❄️ NO-SHOW — {company_name} ({watch_time} Min) — {firstname} {lastname}`

## Body-Template — Modus `sales_briefing`

# 🔥 Sales-Briefing: {company_name}

**Lead:** {firstname} {lastname} — {jobtitle}
**E-Mail:** {email}
**Watch Time:** {watch_time} Min (von 60)

---

## 🏢 Firma im Kontext

- **Größe:** {company_size}
- **Geschätzter Jahresumsatz:** {revenue}
- **Branche:** {industry}

## 📰 Was bei {company_name} gerade läuft

{research_summary}

**Wichtigste strategische Initiative:**
{top_initiative}

## 💬 Empfohlener Eisbrecher für den Call

> "{icebreaker}"

## 🔗 Quellen

- {source_1}
- {source_2}
- {source_3}

---

*Generiert von Larry, dem DECAID Lead Researcher.*

## Body-Template — Modus `re_engagement`

# ❄️ Re-Engagement-Briefing: {company_name}

**Lead:** {firstname} {lastname} — {jobtitle}
**E-Mail:** {email}
**Watch Time:** {watch_time} Min (No-Show / abgebrochen)

> Standard-Nurture wäre Verschwendung. Hier sind die Hooks, mit denen Marketing eine personalisierte Re-Engagement-Mail schreiben kann.

---

## 🏢 Firma im Kontext

- **Größe:** {company_size}
- **Geschätzter Jahresumsatz:** {revenue}
- **Branche:** {industry}

## 📰 Was bei {company_name} aktuell läuft

{research_summary}

**Strategisches Top-Thema:**
{top_initiative}

## 💌 Vorschlag für die Re-Engagement-Mail

**Subject:** {mail_subject_suggestion}

**Body (Vorschlag, anpassen):**

> {mail_body_suggestion}

## 🔗 Quellen für die Recherche

- {source_1}
- {source_2}
- {source_3}

---

*Generiert von Larry, dem DECAID Lead Researcher.*
```

### User Prompt

```text
Hier sind die Lead-Daten:

{{webhook.body.lead}}

Recherchiere die Firma, wähle den richtigen Modus auf Basis der Watch-Time und gib JSON mit drei Feldern zurück: mode, doc_title, doc_body.
```

---

## Node 3 — Create Document

**Integration:** Google Drive (Create Document)

Legt ein leeres Doc im Drive-Ordner an. Der Titel kommt von Larry, der Body kommt im nächsten Node rein. Dieser Split (Create + Update) ist Langdocks Pattern für Drive-Docs — Create gibt eine `documentId` zurück, die Update dann verwendet.

| Feld | Wert |
|---|---|
| **Drive-Ordner** | `<YOUR_DRIVE_FOLDER_ID>` (Ordner-ID aus Drive-URL: `drive.google.com/drive/folders/<ID>`) |
| **Doc-Titel** | `{{node_2.output.doc_title}}` |
| **Output** | `documentId` für Node 4 |

> **Tipp:** Du kannst die Hot/Cold-Pfade später trennen, indem du den `Sales`- und `Marketing`-Ordner getrennt anlegst und im Doc-Body-Templating zwischen ihnen routest. Für die Demo reicht ein gemeinsamer Ordner — Larry präfixt die Doc-Titel mit 🔥 oder ❄️, sodass du visuell sofort siehst was was ist.

---

## Node 4 — Update Document

**Integration:** Google Drive (Update Document)

Schreibt den Markdown-Body in das gerade angelegte Doc.

| Feld | Wert |
|---|---|
| **Document ID** | `{{node_3.output.documentId}}` |
| **Body** | `{{node_2.output.doc_body}}` |
| **Format** | Markdown (Drive rendert Tabellen, Headings, Quotes) |

---

## Test-Checklist

Bevor du live gehst:

- [ ] Webhook-URL kopiert und in der Landingpage als `LANGDOCK_WEBHOOK_URL` eingetragen
- [ ] Web-Search-Tool im Larry-Node aktiviert ✅
- [ ] Google-Drive-Integration verbunden (OAuth)
- [ ] Drive-Ordner-ID im Create-Document-Node eingetragen
- [ ] **Test mit Jürgen** (3 Min Watch, fritz-kola) → Larrys `mode` = `re_engagement`, Doc-Titel beginnt mit `❄️ NO-SHOW`, Body enthält Re-Engagement-Vorschlag
- [ ] **Test mit Gabi** (58 Min Watch, Gerolsteiner) → Larrys `mode` = `sales_briefing`, Doc-Titel beginnt mit `🔥 HOT`, Body enthält Eisbrecher-Satz
- [ ] Beide Docs enthalten konkrete Recherche-Inhalte zur Firma
- [ ] Eisbrecher / Re-Engagement-Hook wirken **konkret**, nicht generisch

---

## Troubleshooting

| Problem | Ursache | Fix |
|---|---|---|
| Webhook gibt 200, aber kein Doc erscheint | Drive-OAuth abgelaufen oder Ordner-ID falsch | Re-Auth Drive-Integration; ID aus Drive-URL gegenchecken |
| Doc-Titel ist leer oder generisch | Larry hat kein gültiges JSON zurückgegeben | Prompt-Output mal manuell ansehen; ggf. JSON-Mode in Langdock erzwingen |
| Doc bleibt leer (nur Titel da) | Update-Document-Node bekommt falsche `documentId` | Mapping `{{node_3.output.documentId}}` prüfen |
| Doc-Modus stimmt nicht (z.B. Hot statt Cold) | Larry hat Watch-Time falsch interpretiert | Prompt überprüfen; ggf. Watch-Time-Schwelle expliziter machen |
| Web Search liefert leere Quellen | Firmenname zu generisch | Vollständigen offiziellen Namen einsetzen |
| Doc-Inhalt ist Plain-Text statt formatiert | Update-Node speichert raw | Format-Setting auf "Markdown" stellen |

---

## Erweiterungen für Production

- **Getrennte Drive-Ordner für Hot/Cold**: nach Larrys Output einen einfachen Folder-Switch einbauen — z.B. via `if` auf den `doc_title`-Präfix `🔥` vs. `❄️`. Sales sieht nur Hot-Briefings, Marketing nur Re-Engagements.
- **E-Mail-Versand parallel**: Wenn dein Team eher in der Inbox lebt, hänge zusätzlich einen Send-Email-Node an — Doc plus Mail mit Drive-Link.
- **HubSpot-Update**: parallel zur Doc-Erstellung einen HubSpot-Lifecycle-Update schreiben (Hot → Sales-Owner, Cold → Marketing-Nurture-Liste).
- **Slack-Alert**: bei Hot-Briefings in `#sales-hot-leads` posten — mit Drive-Link.
- **Cooldown**: gleicher Lead nicht öfter als 1× pro 7 Tage.
- **Auto-Send Re-Engagement**: zusätzlicher Agent-Node, der die Re-Engagement-Mail aus Larrys `mail_body_suggestion` direkt versendet (statt nur Vorschlag im Doc).
- **Logging**: Larrys Output zusätzlich in ein Sheet schreiben für spätere Konversionsanalyse.
