# Lab Title: Inter-VLAN Routing with Router-on-a-Stick

## Objective
1. Configure VLANs on a switch.
2. Set up inter-VLAN routing using a router-on-a-stick configuration.
3. Verify communication between hosts in different VLANs.

---

## Topology
- **Router**: R1
- **Switch**: SW1
- **Hosts**: PC1 (VLAN 10), PC2 (VLAN 20)

## Port Wiring

| Device | Interface | Neighbor | Neigh_int |
|++++++++|+++++++++++|++++++++++|+++++++++++|
| R1     | g0/0      | SW1      |g0/1       |
| SW1    | f0/1      | PC1      | f0        |
|        | f0/2      | PC2      | f0        |
|++++++++|+++++++++++|++++++++++|+++++++++++|

## IP Scheme
| VLAN  | Subnet          | Gateway      | Host IPs        |
|-------|-----------------|--------------|-----------------|
| 10    | 192.168.10.0/24 | 192.168.10.1 | 192.168.10.10   |
| 20    | 192.168.20.0/24 | 192.168.20.1 | 192.168.20.10   |

---

## Steps

### 1. Switch Configuration (SW1)
1. **Create VLANs**:
   - VLAN 10 (Name: Sales)
   - VLAN 20 (Name: IT)
2. **Assign VLANs to Ports**:
   - Assign `f0/1` to VLAN 10 for PC1.
   - Assign `f0/2` to VLAN 20 for PC2.
3. **Configure a Trunk Link to the Router**:
   - Use `g0/1` as the trunk port to the router.

### 2. Router Configuration (R1)
1. **Create Subinterfaces for VLANs**:
   - `g0/0.10` for VLAN 10.
   - `g0/0.20` for VLAN 20.
2. **Assign IP Addresses**:
   - `192.168.10.1/24` for `g0/0.10`.
   - `192.168.20.1/24` for `g0/0.20`.
3. **Enable 802.1Q Encapsulation on Subinterfaces**.

### 3. Host Configuration
1. **Assign Static IP Addresses**:
   - PC1: `192.168.10.10/24`, Gateway: `192.168.10.1`.
   - PC2: `192.168.20.10/24`, Gateway: `192.168.20.1`.

---

## Challenges
1. **Test Connectivity**:
   - PC1 should ping PC2 successfully through inter-VLAN routing.
2. **Misconfiguration Troubleshooting**:
   - Intentionally misconfigure a VLAN or subinterface and resolve it.
3. **Verify Trunking and VLANs**:
   - Use commands like:
     - `show vlan`
     - `show interfaces trunk`
     - `show ip route`

---
## CMD Sequence

### SW1
```plaintext        
        enable
            conf t
                hostname SW1
                vlan 10
                    name Sales
                    vlan 20
                    name IT
                    int f0/1
                    no shut
                    desc PC1
                    switchport mode access
                    switchport access vlan 10
                    int f0/2
                    no shut desc PC2
                    switchport mode access
                    switchport access vlan 20
                    int g0/1
                    desc R1
                    no shut
```                    
### R1
```plaintext
        enable
            conf t
                hostname R1
                int g0/0
                    desc ROAS
                    no shut
                    int g0/0.10
                    desc VLAN10
                    encapsulation dot1Q 10
                    ip add 192.168.10.1 255.255.255.0
                    int g0/0.20
                    desc VLAN20
                    encapsulation dot1Q 20
                    ip add 192.168.20.1 255.255.255.0
```

---

## Misconfiguration

### SW1:
```plaintext
        enable
            cont f
                int g0/1
                    no switchport trunk allowed vlan 10
                    ! pings still got through
                    ! this didn't actually remove the vlan from the allowed trunk list
                    exit
                no vlan 10
                ! this broke the network
                vlan 10
                    name Sales
                    int f0/1
                    no switchport mode access
                    ! reported that the state of the port changed to up
                    ! its still forwarding frames
                    switchport mode access
                    no switchport access vlan 10
                    ! this broke the connection, too
                    switchport access vlan 10
                    ! for the misconfigurations above, it took a second for the switches to
                    ! start working again once the config was fixed.  
                    ! I don't think it took that long for the connection to go down.
```                    
