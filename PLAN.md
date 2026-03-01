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
1.  **Omnichannel-Input:** Sprachnachricht oder Text-Prompt via **Web-Frontend, WhatsApp oder Telegram**.
2.  **Transkription & Analyse:** Verarbeitung durch Azure AI Foundry Bausteine (Whisper für Voice-to-Text).
3.  **Klassifizierung:** Zuordnung der Anfrage zu einem konkreten **Business Case** (z. B. "Zeiterfassung", "Materialbuchung").
4.  **Validierung (Security & Roles):** Prüfung der Zulässigkeit basierend auf der verknüpften Identität (z. B. Telefonnummer in WhatsApp).
5.  **Daten-Vervollständigung:** Interaktive Rückfrage über denselben Kanal (z. B. WhatsApp Antwort), falls Informationen fehlen.
6.  **Ausführung:** Finale Buchung in die Datenbank erst bei Vollständigkeit.

### 3. Rollenkonzept
- **User-Rolle:** Ausführung von zugewiesenen Business-Cases (z. B. "Arbeitszeit von heute loggen").
- **Creator-Rolle:** Anpassung und Erstellung neuer Usecases – ebenfalls via Prompt-Interface.

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
