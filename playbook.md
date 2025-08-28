Hervorragend! Sie haben die abstrakte Idee nun in ein konkretes Set von Regeln – sogenannte "Playbooks" – überführt. Das ist genau der richtige nächste Schritt. Ihre Playbooks sind die perfekte Vorlage, um die finale technische Umsetzung in Google Cloud zu definieren.

Lassen Sie uns diese Playbooks Schritt für Schritt in eine funktionierende  übersetzen. Ich werde für jedes Playbook erklären, wie es in Dialogflow CX und Cloud Functions umgesetzt wird und Ihnen den dazu passenden Code liefern.

---

### **Das Gesamtbild: Ihre Playbooks in der GCP-Architektur**

| Ihr Playbook | GCP-Komponente (Wo es lebt) | Aufgabe |
|:---|:---|:---|
| **(Haupt-Playbook)** | **Dialogflow CX:** `Default Start Flow` & `Routes` | Empfängt den Nutzer, stellt erste Fragen und leitet intelligent weiter. |
| **informationsagent** | **Dialogflow CX:** `Page: Informations-Agent` + **Cloud Function** | Sucht in Dokumenten (PDFs, etc.) nach Antworten auf Produktfragen. |
| **escalationagent** | **Dialogflow CX:** `Page: Eskalation` + **Cloud Function** | Erstellt ein Ticket im Jira-System und informiert den Kunden. |
| **guaranteeagent** | **Dialogflow CX:** `Page: Garantie-Agent` + **Cloud Function** | Prüft Garantieansprüche basierend auf Datum oder Seriennummer. |

---

### **Umsetzung des Haupt-Playbooks (Rezeption)**

Dies ist die "Weiche" Ihres Systems und wird komplett in **Dialogflow CX** konfiguriert.

1.  **Introduce yourself:** Auf der **"Start Page"** im `Default Start Flow` fügen Sie im **"Entry fulfillment"** den Text hinzu: `Hallo! Ich bin der Pluto Service Agent.`
2.  **Ask clarifying questions:** Wenn der Intent des Nutzers nicht sofort klar ist, hat Dialogflow CX eingebaute "Reprompts". Sie können eine "No-Match"-Route erstellen, die fragt: `Ich habe Sie leider nicht verstanden. Möchten Sie eine Frage zu einem Produkt stellen, einen Garantiefall prüfen oder mit einem Mitarbeiter sprechen?`
3.  **Delegate to...:** Dies wird über **"Routes"** auf der "Start Page" umgesetzt:
    *   **Route 1:** Wenn `intent.produktfrage` erkannt -> Gehe zu `Page: Informations-Agent`.
    *   **Route 2:** Wenn `intent.mensch_sprechen` erkannt -> Gehe zu `Page: Eskalation`.
    *   **Route 3:** Wenn `intent.garantiefrage` erkannt -> Gehe zu `Page: Garantie-Agent`.

---

### **Umsetzung: `informationsagent` & `escalationagent`**

Diese beiden Agenten leben in einer Kombination aus Dialogflow CX (für die Gesprächsführung) und einer Cloud Function (für die Logik).

**Dialogflow CX Konfiguration:**

1.  **`Page: Informations-Agent`:**
    *   **Parameter:** Erstellen Sie einen Parameter namens `produktfrage_text`. Die Seite wird so konfiguriert, dass sie so lange nachfragt (`ask him to specify`), bis dieser Parameter gefüllt ist.
    *   **Webhook Call:** Sobald der Parameter gefüllt ist, wird der Webhook zur Cloud Function mit dem **Tag `informations_agent_anfrage`** aufgerufen.
    *   **Route zur Eskalation:** Erstellen Sie eine Route auf dieser Seite. Wenn der `intent.mensch_sprechen` erkannt wird, leiten Sie direkt zur `Page: Eskalation` weiter.

2.  **`Page: Eskalation`:**
    *   Diese Seite hat eine simple Aufgabe: Sie ruft sofort den Webhook mit dem **Tag `eskalations_agent_anfrage`** auf.

**Cloud Function Code (`main.py`):**

