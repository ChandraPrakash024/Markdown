# <p align="center"> FIREWALL PROJECT </p>

### 1. **Introduction (Detailed Expansion)**

   - **Cybersecurity Landscape**: The rapid digitization of industries has drastically increased the attack surface for malicious entities. From ransomware to sophisticated state-sponsored attacks, the variety and intensity of cyber-attacks have skyrocketed. Firewalls, which serve as a critical first line of defense, work by filtering incoming and outgoing traffic based on a set of predefined rules.
   - **Role of Firewalls**: Firewalls form a barrier between trusted internal networks and untrusted external ones (like the internet). They analyze packets based on source/destination IP addresses, ports, protocols, and payloads to determine if they should be allowed or blocked.
   - **Challenge in High-Speed Networks**: As data speeds increase (e.g., 1 Tbps), traditional firewalls struggle to keep up, potentially becoming bottlenecks. For instance, at 100 Gbps, with minimum-sized packets (40 Bytes), a firewall would need to handle over 300 million packets per second.
   - **Hardware Firewalls as a Solution**: To meet these demands, hardware firewalls based on FPGAs offer a compelling alternative. FPGAs can parallelize operations, significantly speeding up packet classification and filtering. Their reconfigurable nature also allows fast updates to adapt to new threats or changes in rules.

---

### 2. **Overall Architecture**

   - **Packet Classification Complexity**: Classifying packets based on predefined rule sets is an inherently complex task. As rule sets grow larger, traditional methods like linear search (with a complexity of O(n)) become inefficient.
   - **CAM and TCAM**: Content Addressable Memory (CAM) allows for fast searches by querying memory based on content rather than an address. TCAM extends CAM by enabling ternary operations ('don’t care' conditions). TCAM is frequently used in hardware packet classifiers, although it's expensive and has low storage density.
   - **Algorithmic Approaches**:
     - **Decision Tree-Based Classifiers**: These break down rules into tree structures, making searches faster but at the cost of memory overhead.
     - **Decomposition-Based Classifiers**: These divide packet classification tasks into smaller sub-tasks that are easier to solve.
     - **Bloom Filters**: A probabilistic data structure that can quickly determine if a packet header matches a rule. Bloom filters are memory efficient but can produce false positives, which must be mitigated with further analysis.

---

### 3. **Hardware Bloom Filter**

   - **Bloom Filter Basics**: A Bloom filter is initialized as an array of bits, all set to 0. To add a rule, 'k' hash functions are applied, setting bits at different positions in the bit array. For each incoming packet, its header is hashed using the same 'k' hash functions, and if all the corresponding bits are 1, the packet may match a rule (although false positives are possible).
   - **Jenkins Hash Function**: In your design, you employ the Jenkins hash function for Bloom filtering, which is known for its speed and minimal resource consumption. The use of parallelism and pipelining further reduces processing latency, making the Bloom filter an effective solution for high-speed packet filtering.
   - **False Positives**: Although Bloom filters provide O(1) time complexity, their downside is the occurrence of false positives. When a packet is falsely identified as matching a rule, it undergoes deeper inspection by a traditional linear search method, which is computationally heavier.

---

### 4. **Calculation of False Positivity**

   - The likelihood of false positives increases with the number of rules (n) and decreases as the size of the Bloom filter array (m) grows. However, increasing the size of the array consumes more memory.
   - **Mathematical Analysis**:
     - Probability that a bit is set by a hash function: \(1/m\)
     - Probability that a bit remains 0 after 'n' rules are applied: \((1 - 1/m)^n\)
     - False positive rate: \((1 - (1 - 1/m)^{nk})^k\)
   - **Trade-offs**:
     - Reducing the false-positive rate requires more memory and larger hash functions, but increases system complexity.
     - A balanced configuration needs to be selected to optimize memory usage, processing speed, and acceptable false-positive rates.

---

