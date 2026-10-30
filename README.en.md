# Operating Long-Lived Coding Agents · An Operations Design Record

> **Give your coding agent a self-maintaining butler — what breaks, how to judge it, and how to roll it back**

> **中文版** → [README.md](README.md)

**This repository is the design record of an operations system built for running coding agents on your own machines over weeks and months.** Fifteen documents answer fifteen questions: **how multiple workstations hand state off to each other without losing anything**, **what autonomy capabilities a single-machine environment actually needs**, **what a full "cast of roles + house rules" discipline looks like**, **how a wrong performance attribution got overturned by its own re-measurement**, and **how a design document should be reviewed before work starts**, **what a read-only monitoring panel should answer — and what it must never do**, **what has to be true before a capability you have already built is allowed to act on its own**, and **how a large body of documentation gets audited for statements that no longer match the machine**, **how to tell a task card that merely claims to be finished from one that is — and an interrupted session from a closed one**, **how to upgrade your upstream dependencies (host program and plugin packages) without breaking everything**, **which lane a piece of work belongs to and which changes may be wired up unaided versus which must wait for a human**, **what granularity to state requirements in and what granularity to accept against so a version can actually be closed**, **how a self-monitoring decision module should judge "am I alive" and "should I page anyone"**, and **what discipline batch file edits need so they can be rolled back**.

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
| [`03-体系全景与家规.md`](03-体系全景与家规.md) | **The current big picture (newest in this repo).** One cast, two faces, two sets of criteria; five-layer architecture; **eleven house rules R1–R11** (bare-run period / shadow view / locked delivery / four-level circuit breaker / rollback + drills / service-side criteria / ledger arbitration / cross-lane contract / autonomy tiers / three-axis effort estimation / custom event members banned in patched plugins); a table untangling the **four different "L-number" schemes**; measured hard constraints (crash chain / auth-gate conflict / capacity and locking); roadmap and gap ledger | ~500 lines |
| [`04-设计稿审核规程.md`](04-设计稿审核规程.md) | **How to review a design document before starting work.** A full review = **five rounds** (engineering contract / implementer's view / runtime failure / four cross-cutting tables + five rings / premise verification) + a **post-implementation conformance gate**. Includes a round-selection matrix, three evidence-gathering steps, the five rings with their breakage patterns, nine anti-patterns, a minimal version and a one-page cheat sheet (Chinese) | ~300 lines |
| [`05-一次错误的性能归因.md`](05-一次错误的性能归因.md) | **A wrong performance attribution, and what the re-measurement said** (measured run). Chasing "everything gets sluggish once several sessions run in parallel": the first version blamed the CPU saturation on full cache rewrites, and the re-measurement (9.7 ms per serialization ≈ 0.8 % of one core) overturned it on the spot. What actually tracks CPU is **how many sessions are running**. Includes the honest ledger of "the fix was real, but it did not fix the original problem", four reusable pitfalls and six lessons (Chinese) |
| [`06-只读监控面板的设计纪律.md`](06-只读监控面板的设计纪律.md) | **Design discipline for a read-only monitoring panel** (Chinese). What a panel for long-lived agents should actually answer (not "is the service up" but "how is today going, and what broke silently"); what one page load really costs (of a measured 134 KB response, **56 % was a field the first screen never used**; sharding + in-process TTL cache + single-flight + a timeout budget); and three lines that must not be crossed (**if you cannot probe it, show "not probeable + why"** — never `0`, never a false green; render only, never recompute; a panel must have no side effects). Includes three real "silent failure" lessons, where the ceiling on making such a panel generic lies, and an honest list of what to cut | ~380 lines |
| [`07-放行判据与开关矩阵.md`](07-放行判据与开关矩阵.md) | **Release criteria and the switch matrix** (Chinese). How a personal, long-running automation stack goes from "built" to "allowed to act": how the threshold is computed (observations pooled **across machines**, but release granted **per machine**; with zero false positives, "95 % confident the false-positive rate is below" = `1 − 0.05^(1/n)`, so n = 50 → 5.8 % — only the do-no-harm tier may relax to 25), why **missing data must never be released** (fail-safe), how to read the **action → risk tier → switch → guardrail → release criteria** matrix, the four hard-coded rules, and how to stop the ruler from being changed mid-observation (frozen surface / segmented reset). Includes "alert counts get padded by repeats" (18 alerts turned out to be 12 distinct conditions) | ~400 lines |
| [`08-文档体系口径一致性审核.md`](08-文档体系口径一致性审核.md) | **Documentation consistency audit** (Chinese). How to audit a body of documentation that has been evolving for months: gather live evidence first, cross-cut the documents against each other, and sort every finding into four buckets (**stale / conflicting / nobody-defined**). One real round found **24 inconsistencies**, and most were not design errors but "a sentence was written, the world changed hours later, and nobody wrote back". Includes the criteria for all three problem classes, how to build the four cross-cutting tables, write-back discipline, recurrence prevention, and a reusable one-round output checklist | ~290 lines |
| [`09-长会话的自动交接与接力.md`](09-长会话的自动交接与接力.md) | **Automatic handoff and relay for long-running sessions** (Chinese). What to do when a long-lived agent session hits its context ceiling: why "just compact it" fails (summaries recur, layer over layer, and operational detail is what gets lost first), which of the three carriers is authoritative (handoff document / ledger / title axis — **one authoritative source per fact**), the **twelve gates** and one reversal in the design stance ("guard against anomalies, not against frequency"), why permissions need **two-segment assertions** (checking config alone misses "config says A while what actually got issued is a wider B"), and the single filter that stands **before anything leaves the machine**. Includes a real production handoff (**2.7 s end to end**), a **59-millisecond title-axis loss** (the writer was changed, the judge was not), and three measured hard facts | ~340 lines |
| [`10-任务卡假完成与被打断会话的识别.md`](10-任务卡假完成与被打断会话的识别.md) | **Telling a "finished" task card from a lie — and an interrupted session from a closed one** (Chinese). In a task board where a card *is* a real session: why an empty execution record is **not** evidence of a fake completion (three legitimate causes), a three-pronged cross-evidence method (a snapshot taken before archiving / session-side cross-checks / archive timestamps as bookkeeping), and why "cannot determine" must be recorded honestly instead of being forced into a verdict. The second half gives the **tail-first, newest-to-oldest fold** over a session log, the trap where a backfilled closing marker hides the real tail, and what it takes to change a criterion that has been copied into several consumers | ~300 lines |
| [`11-上游依赖的受控升级.md`](11-上游依赖的受控升级.md) | **Upgrading your upstream dependencies under control** (Chinese). Why upgrading is the most dangerous routine operation in this kind of system (one small upstream change can make a downstream plugin **fail at startup** or **silently stop detecting what it used to detect**), tiered authorization and its one-way property (closing a switch is easier than opening one), why the **three gates plus an exclusive lock** must run in a fixed order with fail-closed defaults, symbol-level breakage auditing + a single-source event vocabulary + silent-failure detection, and the lesson that **"no error was reported" is not the same as "nothing broke"** | ~320 lines |
| [`12-三线分工与接线授权.md`](12-三线分工与接线授权.md) | **Lane separation and wiring authorization** (Chinese). When one person runs several long-lived lanes at once, the expensive mistake is not knowing whose work is in your hands: the test is **who benefits / who gets hit**, not who is free; "wiring" changes (connect a channel / a producer / an ordering / a switch) may be **connected and used unaided, fixed while in use** — provided you can **recover the data yourself** and you say which of the four ways applies; the only hard line is that **anything irreversible requires the owner in the room**; and why "while I was here anyway" turns into silent cross-lane contamination | ~200 lines |
| [`13-需求粒度与验收两轨制.md`](13-需求粒度与验收两轨制.md) | **Requirement granularity and the two-track acceptance model** (Chinese). Why "split every requirement into hundreds of implementation items and track coverage" makes a release **mathematically impossible to close** (fix a batch, discover a batch — the denominator only grows), the 10–20 item ceiling on the requirement layer, the **two tracks** (the owner signs "did what was asked land", the implementer signs engineering quality), eight readiness criteria and ten definition-of-done items (with the checkbox block format and the rules for dropping items), closing a version = **every unfinished item has an owner** (five buckets + a re-open channel), the tech-debt ledger, and "judgement rows may only shrink" | ~320 lines |
| [`14-判定与通知的四条纪律.md`](14-判定与通知的四条纪律.md) | **Four rules for decision and notification modules** (Chinese). A system that watches itself is most dangerous when it **tells you everything is fine** — **a stale heartbeat does not mean dead** (only one of six causes is actual death, and the heartbeat cannot tell them apart), the only two criteria that may declare death (pid gone / **pid reused**), and "when evidence is unavailable, fall toward the branch with the smaller worst case". Also: observe-only mode must not consume the dedupe quota (otherwise **real alerts are silently crowded out**), a single-run override must not be written into persistent config, and machine-readable output must not share a channel with logs. Includes "prove it red before you fix it" and a real two-process drill | ~280 lines |
| [`15-改动的安全纪律.md`](15-改动的安全纪律.md) | **Safety discipline for changes** (Chinese). Dozens of "batch edit + leave a rollback point + verify afterwards" runs per day are where irreversible accidents actually happen: the dry-run switch must be **inside the function** (judging it only at the CLI layer leaves a back door for every internal caller), **copy a byte-exact snapshot before you edit** (adding it afterwards gives you the *post*-edit content — that rollback point is fake), apply edits descending and roll back in reverse, **refuse unless a block anchor matches exactly once**, refusals come in three states and must never exit successfully, write corrections away from auto-regenerated files, and **identify manifest field names before you compare** (empty equals empty prints a false all-green). Includes the six self-checks for a rollback tool | ~330 lines |
| [`复刻手册/`](复刻手册/00-从零到跑起来.md) | **Assembly manual** (Chinese). How someone who has read none of these documents gets "task board + cross-session memory + session handoff tooling + a read-only panel" running from a blank machine: per-package **install / configure / verify** triplets, the smallest working combination (25 s, 2.5 MB measured), a state-file inventory (what to back up, what is rebuildable, what must never leave the machine), and symptom-indexed troubleshooting. Every command was actually run in a fresh isolated `DSH_HOME`, with its output pasted in | — |
| [`发布说明.md`](发布说明.md) | **Release notes.** Repo topology, licensing, versioning, component index, open-sourcing admission criteria (Chinese only) | — |

---

## How to read these documents

1. **If you only want one thing** — read `03` §4: the eleven house rules, each written as *why it exists / how it is judged / what it costs*. The "what it costs" section is deliberate: **a rule whose cost is not written down gets bypassed the first time it is inconvenient.**

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

12. **If you want long sessions to hand themselves over** — read `09`. It is not about whether to automate, but about **which gates a single handoff must pass and which step fails silently**: the three carriers and which one is authoritative, why a title axis deserves to be engineered like a real interface (with a complete 59-millisecond post-mortem), why permission checks must run **twice in two different ways**, and the stop line — **no explicit "next step" means no new session gets opened**. Its "what this does not do" and "what was never measured" sections are worth reading first.

13. **If you cannot tell whether a task card is actually finished, or whether a session was interrupted** — read `10`. It dismantles a popular misreading first: **an empty execution record is not evidence that no work happened** (three legitimate causes), then gives three cross-evidence checks (a snapshot taken before archiving / session-side cross-checks / archive timestamps as bookkeeping) and the **tail-first, newest-to-oldest fold** over a session log. Its most useful line: **when you cannot tell, write "cannot determine" — do not force a verdict.**

14. **Before upgrading the host program or a plugin package** — read `11`. It lays out the chain "**tiered authorization → three gates → execute while holding the lock → symbol-level breakage audit**", and names the failure mode people miss most: **"no error was reported" is not the same as "nothing broke"** — a criterion can silently stop matching, and nothing will alert you. Start with its conclusions and its "what this deliberately does not do".

15. **If you run several lanes at once, or you are unsure who owns a piece of work** — read `12`. Its test is two questions: **who gets better, and who gets hit if it breaks.** Both on your own lane means decide yourself; either on someone else's means ask first. It also explains one counter-intuitive relaxation: **"wiring" changes (connecting a capability that already exists) may be done unaided and fixed while in use**, because the upside is immediate and the risk is bounded (you can back it out) — whereas making every wire wait for a nod ends in a **decision backlog of a few hundred items, i.e. nothing is actually approved**.

16. **If closing a version never works** — read `13`. It first proves the illness is **arithmetic**: using implementation-level coverage as the measure of completion means one batch of fixes discovers another batch, so the denominator only grows and the number can never reach 100 %. Then the remedy: a 10–20 item ceiling on the requirement layer, criteria written at the layer a human reads, two acceptance tracks (the owner only signs "did it land"), and closing a version = **stop adding work + attach every unfinished item to somewhere**, not "everything is done".

17. **If you are writing a scheduler, a patrol or an alerting module** — read `14`, especially §2. It records a full failure chain: **a stale heartbeat judged as dead → self-healing deletes the lock → two instances run at once → work is done twice** — and **no single line of that chain is wrong**; what is wrong is treating a weak signal (heartbeat) as a survival criterion. It also gives the direction rule: **when evidence is unavailable, pick the branch whose worst case hurts less** (not a mechanical fail-closed).

18. **If you are about to write a script that batch-edits a pile of files** — read `15`. Of the seven rules, two matter most: the **dry-run switch must live inside the function** (judging it at the CLI layer only opens a silent back door for every internal caller), and **copy a byte-exact snapshot before you edit** — adding it afterwards gives you the post-edit content, so the "rollback point" is fake.

---

## Published at

| Platform | URL | Note |
|---|---|---|
| **GitHub** (primary) | <https://github.com/kira905/ops-handoff-design> | International entry point |
| **Gitee** (mirror) | <https://gitee.com/kira905/ops-handoff-design> | Directly reachable from mainland China |

Current version: **v1.18** (tag sequence `v1.0` → … → `v1.17` → `v1.18`; `v1.18` was **published on both Gitee and GitHub on 2026-10-30**). **Four new documents (`12` / `13` / `14` / `15`); documents 01–11 and `复刻手册/` are unchanged.** They bring the house rules and criteria settled around the internal V3.0.0 seal (2026-10-29) into the public set: `12` lane separation and wiring authorization · `13` requirement granularity and the two-track acceptance model · `14` four rules for decision and notification modules · `15` safety discipline for changes. The same release refreshes two component-index versions to measured values (`dsh-memory-toolkit` and `dsh-board-toolkit`, both `v0.1.0 → v0.1.1`), reconciled three ways across both remotes' tags and the registry. The previous release, `v1.17` (2026-10-21), was index-only (two missing repositories registered, two stale versions refreshed, one license field aligned — **sixteen → eighteen component repositories**); `v1.16` (2026-10-19) added the `复刻手册/` assembly manual; `v1.15` narrowed the scope to the "butler suite" (three unrelated repositories removed, nineteen → sixteen); `v1.14` refreshed the component index to nineteen rows; `v1.13` added repository links to every row; `v1.12` brought the internal **V2.9 final-design** baseline into the public documents (house rules **R1–R11**); `v1.11` added documents `10` and `11`; `v1.10` added `09`. The per-version changelog is maintained in the [Chinese README](README.md).

> **On the v1.6 file rename**: the three documents dropped the word "design" from their titles in this version. If you hold a link to an older filename, tag `v1.5` still contains the same content under the old name.

---

## Translation policy

**Deliberately asymmetric — English README + Chinese technical documents, translated on demand.**

| Content | Language | Why |
|---|---|---|
| README (this file), repo description, release notes | **English + Chinese** | So the repo is discoverable and understandable to an international reader before committing to anything |
| Documents (01 – 15) | **Chinese first**; English translation per document, when there is demand | A faithful translation of a thousand-line design document is a second artifact to maintain forever. Translating everything up front means either both copies rot, or the translation becomes the reason not to update the original |
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

- **Already published components** — **eighteen** repositories so far (session archiving · live session titles · a read-only ecosystem health panel · diagnostic tooling · a session-lifecycle toolkit · a self-maintained task board · an incident toolkit · a memory toolkit · board and session operations · operations and upgrades · quality and rules · diagnostics and reconciliation · anchored correction notes · a safe-mode repair layer · a client adapter layer · networking and cross-host helpers · a distilled ops skeleton · decoupling & dependency-closure diagnostics). **The index — what exists, its version, its license, its status and which sections justify its design — is maintained in [`发布说明.md` §4.5](发布说明.md), with Gitee and GitHub links for every component.** Components are admitted one at a time, and repository names are not announced before they are published.
- **What will *not* be open-sourced, and why** → [`发布说明.md` §4.6](发布说明.md) (four admission criteria + a negative list)

---

## License

- **Documents**: [CC BY-NC-SA 4.0](LICENSE) — attribution + non-commercial + share-alike. You may read, quote, translate, adapt and use them for learning or in non-commercial projects; you may not repackage them commercially, and derivative works must keep the same license. **This repository currently contains documentation only**; `LICENSE` covers all of it.
- **Component repositories**: each carries its own license, **independent of this repo** (current component repositories are licensed under **AGPL-3.0**, with commercial licensing available on request).

Third-party standards, papers and official documentation referenced inside remain the property of their respective owners; they are cited only to identify the basis of a design decision.

---

*Derived from internal production drafts. The production versions remain the single source of truth: these are one-way derived copies, and nothing here flows back into them.*
