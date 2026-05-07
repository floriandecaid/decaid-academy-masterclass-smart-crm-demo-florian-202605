# Quick Start

Das Ziel: in 30 Minuten bekommst du zwei Google Docs — ein Sales-Briefing für einen "heißen" Lead und einen Re-Engagement-Vorschlag für einen No-Show — beide personalisiert auf Basis von Live-Web-Recherche.

---

## In 5 Schritten

### 1. Langdock öffnen und Workflow anlegen

1. Login bei [app.langdock.com](https://app.langdock.com) → **New Workflow**
2. Stelle in den Workflow-Settings sicher: **Web-Search-Tool ist aktiviert**. Ohne das funktioniert der zentrale Recherche-Schritt nicht.
3. Verbinde **Google Drive** als Integration (OAuth-Flow durchgehen).

### 2. Drive-Ordner anlegen

In deinem Google Drive zwei Ordner erstellen (oder einen, wenn du es einfacher willst):

- `Sales – Hot Leads` → für Briefings nach Workflow-1-Pfad-A
- `Marketing – Re-Engagement` → für Vorschläge nach Workflow-1-Pfad-B

Ordner-IDs aus den URLs kopieren (`drive.google.com/drive/folders/<DAS_HIER>`). Brauchst du gleich.

### 3. Workflow 1 nachbauen

Öffne die [Workflow-1-Spec](workflow1-lead-triage.html) und gehe sie von oben nach unten durch. Du brauchst genau 5 Nodes:

1. **Webhook** (Trigger)
2. **Web Research Agent** mit aktiviertem Web-Search-Tool — das ist das Herzstück
3. **Condition Router** auf Watch-Time
4. **Create Google Doc** im Sales-Ordner (Pfad A: Hot Lead)
5. **Create Google Doc** im Marketing-Ordner (Pfad B: No-Show / Cold)

Alle Prompts, Mappings und Doc-Templates sind in der Spec copy-paste-ready. Setup-Zeit beim ersten Mal: ~15 Minuten.

### 4. Webhook-URL kopieren

Klick im Webhook-Node auf **"Copy URL"**. Diese URL trägst du in die Landingpage (`index.html`, Variable `LANGDOCK_WEBHOOK_URL`) oder in dein eigenes Trigger-Setup ein.

### 5. Testen

Öffne die Landingpage in deinem Browser, klick auf einen der zwei Demo-Buttons:

- **Jürgen** (3 Min Watch-Time) → Re-Engagement-Doc landet im Marketing-Ordner.
- **Gabi** (58 Min Watch-Time) → Sales-Briefing-Doc landet im Sales-Ordner.

Drive-Ordner offen halten, "Sortieren nach: Zuletzt geändert" — die neuen Docs poppen oben rein.

> **Alternative zum Browser-Trigger:** Im Repo liegt ein [Colab-Notebook](../colab/webinar_simulator.ipynb), falls du die Pipeline aus Python anstoßen willst (z.B. um sie in dein Backend zu integrieren). Für die normale Nutzung reicht aber die Landingpage.

### 6. Workflow 2 dazuholen (optional)

Wenn du regelmäßig Sales-Calls aufzeichnest (Gong, Modjo, MeetGeek), lohnt sich der [Battle-Card-Workflow](workflow2-battle-cards.html). 4 Nodes, ~10 Minuten Setup, Output: wöchentliche Battle Card als Doc im Sales-Ordner.

Die [8 fiktiven Transkripte](../transcripts/fake_sales_calls/) im Repo sind als Testdaten gedacht — wirf sie als Payload in Workflow 2 und prüfe das Ergebnis, bevor du echte Daten anschließt.

---

## Häufige Stolperfallen

**"Webhook gibt 200 zurück, aber kein Doc erscheint"**
→ Drive-Integration ist nicht verbunden oder OAuth ist abgelaufen. Re-Auth in den Langdock-Settings.

**"Doc landet im falschen Ordner"**
→ Ordner-IDs vertauscht zwischen Pfad A und Pfad B. IDs aus Drive-URL nochmal kopieren.

**"Web Search liefert leere Ergebnisse"**
→ Web-Search-Tool ist im Node nicht aktiviert (oft vergessen!). Oder der Firmenname ist zu generisch — verwende den vollständigen offiziellen Namen.

**"Beide Docs werden erstellt statt nur eines"**
→ Der Condition-Node behandelt die Watch-Time als String statt als Zahl. Wrap im Vergleich mit `parseInt()` oder vergleiche gegen den String `"30"`.

**"Doc-Inhalt ist Plain-Text statt formatiert"**
→ Drive-Node speichert raw. Doc-Format auf "Markdown" stellen oder einen Node nutzen, der Markdown rendert.

---

## Was als Nächstes?

- **Slack-Notification** zusätzlich zum Doc, wenn ein Hot Lead reinkommt — mit Drive-Link.
- **HubSpot/Salesforce-Sync**, falls du die Recherche-Daten direkt ins CRM schreiben willst.
- **Cooldown** einbauen, damit derselbe Lead nicht öfter als 1× pro 7 Tage verarbeitet wird.

Konkrete Erweiterungs-Pfade findest du am Ende jeder Workflow-Spec.
