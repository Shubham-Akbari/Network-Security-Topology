# Star Topology LAN – Cisco Packet Tracer Lab

A simple single-subnet LAN built in Cisco Packet Tracer. Three PCs and a router connect through one central switch in a **star topology**.

## Topology

```
                 [ Router0 ]
                      |
 [ PC0 ] ------ [ Switch0 ] ------ [ PC2 ]
                      |
                   [ PC1 ]
```

## Devices

| Device  | Model        | Role                         |
|---------|--------------|------------------------------|
| Router0 | Router-PT    | Default gateway for the LAN  |
| Switch0 | Catalyst 2960-24TT | Central switch (Layer 2) |
| PC0     | PC-PT        | End device                   |
| PC1     | PC-PT        | End device                   |
| PC2     | PC-PT        | End device                   |

## Connections

All links are copper straight-through cables (end device/router to switch).

| From    | To      |
|---------|---------|
| Router0 | Switch0 |
| PC0     | Switch0 |
| PC1     | Switch0 |
| PC2     | Switch0 |

> Green triangles on the links in the screenshot show that all ports are up.

## Suggested IP Addressing

Adjust these to match your own lab. Network: `192.168.1.0/24`

| Device  | Interface | IP Address    | Subnet Mask     | Default Gateway |
|---------|-----------|---------------|-----------------|-----------------|
| Router0 | Gi0/0     | 192.168.1.1   | 255.255.255.0   | N/A             |
| PC0     | NIC       | 192.168.1.10  | 255.255.255.0   | 192.168.1.1     |
| PC1     | NIC       | 192.168.1.11  | 255.255.255.0   | 192.168.1.1     |
| PC2     | NIC       | 192.168.1.12  | 255.255.255.0   | 192.168.1.1     |

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
end
write memory
```

### Switch0

No configuration is required. The 2960 works as a plug-and-play Layer 2 switch with default settings.

### PCs

For each PC, open **Desktop → IP Configuration** and enter the IP address, subnet mask and default gateway from the table above.

## How to Run

1. Open the `.pkt` file in Cisco Packet Tracer.
2. Confirm every link shows a green triangle (up).
3. Check the IP settings on each PC and the router.

## Testing

From **PC0 → Desktop → Command Prompt**:

```
ping 192.168.1.1     # Router
ping 192.168.1.11    # PC1
ping 192.168.1.12    # PC2
```

All pings should succeed. The first ping may time out while ARP resolves; this is normal.

## Learning Objectives

- Understand how a star topology works
- Practice basic router interface configuration
- Learn IP addressing and default gateways
- Verify connectivity with `ping`

## Possible Extensions

- Add VLANs on Switch0 and configure router-on-a-stick
- Configure DHCP on Router0
- Add port security on the switch
- Connect a second router and configure static or dynamic routing

## Tools

- Cisco Packet Tracer
