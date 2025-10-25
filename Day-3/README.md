# 🌐 OSI MODEL & TCP/IP SUITE

## 🧩 What is a Networking Model?
A **networking model** categorizes and provides a structure for how networking protocols and standards operate.  
These models describe how data travels from one device to another.

> **Protocols** are sets of logical rules that define how network devices and software communicate.

---

## 🏗️ OSI Model (Open Systems Interconnection)
A **conceptual model** that categorizes and standardizes the different functions in a network.

- Created by the **International Organization for Standardization (ISO)**  
- Divides networking functions into **7 layers**
- Each layer performs specific tasks and communicates with the layers directly above and below it.

<img width="566" height="463" alt="Screenshot 2025-10-24 092819" src="https://github.com/user-attachments/assets/e361a37b-4ee8-4b4c-a58e-fa8394ac1601" />


---

### 🔽 Data Flow Processes

| Process | Description |
|----------|--------------|
| **Encapsulation** | As data moves **down** the OSI layers (from Application → Physical), headers and trailers are **added**. |
| **De-Encapsulation** | As data moves **up** the layers (from Physical → Application), headers and trailers are **removed**. |
| **Same-Layer Interaction** | Communication between the **same layer** on different devices. |

---

### 🧠 Mnemonic for OSI Layers
**"All People Seem To Need Data Processing"**

| Layer # | Name | Mnemonic Word |
|----------|------|----------------|
| 7 | Application | All |
| 6 | Presentation | People |
| 5 | Session | Seem |
| 4 | Transport | To |
| 3 | Network | Need |
| 2 | Data Link | Data |
| 1 | Physical | Processing |

---

## 🧱 OSI Layer Overview

### **Layer 7 – Application**
- Closest to the **end user**
- Interacts directly with software applications
- Protocol examples: **HTTP**, **HTTPS**, **SMTP**, **FTP**
- Functions:
  - Identifies communication partners
  - Synchronizes communication sessions

---

### **Layer 6 – Presentation**
- Translates data between **application** and **network** formats  
- Handles **data encryption, compression, and translation**
- Example: converting EBCDIC to ASCII

---

### **Layer 5 – Session**
- Controls **dialogues (sessions)** between communicating hosts  
- Establishes, manages, and terminates sessions  
- Example: Login sessions, streaming sessions

> 🧑‍💻 **Note:**  
> Network engineers rarely work with Layers 5–7 directly; these are more relevant for **software/application developers**.

---

### **Layer 4 – Transport**
- Provides **end-to-end (host-to-host)** communication
- Segments and reassembles data for reliable delivery
- Adds a **Layer 4 header**, creating a **SEGMENT**

<< DATA + L4 Header >> → SEGMENT


**Key Protocols:**  
- **TCP (Transmission Control Protocol)** – reliable, connection-oriented  
- **UDP (User Datagram Protocol)** – faster, connectionless

---

### **Layer 3 – Network**
- Provides connectivity between hosts on **different networks**
- Handles **logical addressing (IP addresses)** and **path selection (routing)**
- Adds a **Layer 3 header**, creating a **PACKET**

<< DATA + L4 Header + L3 Header >> → PACKET


**Devices:** Routers  
**Protocol Example:** IP (Internet Protocol)

---

### **Layer 2 – Data Link**
- Provides **node-to-node** connectivity (e.g., PC ↔ Switch)
- Defines **data framing** and detects/corrects **physical errors**
- Uses **MAC (Media Access Control)** addressing
- Adds a **Layer 2 header and trailer**, creating a **FRAME**

<< L2 Trailer + DATA + L4 Header + L3 Header + L2 Header >> → FRAME


**Devices:** Switches  
**Examples:** Ethernet, PPP

---

### **Layer 1 – Physical**
- Defines the **physical medium** for data transfer  
- Converts digital bits into **electrical, optical, or radio** signals
- Specifies:
  - Voltage levels  
  - Transmission distances  
  - Connectors and cable types

**Devices:** Cables, hubs, NICs, repeaters

<img width="826" height="600" alt="image" src="https://github.com/user-attachments/assets/f31297ad-9359-4c32-b523-bd5f73f58b9e" />

<img width="835" height="521" alt="image" src="https://github.com/user-attachments/assets/76e73213-0cc9-40cf-ae62-a7bd5776f8e3" />



---

## 🧮 OSI Model – Protocol Data Units (PDUs)

| OSI Layer | PDU Name | Protocol Data Added |
|------------|-----------|--------------------|
| 7–5 | Data | — |
| 4 | Segment | Layer 4 Header |
| 3 | Packet | Layer 3 Header |
| 2 | Frame | Layer 2 Header + Trailer |
| 1 | Bits | 0s and 1s Transmission |

### 📦 PDU Encapsulation Process
<< L2 Trailer + DATA + L4 Header + L3 Header + L2 Header >>


---

## 🖧 Visual Diagram – OSI Encapsulation Flow

| Layer # | Name         | Description / Function                |
|----------|--------------|---------------------------------------|
| 7 | Application   | User interacts here                      |
| 6 | Presentation  | Data formatting / encryption              |
| 5 | Session       | Session control / communication           |
| 4 | Transport     | TCP / UDP headers (Segments)              |
| 3 | Network       | IP headers (Packets)                      |
| 2 | Data Link     | MAC headers / trailers (Frames)           |
| 1 | Physical      | Bits transmitted over medium              |


---

## 🌍 TCP/IP Suite

A **conceptual model** and **set of communication protocols** used across the Internet and most modern networks.

- Known as **TCP/IP** because of its core protocols:
  - **TCP** (Transmission Control Protocol)
  - **IP** (Internet Protocol)
- Developed by the **U.S. Department of Defense (DARPA)**
- Similar to the OSI model but with **fewer layers**

  <img width="774" height="505" alt="image" src="https://github.com/user-attachments/assets/79ed95f9-96ff-4f75-a934-dddf3f8ee80b" />


> 🧠 **Note:**  
> OSI is theoretical — TCP/IP is the **model in practical use** today.  
> However, OSI still influences how engineers **think and communicate** about networking.

---

## 📊 TCP/IP Model vs OSI Model

| TCP/IP Layer | Corresponding OSI Layers | Description |
|---------------|--------------------------|--------------|
| **Application** | 7 – Application, 6 – Presentation, 5 – Session | User-facing software and data representation |
| **Transport** | 4 – Transport | End-to-end communication (TCP/UDP) |
| **Internet** | 3 – Network | Logical addressing and routing (IP) |
| **Network Access** | 2 – Data Link, 1 – Physical | Hardware, transmission media, frame transmission |

---

## 🔁 Layer Interactions

### Adjacent-Layer Interactions
Communication between **different layers** on the **same host**.

<img width="1036" height="492" alt="image" src="https://github.com/user-attachments/assets/d1b2e8f1-b336-4631-a0d5-111b31277290" />


**Example:**
Layers 5–7 → send data → Layer 4
Layer 4 adds header → creates SEGMENT


### Same-Layer Interactions
Communication between the **same layer** on **different hosts**.
<img width="960" height="640" alt="image" src="https://github.com/user-attachments/assets/df0170c2-f2fe-44a5-a4c5-12698232a2a7" />

**Example:**
Application layer of YouTube’s server ↔ Application layer of your web browser


📘 **Summary:**
- The **OSI Model** helps conceptualize how data travels across a network.  
- The **TCP/IP Suite** defines how it actually happens in real-world systems.  
- Understanding both allows network engineers to **troubleshoot**, **design**, and **optimize** communication systems efficiently.
