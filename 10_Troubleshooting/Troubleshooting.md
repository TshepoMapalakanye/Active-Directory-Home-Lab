# Troubleshooting

Three problems came up while this domain was being built and used. Each is what I saw, what caused it,
and what I changed.

## Domain join failed: "The specified domain either does not exist or could not be contacted"

The client could reach the controller by address and still refused to join the domain.

**Cause.** The client's adapter was using the public DNS server `8.8.8.8` instead of the controller. A
join is a DNS lookup before it is an authentication step: the client asks DNS for the SRV records that
locate a domain controller, and a public resolver knows nothing about `mydomain.com`, so the join reported
that the domain did not exist.

**Fix.** On `client1` I changed the adapter's preferred DNS server to `172.16.0.1`, then flushed and
renewed the lease:

```
ipconfig /flushdns
ipconfig /renew
```

**Checked with** the records the join actually needs:

```
nslookup -type=SRV _ldap._tcp.dc._msdcs.mydomain.com
nslookup mydomain.com
```

The first returned `domaincontroller.mydomain.com` on port 389, the second returned `172.16.0.1` alone.
The join then completed and the computer account landed in `Computers\Workstations`.

## A new hire could not print to the department printer

The new account could sign in, but the department printer was not available to it.

**Cause.** The account had been moved into the department OU but was never added to the department group.
An OU organises objects; a group carries permissions. The printer is granted to `GG-Sales-Print`, which
holds `GG-Sales-Users`, so an account outside the department group inherits nothing whichever OU it sits
in.

**Fix.** In ADUC I added the account to `GG-Sales-Users`. Because that group is nested inside
`GG-Sales-Print`, the printer was there at the next sign-in, with no permission edited by hand.

## Logon denied: "The user has not been granted the requested logon type"

A user back from leave could not sign in, with a logon-type message rather than a wrong password.

**Cause.** The account was still in `Disabled_Users` after the leave period instead of being moved back.
That OU is restricted by security policy, so an account inside it is refused the interactive logon right
even when the account itself is enabled.

**Fix.** In ADUC I moved the account back to `Departments\Sales` and enabled it. The department group was
untouched, so the account worked again without anything being rebuilt.

## What the three had in common

None was a broken service. Two were DNS or group membership configuration, and one was an object sitting
in a container with a policy attached to it. In each case the answer came from where the object actually
was and what it was actually a member of, before looking at the service that seemed to be failing.
