# 06 — Organisational Units and Groups

The structure the directory was rebuilt into, and the groups that carry its permissions.

## Evidence

- Image: `06_01_OU_ADMINS_Created.png`
- Image: `AD_OU_Structure_Final.png`
- Image: `AD_Groups_List.png`
- Image: `06_02_Admin_Account_Created.png`
- Image: `AD_Departments_Populated.png`

## Caption

I rebuilt a flat directory into thirteen organisational units and a set of security groups: three
departments for people, a shelf for the groups, a separate OU for workstations, one for service accounts
and one for leavers. Every permission in the build is granted to a group, and each department group is
nested inside the resource groups it needs, so nobody's name appears on an access control entry.

## The thirteen organisational units

Directly under `mydomain.com`:

1. `_ADMINS` — the named administrator and a retired placeholder account
2. `_USERS` — the intake queue for new accounts, empty at the end
3. `Departments` — holds the three department OUs
4. `Groups` — holds `Security_Groups`
5. `Computers` — holds `Workstations`
6. `Service_Accounts` — holds `svc-backup`
7. `Disabled_Users` — holds the leaver
8. `Domain Controllers` — built in, holds the controller's computer object

Nested:

9. `Departments\Sales`
10. `Departments\IT`
11. `Departments\HR`
12. `Groups\Security_Groups`
13. `Computers\Workstations`

Two names mislead on purpose. `Groups` and `Security_Groups` are themselves organisational units, not
groups; `Security_Groups` holds no members and grants nothing, it is a shelf for the real groups.
`Computers` and `Workstations` are both OUs, and the built-in `CN=Computers` container sits beside the OU
with the same name — policy cannot be linked to the container, which is the whole reason the OU exists.

Two OUs were deliberately not created: `Servers`, because the only server is the domain controller and it
belongs in `Domain Controllers` where its policy is linked; and `Distribution_Lists`, because a
distribution list only routes mail and this lab has no mail system.

## The groups

All are Global, Security groups, each with a description a second administrator could act on.

Department groups, holding people:

- `GG-Sales-Users` — 12 members after the new hire
- `GG-IT-Users` — 9 members
- `GG-HR-Users` — 8 members

Role group:

- `GG-IT-Helpdesk` — the helpdesk identity, the only member is `ktech`

Resource groups, holding other groups:

- `GG-Sales-Print` — contains `GG-Sales-Users`
- `GG-HR-Print` — contains `GG-HR-Users`

## Why the IT counts disagree

The IT OU holds ten accounts but `GG-IT-Users` holds nine. `ktech` sits in the IT OU because that is
where the person is, and is kept out of the department group because its only role is helpdesk
delegation. The two numbers are meant to differ; changing either to match the other would be the
mistake.

## Why nesting

A permission is written once, on the resource group, and everyone in the department inherits it by being
in the department group. A new starter joins one group and gains the department's printer. A
mover changes department group and loses one, gains the other, in a single change. If the model were
flat — people placed directly on resources — every join and every move would mean editing every
resource separately, and the audit question "who can reach Sales?" would have no answer short of reading
every access control entry.

## Population

The departments hold thirty staff: Sales 12, IT 10, HR 8. Each account carries a job title, department,
office, company, description and a manager, with each department lead reporting to the domain
administrator so the org chart has two levels. `AD_Departments_Populated.png` shows the populated Sales
OU with the Department and Title columns added.
