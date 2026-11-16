# Active Directory Domain Design and Administration

**Windows Server 2019 and Windows 10 | VirtualBox Home Lab**
Tshepo Mapalakanye

## Project overview

I built a two-machine Windows domain from scratch in Oracle VirtualBox and then administered it like a
small company: an addressing and DNS foundation, an organisational unit design that matches how staff are
grouped, groups that hold permissions instead of people, a joiner/mover/leaver process, delegated
helpdesk administration, published print queues and Group Policy that enforces the decisions rather than
describing them.

I wanted a directory I could explain end to end rather than a wizard I had clicked through. That meant
building the platform first, then the company on top of it, and proving each part worked with a command
or a test rather than assuming it did.

## Architecture and technologies

One Windows Server 2019 domain controller and one Windows 10 Pro client, joined as `mydomain.com` with
the NetBIOS name `MYDOMAIN` and Windows Server 2016 functional levels.

The controller is dual-homed: one adapter on VirtualBox NAT for outbound internet, one on an isolated
internal network at `172.16.0.1/24` carrying the domain. The client has a single adapter on that internal
network and takes `172.16.0.102` from the controller's DHCP scope of `172.16.0.100` to `.200`. AD DS,
DNS, DHCP, RRAS with NAT and the print server all run on the controller.

The directory is thirteen organisational units: departments for people, a container for groups, a
separate OU for workstations, one for service accounts and one for leavers, with the intake queue left
empty at the end. Security groups carry the permissions: one per department, one for the helpdesk
identity, and resource groups for the printer queues. Each department group is nested inside the
resource groups it needs, so no access control entry names a person.

Thirty staff accounts sit in Sales (12), IT (10) and HR (8), each carrying job title, department, office,
company, description and a manager, with two levels in the reporting structure. Two print queues are
shared and published in the directory. Group Policy covers the domain password and lockout settings plus
a workstation baseline with a logon banner and a ten-minute inactivity lock.

## What I administered

The three lifecycle cases were run as cases, not as clicks. A joiner was pre-provisioned in the intake OU,
disabled, and proven unable to sign in until enabled and filed. Twenty-nine accounts were moved from the
intake queue into departments, with group membership following them. A leaver was disabled, stripped of
job title and department, and moved to its own OU. A service account was created and denied interactive logon by policy.

Delegation was designed rather than granted: the helpdesk account can create, move and reset passwords in
Sales and IT, can only read HR, and belongs to no privileged group. It was then tested as that user —
success on a Sales reset, refusal on an HR one, in the same session.

## Troubleshooting

Three problems came up. A domain join failed because the client was using a public DNS server
instead of the domain controller, so the SRV records that locate a domain controller could not be
resolved. A new hire could sign in but could not use the department printer, because the account
had been moved into the department OU but never added to the department group. And a user
returning from leave was refused sign-in with a logon-type error, because the account was still in
the restricted `Disabled_Users` OU. Each is written up in `10_Troubleshooting/`.

## Skills developed

Directory design with organisational units and group nesting, delegated administration with least
privilege, joiners/movers/leavers, Group Policy scope, DNS record management on a multihomed domain
controller, DHCP authorisation and options, NAT configuration, and the habit of proving a control by
watching it refuse something instead of trusting the tick box.

## Lab limits

This is an isolated home lab: one domain controller, one site, no replication, a single shared lab
credential, and a Windows 10 client that is past end of support.
