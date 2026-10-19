# Standard ACL Lab – Cisco Packet Tracer

A two-router network used to practice **Standard Access Control Lists (ACLs)**. Two LANs, each with a switch and three PCs, are joined by a serial WAN link. A standard ACL is then used to filter traffic between the LANs based on source IP address.

## Topology

```
   PC0                                          PC3
     \                                           /
 PC1 -- Switch0 -- Router0 ====serial==== Router1 -- Switch1 -- PC4
     /                                           \
   PC2                                          PC5
```

## Devices

| Device            | Model              | Role                       |
|-------------------|--------------------|----------------------------|
| Router0           | Router-PT          | Gateway for LAN A          |
| Router1           | Router-PT          | Gateway for LAN B          |
| Switch0, Switch1  | Catalyst 2960-24TT | Layer 2 access switches    |
| PC0, PC1, PC2     | PC-PT              | Hosts in LAN A             |
| PC3, PC4, PC5     | PC-PT              | Hosts in LAN B             |

## Connections

| From    | To      | Cable            |
|---------|---------|------------------|
| Router0 | Router1 | Serial (DCE/DTE) |
| Router0 | Switch0 | Copper straight-through |
| Router1 | Switch1 | Copper straight-through |
| Switch0 | PC0, PC1, PC2 | Copper straight-through |
| Switch1 | PC3, PC4, PC5 | Copper straight-through |

> In the screenshot the serial link is red, meaning it is down. It turns green once both serial interfaces are configured, brought up with `no shutdown`, and a clock rate is set on the DCE side.

## Suggested IP Addressing

Adjust these to match your own lab.

| Network | Subnet            | Purpose   |
|---------|-------------------|-----------|
| LAN A   | 192.168.1.0/24    | Router0 side |
| LAN B   | 192.168.2.0/24    | Router1 side |
| WAN     | 10.0.0.0/30       | Serial link  |

| Device  | Interface | IP Address    | Subnet Mask     | Default Gateway |
|---------|-----------|---------------|-----------------|-----------------|
| Router0 | Gi0/0     | 192.168.1.1   | 255.255.255.0   | N/A             |
| Router0 | Se0/0/0   | 10.0.0.1      | 255.255.255.252 | N/A             |
| Router1 | Gi0/0     | 192.168.2.1   | 255.255.255.0   | N/A             |
| Router1 | Se0/0/0   | 10.0.0.2      | 255.255.255.252 | N/A             |
| PC0     | NIC       | 192.168.1.10  | 255.255.255.0   | 192.168.1.1     |
| PC1     | NIC       | 192.168.1.11  | 255.255.255.0   | 192.168.1.1     |
| PC2     | NIC       | 192.168.1.12  | 255.255.255.0   | 192.168.1.1     |
| PC3     | NIC       | 192.168.2.10  | 255.255.255.0   | 192.168.2.1     |
| PC4     | NIC       | 192.168.2.11  | 255.255.255.0   | 192.168.2.1     |
| PC5     | NIC       | 192.168.2.12  | 255.255.255.0   | 192.168.2.1     |

## Configuration

### Router0

```
enable
configure terminal
hostname Router0
interface gigabitEthernet 0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit
interface serial 0/0/0
 ip address 10.0.0.1 255.255.255.252
 clock rate 64000
 no shutdown
exit
ip route 192.168.2.0 255.255.255.0 10.0.0.2
end
write memory
```

> `clock rate` is only needed on the DCE end of the serial cable. Check which router has the DCE end (the clock icon) in Packet Tracer.

### Router1

```
enable
configure terminal
hostname Router1
interface gigabitEthernet 0/0
 ip address 192.168.2.1 255.255.255.0
 no shutdown
exit
interface serial 0/0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
exit
ip route 192.168.1.0 255.255.255.0 10.0.0.1
end
write memory
```

### Switches and PCs

The switches need no configuration. On each PC, open **Desktop → IP Configuration** and enter the values from the addressing table.

## Standard ACL Configuration

Standard ACLs (numbers 1–99 and 1300–1999) filter **only on source IP address**. Because they cannot match destinations or ports, they should be placed **as close to the destination as possible**.

### Scenario

Block PC0 (192.168.1.10) from reaching LAN B, while all other LAN A hosts are still allowed.

### Router1 (closest to the destination)

```
enable
configure terminal
access-list 10 deny host 192.168.1.10
access-list 10 permit any
interface gigabitEthernet 0/0
 ip access-group 10 out
end
write memory
```

### Notes

- ACL statements are processed top to bottom and stop at the first match.
- Every ACL ends with an implicit `deny any`, so `permit any` is required to let the remaining traffic through.
- `out` filters traffic leaving the interface towards Switch1; `in` filters traffic entering it.

## Testing

Before applying the ACL, confirm that every PC can ping every other PC.

After applying the ACL, from the **Command Prompt** of each PC:

| Test                          | Expected result |
|-------------------------------|-----------------|
| PC0 → PC3 / PC4 / PC5         | Fails (blocked) |
| PC1 → PC3 / PC4 / PC5         | Succeeds        |
| PC2 → PC3 / PC4 / PC5         | Succeeds        |
| PC3 → PC0 / PC1 / PC2         | Succeeds        |

Verification commands on the router:

```
show access-lists
show ip interface gigabitEthernet 0/0
show running-config
```

The match counters in `show access-lists` increase every time a packet hits a rule.

## Learning Objectives

- Configure serial links and static routing between routers
- Understand how standard ACLs work and where to place them
- Use `permit`, `deny`, `host` and `any` correctly
- Apply an ACL to an interface in the right direction
- Verify and troubleshoot ACLs

## Possible Extensions

- Use a wildcard mask to block an entire range (for example `access-list 10 deny 192.168.1.0 0.0.0.255`)
- Convert to a **named** standard ACL (`ip access-list standard NAME`)
- Compare with an **extended ACL** that filters by destination and port
- Restrict VTY (Telnet/SSH) access with `access-class`

## Tools

- Cisco Packet Tracer
