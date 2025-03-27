# DataRun System  
**National Malaria Control Program (NMCP), Yemen**  
*Comprehensive Data Management Solution for Malaria Control Operations*
## Executive Summary  
DataRun is a purpose-built, modular data management platform designed to streamline NMCP's field operations, data collection, and intervention planning. Developed in-house with zero licensing costs, it offers full ownership, offline functionality, and seamless integration with existing health systems. Key benefits include enhanced data integrity, real-time decision-making tools, and adaptable workflows for malaria-specific campaigns.
## 1. Introduction

### 1.1 Purpose

This document provides a comprehensive overview of the DataRun system, designed specifically for managing activities and data for the National Malaria Control Program (NMCP). It serves as a foundational reference for understanding the system’s architecture, features, and functionalities, and is intended to guide ongoing development, maintenance, and integration with other health information systems.

### 1.2 Scope  
Covers system architecture, core features, security protocols, and strategic advantages. Designed to facilitate informed decisions on nationwide rollout and integration.  

**Resource Portal**:  
Access live documentation, API guides, and updates at [https://docs.nmcpye.org](https://docs.nmcpye.org).

---

## 2. DataRun System Overview

### 2.1 Description & Value Proposition

**What is DataRun?**  
DataRun is a dynamic, modular data management system tailored to the NMCP’s needs. It supports field operations by dynamically configuring projects, activities, teams, assignments, and organizational units (including field warehouses). Its key value propositions include:

| **Feature**       | **Typical Systems**       | **DataRun**                  |  
|--------------------|---------------------------|------------------------------|  
| **Cost**           | $5k+/year subscriptions  | **Zero licensing fees**      |  
| **Control**        | Vendor-locked            | Full NMCP ownership          |  
| **Customization**  | Generic templates        | Malaria-optimized workflows  |  
| **Data Hosting**   | Offshore servers         | On-premise/Yemeni cloud      |  


### 2.2 High-Level Architecture

**System Modules:**

- **Mobile App:**  
  - Provides offline data capture with encrypted local storage (SQLite with AES-256).
  - Synchronizes data via HTTPS and supports real-time authentication using OAuth2.

- **Backend Services:**  
  - **Spring Boot API:** Connects to both PostgreSQL (for metadata) and MongoDB (for dynamic data) and utilizes Redis for caching.
  - **Authentication:** Managed by Keycloak, ensuring secure role-based access.
  
- **Integrations:**  
  - Supports formats like ODK, RESTful API integrations (e.g., PowerBI, DHIS2), and webhooks for seamless interoperability.
  
- **Security:**  
  - Implements TLS 1.3 for secure communications.
  - Employs an audit service to log and monitor all user interactions.

**Architecture Diagram:**
![001-High-Level System Architecture](<screenshots/001-High-Level System Architecture-adaptive-adaptive.svg>)


### 2.3 System Objectives

DataRun is engineered to:

- **Enhance Data Management:** Streamline the collection, processing, and storage of NMCP-related data.
- **Support Informed Decision-Making:** Provide robust tools and reports for effective malaria control and prevention.
- **Enable Field Data Collection:** Allow offline data collection in remote areas with later synchronization.
- **Ensure Data Integrity:** Maintain strict data quality and traceability standards.
- **Improve Intervention Planning:** Leverage reusability of activity data to optimize cost and planning efficiency.

### 2.4 Key Features

- **Project Management:**  
  - Capture departmental efforts as distinct projects (e.g., ITNs distribution, larvicide campaigns).

- **Field Activities:**  
  - Target entities such as locations, health facilities, or individuals.
  - Manage tasks like team supervision, resource distribution, and data collection.

- **Assignments & Tracking:**  
  - Link activities with teams, organizational units, and scheduling.
  - Use reference forms to track locations and resource allocations.

- **Dynamic Data Forms:**  
  - Create customizable data collection forms to capture relevant, activity-specific data.
  - Support offline data entry with subsequent synchronization.

- **Data Visualization:**  
  - Generate pivot tables, charts, and dashboards for effective data analysis and reporting.

- **Interoperability:**  
  - Open API architecture allows easy integration with platforms such as DHIS2, ODK, HRMS, LMIS, and supply chain systems.

---

## 3. Mobile & Data Collection

### 3.1 Dynamic Data Collection Forms

- **Form Templates:**  
  - Support multiple field types (e.g., integer, date, entity references).
  - Customizable for specific activities and team assignments.

### 3.2 Mobile-First Design

- **Offline-First Functionality:**  
  - Allow form completion without connectivity; sync data when online.
  
- **User Modes:**  
  - **Supervisors:** Access real-time summaries and transaction histories.
  - **Field Users:** seamless data entry with dynamic form rendering.
  
- **Key Technologies:**  
  - **SQLite:** For secure, local storage (AES-256 encryption).
  - **WorkManager:** For scheduling background sync tasks.
  - **Conflict Resolution:** Configurable mechanisms (e.g., last-write-win, server-win) with versioning.

### 3.3 Data Flow

![Data Flow](<screenshots/002-Data Flow Diagram.svg>)

---

## 4. Backend Architecture

### 4.1 Hybrid Database Setup

DataRun utilizes a hybrid database approach:

- **PostgreSQL:**  
  - Manages structured metadata (users, teams, assignments, organizational units).
  
- **MongoDB:**  
  - Stores dynamic data including forms, submissions, and rapidly evolving entities.

```plaintext
                 +---------------------+
                 |   Spring Boot API   |
                 +----------+----------+
                            |
           +----------------+-----------------+
           |                                  |
+----------+----------+             +---------+----------+
| PostgreSQL (RDBMS)  |             | MongoDB (NoSQL)    |
| - Users             |             | - Data Forms       |
| - Teams             |             | - Submissions      |
| - Assignments       |             | - Dynamic Entities |
| - OrgUnits          |             | - ... Other        |
| - ...Other          |             +--------------------+
+---------------------+
```

### 4.2 Advanced API Capabilities

#### Hybrid Query Engine

The engine supports advanced querying with virtual joins across PostgreSQL and MongoDB, enhanced by a Redis caching layer.

**Example API Query:**

```http
GET /api/v1/dataSubmissions?query={
  "$and": [
    {"assignment.region": "Aden"}, 
    {"submissionDate": {"$gte": "2023-10-01"}},
    {"status": "approved"}
  ]
}&fields=["team.id","items.count"]&page=2&size=50
```

**Engine Execution Flow:**

```mermaid
flowchart TD
    REQ[API Request] --> Parser
    Parser -->|Extract Filters| QE[Query Engine]
    QE -->|SQL| PG[PostgreSQL]
    QE -->|MQL| MG[MongoDB]
    PG -->|JDBC| Combiner
    MG -->|Mongo Driver| Combiner
    Combiner -->|Normalize| Formatter
    Formatter -->|JSON:API| RESP[Response]
```

---

## 5. Security and Compliance

### 5.1 Security Measures

- **End-to-End Encryption:**  
  - Encrypts data in transit (TLS 1.3 with certificate pinning) and at rest (SQLCipher, AES-256-XTS).
  
- **Authentication & RBAC:**  
  - OAuth2 with Keycloak ensures secure, role-based access control.
  
- **Audit Trails:**  
  - Comprehensive logging using PostgreSQL Audit Extensions for immutable record keeping.

### 5.2 Security Architecture
![alt text](<screenshots/003-Security Architecture-adaptive.svg>)


**Compliance Standards:**

- Yemeni Ministry of Health Privacy Guidelines
- WHO Malaria Data Standards

---

## 6. Use Case: ITNs Distribution Campaign

**Workflow Overview:**

1. **Central Warehouse:**  
   - Allocate 10,000 nets.
   - Create assignments:
     - **Team A:** 5,000 nets.
     - **Team B:** 5,000 nets.

2. **Field Operations:**  
   - Field teams scan QR codes.
   - Submit distribution forms capturing:
     - Nets received (5,000)
     - Nets distributed (e.g., 4,872)
     - GPS verification (e.g., -12.3, 44.5)

3. **Dashboard Reporting:**  
   - Real-time dashboards indicate 97.4% coverage.
   - Automatic flagging of discrepancies greater than 2%.

**Roles:**

- **Supervisor:**  
  - Registers daily resource movements.
  - Accesses real-time summaries and detailed logs.
  
- **Field Team:**  
  - Receives assignments based on planned distributions.
  - Uses dynamic forms for data entry (both offline and online).

- **Inventory Reports and Online Management**  
Inventory reports can be configured to be managed online/offline. e.g. When WH operations are configured to be Online operations, it ensures quantities are handled directly and accurately, without discrepancies or errors.  

Key functionalities include:  
1. **Stock Issuance and Returns**:  
   - A team can issue items from their assigned inventory and return them later.  
   - Stock transfers can occur between teams or between warehouses.  

2. **Restrictions on Inventory Use**:  
   - Teams can only issue items if there’s sufficient stock available (configurable).  
   - If a team issues stock to another warehouse, the receiving warehouse's stock must increase accordingly.  

3. **Offline Discrepancy Handling**:  
   - If stock movement occurs while a team is offline (per configuration) (e.g., a team issues stock but doesn’t have connectivity), the operations team can manually reconcile the discrepancy.  

---

### **Configuration decisions fo Supervisor Role**  
Supervisors can:  
1. **Access Team Reports and Summaries**:  
   - Supervisors can view historical reports such as village updates, inventory changes, or any team activities. These reports are organized chronologically, much like a bank statement.  

2. **Reassign Villages Between Teams**:  
   - If a team is unable to cover a village or if another team covers it instead, supervisors can reassign the village from the first team to the second.  
   - Once reassigned, the new team will see the task as **overdue** and must submit the required report promptly.  

3. **Manage Same-Day Tasks**:  
   - Villages scheduled for the current day must be addressed on the same day.  
   - If a village needs to be postponed, supervisors must **reschedule it for another day** and specify the new date.  

### **System Workflow and Automation**  
- The system ensures that every campaign or operation ends with complete and accurate reports for all assigned villages.  
- Each village has a detailed history, showing:  
  - How it started.  
  - The team initially responsible.  
  - Any reassignment history (whether by date or between teams).  

## 7. Other Use Cases:
Datarun can be configured to manage most NMCP activities workflows, such as:
- Routine Data for health facilities malaria cases, submitted or linked to external system such as `EIdews`.
- ICCM data, routine data submission from community volunteers, supervision visit data...etc
- other workflow per need.
---

## 7. Implementation Roadmap
**Timeline:**

```mermaid
gantt
    title Deployment Phases
    dateFormat  YYYY-MM-DD
    section Phase 1
    Training       :2024-08-01, 30d
    Larvicide Campaign   :2024-09-22, 6d
    ITNs Campaign   :2024-11-12, 18d
    section Phase 2
    Training  :2025-04-01, 120d
    ICCM Rollout  :2025-04-01, 60d
    1000 HFs LMIS Rollout :2025-05-01, 90d
```

**Resource Requirements:**

- NMCP Technical Working Group Training  
- Field Coordinator Training
---
# 9. User Interface
## 9.1 Desktop Interfaces

### 9.1.1 Activities and User Management

- **Activities Management:**  
  ![Activity's Teams Management](screenshots/activities_management_screen.jpg)

- **Teams Management:**  
  ![Teams Management Screen](screenshots/Activity_teams_management_screen.jpg)

- **User Management:**  
  ![User Management](screenshots/user_management.jpg)

### 9.1.2 Form Templates and Elements

- **Data Elements Management:**  

  | ![1](screenshots/data_element_1.jpg) | ![2](screenshots/data_element_2.jpg) |  |
  | ----------------------- | ----------------------- |-------------------------|
  | **Data Elements Management Screen** | **Data Element Definition Creation** |  |

- **Options and OptionSets:**  
  
  | ![Options Management](screenshots/Options_OptionSet_management_and_definitions_1.jpg) | ![OptionSets Definitions](screenshots/Options_OptionSet_management_and_definitions_2.jpg) |  |
  | ----------------------- | ----------------------- |-------------------------|
  | **Options Management Screen**  | **OptionSets Definition Creation** |  |

### 9.1.3 Form Template Builder

- **Main Screen:**  
  ![Form Template Builder Main Screen](screenshots/form_templates_builder_main_screen.jpg)

- **Template Management:**  
  ![Form Template Management](screenshots/form_templates_builder_form_template_definition_screen.jpg)

- **Sections & Data Elements Selection:**  
  
  | ![Sections Management](screenshots/form_templates_builder_sections_management_definition.jpg)   | ![Data Elements Selection](screenshots/form_templates_builder_dataelements_selection.jpg) |  |
  | ----------------------- | ----------------------- |-------------------------|
  | **Sections Management** | **Data Elements Selection** |  |

- **Rules and Expressions:**  
 | ![Rules Definition Screen](screenshots/form_templates_builder_rules_definition_screen.jpg)    | ![Rules Expressions](screenshots/form_templates_builder_rules_expressions.jpg) |  |
  | ----------------------- | ----------------------- |-------------------------|
  | **Rules Definition Screen** | **Rules Expressions** |  |

## 9.2 Mobile App
Datarun-Mobile's app was developed with dart (Flutter). The app facilitates the submission and synchronization of malaria-related data with the
main backend system.

## Features

- **Dynamic Form Download**: The app downloads forms designed on the backend, which can include
  various question types.
- **Question Types**: Supports Text, Number, Date, Multi Answer, Single Answer, Image, and File
  questions.
- **Data Submission**: Users can submit various malaria-related data directly from the app.
- **Data Synchronization**: The app syncs submitted data with the Data-run-Api, ensuring all
  information is up-to-date.
- **User Authentication**: Secure login and authentication to ensure data integrity and privacy.
- **User Management**: Quickly create users and assign them to particular teams.
  connection is restored.
- **User-Friendly Interface**: Simple and intuitive design to facilitate ease of use by healthcare
  workers.

### 9.2.1 Assignment/task's card:

| ![1](https://github.com/user-attachments/assets/e16f955c-e9c8-4633-b280-3ae52403f945)   | ![2](https://github.com/user-attachments/assets/9b4b9d96-180a-4e75-9c32-ce00588401a5)  |

### 9.2.2 V1.0.0 App interfaces

| ![activity landing page](https://raw.githubusercontent.com/Hamza-ye/images-uploads/refs/heads/main/v1.0.2-assets/screenshots/001.png)   | ![2](https://raw.githubusercontent.com/Hamza-ye/images-uploads/refs/heads/main/v1.0.2-assets/screenshots/002.png)  | ![3](https://raw.githubusercontent.com/Hamza-ye/images-uploads/refs/heads/main/v1.0.2-assets/screenshots/003.png)   |
|---------------------------|--------------------------|---------------------------|
| ![4](https://raw.githubusercontent.com/Hamza-ye/images-uploads/refs/heads/main/v1.0.2-assets/screenshots/004.png)   | ![5](https://raw.githubusercontent.com/Hamza-ye/images-uploads/refs/heads/main/v1.0.2-assets/screenshots/005.png)  | ![6](https://raw.githubusercontent.com/Hamza-ye/images-uploads/refs/heads/main/v1.0.2-assets/screenshots/006.png)   |
| ![7](https://raw.githubusercontent.com/Hamza-ye/images-uploads/refs/heads/main/v1.0.2-assets/screenshots/007.png)   | ![8](https://raw.githubusercontent.com/Hamza-ye/images-uploads/refs/heads/main/v1.0.2-assets/screenshots/008.png)  | ![9](https://raw.githubusercontent.com/Hamza-ye/images-uploads/refs/heads/main/v1.0.2-assets/screenshots/009.png)   |