---
description: Create a project plan using the project-planner agent. No code — only plan file generation.
argument-hint: <what to plan>
---

# /plan — Project Planning Mode

$ARGUMENTS

---

## 🔴 Rules

1. **NO CODE WRITING** — this command creates a plan file only
2. **Use the `project-planner` agent** (via the `Agent` tool)
3. **Socratic Gate** — ask clarifying questions before planning
4. **Dynamic naming** — plan file named from the task slug

---

## Task

Invoke the `project-planner` agent with this context:

```
CONTEXT:
- User Request: $ARGUMENTS
- Mode: PLANNING ONLY (no code)
- Output: docs/PLAN-{task-slug}.md (dynamic naming)

NAMING:
1. Extract 2-3 key words from the request
2. lowercase, hyphen-separated
3. Max 30 characters
4. Example: "e-commerce cart" → PLAN-ecommerce-cart.md

RULES:
1. Follow project-planner agent's Socratic Gate before planning
2. Create PLAN-{slug}.md with task breakdown, dependencies, verification checklist
3. DO NOT write code
4. REPORT the exact file name created
```

---

## Expected Output

| Deliverable | Location |
|-------------|----------|
| Project Plan | `docs/PLAN-{slug}.md` |
| Task Breakdown | Inside plan file |
| Agent Assignments | Inside plan file |
| Verification Checklist | Final phase of plan file |

---

## After Planning

```
[OK] Plan created: docs/PLAN-{slug}.md

Next:
- Review the plan
- Run /create (new apps) or /enhance (existing) to start implementation
- Or edit the plan manually
```

---

## Naming Examples

| Request | Plan File |
|---------|-----------|
| `/plan e-commerce site with cart` | `docs/PLAN-ecommerce-cart.md` |
| `/plan mobile app for fitness` | `docs/PLAN-fitness-app.md` |
| `/plan add dark mode feature` | `docs/PLAN-dark-mode.md` |
| `/plan fix authentication bug` | `docs/PLAN-auth-fix.md` |
| `/plan FastAPI rate limiting` | `docs/PLAN-rate-limit.md` |

---

## Usage

```
/plan e-commerce site with cart
/plan SaaS dashboard with analytics
/plan LLM observability integration with Langfuse
```
