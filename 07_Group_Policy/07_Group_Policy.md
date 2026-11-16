# 07 — Group Policy

The two policies that enforce the design instead of describing it, and the settings inside them.

## Evidence

- Image: `AD_GPO_Password_Policy.png`
- Image: `AD_GPO_Baseline_Settings.png`

## Caption

I audited the domain's password and lockout policy before changing it, then changed exactly one value:
minimum length from seven to eight. I also built a workstation baseline — logon banner and a ten-minute
inactivity lock — and linked it to the workstations OU. Every piece of evidence comes from the domain controller, using
Group Policy Management and the Group Policy cmdlets.

## The rule that makes the design work

A Group Policy Object linked to an OU applies to the objects in that OU. User settings reach user
accounts; computer settings reach computer accounts. So computer policy is linked to
`Computers\Workstations`, where the client's computer object lives, and would never reach the machine if
it were linked only to `Departments`. Getting this backwards is the most common reason a policy appears
configured and does nothing.

## Default Domain Policy — password and lockout

The measured state of the domain before any change:

- Complexity: enabled
- Password history: 24
- Maximum age: 90 days
- Minimum age: 30 days
- Minimum length: 7
- Lockout threshold: 3 attempts
- Lockout duration: 6 hours
- Lockout observation window: 1 minute
- Reversible encryption: off


These settings work only from a GPO linked at the domain root, because password and account lockout
policy are read from the domain-level policy. The Default Domain Policy already sits there at the highest
precedence, so creating a second policy for them would be a no-op.

The proof is `net accounts` on the controller, which reports the effective values rather than the
values typed into a console: minimum length 8, maximum age 90, lockout threshold 3, lockout duration 360
minutes.

## GPO-Workstation-Baseline

Linked to `Computers\Workstations`, computer configuration only.

Security options:

- **Interactive logon: Message title** — `MYDOMAIN authorised use only`
- **Interactive logon: Message text** — a notice that the machine is part of the lab and access is
  limited to authorised users
- **Interactive logon: Machine inactivity limit** — 600 seconds


## What proves it

- `net accounts` for the password and lockout values.
- `Get-GPInheritance` on `OU=Workstations,OU=Computers,DC=mydomain,DC=com` listing
  `GPO-Workstation-Baseline` as a link.
- `Get-GPO -All` for the policy inventory. The domain carries four objects: the two defaults plus
  `GPO-Workstation-Baseline` and `GPO-User-Baseline`.
- The **Settings** tab of each policy, which lists every configured value in one view — a better record
  than screenshots of individual editors, and the source of the baseline capture above.


