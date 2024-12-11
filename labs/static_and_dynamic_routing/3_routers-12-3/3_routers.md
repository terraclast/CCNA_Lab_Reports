# Static Routing Lab: 3-Router Network

## Topology Overview

Router A connects to Router B and Router C. Each router has a unique LAN network.

**Goal:** Configure static routes to enable full communication between all devices.

---

## IP Address Plan

- **Router A**:
  - LAN: 192.168.1.0/24
  - WAN (to Router B): 10.0.0.1/30
  - WAN (to Router C): 10.0.0.5/30

- **Router B**:
  - LAN: 192.168.2.0/24
  - WAN (to Router A): 10.0.0.2/30

- **Router C**:
  - LAN: 192.168.3.0/24
  - WAN (to Router A): 10.0.0.6/30

---

## Port Connections:

| Router | Interface | Linked Endpoint | IP address/mask |
|++++++++|+++++++++++|+++++++++++++++++|+++++++++++++++++|
| A      | g0/0      | PC0             | 192.168.1.1/24  |
|        | Se0/1/0   | C               | 10.0.0.1/30     |
|        | Se0/1/1   | B               | 10.0.0.5/30     |
|--------|-----------|-----------------|-----------------|
| B      | g0/0      | PC2             | 192.168.2.1/24  |
|        | Se0/1/0   | A               | 10.0.0.2/30     |
|--------|-----------|-----------------|-----------------|
| C      | g0/0      | PC1             | 192.168.3.1/24  |
|        | Se0/1/0   | B               | 10.0.0.6/30     |
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

---

## Instructions

1. **Set Up the Topology**
   - Connect routers and PCs as per the plan.

2. **Assign IP Addresses**
   - Configure LAN and WAN interfaces with the provided IPs.

3. **Configure Static Routes**
    - Router A: 
                - ip route 192.168.2.0 255.255.255.0 10.0.0.2 
                - ip route 192.168.3.0 255.255.255.0 10.0.0.6

    - Router B: 
                - ip route 192.168.1.0 255.255.255.0 10.0.0.1 
                - ip route 192.168.3.0 255.255.255.0 10.0.0.1

    - Router C: 
                - ip route 192.168.1.0 255.255.255.0 10.0.0.5 
                - ip route 192.168.2.0 255.255.255.0 10.0.0.5


4. **Test Connectivity**  
   - Ping PCs on other LANs and use traceroute to check paths.

5. **Verify Routing**  
   - Run `show ip route` on each router.

---

## Optional Challenges

- Introduce misconfigurations or configure default routes pointing to Router A.
- Add ACLs to restrict traffic between LANs.

---

## Setup

**Router A:**

| Interface | Neighbor | IP          |
|+++++++++++|++++++++++|+++++++++++++|
| g0/0      | PC1      | 192.168.1.1 |
| se0/1/0   | B        | 10.0.0.1    |
| se0/1/1   | C        | 10.0.0.5    |
|+++++++++++|++++++++++|+++++++++++++|

**Router B:**

| Interface | Neighbor | IP          |
|+++++++++++|++++++++++|+++++++++++++|
| g0/0      | PC2      | 192.168.2.1 |
| se0/1/0   | A        | 10.0.0.2    |
|+++++++++++|++++++++++|+++++++++++++|

**Router C:**

| Interface | Neighbor | IP          |
|+++++++++++|++++++++++|+++++++++++++|
| g0/0      | PC3      | 192.168.3.1 |
|se0/1/0    | A        | 10.0.0.6    |
|+++++++++++|++++++++++|+++++++++++++|

| Enpoint | IP address  |
|+++++++++|+++++++++++++|
| PC1     | 192.168.1.2 |
| PC2     | 192.168.2.2 |
| PC3     | 192.168.3.2 |

---

## CMD Sequence

### A
```plaintext
    enable
        conf t
            hostname A
            int g0/0
                no shut
                desc PC1
                ip add 192.168.1.1 255.255.255.0
            int g0/1
                shutdown
            int se0/1/0
                no shut
                desc B
                ip add 10.0.0.1 255.255.255.252
            int se0/1/1
                no shut
                desc C
                ip add 10.0.0.5 255.255.255.252
            ip route 192.168.2.0 255.255.255.0 10.0.0.2
            ip route 192.168.3.0 255.255.255.0 10.0.0.2
```

### B
```plaintext
    enable
        conf t
            hostname B
            int g0/0
                no shut
                desc PC2
                ip add 192.168.2.1 255.255.255.0
            int g0/1
                shut
            int se0/1/0
                no shut
                desc A
                ip add 10.0.0.2 255.255.255.252
            int se0/1/1
                shut
            ip route 192.168.1.0 255.255.255.0 10.0.0.1
            ip route 192.168.3.0 255.255.255.0 10.0.0.1
```            

### C
```plaintext
    enable
        conf t
            hostname C
            int g0/0
                no shut
                desc PC3
                ip add 192.168.3.1 255.255.255.0
            int g0/1
                shutdown
            int se0/1/0
                no shut
                desc A
                ip add 10.0.0.6 255.255.255.252
            int se0/1/1
                shut down
            ip route 192.168.1.0 255.255.255.0 10.0.0.5
            ip route 192.168.2.0 255.255.255.0.10.0.0.5
```            

---

## Questions

1. What is the purpose of static routing in this scenario?
    - Partially to practice configuring static routing.
    - Mostly to not need to rely on routing protocols in small topolgies like this.

2. How would you verify if a static route has been added successfully?
    - `show ip route`

3. What could happen if you forget to configure a route between Router A and Router C?
    - If there isn't a routing protocol configured, the router wouldn't be able route the packet between those two routers because they wouldn't know about each other or their connected subnets.

4. Why is testing connectivity essential after configuring static routes?
    - To ensure the routing configuration actually works correctly. A single small type-o can bork the networks connectivity. 

5. How do you configure a default route on Router B to Router A?
    - In Global Configuration mode:
        `ip route 0.0.0.0 0.0.0.0 se0/1/0`
