# Projektplan: ChatERP 🚀

**ChatERP** ist ein revolutionäres ERP-System für Kleinstunternehmen (z. B. Handwerksbetriebe bis ca. 10 Mitarbeiter), das vollständig über natürliche Sprache gesteuert wird. Statt komplexer Menüstrukturen steht ein einziger "Smart Prompt" im Zentrum, der sowohl Text- als auch Sprachnachrichten verarbeitet.

---

## 🏗️ Die Vision
Mitarbeiter im Feld (z. B. auf der Baustelle) bedienen das ERP per Sprachnachricht in ihrer Muttersprache. Das System versteht die Instruktion, ordnet sie einem deterministischen Business-Case zu und führt die Transaktion sicher aus. Das system ist ein Governance layer zwischen den unmittelbaren Bedürfnissen der User im Feld und der abstrakten Datenstruktur im Hintergrund.

### Kern-USP:
- **Voice-First:** Primäre Interaktion über Sprachnachrichten (Transkription via Azure AI).
- **Sprachunabhängig:** Nutzer sprechen in ihrer Muttersprache; das System verarbeitet die Absicht (Intent).
- **Deterministische Sicherheit:** Keine "random" KI-Aktionen. Jeder Befehl wird einem vordefinierten Usecase zugeordnet.
- **Rollen- & Strukturmodell:** Ein striktes Berechtigungssystem verhindert Fehlbedienungen oder das Löschen kritischer Daten.

---

### 1. Datenmodell (Azure SQL Database)
Das Schema ist relational und auf Integrität ausgelegt. Es dient als Fundament für die kontextuelle Ableitung der STFs.

#### A. Partners & CRM (Kunden, Lieferanten, Firmen)
- **Entities:** `Organizations` (Firmen), `Contacts` (Personen), `Interactions` (Log).
- **Key Fields:** `ID`, `Type` (Kunde/Lieferant), `Name`, `Address`, `VatID`, `IsActive`.
- **CRM Log:** `InteractionID`, `ContactID`, `Type` (Anruf/Email/Besuch), `Summary`, `NextStep`.

#### B. HR & Personalplanung (Basis für Context-Retrieval)
- **Employees:** `EmployeeID`, `FirstName`, `LastName`, `Email`, `MessagingID` (WhatsApp/Telegram), `RoleID`.
- **Roles:** `RoleID`, `RoleName` (Monteur, Backoffice, Chef), `PermissionsJSON`.
- **DeploymentPlan (Einsatzplan):** `PlanID`, `EmployeeID`, `ProjectID`, `StartTime`, `EndTime`, `LocationDescription`. *Hieraus leitet das System ab, wo der Mitarbeiter gerade ist.*

#### C. Warenwirtschaft (Produkte & Lager)
- **ProductGroups:** `GroupID`, `Name` (z.B. Elektrotechnik, Sanitär).
- **Products:** `ProductID`, `GroupID`, `SKU`, `Name`, `Description`, `Unit` (Stk, m, h), `BasePrice`.
- **Inventory:** `ProductID`, `StockLevel`, `WarehouseLocation`, `LastRestockDate`.

#### D. Projektmanagement & Zeit
- **Projects:** `ProjectID`, `ClientID`, `Status` (Angebot, Aktiv, Abgeschlossen), `TotalBudget`, `ManagerID`.
- **TimeLogs (Zeit-Rapportierung):** `LogID`, `EmployeeID`, `ProjectID`, `StartTime`, `EndTime`, `Description`, `IsBillable`.

#### E. Finanzen
- **Invoices:** `InvoiceID`, `ProjectID`, `IssueDate`, `DueDate`, `TotalAmount`, `Status` (Open, Paid, Overdue).
- **InvoiceItems:** `InvoiceID`, `ProductID`, `Quantity`, `UnitPrice`.

#### F. Media & Governance
- **Attachments:** `AttachmentID`, `EntityID` (Link zu Projekt/Schaden), `BlobURL`, `MediaType` (Image/PDF), `OcrResultText`.
- **SmartTransactionFlows (STF):** *(Siehe Detail-Spezifikation unten)*

