# Extended ACL Lab – Block Ping from Sales to Admin

A three-router Cisco Packet Tracer lab used to practice **Extended Access Control Lists (ACLs)**. Three departments (Sales, Marketing and Admin) sit on separate LANs connected through serial links. An extended ACL blocks **ICMP ping from the Sales network to the Admin network**, while all other traffic is still allowed.

## Objective

> Block ping from Sales to Admin.

## Topology

```
  PC0 (182.10.0.1)                PC2 (172.12.0.1)               PC4 (192.168.15.1)
  PC1 (182.10.0.2)                PC3 (172.12.0.2)               Server0 (192.168.15.3)
        |                               |                               |
    [Switch0]                       [Switch1]                       [Switch2]
        |                               |                               |
    [Router0] ==== 10.0.0.0 ====== [Router1] ==== 11.0.0.0 ====== [Router2]
      Sales                         Marketing                         Admin
```

## Devices

| Device  | Model              | Department | Role                    |
|---------|--------------------|------------|-------------------------|
| Router0 | Router-PT          | Sales      | Gateway for Sales LAN   |
| Router1 | Router-PT          | Marketing  | Gateway for Marketing LAN, transit router |
| Router2 | Router-PT          | Admin      | Gateway for Admin LAN   |
| Switch0 | Catalyst 2960-24TT | Sales      | Access switch           |
| Switch1 | Catalyst 2960-24TT | Marketing  | Access switch           |
| Switch2 | Catalyst 2960-24TT | Admin      | Access switch           |
| PC0, PC1 | PC-PT             | Sales      | End hosts               |
| PC2, PC3 | PC-PT             | Marketing  | End hosts               |
| PC4     | PC-PT              | Admin      | End host                |
| Server0 | Server-PT          | Admin      | Server                  |

## Connections

| From    | To      | Cable                    |
|---------|---------|--------------------------|
| Router0 | Router1 | Serial (DCE/DTE)         |
| Router1 | Router2 | Serial (DCE/DTE)         |
| Router0 | Switch0 | Copper straight-through  |
| Router1 | Switch1 | Copper straight-through  |
| Router2 | Switch2 | Copper straight-through  |
| Switch0 | PC0, PC1 | Copper straight-through |
| Switch1 | PC2, PC3 | Copper straight-through |
| Switch2 | PC4, Server0 | Copper straight-through |

> In the screenshot both serial links are red, meaning they are down. They turn green once the interfaces are configured and brought up with `no shutdown`, with a clock rate set on the DCE side.

## IP Addressing

The addresses below come from the topology labels. The **subnet masks and LAN gateway addresses were not shown**, so default classful masks and `.254` gateways are assumed. Adjust them to match your lab.

### Networks

| Network   | Subnet            | Notes                  |
|-----------|-------------------|------------------------|
| Sales     | 182.10.0.0/16     | Router0 LAN            |
| Marketing | 172.12.0.0/16     | Router1 LAN            |
| Admin     | 192.168.15.0/24   | Router2 LAN            |
| WAN 1     | 10.0.0.0/8        | Router0 ↔ Router1      |
| WAN 2     | 11.0.0.0/8        | Router1 ↔ Router2      |

### Routers

| Device  | Interface | IP Address     | Subnet Mask     |
|---------|-----------|----------------|-----------------|
| Router0 | Gi0/0     | 182.10.0.254   | 255.255.0.0     |
| Router0 | Se0/0/0   | 10.0.0.1       | 255.0.0.0       |
| Router1 | Gi0/0     | 172.12.0.254   | 255.255.0.0     |
| Router1 | Se0/0/0   | 10.0.0.2       | 255.0.0.0       |
| Router1 | Se0/0/1   | 11.0.0.1       | 255.0.0.0       |
| Router2 | Gi0/0     | 192.168.15.254 | 255.255.255.0   |
| Router2 | Se0/0/0   | 11.0.0.2       | 255.0.0.0       |

### Hosts

| Device  | IP Address     | Subnet Mask     | Default Gateway |
|---------|----------------|-----------------|-----------------|
| PC0     | 182.10.0.1     | 255.255.0.0     | 182.10.0.254    |
| PC1     | 182.10.0.2     | 255.255.0.0     | 182.10.0.254    |
| PC2     | 172.12.0.1     | 255.255.0.0     | 172.12.0.254    |
| PC3     | 172.12.0.2     | 255.255.0.0     | 172.12.0.254    |
| PC4     | 192.168.15.1   | 255.255.255.0   | 192.168.15.254  |
| Server0 | 192.168.15.3   | 255.255.255.0   | 192.168.15.254  |

