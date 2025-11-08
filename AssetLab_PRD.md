# **Project Documentation: AssetLab (Gen-AI ITSM Platform)**

## **1\. Project Vision & Summary**

**Project Name:** AssetLab

**Elevator Pitch:** AssetLab is an enterprise-grade, **Gen-AI powered IT Asset Management (ITAM)** and **Service Management (ITSM)** platform. It replaces spreadsheets by allowing companies to manage their full asset lifecycle. It uses AI to auto-triage support tickets and notifies the IT team in real-time via SignalR, Slack, and conditional email alerts.

**The Problem:** IT teams are bottlenecks. They spend 80% of their time manually reading, prioritizing, and diagnosing simple support tickets, all while trying to manage hardware in a messy spreadsheet.

**The Solution:** A full-stack (Angular \+ .NET \+ PostgreSQL) application that acts as an **AI-first IT agent**. It intelligently analyzes new tickets, provides root cause analysis, and frees up the IT team to execute, not just triage.

**Core Technology Stack:**

* **Frontend:** Angular 17+ & TypeScript  
* **Backend:** .NET 8 Web API & C\#  
* **Real-time:** .NET SignalR  
* **Database:** PostgreSQL (with jsonb for AI data)  
* **Authentication:** .NET Identity with JWT (JSON Web Tokens)  
* **AI:** Google Gemini API (with **Structured JSON Schema Output**)  
* **Email (Outbound):** SendGrid (for critical alerts)  
* **Notifications:** Slack/Teams Webhooks

## **2\. User Roles & Permissions (RBAC)**

A refined, 4-tier role-based access control system.

| Role | Name | Permissions |
| :---- | :---- | :---- |
| **Admin** | **Company Admin** | \- **(The Owner)** Manages billing, company settings, and user accounts. \- Can invite/add new users (Employee, IT Staff, IT Manager). \- Can configure integrations (Email, Slack) and global settings. \- Has all IT Manager permissions. |
| **Manager** | **IT Manager** | \- **(Team Lead)** Has full control over the ITAM/ITSM modules. \- Can see *all* assets, tickets, and audit logs. \- Can view the **Analytics Dashboard** (cost reports, resolution times). \- Can delete/retire assets. \- Manages the IT Staff. |
| **Staff** | **IT Staff** | \- **(The "Worker Bee")** The day-to-day IT user. \- Can manage tickets (assign, change status). \- Can check-in/check-out assets to employees. \- Can run maintenance on assets. \- **Cannot** see cost reports or delete assets. |
| **Default** | **Employee** | \- **(Read-Only)** Can view *only* their own assigned assets. \- Can submit a support ticket for *only* their assets. \- Can track the status of *only* their own tickets. |

## **3\. Core Data Model (PostgreSQL Schema)**

* **AspNetUsers Table (from .NET Identity):**  
  * Id, Email, PasswordHash, TwoFactorEnabled  
  * Role (e.g., "IT Staff") \- *Managed by .NET Identity Roles*  
* **Assets Table:**  
  * AssetID (Primary Key, Guid)  
  * Name, AssetTag, SerialNumber, PurchaseDate, Cost  
  * Status (int, Enum): 0=Purchased, 1=InStock, 2=Assigned, 3=InMaintenance, 4=Retired  
  * **IsCritical (boolean):** *e.g., Is this a CEO's laptop or a core server?*  
  * QRCodeURL (string)  
  * AssignedToUserID (Foreign Key), CategoryID, ManufacturerID  
* **SupportTickets Table:**  
  * TicketID (Primary Key)  
  * Title, Description, Status (int, Enum: Open, InProgress, Closed)  
  * ReportedByUserID (Foreign Key)  
  * AssetID (Foreign Key) \- *This can be auto-linked by the backend.*  
  * **AIAnalysis (jsonb):** A JSONB column to store the *entire* structured JSON object from the Gemini API. This is flexible and powerful.  
    * *Example: {"priority": "High", "category": "Hardware \> Power", "rootCausePrediction": "...", "suggestedActions": \[...\]}*  
