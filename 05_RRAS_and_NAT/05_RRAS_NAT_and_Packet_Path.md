# 05 — RRAS, NAT and the Packet Path

How the isolated domain reaches the internet, and the exact path one packet takes to get there.

## Evidence

- Image: `05_01_RRAS_Role_Installed.png`
- Image: `05_02_RRAS_NAT_Public_Interface.png`
- Image: `05_03_RRAS_NAT_Configured.png`

## Caption

I installed the Remote Access role with the Routing service and configured NAT with the NAT adapter as
the public interface and the internal network as the private one. That gives the `172.16.0.0/24` network
one way out to the internet for updates and downloads while keeping the domain entirely off the host's
real network.

## Installation

I installed the Remote Access role with Routing as its role service, leaving DirectAccess and VPN out:
this lab needs neither, and they would only add services to the list.

`Get-Service RemoteAccess` reports Running with start type Automatic, and the RRAS console opens without
a configuration error. Those two facts are what distinguish an installed role from a working router.

## NAT configuration

In the Routing and Remote Access console I configured and enabled routing, chose network address
translation, and bound the adapter holding `10.0.2.15` as the public interface and the internal adapter at
`172.16.0.1` as the private one. The NAT entry appeared under IPv4, and that binding is what the
configuration capture shows.

NAT translates source addresses from the private network into the public interface's address, and
rewrites the replies on the way back. The internal network needs no route of its own and no default
gateway, because the router that performs the translation is also the internal network's only exit.

## Interface bindings in the cmdlet

`Get-RemoteAccess` reports `InternetInterface` and `InternalInterface` as blank, and
`Get-RemoteAccessRoutingDomain` reports nothing usable. The role is installed and Routing reports
Installed, but the cmdlet does not return the interface bindings, so the public and private interfaces
are shown by the configuration capture rather than by command output.

## The packet path

A client request to an external address, step by step:

1. The client at `172.16.0.102` sends to its default gateway, `172.16.0.1`, which is the controller's
   internal adapter.
2. The controller's routing table sends it out of the NAT adapter, `10.0.2.15`.
3. RRAS NAT rewrites the source from `172.16.0.102:port` to `10.0.2.15:someport` and records the mapping
   so replies can be translated back.
4. The packet leaves the guest through VirtualBox's NAT engine, which translates `10.0.2.15` to the
   host's own address — a second, independent translation.
5. The reply arrives at the host, is translated back to `10.0.2.15`, reaches the guest, and RRAS
   translates it back to `172.16.0.102`.

Two translations at two layers. That is exactly why the NAT address must never appear in a domain
record: `10.0.2.15` is only meaningful inside the guest's own NAT segment, and anything inside the
domain that resolves to it is resolving to nowhere.

## What a failure here would look like

If the client gets an address but no internet: check the gateway is `172.16.0.1`, that the NAT interface
is the one holding `10.0.2.15`, and that the internal adapter has no default gateway of its own. If the
controller has internet but the client does not, the fault is in NAT rather than in VirtualBox, because
the controller itself does not use RRAS to reach the internet.
