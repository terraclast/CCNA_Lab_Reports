# Lab Difficulty Level: 11/100

## Objective:
Add DHCP configuration to lab10 located at /labs/inter-vlan_routing/lab10 

---

## Setup:

### Devices:

| Number | Device   | 
|++++++++|++++++++++|
| 1      | Router   |
| 2      | Switches |
| 4      | PCs      |
|++++++++|++++++++++|

### Topology:
Same as Level 10.

---

## Instructions:
1. Configure DHCP on the router for both LANs.
2. Set each PC to obtain an IP via DHCP.
3. Verify DHCP leases.

### Testing:
- Confirm IP assignment from DHCP.
- Test connectivity between all PCs.

---

## CMD Sequence:

### R1:
```plaintext
    enable 
        conf t 
            ip dhcp pool VLAN2 network 192.168.1.0 255.255.255.0 default-router 192.168.1.1 
            ip dhcp pool VLAN3 network 192.168.2.0 255.255.255.0 default-router 192.168.2.1
```

### PC1-4:
    
    - IP addr configured to use dhcp 

---

## Result:
    - PCs were given IP addresses in the appropriate subnet for the appropriate VLAN, and all PCs were able to ping each other.

---

## Questions to Answer:

1. **What does the `ip dhcp pool` command do on the router?**
    - It creates the dhcp pool from which IP addresses are leased to hosts based on subnet and names the pool.

2. **How does a PC request an IP address via DHCP?**
    - It initiates the DORA process
        **Discover** -- The host sends a dhcp request packet into the network which should end up at a dhcp server either on a router or a dedicated server.
        **Offer**    -- The server checks their lease table and determines an acceptable host IP address to offer as a lease.  The server then sends a dhcp offer packet back to the host.
        **Request**  -- The host acknowledges receipt of the offer message and requests to be granted a lease.
        **Accept**   -- The server accepts the host's request for the lease and everyone is in agreement as to the host's new IP configuration. 

3. **What would happen if the default router was not configured in the DHCP pool?**
    - Its IP addr would regularly change, making it very difficult for hosts to keep track of their default gateway. 

4. **How can you verify that the DHCP server has successfully assigned an IP address to a PC?**
    - Check that the host recieved an IP addr in the specified range
    - From Enable mode, run:
        `show ip dhcp binding`

5. **Why is it important to have DHCP configured in a network with multiple VLANs?**    
    - To automate IP configuration across the VLANs
    - To avoid human error when configuring IP configs
    - To dynamically configure hosts as they are added and removed from the network.
