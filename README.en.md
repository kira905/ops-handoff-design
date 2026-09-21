# Operating Long-Lived Coding Agents · An Operations Design Record

> **中文版** → [README.md](README.md)

**This repository is the design record of an operations system built for running coding agents on your own machines over weeks and months.** Eight documents answer eight questions: **how multiple workstations hand state off to each other without losing anything**, **what autonomy capabilities a single-machine environment actually needs**, **what a full "cast of roles + house rules" discipline looks like**, **how a wrong performance attribution got overturned by its own re-measurement**, and **how a design document should be reviewed before work starts**, **what a read-only monitoring panel should answer — and what it must never do**, **what has to be true before a capability you have already built is allowed to act on its own**, and **how a large body of documentation gets audited for statements that no longer match the machine**.

They are not "how our system looks". They are **what breaks, how to judge it, and how to roll it back** — every conclusion comes from a real incident or a measured run, and **the claims that measurement later disproved are kept in the text, not quietly deleted** (`05` is that whole process, start to finish).

---

## What problems this solves

This is probably relevant to you if:

- you run coding agents **on your own machine(s) over weeks or months**, not in a sandbox that dies after one task;
- you have hit any of these: **a session corrupted itself** · **a task queue silently stalled** · **a plugin update bricked startup** · **a backup job overwrote the other machine's work** · **half the state vanished when you switched machines**;
- you want any form of "**an agent that watches other agents**", which forces three questions: **what may run unattended, what must ask a human, and how do you prove afterwards that it acted correctly?**

**Ordinary web-service operations advice does not transfer cleanly.** The failure surface here is session files, encrypted single-file state, vendor SDK breaking changes — plus one structural reality: **there may be exactly one user, so you will never have enough sample size to trust your own rules.** These documents treat "we cannot be sure it is accurate" as a **design input** rather than something to paper over.

---

## Contents

