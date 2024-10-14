# <p align="center"> V1 ARCHITECTURE </p>

### **1. High-Level Overview of V1 Architecture**

The V1 architecture is structured around several key modules working in harmony to receive packets, analyze them for rule violations, and then either forward or block them based on predefined rules. At the core of this architecture are the **Master Packet Dealer (MPD)**, the **Ethernet IP**, the **Packet Reference Table (PRT)**, and the **Shakti CPU**. The architecture follows a sequential process:

1. **Ethernet frames** are received from the network via the Ethernet IP.
2. The frame’s payload and header are stored and processed in different units.
3. The packet headers are sent to the firewall for classification.
4. Packets flagged as unsafe are further processed through a **linear search** in the Shakti CPU to check against predefined rules.
5. If safe, the packet is forwarded; if unsafe, it is dropped.

Now, let's break down each module in detail.

---

### **2. Ethernet IP**

- **Role**: The **Ethernet IP** is responsible for handling incoming and outgoing Ethernet frames. It interfaces with the physical Ethernet layer (PHY) to receive and transmit data.
- **Functionality**:
  - The **Ethernet IP** consists of two dual-port BRAMs (one for RX and one for TX), which temporarily store Ethernet frames received from or transmitted to the physical layer.
  - The IP uses the AXI4-Lite interface for communication with the **Ethernet Transactor** and the **Master Packet Dealer (MPD)**.
  - **RX (Receive)**: The IP continuously polls the physical interface to detect any incoming packets. Once a packet is received, it stores it in the RX BRAM.
  - **TX (Transmit)**: The IP transmits packets by first polling the physical interface to check if it is ready for transmission. If available, it sends the packet stored in the TX BRAM.

### **3. Ethernet Transactor**

- **Role**: Acts as a bridge between the Ethernet IP and the Master Packet Dealer. It retrieves Ethernet frames from the Ethernet IP and forwards them to the MPD and, eventually, the firewall.
- **Functionality**:
  - Upon receiving an Ethernet frame, the transactor extracts the packet header and forwards it to the MPD, while the payload is stored separately in the Packet Reference Table (PRT).
  - The **Ethernet Transactor** is also responsible for handling communication with the firewall and ensuring that the correct packet headers are sent for rule checks.
  - **Challenges**:
    - In V1, the **Ethernet Transactor** stores the entire frame (both header and payload) in registers before forwarding them to the MPD. This design, while simple, creates unnecessary duplication and increases resource utilization, leading to inefficiency.
    - The transactor uses only **one TX/RX buffer** (even though the Ethernet IP has two). This limits parallel processing, reducing the architecture’s ability to handle packets quickly.

---

### **4. Master Packet Dealer (MPD)**

- **Role**: The MPD is the central hub for managing the packet payloads and interacting with both the hardware firewall and the Shakti CPU.
- **Functionality**:
  - Upon receiving an Ethernet frame from the transactor, the MPD splits the frame into **header** and **payload**.
  - **Header Processing**: The header is passed to the **Firewall** for initial rule checks.
  - **Payload Storage**: The payload is stored in the **Packet Reference Table (PRT)**, along with a tag to uniquely identify it. The tag is associated with the payload for later retrieval.
  - After the firewall checks, if the packet is deemed safe, the MPD retrieves the payload from the PRT (using the tag) and forwards it back to the transactor for transmission.
  - If the packet is unsafe, it is sent to the **Circular Buffer** for further inspection.

---

### **5. Packet Reference Table (PRT)**

- **Role**: The PRT stores the payload of incoming Ethernet frames, allowing the system to retrieve and use it after the firewall has completed its classification.
- **Structure**:
  - The PRT acts as a lookup table where each payload is assigned a **tag** (a unique identifier). This tag is referenced later to retrieve the payload once a classification decision is made.
  - For each incoming packet, the payload is stored in the PRT, while the header is sent to the firewall for classification.
- **Challenges**:
  - In V1, the entire payload is stored before any firewall classification is done. If a packet is later found to be unsafe, the storage in PRT is wasted, as the payload is discarded. This results in inefficiency, especially in high-traffic environments.

