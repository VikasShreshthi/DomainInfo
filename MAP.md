# MAP (Message Access Profile) – Full Flow (IVI ↔ Phone)

## Scenario
- IVI = MCE (Message Client Equipment)
- Phone = MSE (Message Server Equipment)
- ACL + pairing already done

---

# 🧠 What is MAP?

MAP allows IVI to:
- Read SMS / MMS / Email
- Receive message notifications
- Send messages (optional)

---

# 📦 MAP Architecture

```
MAP → OBEX → RFCOMM → L2CAP → ACL
```

---

# 🔍 1. SDP (Service Discovery)

## Purpose
Discover:
- MAS (Message Access Server) channel
- MNS (Message Notification Server) channel

---

## Flow

```
IVI (MCE)                              PHONE (MSE)
--------------------------------------------------------

L2CAP (PSM 0x0001)           ----->

SDP ServiceSearchAttributeRequest ----->

                                <----- SDP Response

Extract:
- MAS Instance(s)
  - RFCOMM Channel (e.g., 12)
- MNS Channel (for notifications)
- Supported Message Types (SMS/MMS/Email)
```

---

## Key UUIDs

- MAS = 0x1132
- MNS = 0x1133

---

# 🔗 2. MAS Connection (Client → Server)

## Purpose
Access messages (pull/send)

---

## RFCOMM + OBEX Connect

```
IVI (MCE)                              PHONE (MSE)
--------------------------------------------------------

RFCOMM SABM (MAS Channel) ----->

                                <----- RFCOMM UA

OBEX CONNECT              ----->

                                <----- OBEX CONNECT Response
```

---

## Key Notes

- MAS = main data channel
- OBEX session created over RFCOMM

---

# 🔔 3. MNS Connection (Reverse Channel)

## Purpose
Receive notifications (new SMS, etc.)

---

## Reverse Role

```
PHONE (MSE)                          IVI (MCE)
--------------------------------------------------------

RFCOMM SABM (MNS Channel) ----->

                                <----- RFCOMM UA

OBEX CONNECT              ----->

                                <----- OBEX CONNECT Response
```

---

## Key Insight

👉 MNS is **server on IVI**, client on phone  
👉 This is opposite of MAS

---

# 📥 4. Message Access via MAS (OBEX Operations)

---

## 📁 Set Folder

```
OBEX SETPATH ----->

                <----- Response
```

Example folders:
- telecom/msg/inbox
- telecom/msg/sent

---

## 📋 Get Message Listing

```
OBEX GET Request
Headers:
- Type = "x-bt/MAP-msg-listing"
- Name = folder

                <----- Response

Body:
- XML listing of messages
```

---

## 📄 Get Message

```
OBEX GET Request
Headers:
- Type = "x-bt/message"
- Name = message_handle

                <----- Response

Body:
- Message content (bMessage format)
```

---

## ✉️ Push Message (Send SMS)

```
OBEX PUT Request
Headers:
- Type = "x-bt/message"

Body:
- bMessage

                <----- Response
```

---

# 🔔 5. Notifications via MNS

## Event Report

```
PHONE → IVI:

OBEX PUT Request
Headers:
- Type = "x-bt/MAP-event-report"

Body:
- XML (event report)
```

---

## Example Events

- New message
- Message deleted
- Message sent

---

## Example Payload

```
<Event type="NewMessage"
       handle="1234"
       folder="inbox" />
```

---

# 🔄 6. OBEX Session Management

## Disconnect

```
OBEX DISCONNECT ----->

                <----- Response
```

---

## RFCOMM Disconnect

```
RFCOMM DISC ----->

                <----- UA
```

---

# 📦 7. Data Formats

## Message Listing
- XML

## Message Content
- bMessage (contains SMS/email data)

---

# ⚠️ Common Automotive Issues

## ❌ No notifications
- MNS channel not established

---

## ❌ Messages not loading
- Wrong folder path
- OBEX GET failure

---

## ❌ Delay in SMS display
- Notification received but GET delayed

---

## ❌ Permission issues
- Phone blocks message access

---

# 🧠 Key Differences: PBAP vs MAP

| Feature | PBAP | MAP |
|--------|------|-----|
| Data | Contacts | Messages |
| Channels | 1 (PCE→PSE) | 2 (MAS + MNS) |
| Notification | No | Yes |
| Format | vCard | bMessage / XML |

---

# 🎯 One-Line Answer

> “MAP uses SDP to discover MAS and MNS channels, establishes an OBEX session over RFCOMM for message access via MAS, and a reverse OBEX session via MNS for notifications, enabling the IVI to pull and receive updates about messages from the phone.”

---
