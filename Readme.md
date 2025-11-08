# **AssetLab: A Gen-AI Powered IT Asset Management (ITSM) Platform**

**AssetLab is a modern, enterprise-grade, full-stack application built with .NET 8, Angular 17, and PostgreSQL.** It's an AI-first platform designed to replace spreadsheets and automate IT operations, from asset lifecycle management to helpdesk support.

This project demonstrates a real-world, scalable architecture including a Gen-AI helpdesk, real-time notifications with SignalR, QR code asset tagging, and role-based access control (RBAC).

## **⭐ Key Features**

This isn't just a simple CRUD app. It's a "Mini-ITSM" platform with advanced, real-world features.

* **Gen-AI Smart Helpdesk:**  
  * **AI Auto-Triage:** When a new ticket is submitted, it's automatically sent to the **Google Gemini API**.  
  * **Structured JSON Analysis:** The AI (using a JSON schema) returns a full analysis: priority, category, rootCausePrediction, and a checklist of suggestedActions.  
  * **AI-Powered Admin:** The IT admin dashboard is a "command center" that shows new tickets *already triaged* by the AI.  
* **Real-time Notifications (SignalR & Webhooks):**  
  * **Instant UI Updates:** The IT Admin dashboard is built with **SignalR**. New tickets appear *instantly* at the top of the list without a page refresh.  
  * **Slack/Teams Integration:** AI-flagged "Urgent" tickets automatically fire a webhook, sending a real-time notification to the IT team's chat.  
  * **Critical Email Alerts:** If an asset is marked as IsCritical (e.g., a CEO's laptop), a ticket submission instantly triggers an email alert (via SendGrid).  
* **Full Asset Lifecycle Management:**  
  * **Full CRUD:** Complete management of assets, categories, manufacturers, and locations.  
  * **Lifecycle Status:** Track an asset from Purchased \-\> InStock \-\> Assigned \-\> InMaintenance \-\> Retired.  
* **QR Code Generation & Scanning:**  
  * **Backend:** The .NET API generates and saves a unique QR code for every new asset.  
  * **Frontend:** A mobile-friendly Angular page allows an IT Admin to scan an asset's QR tag with their phone and instantly pull up its details, history, and tickets.  
* **Enterprise Security & Auditing:**  
  * **Role-Based Access Control (RBAC):** Four distinct roles (Company Admin, IT Manager, IT Staff, Employee) with granular permissions.  
  * **2-Factor Authentication (OTP):** High-privilege admin accounts can be secured with Time-based OTP (e.g., Google Authenticator).  
  * **Audit Logs:** A complete, unchangeable log tracks *who* changed *what* and *when* (e.g., "User 'X' assigned asset 'Y' to user 'Z'").  
* **Analytics Dashboard:**  
  * An IT Manager dashboard with ngx-charts showing:  
    * Total Asset Cost (by department/category).  
    * Ticket Resolution Time.  
    * Asset Utilization (% deployed vs. in stock).

## **Core Technology Stack**

This project is a monorepo containing two separate projects:

### **Backend (/backend)**

* **Framework:** .NET 8 Web API (C\#)  
* **Database:** PostgreSQL  
* **ORM:** Entity Framework (EF) Core  
* **Authentication:** .NET Identity with JWT (JSON Web Tokens) & 2FA  
* **Real-time:** .NET SignalR  
* **AI:** Google Gemini API (using structured JSON output)  
* **Integrations:** SendGrid (email), Slack Webhooks

### **Frontend (/frontend)**

* **Framework:** Angular 17+ (TypeScript)  
* **UI:** Angular Material (for a professional, clean UI)  
* **Charts:** ngx-charts  
* **QR Scanning:** ngx-scanner

## **🚀 How to Run Locally**

### **Prerequisites**

* [.NET 8 SDK](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)  
* [Node.js 18+](https://nodejs.org/en)  
* [PostgreSQL](https://www.postgresql.org/download/) (and a tool like pgAdmin)  
* An IDE (e.g., [VS Code](https://code.visualstudio.com/))

### **1\. Configure Backend**

1. Navigate to the /backend directory.  
2. Set up your User Secrets (right-click project in Visual Studio or use dotnet user-secrets set):  
   {  
     "ConnectionStrings": {  
       "DefaultConnection": "Server=127.0.0.1;Port=5432;Database=assetlab\_db;User Id=postgres;Password=YOUR\_POSTGRES\_PASSWORD;"  
     },  
     "Jwt": {  
       "Key": "YOUR\_SUPER\_SECRET\_JWT\_SIGNING\_KEY\_32\_CHARS\_LONG",  
       "Issuer": "httpsCreator://your-app.com",  
       "Audience": "httpsCreator://your-app.com"  
     },  
     "Gemini": {  
       "ApiKey": "YOUR\_GOOGLE\_GEMINI\_API\_KEY"  
     }  
   }

3. Run the database migration to create your tables:  
   dotnet ef database update

4. Run the backend server:  
   dotnet run

   The API will be running at https://localhost:7001 (or similar).

### **2\. Configure Frontend**

1. In a new terminal, navigate to the /frontend directory.  
2. Install all packages:  
   npm install

3. Run the Angular app:  
   ng serve \--open

   The app will open at http://localhost:4200 and will be ready to connect to your backend.