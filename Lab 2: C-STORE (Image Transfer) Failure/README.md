### Lab 2: C-STORE (Image Transfer) Failure

**1. System Environment**
* Server: Orthanc PACS
* Client Modality: Microdicom
* Protocol: DICOM C-STORE

**2. Incident Report**
A clinical workstation (Microdicom) attempted to push a CT medical imaging study (CTImageStorage) via C-STORE to the central PACS server, but the transmission was immediately rejected during the negotiation phase.

**3. Root Cause Analysis (RCA)**
* The Handshake Breakdown: During the DICOM association phase (A-ASSOCIATE-RQ), the client proposed specific Presentation Contexts linking an Abstract Syntax (CTImageStorage) to required Transfer Syntaxes (data compression formats).
* Configuration Defect: The server's underlying configuration file (configuration.json) contained a corrupted, overly restrictive string array (1.2.999.999) for accepted transfer syntaxes.
* Log Evidence: The server runtime flagged the incompatibility, logging: The DICOM server accepts no transfer syntax; thus C-STORE SCP is disabled.
* Consequently, the server categorized the request context as Abstract Syntax Not Supported and aborted the connection to preserve data compliance.

**4. Diagnostic & Troubleshooting Methodology**
* Live Log Capture: Initiated an interactive server trace session via the command-line interface using --verbose and --trace-dicom parameters to isolate real-time network packets.
* Client-Server Isolation: Filtered incoming association streams to verify that base bi-directional connectivity was active via AET MICRODICOM on IP 127.0.0.1.
* Log Parsing: Isolated the exact point of protocol failure by reading PDU handshake parameters and matching error flags against configuration whitelist rules.
* Configuration Audit: Cross-referenced active runtime parameters with modular configuration matrices to uncover the mismatch in compression syntax permissions.

**5. Remediation & Resolution**
* Configuration Correction: Re-established standard compatibility by reverting the "AcceptedTransferSyntaxes" parameter back to its proper universal wildcard array ([ "1.2.840.10008.1.*" ]), allowing the service to interpret standard uncompressed and compressed image streams.
* Service Lifecycle Cycle: Executed a clean service restart to load the updated validation policy.
* Post-Fix Verification: Re-initiated the image push from Microdicom, confirming a successful A-ASSOCIATE-AC handshake, clean C-STORE RQ execution, and proper raw file indexation into the SQLite database backend.

**6. Business & Operational Impact**
* Successfully restored multi-vendor interoperability, ensuring true Vendor Neutral Archive (VNA) compliance.
* Minimized clinical downtime by establishing a structured, log-driven troubleshooting framework for rapid packet inspection and configuration governance.

[Orthanc Error Log - The DICOM server accepts no transfer syntax](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%202:%20C-STORE%20(Image%20Transfer)%20Failure/The%20DICOM%20server%20accepts%20no%20transfer%20syntax.png?raw=true)
[configuration.json - Corrupted Transfer Syntax String](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%202:%20C-STORE%20(Image%20Transfer)%20Failure/Corrupted%20Transfer%20syntax%20string.png?raw=true)
[configuration.json - Corrected Transfer Syntax](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%202:%20C-STORE%20(Image%20Transfer)%20Failure/Corrected%20Transfer%20Syntax.png?raw=true)
