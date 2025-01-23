## Port Assignments

| Device    | Port      | Nbr       | Nbr_port  |
|-----------|-----------|-----------|-----------|
|SW1        | G0/1      | SW2       | g0/1      |
|           | g0/2      | R1        | g0/0/0    |
|           | f0/1      | PC1       | f0        |
|           | f0/2      | SRV1      | f0        |
|SW2        | g0/1      | SW1       | g0/1      |
|           | g0/2      | R2        | g0/0/0    |
|           | f0/1      | PC2       | f0        |
| R1 (ISR4331)| 
| R2 (ISR4331)|

## IP assignments

| IPv4 addr         | Device    | Port      | vlan  |
|:-----------------:|:---------:|:---------:|:-----:|
| 192.168.1.1       | R1        | g0/0/0.10 | 10    |
| 192.168.1.2       | R2        | g0/0/0.10 | 10    |
| 192.168.1.3       | PC1       | f0        | 10    |
| 192.192.2.1       | R1        | g0/0/0.20 | 20    |
| 192.168.2.2       | R2        | g0/0/0.20 | 20    |
| 192.168.2.3       | PC2       | f0        | 20    |
| 192.168.2.4       | SVR1      | f0        | 20    |


## SW1 config

enable
    conf t
        hostname SW1
        vlan 10
            name Sales
        vlan 20
            name HR
        int vlan 10
            desc Sales
        int vlan 20
            desc HR
        int g0/1
            desc SW2
            switchport mode trunk
            switchport trunk allowed vlan 10,20
        int g0/2
            desc R1
            switchport mode trunk
            switchport trunk allowed vlan 10,20
        int f0/1
            desc PC1
            switchport mode access
            switchport access vlan 10
        int f0/2
            desc SRV1
            switchport mode access
            switchport access vlan 20

## SW2 config

enable
    conf t
        hostname SW2
        vlan 10
            name Sales
        vlan 20
            name HR
        int vlan 10
            desc Sales
        int vlan 20
            desc HR
        int g0/1
            desc SW1
            switchport mode trunk
            switchport trunk allowed vlan 10,20
        int g0/2
            desc R2
            switchport mode trunk
            switchport trunk allowed vlan 10,20
        int f0/1
            desc PC2
            switchport mode access
            switchport access vlan 20

## R1 config

enable
    conf t
        hostname R1
        ip routing
        int g0/0/0
            desc ROAS
            no shut
        int g0/0/0.10
            desc vlan10 gateway
            no shut
            encapsulation dot1q 10
            ip add 192.168.1.1 255.255.255.0
        int g0/0/0.20
            desc vlan20 gateway
            no shut
            encapsulation dot1q 20
            ip add 192.168.2.1 255.255.255.0
        router ospf 1
            router-id 1.1.1.1
            network 192.168.1.0 0.0.0.255 area 0
            network 192.168.2.0 0.0.0.255 area 0
        access-list 10 deny 192.168.2.0 0.0.0.255
        access-list 10 permit any
        int g0/0/0.10
            ip access-group 10 in

## R2 config

enable 
    conf t
        hostname R2
        ip routing
        int g0/0/0
            desc ROAS
            no shut
        int g0/0/0.10
            desc vlan10
            no shut
            encapsulation dot1q 10
            ip add 192.168.1.2 255.255.255.0
        int g0/0/0.20
            desc vlan20
            no shut
            encapsulation dot1q 20
            ip add 192.168.2.2 255.255.255.0
        router ospf 1
            router-id 2.2.2.2
            network 192.168.1.0 0.0.0.255 area 0
            network 192.168.2.0 0.0.0.255 area 0

## Visibility

### SW1

SW1#show int stat
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1     PC1                connected    10         auto    auto  10/100BaseTX
Fa0/2     SRV1               connected    20         auto    auto  10/100BaseTX
Gig0/1    SW2                connected    trunk      auto    auto  10/100BaseTX
Gig0/2    R1                 connected    trunk      auto    auto  10/100BaseTX

SW1#show int trunk
Port        Mode         Encapsulation  Status        Native vlan
Gig0/1      on           802.1q         trunking      1
Gig0/2      on           802.1q         trunking      1

Port        Vlans allowed on trunk
Gig0/1      10,20
Gig0/2      10,20

Port        Vlans allowed and active in management domain
Gig0/1      10,20
Gig0/2      10,20

Port        Vlans in spanning tree forwarding state and not pruned
Gig0/1      10,20
Gig0/2      10,20

SW1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/3, Fa0/4, Fa0/5, Fa0/6
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/18
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Fa0/23, Fa0/24
10   Sales                            active    Fa0/1
20   HR                               active    Fa0/2
1002 fddi-default                     active
1003 token-ring-default               active
1004 fddinet-default                  active
1005 trnet-default                    active

### SW2

