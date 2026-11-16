# 03 — Domain Controller Configuration

The controller's build: adapters identified, addresses set, machine named and the role installed.

## Evidence

- Image: `03_DC_ipconfig_Network_Adapters.png`
- Image: `03_DC_Static_IP_Internal.png`
- Image: `03_DC_Hostname_Rename.png`
- Image: `03_ADDS_Role_Selection.png`
- Image: `03_ADDS_Server_Manager_Installed.png`

## Caption

I built the domain controller by hand rather than from a template: installed Windows Server 2019, added
Guest Additions, identified which of the two adapters carried the internet and which would carry the
domain, gave the internal adapter a static address, named the machine `DomainController`, and then
installed the AD DS role. Identifying the adapters by their reported name and link state rather than by
their position in the list is what stops the static address landing on the NAT adapter.

## Adapter identification

Two Intel PRO/1000 adapters appear in the guest, and Windows labels them `Internet` and `Internal` only
after they are renamed. VirtualBox adapter 1 is NAT and adapter 2 is the internal network, but the guest
does not care about that ordering, so the identification was done from facts:

- The adapter holding `10.0.2.15` with a gateway is the NAT adapter — VirtualBox always serves that
  range to NAT guests.
- The adapter with no address and no gateway is the internal one.

`ipconfig /all` shows both, including which one leased its address. Capturing that output before changing
anything is what makes the static address safe to apply.

## Static addressing

On the `Internal` adapter, IPv4 properties:

- IP address: `172.16.0.1`
- Subnet mask: `255.255.255.0`
- Default gateway: blank
- Preferred DNS server: `127.0.0.1`

Leaving the gateway blank is deliberate. With two default routes, the controller can send internet-bound
traffic out of an adapter that has nowhere to route it, and the symptom is intermittent rather than
obvious. RRAS handles the outbound path instead.

## Machine name

The controller is renamed to `DomainController` before the role is installed, because renaming a domain
controller afterwards means rebuilding the domain. After the restart, `hostname` reports
`DomainController` and the NetBIOS name is derived as `DOMAINCONTROLLE` — fifteen characters, the NetBIOS
limit. Nothing is broken by that; it is what every tool that shows a NetBIOS name will display.

## Role installation

I installed Active Directory Domain Services on the local server through Add Roles and Features. DNS
Server came with the promotion rather than as a separate install, and DHCP, Remote Access and Print and
Document Services were added in their own phases.

After the install the Server Manager flag shows AD DS as installed with a promotion warning. That
warning is expected: the binaries are present and the machine is not yet a domain controller. Promotion
happens in `03_AD_DS_and_Domain_Creation.md`.

## What this configuration proves

The static address proves the internal adapter is not dependent on VirtualBox's DHCP, which is what makes
DNS and the domain survive a host reboot. The hostname proves the identity the domain was created with.
The role screenshot proves the binaries exist before promotion — the difference between "installed" and
"promoted" is the whole reason the two documents are separate.
