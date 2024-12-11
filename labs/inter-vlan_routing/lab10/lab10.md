# Lab Difficulty Level: 10/100

## Lab Objective:
Build a basic network topology with two LANs connected via a single router. Practice basic IP addressing, VLAN configuration, static routes, and basic switch-port configuration to understand fundamental networking concepts.
i
---

## Lab Setup:

### Devices Required:

| Number | Type     | Misc      |
|++++++++|++++++++++|+++++++++++|
| 1      | Router   |           |
| 2      | Switches |           |
| 4      | PCs      | 2 per LAN |
|++++++++|++++++++++|+++++++++++|

### Network Topology:

    LAN 1                        LAN 2
PC1  ---  SW1  ---  R1  ---  SW2  ---  PC3
           |                  |
          PC2                PC4

### Port Connections:

| Device | Interface | Neighbor | Neigh_int |
|++++++++|+++++++++++|++++++++++|+++++++++++|
| R1     | g0/0      | SW1      | g0/1      |
|        | g0/1      | SW2      | g0/1      |
|--------|-----------|----------|-----------|
| SW1    | g0/1      | R1       | g0/0      |
|        | f0/1      | PC1      | f0        |
|        | f0/2      | PC2      | f0        |
|--------|-----------|----------|-----------|
| SW2    | g0/1      | R1       | g0/1      |
|        | f0/1      | PC3      | f0        |
|        | f0/2      | PC4      | f0        |
|++++++++|+++++++++++|++++++++++|+++++++++++|

---

## Steps:

### Set Up IP Addressing:
    
    - LAN 1: 192.168.1.0/24
    - PC1:   192.168.1.2
    - PC2:   192.168.1.3
    
    - LAN 2: 192.168.2.0/24
    - PC3:   192.168.2.2
    - PC4:   192.168.2.3

### Assign IPs to router interfaces connected to each LAN:
    - Router Interface to LAN 1: 192.168.1.1
    - Router Interface to LAN 2: 192.168.2.1

### Configure Basic VLANs (Optional):
    - Switch 1: VLAN 10
    - Switch 2: VLAN 20

### Assign Ports to VLANs:
    - Assign each PC’s switch port to the respective VLAN on both switches.

### Router Configuration:
    - Configure static routes between the two LANs:
        - Add route for 192.168.1.0/24 to 192.168.2.0/24
        - Add route for 192.168.2.0/24 to 192.168.1.0/24

### Test Connectivity:
    - Ping from PC1 to PC3 and from PC2 to PC4 to test inter-VLAN routing.
    - Confirm that all PCs within the same LAN can communicate.

### Expected Outcome:
    - PCs on different LANs should successfully communicate via the router.
    - Basic understanding of IP addressing, static routing, VLANs, and inter-VLAN routing.
    - This lab reinforces foundational network configuration and troubleshooting skills at a manageable difficulty level.

---

## CMD Sequence

### PC Config:

#### PC1:
    - DFG -- 192.168.1.1
    - IP addr -- 192.168.1.2 255.255.255.0
    
#### PC2:
    - DFG -- 192.168.1.1
    - IP addr -- 192.168.1.3 255.255.255.0
    
#### PC3:
    - DFG -- 192.168.2.1
    - IP addr -- 192.168.2.2 255.255.255.0
    
#### PC4:
    - DFG -- 192.168.2.1
    - IP addr -- 192.168.2.3 255.255.255.0

### SW1:
```plaintext
    enable 
        conf t 
            hostname SW1 
            vlan 2 
                name VLAN2 
            int f0/1 
                desc PC1 
                switchport mode access 
                switchport access vlan 2 
            int f0/2 
                desc PC2 
                switchport mode access 
                switchport access vlan 2 
            int g0/1 
                desc R1 
                switchport mode access 
                switchport access vlan 2 
            int range f0/3-24 
                shutdown 
            int g0/2 
                shutdown 
                exit 
            spanning-tree portfast default 
            ip default-gateway 192.168.2.1 
            ^Z 
        show int status 
        show vlan brief 
        show running-config 
        copy running-config startup-config    
```

### SW2:
```plaintext
    enable 
        conf t 
            hostname SW2 
            vlan 3 
                name VLAN3 
            int f0/1 
                desc PC3 
                switchport mode access 
                switchport access vlan 3 
            int f0/2 
                desc PC4 
                switchport mode access 
                switchport access vlan 3 
            int g0/1 
                no shutdown 
                desc R1 
                switchport mode access 
                switchport access vlan 3 
                exit 
            int range f0/3-24 
                shutdown 
            int g0/2 
                shutdown 
                exit 
            spanning-tree port-fast default 
            ip default-gateway 192.168.2.1 
            ^Z
        show int status 
        show vlan brief 
        show running-config 
        copy running-config startup-config
```

### R1:
```plaintext
    enable 
        conf t 
            hostname R1 
            int g0/0 
                desc SW1 
                no shutdown 
                ip address 192.168.1.1 255.255.255.0 
            int g0/1 
                no shutdown 
                desc SW2 
                ip address 192.168.2.1 255.255.255.0 
                exit 
            ip route 192.168.1.0 255.255.255.0 g0/0 
            ip route 192.168.2.0 255.255.255.0 g0/1i
```

--- 

## Questions to Answer:

1. **What is the purpose of static routing in this network setup?**
    - In a topology this small it makes just as much sense to create static routes as it does to configure a routing protocol.
    - This size topology is easy enough to manually manage and so it doesn't hurt to get some practice. 

2. **How do you assign a port on a switch to a specific VLAN?**
    - In Interface configuration mode:
        `switchport mode access`
        `switchport access vlan <vlan_id>`

3. **What happens if the `ip default-gateway` command is omitted on the switches?**
    - The switches will not be able to be managed remotely via telnet or ssh.

4. **Why is inter-VLAN routing necessary in this scenario?**
    - Each LAN is in it's own subnet and each subnet is in its own VLAN.
    - In order for each VLAN to communicate with the other, the VLANs must have their packets routed to each other, via a router or L3 switch. 

5. **How would you verify that static routes are correctly configured on the router?**
    - In enable mode:
        `show ip route`
    - Check the output against the topology and the network plan
    - Check connectivity with ping and, possibly, traceroute.
