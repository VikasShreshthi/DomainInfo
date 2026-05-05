## HCI Over UART 
There are five kinds of HCI packets that can be sent via the UART Transport Layer; 
1. HCI Command packet --> used to send commands to the Controller from the Host
2. HCI Event packet   --> used by the Controller to notify the Host when events occur
3. HCI ACL Data packet --> used to exchange data between the Host and Controller
4. HCI Synchronous ---> used to exchange synchronous data (SCO and eSCO) between the Host and Controller
Data packet 
5. HCI ISO Data packet. ---> used to exchange isochronous data between the Host and Controller.

HCI Command packets can only be sent to the Bluetooth Controller \
HCI Event packets can only be sent from the Bluetooth Controller \
HCI ACL/Synchronous/ISO Data Packets can be sent both to and from the Bluetooth Controller \

***Authentication Requested command :*** used to establish authentication between the two devices associated with \
the specified Connection_Handle. \

***Change Connection Link Key command :*** used to force both devices of a connection associated to the Connection_Handle, \
to generate a new link key. \

***Create Connection command :*** This command will cause the BR/EDR Link Manager to create an ACL connection to the
BR/EDR Controller with the BD_ADDR specified by the parameters.

***Delete Stored Link Key command***: Remove one or more of the link keys stored in the Controller.

***Disconnect command :*** terminate an existing BR/EDR or LE connection.

***Disconnection Complete event :***  event occurs when a connection has been terminated.

***Exit Sniff Mode command :*** This command is used to end Sniff mode for a Connection_Handle which is currently in Sniff mode.

***Extended Inquiry Result event:*** This event indicates that a BR/EDR Controller has responded with an extended inquiry response
during the current Inquiry process.

***Hardware Error event*** : This event is used to indicate some type of hardware failure for the Controller.

***Inquiry command*** : This command will cause the BR/EDR Controller to enter Inquiry Mode. Inquiry Mode is used
to discovery other nearby BR/EDR Controllers.

***Inquiry Cancel command*** This command will cause the BR/EDR Controller to stop the current Inquiry if the BR/EDR
Controller is in Inquiry Mode.

***Inquiry Complete event*** event indicates that the Inquiry is finished.

***Inquiry Result event*** event indicates that a BR/EDR Controller or multiple BR/EDR Controllers have responded
so far during the current Inquiry process.

***IO Capability Request event*** : event is used to indicate that the IO capabilities of the Host are required for a Secure
Simple Pairing process

***Link Key Notification event*** This event is used to indicate to the Host that a new Link Key has been created for the connection
with the BR/EDR Controller specified in BD_ADDR.

***Link Key Request event*** event is used to indicate that a Link Key is required for the connection with the device
specified in BD_ADDR.

***Link Key Request Reply command*** : This command is used to reply to an HCI_Link_Key_Request event from the BR/EDR Controller, 
and specifies the Link Key stored on the Host to be used as the link key for the connection with the other BR/EDR Controller specified by BD_ADDR.

***Synchronous Connection Complete event*** This event indicates to both the Hosts that a new synchronous connection has been established.

***
**Scan Enable**
The Scan_Enable parameter controls whether or not the BR/EDR Controller will periodically scan for page attempts and/or 
inquiry requests from other BR/EDR Controllers. If Page Scan is enabled, then the device will enter page scan mode based
on the value of the Page_Scan_Interval and Page_Scan_Window parameters. If Inquiry Scan is enabled, then the BR/EDR Controller 
will enter Inquiry Scan mode based on the value of the Inquiry_Scan_Interval and Inquiry_Scan_Window parameters.

Value    Parameter Description
0x00     No Scans enabled.
0x01     Inquiry Scan enabled.
         Page Scan always disabled.
0x02     Inquiry Scan disabled.
         Page Scan enabled.
0x03     Inquiry Scan enabled.
         Page Scan enabled.
All other values Reserved for future use

---
**Inquiry Scan Interval**
The Inquiry_Scan_Interval configuration parameter defines the amount of time between
consecutive inquiry scans. This is defined as the time interval from when the BR/EDR
Controller started its last inquiry scan until it begins the next inquiry scan.

Time Default: 2.56 s

---
**Inquiry Scan Window**
The Inquiry_Scan_Window configuration parameter defines the amount of time for the
duration of the inquiry scan

Time Default: 11.25 ms

