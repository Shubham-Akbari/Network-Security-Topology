# Dynamic NAT Lab – Cisco Packet Tracer

A two-router Cisco Packet Tracer lab used to practice **Dynamic NAT (Network Address Translation)**. Each router serves a LAN with three PCs, and the routers are joined by a serial link. Router0 translates the private addresses of LAN A into addresses taken **on demand from a pool**, so LAN A hosts can reach LAN B without exposing the private 192.168.1.0 network.

## Objective

Configure **dynamic NAT** on Router0 so that any host in LAN A (PC0, PC1, PC2) is automatically assigned an address from a NAT pool when it sends traffic towards LAN B.

## Topology

```
  PC0                                                          PC3
    \                                                          /
PC1 -- Switch0 -- Router0 ======== serial ======== Router1 -- Switch1 -- PC4
    /                                                          \
  PC2                                                          PC5
```

## Devices

| Device           | Model              | Role                                 |
|------------------|--------------------|--------------------------------------|
| Router0          | Router-PT          | Gateway for LAN A and the NAT device |
| Router1          | Router-PT          | Gateway for LAN B                    |
| Switch0, Switch1 | Catalyst 2960-24TT | Layer 2 access switches              |
| PC0, PC1, PC2    | PC-PT              | Hosts in LAN A                       |
| PC3, PC4, PC5    | PC-PT              | Hosts in LAN B                       |

## Connections

| From    | To      | Cable                    |
|---------|---------|--------------------------|
| Router0 | Router1 | Serial (DCE/DTE)         |
| Router0 | Switch0 | Copper straight-through  |
| Router1 | Switch1 | Copper straight-through  |
| Switch0 | PC0, PC1, PC2 | Copper straight-through |
| Switch1 | PC3, PC4, PC5 | Copper straight-through |

> In the screenshot the serial link is red, meaning it is down. It turns green once both serial interfaces are configured, enabled with `no shutdown`, and a clock rate is set on the DCE side. Both routers need a serial module (for example HWIC-2T) installed for this link.

## IP Addressing

The screenshot has no labels, so **all addresses below are suggestions**. Adjust them to match your lab.

### Networks

| Network | Subnet           | Purpose                              |
|---------|------------------|--------------------------------------|
| LAN A   | 192.168.1.0/24   | Private LAN behind Router0 (inside)  |
| WAN     | 10.0.0.0/24      | Serial link between routers (outside)|
| LAN B   | 192.168.2.0/24   | Private LAN behind Router1           |

### Routers

| Device  | Interface | IP Address    | Subnet Mask     | NAT Role |
|---------|-----------|---------------|-----------------|----------|
| Router0 | Fa0/0     | 192.168.1.1   | 255.255.255.0   | Inside   |
| Router0 | Se0/0/0   | 10.0.0.1      | 255.255.255.0   | Outside  |
| Router1 | Se0/0/0   | 10.0.0.2      | 255.255.255.0   | N/A      |
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

### NAT Pool

| Item                | Value                         |
|---------------------|-------------------------------|
| Pool name           | NAT_POOL                      |
| Pool range          | 10.0.0.10 – 10.0.0.12         |
| Netmask             | 255.255.255.0                 |
| Addresses allowed to translate (ACL 1) | 192.168.1.0/24 |

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
interface serial 0/0/0
 ip address 10.0.0.1 255.255.255.0
 ip nat outside
 clock rate 64000
 no shutdown
exit
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat pool NAT_POOL 10.0.0.10 10.0.0.12 netmask 255.255.255.0
ip nat inside source list 1 pool NAT_POOL
ip route 0.0.0.0 0.0.0.0 10.0.0.2
end
write memory
```

> `clock rate` is only needed on the DCE end of the serial cable. Check which router has the clock icon in Packet Tracer and move the command if needed.

### Router1

```
enable
configure terminal
hostname Router1
interface serial 0/0/0
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

Router1 needs no route to 192.168.1.0. It reaches the pool addresses (10.0.0.10–12) directly because they are on the same 10.0.0.0/24 link.

### Switches and PCs

The switches need no configuration. On each PC, open **Desktop → IP Configuration** and enter the values from the host table.

## How Dynamic NAT Works

1. PC0 (192.168.1.10) sends a packet to PC3 (192.168.2.10).
2. The packet arrives on Router0's **inside** interface and matches ACL 1.
3. Router0 picks the next free address from `NAT_POOL` (for example 10.0.0.10), creates a translation entry, and rewrites the source address.
4. PC3 replies to 10.0.0.10, and Router0 translates the reply back to 192.168.1.10.
5. When the entry times out or is cleared, the pool address is returned and can be given to another host.

Unlike static NAT, mappings are **temporary and created only when inside hosts start traffic**. Hosts in LAN B cannot initiate connections to LAN A, because no translation exists until an inside host sends something first.

## Testing

| Test                                      | Expected result                      |
|-------------------------------------------|--------------------------------------|
| PC3 → 10.0.0.10 before any traffic        | Fails (no translation exists yet)    |
| PC0 → PC3 (192.168.2.10)                  | Succeeds                             |
| PC1 → PC4 (192.168.2.11)                  | Succeeds                             |
| PC2 → PC5 (192.168.2.12)                  | Succeeds                             |
| PC3 → 192.168.1.10 (private address)      | Fails (no route, NAT hides it)       |

Verification commands on Router0:

```
show ip nat translations
show ip nat statistics
clear ip nat translation *
```

After a ping, `show ip nat translations` lists the inside local address (192.168.1.x) next to the pool address it was given. Use `clear ip nat translation *` to remove the entries and watch the pool addresses be reassigned.

## Learning Objectives

- Understand the difference between static NAT, dynamic NAT and PAT
- Create a NAT pool and an ACL that selects which hosts are translated
- Link the ACL and pool with `ip nat inside source list`
- Verify translations and view pool usage with `show ip nat statistics`
- Understand why LAN A private addresses are not reachable from LAN B

## Possible Extensions

- Shrink the pool to 2 addresses and ping from all three PCs to see what happens when the pool is exhausted
- Change to **PAT (NAT overload)** with `ip nat inside source list 1 pool NAT_POOL overload`
- Combine static NAT for one host with dynamic NAT for the rest
- Use Simulation mode in Packet Tracer to inspect packet headers before and after translation

## Tools

- Cisco Packet Tracer
