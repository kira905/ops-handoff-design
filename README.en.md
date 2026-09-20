# Operating Long-Lived Coding Agents · An Operations Design Record

> **中文版** → [README.md](README.md)

**This repository is the design record of an operations system built for running coding agents on your own machines over weeks and months.** Three documents answer three questions: **how multiple workstations hand state off to each other without losing anything**, **what autonomy capabilities a single-machine environment actually needs**, and **what a full "cast of roles + house rules" discipline looks like**.

They are not "how our system looks". They are **what breaks, how to judge it, and how to roll it back** — every conclusion comes from a real incident or a measured run, and **the claims that measurement later disproved are kept in the text, not quietly deleted**.

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
| [`发布说明.md`](发布说明.md) | **Release notes.** Repo topology, licensing, versioning, component index, open-sourcing admission criteria (Chinese only) | — |

---

## How to read these documents

1. **If you only want one thing** — read `03` §4: the eight house rules, each written as *why it exists / how it is judged / what it costs*. The "what it costs" section is deliberate: **a rule whose cost is not written down gets bypassed the first time it is inconvenient.**

2. **If you want to design your own** — start with `02` §4 (object types + autonomy spectrum) and §5 (safety / concurrency / observability), then `03` for how those ideas survived contact with production.

3. **If you want criteria you can copy** — `03` §4 R6 gives six service-side criteria, each with **threshold / authoritative source / how to verify / where the trace is recorded**. The design stance behind that layout: *a criterion is not a criterion until it can be checked.*

4. **Every document ends with two sections that decide how much to trust it**:
   - *Review records* — how many reviewers, how many issues found, **where each landed** (including the explicit statement that one review round does not certify the next);
   - *Assumptions & validation timing* — what is **still unverified**, when it will be tested, and **what the fallback is if it fails**.
   - A design document without these two sections should be read as "the author's current impression".

5. **Conventions**: placeholders such as `<workspace root>`, `<agent home>`, `Host A / B / C`, `steward-*` are generic names; model and vendor names are illustrative. All measured numbers are **observations at a point in time**, not design parameters — borrow the method and the order of magnitude, not the thresholds.

6. **`02` and `03` are two different documents, not two drafts of one**: `02` is a **design snapshot** (how the thing was thought through, including assumptions that were still unverified); `03` is the **current big picture** (what it looks like now, and why it is trusted to act). Neither replaces the other.

---

## Published at

| Platform | URL | Note |
|---|---|---|
| **GitHub** (primary) | <https://github.com/kira905/ops-handoff-design> | International entry point |
| **Gitee** (mirror) | <https://gitee.com/kira905/ops-handoff-design> | Directly reachable from mainland China |

Current version: **v1.6** (tag sequence `v1.0` → … → `v1.5` → `v1.6`). The per-version changelog is maintained in the [Chinese README](README.md).

> **On the v1.6 file rename**: the three documents dropped the word "design" from their titles in this version. If you hold a link to an older filename, tag `v1.5` still contains the same content under the old name.

---

## Translation policy

**Deliberately asymmetric — English README + Chinese technical documents, translated on demand.**

| Content | Language | Why |
|---|---|---|
| README (this file), repo description, release notes | **English + Chinese** | So the repo is discoverable and understandable to an international reader before committing to anything |
| Design documents (01 / 02 / 03) | **Chinese first**; English translation per document, when there is demand | A faithful translation of a thousand-line design document is a second artifact to maintain forever. Translating everything up front means either both copies rot, or the translation becomes the reason not to update the original |
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
