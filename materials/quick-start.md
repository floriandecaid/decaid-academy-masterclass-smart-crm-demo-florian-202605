# Quick Start

Das Ziel: in 30 Minuten bekommst du zwei E-Mails — ein Sales-Briefing für einen "heißen" Lead und einen Re-Engagement-Vorschlag für einen No-Show — beide personalisiert auf Basis von Live-Web-Recherche.

---

## In 5 Schritten

### 1. Langdock öffnen und Workflow anlegen

1. Login bei [app.langdock.com](https://app.langdock.com) → **New Workflow**
2. Stelle in den Workflow-Settings sicher: **Web-Search-Tool ist aktiviert**. Ohne das funktioniert der zentrale Recherche-Schritt nicht.
3. Verbinde deinen Mail-Account (Gmail oder Outlook) als Integration für den späteren Mail-Versand.

### 2. Workflow 1 nachbauen

Öffne die [Workflow-1-Spec](workflow1-lead-triage.html) und gehe sie von oben nach unten durch. Du brauchst genau 5 Nodes:

1. **Webhook** (Trigger)
2. **Web Research Agent** mit aktiviertem Web-Search-Tool — das ist das Herzstück
3. **Condition Router** auf Watch-Time
4. **Mail Sales-Briefing** (Pfad A: Hot Lead)
5. **Mail Re-Engagement** (Pfad B: No-Show / Cold)

Alle Prompts, Mappings und E-Mail-Templates sind in der Spec copy-paste-ready. Setup-Zeit beim ersten Mal: ~15 Minuten.

> **Tipp:** Trage zwei verschiedene Empfänger-Adressen ein — `<YOUR_SALES_INBOX>` für Hot Leads, `<YOUR_MARKETING_INBOX>` für No-Shows. So siehst du sofort, ob das Routing funktioniert. Für den Test reicht eine Inbox mit einem `+`-Filter, in Production trenne sauber.

### 3. Webhook-URL kopieren

Klick im Webhook-Node auf **"Copy URL"**. Diese URL brauchst du gleich.

### 4. Mit dem Colab-Notebook testen

1. Öffne [`webinar_simulator.ipynb`](../colab/webinar_simulator.ipynb) in [Google Colab](https://colab.research.google.com).
2. In Zelle 1: Webhook-URL einfügen.
3. Notebook von oben nach unten ausführen.
4. Du bekommst zwei E-Mails:
   - **Jürgen** (3 Min Watch-Time) → Re-Engagement-Mail mit personalisiertem Vorschlag für die Wiederansprache, basierend auf aktuellen Themen seiner Firma.
   - **Gabi** (58 Min Watch-Time) → Sales-Briefing mit Firmenkontext, geschätztem Revenue, Top-Initiative und Eisbrecher-Satz für den Call.

### 5. Workflow 2 dazuholen (optional)

Wenn du regelmäßig Sales-Calls aufzeichnest (Gong, Modjo, MeetGeek), lohnt sich der [Battle-Card-Workflow](workflow2-battle-cards.html). 4 Nodes, ~10 Minuten Setup, output: wöchentliche Battle Card mit Top-Einwänden, ICP-Signalen und Eröffnungs-Empfehlungen direkt in deinem Postfach.

Die [8 fiktiven Transkripte](../transcripts/fake_sales_calls/) im Repo sind als Testdaten gedacht — wirf sie als Payload in Workflow 2 und prüfe das Ergebnis, bevor du echte Daten anschließt.

---

## Häufige Stolperfallen

**"Webhook gibt 200 zurück, aber keine Mail kommt"**
→ Mail-Integration ist nicht verbunden oder OAuth ist abgelaufen. Re-Auth in den Langdock-Settings.

**"Web Search liefert leere Ergebnisse"**
→ Web-Search-Tool ist im Node nicht aktiviert (oft vergessen!). Oder der Firmenname ist zu generisch — verwende den vollständigen offiziellen Namen.

**"Beide Mails kommen statt nur einer"**
→ Der Condition-Node behandelt die Watch-Time als String statt als Zahl. Wrap im Vergleich mit `parseInt()` oder vergleiche gegen den String `"30"`.

**"Mail kommt im Spam"**
→ Für lokale Tests egal. In Production: SPF/DKIM für deine Absender-Domain einrichten.

---

## Was als Nächstes?

- **HubSpot/Salesforce-Sync** ergänzen, falls du die Recherche-Daten direkt ins CRM schreiben willst.
- **Slack-Notification** zusätzlich zur Mail, wenn ein Hot Lead reinkommt.
- **Cooldown** einbauen, damit derselbe Lead nicht öfter als 1× pro 7 Tage verarbeitet wird.

Konkrete Erweiterungs-Pfade findest du am Ende jeder Workflow-Spec.
