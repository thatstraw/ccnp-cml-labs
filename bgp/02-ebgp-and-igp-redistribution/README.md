# Lab 02 — eBGP Fundamentals and IGP Redistribution

A single-AS eBGP peering (AS 65100 ↔ AS 65200) with three IGP stub routers hanging off the AS 65100
edge router, each reachable by a different mechanism: EIGRP, OSPF, and a static route. Used to
practise bringing non-BGP-learned routes into BGP.

**➡ [Task sheet](tasks.md)** — objective, requirements, restrictions and grading criteria.
**➡ [topology.yaml](topology.yaml)** — importable CML topology.

## Topology

```
                         R3 (EIGRP AS 1)
                         Lo0 192.168.3.3/32
                                │ 10.13.1.0/24
                                │ .1        .3
R4 ── 10.14.1.0/24 ── R1 (AS 65100) ── 10.12.1.0/24 ── R2 (AS 65200)
Lo0 192.168.4.4/32     .4    .1  Lo0 192.168.1.1/32  .1   .2  Lo0 192.168.2.2/32
(no routing protocol —         │ 10.15.1.0/24
 static route on R1 only)      │ .1        .5
                         R5 (OSPF area 0)
                         Lo0 192.168.5.5/32
```

| Node | Role | Loopback0 | Interfaces | Routing |
|---|---|---|---|---|
| R1 (hostname `AS65100`) | eBGP speaker, IGP↔BGP boundary | `192.168.1.1/32` | Et0/0 `10.12.1.1/24` (R2), Et0/1 `10.13.1.1/24` (R3), Et0/2 `10.14.1.1/24` (R4), Et0/3 `10.15.1.1/24` (R5) | BGP 65100, EIGRP 1, OSPF 1, static route to R4's loopback |
| R2 (hostname `AS65200`) | eBGP peer | `192.168.2.2/32` | Et0/0 `10.12.1.2/24` | BGP 65200 (MP-BGP, `no bgp default ipv4-unicast`) |
| R3 | EIGRP stub | `192.168.3.3/32` | Et0/0 `10.13.1.3/24` | EIGRP AS 1 |
| R4 | Static-route stub | `192.168.4.4/32` | Et0/0 `10.14.1.4/24` | none — only reachable via R1's static route |
| R5 | OSPF stub | `192.168.5.5/32` | Et0/0 `10.15.1.5/24` | OSPF 1, area 0 |

Platform: `iol-xe` (`iol-xe-17-18-02`).

## What is pre-configured

- All addressing above, all interfaces `no shutdown`.
- EIGRP AS 1 between R1 and R3; OSPF area 0 between R1 and R5.
- A static route on R1 for R4's loopback (`ip route 192.168.4.4 255.255.255.255 10.14.1.4`).
- eBGP session R1 ↔ R2, each originating only its own loopback and the `10.12.1.0/24` link via
  `network` statements.

**Nothing redistributes the IGP or static routes into BGP.** R2 does not yet see `192.168.3.3/32`,
`192.168.4.4/32` or `192.168.5.5/32`.

## Importing

**CML web UI:** Dashboard → Lab Manager → Import → select `topology.yaml`.

**CML API:**

```bash
TOKEN=$(curl -k -s -X POST "https://<cml-host>/api/v0/authenticate" \
        -H "Content-Type: application/json" \
        -d '{"username":"<user>","password":"<pass>"}' | tr -d '"')

curl -k -X POST "https://<cml-host>/api/v0/import" \
     -H "Authorization: Bearer $TOKEN" \
     -H "Content-Type: application/json" \
     --data-binary @topology.yaml
```

## Verifying the baseline before you start

```
R1# show ip eigrp neighbors
R1# show ip ospf neighbor
R1# show ip route static
R2# show ip bgp summary
```

R1 should show one EIGRP neighbor (R3) and one FULL OSPF neighbor (R5). R2's BGP table should show
only `10.12.1.0/24` and `192.168.1.1/32` from R1 — the three stub routes are intentionally absent
until Task 1 is complete.

## Resetting

CML web UI: select all nodes → **Wipe**, then **Start**. This restores the baseline (addressing,
EIGRP, OSPF, static route, eBGP session) and discards any redistribution config.