#### G. Audit & Change-Management (Changelog)
- **AuditLog:** `LogID`, `UserID`, `Timestamp`, `ActionType` (STF_EXEC, SCHEMA_CHANGE, AUTO_FLOW), `RawInput`, `ChangesJSON`.
- **SystemSnapshots:** Historisierte Versionen des DB-Schemas für Rollbacks.

#### H. Automation & Scheduling
- **ScheduledFlows:** `FlowID`, `Name`, `Schedule` (Cron/Interval), `ActionJSON` (Welcher STF oder welche Prozedur soll laufen?), `IsActive`.
- **ScheduledFlowLogs:** `RunID`, `FlowID`, `StartTime`, `EndTime`, `Status` (Success, Error), `OutputLink` (z. B. Link zu generierten Rechnungen).

### 2. Der "Smart-Transaction-Flow" (Logik-Ebene)
Jede Anfrage durchläuft folgenden Prozess:
1.  **Omnichannel-Input:** Sprachnachricht, Text-Prompt oder **Bilder** via Web-Frontend, WhatsApp, **MS Teams** oder Telegram.
2.  **Multimodale Analyse:** Whisper für Voice, Azure AI Vision für Bilder (OCR/Doku).
3.  **Intelligente Klassifizierung & Context Retrieval:**
    *   **Probabilistisches Routing:** Analyse der wahrscheinlichsten Business-Cases bei Unklarheit.
    *   **Data-Driven Inference:** Ableitung von Kontext (z. B. Kunde/Projekt) aus der Personalplanung/Einsatzplan.
4.  **Validierung (Security & Roles):** Prüfung der Berechtigung für den spezifischen Use-Case.
5.  **Interaktive Vervollständigung:** Rückfragen zur Klärung oder Bestätigung von Annahmen.
6.  **Ausführung & Output:** Je nach Case erfolgt eine:
    *   **Transaktion:** Datenanlage oder -pflege (z. B. Zeiterfassung, Materialbuchung).
    *   **Abfrage (Read-Only):** Abruf von Informationen (z. B. *"Wie viel Urlaub habe ich noch?"* oder *"Lagerbestand von Produkt X"*).
    *   **Dokumentenerstellung:** Generierung von DocX, PDF oder E-Mail-Entwürfen (z. B. *"Erstelle ein Angebot für Müller"*).

### 3. Rollenkonzept & Dynamische Prozesse
- **User-Rolle:** Ausführung von zugewiesenen Prozessen (Schreiben, Lesen, Dokumente initiieren). Jede Rolle hat Zugriff auf spezifische STFs.
- **Creator-Rolle (Advanced AI Governance):** 
    *   **STF-Management:** Erstellung, Modifikation und Löschung von Smart Transaction Flows per Prompt.
    *   **Schema-Evolution:** Modifikation der ERP-Datenbankstruktur (neue Tabellen, zusätzliche Spalten) via natürlicher Sprache.
    *   **Integrity Guard:** Überwachung der Systemintegrität bei Änderungen.

---

## ⚙️ Detail-Spezifikation: Smart Transaction Flow (STF) & Evolution

### 1. Datenmodell der STF-Tabelle
| Feld | Typ | Beschreibung |
| :--- | :--- | :--- |
| **FlowID** | UUID | Eindeutiger Identifikator des Prozesses. |
| **Name** | String | Kurzer Name (z. B. "Zeiterfassung_Mitarbeiter"). |
| **AllowedRoles** | JSON/Array | Liste der Rollen, die diesen Flow auslösen dürfen (z. B. `["Monteur", "Chef"]`). |
| **IntentDescription** | Text | Beschreibung für die KI, um den Input diesem Flow zuzuweisen. |
| **ActionType** | Enum | `TRANSACTION` (Write), `QUERY` (Read), `DOCUMENT` (Output). |
| **TargetEntity** | String | Ziel-Tabelle in der ERP-Datenbank (z. B. `TimeLogs`). |
| **Slots (RequiredFields)** | JSON | Definition der benötigten Datenfelder: Typ, Pflichtfeld-Status & Herkunft (z. B. `{"Dauer": "float", "Projekt": "DB_Lookup"}`). |
| **ValidationRules** | JSON | Business-Logik (z. B. `{"MinDauer": 0.25, "MaxDauer": 12}`). |

