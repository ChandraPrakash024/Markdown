# <p align="center"> V3 ARCHITECTURE </p>


The **V3 architecture** represents a significant overhaul of the system, aimed at overcoming the limitations found in both V1 and V2. V3 eliminates the reliance on Xilinx’s **AXI EthernetLite IP** and focuses on a **custom Ethernet IP** design, integrated tightly with the rest of the system. This enables even greater throughput, lower latency, and tighter control over resource utilization.

### **1. High-Level Overview of V3 Architecture**

V3 replaces the **Ethernet IP** used in V1 and V2 with a custom-designed Ethernet IP, tailored specifically to meet the requirements of the firewall system. This custom IP is designed to tightly integrate with the **MPD** and the **firewall**, reducing latency and allowing for **on-the-fly processing** of packets.

### **2. Custom Ethernet IP in V3**

- **Custom Design**: In V3, a custom **Ethernet IP** is developed to address the inefficiencies observed with the Xilinx AXI EthernetLite IP used in previous versions.
- **Challenges with Xilinx Ethernet IP**:
  - The Xilinx IP was designed for general-purpose use and included features such as **CRC checks** and **MAC filtering**, which added unnecessary overhead for the firewall application.
  - The IP also did not offer a **promiscuous mode**, which is essential for a firewall, as it needs to inspect all incoming packets without filtering based on MAC addresses.
  - The Xilinx IP’s design increased latency by storing packets and making them available only after they were fully received.
- **Tight Integration**: The custom Ethernet IP in V3 integrates tightly with the MPD, allowing for online processing of packets (i.e., as the packet is received, it is processed and classified).

---

### **3. Master Packet Dealer (MPD) in V3**

- **Enhanced Pipelining**: In V3, the MPD is further optimized to process incoming packets in a fully pipelined fashion. As soon as a packet header is received, it is immediately passed to the firewall for classification.
- **On-the-Fly Packet Processing**: One of the key improvements in V3 is that the MPD no longer waits for an entire packet to be received before processing the header. This allows for real-time processing of packets, reducing latency.
- **Reduced Latency**: The tight integration with the custom Ethernet IP ensures that headers are processed and classified as soon as they are available, without unnecessary storage or delays.

---

### **4. Packet Reference Table (PRT) in V3**

- **More Efficient Use of BRAM**: The PRT in V3 continues to leverage BRAM slices to store packet payloads, but the overall efficiency is further improved due to the tighter integration with the MPD and the custom Ethernet IP.
- **Immediate Action**: Once a packet is classified as unsafe, the MPD can stop the reception of the payload in real-time, preventing the storage of unnecessary data. This further optimizes memory usage and ensures that only safe packets are stored and transmitted.

---

### **5. Firewall in V3**

- **Immediate Classification**: The firewall, now tightly integrated with the custom Ethernet IP, processes packets in real time. As soon as a header is available, it is classified using the **Bloom Filter**.
- **Optimized False Positivity Handling**: V3 continues to use the Bloom filter for initial rule matching. If a false positive occurs, the packet is passed to the **Shakti CPU** for further analysis.
- **Scalability**: V3’s firewall is designed to handle higher data rates and larger rule sets, thanks to the optimized packet processing and reduced memory footprint.

---

### **6. Shakti CPU in V3**

- **Reduced Workload**: The Shakti CPU is now called upon even less frequently, thanks to the faster packet processing in the firewall. When required, it still performs **linear searches** to check flagged packets against predefined rules.
- **Compartmentalization**: The FIDES approach continues to ensure memory safety by compartmentalizing different firewall functions (e.g., rule checking, configuration updates), reducing the risk of vulnerabilities.

---

### **7. Processing Flow in V3**

1. **Packet Reception**: The custom Ethernet IP receives Ethernet frames and immediately starts processing the headers.
2. **Immediate Processing**: As the headers are received, they are sent to the MPD, which forwards them to the firewall for classification.
3. **Firewall Classification**: The firewall checks the headers against predefined rules using the Bloom filter. If safe, the payload is stored in the PRT for transmission.
4. **Transmission**: If the packet is safe, it is forwarded for transmission as soon as the payload is fully received. Unsafe packets are discarded without storing the payload.
5. **Further Inspection**: If the firewall flags a packet as unsafe, it is sent to the Shakti CPU for a deeper analysis.

---

### **8. Challenges Addressed in V3**

- **Custom Ethernet IP**: The design of a custom Ethernet IP eliminates the unnecessary overhead caused by the Xilinx IP. This includes removing unused features (e.g., CRC checks) and enabling **promiscuous mode**, which is crucial for a firewall system.
- **Reduced Latency**: The on-the-fly processing of packets ensures that packet headers are classified as soon as they are received, reducing latency to just **213 cycles**—a dramatic improvement over V1 and V2.
- **Resource Efficiency**: The custom Ethernet IP, combined with optimized BRAM usage in the PRT, reduces the overall resource footprint (764 Slice LUTs in V3 compared to 15839 in V1).

---

### **9. Summary of V3 Architecture**

The V3 architecture represents a significant leap forward in terms of both **performance** and **efficiency**. By moving away from general-purpose Ethernet IPs and developing a custom solution, V3 optimizes the packet processing pipeline, reduces resource usage, and minimizes latency. This architecture sets the stage for high-performance firewall systems capable of handling modern, high-speed networks.
