# 06 — Joiner, Mover and Leaver

The three things a user account does, performed in the order a real workplace does them.

## Evidence

- Image: `06_03_Script_Downloaded_Extracted.png`
- Image: `06_04_Script_ISE_ExecutionPolicy.png`
- Image: `06_05_Script_Running.png`
- Image: `06_06_Script_Users_Created_USERS_OU.png`
- Image: `AD_NewHire_Staged_Disabled.png`
- Image: `06_06_Script_Users_Created_USERS_OU.png`
- Image: `AD_Departments_Populated.png`

## Caption

I ran the three lifecycle cases end to end. The joiner was pre-provisioned in the intake OU, disabled,
and proved unable to sign in until somebody enabled it. The movers were filed from the intake queue into
their departments with group membership following them. The leaver was disabled, stripped of job title
and department, and moved out of the staff structure rather than left sitting in it with a flag.

## The intake queue

`_USERS` is treated as an intake queue, not as storage. Accounts land there and helpdesk files them into
a department. `redirusr "OU=_USERS,DC=mydomain,DC=com"` changes the default container for new user
objects so a hastily created account cannot quietly land in the built-in `CN=Users` container instead.

The queue is empty at the end of the build, and that is the result, not an oversight: everything that
arrived was filed.

## Joiner

I created the account in `_USERS` with the logon name as first initial plus surname in lowercase, ticked
**User must change password at next logon** and **Account is disabled**, and set the description to the
department and start date, such as `Sales - start 24 Nov 2026`. A sign-in attempt on the client as
`MYDOMAIN\<username>` was refused because the account was disabled. I then enabled it, moved it into
`Departments\Sales` and added it to `GG-Sales-Users`.

That order matters. Staging disabled is how a pre-hire is prepared without ever being a live credential,
and proving the refusal is what turns "the box was ticked" into "the control works".

## Mover

Twenty-nine existing accounts were moved out of `_USERS` into departments: Sales 11, IT 10, HR 8. A move
changes the object's parent container; the SID, password, group memberships and history survive, so
nobody needs a new password and nothing is recreated.

The structural move was followed by group membership, because an OU is not a permission. Moving a person
into the IT OU does not give them anything; being in `GG-IT-Users` does. Both steps are needed and
neither substitutes for the other.

## Leaver

I disabled `lmanoke`, moved it to `Disabled_Users`, cleared the title and department on the Organisation
tab, and set the description to the offboarding date and `Access removed.`

Clearing the title and department is the part most often skipped. A disabled account that still reports a live job title pollutes
every department count and any report built from one, and it leaves the impression that a person still
holds a role. The account is kept rather than deleted because its SID and history are the audit trail;
deleting it throws that away for no gain.

## The result

- Sales: 12 (eleven moved plus one hire)
- IT: 10
- HR: 8
- `_USERS`: 0
- Total staff in departments: 30
- Total user objects in the domain: 37

The 37 is thirty staff, the leaver in `Disabled_Users`, the service account, the two named administrator
accounts and the three built-ins: Administrator, Guest and krbtgt.

## Where the accounts came from

The provisioning script read a names file of 1,002 entries and created an account for every one of them.
I then cut the directory back to thirty-two accounts, because a base that size is not something I can
manage or explain in detail, and a thousand accounts bury the department structure I wanted to show.
Everything in this repository describes the trimmed base of thirty-two.

The script created the intake OU unprotected, which is why the deletion protection on `_USERS` had
to be cleared and re-applied later: ticking a box that is already ticked writes nothing.
