# Automated Network Request Management

## Overview

This project simplifies and accelerates the handling of network-related service requests by leveraging ServiceNow's service catalog, workflows, and automation capabilities. It eliminates manual bottlenecks and ensures requests are processed efficiently. End users submit requests through a self-service portal, while automated routing, approvals, and task assignments streamline fulfillment — resulting in faster turnaround, improved SLA compliance, and enhanced transparency for both users and IT teams.

## Table of Contents

1. [Introduction](#introduction)
2. [Project Objectives](#project-objectives)
3. [Key Features](#key-features)
4. [Prerequisites](#prerequisites)
5. [ServiceNow Developer Setup](#servicenow-developer-setup)
6. [Project Implementation](#project-implementation)
   - [a. Service Catalog Creation](#a-service-catalog-creation)
   - [b. Table Creation](#b-table-creation)
   - [c. Request Approvals (Related List)](#c-request-approvals-related-list)
   - [d. Flow Designer — Flow & Actions](#d-flow-designer--flow--actions)
7. [Flow Chart](#flow-chart)
8. [Conclusion](#conclusion)
9. [Author](#author)

## Introduction

Modern enterprises rely heavily on robust and efficient network services to support day-to-day business operations. As organizations grow, network-related requests — such as access provisioning, configuration changes, and connectivity support — increase in volume. Traditional manual handling often leads to delays, errors, and limited visibility.

This project introduces a streamlined solution using ServiceNow's workflow engine, service catalog, and automation features, enabling end users to submit requests through a self-service portal while approvals, task assignments, and notifications are handled automatically.

## Project Objectives

- Design and implement an automated solution for managing network-related service requests in ServiceNow.
- Enable end users to submit requests via a user-friendly self-service portal.
- Use ServiceNow's workflow engine, catalog items, and approval processes to validate and route requests.
- Trigger automated notifications, task assignments, and (where applicable) integration with network automation tools.

## Key Features

- Custom service catalog for common network requests
- Dynamic forms to capture relevant request details
- Automated approval workflows based on request type and sensitivity
- Optional integration with infrastructure management/orchestration tools
- Real-time status updates and notifications to requesters and technicians
- Reporting and analytics on request volume, resolution time, and SLA adherence

## Prerequisites

- A ServiceNow Developer account and Personal Developer Instance (PDI)
- Basic familiarity with ServiceNow Service Catalog, Tables, and Flow Designer

## ServiceNow Developer Setup

1. Go to the [ServiceNow Developer Portal](https://developer.servicenow.com/dev.do) and sign up for a free developer account (Email, First/Last Name, Country, Password).
2. Verify your account via the confirmation email sent to your registered email ID.
3. On the Developer Portal home page, click **Start Building** to request a **Personal Developer Instance (PDI)** or use **App Engine Studio**.
4. Use the **Profile Icon** (top-right corner) to manage your account, request instances, and check your developer profile.

## Project Implementation

Once your PDI is ready, you'll be directed to **Creator Studio** — a guided, no-code environment for building request-based applications by defining forms, tables, and automated workflows.

### a. Service Catalog Creation

**i. Create the Catalog Item**
1. Navigate to **Application Navigator → All → Service Catalog → Maintain Items**.
2. Click **New**.
3. Fill in the details:
   - Name: `Network Request`
   - Catalog: `Service Catalog`
   - Category: `Network`
   - Short Description: `Network request Management`
4. Click **Save**.

**ii. Variables Configuration**

Open the catalog item and, under the **Variables** related list, click **New** for each field:

| Field | Type |
|---|---|
| Is this a New connection or Relocation? | Choice → New / Relocation / None |
| If relocation, provide relocated address | Single Line Text |
| Types of Devices | Choice → Laptop / Mobiles / Others |
| Please provide address here | Single Line Text |
| Provide device details here | Single Line Text |
| If anything else, please specify | Single Line Text |

For each variable, configure: Question (label), Name (used for scripting), Tooltip, Example text, Mandatory/Read-Only settings, and Auto-populate (dot-walking) where applicable.

**iii. Variable Set — Requester Information**

Optionally create a reusable Variable Set and apply it to the catalog item:

| Field | Type | Notes |
|---|---|---|
| Opened on behalf of | Reference | References the User table |
| Email Id | Single Line Text | Auto-populated from "Opened on behalf of" |
| User name | Single Line Text | Auto-populated from "Opened on behalf of" |
| Phone Number | Single Line Text | Auto-populated from "Opened on behalf of" |
| Proof of Document | Attachment | — |

**iv. Catalog UI Policy**

Scenario: When **Types of Devices = Others**, the "please specify" field should become visible.

1. Open the **Network Request** catalog item → **Catalog UI Policy** related list → **New**.
2. Applies to: Catalog item → `Network Request`.
3. Condition: `Types of devices` is `Others`.
4. Save, then add a **UI Policy Action**: select the target variable and set **Visible = True**.
5. Update and test on the catalog form.

### b. Table Creation

**i. Create the Table**
1. Navigate to **System Definition → Tables → New**.
2. Fill in Name/Label (e.g., `Network Database Table` / `u_network_database_table`).
3. Leave **Auto-generate schema** checked if desired.
4. Click **Submit**.

**ii. Create Fields (Columns)**
1. Open the new table's record → go to the **Columns** tab.
2. Click **New** and define each field, e.g.: Work Status, Device Details, Requested For, Date of Enquiry, Customer Address, Request Number, Assignment Group (Reference → Group), Customer Document, Assigned to (Reference → User).

### c. Request Approvals (Related List)

**i. Create the Relationship**
1. Navigate to **System Definition → Relationships → New**.
2. Fill in:
   - Name: `Approval Request`
   - Applies to Table: `Network Database Table`
   - Queries from Table: `Sysapprovals`
   - Active: `True`
3. Save.

**ii. Add the Related List to the Form**
1. Open **Form Designer** for the target table.
2. Add a **Related List** widget and select the related list created above.

### d. Flow Designer — Flow & Actions

**i. Create the Flow**
1. Go to **Flow Designer → New**.
2. Name: `Network Request`, add a description.
3. Click **Build Flow**.

**ii. Configure the Trigger**
1. Click the **(+)** icon.
2. Trigger: **Application → Service Catalog**.
3. Click **Done**.

**iii. Configure Actions**

1. **Get Catalog Variables**
   - Action Inputs → Trigger → Service Catalog → Requested Item.
   - Template catalog items → select table `Network Request`.
   - Select the required variables and move them to the Selected list → **Done**.

2. **Create Record**
   - Table: `Network Database Table`.
   - Map fields (Request Number, Requested For, Work Status = New, Assignment Group = Network, Date of Enquiry, Device Details, Customer Address, etc.) using data pills from the trigger/previous action.
   - Click **Done**.

3. **Ask For Approval**
   - Record/Table: `Requested Item [sc_req_item]`.
   - Approval Reason: `Waiting for approval`.
   - Configure approval rules (Approve/Reject) and approver logic (Anyone approves / Everyone approves — static or dynamic).
   - Click **Done**.

4. **Flow Logic — If Condition**
   - Condition: **Ask For Approval → state is Approved** (or Rejected, as required).
   - Click **Done**.

5. **Send Email**
   - Target Record: the created Network Database Table record.
   - Configure To/CC/BCC (static or dynamic).
   - Subject: `Request had been created`.
   - Body: e.g., "We have received your request with request number: {request number}. Your request will be resolved within 2 business working days."
   - Click **Done**.

6. **Update Record**
   - Record: the created Network Database Table record.
   - Fields: e.g., Assigned to = `<technician>`, Work Status = `In Progress`.
   - Click **Done**.

## Flow Chart

```
Network Request (Active)
│
TRIGGER
└── Service Catalog
│
ACTIONS
1. Get Catalog Variables from Network Request
2. Create Network Database Table Record
3. Ask For Approval
4. If — Request is Approved
     └── 5. Send Email
6. Update Network Database Table Record
```

## Conclusion

The Automated Network Request Management in ServiceNow project delivers a streamlined, reliable, and user-friendly solution for handling network-related service requests. By automating approvals, routing, and notifications, it eliminates delays and errors associated with manual processes while offering transparency through real-time updates and reporting. This improves SLA compliance and user satisfaction while reducing repetitive IT workload and enabling faster fulfillment.

## Author

**Mummalaneni Nithin Kumar**
