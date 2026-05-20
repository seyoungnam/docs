# Packet Delivery Across Networks

When the destination computer is located in a completely different network, computers can no longer talk directly using just a Layer 2 switch. They need a Router (Default Gateway) to bridge the gap between the two networks.

Here is the basic assumption:

* **Computer A (Network 1):** IP `192.168.1.10` | MAC `AA:AA:AA:AA:AA:AA`
* **Router / Default Gateway:** Network 1 IP `192.168.1.1` | Network 1 MAC `RR:RR:RR:RR:RR:RR`
* **Computer B (Network 2):** IP `10.0.0.20` | MAC `BB:BB:BB:BB:BB:BB`

---

## Part 1: Encapsulation (Inside Computer A)

*Data travels **down** the network layers on Computer A. Because the destination is on a different network, Computer A targets the Router.*

```text
  [ Application ]  --> Generates raw Data
         │
         ▼
  [ Transport ]    --> Adds TCP Header (Ports)          📦 [TCP Segment]
         │
         ▼
  [  Network  ]    --> Adds IP Header (Targeting B)    📦 [IP Packet]
         │
         ▼
  [ Data Link ]    --> Adds Ethernet Header (Targeting Router MAC!) 📦 [Ethernet Frame]
         │
         ▼
  [ Physical  ]    --> Converts Frame to Electrical Bits ⚡ 10101011...

```

### Step 1: The Transport Layer (Data $\rightarrow$ Segment)

* **Action:** The operating system takes raw application **Data** and wraps a **TCP Header** around it.
* **Key Info Added:** Source Port (e.g., `45322`) and Destination Port (e.g., `80`).

### Step 2: The Network Layer (Segment $\rightarrow$ Packet)

* **Action:** The segment moves down to Layer 3, where the OS adds an **IP Header**.
* **Addressing Labels:** The logical end-to-end addresses **never change** during the journey:
       * **Source IP:** `192.168.1.10` (Computer A)
       * **Destination IP:** `10.0.0.20` (Computer B)



### Step 3: The Data Link Layer (Packet $\rightarrow$ Frame)

* **The Crucial Decision:** {++Computer A checks its routing table++} and realizes `10.0.0.20` is **not** on its local network. It decides it must send the packet to its **Default Gateway (Router)**.
* **Action:** {++It attaches an **Ethernet Header** targeting the Router's physical address.++}
* **Addressing Labels:**
       * **Source MAC:** `AA:AA:AA:AA:AA:AA` (Computer A's NIC)
       * **Destination MAC:** `RR:RR:RR:RR:RR:RR` (**The Router's Local MAC**, found via ARP)



### Step 4: The Physical Layer (Frame $\rightarrow$ Electrical Bits)

* **Action:** Computer A's network interface card (NIC) translates the binary frame into raw physical **Bits** and transmits them down the cable toward the router.

---

## Part 2: Routing & Re-Encapsulation (At the Router)

*The Router acts as a bridge between Network 1 and Network 2. It strips the old Layer 2 header and writes a new one.*

```text
  [ Incoming Frame ] --> Strips Network 1 MACs (Dest: RR:RR:RR:RR:RR:RR)
         │
         ▼
  [ Router Layer 3 ] --> Reads IP Header (Dest: 10.0.0.20). Looks up Network 2.
         │
         ▼
  [ Outgoing Frame ] --> Writes Brand New Network 2 MACs (Dest: BB:BB:BB:BB:BB:BB)

```

### Step 5: Decapsulation & Inspection at the Router

* The router receives the bits, reassembles the frame, and checks the **Destination MAC** (`RR:RR:RR:RR:RR:RR`). Seeing it matches its own interface, it strips the Ethernet header away.
* The router's CPU looks at the **Destination IP** (`10.0.0.20`). It realizes this belongs to Network 2, which is plugged into its other interface.

### Step 6: Re-Encapsulation for the New Network

* {==The router keeps the internal IP Packet perfectly intact==}, but {++it must build a **brand new Ethernet Header** so the packet can survive on Network 2++}.
* **The New Addressing Labels:**
       * **Source MAC:** The MAC address of the Router's Network 2 port.
       * **Destination MAC:** `BB:BB:BB:BB:BB:BB` (Computer B's physical MAC).


* The router converts this new frame into bits and shoots them down the wire into Network 2.

---

## 🔓 Part 3: Decapsulation (Inside Computer B)

*The signal arrives at Computer B's network card on Network 2, traveling **up** the layers.*

```text
  [ Application ]  <-- Raw Data Delivered to App! 🎉
         ▲
         │
  [ Transport ]    <-- Strips TCP Header (Verifies Port 80)
         ▲
         │
  [  Network  ]    <-- Strips IP Header (Verifies Destination IP 10.0.0.20)
         ▲
         │
  [ Data Link ]    <-- Strips Ethernet Header (Verifies Destination MAC BB:BB:BB:BB:BB:BB)
         ▲
         │
  [ Physical  ]    <-- Reassembles Bits into Frame

```

### Step 7: Layer 1 & 2 Verification (Bits $\rightarrow$ Frame)

* **Action:** Computer B's network card receives the bits, builds the frame, and verifies the **Destination MAC** (`BB:BB:BB:BB:BB:BB`). Finding a match, it strips the Ethernet header and passes the packet up.

### Step 8: Layer 3 Verification (Frame $\rightarrow$ Packet)

* **Action:** The operating system reads the **Destination IP** (`10.0.0.20`). It matches perfectly, so it strips the **IP Header** away.

### Step 9: Layer 4 Verification & Delivery

* **Action:** The OS processes the **TCP Segment**, verifies the **Destination Port**, strips the **TCP Header**, and delivers the original **Data** straight to the waiting application.
