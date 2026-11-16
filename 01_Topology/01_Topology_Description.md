# 01 — Topology, Machines and Addressing

## Evidence

- Image: `01_Network_Diagram_Reference.png`
- Image: `02_VirtualBox_DC_VM_Settings.png`
- Image: `03_DC_ipconfig_Network_Adapters.png`
- Image: `04_02_DHCP_Scope_Range.png`
- Image: `09_02_Client_ipconfig_DHCP_Gateway.png`

## What it shows

Two virtual machines on one host. The domain controller has two adapters: adapter 1 on VirtualBox **NAT**
for outbound internet, adapter 2 on an **Internal Network** named `internal` carrying the domain. The
client has one adapter on the same internal network and takes its address from the controller.

NAT is how VirtualBox gives a guest outbound access; the internal network is a virtual switch that exists
only between guests on this host. The two are separate on purpose: the domain lives on a stable, isolated
range, and the NAT adapter exists so the controller can reach Windows Update and download RSAT.

## Machines

### `DC`

- Windows Server 2019 Standard (Desktop Experience), 2048 MB, 4 vCPU, 50 GB
- Hostname `DomainController`, NetBIOS `DOMAINCONTROLLE` (the fifteen-character limit)
- Adapter 1: NAT, `10.0.2.15/24`, gateway `10.0.2.2`, from VirtualBox DHCP
- Adapter 2: Internal Network `internal`, `172.16.0.1/24`, no gateway, DNS `127.0.0.1`
- Roles: AD DS, DNS, DHCP, Remote Access with NAT, Print and Document Services

### `client1`

- Windows 10 Pro 22H2, end of support 14 October 2025, 3048 MB, 4 vCPU
- Adapter: Internal Network `internal`, DHCP
- Lease `172.16.0.102/24`, gateway and DNS `172.16.0.1`, suffix `mydomain.com`
- Joined to `mydomain.com`; computer object in `Computers\Workstations`

Windows 10 Pro rather than Home, because Home cannot join a domain.

## Addressing

The internal network is `172.16.0.0/24`, mask `255.255.255.0`, usable `172.16.0.1` to `.254`. It is from
the private range, so it cannot collide with anything routable, and it is not the range the host's own
network uses.

- Controller: static `172.16.0.1`, no gateway on that adapter
- DHCP scope: `172.16.0.100` to `172.16.0.200`, lease 8 days, with the router, DNS and domain-name
  options set (`../04_DNS_and_DHCP/04_DHCP_Configuration.md`)
- Client: `172.16.0.102`, the only live lease

The pool starts at `.100` so that infrastructure added later — a printer, a second server, a switch
management address — can take a static address below it without ever being inside the pool. The scope
carries no reservations: an address taken from the pool is what showed the scope and its options working.

No gateway on the controller's internal adapter, because two default routes on one machine is ambiguous.
RRAS does the routing instead, and the NAT adapter is the only path out.
