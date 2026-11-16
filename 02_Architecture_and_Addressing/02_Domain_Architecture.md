# 02 — Domain Architecture

What the domain is made of, which services run on it and how the directory is organised.

## Evidence

- Image: `03_Domain_New_Forest_mydomain.png`
- Image: `06_01_OU_ADMINS_Created.png`

## Domain facts

- DNS name: `mydomain.com`
- NetBIOS name: `MYDOMAIN`
- Distinguished name: `DC=mydomain,DC=com`
- Domain functional level: Windows Server 2016
- Forest functional level: Windows Server 2016
- Domain controller: `DomainController.mydomain.com`, also the global catalog
- Domain controller's NetBIOS name: `DOMAINCONTROLLE`

The NetBIOS name is cut to fifteen characters from the hostname `DomainController`. That truncation is
not a mistake and not a typo: NetBIOS names are limited to fifteen characters, so the domain shows as
`DOMAINCONTROLLE`. Anything that expects the full word in a NetBIOS context is wrong, and the FQDN keeps
the correct spelling.

Windows Server 2016 functional levels are used because this is a single 2019 controller. Raising the
level adds features that need a second controller or newer clients to be worth anything, so the design
stays at the level a 2016-and-later forest supports.

## Services on the controller

### AD DS

Holds the directory database: users, groups, computers, organisational units. Everything else in the
build — delegation, Group Policy scope, DHCP authorisation — is expressed against objects in this
database. The NTDS service is what serves it.

### DNS

The domain's locator service. Clients find a domain controller by querying SRV records under
`_msdcs.mydomain.com`, not by being told an address, so DNS failing looks exactly like the domain being
down. Zones and records are documented in `../04_DNS_and_DHCP/04_DNS_Configuration.md`.

### DHCP

Hands the client its address, gateway, DNS server and DNS suffix in one lease, which is how
`CLIENT1.mydomain.com` registers itself in DNS without anyone typing a name.

### Remote Access (RRAS) with NAT

Routes the internal `172.16.0.0/24` network out through the NAT adapter so the lab can reach Windows
Update and download RSAT. Documented in `../05_RRAS_and_NAT/`.

### Print and Document Services

Two shared, published queues, one per department that has staff. Documented in `../08_Printers/`.

## Directory structure

Thirteen organisational units:

- `_ADMINS` — the named administrator and a retired placeholder account
- `_USERS` — the intake queue, deliberately empty at the end of the build
- `Domain Controllers` — the controller's own computer object, built in
- `Departments` — with `Sales`, `IT` and `HR` nested inside
- `Groups` — with `Security_Groups` nested inside
- `Computers` — with `Workstations` nested inside
- `Service_Accounts` — `svc-backup`
- `Disabled_Users` — the leaver

The shape follows one rule: an OU is the unit of both organisation and policy, so a container that has
policy linked to it must contain objects that belong together. That is why departments are OUs and not
just a `Department` attribute, why workstations are separated from users, and why disabled accounts are
moved out of the departments rather than left in place with a flag.

`CN=Computers` — the built-in container — coexists with `OU=Computers`. They are different objects with
different capabilities: policy cannot be linked to the container, which is exactly why the OU was
created beside it.

## What this architecture is not

The forest is a single domain with no trusts, on one domain controller in one site.