SW2#show int stat
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1     PC1                connected    10         auto    auto  10/100BaseTX
Gig0/1    SW1                connected    trunk      auto    auto  10/100BaseTX
Gig0/2    R2                 connected    trunk      auto    auto  10/100BaseTX

SW2#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/2, Fa0/3, Fa0/4, Fa0/5
                                                Fa0/6, Fa0/7, Fa0/8, Fa0/9
                                                Fa0/10, Fa0/11, Fa0/12, Fa0/13
                                                Fa0/14, Fa0/15, Fa0/16, Fa0/17
                                                Fa0/18, Fa0/19, Fa0/20, Fa0/21
                                                Fa0/22, Fa0/23, Fa0/24
10   Sales                            active
20   HR                               active    Fa0/1
1002 fddi-default                     active
1003 token-ring-default               active
1004 fddinet-default                  active
1005 trnet-default                    active
SW2#show int trunk
Port        Mode         Encapsulation  Status        Native vlan
Gig0/1      on           802.1q         trunking      1
Gig0/2      on           802.1q         trunking      1

Port        Vlans allowed on trunk
Gig0/1      10,20
Gig0/2      10,20

Port        Vlans allowed and active in management domain
Gig0/1      10,20
Gig0/2      10,20

Port        Vlans in spanning tree forwarding state and not pruned
Gig0/1      10,20
Gig0/2      10,20

### R1

R1#ping 192.168.1.3

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 0/0/0 ms

R1#ping 192.168.2.3

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.2.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms

R1#ping 192.168.2.4

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.2.4, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 0/0/1 ms

R1#show ip ospf neighbor


Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/DR         00:00:35    192.168.2.2     GigabitEthernet0/0/0.20
2.2.2.2           1   FULL/BDR        00:00:35    192.168.1.2     GigabitEthernet0/0/0.10

R1#show ip route 

Gateway of last resort is not set

     192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.1.0/24 is directly connected, GigabitEthernet0/0/0.10
L       192.168.1.1/32 is directly connected, GigabitEthernet0/0/0.10
     192.168.2.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.2.0/24 is directly connected, GigabitEthernet0/0/0.20
L       192.168.2.1/32 is directly connected, GigabitEthernet0/0/0.20

### R2

R2#ping 192.168.1.3

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms

R2#ping 192.168.2.3

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.2.3, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 0/3/13 ms

R2#ping 192.168.2.4

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.2.4, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms

R2#show ip ospf neigh


Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           1   FULL/BDR        00:00:31    192.168.2.1     GigabitEthernet0/0/0.20
1.1.1.1           1   FULL/DR         00:00:31    192.168.1.1     GigabitEthernet0/0/0.10

R2#show ip route

Gateway of last resort is not set

     192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.1.0/24 is directly connected, GigabitEthernet0/0/0.10
L       192.168.1.2/32 is directly connected, GigabitEthernet0/0/0.10
     192.168.2.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.2.0/24 is directly connected, GigabitEthernet0/0/0.20
L       192.168.2.2/32 is directly connected, GigabitEthernet0/0/0.20

### PC1

ping 192.168.1.1

Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>ping 192.168.1.2

Pinging 192.168.1.2 with 32 bytes of data:

Reply from 192.168.1.2: bytes=32 time<1ms TTL=255
Reply from 192.168.1.2: bytes=32 time<1ms TTL=255
Reply from 192.168.1.2: bytes=32 time<1ms TTL=255
Reply from 192.168.1.2: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>ping 192.168.2.1

Pinging 192.168.2.1 with 32 bytes of data:

Reply from 192.168.2.1: bytes=32 time<1ms TTL=255
Reply from 192.168.2.1: bytes=32 time<1ms TTL=255
Reply from 192.168.2.1: bytes=32 time<1ms TTL=255
Reply from 192.168.2.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.2.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>ping 192.168.2.2

Pinging 192.168.2.2 with 32 bytes of data:

Reply from 192.168.2.2: bytes=32 time<1ms TTL=255
Reply from 192.168.2.2: bytes=32 time<1ms TTL=255
Reply from 192.168.2.2: bytes=32 time<1ms TTL=255
Reply from 192.168.2.2: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.2.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>ping 192.168.2.3

Pinging 192.168.2.3 with 32 bytes of data:

Reply from 192.168.2.3: bytes=32 time<1ms TTL=127
Reply from 192.168.2.3: bytes=32 time<1ms TTL=127
Reply from 192.168.2.3: bytes=32 time<1ms TTL=127
Reply from 192.168.2.3: bytes=32 time<1ms TTL=127

Ping statistics for 192.168.2.3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>ping 192.168.2.4

Pinging 192.168.2.4 with 32 bytes of data:

Reply from 192.168.2.4: bytes=32 time<1ms TTL=127
Reply from 192.168.2.4: bytes=32 time<1ms TTL=127
Reply from 192.168.2.4: bytes=32 time<1ms TTL=127
Reply from 192.168.2.4: bytes=32 time<1ms TTL=127

