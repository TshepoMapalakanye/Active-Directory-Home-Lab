# 06 — Delegation of Control and Service Accounts

Least privilege, made visible: what the helpdesk identity can do, what it is refused, and why a service
account is not a person.

## Evidence

The delegation results are the `Get-Acl` listings for Sales, IT and HR, and the console outcomes of the
sign-in tests run as the helpdesk account.

## Caption

I delegated administration instead of granting it. The helpdesk account `ktech` can create, move and
reset passwords in Sales and IT, can only read HR, and is a member of no privileged group. I tested it as
that user: the same session succeeded on a Sales password reset and was refused on an HR one, seconds
apart.

## The helpdesk identity

- Account: `ktech`
- Member of: `Domain Users` and `GG-IT-Helpdesk`, nothing else
- Not a member of Domain Admins, and deliberately not a member of `GG-IT-Users`

A named ordinary account whose only privilege is the delegation granted to it. That is the difference
between delegation and administration: `ktech` never gains the ability to change the domain, only to
manage certain objects in certain containers.

## What was delegated, and to where

### `_USERS` — the intake queue

- Create, delete and manage user accounts
- Read all user information

Enough for helpdesk to receive a new starter, set the initial password and file them.

### `Departments\Sales` and `Departments\IT`

- Create, delete and manage user accounts
- Reset user passwords and force password change at next logon
- Read all user information

### `Departments\HR`

- Read all user information — and nothing else

One checkbox is the entire security decision of this phase. HR data is the most sensitive in the
directory, so helpdesk can confirm a person exists and cannot change them. The permission lists for
Sales and HR are visibly different, and that difference is the point.

## How it was tested

- `Get-Acl` on each OU filtered to `GG-IT-Helpdesk` returns one access control entry per permission the
  wizard was asked for, so Sales and IT show several entries and HR shows read-only entries.
- Signed in as `MYDOMAIN\ktech` with RSAT on the client, a Sales password reset succeeded and an HR
  password reset returned access denied.
- A move of a staged account into Sales succeeded; a move into HR was refused.

The refusal is the permission boundary being enforced.

## Service account

- Account: `svc-backup`
- Container: `Service_Accounts`
- Password: never expires
- Denied: log on locally
- Description: `Service account for scheduled backup. No interactive sign-in.`

"Password never expires" is a defensible trade on a service account, because a scheduled job cannot
answer a password-change prompt, and it is a bad idea on a human account for the same reason in reverse.
Denying interactive logon means that even if the credential leaks it cannot be used to sit at a
desktop — the account can only be used the way it is meant to be used.