## Configuration

### Router0 (Sales)

```
enable
configure terminal
hostname Router0
interface gigabitEthernet 0/0
 ip address 182.10.0.254 255.255.0.0
 no shutdown
exit
interface serial 0/0/0
 ip address 10.0.0.1 255.0.0.0
 clock rate 64000
 no shutdown
exit
ip route 172.12.0.0 255.255.0.0 10.0.0.2
ip route 11.0.0.0 255.0.0.0 10.0.0.2
ip route 192.168.15.0 255.255.255.0 10.0.0.2
end
write memory
```

### Router1 (Marketing)

```
enable
configure terminal
hostname Router1
interface gigabitEthernet 0/0
 ip address 172.12.0.254 255.255.0.0
 no shutdown
exit
interface serial 0/0/0
 ip address 10.0.0.2 255.0.0.0
 no shutdown
exit
interface serial 0/0/1
 ip address 11.0.0.1 255.0.0.0
 clock rate 64000
 no shutdown
exit
ip route 182.10.0.0 255.255.0.0 10.0.0.1
ip route 192.168.15.0 255.255.255.0 11.0.0.2
end
write memory
```

### Router2 (Admin)

```
enable
configure terminal
hostname Router2
interface gigabitEthernet 0/0
 ip address 192.168.15.254 255.255.255.0
 no shutdown
exit
interface serial 0/0/0
 ip address 11.0.0.2 255.0.0.0
 no shutdown
exit
ip route 172.12.0.0 255.255.0.0 11.0.0.1
ip route 182.10.0.0 255.255.0.0 11.0.0.1
ip route 10.0.0.0 255.0.0.0 11.0.0.1
end
write memory
```

> `clock rate` is only needed on the DCE end of each serial cable. Check which end has the clock icon in Packet Tracer and move the command if needed.

### Switches and PCs

The switches need no configuration. On each PC and the server, open **Desktop → IP Configuration** and enter the values from the host table.

## Extended ACL Configuration

Extended ACLs (numbers 100–199 and 2000–2699) can filter on **source, destination, protocol and port**. They should be placed **as close to the source as possible**, so unwanted traffic is dropped before it crosses the network.

### Router0 (closest to Sales)

```
enable
configure terminal
access-list 100 deny icmp 182.10.0.0 0.0.255.255 192.168.15.0 0.0.0.255 echo
access-list 100 permit ip any any
interface gigabitEthernet 0/0
 ip access-group 100 in
end
write memory
```

### How it works

| Line | Meaning |
|------|---------|
| `deny icmp 182.10.0.0 0.0.255.255 192.168.15.0 0.0.0.255 echo` | Drops ping requests from any Sales host to any Admin host |
| `permit ip any any` | Allows all other traffic (the ACL otherwise ends with an implicit `deny any`) |
| `ip access-group 100 in` | Applies the ACL to traffic entering Router0 from the Sales LAN |

Only `echo` (ping requests) is blocked. Admin can still ping Sales, because the Sales reply is an `echo-reply`, which the ACL does not match.

## Testing

Before applying the ACL, confirm that every device can ping every other device. Then apply the ACL and test from the **Command Prompt** on each PC.

| Test                             | Expected result |
|----------------------------------|-----------------|
| PC0 → PC4 (192.168.15.1)         | Fails (blocked) |
| PC0 → Server0 (192.168.15.3)     | Fails (blocked) |
| PC1 → PC4 / Server0              | Fails (blocked) |
| PC0 / PC1 → PC2 / PC3            | Succeeds        |
| PC2 / PC3 → PC4 / Server0        | Succeeds        |
| PC4 → PC0 / PC1                  | Succeeds        |

Verification commands on Router0:

```
show access-lists
show ip interface gigabitEthernet 0/0
show running-config
```

The match counters in `show access-lists` increase each time a packet hits a rule.

## Learning Objectives

- Configure multiple routers with serial links and static routes
- Understand the difference between standard and extended ACLs
- Filter specific protocols (ICMP) between specific networks
- Place an extended ACL close to the source
- Verify and troubleshoot ACLs

## Possible Extensions

- Block only PC0 instead of the whole Sales network using `host`
- Allow Sales to reach only the web service on Server0 (`permit tcp ... eq 80`) and deny everything else
- Convert the ACL to a **named** extended ACL (`ip access-list extended NAME`)
- Replace static routes with RIP or OSPF

## Tools

- Cisco Packet Tracer
