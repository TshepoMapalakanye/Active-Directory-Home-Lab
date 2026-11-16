# 04 — DHCP Configuration

The authorised scope, its options as measured, and the lease the client actually received.

## Evidence

- Image: `04_01_DHCP_Role_Installed.png`
- Image: `04_02_DHCP_Scope_Range.png`
- Image: `04_03_DHCP_Option_003_Router.png`
- Image: `04_04_DHCP_Option_006_DNS.png`
- Image: `09_04_DHCP_Lease_ADUC_CLIENT1.png`

## Caption

I installed DHCP on the domain controller, authorised it in the directory, and created one scope over
`172.16.0.100` to `172.16.0.200` with an eight-day lease. The scope carries the three options a domain
client needs — router, DNS server and DNS domain name — and the client took `172.16.0.102` from it and
registered itself in DNS under `mydomain.com`.

## Authorisation

An installed DHCP server is not a serving DHCP server. In an Active Directory domain the service must
also be authorised, otherwise it starts but refuses every request. `Get-DhcpServerInDC` lists
`172.16.0.1` as authorised, which is the check that separates the two states.

Authorisation exists to stop a rogue server handing out addresses on a corporate network. Skipping it in
a lab is a common reason DHCP "does not work" while the console shows everything green.

## Scope

- Scope ID: `172.16.0.0`
- Name: `172.16.0.100-200`
- Start: `172.16.0.100`
- End: `172.16.0.200`
- Mask: `255.255.255.0`
- Lease duration: 8 days
- State: Active

The pool deliberately starts at `.100`. Everything below it — the controller at `.1`, and any printer,
second server or switch management address added later — stays outside the pool and can be static
without a reservation.

## Options

- `003 Router` — `172.16.0.1`
- `006 DNS Servers` — `172.16.0.1`
- `015 DNS Domain Name` — `mydomain.com`
- `051 Lease` — `691200` seconds

`006` pointing at the controller is what makes the client find the domain: the client asks the
controller for the SRV records, which is the only way a domain join resolves. Option `015` is what makes
the client append `mydomain.com` and register `CLIENT1.mydomain.com` in DNS instead of a bare name.

No server-level options exist, so nothing is inherited from outside the scope and every value is visible
where it is set.

## The lease

`Get-DhcpServerv4Lease` shows one active lease: `172.16.0.102`, client `CLIENT1.mydomain.com`, expiry
eight days out. `ipconfig /all` on the client shows the same address with gateway and DNS `172.16.0.1`
and suffix `mydomain.com`.

A lease alone proves the client was answered. Reading every field is what proves it was answered
correctly — the gateway, DNS server and suffix are the parts that make the domain work, and a wrong
value in any of them fails later rather than immediately.
