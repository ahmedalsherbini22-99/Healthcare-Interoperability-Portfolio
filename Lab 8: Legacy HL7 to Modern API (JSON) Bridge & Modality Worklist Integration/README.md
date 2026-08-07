#Lab 8: Legacy HL7 to Modern API (JSON) Bridge & Modality Worklist Integration

## 1. Project Objective
Design and deploy a middleware architecture to bridge the gap between a legacy EMR transmitting TCP/IP HL7 traffic and a modern PACS server requiring RESTful API JSON payloads. The objective was to dynamically translate an inbound clinical radiology order (`ORM^O01`) into a DICOM-compliant JSON object and execute an HTTP POST request to generate a Modality Worklist (MWL) file.

## 2. System Environment
* **Integration Engine:** NextGen Mirth Connect (v4.x)
* **Destination Server:** Orthanc PACS Server
* **Protocols:** TCP/IP (Inbound), HTTP REST (Outbound)
* **Data Standards:** HL7 v2.4, DICOM, JSON
* **Scripting:** JavaScript (Message Tree Manipulation & JSON Stringification)

## 3. Architecture & Execution

**Phase 1: Interface Interception (The Listener)**
* Established a TCP Listener on local port `6662` to capture legacy HL7 order traffic from the source EMR.

**Phase 2: Payload Transformation (HL7 to JSON)**
* Engineered a JavaScript transformation step to extract specific clinical data (Patient ID, Name, Accession, Procedure Description) from the legacy HL7 tree.
* Sliced the 12-digit HL7 DateTime string down to the 8-digit DICOM standard (`YYYYMMDD`).
* Constructed a strict DICOM-compliant JSON object mapped to the correct Modality Worklist tags, and converted it to a string for network transmission.

**Code Deployed:**
```javascript
// Extract raw data from the legacy HL7 tree
var patientId = msg['PID']['PID.3']['PID.3.1'].toString();
var patientLastName = msg['PID']['PID.5']['PID.5.1'].toString();
var patientFirstName = msg['PID']['PID.5']['PID.5.2'].toString();
var accession = msg['ORC']['ORC.2']['ORC.2.1'].toString();
var procDesc = msg['OBR']['OBR.4']['OBR.4.2'].toString();
var dateStr = msg['OBR']['OBR.7']['OBR.7.1'].toString();
var scanDate = dateStr.substring(0, 8); 

// Build the DICOM Modality Worklist JSON object for the API
var orthancPayload = {
    "Tags": {
        "PatientID": patientId,
        "PatientName": patientLastName + "^" + patientFirstName,
        "AccessionNumber": accession,
        "ScheduledProcedureStepSequence": [
            {
                "Modality": "CT",
                "ScheduledProcedureStepStartDate": scanDate,
                "ScheduledProcedureStepDescription": procDesc
            }
        ]
    }
};

// Convert the JSON object to a string for HTTP POST
channelMap.put('jsonPayload', JSON.stringify(orthancPayload));
```
**[ChannelSetup/HTTP Listner](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%208:%20Legacy%20HL7%20to%20Modern%20API%20(JSON)%20Bridge%20&%20Modality%20Worklist%20Integration/ChannelSourceSetup.png?raw=true)
**[Identifying Orthanc API endpoint](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%208:%20Legacy%20HL7%20to%20Modern%20API%20(JSON)%20Bridge%20&%20Modality%20Worklist%20Integration/Identifying%20Orthanc%20API%20endpoint.png?raw=true)
**[WorklistNotEnabled](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%208:%20Legacy%20HL7%20to%20Modern%20API%20(JSON)%20Bridge%20&%20Modality%20Worklist%20Integration/WorklistNotEnabled.png?raw=true)
**[Sent to Orthanc Server](https://github.com/ahmedalsherbini22-99/Healthcare-Interoperability-Portfolio/blob/main/Lab%208:%20Legacy%20HL7%20to%20Modern%20API%20(JSON)%20Bridge%20&%20Modality%20Worklist%20Integration/Success%20in%20Modality%20worklist.png?raw=true)