### 5. **Linear Search and the Role of Shakti CPU**

   - **Bloom Filter + Linear Search Combination**: In scenarios where Bloom filter flagging yields potential false positives, the flagged packets undergo a deeper inspection using a linear search on the Shakti processor. This ensures accuracy while maintaining high performance for the majority of packets.
   - **Why Shakti Modified C-Class?**: The Shakti processor is used due to its incorporation of the FIDES (Fine-grained Compartments in Memory-Safe Languages) scheme. This ensures better security through compartmentalization at the hardware level, restricting what functions can invoke others. By isolating memory spaces for different operations (e.g., packet inspection vs. rule update), it enhances security against cyberattacks exploiting memory vulnerabilities.
   - **Compartmentalization in Shakti**: FIDES creates fine-grained compartments at the function level, reducing the attack surface in the event of a vulnerability. Each function operates in its own memory space, limiting the potential damage from a compromised section of the software.

---

### 6. **V1 Architecture**

   - **Master Packet Dealer (MPD)**: Acts as the central entity managing packet payloads. It interacts with the Ethernet IP to receive frames, extracts headers, and forwards them to the hardware firewall for classification. 
   - **Packet Reference Table (PRT)**: Stores packet payloads and associates them with tags, which are later used for identification when the firewall has classified the packet.
   - **Ethernet Transactor**: Serves as an intermediary between the Ethernet IP and MPD, handling incoming and outgoing frames. In V1, it is configured to use only one TX/RX buffer, limiting performance.

---

### 7. **V2 Architecture Improvements**

   - **Full Pipeline Integration**: The V2 design improves on V1 by allowing packet headers to be processed as soon as they are received, rather than waiting for the entire packet. This parallelization dramatically reduces latency.
   - **Enhanced Use of Ethernet IP**: The V2 architecture leverages the full potential of the Ethernet IP by utilizing its dual-port buffers for receiving and transmitting data simultaneously. This enables the system to operate at 100 Mbps, the maximum capacity of the MII interface.

---

### 8. **V3 Architecture Overhaul**

   - **Custom Ethernet IP**: The limitations of Xilinx’s AXI EthernetLite IP prompted the development of a custom Ethernet IP for V3. This custom IP eliminates unnecessary features (e.g., CRC checks, MAC filtering) and integrates more tightly with the MPD, reducing resource consumption and improving latency.
   - **Online Packet Processing**: The V3 architecture processes packets as they are received. As soon as the packet header arrives, it is sent to the firewall for analysis, allowing for quicker decision-making and packet forwarding. This on-the-fly processing minimizes latency.
   - **Clock Domain Crossing (CDC)**: Due to different clock domains (e.g., 2.5 MHz for PHY, 50+ MHz for FPGA logic), synchronization FIFOs are used to safely transfer data between domains, ensuring system stability.
   - **Latency and Resource Utilization**: V3 achieves a significant reduction in both latency (213 cycles) and resource usage (764 LUTs and 782 registers) compared to V1, where latency was 10642 cycles and resource usage was exponentially higher.

---

### 9. **Dual-Port Testing and Future Developments**

   - **Dual-Port Ethernet Testing**: On the Alinx AX7203B board, the design utilizes two Ethernet ports, enhancing the system's capability to handle bi-directional traffic simultaneously. This is critical for real-world applications where firewalls often manage traffic between multiple network segments.
   - **Challenges with RGMII**: While testing on the Alinx board, autonegotiation caused issues when connecting to devices capable of 1 Gbps speeds. A custom FSM was developed to manually configure the PHY chip to force a lower speed (100 Mbps) to match the system's capabilities.
   - **Future Work**: 
     - Development of Ethernet IP capable of handling 1000 Mbps speeds using the RGMII standard.
     - Exploration of multi-clock domain architectures to further optimize the system's processing capabilities.
     - Hardware-based reconfiguration of the Bloom filter, allowing Shakti to dynamically update rules without halting firewall operations.

---

### 10. **Further Research and Development Directions**

   - **Reducing False Positivity in Bloom Filters**: Various research ideas, such as using XOR-based Boolean expressions to represent rules, were explored but did not significantly improve false-positive rates. Future research could focus on machine learning approaches (e.g., learned Bloom filters) to better approximate rule sets with fewer false positives.
   - **Multi-Clock Domain Architecture**: One potential area for improvement is to allow the firewall to run at a higher clock speed than Shakti. This would allow the firewall to process packets faster, offloading work to Shakti only when deeper inspection is necessary.
   - **Compartmentalization with FIDES**: The development of compartmentalization software to further segment firewall processes (e.g., separating rule configuration from packet analysis) is a critical security improvement that could reduce the impact of successful cyberattacks.
