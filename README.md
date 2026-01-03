# 🛡️ SIEM/XDR Deployment with Wazuh (Ansible Automated)
Deployed a Wazuh SIEM/XDR stack in my homelab to provide centralized threat detection and visibility across multiple endpoints. The entire deployment was automated using Ansible, focusing on repeatability, secure defaults, and real-world troubleshooting.

---
## Project Goal
Build a production-style SIEM/XDR environment that:
  - Monitors multiple endpoints from a central platform
  - Enforces secure, certificate-based communication
  - Is fully reproducible via infrastructure automation
#
## The Build
- Wazuh Manager for log ingestion and rule-based detection
- Wazuh Indexer (OpenSearch) for secure storage and search
- Wazuh Dashboard for visualization and analysis
- 3 monitored endpoints reporting to the manager
- Custom Ansible playbooks to automate:
  - Server configuration
  - Indexer & dashboard deployment
  - Firewall/port management
  - Certificate placement and permissions
#
## Data Flow Overview

    [ Endpoints ]
         |
         |  (Agent telemetry: logs, FIM, security events)
         v
    [ Wazuh Manager ]
         |
         |  (Filebeat over TLS – X.509 cert auth)
         v
    [ Wazuh Indexer (OpenSearch) ]
         |
         |  (Indexed alerts & metadata)
         v
    [ Wazuh Dashboard ]

#
## Component Responsibilities
Endpoints
- Run Wazuh agents
- Collect logs, integrity events, and security telemetry
- Forward data securely to the manager

Wazuh Manager
- Central processing and correlation engine
- Applies detection rules and decoders
- Uses Filebeat with TLS to forward alerts to the indexer

Wazuh Indexer
- Secure OpenSearch backend
- Stores alerts in wazuh-alerts-* indices
- Enforces certificate-based client authentication

Wazuh Dashboard
- Visualization and analysis layer
- Queries indexed alerts
- Relies entirely on index templates for data visibility

#
## ⚠️ The Challenge
After deployment, the dashboard was reachable but showed:

    “No template found for the selected index-pattern title [wazuh-alerts-*]”

Symptoms:
- UI fully operational
- No alerts or indexed data
- Broken ingestion pipeline

Root Cause:
A TLS X.509 certificate mismatch introduced during migration.
Certificates were generated for the wrong IP address, causing Filebeat (on the manager) to reject communication with the Indexer when connecting via its real LAN IP.

<img width="694" height="476" alt="WZ14" src="https://github.com/user-attachments/assets/af873097-ce85-48da-936e-58f9c7f5c99b" />

#
## The Fix
- Regenerated certificates using correct IP addresses
- Updated Filebeat TLS configuration
- Validated trust chains and permissions
- Confirmed end-to-end data flow:

        Manager → Indexer → Dashboard

Once corrected, templates loaded correctly and alerts began indexing immediately.

<img width="867" height="484" alt="WZ13" src="https://github.com/user-attachments/assets/1809763e-741b-4921-8a30-0c1472a44436" />

#
## Key Takeaways
- PKI matters, even small mismatches break secure pipelines
- TLS failures often surface as data problems, not UI errors
- SIEM infrastructure depends on strict service-to-service trust
- Automation amplifies both mistakes and fixes accuracy is critical
- Wazuh’s security defaults prevent silent failure, which is a good thing
#

## 📷 Final Output
<img width="667" height="467" alt="WZ14" src="https://github.com/user-attachments/assets/660fe195-e016-41f6-b56a-8644b3c94558" />


#
