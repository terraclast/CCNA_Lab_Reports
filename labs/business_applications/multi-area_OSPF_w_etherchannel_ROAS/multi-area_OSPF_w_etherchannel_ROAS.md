Here’s a lab that integrates **EtherChannel (L2 and L3)**, **resiliency**, **multi-area OSPF**, and **Router-on-a-Stick (ROAS) with at least two subnets**:

---

### **Lab Title: Multi-Area OSPF with EtherChannel and ROAS**
**Difficulty Level:** 40 (Intermediate-Advanced)

---

### **Lab Scenario**
You are designing a network for a medium-sized business. The network must:
1. Use **Layer 2 and Layer 3 EtherChannel** for increased bandwidth and redundancy.
2. Implement **multi-area OSPF** for scalability:
   - **Area 0**: Backbone area connecting core devices.
   - **Area 1**: Sales department.
   - **Area 2**: IT department.
3. Configure **ROAS (Router-on-a-Stick)** on a Layer 3 switch with at least two VLANs per subnet.
4. Ensure resiliency with redundant paths and EtherChannel connections.

---

### **Topology Overview**
| **Device**  | **Connections**                          |
|-------------|------------------------------------------|
| R1 (Router) | Connects to L3 Switch (Core) and Area 1  |
| R2 (Router) | Connects to L3 Switch (Core) and Area 2  |
| SW1 (Core)  | Core switch, connects to R1, R2, and SW2 |
| SW2 (Access)| Connects to PCs for VLANs 10 and 20      |

---

### **IP Addressing and VLANs**

| **Subnet/VLAN** | **VLAN ID** | **Subnet**        | **Purpose**          |
|------------------|-------------|-------------------|----------------------|
| Sales VLAN       | 10          | 192.168.10.0/24  | Sales PCs            |
| IT VLAN          | 20          | 192.168.20.0/24  | IT PCs               |
| Core Network     | -           | 10.0.0.0/30      | Core router links    |
| Backbone (Area 0)| -           | 10.0.1.0/30      | OSPF Backbone        |

---

### **Tasks**

#### **Task 1: Layer 2 EtherChannel (Between SW1 and SW2)**
1. Configure a **Layer 2 EtherChannel** between **SW1** and **SW2** using interfaces `g0/1` and `g0/2`.
2. Verify EtherChannel is operational with the appropriate mode (e.g., active/active for LACP).

#### **Task 2: Layer 3 EtherChannel (Between SW1 and R1/R2)**
1. Configure a **Layer 3 EtherChannel** between:
   - **SW1** and **R1** (Area 1 link).
   - **SW1** and **R2** (Area 2 link).
2. Assign IP addresses to the EtherChannel interfaces based on the **Core Network** and **Backbone** subnets.

#### **Task 3: Router-on-a-Stick (ROAS)**
1. On **SW2**, configure ROAS for VLANs 10 (Sales) and 20 (IT):
   - Subinterface IP for VLAN 10: `192.168.10.1`
   - Subinterface IP for VLAN 20: `192.168.20.1`
2. Configure trunking between **SW1** and **SW2** to carry VLAN traffic.

#### **Task 4: Multi-Area OSPF**
1. Configure OSPF with:
   - **R1** and **R2** participating in Areas 0, 1, and 2 as appropriate.
   - Ensure **Area 1 (Sales)** and **Area 2 (IT)** have proper routes to each other via Area 0.
2. Use `default-information originate` on **R1** to inject a default route into OSPF.

#### **Task 5: Resiliency Verification**
1. Test path failover by simulating a link failure in:
   - **Layer 2 EtherChannel** (disconnect one interface).  
   - **Layer 3 EtherChannel** (disconnect a link between **SW1** and R1/R2).
2. Verify continued connectivity between PCs in VLANs 10 and 20.

---

### **Questions to Answer**
1. Which command verifies the operational state of EtherChannel for Layer 2 and Layer 3?  
2. How does OSPF handle path selection when both R1 and R2 advertise routes to the Sales and IT subnets?  
3. What happens to OSPF adjacencies if Area 0 loses connectivity to R1 or R2?  
4. Which specific configuration ensures VLANs 10 and 20 traffic can traverse the trunk link on SW2?  

---

### **Hints for Configuration**
- **Layer 2 EtherChannel:** Use `channel-group <ID> mode active` for LACP.
- **Layer 3 EtherChannel:** Create a port-channel interface and assign an IP address.
- **ROAS Trunking:** Use `switchport mode trunk` on interfaces between SW1 and SW2.
- **OSPF Areas:** Use `network <subnet> <wildcard-mask> area <#>`.


