# Scenario

You’ve been hired by a startup to set up their network. The office has:

- Two departments: Sales and IT.
- Employees with PCs and VoIP phones.
- A connection to the internet via a router.

Your task:
- Create VLANs for the Sales and IT departments.
- Configure inter-VLAN routing.
- Ensure the PCs and phones can communicate with each other and access the internet.

---

## Network Requirements

### Devices
- 1 Router (e.g., Cisco 1941)
- 1 Switch (e.g., Cisco 2960)
- 4 PCs (2 for Sales, 2 for IT)
- 2 Phones (1 for Sales, 1 for IT)
- 1 Cloud to simulate internet access.

### IP Addressing Plan

| Device/Interface              | IP Address        | Subnet Mask      | VLAN |
|-------------------------------|-------------------|------------------|------|
| Router g0/0.10 (Sales)        | 192.168.10.1      | 255.255.255.0    | 10   |
| Router g0/0.20 (IT)           | 192.168.20.1      | 255.255.255.0    | 20   |
| Sales_PC_1                    | 192.168.10.2      | 255.255.255.0    | 10   |
| Sales_PC_2                    | 192.168.10.3      | 255.255.255.0    | 10   |
| IT_PC_1                       | 192.168.20.2      | 255.255.255.0    | 20   |
| IT_PC_2                       | 192.168.20.3      | 255.255.255.0    | 20   |
| IT Phone                      | 192.168.30.2      | 255.255.255.0    | 30   |
| Sales Phone                   | 192.168.30.3      | 255.255.255.0    | 30   |
| Cloud (Internet)              | DHCP              | -                | -    |

### Port Connections:

#### R1:
        G0/1 -- Cloud Ethernet6 G0/0 -- SW1 G0/2

#### SW1: 
        F0/1 -- IT Phone
        F0/2 -- IT_PC_2 F0/3 -- Sales Phone F0/4 -- Sales_PC_2

#### IT Phone:
        Switch -- SW1 F0/1 PC -- IT_PC_1 F0

#### Sales Phone:
        Switch -- SW1 F0/3 PC -- Sales_PC_1 F0

#### IT_PC_1:
        F0 -- IT Phone PC

#### IT_PC_2:
        F0 -- SW1 F0/2

#### Sales_PC_1:
        F0 -- Sales Phone

#### Sales_PC_2:
        F0 -- SW1 F0/4

---

## CMD Sequence:

### IT_PC_1:
```plaintext
        default gateway 192.168.20.1
        IP address      192.168.20.2 255.255.255.0
```
### IT_PC_2:
```plaintext        
        default gateway 192.168.20.1
        IP addr         192.168.20.3 255.255.255.0
```

### Sales_PC_1:
```plaintext
        default gateway 192.168.10.1
        IP addr         192.168.10.2 255.255.255.0
```

### Sales_PC_2:
```plaintext
        default gateway 192.168.10.1
        IP addr         192.168.10.3 255.255.255.0
```

### SW1 Configuration:
```plaintext
        enable
            conf t
                hostname SW1
                vlan 10
                    name Sales
                vlan 20
                    name IT
                vlan 30
                    name VoIP
                interface g0/1
                    switchport mode trunk
                    desc R1
                int f0/1
                    switchport mode access
                    switchport access vlan 20
                    switchport voice vlan 30
                    desc IT Phone
                int f0/2
                    switchport mode access
                    switchport access vlan 20
                    desc IT_PC_2
                int f0/3
                    switchport mode access
                    switchport access vlan 10
                    switchport voice vlan 30
                    desc Sales Phone
                int f0/4
                    switchport mode access
                    switchport access vlan 10
                    desc Sales_PC_2
```

