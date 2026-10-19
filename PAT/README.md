# PAT (NAT Overload) Lab – Cisco Packet Tracer

A two-router Cisco Packet Tracer lab used to practice **PAT (Port Address Translation)**, also called **NAT overload**. Each router serves a LAN with three PCs, and the routers are joined by a serial link. Router0 translates **every** LAN A host to the **single IP address** of its outside interface, using port numbers to tell the sessions apart.

## Objective

Configure PAT on Router0 so that PC0, PC1 and PC2 can all reach LAN B at the same time while sharing one outside address (10.0.0.1).

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
| Router0          | Router-PT          | Gateway for LAN A and the PAT router |
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

## Configuration

### Router0 (PAT router)

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
ip nat inside source list 1 interface serial 0/0/0 overload
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

Router1 needs no route to 192.168.1.0. Replies go back to 10.0.0.1, which is directly connected.

### Switches and PCs

The switches need no configuration. On each PC, open **Desktop → IP Configuration** and enter the values from the host table.

## How PAT Works

1. PC0, PC1 and PC2 all send traffic to LAN B at the same time.
2. Router0 matches each packet against ACL 1 and replaces the source address with **10.0.0.1** (the address of its outside interface).
3. To keep the sessions separate, Router0 also tracks a unique **port number** (or ICMP identifier) for each one in its translation table.
4. When replies arrive for 10.0.0.1, Router0 uses the port to find the right inside host and translates the packet back.

The `overload` keyword is what turns NAT into PAT. Thousands of inside hosts can share a single public address this way, which is how most home and office networks reach the Internet.

## Testing

| Test                                      | Expected result                      |
|-------------------------------------------|--------------------------------------|
| PC0 → PC3 (192.168.2.10)                  | Succeeds                             |
| PC1 → PC4 (192.168.2.11)                  | Succeeds                             |
| PC2 → PC5 (192.168.2.12)                  | Succeeds                             |
| PC3 → 192.168.1.10 (private address)      | Fails (no route, NAT hides it)       |
| PC0 → Router1 (10.0.0.2)                  | Succeeds                             |

Verification commands on Router0:

```
show ip nat translations
show ip nat statistics
clear ip nat translation *
```

After pinging from several PCs, `show ip nat translations` shows all inside local addresses (192.168.1.x) mapped to the **same** inside global address (10.0.0.1), each with a different port or ID. Use `debug ip nat` while pinging to watch translations being created.

## NAT Types Compared

| Type        | Mapping                              | Address use                    |
|-------------|--------------------------------------|--------------------------------|
| Static NAT  | One private → one fixed public       | One public address per host    |
| Dynamic NAT | One private → one public from a pool | Pool addresses are used up     |
| PAT         | Many private → one public + ports    | One public address for all     |

## Learning Objectives

- Understand how PAT differs from static and dynamic NAT
- Configure NAT overload on an interface
- See how port numbers distinguish sessions that share one address
- Verify translations with `show ip nat translations`

## Possible Extensions

- Use a pool with overload: `ip nat pool NAT_POOL 10.0.0.10 10.0.0.12 netmask 255.255.255.0` and `ip nat inside source list 1 pool NAT_POOL overload`
- Add a static PAT entry to expose a server, for example `ip nat inside source static tcp 192.168.1.10 80 10.0.0.1 80`
- Add NAT on Router1 for LAN B as well
- Use Simulation mode in Packet Tracer to inspect packet headers before and after translation

## Tools

- Cisco Packet Tracer
