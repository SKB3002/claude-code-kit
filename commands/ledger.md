---
description: Read-only views over .kit/usage.json — weekly totals, per-agent/skill/tier breakdowns, ROI, and verdict tagging.
argument-hint: [weekly|week <YYYY-Www>|by-agent|by-skill|by-tier|command <name>|roi|verdict <id> <tag>|clear]
tier: LIGHT
tier-rationale: Pure read of a local JSON file plus arithmetic aggregation. No agents, no network.
estimated-tokens: "<3k"
risk: Only `clear` is destructive, and it requires explicit confirmation.
---

# /kit:ledger — Usage log views

$ARGUMENTS

> All views read `<project-root>/.kit/usage.json`. Every number shown here is **approximate** — derived from response sizes, not Anthropic billing. Prefixed `~` throughout.

---

## Flow

**Step 1 — Locate the log.**
Walk upward from cwd to find `.kit/usage.json` under the project root (detected by `.git/`, `package.json`, `pyproject.toml`, `Cargo.toml`, or `go.mod`). If none exists, print:

```
📒  No usage log yet at .kit/usage.json — run any MEDIUM or HEAVY command to start tracking.
    e.g. /kit:debug <bug>, /kit:plan <feature>, /kit:brainstorm <idea>
```

and stop. Never auto-create; the file is born from the first MEDIUM or HEAVY command run (those commands write it themselves after dispatch).

**Step 2 — Parse the sub-command.**

| `$ARGUMENTS` | View |
|---|---|
| *empty*                               | RECENT — last 5 runs, most recent first |
| `weekly`                              | WEEKLY — current ISO week totals |
| `week <YYYY-Www>`                     | WEEKLY for a specific ISO week (e.g. `2026-W16`) |
| `by-agent`                            | Rank all agents across history by ~tokens + run count |
| `by-skill`                            | Same, for skills |
| `by-tier`                             | Totals per LIGHT / MEDIUM / HEAVY |
| `command <name>`                      | Last 10 runs of one command + tier drift markers |
| `roi`                                 | ROI view (see §ROI) |
| `verdict <id> <useful\|wasted\|partial> [note]` | Tag one run after the fact |
| `clear`                               | Wipe the log (confirmation required) |
| anything else                         | Print usage help |

---

## Rendering contracts

### RECENT (default)

```
📒  /kit:ledger — last 5 runs

<YYYY-MM-DD HH:MM>  /kit:<cmd>  <tier> · ~<N>k · verdict: <tag|—>   <id>
...
```

### WEEKLY

```
📅  /kit:ledger weekly — <YYYY-Www> (<Mon DD>–<Sun DD>)

Runs: <total>
  LIGHT   <N> (<%>)  — ~<N>k tokens · avg ~<N>k
  MEDIUM  <N> (<%>)  — ~<N>k tokens · avg ~<N>k
  HEAVY   <N> (<%>)  — ~<N>k tokens · avg ~<N>k

Top agents (by ~tokens): <a> (<%>), <b> (<%>), <c> (<%>)
Top skills: <x> (used Nx), <y> (Nx), <z> (Nx)

Cancelled at gate: <N> runs (estimated saved: ~<N>k tokens)
Tier drift: <N> runs exceeded declared tier <(list commands)>

Compared to last week (<prev-week>): <+/-N%> total, <+/-N> HEAVY runs
```

Cancelled-saved is computed from the midpoint of each cancelled run's declared `estimated_tokens` range. Mark it `~` and `estimated`.

### BY-AGENT / BY-SKILL / BY-TIER

Ranked tables, one line per item:
```
<rank>  <name>       ~<tokens>  (<run count> runs)  avg ~<N>k
```

### COMMAND `<name>`

```
📒  /kit:ledger command /kit:<name> — last 10

<YYYY-MM-DD HH:MM>  ~<N>k   declared <tier> / observed <tier>   <in-tier ✓ | drift ↑N% ✗>   <id>
...
Drift summary: <N>/10 runs exceeded declared tier.
  If drift ≥ 3, consider: <suggested tier promotion>
```

### VERDICT

```
/kit:ledger verdict r_2026-04-19_001 useful
/kit:ledger verdict r_2026-04-19_001 wasted "scope too broad, should have picked (b)"
/kit:ledger verdict r_2026-04-19_001 partial
```

Updates that run's `user_verdict` (and `notes` if provided). Atomic write (temp + rename). If the id is not found, print `Run <id> not found.` and stop.

### ROI

```
💰  /kit:ledger roi (all time in this project)

Total approximate tokens: ~<N>M across <N> runs since <first date>
Runs you tagged "useful":    <N>  (<%>) — ~<N>k tokens
Runs you tagged "wasted":    <N>  (<%>) — ~<N>k tokens
Runs you tagged "partial":   <N>  (<%>) — ~<N>k tokens
Untagged:                    <N>  (<%>) — ~<N>k tokens

Most-useful-per-token command:  /kit:<name>  (<%> useful, avg ~<N>k)
Least-useful-per-token command: /kit:<name>  (<%> useful, avg ~<N>k)
Most-cancelled-at-gate:         /kit:<name>  (<N> cancels in <N> approvals)

Interpretation is yours. We don't assign dollar values.
```

Only tagged runs count toward "useful/wasted/partial" ratios. Untagged runs stay untagged — never inferred.

### CLEAR

1. Print the file path and total run count.
2. Ask: `Wipe .kit/usage.json? This cannot be undone. (y/N)`
3. On `y`: delete the file. Print `🧹 Wiped <path>.`
4. Anything else: `Cancelled.` and stop.

---

## Honesty rules (inherited from [skills/approval-gate/SKILL.md](../skills/approval-gate/SKILL.md))

- Every token number carries `~`
- Never convert tokens to dollars
- "Approximate, not Anthropic billing" appears in the header of every aggregated view
- If the log is corrupt, bail loudly — never silently overwrite. Suggest inspecting `.kit/usage-RECOVERED.json` if it exists.

---

## Usage

```
/kit:ledger                                  # last 5 runs
/kit:ledger weekly                           # this ISO week
/kit:ledger week 2026-W16                    # specific week
/kit:ledger by-agent                         # ranked agents
/kit:ledger by-skill                         # ranked skills
/kit:ledger by-tier                          # LIGHT/MEDIUM/HEAVY totals
/kit:ledger command /kit:enhance             # last 10 runs of one command
/kit:ledger roi                              # all-time ROI view
/kit:ledger verdict r_2026-04-19_001 useful  # tag a run
/kit:ledger clear                            # wipe the log
```