---
***Page Timeout***
The Page_Timeout configuration parameter together with Extended_Page_Timeout
defines the maximum time the local Link Manager will wait for a Baseband page
response from the remote device at a locally initiated connection attempt. If this time
expires and the remote BR/EDR Controller has not responded to the page at Baseband
level, the connection attempt will be considered to have failed.

Time Default: 5.12 s

---
**Connection Accept Timeout**
The Connection_Accept_Timeout configuration parameter allows the BR/EDR or LE
Controller to automatically deny a connection request after a specified time period has
occurred and the new connection is not accepted. The parameter defines the time
duration from when the BR/EDR Controller sends an HCI_Connection_Request

Time Default: 5 s

---
**Page Scan Interval**
The Page_Scan_Interval configuration parameter defines the amount of time between
consecutive page scans. This time interval is defined from when the Controller started
its last page scan until it begins the next page scan.

Time Default: 1.28 s

---

**Link Supervision Timeout**
The Link_Supervision_Timeout parameter is used by the BR/EDR Controller to monitor
link loss. If, for any reason, no packets are received from that Connection_Handle
for a duration longer than the Link_Supervision_Timeout, the connection shall be
disconnected. The same timeout value is used for both synchronous and ACL
connections for the device specified by the Connection_Handle.

Time Default: 20 s

---
# HCI Command Flow for ACL Connection

Scenario
IVI = Initiator (Master)
Phone = Acceptor (Slave)
Device was discovered earlier via Inquiry (BD_ADDR known)

# ACL Connection Flow (IVI ↔ Phone)

## Scenario
- IVI = Initiator (Master)
- Phone = Acceptor (Slave)

---

## Connection Establishment

```
IVI (Initiator)                          PHONE (Acceptor)
--------------------------------------------------------------

HCI_Create_Connection  ----->

                        <-----  HCI_Connection_Request

HCI_Accept_Connection_Request ----->

                        <-----  HCI_Connection_Complete

<----- HCI_Connection_Complete
```

---

## Authentication Phase

```
IVI                                      PHONE
-----------------------------------------------

<----- HCI_Link_Key_Request

-----> Link_Key_Request_Reply
        OR
-----> Link_Key_Request_Negative_Reply
```

---

## Pairing Phase (if no link key)

```
IVI                                      PHONE
-----------------------------------------------

<----- HCI_IO_Capability_Request

-----> HCI_IO_Capability_Response

<----- HCI_User_Confirmation_Request

-----> HCI_User_Confirmation_Request_Reply

<----- HCI_Link_Key_Notification
```

---

## Encryption Phase

```
IVI                                      PHONE
-----------------------------------------------

HCI_Set_Connection_Encryption ----->

<----- HCI_Encryption_Change
```

---

## Final State

```
ACL Link Established
Encryption Enabled
Ready for L2CAP (A2DP, HFP, PBAP, etc.)
```

---

## Failure Case Example

```
HCI_Connection_Complete
Status = Page Timeout
```

Cause:
- Phone not in Page Scan
- Device out of range

Host → Controller (IVI)
HCI_Create_Connection
  BD_ADDR = <Phone_BD_ADDR>
  Packet_Type = DM1, DH1, DM3, DH3, DM5, DH5
  Page_Scan_Repetition_Mode = R1 / R2 / R0
  Clock_Offset = <from Inquiry> (optional but improves performance)
  Allow_Role_Switch = 0x01 (True) / 0x00 (False)

Controller → Host (IVI)
HCI_Command_Status
  Status = 0x00 (Success / Pending)
  Opcode = HCI_Create_Connection

Controller → Host (Phone)
HCI_Connection_Request
  BD_ADDR = <IVI_BD_ADDR>
  Class_of_Device = <IVI_COD>
  Link_Type = ACL (0x01)

Host → Controller (Phone)
HCI_Accept_Connection_Request
  BD_ADDR = <IVI_BD_ADDR>
  Role = 0x00 (Become Slave)

ACL Link Established

Controller → Host (IVI)
HCI_Connection_Complete
  Status = 0x00 (Success)
  Connection_Handle = 0xXXXX
  BD_ADDR = <Phone_BD_ADDR>
  Link_Type = ACL
  Encryption_Mode = Disabled (0x00)

Controller → Host (Phone)
HCI_Connection_Complete
  Status = 0x00 (Success)
  Connection_Handle = 0xYYYY
  BD_ADDR = <IVI_BD_ADDR>
  Link_Type = ACL
  Encryption_Mode = Disabled (0x00)

Optional Role Switch
HCI_Role_Change
  Status = 0x00
  BD_ADDR = <Peer>
  New_Role = Master / Slave