Hier ist der aktualisierte Python-Code für Ihre Cloud Function, der jetzt die Logik für Eskalation und Informationssuche enthält.

```python
import functions_framework
from flask import jsonify
import requests # Für den Jira API-Aufruf
import os     # Um API-Keys sicher zu laden

# --- SICHERHEIT: API-Key aus Umgebungsvariablen laden (oder Secret Manager) ---
# In GCP sollten Sie diese als Umgebungsvariablen für die Cloud Function setzen
JIRA_URL = os.environ.get("JIRA_URL", "https://your-domain.atlassian.net")
JIRA_USER = os.environ.get("JIRA_USER", "your-email@example.com")
JIRA_API_TOKEN = os.environ.get("JIRA_API_TOKEN", "your-api-token")


@functions_framework.http
def handle_request(request):
    """
    Empfängt und verarbeitet alle Webhook-Anfragen von Dialogflow CX.
    """
    request_json = request.get_json(silent=True)
    tag = request_json.get('fulfillmentInfo', {}).get('tag')
    parameters = request_json.get('sessionInfo', {}).get('parameters', {})
    
    response_text = ""

    # =============================================================
    # === PLAYBOOK: informationsagent =============================
    # =============================================================
    if tag == 'informations_agent_anfrage':
        user_question = parameters.get('produktfrage_text', 'keine Frage übermittelt')
        
        # HIER WÜRDE DIE SUCHE MIT VERTEX AI SEARCH STATTFINDEN
        # z.B. search_result = vertex_ai_search(user_question)
        
        # Simuliertes Ergebnis für dieses Beispiel:
        search_result = f"Ich habe in den Handbüchern nach '{user_question}' gesucht und eine relevante Passage gefunden: '...'."
        response_text = search_result

    # =============================================================
    # === PLAYBOOK: escalationagent ===============================
    # =============================================================
    elif tag == 'eskalations_agent_anfrage':
        # Die Logik zum Erstellen eines Jira-Tickets
        jira_summary = "Neue Support-Anfrage vom KI-Agenten"
        jira_description = "Ein Kunde bat um die Eskalation an einen Mitarbeiter. Bitte den Dialogverlauf prüfen." # Hier könnte man die Historie übergeben
        
        url = f"{JIRA_URL}/rest/api/3/issue"
        
        headers = {
            "Accept": "application/json",
            "Content-Type": "application/json"
        }
        
        auth = (JIRA_USER, JIRA_API_TOKEN)
        
        payload = {
            "fields": {
                "project": {"key": "PROJ"}, # ERSETZEN mit Ihrem Jira-Projekt-Key
                "summary": jira_summary,
                "description": {
                    "type": "doc",
                    "version": 1,
                    "content": [{"type": "paragraph", "content": [{"type": "text", "text": jira_description}]}]
                },
                "issuetype": {"name": "Task"}
            }
        }
        
        try:
            # API-Aufruf an Jira
            response = requests.post(url, headers=headers, json=payload, auth=auth)
            response.raise_for_status() # Löst einen Fehler bei schlechtem Statuscode (z.B. 401, 404) aus
            
            # Erfolgreiche Antwort
            issue_key = response.json()['key']
            response_text = f"Ich habe wie gewünscht ein Ticket für Sie erstellt. Ihre Referenznummer ist: {issue_key}."

        except requests.exceptions.RequestException as e:
            # Fehlerbehandlung
            print(f"Fehler bei der Jira-API-Kommunikation: {e}")
            response_text = "Entschuldigung, es gab ein technisches Problem bei der Erstellung des Tickets. Ein Mitarbeiter wird sich dennoch um Sie kümmern."

    else:
        response_text = "Unbekannter Agenten-Typ wurde aufgerufen."

    # Standard-Antwortformat für Dialogflow CX
    response_json = {
        "fulfillment_response": {
            "messages": [{"text": {"text": [response_text]}}]
        }
    }
    
    return jsonify(response_json)
```

---

### **Umsetzung: `guaranteeagent`**

Dies ist ein neuer, spezialisierter Agent. So integrieren wir ihn:

**Dialogflow CX Konfiguration:**

