# Lab: Automated Quality Assurance & Error Routing

## 1. Project Objective
Design and implement an automated data hygiene pipeline to protect a production database from corrupted or incomplete clinical messages. The goal was to configure strict logic gates that actively interrogate inbound HL7 traffic, quarantine defective messages, and preserve data integrity without interrupting the overall system workflow.

## 2. System Environment
* **Integration Engine:** NextGen Mirth Connect (v4.x)
* **Local File System:** Windows OS (Quarantine Directory)
* **Standard:** HL7 v2.3 (ADT^A01)
* **Scripting:** JavaScript (Filter Logic)

## 3. Clinical Support Scenario
In a live hospital environment, if a patient is admitted via the EMR but the HL7 message is missing critical demographic data (such as the patient's name), processing that message will create a "ghost" record in the downstream database or PACS. This architecture prevents system pollution by intercepting defective records at the integration layer, actively reducing high-tier support tickets and eliminating the need for manual database cleanup.

## 4. Execution & Channel Architecture

**Phase 1: Quarantine Zone Configuration**
* Established a localized physical directory (`C:\Mirth_Quarantine`) to securely hold intercepted, corrupted messages for support team review.

**Phase 2: Dual-Destination Routing**
* Expanded a standard TCP-to-PostgreSQL pipeline by introducing a secondary "File Writer" destination.
* Configured the File Writer to dynamically generate unique file names using Mirth's internal variables (`error_${message.messageId}.hl7`) to ensure no corrupted files are overwritten.
* Utilized the `${message.rawData}` template to preserve the exact original HL7 text for root-cause analysis.

**Phase 3: JavaScript Filter Logic (The Gatekeepers)**
* **Gatekeeper 1 (Database Protection):** Developed a JavaScript filter on the primary PostgreSQL destination to evaluate the inbound payload. The script explicitly rejects the transmission if the Patient Name field is empty.
```javascript
var patientName = msg['PID']['PID.5']['PID.5.1'].toString();

if (patientName.length == 0) {
    return false; // The name is missing! REJECT the message.
} else {
    return true; // The name is there. ACCEPT the message.
}

```

**[The Dual-Destination Architecture](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%207:%20Automated%20Quality%20Assurance%20&%20Error%20Routing/Dual-Destination%20Architecture.png?raw=true)
**[Database Protection Logic](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%207:%20Automated%20Quality%20Assurance%20&%20Error%20Routing/Database%20Protection%20Logic.png?raw=true)
**[Quarantined Payload Validation](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%207:%20Automated%20Quality%20Assurance%20&%20Error%20Routing/Filtered%20successful%20message.png?raw=true)
