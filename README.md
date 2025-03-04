# micro-ROS ESP32 Multi-Array Communication Example

This project demonstrates how to use micro‑ROS on an ESP32 to both publish and subscribe to multi‑array messages using the `UInt8MultiArray` message type. The ESP32 connects via WiFi to a micro‑ROS agent and communicates with external hardware via a secondary serial interface (Serial2).

---

## Table of Contents

- [Overview](#overview)
- [System Architecture](#system-architecture)
- [Code Features](#code-features)
- [How It Works](#how-it-works)
- [Dependencies and Useful Links](#dependencies-and-useful-links)
- [Setup and Running](#setup-and-running)
- [License](#license)

---

## Overview

- **WiFi & micro‑ROS Setup:** The ESP32 connects to a WiFi network and initializes a micro‑ROS node to communicate with a remote micro‑ROS agent.
- **Publisher:** Publishes a 32‑byte `UInt8MultiArray` message on the `/current_pose` topic. Data is sourced from Serial2 or generated as dummy data if none is available.
- **Subscriber:** Listens on the `/desired_theta` topic for a 4‑byte `UInt8MultiArray` message. A manually allocated static buffer ensures proper memory initialization.
- **Data Processing:** When a new 4‑byte message is received, an 8‑byte packet is built—including a header, the data, a checksum, and terminators—and then transmitted via Serial2.
- **Visual Feedback:** The onboard LED blinks upon successful data transmission.

---

## System Architecture

Below is a high-level diagram illustrating the system architecture:

```mermaid
flowchart TD
    A[ESP32 Board] -->|WiFi| B[micro-ROS Agent]
    A --> C[Serial2 Interface]
    C --> D[External Hardware]
    B --> E[ROS 2 Ecosystem]
    
    subgraph Micro-ROS Node on ESP32
      F[Publisher: /current_pose (32-byte UInt8MultiArray)]
      G[Subscriber: /desired_theta (4-byte UInt8MultiArray)]
      F & G --> H[Executor]
    end
```

*Figure 1: System Architecture*

---

## Code Features

### WiFi & micro‑ROS Setup
- **WiFi Connection:** Uses provided SSID and password to connect.
- **micro‑ROS Transport:** Configured for WiFi to communicate with the micro‑ROS agent.

### Publisher
- **Topic:** `/current_pose`
- **Message:** A 32‑byte `UInt8MultiArray` message.
- **Buffer:** Uses a statically allocated 32‑byte buffer to store data read from Serial2.

### Subscriber
- **Topic:** `/desired_theta`
- **Message:** A 4‑byte `UInt8MultiArray` message.
- **Memory Management:** Manually assigns a 4‑byte static buffer to avoid dynamic allocation issues.

### Data Processing and Packet Construction
- **Packet Structure:**
  - **Byte 0:** Header (`0xAA`)
  - **Bytes 1–4:** Data from the subscriber
  - **Byte 5:** XOR checksum (computed over bytes 0–4)
  - **Bytes 6–7:** Terminators (`0xEF`)
- **Visual Indicator:** Blinks LED to signal packet transmission.

---

## How It Works

1. **Initialization:**
   - Connects to WiFi.
   - Configures micro‑ROS WiFi transport.
   - Initializes micro‑ROS support, node, publisher, and subscriber.
   - For the publisher, a 32‑byte buffer (`rxBuffer`) is allocated.
   - For the subscriber, a static 4‑byte buffer (`sub_data_buffer`) is assigned to the message data.

2. **Main Loop:**
   - **Publisher Section:**  
     Reads bytes from Serial2. Once 32 bytes are collected (or if no data is received for 5 seconds, dummy data is generated), the data is published on `/current_pose`.
   - **Subscriber Processing:**  
     The executor spins to check for new incoming messages. When a valid 4‑byte message is received on `/desired_theta`, the callback copies the data into a local buffer.
   - **Packet Construction:**  
     On detecting new data, an 8‑byte packet is built and sent out via Serial2, and the onboard LED blinks.

For more detailed information on how micro‑ROS handles message memory (particularly for sequences), see the [micro‑ROS Memory Management Documentation](https://docs.vulcanexus.org/en/iron/rst/tutorials/micro/memory_management/memory_management.html).

---

## Dependencies and Useful Links

- **[micro-ROS Official Website](https://micro.ros.org)**
- **[micro-ROS Arduino Library on GitHub](https://github.com/micro-ROS/micro_ros_arduino)**
- **[ESP32 Arduino Core](https://github.com/espressif/arduino-esp32)**
- **[WiFi Library for ESP32](https://github.com/espressif/arduino-esp32/tree/master/libraries/WiFi)**
- **[micro-ROS Memory Management Documentation](https://docs.vulcanexus.org/en/iron/rst/tutorials/micro/memory_management/memory_management.html)**

---

## Setup and Running

1. **Install ESP32 Board Support:**  
   Follow the instructions [here](https://github.com/espressif/arduino-esp32) to add ESP32 support to your Arduino IDE.

2. **Install micro‑ROS Arduino Library:**  
   Clone or download the library from the [micro-ROS Arduino GitHub Repository](https://github.com/micro-ROS/micro_ros_arduino).

3. **Configure the Code:**  
   Update the WiFi credentials (`wifi_ssid`, `wifi_password`) and micro‑ROS agent settings (`agent_ip`, `agent_port`).

4. **Upload the Code:**  
   Compile and upload the code to your ESP32.

5. **Run the micro‑ROS Agent:**  
   On your host machine, run the micro‑ROS agent (for example, using `micro_ros_agent`) to establish communication with the ESP32 node.

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

Feel free to contribute, open issues, or suggest improvements. Contributions and enhancements are welcome!
