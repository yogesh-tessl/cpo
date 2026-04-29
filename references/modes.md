# Secondary Modes

## `--review` Mode

Verify past decisions against current reality. Only runs when explicitly requested.

```bash
grep -l "status: active" ~/.cpo/decisions/*.yaml 2>/dev/null | while read -r f; do echo "---"; cat "$f"; done
```

For each active decision, output:
```
**#[decision_id]** — [decision summary] ([date])
Kill criteria: [list each criterion with status]
→ Ask: "Has [criterion metric] crossed [threshold]? Share current data."
Action: [keep active / close / update]
```

Surface criteria and ask the user for current data — do not evaluate independently.

---

## `--outcome` Mode

Close the loop on a past decision. Usage: `/cpo --outcome #decision-name` or `/cpo --outcome [topic]`.

```bash
grep -rl "decision_id: DECISION_ID" ~/.cpo/decisions/*.yaml 2>/dev/null | while read -r f; do cat "$f"; echo "---"; done
```

Present: decision summary, verdict, door type, each kill criterion with **Status?**, each flagged assumption with **Still true?**.

Then AskUserQuestion:
- A) Walk through each kill criterion (recommended for one-way doors)
- B) Quick close — one-line summary of what happened
- C) Decision was wrong — I want to understand why

**If A:** Ask for current data per criterion via AskUserQuestion. Write outcome YAML block with: `date`, `result` (succeeded/failed/pivoted/abandoned), `kill_criteria_results` (criterion, threshold, actual, triggered), `lesson`, `assumptions_validated`. Update `status: active` → `status: closed`.

**If B:** Ask "What happened in one sentence?" Write minimal outcome with `result` and `lesson` only.

**If C:** Present a decision replay (information state at decision time: Truths, assumptions, blind spots). Ask what they'd change about the frame. Write outcome with `result: failed`.

After any close: *"This is your Nth closed decision. Pattern so far: [X succeeded, Y failed, Z pivoted]."*

---

## `--save-context` Mode

Bootstrap or update `~/.cpo/context.md`. Ask via AskUserQuestion (one at a time):
1. Stage (pre-PMF / post-PMF / Series B+)
2. Business model (SaaS / marketplace / API / other)
3. Core constraint (time / money / people / tech)
4. Top 3 priorities right now
5. Biggest open question

After all 5 answered, write to `~/.cpo/context.md`. Confirm: *"Context saved."*

---

## `--decide` Mode (Inbound Handoff)

Look for a `CPO Handoff Request` block (`From`, `Context`, `Decision`, `Options considered`). If found: use as prompt, skip "Right problem?"/forcing question/delay test, keep "Who benefits?", run standard flow from Frame. After verdict, suggest returning to the calling skill. If no block found: treat as normal `/cpo`.

---

## `--version` Mode

Output health check status (install path, context file, journal count, signals). Run:
```bash
ls ~/.cpo/context.md 2>/dev/null && echo "context: found" || echo "context: not found"
ls ~/.cpo/decisions/ 2>/dev/null | wc -l | tr -d ' '
ls ~/.cpo/signals/ 2>/dev/null && echo "signals: found" || echo "signals: not found"
```
No gates. No AskUserQuestion. Pure status output.
