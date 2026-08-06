# Lab: Enterprise HL7 Integration & Database Routing 

## 1. Project Objective
Architect a unidirectional healthcare integration pipeline to simulate real-world Electronic Medical Record (EMR) interoperability. The objective was to capture raw HL7 ADT (Admit, Discharge, Transfer) messages over a TCP network, extract key clinical metadata, and route it into a centralized relational database without manual data entry.

## 2. System Environment
* **Integration Engine:** NextGen Mirth Connect (v4.5)
* **Database Backend:** PostgreSQL (pgAdmin 4)
* **Protocol / Standard:** TCP/IP, HL7 v2.3
* **Scripting:** JavaScript (Transformation), SQL (Routing)

## 3. Architecture & Execution
This pipeline was built utilizing the standard integration engine channel architecture:
* **The Source (Network Listener):** Configured a TCP Listener on port `9001` to act as the receiving endpoint for inbound EMR messages. 
* **The Transformer (Data Parsing):** Developed a JavaScript transformation layer mapping to extract specific node values from the HL7 message tree:
  * Patient Name: `PID.5`
  * Medical Record Number (MRN): `PID.3`
  * Date of Birth (DOB): `PID.7`
* **The Destination (Database Writer):** Configured a JDBC connection to the local PostgreSQL `orthanc_db` database, utilizing a `PreparedStatement` SQL query to insert the extracted variables into a dedicated `patient_admissions` table.

## 4. Execution & Root Cause Troubleshooting
To validate the pipeline, I simulated an EMR system by transmitting a raw HL7 ADT-A01 message into the TCP listener. The channel successfully received and transformed the data, but the initial database insertion failed, requiring systematic root-cause analysis.

* **Issue:** The destination step failed with a generic `DatabaseDispatcherException`. 
* **Root Cause Analysis:** I bypassed the UI error and analyzed the raw Java stack trace, identifying an `org.postgresql.util.PSQLException` indicating 0 columns were found. I isolated the root cause to a syntax conflict: standard SQL utilizes single quotes (`' '`) for text strings, but Mirth utilizes Java `PreparedStatements` to securely bind variables. The hardcoded quotes caused the driver to interpret the variables as literal text rather than dynamic parameters.
* **Resolution:** Removed the single quotes from the SQL configuration, allowing the integration engine to handle the data typing natively. 
* **Validation:** Instead of requesting a new transmission from the source, I utilized the engine's internal queue to reprocess the failed messages. The backlog flushed successfully, populating the PostgreSQL table with zero data loss.

## 5. Visual Documentation
*[Creating a table in the SQL DB](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%206:%20Enterprise%20HL7%20Integration%20&%20Database%20Routing/Creating%20a%20table%20in%20the%20SQL%20DB.png?raw=true)
*[Mirth Connect Dashboard showing the Deployed Channel and active port](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%206:%20Enterprise%20HL7%20Integration%20&%20Database%20Routing/Mirth%20Connect%20Dashboard%20showing%20the%20Deployed%20Channel%20and%20active%20port.png?raw=true)
*[The JavaScript Transformer mapping the HL7 PID segment-The transformer](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%206:%20Enterprise%20HL7%20Integration%20&%20Database%20Routing/The%20JavaScript%20Transformer%20mapping%20the%20HL7%20PID%20segment%5D.png?raw=true)
*[The JavaScript Transformer mapping the HL7 PID segment1](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%206:%20Enterprise%20HL7%20Integration%20&%20Database%20Routing/The%20JavaScript%20Transformer%20mapping%20the%20HL7%20PID%20segment1.png?raw=true)
*[The JavaScript Transformer mapping the HL7 PID segment2](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%206:%20Enterprise%20HL7%20Integration%20&%20Database%20Routing/The%20JavaScript%20Transformer%20mapping%20the%20HL7%20PID%20segment2.png?raw=true)
*[The JavaScript Transformer mapping the HL7 PID segment3](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%206:%20Enterprise%20HL7%20Integration%20&%20Database%20Routing/The%20JavaScript%20Transformer%20mapping%20the%20HL7%20PID%20segment3.png?raw=true)
*[Error in channel logs](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%206:%20Enterprise%20HL7%20Integration%20&%20Database%20Routing/Error%20in%20channel%20Logs.png?raw=true)
*[The fix in the Transformer](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%206:%20Enterprise%20HL7%20Integration%20&%20Database%20Routing/The%20fix%20in%20the%20Transformer.png?raw=true)
*[pgAdmin SQL query showing the successfully inserted Patient Name, MRN, and DOB](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%206:%20Enterprise%20HL7%20Integration%20&%20Database%20Routing/Success.png?raw=true)
