---
name: myrootcause
description: Verify a proposed root cause before posting it. Use when the user runs /myrootcause, right after /mytask confirms a bug, or whenever someone states why a bug happened — including an explanation produced by an AI — and it has not been checked against evidence. Also use when /mytask returned Not Reproducible but the reporter insists the bug is real, or when the user asks "why did this break", "is this the root cause", "verify this cause", "is PR 123 responsible", or "check this explanation". Takes a proposed cause if one exists; otherwise investigates enough to produce one, then audits it the same way. Runs five checks: the timeline (report date versus the ship date of the suspected change), the environment (what commit and configuration the reported environment is actually running), reproduction in the environment where it was reported rather than only locally, symptom coverage (the cause must explain every symptom, including the behavior that still works), and evidence (every claim cites a commit SHA, file:line, log timestamp, or a configuration value read from the real environment — never prose). Returns Proven, Disproven, or Unproven, and separately rules whether this is a code problem at all: a stale deployment, configuration drift, data-specific state, a wrong role, an already-shipped fix, or a third-party outage ends the chain, because there is nothing to fix. Read-only — never edits code, never runs state-modifying git. Do NOT use for deep debugging and fixing (use ce-debug), for writing verification steps (use /myrepro), or for triaging PR review findings (use /myverdict).
---

# 🧷 /myrootcause — Root Cause Verification

A gate between *thinking* you know why something broke and *saying so*. An explanation that sounds right and an explanation that is right look identical in a ticket comment — the difference only shows up when someone acts on it.

Run it on every confirmed bug. The effort scales with the bug; the trigger does not. "Obvious" is a feeling you have before you have checked, and the expensive bugs are the ones that looked obvious.

## Rules

- **Evidence, or it did not happen.** Every claim cites something checkable: a commit SHA, a `file:line`, a log line with its timestamp, a config value read from the real environment, a query result. Reasoning that cites nothing is **Unproven**, however sound it reads. This rule has no exceptions and applies hardest to explanations that feel obviously correct.
- **Treat an AI's explanation as a hypothesis, never a finding.** A model will reliably produce a fluent, plausible cause — including this one. Fluency is not evidence. Check it against real data before it goes anywhere near a ticket.
- **Rule out the non-code causes first.** Stale deployment, config drift, data-specific state, wrong account or role, a fix already shipped but not released, a third-party outage. These are the cheapest to check and the most embarrassing to miss, and they are invisible if you only read code.
- **The cause must explain what still works.** A cause that accounts for every broken thing but cannot explain why the neighbouring feature is fine is not the cause, or not all of it. This check catches more wrong answers than any other.
- **Check the timeline before you read the diff.** A change that shipped after the first report cannot have caused it. Confirm the dates before spending an hour in a diff.
- **Reproduce where it was reported.** Local behavior is evidence about local. If it was reported on staging, production, a device, or a specific account, that is where the question lives. "Works on my machine" may be solving a different problem.
- **Never invent an environment.** If you cannot establish which commit or config the reported environment runs, that is not a detail to assume past — it is the finding. Return **Unproven** and name exactly what to collect.
- **Do not fix anything.** No code edits, no state-modifying git. Reading code, reading logs, running read-only commands and existing tests is the job.
- **Print the verdict in the chat/terminal** using the exact output format below.

## Steps

1. **State the claim under test**, in one sentence, before investigating. If a cause was handed to you — yours, a teammate's, an AI's — quote it verbatim and name its source. If there is no claim yet, investigate only far enough to form one, then audit it on the same five checks. Writing the claim down first is what stops the verdict from quietly reshaping itself to fit whatever you find.

2. **Rule out the non-code causes.** Before touching the diff, check in this order — cheapest first:
   - Is the reported environment running the commit you think it is?
   - Does its config differ from local: env vars, feature flags, API keys, queue workers, cache?
   - Is it data-specific — one record, one tenant, a missing seed or migration?
   - Is it account- or role-specific?
   - Has it already been fixed, with the reporter seeing a cached or stale page?
   - Was a third-party service degraded at that time?

   Any hit here likely ends the chain. Say so loudly — there is nothing to fix in code.

3. **Check the timeline.** Record when the bug was first reported and when the suspected change shipped, with dates and SHAs. If the change landed after the report, the claim is **Disproven** and the investigation restarts from an earlier window. Do this before reading the diff.

4. **Check the environment.** Establish what the reported environment is actually running — deployed commit, branch, build number, relevant config — and how it differs from where you are testing. Name the source of each value: a deploy log, a health endpoint, a dashboard, a `git log` on the box. If you cannot get it, that is the gap, not an assumption to make.

