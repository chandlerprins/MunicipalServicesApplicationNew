# 🏛️ Municipal Services Web Application

---

## 📋 Table of Contents

1. [Project Overview](#project-overview)  
2. [Changelog](#changelog)  
   - [Improvements from Part 1 → Part 2](#improvements-from-part-1--part-2)  
   - [Part 3 — Service Status Requests (this submission)](#part-3--service-status-requests-this-submission)  
3. [Features](#features)  
4. [Technologies & Architecture](#technologies--architecture)  
5. [Prerequisites](#prerequisites)  
6. [How to get the project and open it in Visual Studio](#how-to-get-the-project-and-open-it-in-visual-studio)  
7. [Compile & Run (Visual Studio)](#compile--run-visual-studio)  
8. [Using the Application — focused: Service Status Requests](#using-the-application---focused-service-status-requests)  
9. [Developer Notes & Implementation Details (Part 3)](#developer-notes--implementation-details-part-3)  
10. [File storage & permissions](#file-storage--permissions)  
11. [Troubleshooting & common issues](#troubleshooting--common-issues)  
12. [References / Demo](#references--demo)

---

## 🏗️ Project Overview

Municipal Services is a web application that lets residents report municipal issues (water, electricity, sanitation, roads), attach supporting files, view submitted reports, and receive local events & announcement updates. The UI uses Razor views (ASP.NET Core MVC). The project includes several custom data structures implemented in C# to demonstrate algorithmic approaches (binary search tree map and a min-heap), used in the Service Status Request functionality.

---

## 🔧 Changelog

### Improvements from Part 1 → Part 2
- Added Events & Announcements pages.
- Introduced RecommendationTracker to track event click/category affinity.
- UI improvements and toast notifications (Notyf).
- Collections usage for event/announcement management (Dictionaries, Queues, HashSets).

### Part 3 — Service Status Requests 
- Implemented Service Status Request system using:
  - BinarySearchMap<string, Issue> — ordered case-insensitive string key index for IssueCode -> Issue, enabling efficient search/scan and IssueCode generation.
  - MinHeap<Issue> per department — prioritized department queues (status -> priority -> createdAt -> issue code).
  - RequestManager centralizes in-memory storage: canonical LinkedList<Issue>, BinarySearchMap index, and per-department MinHeap queues.
  - StatusController provides department queue views, sorting, searching, and status change endpoints that keep heap/index consistent.
- Additions and changes:
  - Domain/MunicipalCollections/BinarySearchMap.cs
  - Domain/MunicipalCollections/MinHeap.cs
  - Services/RequestManager.cs
  - Controllers/StatusController.cs
  - Views/Status/*.cshtml for reporting and department queues
- Behavior highlights:
  - Issue codes generated like W001, S002 using BinarySearchMap traversal.
  - Changing an issue status updates the issue object and rebalances the department heap (remove + reinsert).
  - Department views show a snapshot from the heap sorted by priority rules.

---

## ✨ Features (high level)

- Create issues with category, description, optional attachments (.jpg/.jpeg/.png/.pdf).
- My Reports: view your previously submitted issues.
- Feedback & survey for issues and general site satisfaction.
- Events, announcements and recommendation hints.
- Service Status Requests with per-department prioritized queues and administrative controls to change status.

---

## 🛠️ Technologies & Architecture

- .NET target: .NET 8  
- Language: C# 12  
- Web framework: ASP.NET Core MVC (Controllers + Razor Views)  
- Frontend: Bootstrap 5, minimal jQuery  
- Custom domain collections: BinarySearchMap, MinHeap, LinkedList  
- Notifications: AspNetCoreHero.Notyf

---

## ⚙️ Prerequisites

- Visual Studio 2022 (latest updates recommended)  
- .NET 8 SDK installed  
- Permission to write to project folder (for uploads in /wwwroot/uploads)

---

## 📥 How to get the project and open it in Visual Studio

1. Open Visual Studio 2022.  
2. Choose __Clone Repository__ from the start window or the __Git__ menu.  
3. Paste repository URL:  
   https://github.com/VCNMB-3rd-years/vcnmb-prog7312-2025-poe-chandlerprins.git  
   and clone locally.  
4. In Visual Studio choose __File > Open > Project/Solution__ and open the solution (.sln) file.  
5. If packages are not already restored, right-click the solution and choose __Restore NuGet Packages__.

---

## ▶️ Compile & Run (Visual Studio)

1. In Solution Explorer, right-click the web project (MunicipalServices) and choose __Set as Startup Project__.  
2. Select a launch profile in the Visual Studio toolbar (IIS Express or Project).  
3. Build: __Build > Build Solution__ (or press Ctrl+Shift+B).  
4. Run: __Debug > Start Debugging__ (F5) or __Debug > Start Without Debugging__ (Ctrl+F5).  
5. The site opens in your browser; navigate to Home or use the top navigation links.

All steps above use Visual Studio UI 

---

## 🖱️ Using the Application — focused: Service Status Requests

- Access: Top navigation -> Service Requests or navigate to /Status/Index.  
- Departments: Select a department (Water, Sanitation, Electricity, Roads) to open its queue.  
- Queue behavior:
  - Items are shown ordered by status (Pending → In Progress → Resolved), then numeric priority (1 = highest), then oldest first, then IssueCode as tie-breaker.
  - Search the queue by partial IssueCode using the search box on the Department view.
  - Sort options include date ascending/descending, status, and priority.
- Change status:
  - From the department view there are controls to mark an issue __In Progress__ or __Resolved__. When you change status the RequestManager:
    - Updates the Issue object in-place.
    - Removes and reinserts the Issue in the department MinHeap to preserve ordering.
- Generate Issue Codes:
  - New issues get codes like W001, S002; generation inspects existing keys in the BinarySearchMap to find the highest number for the same prefix.

---

## 🧭 Developer Notes & Implementation Details (Part 3)

- RequestManager responsibilities:
  - Canonical issue list stored in a custom LinkedList<Issue>.
  - BinarySearchMap<Issue> used as a case-insensitive ordered index keyed by IssueCode. Useful for:
    - Fast lookups by exact IssueCode (TryGet).
    - Scanning existing keys when generating new IssueCode prefixes.
    - Returning Entries() in-order for prefix/partial searches.
  - _departmentQueues: Dictionary<string, MinHeap<Issue>> — each department has a MinHeap with comparator:
    - Compare by StatusWeight (Pending=0, In Progress=1, Resolved=2),
    - Then by Priority (lower numeric = higher priority),
    - Then by CreatedAt (older first),
    - Then by IssueCode (case-insensitive).
  - When adding an issue: store in linked list, add to BinarySearchMap, push to corresponding department heap.
  - When updating status: update the Issue instance, remove+push in heap to re-evaluate position.
- BinarySearchMap characteristics:
  - Case-insensitive string keys (uses StringComparison.OrdinalIgnoreCase).
  - Supports Add (insert/update), TryGet, Remove, Contains, Count, Entries() (in-order traversal).
  - Complexity: average O(h) where h = height of tree (unbalanced). For the project scale this is acceptable; balancing (AVL/Red-Black) not implemented.
- MinHeap characteristics:
  - Push/Pop/Remove — typical heap operations O(log n) (push/pop) and O(n) for Remove (index search + adjustments), but works for small departmental queues.
- Files of interest (Part 3):
  - Domain/MunicipalCollections/BinarySearchMap.cs  
  - Domain/MunicipalCollections/MinHeap.cs  
  - Services/RequestManager.cs  
  - Controllers/StatusController.cs  
  - Views/Status/Index.cshtml, Department.cshtml, Reports.cshtml
- Testing tips:
  - Seed data is applied on first run (check SeedData class). It creates sample issues and events so you can exercise department queues immediately.
  - Use the Department view to verify ordering and status transitions.

---

## 📁 File storage & permissions
- Uploaded attachments stored under /wwwroot/uploads. 

---

## ⚠️ Troubleshooting & common issues

- Missing packages: right-click solution -> __Restore NuGet Packages__.  
- Static files missing: confirm files exist under /wwwroot and file URLs begin with `/uploads/`.  
- Seed data not visible: ensure application starts without exceptions and check the Visual Studio __Output__ window for logs.

---

## 📌 Demo

- YouTube Demo: https://youtu.be/Hpi-wnhhRbc
