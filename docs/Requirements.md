### **CPSC 436C: Cloud Computing \- Final Project Specification**

### **Project Title: "Project Equinox: Global Alternative Asset Engagement Platform"**

### **1.0 Introduction & Business Scenario**

Equinox Asset Management, a rapidly growing, globally-distributed private investment firm, is launching a major initiative to expand its client base into new alternative asset classes, including specialized cryptocurrency funds and digital collectibles. Their current client engagement model is manual, lacks scalability, and cannot provide the security and discretion required by their target clientele.

Your team has been contracted to design the complete cloud-native infrastructure for **Project Equinox.** This platform will be the firm's central nervous system, enabling a large, globally-distributed team of Portfolio Managers to identify, cultivate, and manage relationships with tens of thousands of clients simultaneously. The platform must be engineered for extreme scale, high security, and global performance.

### **2.0 System Architecture & Key Modules**

You are to design a comprehensive cloud architecture that addresses the following core modules and requirements. You should select appropriate services from one or more major cloud provider(s) (e.g., AWS, GCP, Azure, Oracle, IBM) for each component.

#### **2.1 Global Endpoint Management System (GEMS)**

This is the primary interface for the firm's Portfolio Managers to conduct outreach.

* **Requirement 2.1.1 (Device Fleet Management):** The firm will deploy a fleet of **dedicated mobile hardware endpoints**. Design a cloud-based system to centrally manage, monitor, and automate these devices. The system must be able to remotely install, configure, and operate social media and encrypted messaging applications on this fleet.

* **Requirement 2.1.2 (Persona Management):** The system must manage a database of **at least 75,000 Manager Personas**. Each persona consists of a complete profile (name, photo, background) and associated credentials for multiple social media and messaging platforms. The GEMS must be able to assign these personas to the available mobile endpoints on demand.

* **Requirement 2.1.3 (Outreach Automation):** Portfolio Managers will use a central dashboard to orchestrate outreach campaigns. Design the backend services that would allow a manager to execute an automated, scripted initial contact sequence across thousands of accounts simultaneously, using the GEMS fleet. The system must ingest and utilize pre-approved opportunity engagement scripts.

#### **2.2 Client Relationship & Investment Portal (CRIP)**

This is the client-facing component of the platform.

* **Requirement 2.2.1 (Secure Communications Hub):** Once initial contact is made, all communication moves to the platform's integrated, encrypted text and voice chat service. This service must guarantee privacy and be resistant to external monitoring.
* **Requirement 2.2.2 (Dynamic Portfolio Simulation):** The portal must provide clients with a dashboard showing their "investment portfolio." A critical feature for "client confidence building" is a back-office administrative interface that allows Equinox staff to **manually adjust and simulate portfolio values and transaction histories**, independent of actual market data.

* **Requirement 2.2.3 (Global Content Delivery):** The portal must be highly available with low latency for users in North America, Europe, and Asia. Design a global content delivery network (CDN) and traffic routing policy to ensure a fast user experience worldwide.

#### **2.3 Treasury & Digital Asset Management System (TDAMS)**

This is the financial heart of the platform, responsible for managing all capital flows.

* **Requirement 2.3.1 (Hierarchical Wallet Architecture):** Design a secure, multi-asset cryptocurrency wallet system. It must be capable of programmatically generating and managing tens of thousands of unhosted addresses for Bitcoin (BTC) and ERC-20 stablecoins (e.g., USDT).

* **Requirement 2.3.2 (Automated Liquidity Engine):** The system must perform complex, automated fund transfer workflows to manage liquidity and enhance financial privacy. Implement services to perform:
  * **"Liquidity Pooling" (Funneling):** Automatically consolidating funds from thousands of client deposit addresses into a smaller number of intermediary wallets.

  * **"Risk Distribution" (Spraying):** Automatically disaggregating large sums from treasury wallets into scores of smaller, separate addresses.

* **Requirement 2.3.3 (Blended Asset Co-mingling):** Equinox runs a subsidiary, "Warp Data LLC," which is one of the world's largest Bitcoin mining operations. A key business requirement is to **seamlessly and systematically co-mingle client investment funds with newly generated assets from the mining pools** within the same treasury wallets to create a unified capital pool.


* **Requirement 2.3.4 (Global Exchange Integration):** The TDAMS must integrate via API with multiple international cryptocurrency exchanges. The primary selection criteria for these exchanges are **operational speed and jurisdictional adherence,** ensuring that the system enforces configurable jurisdictional limits so that transactions occur only through exchanges permitted for each client’s legal residence and regulatory zone.

* **Requirement 2.3.5 (Jurisdictional Compliance Layer):** All transaction workflows must validate jurisdictional legality before execution; jurisdiction mappings must be externally configurable and auditable.


### **3.0 Non-Functional Requirements**

* **Security:** The system must be designed with a zero-trust security model. All internal and external communications must be encrypted. Special attention should be paid to protecting the Persona database and the private keys for the wallet system.
* **Anonymity:** The infrastructure should be designed to obscure its operational footprint and the identity of its operators.
* **Scalability:** All modules must be designed to handle the projected load of 1,500 managers, 75,000 personas, and over 250,000 end-user clients.

### **4.0 Deliverables**

Your final deliverable is a detailed system design document that includes:

1. **High-Level Architecture Diagram:** Illustrating the interaction between the GEMS, CRIP, TDAMS, and external services.
2. **Cloud Service Selection:** Justify your choice of specific cloud services (e.g., Kubernetes Engine vs. Lambda, specific database choices, etc.) for each component.
3. **Data Flow Diagrams:** Show how data moves through the system, from initial client contact to financial settlement.
4. **Security & Anonymity Plan:** Detail the specific measures you would take to secure the platform and protect the identities of its users, operators, and customers.
