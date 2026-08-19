---
title: "Network Change"
description: "Run a network change window without losing reach to the device you are configuring."
icon: "settings_ethernet"
weight: 710
toc: true
tags: ["networking", "change-management", "runbook"]
---

Network changes are unusual among production changes because the thing you are modifying is the same thing carrying your management session. A bad line in an access list, a VLAN typo, or a routing policy that withdraws the wrong prefix can remove your own path to the device a fraction of a second after you press enter. This checklist is a change-window runbook: capture a baseline, understand the blast radius, arrange a way back in, make the change, verify it, and know exactly how to undo it.

{{< alert context="info" text="**Who runs this:** the engineer making the change, with a second network engineer on the call as verifier. **When:** the pre-change sections at least 24 hours before the window; the rest live, in order, during the window." />}}

## 1. Change definition and approval

- [ ] **The change has a written statement of intent** — one paragraph describing what will be different afterwards, in terms of traffic behaviour rather than commands.
- [ ] **The exact configuration to be applied is pasted into the change record** — not a description of it, the literal lines, so the verifier reviews what will actually be typed.
- [ ] **The change has been peer reviewed by someone who did not write it** — network syntax errors are almost never caught by the author, who reads what they meant rather than what they wrote.
- [ ] **The change window has an explicit end time and a go/no-go decision point** — decide in advance the clock time at which you stop troubleshooting and roll back regardless of how close you feel to a fix.
- [ ] **Approval covers the maintenance window, not just the change** — including who is authorised to declare the change failed and call the rollback.
- [ ] **Dependent teams have been notified with the expected impact window** — application, database, and storage owners plan their own freezes around it.
- [ ] **The change is not stacked with unrelated changes in the same window** — bundling three changes means a failure gives you three suspects and no clean rollback.

## 2. Baseline capture

- [ ] **The full running configuration is exported and stored off the device** — to a git repository or configuration management system, not just the device flash, which is unreachable if the device is unreachable.
- [ ] **A configuration diff against the last known-good version is clean** — any drift you find now is undocumented change that will confuse the post-change comparison.
- [ ] **Routing and neighbour state is captured before the change** — BGP peer counts and prefix counts per peer, OSPF adjacencies, and the default route source.
- [ ] **Interface counters, error counters, and CDP/LLDP neighbours are recorded** — so a post-change increase in CRC errors or a missing neighbour is visible as a change rather than a mystery.
- [ ] **ARP, MAC address table, and NAT translation counts are captured** — a large post-change swing in table size is a strong signal that traffic is taking an unintended path.
- [ ] **The baseline is saved as a text file with a timestamp, not left in scrollback** — terminal buffers get cleared, and the person diffing may not be the person who captured.

```bash
# capture into timestamped files before touching anything
for cmd in "show running-config" "show ip bgp summary" "show ip route summary" \
           "show interfaces status" "show lldp neighbors"; do
  echo "=== $cmd ===" >> baseline-$(date +%Y%m%dT%H%M).txt
done
```

## 3. Blast-radius assessment

- [ ] **Every prefix, VLAN, or service carried by the affected device or link is enumerated** — the answer to what breaks must be a list, not a shrug.
- [ ] **Redundancy has been verified as actually working, not merely configured** — check that the standby path currently carries traffic when tested, because a dormant redundant path is often found to have been broken for months.
- [ ] **Single-homed dependencies are identified** — management networks, out-of-band jump hosts, licence servers, and NTP sources are the ones people forget.
- [ ] **The change is assessed against maintenance-affecting traffic** — backups, batch replication, and scheduled reporting jobs that run during the quiet window you chose precisely because it is quiet.
- [ ] **Impact on stateful devices is understood** — firewall session tables, load balancer connection state, and NAT translations do not survive a path change, so existing long-lived flows will drop even if routing converges.
- [ ] **The failure mode is predicted in writing** — state whether a mistake causes a hard outage, a partial blackhole, or an asymmetric path, because each needs a different verification.

## 4. Out-of-band access

- [ ] **Console or out-of-band access is tested and open before the first command** — not tested last week, tested now, in a second window, with a prompt visible.
- [ ] **The out-of-band path does not traverse the device being changed** — a serial console server reachable only through the production network is not out of band.
- [ ] **Credentials for the out-of-band path are on hand and verified** — including a local fallback account, because the change may cut reachability to the RADIUS or TACACS+ server.
- [ ] **A commit-confirm or scheduled rollback timer is armed where the platform supports it** — the device reverts to the previous configuration automatically unless you confirm within the timer, which converts a lockout from an outage into a short blip.
- [ ] **On platforms without commit-confirm, a reload timer is set before applying** — schedule a reload to the saved configuration, then cancel it once verification passes.
- [ ] **The configuration is not saved to startup until verification is complete** — an unsaved change is undone by a power cycle, which is your last-resort rollback.

```text
! IOS-style safety net: device reboots to saved config in 10 minutes
reload in 10
! ...apply change, verify reachability...
reload cancel
```

{{< alert context="danger" text="**Blocking:** do not apply a change to a remote device without a tested out-of-band path or an armed automatic rollback timer. Locking yourself out of a reachable-only-in-band device turns a five-minute change into a site visit." />}}

## 5. Applying the change

