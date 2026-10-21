# Networking Labs – Cisco Packet Tracer

A collection of hands-on **Cisco Packet Tracer** labs covering core networking and network security topics. Each lab sits in its own folder with the Packet Tracer file and a README describing the topology, IP addressing, configuration commands and test steps.

## Labs

| Lab | Folder | What it covers |
|-----|--------|----------------|
| Standard ACL | [Standard ACL](./Standard%20ACL) | Filter traffic by source IP address with numbered standard ACLs |
| Extended ACL | [Extended ACL](./Extended%20ACL) | Filter by source, destination and protocol (blocking ping from Sales to Admin) |
| Static NAT | [Static NAT](./Static%20NAT) | One-to-one mapping of private addresses to fixed translated addresses |
| Dynamic NAT | [Dynamic NAT](./Dynamic%20NAT) | Translate private hosts using an on-demand pool of addresses |
| PAT | [PAT](./PAT) | NAT overload: many hosts share one address using port numbers |
| Port Security | [Port Security](./Port%20Security) | Restrict which devices can connect to a switch port |
| AAA Security | [AAA security](./AAA%20security) | Authentication, authorization and accounting for device access |
| Firewall | [Firewall](./Firewall) | Filter and control traffic between inside, outside and DMZ networks with a firewall |

## Repository Structure

```
.
├── AAA security/
├── Dynamic NAT/
├── Extended ACL/
├── Firewall/
├── PAT/
├── Port Security/
├── Standard ACL/
├── Static NAT/
└── README.md
```

Each lab folder contains:

- The Packet Tracer file (`.pkt`)
- A `README.md` with topology, addressing, configuration and testing steps

## Getting Started

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (a free Cisco Networking Academy account is required).
2. Clone this repository:
   ```bash
   git clone <repository-url>
   ```
3. Open the `.pkt` file inside any lab folder with Packet Tracer.
4. Follow that lab's README to configure the devices and run the tests.

## Topics and Skills Covered

- IP addressing and static routing
- Access control lists (standard and extended)
- Network Address Translation (static, dynamic and PAT)
- Switch security with port security
- AAA for secure device access
- Firewall configuration and zone-based traffic control
- Verification and troubleshooting with `show` and `ping` commands

## Suggested Learning Order

1. **Standard ACL** → **Extended ACL**: learn traffic filtering
2. **Static NAT** → **Dynamic NAT** → **PAT**: learn address translation, from simplest to most common
3. **Port Security** → **AAA Security**: secure switch ports and device access
4. **Firewall**: control traffic between network zones

## Tools

- Cisco Packet Tracer

## Author

[Shubham-Akbari](https://github.com/Shubham-Akbari)
