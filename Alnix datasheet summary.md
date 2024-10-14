# Alinix Datasheet Summary

### 1. **FPGA Development Board Introduction**
   - **Overview**: The AX7203B FPGA development platform follows a core board + carrier board design model. This modular design allows users to separately develop high-speed data processing features (core board) and interact with external systems (carrier board).
   - **Core Board**: The core board, which includes the Xilinx Artix-7 FPGA (XC7A200T), two DDR3 memory modules, and QSPI flash, is responsible for data processing and storage. It supports high-speed memory access and data bandwidth up to 25Gbps.
   - **Carrier Board**: The carrier board expands functionality with interfaces like PCIe x4, HDMI, Gigabit Ethernet, UART, SD card slot, and various I/O headers for industrial control, high-speed video, and data communication tasks.

---

### 2. **AC7200 Core Board Introduction**
   - **Core Board Features**: The AC7200 is a high-performance FPGA core board, using the Artix-7 series XC7A200T FPGA. It is designed for high-speed data communication, video processing, and data acquisition. The board integrates two 4Gbit DDR3 memory modules and supports PCIe data communication with GTP transceivers.
   - **IO Expansion**: The board exposes 180 IO pins at 3.3V and 15 IO pins at 1.5V, plus four high-speed GTP transceivers for high-bandwidth communication.
   - **Size**: The core board is compact (45x55 mm), making it suitable for secondary development and embedded system applications.

---

### 2.1 **FPGA Chip (XC7A200T)**
   - **Chip Overview**: The Xilinx Artix-7 XC7A200T FPGA is a mid-range, low-power device with 215,360 logic cells, 269,200 CLB flip-flops, 13,140 KB block RAM, and 740 DSP slices.
   - **Transceivers & PCIe**: It supports PCIe Gen2 and has four GTP transceivers, each supporting up to 6.6Gbps, ideal for high-speed data tasks like PCIe x4.
   - **Power Management**: The FPGA has multiple power rails, such as VCCINT (core), VCCBRAM, VCCAUX (auxiliary), and VCCO (I/O banks), which must be powered in a specific sequence for proper operation.

---

### 2.2 **Active Differential Crystal**
   - **Clock Generators**: The board includes two active differential crystal oscillators, which provide the necessary clock signals for the FPGA and high-speed transceivers. A 200MHz crystal drives the FPGA’s system clock, while a 125MHz crystal is used for the GTP transceiver reference clock.

---

### 2.3 **200MHz Active Differential Clock**
   - **System Clock**: The 200MHz active differential crystal is the main system clock for the FPGA, connected to its global clock pins (R4 and T4). This clock can be manipulated via the FPGA’s internal PLLs (Phase-Locked Loops) and DCMs (Digital Clock Managers) to generate different clock frequencies as required by user designs.

---

### 2.4 **148.5MHz Active Differential Crystal**
   - **GTP Transceiver Clock**: This 148.5MHz differential crystal provides a reference clock for the FPGA’s GTP transceivers, enabling high-speed serial communication via the transceiver modules. It ensures stable high-speed data transmission for applications like PCIe and fiber-optic communication.

---

### 2.5 **DDR3 DRAM**
   - **Memory Configuration**: The core board features two Micron DDR3 DRAM chips, each 4Gbit (512MB) in capacity. Together, they form a 32-bit wide data bus that allows high-speed memory read/write operations between the FPGA and the DDR3 memory.
   - **Data Rate**: The memory runs at 800MHz (1600Mbps data rate), with careful PCB design ensuring signal integrity through differential routing and impedance matching.
   - **Use Cases**: The large memory bandwidth is suitable for applications requiring high-speed data buffering and real-time processing, such as video processing or data acquisition.

---

### 2.6 **QSPI Flash**
   - **Flash Memory**: The board includes a 128Mbit (16MB) QSPI flash memory, used for non-volatile storage. This memory is typically used to store boot images for the FPGA, including bitstreams, software applications, and configuration files. 
   - **FPGA Interface**: The QSPI flash connects to dedicated pins on BANK0 and BANK14 of the FPGA, allowing for fast data access during the boot process.

---

### 2.7 **LED Light on Core Board**
   - **LED Indicators**: Three red LEDs are available: 
     - **Power LED (PWR)**: Lights up when the board is powered.
     - **Configuration LED (DONE)**: Lights up when the FPGA is properly configured.
     - **User LED**: Connected to an IO pin on BANK34, this LED can be controlled programmatically by the user for debugging or signaling purposes.

---

### 2.8 **Reset Button**
   - **Manual Reset**: A reset button connected to BANK34 allows users to manually reset the FPGA. When pressed, the button drives a low signal to the reset pin, reinitializing the FPGA design.

---

### 2.9 **JTAG Interface**
   - **Debugging and Programming**: A 6-pin JTAG interface (2.54mm pitch) is provided for downloading FPGA configurations and debugging the system. It interfaces with standard JTAG tools like Xilinx Vivado for easy development and testing.

---

