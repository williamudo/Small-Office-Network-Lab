# Small Office Network in Cisco Packet Tracer

A simulated office network with two departments, an internal server room, and a router to an outside network. Workstations get their addresses automatically over DHCP, look up names through an internal DNS server, and browse three websites: two hosted inside the network and one reached through the router.

!![image](/screenshots/topology.png)

## Skills demonstrated

- Static IP addressing for servers and router interfaces
- DHCP setup (pool, default gateway, DNS server)
- DNS setup with A records for internal and external hostnames
- HTTP servers with distinct web pages
- Basic routing between an internal LAN and an external network
- Testing with `ipconfig`, `ping`, `nslookup`, and a web browser

## Network design

Three 2960-24TT switches (Office, Datacenter, HR) are linked in a row, with the Datacenter switch in the middle. Four Office PCs connect to the Office switch, four HR PCs to the HR switch, and four servers to the Datacenter switch. The router connects the Datacenter switch to the external network, where the www.google.com server sits.

Office and HR share one subnet, so no VLANs or inter-VLAN routing are needed.

| Network | Range | Purpose |
|---|---|---|
| Internal | 192.168.10.0/24 | Workstations and internal servers |
| External | 172.12.0.0/16 | Stand-in for the outside network |

### Addressing

| Device | IP address | Subnet mask | Gateway |
|---|---|---|---|
| Router, inside (Fa0/0) | 192.168.10.1 | 255.255.255.0 | n/a |
| Router, outside (Fa1/0) | 172.12.0.1 | 255.255.0.0 | n/a |
| www.umkc.edu | 192.168.10.251 | 255.255.255.0 | 192.168.10.1 |
| DHCP server | 192.168.10.252 | 255.255.255.0 | 192.168.10.1 |
| www.abc.com | 192.168.10.253 | 255.255.255.0 | 192.168.10.1 |
| DNS server | 192.168.10.254 | 255.255.255.0 | 192.168.10.1 |
| www.google.com | 172.12.0.254 | 255.255.0.0 | 172.12.0.1 |
| 8 workstations | From DHCP | 255.255.255.0 | 192.168.10.1 |

### Cabling

| Link | Cable |
|---|---|
| PCs and servers to their switch | Copper straight-through |
| Router Fa0/0 to Datacenter switch | Copper straight-through |
| Office and HR switches to Datacenter switch | Copper crossover |
| Router Fa1/0 to www.google.com | Copper crossover |

## Services

**DHCP** (192.168.10.252)
- Serves 192.168.10.0/24 with gateway 192.168.10.1 and DNS 192.168.10.254
- Addresses start at 192.168.10.10, which keeps the router (.1) and servers (.251 to .254) out of the pool

**DNS** (192.168.10.254)

| Name | Record | Address |
|---|---|---|
| www.umkc.edu | A | 192.168.10.251 |
| www.abc.com | A | 192.168.10.253 |
| www.google.com | A | 172.12.0.254 |

**HTTP**
- Running on the umkc, abc, and google servers, each with its own index page

## Router configuration

The full running configuration is in [`router-config.txt`](router-config.txt). The parts that matter:

```
interface FastEthernet0/0
 ip address 192.168.10.1 255.255.255.0
!
interface FastEthernet1/0
 ip address 172.12.0.1 255.255.0.0
```

No static routes are needed. Both networks are directly connected to the router, so it routes between them on its own.

## How to run it

1. Open `Student_Network_Build.pkt` in Cisco Packet Tracer 9.0.1 or later.
2. Wait for the link lights to turn green.
3. On any workstation, open **Desktop > IP Configuration** and select **DHCP**.
4. Use the checks below.

## Verification

From a workstation's command prompt:

```
ipconfig /all
ping 192.168.10.1
ping 172.12.0.254
nslookup www.umkc.edu
```

The workstation should show an address in 192.168.10.10 or higher, gateway 192.168.10.1, and DNS 192.168.10.254. The first ping may time out while ARP resolves, so run it twice.

Then open the web browser and visit `www.umkc.edu`, `www.abc.com`, and `www.google.com`. Each should load its own page.

From the router's CLI:

```
show ip interface brief
show ip route
```

Both interfaces should be **up/up**, and the routing table should list 192.168.10.0/24 and 172.12.0.0/16 as connected.

## Repository contents

| File | Description |
|---|---|
| `Student_Network_Build.pkt` | Packet Tracer project file |
| `router-config.txt` | Cleaned router running configuration |
| `images/topology.png` | Topology diagram |

## Notes

- 172.12.0.0/16 is a public address range. The private range is 172.16.0.0/12, which starts at 172.16.x.x. It is used here only as an outside network in a closed lab and never touches the real internet.
- Cisco Packet Tracer is free to download with a Cisco Networking Academy account.
