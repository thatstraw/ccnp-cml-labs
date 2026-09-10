# CCNP CML Labs

Hands-on lab topologies for CCNP ENCOR/ENARSI study, built for [Cisco Modeling Labs](https://www.cisco.com/go/cml).

Each lab ships as an importable CML topology plus a written task sheet. The topologies boot with a
**baseline configuration only** — addressing, loopbacks and whatever protocol scaffolding the exercise
assumes. The graded tasks are left unconfigured, so each lab can be sat as an assessment rather than
read as a walkthrough.

## Repository layout

```
bgp/
  01-route-filtering-af-manipulation/
    README.md        lab overview, topology diagram, import steps
    tasks.md         the task sheet (objectives, requirements, restrictions, grading)
    topology.yaml    importable CML topology
```

## Importing a lab

1. In the CML web UI, go to **Dashboard → Lab Manager → Import**.
2. Select the lab's `topology.yaml`.
3. Start the lab and wait for all nodes to reach *Booted*.
4. Open the task sheet — it is also embedded in the lab's own **Notes** pane inside CML, so it
   travels with the topology.

Import is also possible from the CML API:

```bash
curl -k -X POST "https://<cml-host>/api/v0/import" \
     -H "Authorization: Bearer $TOKEN" \
     -H "Content-Type: application/json" \
     --data-binary @topology.yaml
```

## Lab index

| # | Topic | Lab | Nodes | Platform |
|---|---|---|---|---|
| 1 | BGP | [Route Filtering and Address Family Manipulation](bgp/01-route-filtering-af-manipulation/) | 4 | IOSv |

## Node platform

Labs are built on `iosv` (`iosv-159-3-m12`) unless stated otherwise. If your CML install carries a
different IOSv image, adjust `image_definition` in the topology YAML before importing — the
`node_definition: iosv` line can stay as-is.

## Conventions

- Point-to-point links are `/24`, numbered from the two router IDs (R1↔R2 becomes `10.1.2.0/24`).
- A router's host octet matches its number (R2 is always `.2` on every segment it touches).
- Loopback0 is the BGP router-ID: `<n>.<n>.<n>.<n>/32`.
- Canvas annotations label every segment with its subnet, every link end with its host octet,
  and every node with a summary box of its loopback, interfaces and peers.
