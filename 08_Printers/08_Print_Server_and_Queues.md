# 08 — Print Server and Published Queues

Two department queues, installed on the controller, shared and published in the directory.

## Evidence

- Image: `AD_Printers_Shared.png`

## Caption

I installed the Print and Document Services role on the domain controller and created two print queues,
`SALES-PRINTER01` and `HR-PRINTER01`, each shared and published in Active Directory. Publishing is what
makes a queue addressable by name instead of by server path, which is how a user finds a printer without
being told where it lives.

## Installation

I installed Print and Document Services with Print Server as the role service. The feature was not
present beforehand, so this is a fresh install.

## The two queues

Each queue was created in Print Management against an existing port, with these settings:

- Port type: **Local Port**
- Port name: `FILE:`
- Driver: **Generic / Text Only**
- Printers: `SALES-PRINTER01`, `HR-PRINTER01`
- Sharing: **Share this printer** ticked, share name equal to the printer name, **List in the
  directory** ticked

`FILE:` as a port means print jobs are written to a file rather than sent to a printer. With no printer
hardware in the lab, the queue, the driver, the sharing and the directory publication are real and only
the output device is simulated.

## What proves it

`Get-Printer` on the controller lists both queues with `Shared` True, a share name, and `Published` True.
The capture shows the same two rows in Print Management with the Shared and Share Name columns widened
so both values are readable.

Both queues exist, are shared and are published. No print job reached a physical device, because there
is none.

## What a production deployment would add

Print management policy for driver deployment, per-department queue deployment by group policy,
point-and-print restrictions, and a print server that is not also the domain controller. All of those are
listed under lab limits in the repository README.
