# Lab 5: HL7 ORM Payload Parsing & Transformation

## 1. Project Objective
Design, deploy, and validate a middleware TCP listener channel in NextGen Mirth Connect. The objective was to intercept inbound HL7 ORM (Order) traffic and dynamically manipulate misaligned patient demographic data using JavaScript to ensure downstream system interoperability.

## 2. System Environment
* **Integration Engine:** NextGen Mirth Connect
* **Standard:** HL7 v2.4 (ORM^O01)
* **Scripting:** JavaScript (E4X / Message Tree Manipulation)

## 3. Clinical Support Scenario
During a PACS implementation, the source EMR was transmitting the patient identifier in the `PID-2` field of an HL7 ORM message. The receiving Modality Worklist (MWL) server strictly required this identifier to be present in the `PID-3` field. Without middleware intervention, the orders were rejected by the PACS, causing clinical technologists' schedules to appear blank.

## 4. Execution & Channel Architecture

**Phase 1: Interface Construction**
* Established a TCP Listener on local port `6661` to intercept inbound clinical orders.
* Configured a Channel Writer destination to internally capture and hold the processed payload for QA auditing.

**Phase 2: Payload Transformation Logic**
* Mapped a sample `ORM^O01` message into the Source Transformer to generate the data tree structure.
* Engineered a custom JavaScript mapping step to dynamically capture the data string from the `PID.2` segment and write it into the empty `PID.3` segment on the fly.

**Code Deployed:**
```javascript
// Extracts the ID from PID-2, converts to a standard string, and assigns it to PID-3
msg['PID']['PID.3']['PID.3.1'] = msg['PID']['PID.2']['PID.2.1'].toString();
```
## 6. Visual Documentation

### 1. Inbound Channel Configuration (TCP Listener)
[TCP Listener Configuration](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/images/%60tcp_listener.png%60.png?raw=true)

### 2. JavaScript Transformation Mapping
[JavaScript Transformation Logic](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/images/%60javascript_mapping.png%60.png?raw=true)

### 3. Log Auditing & Encoded Validation
[Corrected Outbound Payload](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/images/%60encoded_success.png%60.png?raw=true)