Ping statistics for 192.168.2.4:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

### PC2

ping 192.168.1.1

Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>ping 192.168.1.2

Pinging 192.168.1.2 with 32 bytes of data:

Reply from 192.168.1.2: bytes=32 time<1ms TTL=255
Reply from 192.168.1.2: bytes=32 time<1ms TTL=255
Reply from 192.168.1.2: bytes=32 time<1ms TTL=255
Reply from 192.168.1.2: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>ping 192.168.2.1

Pinging 192.168.2.1 with 32 bytes of data:

Reply from 192.168.2.1: bytes=32 time<1ms TTL=255
Reply from 192.168.2.1: bytes=32 time<1ms TTL=255
Reply from 192.168.2.1: bytes=32 time<1ms TTL=255
Reply from 192.168.2.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.2.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>ping 192.168.2.2

Pinging 192.168.2.2 with 32 bytes of data:

Reply from 192.168.2.2: bytes=32 time=39ms TTL=255
Reply from 192.168.2.2: bytes=32 time<1ms TTL=255
Reply from 192.168.2.2: bytes=32 time<1ms TTL=255
Reply from 192.168.2.2: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.2.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 39ms, Average = 9ms

C:\>ping 192.168.1.3

Pinging 192.168.1.3 with 32 bytes of data:

Reply from 192.168.1.3: bytes=32 time<1ms TTL=127
Reply from 192.168.1.3: bytes=32 time<1ms TTL=127
Reply from 192.168.1.3: bytes=32 time<1ms TTL=127
Reply from 192.168.1.3: bytes=32 time<1ms TTL=127

Ping statistics for 192.168.1.3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>ping 192.168.2.3

Pinging 192.168.2.3 with 32 bytes of data:

Reply from 192.168.2.3: bytes=32 time<1ms TTL=128
Reply from 192.168.2.3: bytes=32 time=11ms TTL=128
Reply from 192.168.2.3: bytes=32 time=11ms TTL=128
Reply from 192.168.2.3: bytes=32 time=11ms TTL=128

Ping statistics for 192.168.2.3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 11ms, Average = 8ms

C:\>ping 192.168.2.4

Pinging 192.168.2.4 with 32 bytes of data:

Reply from 192.168.2.4: bytes=32 time<1ms TTL=128
Reply from 192.168.2.4: bytes=32 time<1ms TTL=128
Reply from 192.168.2.4: bytes=32 time<1ms TTL=128
Reply from 192.168.2.4: bytes=32 time<1ms TTL=128

Ping statistics for 192.168.2.4:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

## Book work tasks

### Task 3

There is no direct connection between R1 and R2 so the only consideration, atm, is the cabling lengths between: 
R1 -- SW1
SW1 -- PC1
SW1 -- SRV1
SW1 -- SW2
SW2 -- R2
SW2 -- PC2
Since each are using either 1000BASE-T or 100BASE-T interfaces, all cabling should be < 100m.

### Task 4

Subnetting:

Break 172.16.0.0/20 into 8 subnets
    2^S = 8
    S = 3
    New mask: /23 -- 255.255.254.0
Useable host space
    2^H - 2 = host#
    H = 9
    host# = 510
First two subnetIDs
    172.16.0.0
    172.16.2.0

#### Configuration
##### R1
    enable
        conf t
            int g0/0/0
                ip add 172.16.0.1
##### R2
    enable
        conf t
            int g0/0/0
                ip add 172.16.2.1
### Task 5

After messing with ACLs a little bit I've done the following:

the following config doesn't work:
```
ip access-list 10 deny 192.168.2.0 0.0.0.255
ip access-list 10 permit any
int g0/0/0.10 
    ip access-group 10 in
```
Seems to be an issue with how packettracer handles routing ACLs. 
Moving the ACL to int g0/0/0.20 does work:
```
int g0/0/0.20
    ip access-group 10 in
```
This makes it so no device in 2.0/24 can reach the gateway, even though R2 is also available on the network. 
I've verified the application of ACL 10 on g0/0/0.10 as such
```
R1#show ip int g0/0/0.10
GigabitEthernet0/0/0.10 is up, line protocol is up (connected)
  Internet address is 192.168.1.1/24
  Broadcast address is 255.255.255.255
  Address determined by setup command
  MTU is 1500 bytes
  Helper address is not set
  Directed broadcast forwarding is disabled
  Outgoing access list is not set
  Inbound  access list is 10
```
But this is unexpected behavior and seems to point out limitations within packettracer.
It could be that packettracer cannot block ICMP traffic because protocol specific ACLs aren't supported by packettracer, per the interface when trying the following cmd:
```
ip access-list 10 deny icmp 192.168.2.0 0.0.0.255
```