Authentication Phase
Controller → Host
HCI_Link_Key_Request
  BD_ADDR = <Peer>

Host → Controller
HCI_Link_Key_Request_Reply
  BD_ADDR = <Peer>
  Link_Key = <Stored_Key>

No Link key
HCI_Link_Key_Request_Negative_Reply
  BD_ADDR = <Peer>

If link key not present then need to do pairing for which it checks capablities using
HCI_IO_Capability_Request

HCI_IO_Capability_Response
  IO_Capability = DisplayYesNo / NoInputNoOutput / etc
  OOB_Data_Present = 0x00
  Authentication_Requirements

  HCI_User_Confirmation_Request
  Numeric_Value = <6-digit number>

  HCI_User_Confirmation_Request_Reply

  HCI_Link_Key_Notification
  BD_ADDR
  Link_Key
  Key_Type

❗ Failure Scenarios (Important for Debugging)

HCI_Connection_Complete
  Status = 0x04 (Page Timeout)

Causes:
- Phone not in Page Scan
- Device out of range

HCI_Connection_Complete
  Status = 0x0F (Rejected due to limited resources)

HCI_Authentication_Complete
  Status != 0x00

---

# Bluetooth Device Inquiry and Pairing Flow (IVI ↔ Phone)

## Scenario
- IVI = Initiator
- Phone = Responder
- Phone is in Inquiry Scan (discoverable) and Page Scan (connectable)

---

# 🔍 1. Device Discovery (Inquiry Phase)

## HCI Flow

```
IVI (Initiator)                          PHONE (Responder)
--------------------------------------------------------------

HCI_Inquiry  --------->

                (Phone in Inquiry Scan)

                <---------  FHS Packet (over air)

<--------- HCI_Inquiry_Result
            (BD_ADDR, Class_of_Device, Clock_Offset)

<--------- HCI_Inquiry_Complete
```

---

## Key Notes

- Inquiry uses **GIAC (General Inquiry Access Code)**
- Phone responds with **FHS packet (NOT ID packet)**
- IVI learns:
  - BD_ADDR
  - Device Class
  - Clock Offset (used for faster paging)

---

# 👆 2. User Selects Device

```
User selects phone from IVI UI
→ BD_ADDR is now used for connection
```

---

# 🔗 3. Connection Establishment (Paging)

```
IVI                                      PHONE
-----------------------------------------------

HCI_Create_Connection  ----->

                        <-----  HCI_Connection_Request

HCI_Accept_Connection_Request ----->

                        <-----  HCI_Connection_Complete

<----- HCI_Connection_Complete
```

---

# 🔐 4. Pairing Phase (Secure Simple Pairing)

## Step 1: Check for existing key

```
IVI                                      PHONE
-----------------------------------------------

<----- HCI_Link_Key_Request

-----> HCI_Link_Key_Request_Negative_Reply
        (if first-time pairing)
```

---

## Step 2: IO Capability Exchange

```
IVI                                      PHONE
-----------------------------------------------

<----- HCI_IO_Capability_Request

-----> HCI_IO_Capability_Response
        (DisplayYesNo / NoInputNoOutput / etc)
```

---

## Step 3: User Confirmation

```
IVI                                      PHONE
-----------------------------------------------

<----- HCI_User_Confirmation_Request
        (6-digit number)

-----> HCI_User_Confirmation_Request_Reply
```

---

## Step 4: Link Key Generation

```
IVI                                      PHONE
-----------------------------------------------

<----- HCI_Link_Key_Notification
        (Link Key generated and stored)
```

---

# 🔒 5. Enable Encryption

```
IVI                                      PHONE
-----------------------------------------------

HCI_Set_Connection_Encryption ----->

<----- HCI_Encryption_Change
```

---

# ✅ Final State

```
✔ Devices Paired (Bonded)
✔ Link Key Stored
✔ Encryption Enabled
✔ Ready for Profiles (A2DP, HFP, PBAP, etc.)
```

---

# ❗ Failure Cases

## Inquiry Failure
```
HCI_Inquiry_Complete
Status != Success
```

## Pairing Failure
```
HCI_Authentication_Complete
Status != Success
```

## Connection Failure
```
HCI_Connection_Complete
Status = Page Timeout
```

---

# 🧠 Important Interview Points

- Inquiry = Discovery (no connection)
- Paging = Connection establishment
- Pairing happens **after ACL link is created**
- FHS packet is used in inquiry response
- Encryption is **disabled initially**, enabled after pairing
- Inquiry Scan ≠ Page Scan (both required for full functionality)

