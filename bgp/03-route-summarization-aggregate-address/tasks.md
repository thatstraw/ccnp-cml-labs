# BGP Route Summarization with aggregate-address

**Self-study lab — candidate instructions**

R1 originates three stub /24s (`172.16.1.0/24`, `172.16.2.0/24`, `172.16.3.0/24`) plus its own
loopback into AS 65100. R3, two AS hops away, currently sees all three stubs as individual routes
with AS path `65200 65100`. Use dynamic route summarization to reduce that to a single aggregate.

---

## Task 1 — Summarize R1's stub networks

**Device:** R1 (AS 65100)

### Objective

Replace the three individual `/24` advertisements for `172.16.1.0/24`, `172.16.2.0/24` and
`172.16.3.0/24` with a single summary, without withdrawing R1's own loopback or the `10.12.1.0/24`
transit link.

### Requirements

1. Use the BGP address-family command `aggregate-address <network> <subnet-mask> [summary-only]
   [as-set]` on R1.
2. The three /24s summarize cleanly to a single `/22`: work out the correct network and mask.
3. Use `summary-only` so the three more-specific /24s are suppressed from advertisement — R2 and R3
   should see only the aggregate, not the /24s underneath it.

### Restrictions

- Do not use `network` statements to solve this — the aggregate must be dynamically generated from
  the existing `redistribute connected` routes already in R1's BGP table.
- Do not filter routes with a distribute-list, prefix-list or route-map — `aggregate-address` is the
  only mechanism this task is testing.

### Verify

```
R1# show bgp ipv4 unicast
R2# show ip bgp
R3# show ip bgp
```

R3 should now see one summary route in place of the three /24s. Compare its AS path and origin code
against what the three individual routes carried before — and consider what changes if you add
`as-set` to the command.

### Think about

- What happens to R1's own `192.168.1.1/32` and the `10.12.1.0/24` link network — are they part of
  the aggregate, or do they still advertise individually? Why?
- `summary-only` suppresses the more-specifics from being *advertised*. Are they still in R1's own
  BGP table and RIB?
- Try the same command without `summary-only` and compare what R3 sees. Then try adding `as-set` and
  compare the AS path on the aggregate itself.
