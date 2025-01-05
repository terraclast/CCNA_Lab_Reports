### **EtherChannel Lab: Layer 2 and Layer 3 Concepts**

This lab will involve configuring EtherChannel in both Layer 2 and Layer 3 modes, with VLANs, trunking, and basic IP connectivity. 

---

#### **Scenario**
You are tasked with configuring an EtherChannel between two switches for redundancy and increased bandwidth. The switches will also participate in inter-VLAN routing through a Layer 3 EtherChannel. VLANs will be configured for department segregation.

---

### **Lab Topology**
- **Switch1 (SW1)** and **Switch2 (SW2)** connected via:
  - **FastEthernet0/1** and **FastEthernet0/2** for Layer 2 EtherChannel.
  - **GigabitEthernet0/1** and **GigabitEthernet0/2** for Layer 3 EtherChannel.
- VLANs:
  - VLAN 10: Sales (`192.168.10.0/24`)
  - VLAN 20: IT (`192.168.20.0/24`)
- PCs:
  - Sales:
    - Sales1
    - Sales2
    - Sales3
    - Sales4
  - IT:
    - IT1
    - IT2
    - IT3
    - IT4

#### Port Assignments:

| Device    | Port      | Nbr       | Nbr Port  | Conn Type & name    |
+-----------+-----------+-----------+-----------+---------------------+
| SW1       | f0/1      | SW2       | f0/1      | EtherChannel L2 EC1 |
|           | f0/2      | SW2       | f0/2      | EtherChannel L2 EC1 |
|           | g0/1      | SW2       | g0/1      | EtherChannel L3 EC2 |
|           | g0/2      | SW2       | g0/2      | EtherChannel L3 EC2 |
|           | f0/3      | Sales1    | f0        | Ethernet            |
|           | f0/4      | Sales2    | f0        | Ethernet            |
|           | f0/5      | Sales3    | f0        | Ethernet            |
|           | f0/6      | Sales4    | f0        | Ethernet            |
|           | f0/7      | IT1       | f0        | Ethernet            |
|           | f0/8      | IT2       | f0        | Ethernet            |
|           | f0/9      | IT3       | f0        | Ethernet            |
|           | f0/10     | IT4       | f0        | Ethernet            |

---

### **Objectives**
1. Configure VLANs and assign them to access ports on SW1.
2. Configure a Layer 2 EtherChannel (PAgP) between SW1 and SW2 for VLAN trunking.
3. Configure a Layer 3 EtherChannel (LACP) for inter-VLAN routing.
4. Verify connectivity between VLANs.

---

### **Tasks**

#### **Task 1: Configure VLANs on SW1**
1. Create VLAN 10 and VLAN 20.
2. Assign VLAN 10 to FastEthernet0/3-6 (Sales department).
3. Assign VLAN 20 to FastEthernet0/7-10 (IT department).

---

#### **Task 2: Configure Layer 2 EtherChannel**
1. Create a Layer 2 EtherChannel between SW1 and SW2 using **FastEthernet0/1** and **FastEthernet0/2**.
2. Use **PAgP** for negotiation.
3. Configure the EtherChannel as a trunk and allow only VLANs 10 and 20.

---

#### **Task 3: Configure Layer 3 EtherChannel**
1. Create a Layer 3 EtherChannel between SW1 and SW2 using **GigabitEthernet0/1** and **GigabitEthernet0/2**.
2. Use **LACP** for negotiation.
3. Assign IP addresses to the Layer 3 EtherChannel:
   - SW1: `192.168.1.1/30`
   - SW2: `192.168.1.2/30`
4. Enable inter-VLAN routing on SW1.

---

#### **Task 4: Verify Configuration**
1. Verify VLAN configurations using `show vlan brief`.
2. Verify EtherChannel status using `show etherchannel summary`.
3. Test inter-VLAN connectivity by pinging between VLANs.

---

### **Verification Checklist**
- **Layer 2 EtherChannel:** Check the trunk's operational status and VLAN allowance.
- **Layer 3 EtherChannel:** Verify IP connectivity between SW1 and SW2.
- **Inter-VLAN Routing:** Ensure devices in VLAN 10 and VLAN 20 can communicate through SW1.

---

# CMD Sequence

## SW1:

enable
    conf t
        hostname SW1
        vlan 10
            name Sales
        vlan 20
            name IT
        int f0/3
            desc Sales1
            switchport mode access
            switchport access vlan 10
        int f0/4
            desc Sales2
            exit
        int range f0/4-10
            switchport mode access
        int range f0/4-6
            switchport access vlan 10
        int range f0/7-10
            switchport access vlan 20
        int f0/5
            desc Sales3
        int f0/6
            desc Sales4
        int f0/7
            desc IT1
        int f0/8
            desc IT2
        int f0/9
            desc IT3
        int f0/10
            desc IT4
        int range f0/1-2
            desc EC1
            switchport trunk encapsulation dot1q
            switchport mode trunk
            switchport trunk allowed vlan 10,20
            channel-group 1 mode desirable
        int port-channel 1
            desc EC1
            switchport trunk encapsulation dot1q
            switchport mode trunk
            switchport trunk allowed vlan 10,20
        int range g0/1-2
            desc EC2
            no switchport
        int port-channel 2
            desc EC2
            no switchport
            ip add 192.168.1.1 255.255.255.252
        int range g0/1-2
            channel-group 2 mode active
        int vlan 10
            ip add 192.168.10.1 255.255.255.0
        int vlan 20
            ip add 192.168.20.1 255.255.255.0
            exit
        ip routing


## SW2

enable 
    conf t
        hostname SW2
        ip routing
        int range f0/1-2
            desc EC1
            switchport trunk encapsulation dot1q
            switchport mode trunk
            switchport trunk allowed vlan 10,20
            channel-group 1 mode desirable
        int port-channel 1
            desc EC1
            switchport trunk encapsulation dot1q
            switchport mode trunk
            switchport trunk allowed vlan 10,20
        int range g0/1-2
            desc EC2
        int port-channel 2
            desc EC2
            no switchport
            ip add 192.168.1.2 255.255.255.252