---

# Bluetooth SDP + HFP Connection Flow (IVI ↔ Phone)

## Scenario
- IVI = Hands-Free (HF) device (client)
- Phone = Audio Gateway (AG)
- ACL link is already established and encrypted

---

# 🔍 1. SDP (Service Discovery Protocol)

## Purpose
IVI discovers:
- Whether phone supports HFP
- Which RFCOMM channel to use

---

## SDP Flow

```
IVI (HF)                               PHONE (AG)
--------------------------------------------------------

L2CAP Connection (PSM 0x0001)  ----->

SDP ServiceSearchAttributeRequest ----->

                                <----- SDP ServiceSearchAttributeResponse
                                       (Contains HFP service record)

<----- Extract:
        - Service Class = Handsfree Audio Gateway
        - RFCOMM Channel Number (e.g., channel 3)
```

---

## Key Notes

- SDP runs over **L2CAP (PSM = 0x0001)**
- IVI searches for:
  - UUID = HFP Audio Gateway
- Response includes:
  - RFCOMM server channel
  - Supported features

---

# 🔗 2. RFCOMM Connection (Control Channel)

## Purpose
Create serial-like control channel for AT commands

---

## RFCOMM Flow

```
IVI (HF)                               PHONE (AG)
--------------------------------------------------------

L2CAP Connection (PSM 0x0003)  ----->

RFCOMM SABM (Channel X)       ----->

                                <----- RFCOMM UA (Unnumbered Acknowledgement)

RFCOMM DLC Established
```

---

## Key Notes

- RFCOMM acts like a **virtual serial port**
- Channel number comes from SDP
- This channel is used for **AT command exchange**

---

# 🎧 3. HFP Service Level Connection (SLC)

## What is SLC?

👉 **Service Level Connection (SLC)** means:

```
Control channel is established (RFCOMM)
AND
Initial AT command negotiation is completed
```

👉 No audio yet — only control plane is ready

---

## SLC Establishment Flow (AT Commands)

```
IVI (HF)                               PHONE (AG)
--------------------------------------------------------

AT+BRSF=HF_Features        ----->

                                <----- +BRSF=AG_Features
                                <----- OK

AT+CIND=?                  ----->

                                <----- +CIND: ("service",...),("call",...)
                                <----- OK

AT+CIND?                   ----->

                                <----- +CIND: current_status
                                <----- OK

AT+CMER=3,0,0,1            ----->

                                <----- OK
```

---

## After this:

```
✔ Service Level Connection (SLC) is established
✔ HF and AG know each other's capabilities
✔ Indicator reporting is enabled
```

---

# 📞 4. What Happens After SLC?

Now system is ready for:

## Incoming Call
```
PHONE → IVI:
RING
+CIEV: call=1
```

## Outgoing Call
```
IVI → PHONE:
ATD<number>;
```

---

# 🔊 5. Audio Connection (SCO/eSCO)

👉 Separate from SLC

```
IVI                                      PHONE
-----------------------------------------------

HCI_Setup_Synchronous_Connection ----->

<----- HCI_Synchronous_Connection_Complete
```

---

## Key Point

- **SLC ≠ Audio**
- SLC = Control plane ready
- SCO = Audio path

---

# ❗ Important Interview Points

- SDP is mandatory to get RFCOMM channel
- RFCOMM carries AT commands
- SLC = RFCOMM + AT negotiation complete
- Audio (SCO) comes AFTER SLC
- HFP has two roles:
  - HF (IVI)
  - AG (Phone)

---

# 🎯 One-Line Answer

> “SLC in HFP means the RFCOMM control channel is established and the mandatory AT command exchange between HF and AG is completed, enabling call control and status reporting, but not the audio path.”

---

# HFP SCO Connection, Codec Negotiation, In-Band Ringtone & Role Switch

## Scenario
- IVI = Hands-Free (HF)
- Phone = Audio Gateway (AG)
- ACL + RFCOMM + SLC already established

---

# 🔊 1. SCO / eSCO Connection Creation

## Purpose
Create **audio link** (voice path) separate from control channel

---

## HCI Flow

```
IVI (HF)                               PHONE (AG)
--------------------------------------------------------

HCI_Setup_Synchronous_Connection ----->

                                <----- HCI_Synchronous_Connection_Complete
```

---

## Parameters (Important)

