---
description: Coordinate multiple agents for complex multi-domain tasks. Use for comprehensive reviews or cross-stack implementation.
argument-hint: <task to orchestrate>
tier: HEAVY
tier-rationale: Contract guarantees ≥3 agents with 2-phase Plan→approve→Implement; parallel fan-out.
estimated-tokens: "80k–250k"
risk: Broad tasks ("review the whole repo") can pull 5+ agents; always the most expensive command in the kit.
---

# /orchestrate — Multi-Agent Orchestration

You are now in **ORCHESTRATION MODE**. Your task: coordinate specialized agents to solve this complex problem.

## Task

$ARGUMENTS

---

## 🔴 Minimum Agent Requirement

> **ORCHESTRATION = MINIMUM 3 DIFFERENT AGENTS**
>
> Fewer than 3 ≠ orchestration — it's just delegation.
>
> Before completion: count invoked agents. If `< 3` → invoke more or run validation scripts.

### Agent Selection Matrix

| Task Type | Minimum Agents |
|-----------|----------------|
| **Web App** | frontend-specialist, backend-specialist, test-engineer |
| **API (Python/FastAPI)** | backend-specialist, database-architect, security-auditor |
| **API (Node)** | backend-specialist, security-auditor, test-engineer |
| **UI / Design** | frontend-specialist, seo-specialist, performance-optimizer |
| **Database** | database-architect, backend-specialist, security-auditor |
| **Full Stack** | project-planner, frontend-specialist, backend-specialist, devops-engineer |
| **Debug** | debugger, explorer-agent, test-engineer |
| **Security Audit** | security-auditor, penetration-tester, devops-engineer |
| **LLM Feature** | backend-specialist, test-engineer, performance-optimizer |

---

## 🔴 2-Phase Protocol

### PHASE 1: Planning (Sequential)

| Step | Agent | Action |
|------|-------|--------|
| 1 | `project-planner` | Create `docs/PLAN-{slug}.md` |
| 2 | (optional) `explorer-agent` | Deep codebase discovery |

> NO OTHER AGENTS DURING PLANNING.

### ⏸ Checkpoint: User Approval

```
Plan ready: docs/PLAN-{slug}.md
Approve? (Y/N)
  Y → implementation
  N → revise
```

> Do NOT proceed to Phase 2 without explicit approval.

### PHASE 2: Implementation (Parallel)

Run independent agents in PARALLEL (one assistant turn, multiple `Agent` tool calls):

| Wave | Agents |
|------|--------|
| Foundation | database-architect, security-auditor |
| Core | backend-specialist, frontend-specialist |
| Polish | test-engineer, devops-engineer |

---

## Kit Agents (20)

`orchestrator`, `project-planner`, `product-manager`, `product-owner`, `security-auditor`, `penetration-tester`, `backend-specialist`, `frontend-specialist`, `mobile-developer`, `database-architect`, `test-engineer`, `qa-automation-engineer`, `devops-engineer`, `performance-optimizer`, `debugger`, `code-archaeologist`, `explorer-agent`, `documentation-writer`, `seo-specialist`, `game-developer`

See the `parallel-agents` skill for the full capability matrix.

---

## Context Passing (MANDATORY)

When invoking any subagent, include:

1. **Original User Request** — full text
2. **Decisions Made** — answers to Socratic questions
3. **Previous Agent Work** — summary
4. **Current Plan State** — reference or excerpt of `PLAN-{slug}.md`

Example:

```
Agent(subagent_type="backend-specialist", prompt="""
CONTEXT:
- User Request: "Add rate limiting to the LLM gateway"
- Decisions: token bucket, Redis backend, 60 req/min per user
- Previous Work: project-planner wrote docs/PLAN-rate-limit.md with 4 tasks
- Plan State: Tasks 1-2 done (schema, middleware skeleton); Tasks 3-4 remaining

TASK: Implement tasks 3 and 4 from PLAN-rate-limit.md. Use the fastapi-expert skill.
""")
```

Never invoke a subagent without this context — it will re-derive things and make wrong assumptions.

---

## Verification (Final Step)

After implementation, run appropriate validation scripts:

```bash
python ${CLAUDE_PLUGIN_ROOT}/skills/vulnerability-scanner/scripts/security_scan.py .
python ${CLAUDE_PLUGIN_ROOT}/skills/lint-and-validate/scripts/lint_runner.py .
```

---

## Output Format

```markdown
## 🎼 Orchestration Report

### Task
[Summary]

### Agents Invoked (≥3)
| # | Agent | Focus | Status |
|---|-------|-------|--------|
| 1 | project-planner | Plan | ✅ |
| 2 | backend-specialist | API routes | ✅ |
| 3 | test-engineer | Coverage | ✅ |

### Scripts Run
- security_scan.py → Pass
- lint_runner.py → Pass

### Key Findings
1. [Agent 1]: finding
2. [Agent 2]: finding
3. [Agent 3]: finding

### Deliverables
- [x] PLAN-{slug}.md
- [x] Code implemented
- [x] Tests passing

### Summary
[One-paragraph synthesis]
```

---

## Exit Gate

Before marking complete:

1. ✅ `agents_invoked >= 3`
2. ✅ At least one validation script ran
3. ✅ Orchestration Report generated

> If any fails → keep going. Do not close the loop.