| File | Content | Size |
|---|---|---|
| [`01-多机交接与云中继.md`](01-多机交接与云中继.md) | **Multi-machine handoff.** Why "using a backup repo as a sync drive" destroys data; the Depart / Arrive protocol; four-layer architecture; semantic merge; fail-open vs fail-closed tiers; one cloud host serving three roles (fixed entry / active-host registry / handoff relay) | ~700 lines + 67 review issues from four rounds |
| [`02-自治式运维管家.md`](02-自治式运维管家.md) | **Autonomous operations steward** (**a 2026-09-11 snapshot**): a "constitution" (three object types + autonomy spectrum S0–S5), write gatekeeper, restart gatekeeper, proactive alerting, pitfall auto-learning, external-signal sensing, cross-carrier reconciliation — 9 capability domains | ~1100 lines + 15 unverified assumptions |
| [`03-体系全景与家规.md`](03-体系全景与家规.md) | **The current big picture (newest in this repo).** One cast, two faces, two sets of criteria; five-layer architecture; **eight house rules** (bare-run period / shadow view / locked delivery / four-level circuit breaker / rollback + drills / service-side criteria / ledger arbitration / cross-lane contract); a table untangling the **four different "L-number" schemes**; measured hard constraints (crash chain / auth-gate conflict / capacity and locking); roadmap and gap ledger | ~500 lines |
| [`04-设计稿审核规程.md`](04-设计稿审核规程.md) | **How to review a design document before starting work.** A full review = **five rounds** (engineering contract / implementer's view / runtime failure / four cross-cutting tables + five rings / premise verification) + a **post-implementation conformance gate**. Includes a round-selection matrix, three evidence-gathering steps, the five rings with their breakage patterns, nine anti-patterns, a minimal version and a one-page cheat sheet (Chinese) | ~300 lines |
| [`05-一次错误的性能归因.md`](05-一次错误的性能归因.md) | **A wrong performance attribution, and what the re-measurement said** (measured run). Chasing "everything gets sluggish once several sessions run in parallel": the first version blamed the CPU saturation on full cache rewrites, and the re-measurement (9.7 ms per serialization ≈ 0.8 % of one core) overturned it on the spot. What actually tracks CPU is **how many sessions are running**. Includes the honest ledger of "the fix was real, but it did not fix the original problem", four reusable pitfalls and six lessons (Chinese) |
| [`06-只读监控面板的设计纪律.md`](06-只读监控面板的设计纪律.md) | **Design discipline for a read-only monitoring panel** (Chinese). What a panel for long-lived agents should actually answer (not "is the service up" but "how is today going, and what broke silently"); what one page load really costs (of a measured 134 KB response, **56 % was a field the first screen never used**; sharding + in-process TTL cache + single-flight + a timeout budget); and three lines that must not be crossed (**if you cannot probe it, show "not probeable + why"** — never `0`, never a false green; render only, never recompute; a panel must have no side effects). Includes three real "silent failure" lessons, where the ceiling on making such a panel generic lies, and an honest list of what to cut | ~380 lines |
| [`07-放行判据与开关矩阵.md`](07-放行判据与开关矩阵.md) | **Release criteria and the switch matrix** (Chinese). How a personal, long-running automation stack goes from "built" to "allowed to act": how the threshold is computed (observations pooled **across machines**, but release granted **per machine**; with zero false positives, "95 % confident the false-positive rate is below" = `1 − 0.05^(1/n)`, so n = 50 → 5.8 % — only the do-no-harm tier may relax to 25), why **missing data must never be released** (fail-safe), how to read the **action → risk tier → switch → guardrail → release criteria** matrix, the four hard-coded rules, and how to stop the ruler from being changed mid-observation (frozen surface / segmented reset). Includes "alert counts get padded by repeats" (18 alerts turned out to be 12 distinct conditions) | ~400 lines |
| [`08-文档体系口径一致性审核.md`](08-文档体系口径一致性审核.md) | **Documentation consistency audit** (Chinese). How to audit a body of documentation that has been evolving for months: gather live evidence first, cross-cut the documents against each other, and sort every finding into four buckets (**stale / conflicting / nobody-defined**). One real round found **24 inconsistencies**, and most were not design errors but "a sentence was written, the world changed hours later, and nobody wrote back". Includes the criteria for all three problem classes, how to build the four cross-cutting tables, write-back discipline, recurrence prevention, and a reusable one-round output checklist | ~290 lines |
| [`发布说明.md`](发布说明.md) | **Release notes.** Repo topology, licensing, versioning, component index, open-sourcing admission criteria (Chinese only) | — |

---

## How to read these documents

1. **If you only want one thing** — read `03` §4: the eight house rules, each written as *why it exists / how it is judged / what it costs*. The "what it costs" section is deliberate: **a rule whose cost is not written down gets bypassed the first time it is inconvenient.**

2. **If you want to design your own** — start with `02` §4 (object types + autonomy spectrum) and §5 (safety / concurrency / observability), then `03` for how those ideas survived contact with production.

3. **If you want criteria you can copy** — `03` §4 R6 gives six service-side criteria, each with **threshold / authoritative source / how to verify / where the trace is recorded**. The design stance behind that layout: *a criterion is not a criterion until it can be checked.*

4. **Documents `01`–`03` end with two sections that decide how much to trust them**:
   - *Review records* — how many reviewers, how many issues found, **where each landed** (including the explicit statement that one review round does not certify the next);
   - *Assumptions & validation timing* — what is **still unverified**, when it will be tested, and **what the fallback is if it fails**.
   - A design document without these two sections should be read as "the author's current impression".

5. **If you want to see how a wrong conclusion gets corrected** — read `05`. It keeps the wrong attribution, the re-measurement that killed it, and the honest ledger of what the fix actually bought, in one piece.

6. **If you want to review your own design before starting work** — read `04`: five review rounds (engineering contract / implementer's view / runtime / four cross-cutting tables + five rings / premise verification), a round-selection matrix, and a post-implementation conformance gate that catches parameters invented during implementation.

7. **Conventions**: placeholders such as `<workspace root>`, `<agent home>`, `Host A / B / C`, `steward-*` are generic names; model and vendor names are illustrative. All measured numbers are **observations at a point in time**, not design parameters — borrow the method and the order of magnitude, not the thresholds.

8. **`02` and `03` are two different documents, not two drafts of one**: `02` is a **design snapshot** (how the thing was thought through, including assumptions that were still unverified); `03` is the **current big picture** (what it looks like now, and why it is trusted to act). Neither replaces the other.

9. **If you want to build a read-only panel for your own agent** — read `06`. It covers the display discipline (**unreadable means "unknown": never a `0`, never a green**), what one page load costs and how to shard it, why a panel must **render only and never recompute** (one authoritative source per fact), and where the ceiling on "making the panel generic" actually sits. Like `04` and `05`, it does not carry the review-records / assumptions sections: it is a discipline, not a design under review.

10. **If you have built an automation capability and are afraid to switch it on** — read `07`. It is not an opinion about whether automation is good; it is a **path from "built" to "released"**: how the threshold is computed, why samples are pooled across machines but release is granted per machine, why missing data must never be released, how to read the action → risk tier → switch → guardrail → criteria matrix, and why a release day should only ever be "edit the table + leave a trace".

11. **If you want to know whether your own documentation is still true** — read `08`. It covers the three classes of decay (**stale / conflicting / nobody-defined**), how to gather live evidence before reading any document, the four cross-cutting tables, and the write-back discipline. Its most useful line: **what gets built is usually fine — what rots is the manual.**

---

## Published at

| Platform | URL | Note |
|---|---|---|
| **GitHub** (primary) | <https://github.com/kira905/ops-handoff-design> | International entry point |
| **Gitee** (mirror) | <https://gitee.com/kira905/ops-handoff-design> | Directly reachable from mainland China |

Current version: **v1.8** (tag sequence `v1.0` → … → `v1.7` → `v1.8`; `v1.8` = commit `8233ef4`). The `v1.9` addition — documents `07` and `08` — is already committed locally and is **not tagged yet**. The per-version changelog is maintained in the [Chinese README](README.md).

> **On the v1.6 file rename**: the three documents dropped the word "design" from their titles in this version. If you hold a link to an older filename, tag `v1.5` still contains the same content under the old name.

---

## Translation policy

**Deliberately asymmetric — English README + Chinese technical documents, translated on demand.**

| Content | Language | Why |
|---|---|---|
| README (this file), repo description, release notes | **English + Chinese** | So the repo is discoverable and understandable to an international reader before committing to anything |
| Documents (01 / 02 / 03 / 04 / 05 / 06 / 07 / 08) | **Chinese first**; English translation per document, when there is demand | A faithful translation of a thousand-line design document is a second artifact to maintain forever. Translating everything up front means either both copies rot, or the translation becomes the reason not to update the original |
| Component READMEs | **English required** | A component's README is its user interface; it is short, and it is what a stranger reads first |

**How a translation happens**: an English file is added as `XX-<name>.en.md` **next to** the Chinese original (never replacing it), linked from both READMEs, and the Chinese file remains the source of truth. If you need one of the design documents in English, **open an issue naming the document** — demand is itself the scheduling signal.

---

## Repo topology (components will be open-sourced separately)

This repository holds the **methodology only**. Implementations ship as **one repository per component**:

```
ops-handoff-design     [docs]       ← this repo: the authoritative narrative
dsh-<component>        [component]  ← one repo each; its README links back to the
...                                    specific sections here that justify its design
```

Reasons: the licensing is deliberately split (**docs CC BY-NC-SA 4.0 so they cannot be commercially repackaged; component code under AGPL-3.0 with commercial licensing available**), components version and release independently, and third-party issues have an unambiguous home.

- **Component index** (what exists, its status, which sections justify it) → [`发布说明.md` §4.5](发布说明.md)
- **What will *not* be open-sourced, and why** → [`发布说明.md` §4.6](发布说明.md) (four admission criteria + a negative list)

---

## License

- **Documents**: [CC BY-NC-SA 4.0](LICENSE) — attribution + non-commercial + share-alike. You may read, quote, translate, adapt and use them for learning or in non-commercial projects; you may not repackage them commercially, and derivative works must keep the same license. **This repository currently contains documentation only**; `LICENSE` covers all of it.
- **Component repositories**: each carries its own license, **independent of this repo** (current component repositories are licensed under **AGPL-3.0**, with commercial licensing available on request).

Third-party standards, papers and official documentation referenced inside remain the property of their respective owners; they are cited only to identify the basis of a design decision.

---

*Derived from internal production drafts. The production versions remain the single source of truth: these are one-way derived copies, and nothing here flows back into them.*
