### **Die Rollen der Agenten im Detail**

Jeder Agent hat eine spezialisierte Rolle, um den Prozess modular und effizient zu gestalten.

| Agent | Aufgabe | Ausführliche Erklärung |
|:---|:---|:---|
| **Empfangsagent** | **Anliegen erkennen & weiterleiten** | Er ist die erste Anlaufstelle. Mittels Natural Language Understanding (NLU) analysiert er die Anfrage des Kunden, um den Kern des Anliegens (den "Intent") zu verstehen. Seine Hauptaufgabe ist es, zu entscheiden, welcher spezialisierte Agent am besten für die Bearbeitung geeignet ist, und die Anfrage dorthin weiterzuleiten. |
| **Informationsagent** | **Wissen bereitstellen** | Dieser Agent ist der Experte für die Wissensdatenbank. Er beantwortet allgemeine Fragen, indem er strukturierte Informationen aus Quellen wie **PDF-Handbüchern** und **Confluence-Artikeln** extrahiert und für den Kunden verständlich aufbereitet. Er ist ideal für FAQs und "How-to"-Anleitungen. |
| **Analyseagent** | **Technische Probleme diagnostizieren** | Wenn ein Kunde ein technisches Problem meldet (z. B. einen Fehlercode), tritt dieser Agent in Aktion. Er ist darauf trainiert, spezifische technische Daten zu interpretieren, logische Prüfungen durchzuführen und dem Kunden gezielte Lösungsschritte basierend auf den Handbüchern vorzuschlagen. |
| **Garantie-Agent** | **Garantiestatus prüfen** | Dieser Agent greift auf das CRM- oder Jira-System zu, um anhand von Seriennummer, Kaufdatum oder Rechnungsnummer den Garantiestatus eines Produkts zu verifizieren. Er informiert den Kunden, ob und wie lange seine Garantie gültig ist. |
| **Eskalationsagent** | **Entscheiden, wann ein Mensch hilft** | Eine der wichtigsten Rollen. Dieser Agent überwacht die Interaktionen. Wenn die KI-Agenten keine sichere Antwort finden (niedriger Konfidenz-Score), der Kunde explizit nach einem Menschen fragt oder Frustration zeigt, entscheidet er, die Anfrage an einen menschlichen Agenten zu übergeben. Dabei stellt er sicher, dass der gesamte bisherige Kontext mitgeliefert wird. |
| **Assistenzagent** | **Menschliche Agenten unterstützen** | Er arbeitet im Hintergrund als "Co-Pilot" für den menschlichen Mitarbeiter. Er kann auf Befehl Suchen in den Wissensdatenbanken durchführen, Lösungsvorschläge basierend auf ähnlichen, vergangenen Fällen machen und repetitive Aufgaben wie das **Erstellen eines Jira-Tickets** automatisieren. |
| **Lernagent** | **Das System kontinuierlich verbessern** | Dieser Agent ist für die Nachhaltigkeit des Systems zuständig. Er analysiert das Feedback von Kunden und die Lösungswege der menschlichen Agenten. Wenn er erkennt, dass Informationen in der Wissensdatenbank fehlen oder veraltet sind, schlägt er Aktualisierungen vor oder markiert Inhalte zur Überprüfung. |

---

### **Herausforderungen bei der Umsetzung**

Die Implementierung eines solchen agentenbasierten Systems ist anspruchsvoll. Folgende Punkte waren in diesem Anwendungsfall besonders schwierig umzusetzen:

1.  **Qualität und Integration der Datenquellen:**
    *   **Unstrukturierte PDFs:** Bedienungsanleitungen im PDF-Format sind oft nicht für maschinelles Lesen optimiert. Informationen sind in komplexen Layouts, Tabellen und Grafiken "gefangen". Das Extrahieren präziser Informationen (z. B. "Was bedeutet Fehlercode E02 bei Modell X?") erfordert fortgeschrittene Parser und ist fehleranfällig.
    *   **Schnittstellen (APIs):** Die Anbindung an diverse Systeme wie Confluence, Jira und ein CRM erfordert stabile APIs. Nicht immer liefern diese APIs die Daten in der benötigten Form, was eine aufwendige Transformation der Daten nach sich zieht.

2.  **Präzise Intent-Erkennung (NLU):**
    *   Die größte Herausforderung für den **Empfangsagenten** ist die Vieldeutigkeit menschlicher Sprache. Ein Kunde könnte schreiben: "Mein Gerät geht nicht mehr an." Dies könnte eine technische Frage, eine Garantieanfrage oder eine allgemeine Supportfrage sein. Das System muss lernen, durch gezielte Rückfragen oder aus dem Kontext die wahre Absicht zu erkennen, um eine Falschzuweisung zu vermeiden.

3.  **Definition der Eskalationslogik:**
    *   Wann ist der richtige Zeitpunkt, an einen Menschen zu übergeben? Ein niedriger Konfidenz-Score der KI ist ein Indikator, aber keine Garantie für eine falsche Antwort. Eine zu frühe Eskalation senkt die Effizienz, eine zu späte frustriert den Kunden. Das Finden der perfekten Balance erfordert viel Feinabstimmung und Analyse von realen Kundengesprächen.

4.  **Aufbau einer effektiven Feedback-Schleife (Lernagent):**
    *   Ein System, das wirklich "lernt", ist die Königsdisziplin. Der **Lernagent** muss nicht nur Feedback sammeln, sondern auch dessen Qualität bewerten. Er muss unterscheiden, ob eine Eskalation aufgrund einer Wissenslücke oder wegen einer schlecht formulierten Kundenfrage erfolgte. Die automatische Erstellung von Vorschlägen zur Verbesserung der Wissensbasis aus diesen Erkenntnissen ist technisch sehr komplex.
  
    *   
