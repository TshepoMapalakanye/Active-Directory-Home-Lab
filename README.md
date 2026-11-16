# Active Directory Home Lab — Windows Server 2019 and Windows 10

A two-machine Windows domain built in Oracle VirtualBox: one Windows Server 2019 domain controller
running AD DS, DNS, DHCP, RRAS and the print server, and one Windows 10 Pro client joined to it. The
directory is then run like a small company — three departments, thirty staff, groups that carry the
permissions, a joiner/mover/leaver process, delegated helpdesk administration and Group Policy.

## What this is

I wanted a domain I could administer rather than a wizard I had clicked through, so I built the
platform first, then the company on top of it: addressing and DNS, then an organisational unit design
that matches how staff are actually grouped, then groups that hold permissions instead of people, then
policy that enforces the decisions instead of describing them. Every claim in this repository is backed
by a screenshot, a command output or a test result.

- Domain controller: `DC`, Windows Server 2019 Standard, hostname `DomainController`, `172.16.0.1/24`
  on the internal network, second adapter on VirtualBox NAT for internet.
- Client: `client1`, Windows 10 Pro 22H2, one adapter on the same internal network, lease
  `172.16.0.102` from the domain controller's DHCP scope.
- Domain: `mydomain.com`, NetBIOS `MYDOMAIN`, domain and forest functional level Windows Server 2016.

## Design in short

The controller has two adapters and the client has one. AD DS, DNS and DHCP all live on the controller
because a domain controller is already the DNS server the domain depends on, and a single-server lab is
the sensible shape for a home lab. The internal network is `172.16.0.0/24`; the controller holds `.1`
statically, the DHCP scope hands out `.100` to `.200`, and `172.16.0.102` is the only live lease. RRAS
with NAT on the internet-facing adapter gives the internal network a way out to Windows Update and
RSAT downloads.

The directory is thirteen organisational units. Five sit directly under the domain — `Departments`,
`Groups`, `Computers`, `Service_Accounts` and `Disabled_Users` — plus the original `_ADMINS` and
`_USERS` and the built-in `Domain Controllers`. `Departments` holds Sales, IT and HR; `Groups` holds
`Security_Groups`; `Computers` holds `Workstations`. The distinction that matters throughout: users
live in `Departments`, permissions live in groups, and machines live in their own OU so computer policy
can be linked to them.

## What I built

- AD DS forest `mydomain.com`, DNS zones `mydomain.com` and `_msdcs.mydomain.com` with secure dynamic
  update, DHCP authorised with a `/24` scope and the router, DNS and domain-name options set.
- RRAS with NAT so the isolated network reaches the internet through the host.
- Security groups for each department, for the helpdesk identity, and for the printer permissions.
  Department groups are nested inside their resource groups, so no permission entry names a person.
- Thirty staff accounts across Sales (12), IT (10) and HR (8), each carrying job title, department,
  office, company and a manager, with `_USERS` left empty because it is an intake queue, not storage.
- A joiner staged disabled and proved unable to sign in, a mover moved between departments with group
  membership following, and a leaver disabled, stripped of job title and moved to `Disabled_Users`.
- A service account denied interactive logon by policy.
- Delegated administration: the helpdesk account can create, move and reset passwords in Sales and IT,
  and can only read HR. Tested until it was refused.
- Two shared printers published in the directory, and Group Policy for the domain password and lockout
  settings plus a workstation baseline with a logon banner and a ten-minute inactivity lock.

## Repository layout

- `01_Topology/` — the two-machine design, the machines themselves and the addressing plan.
- `02_Architecture_and_Addressing/` — the domain's architecture and directory structure.
- `03_Domain_Controller_and_AD_DS/` — the controller build, role installation and forest creation.
- `04_DNS_and_DHCP/` — zones, records, scope and options as measured.
- `05_RRAS_and_NAT/` — routing, NAT and the packet path out of the lab.
- `06_Identity_Design/` — organisational units, groups, joiner/mover/leaver, delegation.
- `07_Group_Policy/` — password policy and the workstation baseline.
- `08_Printers/` — the print server and its two published queues.
- `09_Client_and_Domain_Join/` — the Windows 10 build, the join and domain authentication.
- `10_Troubleshooting/` — the three problems that came up during the build and what fixed them.
The numbered folders carry the detail. `Tshepo_Mapalakanye_Active_Directory_Home_Lab.md` at the root is
the report that reads the whole project in one pass.

## Lab limits

This is an isolated learning lab. The lab password is a single shared credential and is not reproduced
in this repository. Execution policy was set to Unrestricted to run the provisioning script, and
"password never expires" is set on the administrator and service account. Windows 10 22H2 went out of
support on 14 October 2025 and is used only inside this isolated environment.

— Tshepo Mapalakanye