### 2. Der STF-Lifecycle (Deterministischer Ablauf)
1.  **Klassifizierung & Autorisierung:** Intent-Erkennung und Rollenprüfung.
2.  **Slot-Filling:** Extraktion aus Prompt + kontextuelle Ableitung aus der DB.
3.  **Interaktive Klärung:** Rückfragen bei fehlenden Pflicht-Informationen.
4.  **Finalisierung:** Ausführung der Transaktion / Abfrage / Dokumentenerstellung.

### 3. Evolutionary Logic & Dependency Check
Wenn der Creator das Schema oder die STFs verändert (z. B. neue Spalten hinzufügen), führt das System eine **Impact-Analyse** durch:
1.  **Dependency Discovery:** Prüfung aller STFs auf Abhängigkeiten zur betroffenen Struktur.
2.  **Conflict Reporting:** Identifikation von "Breaking Changes".
3.  **Resilience-Optionen:** 
    *   **Auto-Update:** Automatische Anpassung der betroffenen STFs.
    *   **Mapping:** Virtuelles Mapping alter Logik auf neue Felder.
    *   **Guided Refactoring:** Anleitung zur manuellen Korrektur.
    *   **Rollback:** Sicherer Rückzug bei hohem Risiko.

### 4. Audit & Time-Travel (Das "Ausagebügel-System")
ChatERP ist darauf ausgelegt, dass Fehler (KI-Falschinterpretationen oder Fehlklicks) jederzeit korrigiert werden können:
1.  **Immutability-Ansatz:** Daten werden vorzugsweise versioniert oder Änderungen im AuditLog so detailliert erfasst, dass eine exakte Gegen-Buchung möglich ist.
2.  **Schema-Rollback:** Der Creator kann per Prompt (z. B. *"Mache die letzte Strukturänderung rückgängig"*) das Datenbank-Schema auf einen vorherigen Snapshot zurücksetzen.
3.  **Transaction-Undo:** User können Aktionen per Sprache revidieren (z. B. *"Lösche meine letzte Zeiterfassung, das Projekt war falsch"*). Das System identifiziert die letzte Transaktion und führt den Rollback-Flow aus.

---

## 🤖 Automated Flow Engine (Background Tasks)

Neben den interaktiven STFs verfügt ChatERP über eine Engine für zeitgesteuerte Hintergrundprozesse:

1.  **Zweck:** Automatisierung von Routineaufgaben ohne manuellen Prompt (z. B. tägliche Backup-Checks, monatliche Rechnungsläufe, wöchentliche Statusberichte).
2.  **Definition per Sprache:** Der Creator kann Automated Flows per Prompt anlegen: *"Erstelle einen Flow, der jeden 1. des Monats alle abgeschlossenen Projekte abrechnet und die Rechnungen als PDF bereitstellt."*
3.  **Monitoring & Kontrolle:**
    *   **Dashboard:** In der Web-Oberfläche können alle terminierten Flows und deren Status (Erfolg/Fehler) eingesehen werden.
    *   **Benachrichtigung:** Bei kritischen Fehlern in einem Automated Flow wird der Creator proaktiv (z. B. via Teams/WhatsApp) informiert.
4.  **Output:** Ähnlich wie STFs können diese Flows Daten in der DB ändern, Dokumente erzeugen oder Benachrichtigungen versenden.

---

---

## 🛠️ Technologie-Stack (Azure Cloud Native)

| Komponente | Technologie |
| :--- | :--- |
| **Hosting & Cloud** | Azure App Service |
| **Automation / Jobs**| **Azure Functions** (für zeitgesteuerte Scheduled Flows) |
| **Messaging / Bots** | **Azure Bot Service** (Connectivity für WhatsApp/Teams/Telegram) |
| **Datenbank** | Azure SQL Database (**inkl. Temporal Tables** für native Historisierung) |
| **KI / NLP / Speech** | Azure AI Foundry (Speech-to-Text, Azure OpenAI) |
| **Logic-Engine** | Python (FastAPI / LangGraph für die STF-Steuerung & **Rollback-Logik**) |
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