- Connection_Handle (ACL)
- Packet Type:
  - HV1 / HV2 / HV3 (SCO)
  - EV3 / EV5 (eSCO)
- Bandwidth (TX/RX)
- Latency
- Retransmission effort

---

## Key Notes

- SCO = circuit-switched (fixed slots)
- eSCO = retransmissions allowed (better quality)
- Audio flows directly between controllers (bypasses host mostly)

---

# 🎧 2. Codec Negotiation (HFP Wideband Speech)

## Why needed?
To decide:
- Narrowband (CVSD)
- Wideband (mSBC)

---

## Happens BEFORE SCO

### Step 1: Feature Exchange (during SLC)

```
AT+BRSF exchange → both sides advertise codec support
```

---

### Step 2: Codec Negotiation

```
IVI (HF)                               PHONE (AG)
--------------------------------------------------------

AT+BAC=1,2                ----->   (1=CVSD, 2=mSBC)

                                <----- OK

                                <----- AT+BCS=2   (AG selects codec)

AT+BCS=2 (confirm)        ----->

                                <----- OK
```

---

## Result

```
Codec Selected = mSBC (Wideband)
```

---

## Important

- Codec MUST be selected before SCO setup
- If negotiation fails → fallback to CVSD

---

# 🔔 3. In-Band Ringtone

## What is it?

👉 Ringtone comes from **phone over SCO audio**, not generated locally by IVI

---

## Flow

```
Incoming Call:

PHONE → IVI:
RING
+CIEV: callsetup=1

AG decides:
→ Use In-Band Ring

SCO connection is created EARLY

Audio Path:
Phone ringtone → SCO → IVI speaker
```

---

## Key Difference

| Mode | Ringtone Source |
|------|----------------|
| In-band | Phone |
| Out-of-band | IVI |

---

# ⚠️ 4. Role Switch (Critical Topic)

## Default

- Phone = Master
- IVI = Slave

---

## Why Role Switch?

IVI may want to become **Master** for:
- Better scheduling
- Audio stability
- Power management

---

## HCI Flow

```
HCI_Switch_Role ----->

<----- HCI_Role_Change
```

---

# ⚠️ 5. Complexity with SCO + Role Switch

👉 THIS is where real issues happen

---

## Problem 1: SCO Timing Sensitivity

- SCO uses **fixed time slots**
- Role switch disturbs:
  - Slot timing
  - Scheduling

👉 Result:
```
Audio glitch / drop
```

---

## Problem 2: Controller Limitations

Some controllers:
- ❌ Do not support role switch during SCO
- ❌ Drop SCO link

---

## Problem 3: Race Conditions

```
Incoming call → SCO setup
At same time → Role switch triggered
```

👉 Leads to:
- Connection failure
- No audio
- One-way audio

---

## Problem 4: In-Band Ringtone Impact

Since SCO is created early:

```
Role switch during ringtone
→ Audio path unstable
→ Missing ringtone or distorted sound
```

---

## Problem 5: Wideband Codec Sensitivity

- mSBC requires strict timing
- Role switch → packet loss → decoder issues

---

# 🚨 6. Real-World Issues Seen in IVI

### Issue 1: No Audio After Call Connect
- Cause: Role switch during SCO setup

---

### Issue 2: Ringtone Not Heard
- Cause: In-band ringtone + failed SCO

---

### Issue 3: One-Way Audio
- Cause:
  - Role switch timing mismatch
  - Codec mismatch after reconnection

---

### Issue 4: Audio Glitches
- Cause:
  - eSCO retransmission + role switch conflict

---

# ✅ 7. Best Practices (Industry)

### ✔ Avoid Role Switch During SCO
- Lock role before call

---

### ✔ Perform Role Switch Early
- During ACL stage, NOT during call

---

### ✔ Retry SCO Setup
- If failed due to role switch

---

### ✔ Handle Codec Fallback
- mSBC → CVSD fallback logic

---

### ✔ Synchronize Call State
- Avoid parallel procedures

---

# 🎯 Final Understanding

```
SLC → Codec Negotiation → SCO Setup → Audio Flow
```

- SLC = control ready
- Codec negotiation = audio format decided
- SCO = actual voice path
- Role switch = risky during audio phase

---

# 🧠 Interview One-Liner

> “SCO connection is created using HCI_Setup_Synchronous_Connection after codec negotiation via AT commands. In-band ringtone requires early SCO setup. Role switching during SCO is risky due to strict timing constraints and can cause audio glitches, call failures, or one-way audio, so it is typically avoided once audio setup begins.”

---
