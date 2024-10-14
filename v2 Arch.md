 # <p align="center"> V2 ARCHITECTURE </p>
---

## **V2 Architecture (Detailed Expansion)**

The **V2 architecture** builds upon the V1 architecture by addressing many of its inefficiencies, primarily focusing on reducing resource consumption, increasing parallelism, and improving latency. Significant changes are introduced in terms of packet processing flow and hardware optimizations.

### **1. High-Level Overview of V2 Architecture**

The V2 architecture resolves key issues found in V1, particularly around resource utilization and latency. It introduces a **pipelined architecture**, improves **BRAM utilization**, and optimizes the **Ethernet Transactor** to better handle high-speed network traffic. Unlike V1, V2 begins processing packets immediately after headers are received, reducing the waiting period before making decisions on packet classification.

### **2. Key Improvements in V2**

- **Pipeline Integration**: V2 allows for parallel processing of packets at different stages (e.g., receiving, firewall classification, and forwarding), whereas V1 processes packets sequentially, leading to delays.
- **Efficient Use of BRAM**: Instead of using registers and flip-flops to store packets, V2 uses **Block RAM (BRAM)** slices. BRAM is more abundant and less resource-intensive, allowing for more efficient storage of frames.
- **Simultaneous Packet Processing**: In V1, the Ethernet transactor stored an entire frame before passing it to the Master Packet Dealer. In V2, the transactor can pass packet data to the MPD as soon as it receives it, which improves the system’s throughput and reduces packet processing time.

---

### **3. Ethernet IP and Ethernet Transactor in V2**

- **Ethernet IP**: The **Ethernet IP** in V2 is configured to use both of its **dual-port buffers** (for RX and TX), allowing it to process multiple packets concurrently. This enhancement improves the system’s ability to handle traffic without delays.
- **Ethernet Transactor**:
  - Unlike in V1, where the transactor stores an entire frame before sending it to the MPD, in V2, the transactor starts forwarding the packet to the MPD word-by-word as soon as it receives the data.
  - This significantly reduces the latency between receiving a packet and sending it to the firewall for classification.
  - **Challenges Addressed**:
    - The V1 transactor architecture caused unnecessary storage and forwarding delays due to sequential processing. V2 solves this by making the transactor part of the pipeline, allowing it to process packets in a **streaming fashion**.

---

### **4. Master Packet Dealer (MPD) in V2**

- **Role**: As in V1, the MPD in V2 manages packet headers and payloads. However, the V2 MPD has been optimized for faster handling of packets.
- **Parallel Processing**: The MPD now processes packet headers as soon as they are available, without waiting for the full packet payload. This increases the overall system throughput.
- **BRAM Storage**: The packet payload is stored in BRAM slices in the **Packet Reference Table (PRT)**, which significantly reduces the utilization of flip-flops and LUTs that were overused in V1.
- **Optimized Interaction**: The MPD can retrieve packet payloads from BRAM and forward them to the transactor immediately after the firewall has marked the packet as safe.

---

### **5. Packet Reference Table (PRT) in V2**

- **Efficient Storage**: V2 leverages BRAM to store packet payloads, making better use of FPGA resources. This resolves the **resource utilization** challenges observed in V1, where registers were used for frame storage, leading to higher LUT and flip-flop usage.
- **Reduced Latency**: By allowing the PRT to work directly with BRAM slices, the system can handle larger packets and more packet data without consuming valuable logic resources.

---

### **6. Firewall in V2**

- **Parallel Firewall Classification**: As soon as the MPD receives the packet header, it sends it to the **firewall** for classification without waiting for the entire payload. This parallelism enables the system to classify packets faster.
- **Bloom Filter Integration**: The firewall continues to use the Bloom filter for quick rule-checking. If a packet is flagged as unsafe, the packet header is passed to the Shakti CPU for deeper analysis.

---

### **7. Shakti CPU in V2**

- **Role**: Similar to V1, the **Shakti CPU** in V2 performs linear searches on flagged packets. However, because the firewall works faster in V2, the Shakti CPU is called upon less frequently, reducing its workload.
- **Compartmentalization and Memory Safety**: The use of the **FIDES** scheme remains critical in V2, ensuring that the Shakti CPU’s memory is compartmentalized, reducing the risk of attacks.

---

### **8. Processing Flow in V2**

1. **Packet Reception**: Ethernet frames are received and stored in **dual-port buffers** in the Ethernet IP.
2. **Parallel Processing**: As soon as the header is available, the Ethernet transactor starts passing it to the MPD. Meanwhile, the payload is stored in the PRT.
3. **Firewall Classification**: The header is immediately sent to the firewall, and the Bloom filter checks for rule matches.
4. **Further Analysis (If Needed)**: If the Bloom filter flags the packet as unsafe, the header is sent to the Shakti CPU, which performs a deeper linear search to check against predefined rules.
5. **Transmission**: Once the packet is classified as safe, the MPD retrieves the payload from the PRT and forwards the full frame to the transactor for transmission.

---

### **9. Challenges Resolved in V2**

- **Reduced Latency**: The pipelining and word-by-word processing of packets significantly reduce latency in V2 compared to V1, where frames had to be fully stored before processing could begin.
- **Better Resource Utilization**: The shift from registers to BRAM for payload storage reduces the consumption of flip-flops and LUTs, making the system more scalable and efficient.
- **Throughput Improvements**: By using both TX and RX buffers and allowing concurrent processing of packets, V2 improves the throughput, ensuring that the system can handle high network traffic without delays.

---

---

This comprehensive breakdown of **V2** and **V3** architectures shows the evolution of the design, from the resource-heavy and less efficient V1 to the highly optimized and tightly integrated V3. Each iteration brings improvements in latency, throughput, and resource efficiency, making V3 the most powerful and scalable architecture in the series.
