### **CCNA Troubleshooting Lab: Key Focus Areas**

#### **Lab Overview**
This lab will help reinforce your understanding of VLANs, trunk troubleshooting, OSPF configurations, Ethernet standards, subnetting, and ACL basics. The tasks simulate real-world scenarios you might encounter on the CCNA exam or in practical network configurations.

---

#### **Lab Topology**
- **Devices**:
  - 2 Switches (SW1, SW2)
  - 2 Routers (R1, R2)
  - 2 PCs (PC1, PC2)
  - 1 Server (SRV1)
- **Connections**:
  - R1 <--> SW1 <--> SW2 <--> R2
  - PC1 connected to SW1
  - PC2 connected to SW2
  - SRV1 connected to SW1

---

### **Lab Tasks**

#### **Task 1: VLAN and Trunk Troubleshooting**
1. Create and assign VLANs on SW1:
   - VLAN 10: `Sales`
   - VLAN 20: `HR`
   - Assign `PC1` to VLAN 10 and `SRV1` to VLAN 20.
2. On SW2, assign `PC2` to VLAN 20.
3. Configure a trunk link between SW1 and SW2 to allow VLAN 10 and 20 traffic.
4. Verify VLANs and trunk configuration using the following commands:
   - `show vlan brief`
   - `show interfaces trunk`
5. Troubleshoot connectivity issues if VLAN 20 traffic does not pass between SW1 and SW2.

#### **Task 2: OSPF Neighbor Verification and Configuration**
1. Configure OSPF on R1 and R2:
   - R1 OSPF Router ID: `1.1.1.1`
   - R2 OSPF Router ID: `2.2.2.2`
   - Enable OSPF on the interfaces connecting to the switches.
2. Verify OSPF neighbor relationships using:
   - `show ip ospf neighbor`
   - `show ip route`
3. Simulate an issue by mismatching OSPF Hello timers on R1 and R2. Resolve the issue and confirm the neighbor relationship forms successfully.

#### **Task 3: Ethernet Standards and Cable Lengths**
1. Identify the appropriate Ethernet standards for the following scenarios:
   - Connection between SW1 and SW2.
   - Long-distance connection between R1 and R2 (assume fiber).
2. Use the following commands to confirm link status and speeds:
   - `show interfaces status`
   - `show interfaces [interface]`

#### **Task 4: Subnetting Exercise**
1. Given the network `172.16.0.0/20`, subnet it into 8 equal subnets:
   - What is the new subnet mask?
   - How many usable addresses per subnet?
   - Provide the first two subnet IDs.
2. Configure the first subnet on R1 and the second subnet on R2.

#### **Task 5: ACL Basics**
1. On R1, create a standard ACL to block traffic from `192.168.1.0/24` while allowing all other traffic.
2. Apply the ACL to the appropriate interface in the inbound direction.
3. Verify the ACL is working:
   - Use `ping` from PC1 to a remote IP outside the blocked network.
   - Check ACL statistics with `show access-lists`.

---

### **Verification and Questions**
- **VLANs and Trunking**:
  - Are VLANs configured correctly, and can PC1 communicate with SRV1?
  - Can PC2 communicate with SRV1 and other VLAN 20 devices?
  - Is the trunk link passing traffic for allowed VLANs?
- **OSPF**:
  - Are OSPF neighbors forming correctly?
  - Can R1 and R2 exchange routes?
- **Ethernet Standards**:
  - Are the correct cable types used for each connection?
- **Subnetting**:
  - Are the subnets correctly calculated and applied?
- **ACL**:
  - Does the ACL block traffic as intended while allowing other communication?

---

### **Submission**
Document your configuration commands and verification outputs for each task. Highlight any troubleshooting steps taken. Let me know if you encounter any issues or have questions during the lab!