---

### **6. Circular Buffer**

- **Role**: This buffer stores the headers of packets flagged by the firewall as "potentially unsafe" for further analysis.
- **Functionality**:
  - The **Circular Buffer** is a FIFO buffer, ensuring that headers of flagged packets are sent to the Shakti CPU for further analysis in the order they arrive.
  - Once a header is processed by the Shakti CPU, a rule-based decision is made on whether to accept or reject the packet.

---

### **7. Shakti CPU**

- **Role**: The **Shakti CPU** plays a pivotal role in performing deeper inspection and analysis of packets flagged as unsafe by the firewall.
- **FIDES and Security**: The Shakti processor uses the **FIDES** scheme for fine-grained compartmentalization, enhancing security. Each function (e.g., packet analysis, rule updates, or system control) operates in isolated memory compartments, reducing the attack surface in case of a breach.
- **Linear Search for Rule Matching**:
  - Packets flagged as unsafe are further analyzed by comparing their headers to predefined firewall rules stored in the Shakti CPU.
  - This process uses a **linear search** to find a rule match, though linear searches are slower than other methods, such as decision trees or Bloom filters.
- **Actions Based on Rule Matching**:
  - If the rule matching determines the packet is unsafe, its associated payload in the PRT is discarded.
  - If the packet is safe, the Shakti CPU informs the MPD, which retrieves the payload from the PRT and forwards it for transmission.

---

### **8. Processing Flow in V1 Architecture**

Here’s a step-by-step flow of how the V1 architecture processes a packet:

1. **Packet Reception**: An Ethernet frame arrives from the network and is received by the **Ethernet IP**, which stores the frame in its RX buffer.
2. **Frame Extraction**: The **Ethernet Transactor** retrieves the frame from the Ethernet IP. The payload is separated and stored in the PRT, and the header is forwarded to the firewall.
3. **Firewall Classification**:
   - The packet header is compared against firewall rules using a **Bloom Filter**.
   - If the packet is deemed safe, the MPD retrieves the payload from the PRT and forwards the complete frame for transmission back to the Ethernet IP.
   - If the packet is flagged as unsafe, it is sent to the **Circular Buffer** for further inspection.
4. **Further Analysis (Shakti CPU)**:
   - The **Shakti CPU** retrieves flagged packet headers from the Circular Buffer and performs a linear search to match the header against predefined rules.
   - If a rule match indicates the packet is unsafe, the payload in the PRT is discarded. If safe, the MPD is informed, and the packet is forwarded.
5. **Transmission**: If the packet is safe, it is passed to the **Ethernet Transactor**, which sends it to the Ethernet IP for final transmission back to the network.

---

### **9. Challenges in V1 Architecture**

- **High Resource Utilization**: The use of registers and flip-flops to store entire frames in the transactor leads to high LUT and register consumption, making the architecture less scalable for high-speed data processing.
- **Limited Parallelism**: The Ethernet transactor is designed to use only one TX and one RX buffer at a time, which limits the system’s ability to handle multiple packets concurrently.
- **Inefficient Storage Mechanism**: Storing packet payloads in the PRT before any classification results in wasted resources when unsafe packets are discarded. Additionally, waiting for the full frame to arrive before processing the header increases latency.
- **Latency**: The linear search performed by the Shakti CPU for rule matching is slow, especially in high-traffic environments where many packets may need deeper inspection. The lack of pipelining between frame reception, firewall checking, and rule matching further contributes to high latency.

---

### **10. Summary of V1 Architecture**

The V1 architecture establishes a foundation for hardware-based firewalls by integrating the Shakti CPU, MPD, and Ethernet IP for packet processing. However, its limitations in resource utilization, inefficiency in storing unnecessary packet payloads, and the bottleneck created by linear searches indicate areas that could be improved in later iterations.

In subsequent versions (like V2 and V3), many of these inefficiencies are addressed through architectural improvements such as better use of BRAMs, pipeline processing, and enhanced parallelism, reducing resource overhead and improving system throughput.
