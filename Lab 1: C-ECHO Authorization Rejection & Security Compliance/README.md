### Lab 1: C-ECHO Authorization Rejection & Security Compliance

**1. System Environment**
* Server: Orthanc PACS
* Client Modality: Microdicom
* Protocol: DICOM (Port 4242)

**2. Incident Report**
A local DICOM viewer (Microdicom) encountered an immediate connection termination when attempting to verify network connectivity (C-ECHO) with the centralized PACS server.

**3. Root Cause Analysis (RCA)**
* The connection failure was not caused by a network outage, but by the server’s strict internal security protocols.
* The PACS server actively refused the Association Request because the client's Application Entity Title (AET) was not registered in the server's trusted modalities matrix (DicomModalities).

[Insert Screenshot: Microdicom DICOM Nodes Options]
[Insert Screenshot: Orthanc Logs - DICOM authorization rejected]
[Insert Screenshot: configuration.json - DicomAlwaysAllowEcho set to false]

**4. Troubleshooting Methodology**
* Executed a standard C-ECHO request from the client modality, which resulted in a visible system failure.
* Accessed the real-time server trace logs to capture the exact transaction data during the connection attempt.
* Isolated the point of failure in the log output: DICOM authorization rejected for AET MICRODICOM on IP 127.0.0.1: This AET is not listed in configuration option "DicomModalities".
* Verified the server's standard operating parameters, confirming that strict security enforcement (DicomAlwaysAllowEcho) was properly set to false.
* Audited the configuration.json file to identify the gap between the incoming request parameters and the allowed system inputs.

**5. Remediation & Resolution**
* Updated the server's configuration.json file to explicitly map and authorize the client's parameters.
* Added the specific AET, IP address, and listener port to the "DicomModalities" JSON object.
* A subsequent service restart applied the new security policy, yielding a successful Association Acknowledged log status and restoring verified connectivity.

[Insert Screenshot: The Fix - Adding AET to configuration.json]
[Insert Screenshot: Verified Microdicom Configuration]

**6. Business & Operational Impact**
* This proactive security configuration enforces strict data governance.
* By ensuring that only vetted and explicitly authorized clinical tools can interact with the centralized server, it safeguards sensitive patient imaging data from unauthorized internal or external network queries.
