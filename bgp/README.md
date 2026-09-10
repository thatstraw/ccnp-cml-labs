# BGP Labs

Border Gateway Protocol labs — path selection, route filtering, attribute manipulation and
address-family structure.

| # | Lab | Focus | Nodes | Autonomous Systems |
|---|---|---|---|---|
| 01 | [Route Filtering and Address Family Manipulation](01-route-filtering-af-manipulation/) | `distribute-list`, `prefix-list`, AS_PATH `filter-list`, `route-map` weight | 4 × IOSv | 65100 / 65200 / 65300 / 65400 |

## Recurring conventions in these labs

**MP-BGP syntax is mandatory.** Every router disables legacy IPv4 auto-activation:

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
