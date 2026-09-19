# BGP Labs

Border Gateway Protocol labs — path selection, route filtering, attribute manipulation and
address-family structure.

| # | Lab | Focus | Nodes | Autonomous Systems |
|---|---|---|---|---|
| 01 | [Route Filtering and Address Family Manipulation](01-route-filtering-af-manipulation/) | `distribute-list`, `prefix-list`, AS_PATH `filter-list`, `route-map` weight | 4 × IOSv | 65100 / 65200 / 65300 / 65400 |
| 02 | [eBGP Fundamentals and IGP Redistribution](02-ebgp-and-igp-redistribution/) | `redistribute eigrp`/`ospf`/`static` into BGP | 5 × IOL-XE | 65100 / 65200 |
| 03 | [Route Summarization with aggregate-address](03-route-summarization-aggregate-address/) | `aggregate-address`, `summary-only`, `as-set` | 3 × IOSv | 65100 / 65200 / 65300 |
| 04 | [INE BGP Path Attributes](04-bgp-path-attributes/) | Weight, Local Preference, AS_PATH prepend, Origin, MED — plus OSPF-redistribution transit and an iBGP-connected dual-homed AS | 4 × IOSv + 2 × CSR1000v | 2 / 134 / 12 |

## Recurring conventions in these labs

Labs 02 and 03 intentionally use classic BGP syntax (`network`/`redistribute` directly under
`router bgp`, no explicit address-family) rather than the MP-BGP convention below — that's a
deliberate contrast, not an oversight. The MP-BGP conventions are specific to Lab 01.

**MP-BGP syntax is mandatory (Lab 01).** Every router disables legacy IPv4 auto-activation:

```
router bgp <LOCAL_AS>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <NEIGHBOR_IP> remote-as <REMOTE_AS>
 !
 address-family ipv4 unicast
  neighbor <NEIGHBOR_IP> activate
  network <LOOPBACK> mask 255.255.255.255
 exit-address-family
```

Peers must be explicitly activated per address family. A neighbor that is defined but never
activated will form a session and exchange nothing — a common and deliberate trap.

**Soft reset only.** Policy changes are brought into force with:

```
clear ip bgp * soft
```

A hard `clear ip bgp *` tears down the TCP session and is treated as a failure where a lab
audits session uptime.
