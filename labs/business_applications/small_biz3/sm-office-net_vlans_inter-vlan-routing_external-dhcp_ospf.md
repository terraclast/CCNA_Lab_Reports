### **Lab Title:** Small Office Network with VLANs, Inter-VLAN Routing, and OSPF  

**Difficulty Level:** 15/100  

**Objective:**  
- Create a network for a small office with two departments: **IT** and **Sales**.  
- Configure VLANs, inter-VLAN routing, and basic OSPF.  
- Verify connectivity and routing functionality.

---

### **Lab Topology**
#### Devices:
- **Switch SW1** (Layer 3 Switch for VLANs and Inter-VLAN Routing)  
- **Router R1** (Connects to an external network)  
- **PCs** (One for each VLAN)  

#### Network Layout:
1. **SW1**:
   - VLAN 10: IT Department (`192.168.10.0/24`)
   - VLAN 20: Sales Department (`192.168.20.0/24`)  

2. **R1**:
   - Connects to SW1 via a routed link (`10.0.1.0/30`).  
   - Simulates a connection to an external network (`10.0.2.0/24`).

---

### **IP Addressing Table**

| Device | Interface      | IP Address       | Subnet Mask       |
|--------|----------------|------------------|-------------------|
| SW1    | VLAN 10        | 192.168.10.1     | 255.255.255.0     |
| SW1    | VLAN 20        | 192.168.20.1     | 255.255.255.0     |
| SW1    | G0/1           | 10.0.1.2         | 255.255.255.252   |
| R1     | G0/0           | 10.0.1.1         | 255.255.255.252   |
| R1     | G0/1           | 10.0.2.1         | 255.255.255.0     |
| IT PC  | NIC            | 192.168.10.10    | 255.255.255.0     |
| Sales PC | NIC          | 192.168.20.10    | 255.255.255.0     |

---

### **Tasks**

#### **1. Configure VLANs on SW1**
- Create VLAN 10 (IT) and VLAN 20 (Sales).
- Assign the appropriate ports to each VLAN:
  - IT PC on VLAN 10.
  - Sales PC on VLAN 20.

#### **2. Configure Inter-VLAN Routing on SW1**
- Enable routing on the Layer 3 switch.
- Create SVIs (Switched Virtual Interfaces) for VLAN 10 and VLAN 20.

#### **3. Configure Routed Link Between SW1 and R1**
- Convert the port connecting SW1 and R1 to a routed port.
- Assign IP addresses on both ends of the link.

#### **4. Configure OSPF**
- Enable OSPF on both SW1 and R1.
- Advertise all networks in OSPF Area 0.

#### **5. Test Connectivity**
- Verify that:
  - IT PC can ping Sales PC.
  - Both PCs can ping the R1 external interface (`10.0.2.1`).
  - R1 can ping both VLAN gateways (`192.168.10.1` and `192.168.20.1`).

#### **6. Configure a Default Route on R1**
- Configure a default route on R1 pointing to the external network.

#### **7. Verify Routing**
- Use `show ip route` on both R1 and SW1 to confirm routes learned via OSPF.

---

### **Questions to Answer**
1. Are VLANs correctly isolated?
2. Does inter-VLAN routing work?
3. Does OSPF establish a neighborship between SW1 and R1?
4. Are routes to all networks visible in the routing table?
5. Can both PCs communicate with each other and the external network?

---

### **Guidance Without Configuration Commands**
- Configure VLANs and assign ports carefully. Test VLAN membership using `ping`.  
- Enable `ip routing` on SW1 and assign IP addresses to the SVIs.  
- Ensure the link between SW1 and R1 is configured as a routed port with OSPF enabled.  
- For OSPF, remember to advertise directly connected networks using wildcard masks.

---

## Port Connections

| Device    | Port      | Port type                 | Neighbor  | Neigh_port    |
|+++++++++++|+++++++++++|+++++++++++++++++++++++++++|+++++++++++|+++++++++++++++|
| Ext Net   | G9        | PT-CLOUD-NM-1FGE          | R1        | G0/1/0        |
| R1        | G0/1/0    | HWIC-1GE-SPF/GLC-LH-SMD   | Ext Net   | G9            |
|           | G0/0      | Built-in GigabitEthernet  | SW1       | G0/1          |
| SW1       | G0/1      | Built-in GigabitEthernet  | R1        | G0/0          |
|           | F0/1      | Built-in FastEthernet     | Sales     | F0            |
|           | F0/2      | "                     "   | IT        | F0            |

## Notes
- I decided to add an external DHCP server on R1 for practice. 
- There was some troubleshooting required to figure out why DHCP was not loading networking configs to the PCs. 
    - Turns out OSPF is not setup by default on routers and L3 switches.  
    - Setting static routes got DHCP running. 
    - Sales and IT PCs successfully aquired IP configs via DHCP
- Setting up OSPF is fairly easy and straightforward.  All verifications looked good. 
- Removed static routes to double-test OSPF functionality. 
    - *show ip route* before and after removing routes shows OSPF is working as intended. 
    - *show ip ospf neighbor* correctly showed R1 and SW1's adjacent neighbor relationship. 


## CMD Sequence

### R1
    enable
        conf t
            hostname R1
            int g0/1/0
                desc Ext Net
                no shut
                ip add 10.0.2.2 255.255.255.252
            int g0/0
                desc SW1
                no shut
                ip add 10.0.1.1 255.255.255.252
                exit
            ip dhcp pool Sales
                network 192.168.10.0 255.255.255.0
                default-router 192.168.10.1
            ip dhcp pool IT
                network 192.168.20.0 255.255.255.0
                default-router 192.168.20.1
            ip dhcp excluded-address 192.168.10.1
            ip dhcp excluded-address 192.168.20.1
            ip route 0.0.0.0 0.0.0.0 10.0.2.1
            ip route 192.168.10.0 255.255.255.0 10.0.1.2
            ip route 192.168.20.0 255.255.255.0 10.0.1.2
            router ospf 1
                network 10.0.1.0 0.0.0.3 area 0
            no ip route 192.168.10.0 255.255.255.0
            no ip route 192.168.20.0 255.255.255.0

### SW1
    enable
        conf t
            ip routing
            hostname SW1
            vlan 10
                name Sales
            vlan 20
                name IT
            int vlan 10
                ip add 192.168.10.1 255.255.255.0
            int vlan 20
                ip add 192.168.20.1 255.255.255.0
            int g0/1
                no shut
                desc R1
                no switchport
                ip add 10.0.1.2 255.255.255.252
            int f0/1
                no shut
                desc Sales
                switchport mode access
                switchport access vlan 10
            int f0/2
                no shut
                desc IT
                switchport mode access
                switchport access vlan 20
                exit
            int vlan 10
                ip helper-address 10.0.1.1
            int vlan 20
                ip helper-address 10.0.1.1
            router ospf 1
                network 10.0.1.0 0.0.0.3 area 0
                network 192.168.10.0 0.0.0.255 area 0
                network 192.168.20.0 0.0.0.255 area 0
