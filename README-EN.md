# RPA Automation — COMEX (DHL Shipping Document Verification)

## Related Projects

🔒 Private Repository: `https://github.com/EveMurta/MS_Power_Automate_Agente_Conferencia_Doc_Embarque`

## 📌 Description

This set of automations (Power Automate / Azure Logic Apps + UiFlow) manages the receipt, organization, and verification of DHL air shipment documents. The process covers everything from the formal DHL alert to the insertion of validated data into FollowNet.

The workflow is divided into two main email triggers:

1. **DHL Alert** – receives emails with the subject `DHL Express – Formal Alert`, extracts the **HAWB** (House Air Waybill) number from the subject, and saves all attachments into a SharePoint folder.

2. **FOR ENTRY** – receives emails with the subject `FOR ENTRY - Process:`, extracts the **IMP** and the **HAWB** from the body/subject, creates the same folder structure, processes invoices and packing lists, waits for the HAWB PDF to arrive, and then executes a desktop robot that performs the **complete document verification**, inserts the data into FollowNet, and records occurrences with all validations.

---

## 🎯 Objective

* Automate the receipt and organization of DHL shipping documents

* Eliminate the need for manual folder creation and attachment handling

* Perform automatic verification of **HAWB, Invoice, and Packing List**, comparing data such as:

  * Delivery and invoice addresses (compatibility)
  * Gross and chargeable weights (with 10% tolerance)
  * Number of volumes
  * SKU item validation (identifying discrepancies)
  * Fuel surcharge (retrieved from the DHL website for the HAWB period)

* Calculate the total freight value based on extracted data and validate it against the value declared in the HAWB

* Automatically insert verification data into **FollowNet**, opening an occurrence with all validations documented in the "Observations" field

* Notify the team when an essential document (Waybill) is not found

---

## 🛠 Technologies Used

* Power Automate / Azure Logic Apps (cloud flows)
* Office 365 Outlook (email connector)
* SharePoint Online (document storage)
* Conversions Service (HTML → text for HAWB extraction)
* UiFlow (Power Automate Desktop)

  * `COMEX-ConferenciaDeDocumentosDeEmbarqueDHLV6`
  * `COMEX-LeitorInvoiceePackingList`
  * `COMEX-LoginFollowNet`

---

## 🔄 Process Flow

### Cloud Flow (receipt and organization)

```text id="n3k7pd"
[Email: DHL Express – Formal Alert]
    ↓
Extract HAWB from subject
    ↓
Create SharePoint folder: /COMEX - DHL Pre-Alert Verification/HAWB <number>
    ↓
Save attachments in the folder
```

```text id="a5m8tr"
[Email: FOR ENTRY - Process: ...]
    ↓
Extract IMP and HAWB
    ↓
Create or reuse process folder
    ↓
Filter attachments (invoice / packing list)
    ↓
Save attachments in the folder
    ↓
Move email to: [OUTLOOK_PROCESSING_FOLDER]
    ↓
Wait for HAWB (PDF) arrival
    ↓
If HAWB found → execute desktop flow
Otherwise → send alert email to [ALERT_EMAIL]
```

---

### Desktop Flow (verification and follow-up)

```text id="q9v2lf"
Login to FollowNet
    ↓
Search process by HAWB/IMP
    ↓
Open occurrence
    ↓
Execute complete verification (OCR + validations)
    ↓
Compare HAWB x Invoice x Packing List data
    ↓
Validate freight, weights, volumes, and SKUs
    ↓
Fill occurrence in FollowNet
    ↓
Finalize process
```

---

## 📂 Directory Structure (SharePoint)

**Site:** [SHAREPOINT_URL]
**Base Path:** [BASE_PATH]

Example:

```text id="u4h6ws"
/COMEX - DHL Pre-Alert Verification/
└── HAWB <number>/
    ├── InvoicePackinglist <number>_<file>.pdf
    └── other attachments
```

---

## ⚙️ Environment Configuration

| Parameter      | Type   | Default  | Description           |
| -------------- | ------ | -------- | --------------------- |
| Environment    | String | DEV      | Defines DEV or PRD    |
| Execution Mode | String | attended | attended / unattended |

---

## 🧩 Main Flows

| Flow                                               | Description                                |
| -------------------------------------------------- | ------------------------------------------ |
| COMEX-Obtemanexosdee-mailHAWB                      | Processes DHL alerts and saves attachments |
| COMEX-ObtemeconferedocumentoDHLHAWBeInvoicePacking | Processes entry emails                     |
| COMEX-ConfernciadedocumentosdeembarqueDHLV6        | Complete desktop verification              |
| COMEX-LeitorInvoiceePackingList                    | OCR document reading                       |
| COMEX-LoginFollowNet                               | Automatic login to FollowNet               |

---

## 📧 Trigger Rules

* Processed emails are moved to: **[OUTLOOK_PROCESSING_FOLDER]**
* Error alerts are sent to: **[ALERT_EMAIL]**
* Processing is based on HAWB and IMP extracted from the email subject/body

---

## 🚨 Error Handling

* Automatic retry configured in the cloud flow
* Missing HAWB triggers email notification
* Non-PDF files are automatically removed
* Serial execution (no parallelism)

---

## 📝 Logs and Evidence

* History maintained in SharePoint by HAWB folder
* Emails archived in the processing folder
* Occurrences registered in FollowNet with complete validation records

---

## 🔧 Dependencies

* Office 365 account ([CORPORATE_ACCOUNT])
* SharePoint site: [SHAREPOINT_URL]
* Installed UiFlow Agent (ID: [AGENT_ID])
* Configured Power Automate connections:

  * [CONN_OFFICE365]
  * [CONN_SHAREPOINT]
  * [CONN_UIFLOW]
  * [CONN_CONVERSION]

---

## 🔐 Security

* No credentials stored in plain text
* Authentication via connection references
* Restricted SharePoint access by security groups
* Desktop execution based on secure session

---

## 👥 Responsible Team

* RPA / Power Platform Automation Team
* COMEX Team
* SharePoint Infrastructure Team
* FollowNet System Team

---

## 📄 License

Internal corporate use only. Redistribution without authorization is prohibited.

---
