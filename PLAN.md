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
- **Creator-Rolle:** 
    *   **Prozess-Design:** Definition von STFs per Sprache (Schema, Berechtigungen, Logik).
    *   **Governance:** Überwachung und Anpassung der Prozess-Leitplanken.

---

## ⚙️ Detail-Spezifikation: Smart Transaction Flow (STF)

Die STFs sind das Herzstück von ChatERP. Sie bilden die kontrollierte Brücke zwischen Sprache und Datenbank. Jeder STF wird in einer dedizierten Tabelle gespeichert.

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

1.  **Klassifizierung & Autorisierung:**
    *   System erkennt den Intent aus Sprache/Text/Bild.
    *   Prüfung: Ist die Rolle des anfragenden Users in `AllowedRoles` enthalten? Falls nein: Abbruch mit Hinweis.
2.  **Slot-Filling (Kontextuelle Intelligenz):**
    *   **Extraktion:** Daten direkt aus dem Prompt ziehen.
    *   **Inference:** Fehlende Daten aus der DB ableiten (z. B. aus der Einsatzplanung des Mitarbeiters).
3.  **Interaktive Klärung:**
    *   Das System fragt gezielt nach fehlenden Slots, für die keine Ableitung möglich ist.
4.  **Finalisierung:**
    *   Sobald alle Pflicht-Informationen vorhanden sind, wird die Transaktion / Abfrage / Dokumentenerstellung ausgeführt.

---

---

## 🛠️ Technologie-Stack (Azure Cloud Native)

| Komponente | Technologie |
| :--- | :--- |
| **Hosting & Cloud** | Azure App Service |
| **Messaging / Bots** | **Azure Bot Service** (Connectivity für WhatsApp/Teams/Telegram) |
| **Datenbank** | Azure SQL Database |
| **KI / NLP / Speech** | Azure AI Foundry (Speech-to-Text, Azure OpenAI) |
| **Logic-Engine** | Python (FastAPI / LangGraph für die STF-Steuerung) |
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
