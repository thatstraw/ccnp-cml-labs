# Lab 04 — INE BGP Path Attributes

Companion topology for the INE video *BGP Path Attributes*: https://www.youtube.com/watch?v=Ly6wg4PdunM
— redrawn without the end-host attached to R4, which added nothing to the BGP exercise.

This is **not** a graded assessment — baseline connectivity is pre-configured so you can jump
straight into following the video and typing the Weight / Local Preference / AS_PATH / Origin / MED
commands yourself as they're introduced, without having to rebuild the topology first.

**➡ [topology.yaml](topology.yaml)** — importable CML topology.

## Topology

```
   BGP ASN: 2                    BGP ASN: 134  (OSPF area 0 core,                BGP ASN: 12
                                  tagged redistribution at each                 (R1 <-> R2 run
                                  border router, no iBGP)                        iBGP together)

                          .3           .11
              32.32.32.0/24 ---- R3 ---- 31.31.31.0/24 ---- CSR1 ---- 11.11.11.0/24 ---- R1
             /  .22                .3          .11                    .11         .1     |
   CSR2 ----+                            \                          /    \               | 112.112.112.0/24
             \  .22                34.34.34.0/24            21.21.21.0/24 \              |
              42.42.42.0/24 ---- R4                                        \             |
                          .4       .4                                       \            |
                    .22   |                                                  '---------- R2
                    |     '---------------------- 14.14.14.0/24 ---------------------.11   .2  .2
                    '------------------------- 222.222.222.0/24 (backdoor) --------------'

   CSR2 Lo0 22.22.22.22/32          R3 Lo0 3.3.3.3/32   R4 Lo0 4.4.4.4/32   CSR1 Lo0 134.134.134.134/32
                                     R1 Lo0 12.12.12.1/24, Lo1 12.21.21.1/24
                                     R2 Lo0 12.12.12.2/24, Lo1 12.21.21.2/24
```

| Node | AS | Loopback0 | Other loopbacks | Interfaces | Role |
|---|---|---|---|---|---|
| CSR2 | 2 | `22.22.22.22/32` | — | Gi1 `32.32.32.22/24`→R3, Gi2 `42.42.42.22/24`→R4, Gi3 `222.222.222.22/24`→R2 (backdoor) | Origin of `22.22.22.22/32`, dual-homed into AS134 plus a direct backdoor into AS12 |
| R3 | 134 | `3.3.3.3/32` | — | Gi0/0 `32.32.32.3/24`→CSR2, Gi0/1 `31.31.31.3/24`→CSR1, Gi0/2 `34.34.34.3/24`→R4 | AS134 border router (OSPF + eBGP to AS2) |
| R4 | 134 | `4.4.4.4/32` | — | Gi0/0 `42.42.42.4/24`→CSR2, Gi0/1 `34.34.34.4/24`→R3, Gi0/2 `14.14.14.4/24`→CSR1 | AS134 border router (OSPF + eBGP to AS2) |
| CSR1 | 134 | `134.134.134.134/32` | — | Gi1 `31.31.31.11/24`→R3, Gi2 `14.14.14.11/24`→R4, Gi3 `11.11.11.11/24`→R1, Gi4 `21.21.21.11/24`→R2 | AS134 border router (OSPF + eBGP to AS12, both R1 and R2) |
| R1 | 12 | `12.12.12.1/24` | Lo1 `12.21.21.1/24` | Gi0/0 `11.11.11.1/24`→CSR1, Gi0/1 `112.112.112.1/24`→R2 | AS12 edge router, no backdoor to AS2 |
| R2 | 12 | `12.12.12.2/24` | Lo1 `12.21.21.2/24` | Gi0/0 `21.21.21.2/24`→CSR1, Gi0/1 `112.112.112.2/24`→R1, Gi0/2 `222.222.222.2/24`→CSR2 (backdoor) | AS12 edge router, also holds the backdoor to AS2 |

Note `Lo0 12.12.12.1/24` (R1) and `Lo0 12.12.12.2/24` (R2) sit in the **same** `/24` — both routers
independently originate `12.12.12.0/24` (and `12.21.21.0/24` off Lo1), so CSR1 receives the
*identical* prefix from two different eBGP neighbors — exactly the kind of tie the video breaks with
Weight and Local Preference.

Platform: `csr1000v` (`csr1000v-17-03-08a`) for CSR1 and CSR2 — matching the source diagram's plain
`GigabitEthernet1`/`Gi2`/`Gi6`/`Gi7` numbering (renumbered here as Gi1–Gi4 per node) — and `iosv`
(`iosv-159-3-m12`) for R1–R4, using `GigabitEthernet0/x`.

## What is pre-configured

- All addressing above, all interfaces `no shutdown`.
- **OSPF area 0** among CSR1, R3 and R4 — only the three AS134-internal links plus each router's
  Loopback0. The externally-facing links (to CSR2, R1, R2) are not in OSPF.
- **No iBGP inside AS134.** Each of CSR1/R3/R4 redistributes its own eBGP-learned routes into OSPF
  (tagged 11/33/44) and redistributes OSPF back into its own BGP process (excluding its own tag, so
  it never re-advertises what it just injected back out the same eBGP session). This is how AS134
  carries AS2 ↔ AS12 reachability without a full iBGP mesh.
- **AS12 runs proper iBGP** between R1 and R2 over `112.112.112.0/24`, with `next-hop-self`.
- eBGP sessions: CSR1↔R1, CSR1↔R2, R3↔CSR2, R4↔CSR2, and the backdoor CSR2↔R2.
- MP-BGP syntax everywhere — `no bgp default ipv4-unicast`, every neighbor manually `activate`d.

**Nothing attribute-related is configured** — no weight, local-preference, AS_PATH prepending, MED
or origin manipulation. That's the video content; type it in yourself as you follow along.

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
CSR1# show ip ospf neighbor
CSR1# show ip bgp summary
CSR1# show ip bgp 12.12.12.0
```

CSR1 should show two FULL OSPF neighbors (R3, R4) and two established eBGP sessions (R1, R2), with
`12.12.12.0/24` and `12.21.21.0/24` each showing two paths.

## Resetting

CML web UI: select all nodes → **Wipe**, then **Start**. This restores the baseline and discards
anything you configured while following the video.
