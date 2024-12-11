# Lab Difficulty Level: 12/100

## Objective
Practice subnetting and configure DHCP relay.

---

## Setup

### Devices

| Number | Device   |
|++++++++|++++++++++|
| 1      | Router   |
| 2      | Switches |
| 6      | PCs      |
|++++++++|++++++++++|

---

### Topology
Two subnets.

## Instructions
1. Use subnetting to divide 192.168.0.0/24 into two subnets.
2. Set up DHCP on the router with DHCP relay.
3. Ensure each subnet gets its designated IP range.

### Testing
- Verify DHCP leases.
- Test connectivity between PCs across subnets.

---

## Subnetting
    - Mask: 255.255.255.128 /25

### Subnet1
    - 192.168.0.0 - 192.168.0.127
    - Network: 192.168.0.1
    - Broadcast: 192.168.0.126

### Subnet2
    - 192.168.0.128 - 192.168.0.255
    - Network: 192.168.0.129
    - Broadcast: 192.168.0.254

---

## Port Assignments:

| Device | Int  | Neigh | Neigh_int |
|++++++++|++++++|+++++++|+++++++++++|
| R1     | g0/0 | SW1   | g0/1      |
|        | g0/1 | SW2   | g0/1      |
|--------|------|-------|-----------|
| SW1    | g0/1 | R1    | g0/0      |
|        | f0/1 | PC0   | f0        |
|        | f0/2 | PC1   | f0        |
|        | f0/3 | PC2   | f0        |
|--------|------|-------|-----------|
| SW2    | g0/1 | R1    | g0/1      |
|        | f0/1 | PC3   | f0        |
|        | f0/2 | PC4   | f0        |
|        | f0/3 | PC5   | f0        |
|++++++++|++++++|+++++++|+++++++++++|

---

## CMD Sequence:

### SW1:
```plaintext
    enable 
        conf t 
            hostname SW1 
            vlan 2 
                name VLAN2 
            int f0/1 
                no shut 
                desc PC0 
                switchport mode access 
                switchport access vlan 2 
            int f0/2 
                no shut 
                desc PC1 
                switchport mode access 
                switchport access vlan 2 
            int f0/3 
                no shut 
                desc PC2 
                swi mo acc 
                swi acc v 2 
            int g0/1 
                no shut 
                desc R1 
                swi mo acc 
                swi acc v 2 
            int range f0/4-24 
                shut 
            int g0/2 
                shut 
                exit 
            int range f0/1-3 
                spanning-tree portfast 
            int g0/1 spanning-tree portfast 
            ip default-gateway 192.168.0.1
```

### SW2:
```plaintext
    enable 
        conf t 
            hostname SW2 
            vlan 3 
                name VLAN3 
            int f0/1 
                no shut 
                desc PC3 
                swi mo acc 
                swi acc v 3 
            int f0/2 
                no shut 
                desc PC4 
                swi mo acc 
                swi acc v 3 
            int f0/3 
                no shut 
                desc PC5 
                swi mo acc 
                swi acc v 3 
            int g0/1 
                no shut 
                desc R1 
                swi mo acc 
                swi acc v 3 
            int range f0/4-24 
                shut 
            int g0/2 
                shut 
            int range f0/1-3 
                spanning-tree portfast 
            int g0/1 spanning-tree portfast 
            ip default-gateway 192.168.0.129
```

### R1:
```plaintext
    enable 
        conf t 
            hostname R1 
            int g0/0 
                no shut 
                desc SW1 
                ip address 192.168.0.1 255.255.255.128 
            int g0/1 
                no shut 
                desc SW2 
                ip address 192.168.0.129 255.255.255.128 
            int f0/1/0 
                no shut 
                desc R2 
                switchport mode access 
                switchport access vlan 4 
            int vlan 4 
                ip address 10.0.0.1 255.255.255.252 
                exit 
            ! int g0/0
            !     ip helper-address 10.0.0.2
            !     int g0/1
            !     ip helper-address 10.0.0.2

            ! I couldn't get DHCP setup and working on R2, going back to having DHCP configured on R1
            ip dhcp pool VLAN2 
                network 192.168.0.0 255.255.255.128 
                default-gateway 192.168.0.1 
            ip dhcp pool VLAN3 
                network 192.168.0.128 255.255.255.128 
                default-gateway 192.168.0.129
```

### R2:
```plaintext
    enable 
        conf t 
            hostname R2 
                ip dhcp pool VLAN2 
                    network 192.168.0.0 255.255.255.128 
                    default-router 192.168.0.1 
                ip dhcp pool VLAN3 
                    network 192.168.0.128 255.255.255.128 
                    default-router 192.168.0.129 
                int g0/0 
                    no shut 
                    desc R1 
                    ip address 10.0.0.2 255.255.255.252
```

--- 

## Questions to Answer:

1. **What is the purpose of DHCP relay in this lab setup?**
    - I didn't setup DHCP relay because the lab never specified that an external DHCP server was necessary.
    - So it wasn't necessary.

2. **How does subnetting divide the IP address space for the two LANs?**
    - The subnetting divides the IP space into two halfs each with 126 host addresses due to the number of subnet bits and host bits.
    - The network bits are that of a class C network: /24 then there is one more bit making it a /25 where the final octet has one subnet bit filled: 128.
    - The remaining 7 bits are host bits, "H", and via `2^H - 2 = # of host addresses` we get 126.

3. **What would happen if the DHCP relay configuration was omitted on R1?**
    - Nothing.  The lab worked perfectly. 

4. **How do you verify if a DHCP server is properly assigning IP addresses to the PCs?**
    - From enable mode, run:
        `show ip dhcp binding`
5. **What role does the `default-gateway` configuration in the DHCP pool serve?**
    - It configures which address in the subnet is set as the default gateway for that subnet and reserves that address so it will not be leased out. 
    - It also configures that address onto the hosts requesting IP configuration via DHCP services.
