# 04 — DNS Configuration

The zones and records the domain depends on, and how name registration is set up.

## Evidence

- Image: `AD_DNS_Multihomed_Before.png`

## Caption

The promotion created the two zones the domain needs, `mydomain.com` and `_msdcs.mydomain.com`, both
Active Directory integrated with secure dynamic update only. A domain controller registers its own
service records, so I did not create any of the SRV records by hand — I checked they existed, and I
stopped the second network adapter registering an address that only exists beyond NAT.

## Zones

### mydomain.com

- Type: Primary, Active Directory integrated
- Dynamic update: **Secure only**
- Replication: to all domain controllers in the domain
- Holds the host records: `@` (`172.16.0.1`), `domaincontroller` (`172.16.0.1`), `CLIENT1`
  (`172.16.0.102`), and the `DomainDnsZones` and `ForestDnsZones` alias records

Secure-only dynamic update means a client must authenticate before it may register a record. That is
what stops a workstation from claiming a name it does not own, and it is the default a domain controller
sets.

### _msdcs.mydomain.com

The domain controller locator zone. It holds the SRV records a client queries at logon, and the CNAME
that points at the controller's global catalog. If this zone is missing or empty, the domain still pings
but nothing can join it or sign in.

## Registration on a two-adapter server

The controller has two adapters, and by default both register their addresses in DNS. Only the internal
adapter registers here: the NAT adapter has **Register this connection's addresses in DNS** cleared, so
`mydomain.com` and `domaincontroller.mydomain.com` each carry the single address `172.16.0.1`. A domain
that also published `10.0.2.15` would hand clients an address that exists only beyond NAT.

## Reverse lookup zones

`0.in-addr.arpa`, `127.in-addr.arpa` and `255.in-addr.arpa` exist as the default reverse zones created
with the role. No PTR record was created for the internal subnet.

