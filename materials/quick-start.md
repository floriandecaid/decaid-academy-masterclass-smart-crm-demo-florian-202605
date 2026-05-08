# Quick Start

Das Ziel: in 30 Minuten bekommst du zwei Google Docs — ein Sales-Briefing für einen "heißen" Lead und einen Re-Engagement-Vorschlag für einen No-Show — beide personalisiert auf Basis von Live-Web-Recherche.

---

## In 5 Schritten

### 1. Langdock öffnen und Workflow anlegen

1. Login bei [app.langdock.com](https://app.langdock.com) → **New Workflow**
2. Stelle in den Workflow-Settings sicher: **Web-Search-Tool ist aktiviert**. Ohne das funktioniert der zentrale Recherche-Schritt nicht.
3. Verbinde **Google Drive** als Integration (OAuth-Flow durchgehen).

### 2. Drive-Ordner anlegen

In deinem Google Drive einen Ordner erstellen (z.B. `DECAID Lead Briefings`). Larry präfixt die Doc-Titel mit 🔥 (Hot) oder ❄️ (No-Show), sodass du visuell sofort siehst was was ist. Hot/Cold-Ordner-Trennung kannst du später einbauen.

Ordner-ID aus der URL kopieren (`drive.google.com/drive/folders/<DAS_HIER>`). Brauchst du gleich.

### 3. Workflow 1 nachbauen

Öffne die [Workflow-1-Spec](workflow1-lead-triage.html) und gehe sie von oben nach unten durch. Du brauchst genau 4 Nodes:

1. **Webhook** (Trigger)
2. **Larry the Lead Researcher** — Agent mit aktiviertem Web-Search-Tool. Routet im Prompt selbst zwischen Hot- (Sales-Briefing) und Cold-Briefing (Re-Engagement).
3. **Create Document** im Drive-Ordner — legt das leere Doc mit Larrys Titel an
4. **Update Document** — schreibt Larrys Markdown-Body ins Doc

Alle Prompts, Mappings und Doc-Templates sind in der Spec copy-paste-ready. Setup-Zeit beim ersten Mal: ~10 Minuten.

### 4. Webhook-URL kopieren

In Langdock auf den **Webhook-Node** klicken (ganz links im Workflow). Im Detail-Panel rechts erscheint ein Button **"Copy URL"** — Klick.

Du bekommst eine URL der Form `https://app.langdock.com/api/hooks/workflows/<workflow-id>`.

**Diese URL trägst du an zwei Stellen ein, je nachdem wie du triggern willst:**

- **Browser-Trigger via Landingpage** → in [`index.html`](../index.html), Variable `LANGDOCK_WEBHOOK_URL` (im `<script>`-Block am Ende). Demo-Buttons funktionieren dann.
- **Battle-Cards-Demo** → in [`battle-cards.html`](../battle-cards.html), Variable `LANGDOCK_W2_WEBHOOK_URL` (zweiter Workflow, anderer Webhook).
- **Eigener Trigger** (curl, Python, dein CRM-Webhook) → POST-Endpoint mit JSON-Body wie in der [Workflow-Spec](workflow1-lead-triage.html) beschrieben.

### 5. Testen

Öffne die Landingpage in deinem Browser, klick auf einen der zwei Demo-Buttons:

- **Jürgen** (3 Min Watch-Time) → Re-Engagement-Doc mit ❄️-Präfix landet in deinem Drive-Ordner.
- **Gabi** (58 Min Watch-Time) → Sales-Briefing-Doc mit 🔥-Präfix landet im selben Ordner.

Drive-Ordner offen halten, "Sortieren nach: Zuletzt geändert" — die neuen Docs poppen oben rein.

> **Alternative zum Browser-Trigger:** Im Repo liegt ein [Colab-Notebook](../colab/webinar_simulator.ipynb), falls du die Pipeline aus Python anstoßen willst (z.B. um sie in dein Backend zu integrieren). Für die normale Nutzung reicht aber die Landingpage.

### 6. Workflow 2 dazuholen (optional)

Wenn du regelmäßig Sales-Calls aufzeichnest (Gong, Modjo, MeetGeek), lohnt sich der [Battle-Card-Workflow](workflow2-battle-cards.html). 4 Nodes, ~10 Minuten Setup, Output: wöchentliche Battle Card als Doc im Sales-Ordner.

Die [8 fiktiven Transkripte](../transcripts/fake_sales_calls/) im Repo sind als Testdaten gedacht — wirf sie als Payload in Workflow 2 und prüfe das Ergebnis, bevor du echte Daten anschließt.

---

## Häufige Stolperfallen

**"Webhook gibt 200 zurück, aber kein Doc erscheint"**
→ Drive-Integration ist nicht verbunden oder OAuth ist abgelaufen. Re-Auth in den Langdock-Settings.

**"Doc-Titel ist leer oder Doc-Body fehlt"**
→ Larry hat kein gültiges JSON zurückgegeben. Schau in die Run-Logs, ob der Output im Format `{ "mode": "...", "doc_title": "...", "doc_body": "..." }` kommt. Falls nicht: JSON-Output-Mode in Langdock erzwingen.

**"Doc-Modus stimmt nicht (z.B. Hot statt Cold)"**
→ Larry hat die Watch-Time falsch interpretiert. Prompt anpassen, ggf. die 30-Min-Schwelle expliziter machen oder mit Beispielen verstärken.

**"Web Search liefert leere Ergebnisse"**
→ Web-Search-Tool ist im Larry-Node nicht aktiviert (oft vergessen!). Oder der Firmenname ist zu generisch — verwende den vollständigen offiziellen Namen.

**"Doc-Inhalt ist Plain-Text statt formatiert"**
→ Update-Document-Node speichert raw. Format-Setting auf "Markdown" stellen.

---

## Was als Nächstes?

- **Slack-Notification** zusätzlich zum Doc, wenn ein Hot Lead reinkommt — mit Drive-Link.
- **HubSpot/Salesforce-Sync**, falls du die Recherche-Daten direkt ins CRM schreiben willst.
- **Cooldown** einbauen, damit derselbe Lead nicht öfter als 1× pro 7 Tage verarbeitet wird.

Konkrete Erweiterungs-Pfade findest du am Ende jeder Workflow-Spec.
