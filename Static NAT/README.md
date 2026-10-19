# Static NAT Lab – Cisco Packet Tracer

A two-router Cisco Packet Tracer lab used to practice **Static NAT (Network Address Translation)**. Each router serves a private LAN with three PCs. Router0 translates the private addresses of LAN A into fixed public-side addresses on the 10.0.0.0 link, so hosts in LAN B can reach them without ever seeing the private 192.168.1.0 network.

## Objective

Configure **one-to-one static NAT** on Router0 so that each LAN A host (PC0, PC1, PC2) is always reachable from LAN B through its own fixed inside global address.

## Topology

```
  PC0                                                          PC3
    \                                                          /
PC1 -- Switch0 -- Router0 ======== 10.0.0.0 ======== Router1 -- Switch1 -- PC4
    /           192.168.1.0                        192.168.2.0     \
  PC2                                                          PC5
```

## Devices

| Device           | Model              | Role                               |
|------------------|--------------------|------------------------------------|
| Router0          | Router-PT          | Gateway for LAN A and the NAT device |
| Router1          | Router-PT          | Gateway for LAN B                  |
| Switch0, Switch1 | Catalyst 2960-24TT | Layer 2 access switches            |
| PC0, PC1, PC2    | PC-PT              | Hosts in LAN A (192.168.1.0)       |
| PC3, PC4, PC5    | PC-PT              | Hosts in LAN B (192.168.2.0)       |

## Connections

| From    | To      | Cable                     |
|---------|---------|---------------------------|
| Router0 | Router1 | Copper cross-over         |
| Router0 | Switch0 | Copper straight-through   |
| Router1 | Switch1 | Copper straight-through   |
| Switch0 | PC0, PC1, PC2 | Copper straight-through |
| Switch1 | PC3, PC4, PC5 | Copper straight-through |

> In the screenshot the link between the routers is red, meaning it is down. It turns green once both router interfaces are configured and enabled with `no shutdown`. Use a cross-over cable if you connect the two routers with Ethernet. If you use a serial link instead, add a `clock rate` on the DCE side.

## IP Addressing

The screenshot only shows the network addresses. The subnet masks, host addresses, gateway addresses, interface names and the NAT addresses below are **assumptions**, so adjust them to match your lab.

### Networks

| Network | Subnet           | Purpose                              |
|---------|------------------|--------------------------------------|
| LAN A   | 192.168.1.0/24   | Private LAN behind Router0 (inside)  |
| WAN     | 10.0.0.0/24      | Link between routers (outside)       |
| LAN B   | 192.168.2.0/24   | Private LAN behind Router1           |

### Routers

| Device  | Interface | IP Address    | Subnet Mask     | NAT Role |
|---------|-----------|---------------|-----------------|----------|
| Router0 | Fa0/0     | 192.168.1.1   | 255.255.255.0   | Inside   |
| Router0 | Fa0/1     | 10.0.0.1      | 255.255.255.0   | Outside  |
| Router1 | Fa0/1     | 10.0.0.2      | 255.255.255.0   | N/A      |
| Router1 | Fa0/0     | 192.168.2.1   | 255.255.255.0   | N/A      |

### Hosts

| Device | IP Address    | Subnet Mask     | Default Gateway |
|--------|---------------|-----------------|-----------------|
| PC0    | 192.168.1.10  | 255.255.255.0   | 192.168.1.1     |
| PC1    | 192.168.1.11  | 255.255.255.0   | 192.168.1.1     |
| PC2    | 192.168.1.12  | 255.255.255.0   | 192.168.1.1     |
| PC3    | 192.168.2.10  | 255.255.255.0   | 192.168.2.1     |
| PC4    | 192.168.2.11  | 255.255.255.0   | 192.168.2.1     |
| PC5    | 192.168.2.12  | 255.255.255.0   | 192.168.2.1     |

### Static NAT Mapping Table

| Host | Inside Local (private) | Inside Global (translated) |
|------|------------------------|----------------------------|
| PC0  | 192.168.1.10           | 10.0.0.10                  |
| PC1  | 192.168.1.11           | 10.0.0.11                  |
| PC2  | 192.168.1.12           | 10.0.0.12                  |

## Configuration

### Router0 (NAT router)

```
enable
configure terminal
hostname Router0
interface fastEthernet 0/0
 ip address 192.168.1.1 255.255.255.0
 ip nat inside
 no shutdown
exit
interface fastEthernet 0/1
 ip address 10.0.0.1 255.255.255.0
 ip nat outside
 no shutdown
exit
ip nat inside source static 192.168.1.10 10.0.0.10
ip nat inside source static 192.168.1.11 10.0.0.11
ip nat inside source static 192.168.1.12 10.0.0.12
ip route 0.0.0.0 0.0.0.0 10.0.0.2
end
write memory
```

### Router1

```
enable
configure terminal
hostname Router1
interface fastEthernet 0/1
 ip address 10.0.0.2 255.255.255.0
 no shutdown
exit
interface fastEthernet 0/0
 ip address 192.168.2.1 255.255.255.0
 no shutdown
exit
end
write memory
```

Router1 needs no route to 192.168.1.0. It reaches the translated addresses (10.0.0.10–12) directly because they are on the same 10.0.0.0/24 link. Leaving that route out also proves that LAN B cannot reach LAN A's private addresses.

### Switches and PCs

The switches need no configuration. On each PC, open **Desktop → IP Configuration** and enter the values from the host table.

## How Static NAT Works

1. PC0 (192.168.1.10) sends a packet to PC3 (192.168.2.10).
2. The packet reaches Router0 on its **inside** interface.
3. Router0 rewrites the source address to **10.0.0.10** (inside global) and forwards it out its **outside** interface.
4. PC3 sees the packet coming from 10.0.0.10 and replies to that address.
5. Router0 receives the reply, translates the destination back to 192.168.1.10 and delivers it to PC0.

Because the mapping is permanent and one-to-one, hosts in LAN B can also start a connection **to** 10.0.0.10, and Router0 forwards it to PC0.

## Testing

| Test                                | Expected result |
|-------------------------------------|-----------------|
| PC0 → PC3 (192.168.2.10)            | Succeeds        |
| PC1 → PC4 (192.168.2.11)            | Succeeds        |
| PC3 → 10.0.0.10 (PC0's global address) | Succeeds     |
| PC3 → 192.168.1.10 (PC0's private address) | Fails (no route, NAT hides it) |
| PC0 → Router1 (10.0.0.2)            | Succeeds        |

Verification commands on Router0:

```
show ip nat translations
show ip nat statistics
show running-config
```

`show ip nat translations` lists each static mapping. After a ping, the entries show the inside local and inside global addresses side by side. To watch translations happen live, use `debug ip nat` and ping from PC0 to PC3.

## Learning Objectives

- Understand inside local, inside global, outside local and outside global addresses
- Mark interfaces with `ip nat inside` and `ip nat outside`
- Configure one-to-one static NAT mappings
- Verify translations with `show ip nat translations`
- Understand why private addresses are not routable across networks

## Possible Extensions

- Replace static NAT with **dynamic NAT** using a pool and ACL
- Configure **PAT (NAT overload)** so all LAN A hosts share one address
- Add NAT on Router1 for LAN B as well
- Use Simulation mode in Packet Tracer to inspect packet headers before and after translation

## Tools

- Cisco Packet Tracer
