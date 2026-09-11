# Multi-Machine Handoff & Cloud Relay · An Autonomous Operations Steward

> **Two production-derived design documents** on running a coding-agent workstation (or a pair of them) as a long-lived system: how machines hand state off to each other without losing anything, and how a "steward" layer watches, audits, restarts, learns and alerts — with an autonomy level you can actually justify.
>
> **This is not a tutorial or a best-practice list.** It is the working design record of a system that has been running in production, including the three real incidents that forced a rewrite, the four rounds of review that found 67 issues in it, and the assumptions that are **still unverified**.

---

## What's inside

| File | Content |
|---|---|
| [`01-多机交接与云中继设计.md`](01-多机交接与云中继设计.md) | **Multi-machine handoff & cloud relay.** Why "backup repo used as a sync drive" fails; the Arrive/Depart protocol; four-layer architecture; semantic merge; fail-open vs fail-closed tiers; one cloud host serving three roles (mobile entry / active-host registry / handoff relay) |
| [`02-自治式运维管家设计.md`](02-自治式运维管家设计.md) | **Autonomous operations steward.** A "constitution" (three object types + autonomy spectrum S0–S5), write gatekeeper, restart gatekeeper, proactive alerting, pitfall auto-learning, external-signal sensing, cross-carrier reconciliation — 9 capability domains in total |
| [`发布说明.md`](发布说明.md) | **Release notes.** Repo topology (1 docs repo + N component repos), licensing, versioning, component index, open-sourcing admission criteria |

> 📌 **Documents are currently in Chinese.** The repository is organized so that each technical document can be translated independently — see *Translation policy* below.

---

## Who this is for

You will find this useful if you:

- run coding agents **on your own machine over weeks or months**, not in a sandbox for one task;
- have had a session corrupt itself, a task queue silently stall, a plugin update brick startup, or a backup job **overwrite the other machine's work**;
- are building any "agent that watches other agents" and need to decide **what may run unattended, what must ask a human, and how to prove it after the fact**.

Ordinary web-service operations advice does not transfer cleanly here: the failure modes are session files, encrypted single-file state, vendor SDK breaking changes, and the fact that **there may be exactly one user, so there is never enough sample size to trust your own rules**. The documents treat "we cannot be sure it's accurate" as a design input rather than something to paper over.

---

## How to read these documents

1. **The three most counter-intuitive conclusions** (if you read nothing else):
   - *A backup repo is not a sync drive.* Encrypted single-file state cannot be merged by Git, so **"whoever pushes last wins"** silently destroys the other side's data. Fix the semantics (handoff, single writer), not the tooling.
   - *Your gatekeeper covers less than you think.* Host filesystem event hooks only see the editor/tool write path. Writes made by scripts or the command line bypass them entirely — say so in your own docs instead of claiming "tamper-proof".
   - *Anomaly detection without calibration is noise.* With one user, "false-positive rate" is the **first KPI**; a rule that feels right and is never measured will get muted within two weeks.

2. **Every document ends with two sections that decide how much to trust it**:
   - *Review records* — how many reviewers, how many issues found, **where each landed** (and the explicit statement that one review round does not certify the next);
   - *Assumptions & validation timing* — what is **still unverified**, when it will be tested, and **what the fallback is if it fails**.
   - A design document without these two sections should be read as "the author's current impression".

3. **Conventions**: placeholders such as `<workspace root>`, `<agent home>`, `Host A / Host B`, `steward-*` are generic names; model and vendor names are illustrative. All measured numbers are **observations at a point in time**, not design parameters — borrow the method and the order of magnitude, not the thresholds.

---

## Repo topology (components will be open-sourced separately)

This repository holds the **methodology only**. Implementations ship as **one repository per component**:

Both platforms host them under the same account name (`kira905`):

```
ops-handoff-design     [docs]       ← this repo: the authoritative narrative
dsh-<component>        [component]  ← one repo each; its README links back to the
...                                    specific sections here that justify its design
```

Reasons: the licensing is deliberately split (**docs CC BY-NC-SA 4.0 so they cannot be commercially repackaged; code MIT to keep the barrier low**), components then version and release independently, and third-party issues have an unambiguous home.

- **Component index** (what exists, its status, which sections justify it) → [`发布说明.md` §4.5](发布说明.md)
- **What will *not* be open-sourced, and why** → [`发布说明.md` §4.6](发布说明.md) (four admission criteria + a negative list)
- **Component authors**: copy the "Design basis" README snippet from `发布说明.md` §4.3 and add a row to the index.

---

## Translation policy

**Deliberately asymmetric — English README + Chinese technical documents, translated on demand.**

| Content | Language | Why |
|---|---|---|
| README (this file), repo description, release notes | **English + Chinese** | So the repo is discoverable and understandable to an international reader before committing to anything |
| Design documents (01 / 02) | **Chinese first**; English translation per document, when there is demand | A faithful translation of a 1000-line design document is a second artifact to maintain forever. Translating everything up front means either both copies rot, or the translation becomes the reason not to update the original |
| Component READMEs | **English required** | A component's README is its user interface; it is short, and it is what a stranger reads first |

**How a translation happens**: an English file is added as `XX-<name>.en.md` **next to** the Chinese original (never replacing it), linked from both READMEs, and the Chinese file remains the source of truth. If you need one of the design documents in English, **open an issue naming the document** — demand is itself the scheduling signal.

---

## License

- **Documents**: [CC BY-NC-SA 4.0](LICENSE) — attribution + non-commercial + share-alike. You may read, quote, translate, adapt and use them for learning or in non-commercial projects; you may not repackage them commercially, and derivative works must keep the same license.
- **Code / scripts** (if any are distributed alongside): MIT — use freely, including commercially; keep the copyright notice.

Third-party standards, papers and official documentation referenced inside remain the property of their respective owners; they are cited only to identify the basis of a design decision.

---

*Derived from internal production drafts. The production versions remain the single source of truth: these are one-way derived copies, and nothing here flows back into them.*