### 2.10 **Power Interface on the Core Board**
   - **Standalone Power**: The core board has a 2-pin power interface (J3) for standalone operation, powered by 5V DC. Care must be taken to avoid powering the board from both the carrier and the J3 interface simultaneously to prevent electrical conflicts.

---

### 2.11 **Board to Board Connectors**
   - **High-Speed Connectors**: Four 80-pin connectors are used to interface the core board with the carrier board, allowing high-speed data and signal communication. The differential routing of these connectors ensures signal integrity for high-speed tasks like PCIe.

---

### 2.12 **Power Supply**
   - **Power Rails**: The board derives its power from a DC5V source, which is converted to 1.0V, 1.5V, 1.8V, and 3.3V power rails to supply the FPGA, DDR3, and GTP transceivers. Voltage regulation is managed by four DC/DC converters (TLV62130RGT), each supplying up to 3A of current.

---

### 3. **Carrier Board**

---

### 3.1 **Carrier Board Introduction**
   - **Overview**: The carrier board extends the core board’s functionality by providing multiple peripheral interfaces, including PCIe x4, dual Gigabit Ethernet, HDMI (input and output), USB-UART, and expansion headers for additional IO modules.

---

### 3.2 **Gigabit Ethernet Interface**
   - **Dual Gigabit Ethernet Ports**: The carrier board features two Gigabit Ethernet ports using the JL2121 GPHY chips, which support 10/100/1000 Mbps transmission. The Ethernet PHYs communicate with the FPGA via the RGMII interface, and the JL2121 chips are auto-configurable for various network conditions.
   - **FPGA Communication**: The FPGA interacts with the Ethernet PHY via dedicated RGMII pins, with a clock signal (125MHz) used for data transfer at Gigabit speeds.

---

### 3.3 **PCIe x4 Interface**
   - **High-Speed Data Transfer**: The PCIe x4 interface connects directly to the GTP transceivers of the FPGA, enabling high-speed data transfers to a host PC. Each lane can handle data rates up to 5Gbps, with a reference clock of 100MHz provided by the PCIe slot.

---

### 3.4 **HDMI Output Interface**
   - **HDMI Transmission**: The HDMI output uses the SIL9134 HDMI encoding chip, capable of transmitting video at resolutions up to 1080p@60Hz. This also supports 3D video output, making it suitable for high-quality video applications.
   - **FPGA Control**: The SIL9134 is controlled via I2C by the FPGA, with additional control for signal timing like horizontal sync (HS) and vertical sync (VS).

---

### 3.5 **HDMI Input Interface**
   - **HDMI Reception**: The HDMI input interface is handled by the SIL9013 HDMI decoder chip, supporting 1080p@60Hz video. It allows the FPGA to capture video streams in various formats for real-time video processing applications.
   - **Control**: Similar to the HDMI output, the input interface is controlled via I2C from the FPGA.

---

### 3.6 **SD Card Slot**
   - **Storage Interface**: The SD card slot supports MicroSD cards in both SPI and SD modes, making it a convenient option for external storage or firmware updates.

---

### 3.7 **USB to Serial Port**
   - **Serial Communication**: A USB-to-UART interface based on the CP2102GM chip allows for serial communication between the FPGA and a PC, typically used for debugging. The interface uses the MINI USB format for connectivity.

---

### 3

.8 **EEPROM 24LC04**
   - **Non-Volatile Storage**: The carrier board includes a 4Kbit EEPROM, connected via I2C, which can be used for small-scale data storage, such as configuration settings or calibration data. 

---

### 3.9 **Expansion Header**
   - **Additional IO**: Two 40-pin expansion headers allow users to connect custom modules or peripherals to the FPGA. The headers provide power (3.3V and 5V), ground, and 34 IO signals for user-defined functionality.

---

### 3.10 **JTAG Interface**
   - **JTAG Debugging**: A JTAG interface on the carrier board allows for FPGA programming and debugging. The design includes protection diodes to prevent damage from hot-swapping during JTAG cable connection.

---

### 3.11 **XADC Interface**
   - **Analog Input**: The XADC interface expands the analog-to-digital conversion capability of the FPGA by providing three differential ADC input channels. These channels connect to the FPGA’s internal 12-bit XADC for high-precision analog signal processing.

---

### 3.12 **Keys**
   - **User-Configurable Buttons**: The carrier board includes two user-configurable buttons connected to the FPGA, typically used for reset, mode selection, or other custom functions. These buttons are active-low and can be programmed via the FPGA.

---

### 3.13 **LED Light**
   - **User LEDs**: Four user-controllable LEDs are connected to the FPGA IOs. These LEDs provide visual feedback and can be toggled via custom logic programmed into the FPGA.

---

### 3.14 **Power Supply**
   - **Carrier Board Power**: The carrier board is powered by a 12V input, which is converted to lower voltages (+5V, +3.3V, +1.8V) via the ETA1471 DC/DC power chip. Power can also be supplied from a PCIe slot or an ATX chassis power supply.

---

This detailed breakdown gives an in-depth explanation of each section, covering both the core and carrier boards of the AX7203B FPGA development platform. It highlights key features, configurations, and use cases for each component.
