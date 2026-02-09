# DynamicBandwidth
Okay, let's break down the "SDN-Enabled Dynamic Bandwidth Orchestrator" project idea in more detail. This is a very solid and comprehensive project for a CS student, combining several cutting-edge networking and AI concepts.

### Project Title: SDN-Enabled Dynamic Bandwidth Orchestrator for Intelligent WLAN QoS

### 1. Goal: Design and implement an SDN controller application that centrally manages bandwidth across multiple virtualized APs.

**Explanation:**

*   **SDN Controller Application:** This is the "brain" of your network. Instead of individual Access Points (APs) making their own, isolated decisions, a central software application (your controller) will oversee and direct all bandwidth-related activities across the entire WLAN. This application will be written in Python.
*   **Centrally Manages Bandwidth:** This means your controller will have a global view of the network state (e.g., how many users are connected to each AP, what kind of traffic they're generating, current load on each AP) and will make decisions for all APs. This is a key advantage of SDN over traditional distributed management.
*   **Multiple Virtualized APs:** You won't be working with physical APs directly. Instead, you'll simulate a WLAN environment where each "AP" is a software entity running within a network emulator. This allows you to easily create a complex topology with many APs and users without needing expensive hardware.

### 2. Focus: Implement the paper's user-priority scheme, but extend it with application-aware QoS and intelligent misuse detection using ML.

**Explanation:** This is where you build upon the foundational paper and introduce modern intelligence.

#### a. Implement the Paper's User-Priority Scheme:

*   **User Categorization:** You'll need a mechanism (e.g., a simple database or configuration file within your controller) to assign priorities to users (e.g., "high priority," "low priority"). This could be based on their MAC address, IP address, or even a simulated user ID.
*   **AIFS/CW Adjustment:** Your controller will need to be able to instruct the simulated APs to modify the Arbitration Inter-Frame Space (AIFS) and Contention Window (CW) values for different users.
    *   **High-priority users:** Get lower AIFS/CW values, increasing their chance to transmit.
    *   **Low-priority users:** Get higher AIFS/CW values, making them wait longer.
    *   **How SDN helps:** In a real SDN setup, the controller would send OpenFlow messages or use vendor-specific APIs to configure these parameters on the APs. In Mininet, you'll simulate this control.

#### b. Extend with Application-Aware QoS:

*   **Problem:** The original paper's scheme is purely user-priority based. A low-priority user might be doing a critical video conference, but still gets deprioritized.
*   **Solution:** Your system will need to identify the *type* of application traffic.
    *   **Traffic Classification (using ML):** This is a crucial ML component. Your controller will collect traffic flow data (e.g., source/destination IP/port, packet size, inter-arrival times) from the simulated APs. An ML model (e.g., a supervised classifier trained on a dataset of different application traffic) will analyze this data to determine if a flow is VoIP, video streaming, web browsing, file download, etc.
    *   **Application QoS Mapping:** You'll define QoS requirements for different application types (e.g., VoIP needs very low latency, video needs high bandwidth and moderate latency, file downloads are best-effort).
    *   **Dynamic Prioritization:** The controller will then combine the user's static priority with the application's dynamic QoS requirement to make a more intelligent decision. For example:
        *   High-priority user + VoIP = Highest priority
        *   Low-priority user + VoIP = Elevated priority (above a low-priority user doing a file download)
        *   High-priority user + File Download = Moderate priority (might be deprioritized if a low-priority user is doing VoIP)

#### c. Intelligent Misuse Detection using ML:

*   **Problem:** The original paper's "misuse" detection is vague. How do you define it?
*   **Solution:** Use ML to identify abnormal or undesirable traffic patterns.
    *   **Anomaly Detection (using ML):** Train an unsupervised ML model (e.g., clustering, isolation forest) on "normal" traffic patterns for different user types. Deviations from these patterns could indicate misuse (e.g., a low-priority user suddenly consuming massive bandwidth for an unknown application, or a user exceeding a defined data cap for their priority level).
    *   **Policy Enforcement:** If misuse is detected, the controller will dynamically adjust the AIFS/CW for that specific user, reducing their bandwidth. The reclaimed bandwidth can then be reallocated to other users (as per the original paper's concept, but now with more intelligence).

### 3. Tools: Mininet for network simulation, OpenFlow for SDN, Python for controller logic.

**Explanation:** These are the practical components you'll use.

*   **Mininet:**
    *   **What it is:** A network emulator that creates virtual hosts, switches, and links on a single machine. It's excellent for prototyping SDN concepts.
    *   **How you'll use it:** You'll create a Mininet topology that simulates your WLAN. This will involve:
        *   **Virtual Hosts:** Representing your Wi-Fi clients (laptops, phones).
        *   **Virtual Switches:** Representing your Access Points (APs). Mininet's `OVSKernelSwitch` can be configured to behave like an AP in a simplified manner, and you can attach hosts to it as if they were connecting wirelessly.
        *   **Links:** Connecting your APs to a central wired network (also simulated in Mininet).
        *   **Traffic Generation:** You'll use tools like `iperf`, `hping3`, or custom Python scripts on your virtual hosts to generate different types of traffic (VoIP, video, file transfers) to simulate real-world usage.

*   **OpenFlow for SDN:**
    *   **What it is:** A protocol that allows an SDN controller to directly program the forwarding plane of network switches (or APs in your case).
    *   **How you'll use it:** Your Python controller will communicate with the virtual APs (OpenFlow switches in Mininet) using OpenFlow messages. While OpenFlow doesn't directly control AIFS/CW, you can *simulate* this control. For example, your controller might send a message to the AP saying "for traffic from MAC X, apply a lower priority queue" or "for traffic from MAC Y, apply a higher priority queue." You'll need to define how your simulated APs interpret these OpenFlow messages to adjust their AIFS/CW behavior.

*   **Python for Controller Logic:**
    *   **What it is:** The programming language for your SDN controller.
    *   **How you'll use it:** You'll write your controller using an SDN framework like **Ryu** or **POX** (both Python-based).
        *   **Network Monitoring:** Your controller will use OpenFlow to collect statistics (packet counts, byte counts, flow entries) from the simulated APs.
        *   **AI/ML Integration:** You'll integrate your Python-based ML models (using libraries like `scikit-learn`, `TensorFlow`, or `PyTorch`) directly into your controller application.
        *   **Decision Making:** Your Python code will implement the logic for user priority, application-aware QoS, and misuse detection.
        *   **Policy Enforcement:** Based on AI decisions, your Python controller will construct and send OpenFlow messages back to the simulated APs to modify their forwarding behavior, effectively implementing your dynamic bandwidth management.

### Project Steps (High-Level):

1.  **Literature Review:** Deep dive into the papers we discussed.
2.  **Environment Setup:** Install Mininet, an SDN controller framework (Ryu/POX), and Python ML libraries.
3.  **Network Topology Design:** Define your Mininet topology (e.g., 3 APs, 10-15 clients per AP, a central server).
4.  **Traffic Generation:** Develop scripts to generate realistic traffic patterns (mix of high/low priority users, different application types).
5.  **SDN Controller Development (Core):**
    *   Implement basic OpenFlow communication (connecting to APs, receiving events).
    *   Develop modules for collecting network statistics.
    *   Implement the basic user-priority scheme (AIFS/CW simulation).
6.  **ML Model Development (Traffic Classification & Misuse Detection):**
    *   Collect training data from your simulated network or use public datasets.
    *   Train ML models for traffic classification and anomaly detection.
    *   Integrate these models into your Python controller.
7.  **Intelligent Orchestration Logic:**
    *   Combine user priority, application QoS (from ML classifier), and misuse detection (from ML anomaly detector) to make dynamic bandwidth decisions.
    *   Translate these decisions into OpenFlow actions for your simulated APs.
8.  **Evaluation:**
    *   Define metrics (throughput, latency, jitter, packet loss for different user/application types).
    *   Run simulations with and without your orchestrator.
    *   Compare performance against the original paper's scheme and a static baseline.
    *   Analyze how effectively your system handles misuse and prioritizes critical traffic.

This project offers a fantastic opportunity to explore the practical application of AI and SDN in a highly relevant and challenging domain.
