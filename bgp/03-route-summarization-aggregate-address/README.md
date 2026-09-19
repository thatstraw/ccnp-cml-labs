# Lab 03 — BGP Route Summarization with aggregate-address

A three-AS eBGP chain (mirrors the CCNP ENCOR Official Cert Guide's "BGP Summarization Topology"
figure) with R1 originating three stub /24s plus its own loopback. Every router redistributes its
connected interfaces into BGP instead of using `network` statements, so the BGP table carries the
`?` (incomplete) origin code throughout — matching the book's worked example before summarization is
applied.

**➡ [Task sheet](tasks.md)** — objective, requirements, restrictions and grading criteria.
**➡ [topology.yaml](topology.yaml)** — importable CML topology.

## Topology

```
R1 (AS 65100) ──10.12.1.0/24── R2 (AS 65200) ──10.23.1.0/24── R3 (AS 65300)
   .1                    .2       .2                    .3
```

| Node | AS | Loopback0 | Other loopbacks | Interfaces |
|---|---|---|---|---|
| R1 | 65100 | `192.168.1.1/32` | Lo1 `172.16.1.1/24`, Lo2 `172.16.2.1/24`, Lo3 `172.16.3.1/24` (stub networks) | Gi0/0 `10.12.1.1/24` |
| R2 | 65200 | `192.168.2.2/32` | — | Gi0/0 `10.12.1.2/24`, Gi0/1 `10.23.1.2/24` |
| R3 | 65300 | `192.168.3.3/32` | — | Gi0/0 `10.23.1.3/24` |

Platform: `iosv` (`iosv-159-3-m12`).

## What is pre-configured

- All addressing and loopbacks above, `no shutdown` on every used interface.
- eBGP: R1↔R2, R2↔R3.
- **`redistribute connected`** on every router — no `network` statements anywhere. This is why
  every route in the BGP tables carries origin code `?` rather than `i`.
- No filtering, no summarization.

R3 should currently see `172.16.1.0/24`, `172.16.2.0/24` and `172.16.3.0/24` individually, each with
AS path `65200 65100`, origin `?`.

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
show ip bgp summary
show bgp ipv4 unicast
```

on all three routers. Every eBGP session should be up, and R3's table should list `10.12.1.0/24`,
`10.23.1.0/24`, `172.16.1.0/24`, `172.16.2.0/24`, `172.16.3.0/24`, `192.168.1.1/32`, `192.168.2.2/32`
and `192.168.3.3/32` — all with a trailing `?`.

## Resetting

CML web UI: select all nodes → **Wipe**, then **Start**. This restores the pre-summarization
baseline and discards any `aggregate-address` configuration.
