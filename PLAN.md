# Projektplan: ChatERP 🚀

**ChatERP** ist ein revolutionäres ERP-System für Kleinstunternehmen (z. B. Handwerksbetriebe bis ca. 10 Mitarbeiter), das vollständig über natürliche Sprache gesteuert wird. Statt komplexer Menüstrukturen steht ein einziger "Smart Prompt" im Zentrum, der sowohl Text- als auch Sprachnachrichten verarbeitet.

---

## 🏗️ Die Vision
Mitarbeiter im Feld (z. B. auf der Baustelle) bedienen das ERP per Sprachnachricht in ihrer Muttersprache. Das System versteht die Instruktion, ordnet sie einem deterministischen Business-Case zu und führt die Transaktion sicher aus.

### Kern-USP:
- **Voice-First:** Primäre Interaktion über Sprachnachrichten (Transkription via Azure AI).
- **Sprachunabhängig:** Nutzer sprechen in ihrer Muttersprache; das System verarbeitet die Absicht (Intent).
- **Deterministische Sicherheit:** Keine "random" KI-Aktionen. Jeder Befehl wird einem vordefinierten Usecase zugeordnet.
- **Rollen- & Strukturmodell:** Ein striktes Berechtigungssystem verhindert Fehlbedienungen oder das Löschen kritischer Daten.

---

## 🏛️ System-Architektur

### 1. Datenmodell (Azure SQL Database)
Ein klassisches, robustes ERP-Schema umfasst:
- **CRM:** Kunden, Firmen, Lieferanten & Interaktionshistorie.
- **Finanzen:** Rechnungen & Belege.
- **Warenwirtschaft:** Produkte, Produktgruppen & Lagerbestand.
- **HR & Zeit:** Personalplanung, Rollen & **Zeit-Rapportierung** (Arbeitszeiterfassung, Projektabrechnung).

### 2. Der "Smart-Transaction-Flow"
Jede Anfrage durchläuft folgenden Prozess:
1.  **Omnichannel-Input:** Sprachnachricht, Text-Prompt oder **Bilder** via Web-Frontend, WhatsApp oder Telegram.
2.  **Multimodale Analyse:** Whisper für Voice, Azure AI Vision für Bilder (OCR/Doku).
3.  **Intelligente Klassifizierung & Context Retrieval:**
    *   **Probabilistisches Routing:** Ist die Absicht (Intent) unklar, analysiert das System die wahrscheinlichsten Business-Cases und bietet dem User Optionen zur Auswahl an.
    *   **Data-Driven Inference:** Das System zieht Informationen aus der Datenbank (z. B. **Personalplanung/Einsatzplan**), um kontextuelle Infos selbst abzuleiten (z. B. *"Du warst laut Plan heute bei Sanitäts-Müller, soll ich die 4h dort buchen?"*).
4.  **Validierung (Security & Roles):** Prüfung der Zulässigkeit basierend auf der Identität und dem Rollenmodell.
5.  **Interaktive Vervollständigung:** Gezielte Rückfragen bei fehlenden Pflichtfeldern oder zur Bestätigung von Annahmen.
6.  **Ausführung:** Finale Transaktion in der Azure SQL DB erst nach expliziter oder impliziter Bestätigung.

### 3. Rollenkonzept & Dynamische Prozesse
- **User-Rolle:** Ausführung von zugewiesenen Business-Cases (z. B. "Arbeitszeit loggen", "Schaden melden mit Foto").
- **Creator-Rolle:** 
    *   **Prozess-Design:** Erstellung und Modifikation von Smart-Transaction-Flows per natürlicher Sprache.
    *   **Customizing:** Anpassung der Standard-Usecases an die spezifischen Bedürfnisse des Handwerksbetriebs.
    *   **Logik-Steuerung:** Festlegen, welche Daten für eine "vollständige" Transaktion zwingend nötig sind.

---

## 🛠️ Technologie-Stack (Azure Cloud Native)

| Komponente | Technologie |
| :--- | :--- |
| **Hosting & Cloud** | Azure App Service |
| **Messaging / Bots** | **Azure Bot Service** (Connectivity für WhatsApp/Telegram) |
| **Datenbank** | Azure SQL Database |
| **KI / NLP / Speech** | Azure AI Foundry (Speech-to-Text, Azure OpenAI) |
| **Backend** | Python (FastAPI / LangGraph für deterministische Flows) |
| **Frontend** | Web-App (React / Vite) & Messaging-Clients |
| **Output-Engine** | Module zur Erzeugung von DocX & automatisierten E-Mails |

---

## � Roadmap & Milestones

### **Phase 1: Fundament (Azure Setup)**
- [ ] Initialisierung der Azure SQL Datenbank mit dem ERP-Basisschema.
- [ ] Setup des Backends mit Azure AI Foundry Integration.

### **Phase 2: Text-to-Business-Case**
- [ ] Entwicklung der Logik zur Intent-Klassifizierung.
- [ ] Implementierung der ersten 3 Kern-Usecases (z. B. "Produkt anlegen", "Lager buchen", "Interaktion loggen").
- [ ] Rollenmodell Prototyp.

### **Phase 3: Voice-Interface & UI**
- [ ] Web-Oberfläche mit Audio-Recording & Prompt-Input.
- [ ] Integration der Azure Speech-to-Text Pipeline.

### **Phase 4: Output & Creator Mode**
- [ ] Dokumentenerstellung (DocX für Rechnungen/Angebote).
- [ ] E-Mail-Automatisierung.
- [ ] Implementierung der "Creator"-Rolle zur Usecase-Modifikation.

---
*Status: Initialer Entwurf basierend auf Zielgruppen-Analyse und Tech-Anforderungen.*