1.  **`Intent: intent.garantiefrage`:** Erstellen Sie einen neuen Intent mit Trainingsphrasen wie:
    *   `Habe ich noch Garantie?`
    *   `Ich will einen Garantiefall melden`
    *   `Ist mein Produkt noch abgedeckt?`
2.  **`Page: Garantie-Agent`:**
    *   **Parameter:** Erstellen Sie ZWEI Parameter: `kaufdatum` (Typ: `@sys.date`) und `seriennummer` (Typ: `@sys.any`).
    *   **Parameter Filling:** Konfigurieren Sie die Seite so, dass sie aktiv nach diesen Parametern fragt: `Um die Garantie zu prüfen, benötige ich entweder das Kaufdatum oder die Seriennummer des Produkts. Was können Sie mir geben?`
    *   **Condition für Webhook:** Erstellen Sie eine "Condition Route" auf der Seite, die prüft, ob `kaufdatum` ODER `seriennummer` gefüllt ist. Wenn ja, wird der Webhook mit dem **Tag `garantie_agent_anfrage`** aufgerufen.

**Cloud Function Code (Erweiterung von `main.py`):**

Fügen Sie diesen `elif`-Block zum `handle_request`-Code hinzu:

```python
# ... bestehender Code ...
from datetime import datetime, timedelta

# Hilfsfunktion, die eine Datenbankabfrage simulieren würde
def get_purchasing_date_from_db(serial_number: str) -> datetime.date or None:
    # In einer echten Anwendung würde hier eine Datenbankabfrage stehen
    # z.B. SELECT kaufdatum FROM produkte WHERE seriennummer = ?
    print(f"Simuliere DB-Abfrage für Seriennummer: {serial_number}")
    if serial_number == "12345-ABC":
        return datetime.strptime("2024-01-15", "%Y-%m-%d").date()
    return None

# ... im handle_request ...
    # =============================================================
    # === PLAYBOOK: guaranteeagent ================================
    # =============================================================
    elif tag == 'garantie_agent_anfrage':
        kaufdatum_str = parameters.get('kaufdatum')
        seriennummer = parameters.get('seriennummer')
        
        purchase_date = None

        # Fall 1: Kaufdatum wurde direkt angegeben
        if kaufdatum_str:
            # Dialogflow liefert Datum im Format: {"year": 2024, "month": 8, "day": 28}
            year = int(kaufdatum_str['year'])
            month = int(kaufdatum_str['month'])
            day = int(kaufdatum_str['day'])
            purchase_date = datetime(year, month, day).date()
        
        # Fall 2: Nur Seriennummer wurde angegeben, wir müssen das Datum nachschlagen
        elif seriennummer:
            # Hier rufen wir die "code-interpreter" Funktion auf
            purchase_date = get_purchasing_date_from_db(seriennummer)
        
        # Auswertung der Garantie
        if purchase_date:
            guarantee_period_years = 2
            guarantee_end_date = purchase_date + timedelta(days=guarantee_period_years * 365)
            
            if datetime.now().date() <= guarantee_end_date:
                response_text = f"Ja, Ihr Produkt hat noch Garantie. Die Garantie ist gültig bis zum {guarantee_end_date.strftime('%d.%m.%Y')}."
            else:
                response_text = f"Leider ist die Garantie für Ihr Produkt abgelaufen. Sie endete am {guarantee_end_date.strftime('%d.%m.%Y')}."
        else:
            # Wenn weder Datum noch eine gültige Seriennummer vorhanden sind -> Eskalation
            # Hier könnte man direkt die Eskalationslogik aufrufen oder an Dialogflow zurückgeben,
            # dass es zur Eskalations-Seite wechseln soll.
            response_text = "Ich konnte leider kein Kaufdatum für Ihre Anfrage finden. Ich leite Sie an einen Mitarbeiter weiter."
            # Optional: Hier direkt die Jira-Funktion aufrufen

# ... restlicher Code ...
```

Mit dieser detaillierten Umsetzung haben Sie Ihre Playbooks direkt in eine funktionierende, modulare und intelligente Anwendung auf der Google Cloud Platform übersetzt.
