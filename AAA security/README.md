# AAA Security Topology (Cisco Packet Tracer)

A Packet Tracer lab that secures router access with **AAA (Authentication, Authorization and Accounting)**. A central **AAA server (Server1)** stores the user accounts, and Router1 asks the server to check the username and password whenever someone logs in remotely. The router also keeps a local password as a backup.

## Topology

```
                     Router1
              (Router password: 123)
                        |
 PC0 ─────────────── Switch0 ─────────────── Server1
 PC1 ───────────────┘                   (AAA server)
                                    Username: nihar
                                    Password: xyz
```

## Devices

| Device | Model | Count |
|--------|-------|-------|
| Router | Router-PT (Router1) | 1 |
| Switch | Catalyst 2960-24TT (Switch0) | 1 |
| Server | Server-PT (Server1) | 1 |
| PCs | PC-PT (PC0, PC1) | 2 |

## Credentials (from the diagram)

| Where | Username | Password |
|-------|----------|----------|
| Router1 (local password) | none | `123` |
| AAA user on Server1 | `nihar` | `xyz` |

## Connections

| Link | Cable (as drawn) |
|------|------------------|
| Router1 ↔ Switch0 | Solid (straight-through) |
| Server1 ↔ Switch0 | Solid (straight-through) |
| PC0 ↔ Switch0 | Solid (straight-through) |
| PC1 ↔ Switch0 | Solid (straight-through) |

## IP Addressing

The diagram does not show IP addresses, so the values below are examples **(confirm)**. Subnet mask `255.255.255.0` is assumed.

| Device | IP Address |
|--------|-----------|
| Router1 | 192.168.1.254 |
| Server1 | 192.168.1.1 |
| PC0 | 192.168.1.2 |
| PC1 | 192.168.1.3 |

Default gateway for the PCs and server: `192.168.1.254`.

## Configuration (example)

### 1. Server1: AAA service

1. Click Server1 → **Desktop** → **IP Configuration** and set the static IP above.
2. Go to **Services** → **AAA** and turn the service **On**.
3. Add a **network configuration** (the router as an AAA client):

   | Field | Value |
   |-------|-------|
   | Client name | Router1 |
   | Client IP | 192.168.1.254 |
   | Server type | Radius |
   | Key | `secretkey` (example) |

4. Add a **user setup**:

   | Field | Value |
   |-------|-------|
   | Username | `nihar` |
   | Password | `xyz` |

### 2. Router1: interface, local password and AAA

```
enable
configure terminal
hostname Router1

interface fa0/0
 ip address 192.168.1.254 255.255.255.0
 no shutdown
exit

! Local fallback and privileged mode
enable secret 123
username admin password 123

! AAA with the RADIUS server, local database as backup
aaa new-model
radius-server host 192.168.1.1 key secretkey
aaa authentication login default group radius local

line vty 0 4
 login authentication default
 transport input telnet
end
write memory
```

The key (`secretkey`) must be **identical** on Server1 and the router. The local `admin` account is only an example fallback so you can still log in if the server is down.

### 3. PCs

Click each PC → **Desktop** → **IP Configuration** and set a static IP from the table, with `192.168.1.254` as the default gateway.

## Testing

1. On PC0, open **Desktop** → **Command Prompt**.
2. Run: `telnet 192.168.1.254`
3. Enter username `nihar` and password `xyz`. The router should accept the login through the AAA server.
4. Type `enable` and use the router password `123` to reach privileged mode.
5. Try a wrong username or password and confirm that the login is rejected.
6. Turn Server1 off and check whether the local fallback account still works.

Useful commands on the router:

- `show running-config | include aaa`
- `debug aaa authentication` (turn off with `undebug all`)

## Notes

- The username, password and router password in this README come from the labels in the diagram. They are weak and only suitable for practice.
- Telnet sends passwords in plain text. In a real network, use **SSH** with AAA.
- Packet Tracer's AAA server supports **RADIUS** and **TACACS+**. This README uses RADIUS, so switch it if your lab uses TACACS+ **(confirm)**.
