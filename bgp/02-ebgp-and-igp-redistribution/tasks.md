# eBGP Fundamentals and IGP Redistribution

**Assessment lab — candidate instructions**

R1 sits at the edge of AS 65100, eBGP-peered with AS 65200 via R2. Three routers hang off R1 inside
AS 65100, each reachable through a different mechanism: EIGRP (R3), OSPF (R5), and a static route
(R4). The baseline addressing and IGP/static configuration is already in place. Complete the task
below.

---

## Topology

```
                         R3 (EIGRP AS 1)
                         Lo0 192.168.3.3/32
                                │ 10.13.1.0/24
R4 ── 10.14.1.0/24 ── R1 (AS 65100) ── 10.12.1.0/24 ── R2 (AS 65200)
Lo0 192.168.4.4/32     Lo0 192.168.1.1/32              Lo0 192.168.2.2/32
(static route on R1)           │ 10.15.1.0/24
                         R5 (OSPF area 0)
                         Lo0 192.168.5.5/32
```

## Baseline already in place — do not remove

- EIGRP AS 1 on R1 and R3 (`network 10.13.1.0 0.0.0.255`).
- OSPF area 0 on R1 and R5 (`network 10.15.1.0 0.0.0.255 area 0` / `network 192.168.5.5 0.0.0.0 area 0`).
- Static route on R1: `ip route 192.168.4.4 255.255.255.255 10.14.1.4`.
- eBGP session R1 ↔ R2, each originating its own loopback and the `10.12.1.0/24` transit link via
  `network` statements.

---

## Task 1 — Bring the IGP and static routes into BGP

**Device:** R1 (AS 65100)

### Objective

R2 (AS 65200) must learn all three of R3's, R4's and R5's loopbacks — `192.168.3.3/32`,
`192.168.4.4/32`, `192.168.5.5/32` — via BGP.

### Requirements

1. Use redistribution (`redistribute eigrp 1`, `redistribute ospf 1`, `redistribute static`) under
   `router bgp 65100` on R1. Do not add individual `network` statements for these three prefixes.
2. All three prefixes must appear in R2's BGP table with next hop `10.12.1.1`.
3. The existing eBGP session to R2 must not be torn down — apply and verify with a soft
   clear if needed (`clear ip bgp * soft`), not a hard reset.

### Restrictions

- Do not modify R2, R3, R4 or R5.
- Do not redistribute BGP back into EIGRP or OSPF — this is one-directional, IGP → BGP only.
- Do not filter or summarize anything yet; every learned prefix should appear as its own `/32`.

### Think about

`redistribute static` also picks up any other static routes on R1 — check what a bare
`redistribute static` would sweep in on a router with more than one static route, and whether a
route-map would be the safer tool on a less deliberately simple topology than this one.

---

## Grading — commands the assessor will run

| Check | Device | Command | Expected outcome |
|---|---|---|---|
| Baseline intact | R1 | `show ip eigrp neighbors` / `show ip ospf neighbor` | R3 and R5 (FULL) still present |
| Task 1 | R2 | `show ip bgp summary` | 4 prefixes received from `10.12.1.1` (was 2) |
| Task 1 | R2 | `show ip bgp` | `192.168.3.3/32`, `192.168.4.4/32`, `192.168.5.5/32` all present, next hop `10.12.1.1` |
| Task 1 | R1 | `show running-config \| section router bgp` | `redistribute eigrp 1`, `redistribute ospf 1`, `redistribute static` present |
| Session integrity | R2 | `show ip bgp neighbors 10.12.1.1` | uptime not reset by the change |
