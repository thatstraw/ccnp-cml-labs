# BGP Route Filtering and Address Family Manipulation

**Assessment lab — candidate instructions**

You are the Senior Network Architect for a four-provider transit chain. Each router resides in its
own Autonomous System and peers via eBGP over point-to-point `/24` links. The baseline configuration
has already been applied. Complete all five tasks below.

All requirements are graded. **Partial credit is not awarded on a task where a stated restriction has
been violated.**

---

## Topology

```
   R1 (AS 65100) ──── R2 (AS 65200) ──── R3 (AS 65300) ──── R4 (AS 65400)
        Gi0/1    Gi0/1     Gi0/2    Gi0/2     Gi0/3    Gi0/3
        .1          .2     .2          .3     .3          .4
          10.1.2.0/24        10.2.3.0/24        10.3.4.0/24
```

## Addressing

| Node | AS | Loopback0 | Interfaces |
|---|---|---|---|
| R1 | 65100 | `1.1.1.1/32` | Gi0/1 `10.1.2.1/24` |
| R2 | 65200 | `2.2.2.2/32` | Gi0/1 `10.1.2.2/24`, Gi0/2 `10.2.3.2/24` |
| R3 | 65300 | `3.3.3.3/32` | Gi0/2 `10.2.3.3/24`, Gi0/3 `10.3.4.3/24` |
| R4 | 65400 | `4.4.4.4/32` | Gi0/3 `10.3.4.4/24` |

## Baseline already in place — do not remove

Every router runs MP-BGP with legacy IPv4 auto-activation disabled:

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

> Removing `no bgp default ipv4-unicast` from **any** router fails the entire lab, regardless of
> every other outcome.

---

## Task 1 — Inbound prefix suppression on R2

**Device:** R2 (AS 65200)

### Objective

R2 must not install the prefix `1.1.1.1/32` in its BGP table. Every other prefix advertised by R1
must continue to be accepted and installed normally.

### Requirements

1. Filtering must be performed using a **standard access control list numbered 10**.
2. The filter must be applied to neighbor `10.1.2.1` in the **inbound** direction.
3. The application must be made from within the **IPv4 unicast address-family submode**, not from
   the global BGP process configuration mode.

### Restrictions

- Do not solve this task with a prefix-list, filter-list or route-map.
- Do not modify any configuration on R1.

---

## Task 2 — Prefix-list admission control on R3

**Device:** R3 (AS 65300)

### Objective

R3 must accept the prefix `2.2.2.2/32` from R2 and must reject every other prefix received on that
peering.

### Requirements

1. The filter must be an IP prefix-list named exactly **`FILTER_R2`** (case sensitive).
2. The permit entry must be written using the **`le 32`** prefix-length modifier.
3. The prefix-list must be applied to neighbor `10.2.3.2` in the **inbound** direction.

### Restrictions

- Do not solve this task with an access list or a distribute-list.

### Note

The prefix-list hit counter must be non-zero at grading time, so the policy has to be in force while
updates are actually being processed.

---

## Task 3 — AS_PATH filtering on R4

**Device:** R4 (AS 65400)

### Objective

R4 must reject any BGP update whose AS_PATH shows that the route **originated** in AS 65100. Updates
originated in any other Autonomous System must still be accepted and installed — the filter must not
black-hole the whole peering.

### Requirements

1. Define an **AS_PATH access list numbered 1**.
2. The deny entry must use the regular expression **`_65100$`** to identify the target routes. No
   other regular expression is accepted for the deny entry.
3. The list must be applied to the peering with R3 (neighbor `10.3.4.3`) in the **inbound**
   direction, using the **filter-list** mechanism.

### Restrictions

- Do not filter on the prefix value itself; the match must be made on AS_PATH.
- Do not modify any configuration on R1, R2 or R3.

### Think about

What does an AS_PATH access list do with a route that matches none of its entries?

---

## Task 4 — Traffic engineering on R2

**Device:** R2 (AS 65200)

### Objective

Influence R2's local BGP best-path selection so that the prefix `4.4.4.4/32`, as received from R3,
carries an administrative preference of 500. All other prefixes received from R3 must continue to be
accepted with their default attributes.

### Requirements

1. The policy must be implemented in a route-map named exactly **`SET_WEIGHT`**.
2. The matched prefix must be assigned a **Weight of exactly 500**.
3. The route-map must contain a **second permit sequence** so that prefixes not matched by the first
   sequence are still accepted.
4. The route-map must be applied to neighbor `10.2.3.3` in the **inbound** direction.

### Notes

- The method used to match `4.4.4.4/32` is left to your judgement; any supported match mechanism is
  acceptable.
- Weight is a Cisco-proprietary attribute. Consider whether it is advertised to R2's other peer, and
  what that means for the rest of the chain.

---

## Task 5 — Policy activation and session maintenance

**Devices:** all

### Objective

Bring each policy into force without disrupting the underlying peering.

### Requirements

1. After completing each task, refresh the Adj-RIB-In and Adj-RIB-Out so the new policy is applied
   to already-exchanged routes.
2. BGP sessions must **not** be torn down at any point. Neighbor uptime is audited: if a session's
   uptime is shorter than the elapsed time since the policy was applied, this task is scored as
   failed even if the filtering itself is correct.

---

## Grading — commands the assessor will run

| Check | Device | Command | Expected outcome |
|---|---|---|---|
| Architecture | any | `show running-config \| section bgp` | `no bgp default ipv4-unicast` present on **all four** routers |
| Task 1 | R2 | `show ip bgp` | `1.1.1.1/32` **absent** |
| Task 1 | R2 | `show running-config \| section bgp` | `distribute-list 10 in` under the address family |
| Task 2 | R3 | `show ip bgp` | `2.2.2.2/32` **present** |
| Task 2 | R3 | `show ip prefix-list FILTER_R2` | `permit 2.2.2.2/32 le 32`, hit count > 0 |
| Task 3 | R4 | `show ip as-path-access-list 1` | regex `_65100$` |
| Task 3 | R4 | `show ip bgp` | no path ending in 65100 |
| Task 4 | R2 | `show ip bgp 4.4.4.4` | Weight is exactly 500 |
| Task 5 | all | `show ip bgp neighbors` | uptime consistent with a soft reset only |
