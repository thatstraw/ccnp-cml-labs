# Lab 01 — BGP Route Filtering and Address Family Manipulation

A linear four-AS eBGP chain used to practise controlling prefix propagation with four different
filtering mechanisms, plus inbound attribute manipulation, all under MP-BGP address-family syntax.

**➡ [Task sheet](tasks.md)** — objectives, requirements, restrictions and grading criteria.
**➡ [topology.yaml](topology.yaml)** — importable CML topology.

## Topology

```
   R1 (AS 65100) ──── R2 (AS 65200) ──── R3 (AS 65300) ──── R4 (AS 65400)
        Gi0/1    Gi0/1     Gi0/2    Gi0/2     Gi0/3    Gi0/3
        .1          .2     .2          .3     .3          .4
          10.1.2.0/24        10.2.3.0/24        10.3.4.0/24
```

| Node | AS | Loopback0 | Interfaces | Graded task |
|---|---|---|---|---|
| R1 | 65100 | `1.1.1.1/32` | Gi0/1 `10.1.2.1/24` | — (origin) |
| R2 | 65200 | `2.2.2.2/32` | Gi0/1 `10.1.2.2/24`, Gi0/2 `10.2.3.2/24` | Task 1 + Task 4 |
| R3 | 65300 | `3.3.3.3/32` | Gi0/2 `10.2.3.3/24`, Gi0/3 `10.3.4.3/24` | Task 2 |
| R4 | 65400 | `4.4.4.4/32` | Gi0/3 `10.3.4.4/24` | Task 3 |

Note the interface numbering: each segment uses the interface number matching its position in the
chain, so R2 faces R1 on `Gi0/1` and R3 on `Gi0/2`. `Gi0/0` is unused on every router.

## What is pre-configured

The topology boots with a baseline only:

- hostnames, `no ip domain lookup`, console `exec-timeout 0 0`
- Loopback0 and the point-to-point interfaces, `no shutdown`
- a BGP process per AS with `no bgp default ipv4-unicast`, the eBGP neighbor(s), per-AF `activate`,
  and a `network` statement for the local loopback

**Nothing from Tasks 1–5 is configured.** No access lists, prefix-lists, AS_PATH lists or route-maps
exist on any device.

## Skills exercised

| Task | Mechanism | Device |
|---|---|---|
| 1 | Standard ACL + `distribute-list` inside the address family | R2 |
| 2 | `ip prefix-list` with a `le` prefix-length modifier | R3 |
| 3 | `ip as-path access-list` with a regex + `filter-list` | R4 |
| 4 | `route-map` with a `set weight` clause | R2 |
| 5 | `clear ip bgp * soft` — non-disruptive policy activation | all |

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

The task sheet is also embedded in the lab's **Notes** pane, so it is available inside CML without
leaving the topology.

## Verifying the baseline before you start

Once all four nodes are booted, confirm the three eBGP sessions are established and each router is
originating its loopback:

```
show ip bgp summary
show ip bgp
```

R4 should see `1.1.1.1/32`, `2.2.2.2/32` and `3.3.3.3/32` before any filtering is applied. If it
does not, the tasks cannot be graded meaningfully — fix the baseline first.

## Resetting

To re-sit the lab, wipe the nodes so they reboot from the startup configuration:

- CML web UI: select all nodes → **Wipe**, then **Start**.
- This restores the baseline and discards every task configuration.
