# 09 — Client Configuration, Domain Join and Authentication

The Windows 10 machine, how it joined the domain, and what changed about signing in.

## Evidence

- Image: `09_01_VirtualBox_Client1_VM.png`
- Image: `09_02_Client_ipconfig_DHCP_Gateway.png`
- Image: `09_03_Domain_Join_Dialog.png`
- Image: `09_04_DHCP_Lease_ADUC_CLIENT1.png`

## Caption

I built a Windows 10 Pro 22H2 client with a single adapter on the internal network, confirmed it had a
lease and could resolve the domain, then joined it to `mydomain.com` and signed in with a domain account
rather than a local one. Windows 10 is past end of support, which is acceptable in an isolated lab and
would not be in a production estate.

## The machine

- Windows 10 Pro, version 22H2 — end of support 14 October 2025
- One adapter, Internal Network `internal`, DHCP
- Lease `172.16.0.102`, gateway `172.16.0.1`, DNS `172.16.0.1`, suffix `mydomain.com`
- Computer object `CLIENT1` in `Computers\Workstations`

Pro rather than Home, because Home cannot join a domain. One adapter rather than two, because the client
has no business routing anything: it needs one path to the domain and the controller provides the way
out.

## Before joining

1. `ipconfig /all` confirms the lease, the gateway, the DNS server and the connection-specific suffix.
2. `ping 172.16.0.1` reaches the controller.
3. `nslookup mydomain.com` returns the controller's internal address and nothing else.

Step 3 is the one that matters. A domain join is a DNS operation before it is an authentication
operation: the client asks DNS for the SRV records that locate a domain controller, so a client that can
ping the server but not resolve the domain will fail to join with an error that sounds like a credential
problem.

## The join

I joined it through the system properties dialog, entering `mydomain.com` and supplying domain
administrator credentials, then restarted. After the restart the sign-in screen offers **Other user**, and
the computer account `CLIENT1` sits in `Computers\Workstations` in ADUC, because I moved it there so
computer policy would reach it.

Moving the computer object is not cosmetic. Computer configuration in a GPO applies to computer objects
in the linked OU, so a workstation left in the built-in `CN=Computers` container receives none of the
baseline settings.

## Authentication

- `whoami` on the client returns `mydomain\<user>`, not `<computername>\<user>`.
- Signing in with `MYDOMAIN\<username>` and the domain password succeeds; the same username without the
  domain prefix, or with a local account, is a different identity entirely.
- A disabled account is refused with "Your account has been disabled.", which is the evidence that
  disabling is enforced at authentication rather than being a label in a console.

## RSAT

The client received the RSAT Active Directory tools so delegation could be tested the way a helpdesk
would use it, from a workstation:

```powershell
Get-WindowsCapability -Online -Name "Rsat.ActiveDirectory.DS-LDS*" | Add-WindowsCapability -Online
```

It completed with `Online: True` and `RestartNeeded: False`. ADUC was then started with **Run as
different user** so the helpdesk account could be tested without signing out of the client.

## Policy evidence

Group Policy results are taken on the controller with Group Policy Management, `Get-GPInheritance`,
`Get-GPO` and `net accounts`. The client is here to prove the domain works: the join, the DHCP lease, a
domain sign-in and a refusal. With one client, users sign in one department at a time rather than
several machines working at once.