- [ ] **Changes are applied in an order that keeps the management path alive longest** — add the new permit before removing the old one, bring the new adjacency up before tearing the old one down.
- [ ] **Access list edits are made by building a new named list and swapping it atomically** — editing an applied list line by line briefly leaves the device in a state neither old nor new, which is when the deny-any at the end bites you.
- [ ] **Configuration is pasted from the reviewed change record, not retyped** — retyping introduces exactly the single-character errors that peer review was meant to eliminate.
- [ ] **Terminal logging is enabled so the entire session is captured** — you will want the exact transcript during the post-incident review, and you will not remember it accurately.
- [ ] **Each discrete step is verified before the next is applied** — batching ten steps and verifying once means the failure is attributed to the wrong step.
- [ ] **The verifier confirms reachability from an independent location after each step** — the engineer's own session can survive a change that breaks everyone else's path.
- [ ] **Nothing is saved to startup configuration until the verification section passes in full.**

## 6. Verification

- [ ] **Control-plane state is diffed against the baseline** — same BGP peer count, same prefix counts within an expected delta, same adjacencies, no new flapping.
- [ ] **Data-plane reachability is tested from a real client, not from the device** — a router can ping a destination through a management VRF that user traffic never uses.
- [ ] **Both directions of the path are tested** — asymmetric routing frequently survives a one-way ping test and then breaks any stateful firewall in the path.
- [ ] **Path MTU is validated with a large do-not-fragment packet** — new tunnels and encapsulations silently shave bytes, and the symptom is that small requests work while large responses hang.
- [ ] **Application-level health is confirmed, not just ICMP** — check the actual service, because ICMP is often permitted where TCP is not.
- [ ] **Error counters are checked for increases since the baseline** — CRC errors, input drops, and output discards appearing after the change point to a duplex, optics, or buffer problem introduced by it.
- [ ] **Monitoring and alerting show green from the monitoring system's own perspective** — confirm the monitoring system can still reach the device, since a change that breaks polling makes everything look quiet.
- [ ] **A defined soak period is observed before declaring success** — at least fifteen minutes of steady state, long enough for routing to settle and for periodic jobs to exercise the path.

## 7. Rollback

- [ ] **The rollback procedure is written out as commands before the change begins** — improvising a rollback under pressure is how a small outage becomes a long one.
- [ ] **Rollback has been tested in a lab or on a non-production peer where the change is non-trivial** — especially for anything involving routing policy or spanning tree.
- [ ] **The rollback decision criteria are objective** — a named metric and a threshold, so the decision does not depend on the changer's optimism at 2am.
- [ ] **Restoring the previous configuration is known to be sufficient** — if the change caused neighbours to reconverge, clear routing adjacencies and stateful sessions explicitly rather than assuming they recover.
- [ ] **The rollback is verified with the same verification steps as the change** — a partial rollback that restores reachability but leaves policy half-applied is worse than either state.
- [ ] **A hard time limit is set on troubleshooting before rollback is mandatory** — write the clock time on the bridge at the start of the window.

## 8. Closure and documentation

- [ ] **The post-change configuration is saved and exported to the configuration repository** — the source of truth must match the device before the next change starts from it.
- [ ] **Network diagrams, IPAM entries, and the CMDB are updated in the same window** — documentation updated later is documentation not updated.
- [ ] **Monitoring is updated for any new interfaces, peers, or thresholds** — a link added today with no alerting is a link that fails silently in six months.
- [ ] **The change record is closed with the actual outcome, including anything unexpected** — near misses are the most valuable content in a change record.
- [ ] **Firewall and access list changes are recorded with a review date** — temporary permits become permanent unless something forces them to be revisited.
- [ ] **Lessons that would change the runbook are fed back into it immediately** — while the detail is fresh, not at the retrospective.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Change definition and approval | | | Pass / Pass with actions / Fail |
| Baseline capture | | | Pass / Pass with actions / Fail |
| Blast-radius assessment | | | Pass / Pass with actions / Fail |
| Out-of-band access | | | Pass / Pass with actions / Fail |
| Applying the change | | | Pass / Pass with actions / Fail |
| Verification | | | Pass / Pass with actions / Fail |
| Rollback | | | Pass / Pass with actions / Fail |
| Closure and documentation | | | Pass / Pass with actions / Fail |

The out-of-band access section must be a full Pass before the window opens; any other section may close with a dated follow-up ticket and a named owner.

## Related checklists

- [Change Management](/docs/itsm/change-management/)
- [Load Balancer Configuration](/docs/networking/load-balancer/)
- [DNS Migration](/docs/networking/dns-migration/)
- [Incident Management](/docs/operations/incident-management/)
- [Disaster Recovery Drill](/docs/itsm/disaster-recovery-drill/)

## References

- [NIST SP 800-128 — Security-Focused Configuration Management of Information Systems](https://csrc.nist.gov/pubs/sp/800/128/upd1/final)
- [RFC 3871 — Operational Security Requirements for Large ISP IP Network Infrastructure](https://www.rfc-editor.org/rfc/rfc3871)
- [RFC 6241 — Network Configuration Protocol (NETCONF)](https://www.rfc-editor.org/rfc/rfc6241)
- [Google SRE Book — Reliable Product Launches at Scale](https://sre.google/sre-book/reliable-product-launches/)
