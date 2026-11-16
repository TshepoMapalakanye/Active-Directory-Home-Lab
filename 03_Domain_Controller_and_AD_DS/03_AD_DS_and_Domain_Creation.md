# 03 — AD DS Installation and the Domain

Promoting the server into the first domain controller of a new forest, and the first logon that proves
it worked.

## Evidence

- Image: `03_Domain_New_Forest_mydomain.png`
- Image: `03_DC_Promotion_Install.png`
- Image: `03_Domain_Login_MYDOMAIN_Administrator.png`
- Image: `03_ADDS_Server_Manager_Installed.png`

## Caption

I promoted the server to the first domain controller in a new forest named `mydomain.com`, chose
Windows Server 2016 functional levels, accepted the DNS warning about delegation, and set the Directory
Services Restore Mode password without recording it anywhere in this repository. The promotion runs
through prerequisite checks, then installs and reboots into the domain. I proved it by signing in as
`MYDOMAIN\Administrator` rather than by trusting the success message.

## The promotion

I promoted the server from Server Manager as a new forest, root domain `mydomain.com`, Windows Server
2016 functional levels, DNS server and global catalog on the same machine, NetBIOS name accepted as
`MYDOMAIN`, DSRM password set to a value kept out of this repository. The DNS step warned that a
delegation could not be created, which is what happens when the first controller is also the
authoritative DNS server for the zone. Prerequisites passed, and the reboot brought the server back as a
domain controller.

The prerequisite report is where a missing static address, a DNS server pointing outwards or a duplicate
domain name would have been caught before that reboot.

## What the promotion actually did

- Created the directory database and the `SYSVOL` structure the domain needs.
- Created the domain's DNS zones, `mydomain.com` and `_msdcs.mydomain.com`, and registered the SRV
  records that let clients locate the controller.
- Created the built-in containers and the default groups, including Domain Admins, Domain Users and
  Domain Computers.
- Made this server the global catalog and the holder of all five FSMO roles, which is what a
  single-controller forest looks like.

## First domain logon

Signing in as `MYDOMAIN\Administrator` rather than the local account is the proof: the local
`Administrator` and the domain `Administrator` are different accounts, and only a working directory can
authenticate the domain one. In Server Manager the computer now appears as a domain member, and ADUC and
DNS Manager are available under Tools.

## What I would check if the promotion had failed

The prerequisite report names the failing check, so the fix follows from it: a static address that is
not set, DNS pointing at an external server, or a pending rename. Because the server had already been
renamed and given its static internal address, none of those applied.
