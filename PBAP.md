# PBAP (Phone Book Access Profile) Flow with OBEX & Connection Sequence

## Scenario
- IVI = PBAP Client (PCE – Phonebook Client Equipment)
- Phone = PBAP Server (PSE – Phonebook Server Equipment)
- ACL link is already established

---

# 🔍 1. SDP (Service Discovery)

## Purpose
Discover PBAP service and get RFCOMM channel

---

## Flow

```
IVI (PCE)                              PHONE (PSE)
--------------------------------------------------------

L2CAP Connection (PSM 0x0001)  ----->

SDP ServiceSearchAttributeRequest ----->

                                <----- SDP ServiceSearchAttributeResponse

Extract:
- Service Class UUID = PBAP PSE
- RFCOMM Channel Number (e.g., 19)
- Supported Repositories (local, SIM)
- Supported Features
```

---

## Key Notes

- SDP runs on **L2CAP PSM 0x0001**
- PBAP UUID:
  - PSE = 0x112F
- IVI stores RFCOMM channel for next step

---

# 🔗 2. RFCOMM Connection (OBEX Transport)

## Purpose
Create transport channel for OBEX

---

## Flow

```
IVI (PCE)                              PHONE (PSE)
--------------------------------------------------------

L2CAP Connection (PSM 0x0003)  ----->

RFCOMM SABM (DLCI = Server Channel) ----->

                                <----- RFCOMM UA

RFCOMM DLC Established
```

---

## Key Notes

- RFCOMM acts like **serial port**
- OBEX runs on top of RFCOMM
- DLCI = (Server Channel * 2) + 1

---

# 📦 3. OBEX Session Establishment

## Purpose
Start OBEX session over RFCOMM

---

## OBEX Connect

```
IVI (PCE)                              PHONE (PSE)
--------------------------------------------------------

OBEX CONNECT Request  ----->

Headers:
- Version
- Flags
- Max Packet Length

                                <----- OBEX CONNECT Response

Headers:
- Connection ID
- Max Packet Length
```

---

## Key Notes

- OBEX is session-based protocol
- Connection ID used in further requests

---

# 📁 4. PBAP Operations (Phonebook Access)

## Common Operations

---

## 📥 Pull Phonebook

```
IVI → PHONE:

OBEX GET Request
Headers:
- Type = "x-bt/phonebook"
- Name = "telecom/pb.vcf"

                                <----- OBEX GET Response

Body:
- vCard listing
```

---

## 📥 Pull vCard Listing

```
OBEX GET Request
Headers:
- Type = "x-bt/vcard-listing"
- Name = "telecom/pb"

                                <----- OBEX Response

Body:
- List of contacts (handles)
```

---

## 📥 Pull vCard Entry

```
OBEX GET Request
Headers:
- Type = "x-bt/vcard"
- Name = "telecom/pb/1.vcf"

                                <----- OBEX Response

Body:
- Single contact details
```

---

## Key Notes

- Data format = **vCard (VCF)**
- Large phonebooks are split into multiple packets

---

# 🔄 5. OBEX Packet Structure

## Request

```
Opcode + Headers
```

## Response

```
Response Code + Headers + Body
```

---

## Common Headers

- Name
- Type
- Length
- Body / End-of-Body
- Connection ID

---

# 🔚 6. OBEX Disconnect

```
IVI (PCE)                              PHONE (PSE)
--------------------------------------------------------

OBEX DISCONNECT  ----->

                                <----- OBEX DISCONNECT Response
```

---

# 🔌 7. RFCOMM Disconnection

```
RFCOMM DISC  ----->

                <----- RFCOMM UA
```

---

# ❗ Important Interview Points

## PBAP Stack

```
PBAP → OBEX → RFCOMM → L2CAP → ACL
```

---

## Roles

- PCE = Client (IVI)
- PSE = Server (Phone)

---

## Transport

- OBEX over RFCOMM (not over L2CAP directly in classic PBAP)

---

## Data Format

- vCard (2.1 / 3.0)

---

## Security

- Requires pairing + authorization
- May trigger user permission on phone

---

# ⚠️ Common Issues in Automotive

## ❌ Phonebook not downloading
- SDP channel mismatch
- OBEX connect failure

---

## ❌ Partial contacts
- OBEX packet fragmentation issues

---

## ❌ Slow sync
- Large phonebook + sequential GET

---

## ❌ Permission denied
- Phone user did not allow access

---

# 🎯 One-Line Answer

> “PBAP uses SDP to discover the RFCOMM channel, establishes an RFCOMM connection, then creates an OBEX session over it, through which the client performs GET operations to pull phonebook data in vCard format, and finally disconnects OBEX and RFCOMM.”

---
