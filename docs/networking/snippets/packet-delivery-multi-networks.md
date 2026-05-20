# Packet Delivery Across Multiple Networks

When a router performs **NAT (Network Address Translation)**, it doesn't just swap the Layer 2 Ethernet headers—it actually modifies the **Layer 3 IP Header** as well.

This is exactly how your Vagrant VM handles traffic: your container has a private IP (`172.17.0.2`), but to talk to the internet, your VM translates that traffic to its own IP (`10.0.2.15`).

Let's look at how the packet travels when the Router translates Computer A's private IP into a public IP so it can reach Computer B across the internet.

### Scenario Details

* **Computer A (Private Network):** IP `192.168.1.10` | MAC `AA:AA:AA:AA:AA:AA`
* **NAT Router:** Private Side IP `192.168.1.1` | Public Side IP `203.0.113.5` | Router MAC `RR:RR:RR:RR:RR:RR`
* **Computer B (External/Public Network):** IP `8.8.8.8` | MAC `BB:BB:BB:BB:BB:BB`

---

## Part 1: Encapsulation (Inside Computer A)

*Data travels **down** the network layers on Computer A. Computer A uses its private IP and targets the local router.*

```text
  [ Application ]  --> Generates raw Data
         │
         ▼
  [ Transport ]    --> Adds TCP Header (Source Port: 45322)     📦 [TCP Segment]
         │
         ▼
  [  Network  ]    --> Adds IP Header (Src: 192.168.1.10)       📦 [IP Packet]
         │
         ▼
  [ Data Link ]    --> Adds Ethernet Header (Dest: Router MAC)   📦 [Ethernet Frame]
         │
         ▼
  [ Physical  ]    --> Converts Frame to Electrical Bits ⚡ 10101011...

```

### Step 1: The Transport Layer (Data $\rightarrow$ Segment)

* **Action:** The OS wraps the raw **Data** in a **TCP Header**.
* **Key Info Added:** Source Port `45322` and Destination Port `80`.

### Step 2: The Network Layer (Segment $\rightarrow$ Packet)

* **Action:** The OS adds the original private **IP Header**.
    * **Source IP:** `192.168.1.10` (Computer A's private IP)
    * **Destination IP:** `8.8.8.8` (Computer B's public IP)



### Step 3: The Data Link Layer & Physical Layer (Packet $\rightarrow$ Bits)

* **Action:** Computer A realizes `8.8.8.8` is external, so it creates an **Ethernet Header** targeting the gateway.
    * **Source MAC:** `AA:AA:AA:AA:AA:AA` (Computer A)
    * **Destination MAC:** `RR:RR:RR:RR:RR:RR` (Router's Private Port MAC)


* The frame is converted into electrical bits and sent down the cable.

---

## Part 2: NAT & Routing (At the Router)

*The Router strips the Layer 2 header, **rewrites the Layer 3 IP source address**, records the change in a translation table, and builds a new Layer 2 header.*

```text
  [ Incoming Frame ]   --> Strips Network 1 MACs (Dest: RR:RR:RR:RR:RR:RR)
         │
         ▼
  [ NAT Translation ]  --> Changes Source IP from 192.168.1.10 to Public 203.0.113.5
   (IP Header Altered)     Changes Source Port if needed (e.g., keeps 45322)
         │
         ▼
  [ Outgoing Frame ]   --> Writes Brand New Public MACs (Dest: BB:BB:BB:BB:BB:BB)

```

### Step 4: Decapsulation and NAT Table Lookup

* The router receives the bits, verifies the Destination MAC, and strips the Ethernet header.
* It reads the IP header and notes that a private IP (`192.168.1.10`) wants to talk to the public internet.

### Step 5: Modifying the IP Packet (The NAT Magic)

* Because private IPs are not allowed on the public internet, the router modifies the **IP Header itself**:
* It overwrites **Source IP:** `192.168.1.10` $\rightarrow$ **`203.0.113.5`** (The Router's Public IP).
* The Destination IP stays exactly the same (`8.8.8.8`).


* **The NAT Table Entry:** The router saves a temporary note in its memory tracking this exact connection:
| Private Internal IP & Port | Public External IP & Port | Destination IP |
| --- | --- | --- |
| `192.168.1.10:45322` | `203.0.113.5:45322` | `8.8.8.8:80` |



### Step 6: Re-Encapsulation for the Public Internet

* The router attaches a **brand new Ethernet Header** for the public internet space.
    * **Source MAC:** The MAC of the Router's public-facing port.
    * **Destination MAC:** `BB:BB:BB:BB:BB:BB` (Computer B's MAC, or the next hop router's MAC).


* The router transmits the modified packet out to the internet.

---

## 🔓 Part 3: Decapsulation (Inside Computer B)

*The packet arrives at Computer B. To Computer B, it looks like the traffic came entirely from the router.*

```text
  [ Application ]  <-- Raw Data Delivered to App! 🎉
         ▲
         │
  [ Transport ]    <-- Strips TCP Header (Verifies Port 80)
         ▲
         │
  [  Network  ]    <-- Strips IP Header (Sees Source as Public Router IP: 203.0.113.5)
         ▲
         │
  [ Data Link ]    <-- Strips Ethernet Header (Verifies Destination MAC)

```

### Step 7: Layer 1 & 2 Verification

* Computer B reassembles the bits, verifies its Destination MAC (`BB:BB:BB:BB:BB:BB`), and strips the Ethernet header away.

### Step 8: Layer 3 Verification (The Translated View)

* Computer B reads the **IP Header**:
    * **Source IP:** `203.0.113.5` (It thinks the *Router* sent this data!)
    * **Destination IP:** `8.8.8.8` (Matches Computer B)


* Computer B accepts the packet, strips the IP Header, and processes the TCP segment up to the application.

---

### What happens when Computer B replies?

When Computer B sends a response back, the process runs in reverse:

1. Computer B sends a packet with **Destination IP:** `203.0.113.5` (The Router's public IP).
2. The Router receives the packet, checks its **NAT Table**, and looks up who was using port `45322`.
3. It finds the entry matching `192.168.1.10`, changes the Destination IP back to the private address, and forwards it safely back to Computer A!