5. **Reproduce where it was reported.** Run the trigger in the environment the report came from, as the role it came from. Record the outcome as *reproduced there*, *reproduced only locally*, *could not reproduce anywhere*, or *not runnable here* with the reason. Keep this short — raw evidence, not a walkthrough. `/myrepro` writes the walkthrough.

6. **Test symptom coverage.** List every symptom from the report, then add a row for everything adjacent that **still works**. For each, state whether the proposed cause explains it. One unexplained row means the cause is incomplete or wrong — say which.

7. **Audit the evidence.** Walk back through every assertion made so far and attach its citation. Any assertion that cannot be cited is struck or demoted to an open question. Count what is left.

8. **Rule.** One of:
   - **Proven** — cited evidence supports the cause, and it explains every symptom including what still works.
   - **Disproven** — the timeline, the environment, or an unexplained symptom contradicts it. Say what the evidence points at instead, if anything.
   - **Unproven** — plausible, nothing hard behind it. List exactly what to collect to settle it.

   Separately state **code problem: yes / no**. These are independent — a non-code cause can be fully proven.

9. **Hand off.** On **Proven + code problem**, pass forward what the next skills need: environment, account and role, deployed commit, and the exact trigger — the before-state evidence, captured while the code is still broken. On **Proven + not a code problem**, state the actual remedy (redeploy, fix the config, correct the data) and **end the chain**: there is nothing for `/myfix` to do and no AFTER column for `/myrepro` to write. On **Disproven** or **Unproven**, stop and say what is needed rather than handing a guess downstream.

## Output format

Print exactly this structure:

````
### 🎯 Claim under test
- Claim: "[verbatim]"
- Source: [me / teammate / AI / none — formed during this check]
- Bug: [one line — what was reported, by whom, when]

### 🧰 Is this a code problem at all?
| Check | Finding | Evidence |
|---|---|---|
| Deployed commit matches expectation | ✅ / ❌ | [SHA + where it was read] |
| Config matches local | ✅ / ❌ | [the differing value + source] |
| Data-specific | ✅ / ❌ | [query or record] |
| Role / account specific | ✅ / ❌ | ... |
| Already fixed, stale view | ✅ / ❌ | ... |
| Third-party degraded | ✅ / ❌ | ... |

### ⏱️ Timeline
| Event | When | Evidence |
|---|---|---|
| First reported | [date] | [ticket / message] |
| Suspected change shipped | [date] | `[SHA]` [merge or deploy record] |
- Verdict: [change predates the report ✅ / postdates it ❌ — claim cannot hold]

### 🖥️ Environment
| | Reported environment | Where I tested |
|---|---|---|
| Commit | `[SHA]` | `[SHA]` |
| Config that matters | ... | ... |
| Account / role | ... | ... |
- Source of these values: [deploy log / health endpoint / dashboard]

### 🔁 Reproduction
- Result: **reproduced there** / **reproduced only locally** / **could not reproduce anywhere** / **not runnable here**
- What I ran: `...`
- What happened: [quoted output]

### 🧩 Symptom coverage
| Symptom | Explained by this cause? | Evidence |
|---|---|---|
| [broken thing from the report] | ✅ | `path/file.ts:120` |
| [another broken thing] | ✅ | ... |
| **[adjacent thing that STILL WORKS]** | ✅ / ❌ | [why the cause leaves it intact] |
- Unexplained rows: [none / list them — each one means the cause is incomplete]

### ⚖️ Verdict
- **Proven** / **Disproven** / **Unproven**
- **Code problem: yes / no**
- Why, in two lines: ...

### 📋 Evidence ledger
| # | Assertion | Citation |
|---|---|---|
| 1 | ... | `[SHA]` / `file:line` / [log @ timestamp] |
- Uncited assertions: [none / struck — listed here]

### ❓ To settle this  *(Unproven only)*
- [the exact artifact to collect, and who can get it]

### ➡️ Next
- [Proven + code → hand to /myrepro: environment, role, commit, trigger]
- [Proven + not code → the real remedy, and the chain ends here]
- [Disproven / Unproven → what to do instead]
````

If the cause came from an AI, say so in **Source** and keep it there through the verdict. A model's explanation that survived this check is worth more than one that was never tested; one that did not survive is worth recording so the same story is not retold next week.

## Goal

Make a root cause something you proved rather than something you found convincing. Run it after `/mytask` confirms a bug and before the explanation reaches a ticket, a standup, or a fix — so the work that follows is aimed at the actual cause, and so the cheapest answers, the ones that are not code at all, get ruled out before anyone opens a diff.