* **AuditLogs Table:**  
  * LogID (Primary Key)  
  * Timestamp (DateTime)  
  * UserID (Foreign Key) \- *Who made the change?*  
  * Action (string) \- *e.g., "User 'admin@acme.com' logged in."*  
  * TargetEntity (string) \- *e.g., "Asset"*  
  * TargetEntityID (string) \- *e.g., "\[AssetID\]"*  
  * Changes (JSONB) \- *e.g., {"status": {"old": "Assigned", "new": "InMaintenance"}}*  
* **Settings Table:**  
  * SettingKey (string, e.g., "SlackWebhookURL", "SupportEmailAddress")  
  * SettingValue (string)

*(Note: Categories, Manufacturers, Departments, and Locations tables are also included as simple lookup tables.)*

## **4\. Core Features & "Star" Features (Epics)**

### **Epic 1: Asset Lifecycle Management**

* **Story 1 (Purchase):** An IT Staff can add a new asset with a Status of "Purchased" or "InStock."  
* **Story 2 (Assign):** An IT Staff can check-out an asset to an Employee, changing Status to "Assigned."  
* **Story 3 (Maintain):** An IT Staff can check-in an asset for repair, changing Status to "InMaintenance."  
* **Story 4 (Retire):** An IT Manager can "Retire" an asset, removing it from circulation.

### **Epic 2: QR Code Management**

* **Story 5 (Generate):** When an asset is created, the .NET backend generates a QR code and saves its URL.  
* **Story 6 (Scan):** An IT Staff can use their phone to scan the QR tag and instantly view the asset's details.

### **Epic 3: ⭐ Star Feature: Gen-AI Smart Helpdesk (with SignalR)**

* **Story 7 (Submit):** An Employee submits a ticket via the web portal.  
* **Story 8 (Auto-Detect):** The .NET backend logic attempts to auto-link the ticket to the user's assigned assets based on keywords (e.g., "laptop").  
* **Story 9 (AI Triage):** The .NET backend calls the Gemini API with a **Structured JSON Schema**, asking for a full analysis: priority, category, rootCausePrediction, and suggestedActions.  
* **Story 10 (Save):** The backend saves this complete JSON object in the SupportTickets.AIAnalysis column.  
* **Story 11 (Real-time Notify):** The backend uses **SignalR** to push a notification to all connected IT Staff dashboards. The Angular app receives this and *instantly* adds the new ticket to the top of the list without a page refresh.  
* **Story 12 (Admin View):** The IT Staff dashboard shows the new ticket, its AI-priority, predicted cause, and a checklist of suggested actions.

### **Epic 4: ⭐ Star Feature: Audit Logs**

* **Story 13 (Track):** The system automatically logs critical events (User Login, Asset Created, Ticket Status Changed) to the AuditLogs table.  
* **Story 14 (View):** An IT Manager can view a complete, filterable audit trail to see "who changed what, and when."

### **Epic 5: ⭐ Star Feature: Analytics & Notifications**

* **Story 15 (Analytics):** The IT Manager can view a dashboard with ngx-charts showing:  
  * **Cost Tracking:** Total asset cost by department.  
  * **Ticket Resolution Time:** Average time to close tickets.  
  * **Asset Utilization:** % of assets deployed vs. in stock.  
* **Story 16 (Webhook Config):** An IT Manager can paste a **Slack/Teams Webhook URL** into the settings.  
* **Story 17 (Webhook Notify):** When an Urgent ticket is flagged by the AI, the .NET backend *immediately* sends a message to that webhook.  
* **Story 18 (Critical Email Alert):** When a ticket is created (Story 7\) AND (the AI flags it as 'Urgent' OR the linked Asset.IsCritical is true), the backend sends an email alert (via SendGrid) to the IT Manager role.

### **Epic 6: ⭐ Star Feature: Advanced Security**

* **Story 19 (2FA/OTP):** IT Admin and IT Manager roles can be *required* to set up 2-Factor Authentication (OTP) using an app like Google Authenticator.  
* **Story 20 (Login Audit):** The AuditLogs will specifically track successful and *failed* login attempts.