# Lab: Static and Dynamic Routing with Fault Tolerance

## Overview
This lab focuses on configuring and testing static and dynamic routing across three routers, ensuring connectivity between subnets, and simulating fault tolerance. It also includes optional add-ons like a loopback interface to represent a server. The configurations and command sequences for all devices are included.

---

## Topology

### Devices
- **3 Routers**: R1, R2, R3
- **3 PCs**: PC1, PC2, PC3 (one connected to each router)
- **3 Switches**: SW1, SW2, SW3 (optional, for added realism)

### Subnet Information
| Subnet  | Address        | Description                        |
|---------|----------------|------------------------------------|
| 101     | 192.168.1.0/24 | Connects R1 and PC1                |
| 102     | 192.168.2.0/24 | Connects R2 and PC2                |
| 103     | 192.168.3.0/24 | Connects R3 and PC3                |
| 104     | 10.0.0.0/30    | Point-to-Point between R1 and R2   |
| 105     | 10.0.0.4/30    | Point-to-Point between R2 and R3   |

### Topology Diagram

PC1 -- SW1 -- R1 ---- R2 ---- R3 -- SW3 -- PC3 
                      |
                      SW2
                      |
                      PC2

### Router Interfaces

| Router | Interface | Address          | Subnet |
|--------|-----------|------------------|--------|
| R1     | g0/0      | 192.168.1.1/24   | 101    |
| R1     | g0/1/0    | 10.0.0.1/30      | 104    |
| R2     | g0/0      | 192.168.2.1/24   | 102    |
| R2     | g0/1/0    | 10.0.0.2/30      | 104    |
| R2     | g0/0/0    | 10.0.0.5/30      | 105    |
| R3     | g0/0      | 192.168.3.1/24   | 103    |
| R3     | g0/1/0    | 10.0.0.6/30      | 105    |

---

## Challenges

### Challenge 1: Basic Connectivity
- Configure IP addresses for all router interfaces and PCs.
- Test connectivity between directly connected devices.

### Challenge 2: Static Routes
- Configure static routes to enable end-to-end communication between PCs.
- Test connectivity across subnets.

### Challenge 3: Default Routes
- Configure R1 and R3 to use R2 as the default gateway.
- Test traffic flow with specific static routes removed.

### Challenge 4: Fault Tolerance
- Simulate a link failure by shutting down the link between R1 and R2.
- Test and restore connectivity.

### Challenge 5: Optional Add-Ons
- Configure a loopback interface on R2 to simulate a server.
- Add static routes to reach the loopback network.

---

## Command Sequences

### Switch Configurations

#### SW1
```plaintext
    enable
        conf t
            hostname SW1
            int g0/1
                no shut
                 desc R1
                 switchport mode access
                 switchport access vlan 101
                 int f0/1
                 no shut
                 desc PC1
                 switchport mode access
                 switchport access vlan 101
            copy run start
            ```

#### SW2
```plaintext
    enable
        conf t
           hostname SW2
           int g0/1
               no shut
               desc R2
               switchport mode access
               switchport access vlan 102
               int f0/1
               no shut
               desc PC2
               switchport mode access
               switchport access vlan 102
           copy run start
           ```

#### SW3       
```plaintext
    enable
        conf t
            hostname SW3
            int g0/1
                no shut
                desc R3
                switchport mode access
                switchport access vlan 103
                int f0/1
                no shut
                desc PC3
                switchport mode access
                switchport access vlan 103
        copy run start
        ```
### Router Configurations        

#### R1
```plaintext
    enable
        conf t
            hostname R1
            int g0/0
                no shut
                desc SW1
                ip address 192.168.1.1 255.255.255.0
                int g0/1/0
                no shut
                desc R2
                ip address 10.0.0.1 255.255.255.252
            ip route 192.168.2.0 255.255.255.0 10.0.0.2
            ip route 192.168.3.0 255.255.255.0 10.0.0.2
            ```

#### R2
```plaintext
    enable
        conf t
            hostname R2
            int g0/0
                no shut
                desc SW2
                ip address 192.168.2.1 255.255.255.0
                int g0/1/0
                no shut
                desc R1
                ip address 10.0.0.2 255.255.255.252
                int g0/0/0
                no shut
                desc R3
                ip address 10.0.0.5 255.255.255.252
            ip route 192.168.1.0 255.255.255.0 10.0.0.1
            ip route 192.168.3.0 255.255.255.0 10.0.0.6
            ```

#### R3
```plaintext
    enable
        conf t
            hostname R3
            int g0/0
                no shut
                desc SW3
                ip address 192.168.3.1 255.255.255.0
                int g0/1/0
                no shut
                desc R2
                ip address 10.0.0.6 255.255.255.252
            ip route 192.168.1.0 255.255.255.0 10.0.0.5
            ip route 192.168.2.0 255.255.255.0 10.0.0.5
            ```

---

## Questions to Answer

1. What is the purpose of configuring default routes on R1 and R3?
    There are a couple reasons:
        a. It exercises being able to configure default routes.
        b. It enables the interface to forward/route traffic regardless of whether there is a route in the ip routing table or not for that destination.

2. How does shutting down g0/1/0 on R3 affect network connectivity?
    It brings PC3 offline and brings the link between R2 and R3 down. It causes PC3 to lose connection with any other endpoint aside from its default gateway and no other endpoint can reach it past R3. 

3. Why is the loopback interface on R2 useful in this lab setup?
    Because it emulates a server on the network in that if you can reach the loopback interface, you would be able to reach the server, assuming everything between the server and R2 was properly configured. 

4. Explain the difference between static and dynamic routing in this context. 
    In this context, static routes are manually configured on the switch with the following cmd:
    `ip route <dst_addr> <mask> <next_hop_addr | outgoing_port> [<cost>]`