### R1 Configuration:
```plaintext
        enable
            conf t
                hostname R1
                int g0/0.10
                    no shutdown
                    desc Sales
                    encapsulation dot1q 10
                    ip add 192.168.10.1 255.255.255.0
                int g0/0.20
                    no shutdown
                    encapsulation dot1q 20
                    ip add 192.168.20.1 255.255.255.0
                int g0/0.30
                    no shutdown
                    encapsulation dot1q 30
                    ip add 192.168.30.1 255.255.255.0
                int g0/1
                    desc Cloud
                    ip add 192.168.1.1 255.255.255.0
                    exit
                ip dhcp pool Sales
                    network 192.168.10.0 255.255.255.0
                    default-router 192.168.10.1
                ip dhcp pool IT
                    network 192.168.20.0 255.255.255.0
                    default-router 192.168.20.1
                ip dhcp pool VoIP
                    network 192.168.30.0 255.255.255.0
                    default-network 192.168.30.1
```

---

## Final Show Run for SW1 and R1:

### SW1:
```plaintext
SW1#show run
Building configuration...

Current configuration : 1464 bytes
!
version 15.0
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname SW1
!
spanning-tree mode pvst
spanning-tree extend system-id
!
interface FastEthernet0/1
 description IT Phone
 switchport access vlan 20
 switchport mode access
 switchport voice vlan 30
!
interface FastEthernet0/2
 description IT_PC_2
 switchport access vlan 20
 switchport mode access
!
interface FastEthernet0/3
 description Sales Phone
 switchport access vlan 10
 switchport mode access
 switchport voice vlan 30
!
interface FastEthernet0/4
 description Sales_PC_2
 switchport access vlan 10
 switchport mode access
!
interface GigabitEthernet0/1
 description R1
 switchport mode trunk
!
interface GigabitEthernet0/2
!
interface Vlan1
 no ip address
 shutdown
!
line con 0
!
line vty 0 4
 login
line vty 5 15
 login
!
end
```

### R1
```plaintext
show run
Building configuration...

Current configuration : 1184 bytes
!
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname R1
!
ip dhcp pool Sales
    network 192.168.10.0 255.255.255.0
    default-router 192.168.10.1
ip dhcp pool IT
    network 192.168.20.0 255.255.255.0
    default-router 192.168.20.1
ip dhcp pool VoIP
    network 192.168.30.0 255.255.255.0
    default-router 192.168.30.1
!
ip cef
no ipv6 cef
!
interface GigabitEthernet0/0.10
    description Sales
    encapsulation dot1Q 10
    ip address 192.168.10.1 255.255.255.0
interface GigabitEthernet0/0.20
    encapsulation dot1Q 20
    ip address 192.168.20.1 255.255.255.0
interface GigabitEthernet0/0.30
    encapsulation dot1Q 30
    ip address 192.168.30.1 255.255.255.0
interface GigabitEthernet0/1
    description Cloud
    ip address 192.168.1.1 255.255.255.0
!
ip classless
!
ip flow-export version 9
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
end
```

---

## Questions to Answer:

1. What is the purpose of the encapsulation dot1q command in the router configuration?
    - It associates the VLAN to the particular subinterface and enables the subinterface to route that VLAN's traffic.

2. How do you configure a VLAN for voice traffic on a switch?
    - In Interface Config mode for the particular interface that will be forwarding voice traffic, you configure the folloiwng cmd:
        `switchport voice vlan <vlan_id>`
        You will also need to configure a separate access VLAN for any computer attached to the phone's switch. 

3. What is the significance of assigning a default gateway to the PCs in each department?
    - This is the only way the PCs can communicate outside of their VLAN and subnet. Switches connect devices within a broadcast domain while routers (aka default-gateways in the LAN) connect separate broadcast domains.  

4. Why is a DHCP pool configured for each VLAN on the router?
    - Because each VLAN represents a different subnet.
    - Within each subnet there are two addresses that are automatically reserved: subnet ID and Broadcast address
    - There will also be other addresses that need to be reserved, like routers, servers, and other network appliances that shouldn't have their IP address change over time.

5. What is the role of trunking between the router and the switch in this setup?
    - It simplifies the number of interfaces both the switch and router need to use to route traffic on the different VLANS.  
    - It serves as a single pipe to bring all the traffic from the switch to the gateway and enables L3 EtherChannel redundancy. 
