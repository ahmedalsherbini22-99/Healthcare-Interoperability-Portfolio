**1. Technical Architecture & Competencies**
* Primary Technologies: Orthanc PACS, PostgreSQL, pgAdmin, DICOM (C-STORE, Storage Commitment)
* Core Competencies: Database Architecture, JSON Configuration Management, SQL Maintenance, Operational Troubleshooting

**2. Incident Report & Objective**
* Incident: A high-volume imaging modality experienced critical local disk space exhaustion. While image transmissions (C-STORE) were successful, the scanner reported persistent Storage Commitment (N-ACTION) timeouts, preventing it from safely purging its local data caches.
* Objective: Migrate the PACS backend from a lightweight SQLite flat-file to a robust PostgreSQL relational database, and execute database maintenance to restore index seek capabilities and resolve TCP timeouts.

**3. Phase 1: Enterprise Database Migration (PostgreSQL)**
* To support advanced data analytics and resolve backend bottlenecks, the Orthanc PACS was re-architected to run natively on a PostgreSQL environment.
* Database Provisioning: Accessed the local PostgreSQL server via pgAdmin and provisioned a blank target database named orthanc_db.
* Plugin Integration: Verified the presence of the OrthancPostgreSQLIndex.dll within the Orthanc plugins directory to bridge the application and database layers.
* Configuration Architecture: Implemented a modular configuration strategy. Authored a dedicated postgresql.json file containing the strict authentication and routing parameters.
* Configuration Hygiene (Troubleshooting): Identified and resolved a configuration override conflict. The default Orthanc directory contained multiple generic .json files (e.g., a default postgresql.json disabling the index) that were superseding the custom configuration. Isolated the master postgresql.json file and migrated all deprecated files to a backup directory, ensuring a clean, single-source-of-truth boot sequence.
* Initialization & Verification: Booted the Orthanc service via the command line interface (Orthanc.exe --verbose Configuration). Verified the automated injection of relational tables (e.g., resources, metadata, attachedfiles) into the orthanc_db schema via pgAdmin.

**4. Phase 2: Root Cause Analysis & SQL Maintenance**
* Following the migration, the root cause of the N-ACTION timeouts was isolated and remediated using standard database optimization techniques.
* Root Cause Analysis: Heavy transactional inserts from modalities had severely fragmented the table indexes containing DICOM SOP Instance UIDs (publicid). The PostgreSQL engine abandoned the corrupted indexes and executed Full Table Scans to verify 3,000+ UIDs, artificially inflating query times to over 85 seconds and violating the 60-second DICOM TCP timeout limit.
* GUI-Driven Concurrent Maintenance: Accessed the resources table via the pgAdmin Object Explorer. Utilized the Maintenance dialog to execute a REINDEX operation. Toggled the CONCURRENTLY parameter to ensure the index rebuilt in the background without locking the table, preventing hospital downtime during production hours.
* SQL Script Execution: Followed up the reindex with a secondary maintenance script via the Query Tool to clear dead tuples and update the query planner statistics: VACUUM ANALYZE resources;
* Validation Check: Executed a standard SELECT * FROM resources; query to verify data integrity and confirm the query planner was actively routing through the newly optimized publicindex.

**5. Business & Operational Impact**
* Performance Recovery: The index rebuild reduced the Storage Commitment validation query time from 85 seconds down to under 0.5 seconds.
* Workflow Stabilization: The N-EVENT-REPORT handshake was stabilized, allowing the clinical modality to successfully receive safe-storage receipts and automatically purge local files, preventing a halt in clinical operations.
* Analytics Readiness: Bypassed the closed application layer, successfully exposing clinical metadata in a structured PostgreSQL environment. This established the foundational architecture required to connect business intelligence tools (e.g., Power BI) for future operational dashboards.

[postgresql.json Authentication Parameters](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%204:%20PACS%20Migration%20&%20SQL%20Database%20Optimization/postgresql.json%20Authentication%20Parameters.png?raw=true)
[pgAdmin - Index Maintenance Selection](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%204:%20PACS%20Migration%20&%20SQL%20Database%20Optimization/pgAdmin%20-%20Index%20Maintenance%20Selection.png?raw=true)
[pgAdmin - REINDEX Process Completion](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%204:%20PACS%20Migration%20&%20SQL%20Database%20Optimization/pgAdmin%20-%20REINDEX%20Process%20Completion.png?raw=true)
[pgAdmin - Maintenance Concurrently Toggle](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%204:%20PACS%20Migration%20&%20SQL%20Database%20Optimization/pgAdmin%20-%20Maintenance%20Concurrently%20Toggle.png?raw=true)
[pgAdmin - SQL Query Execution for Database Maintenance](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%204:%20PACS%20Migration%20&%20SQL%20Database%20Optimization/SQL%20Query%20Execution%20for%20Database%20Maintenance.png?raw=true)
