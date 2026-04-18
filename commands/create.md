---
description: Create a new application. Triggers the app-builder skill and coordinates project-planner, database-architect, backend-specialist, and frontend-specialist.
argument-hint: <what to build>
---

# /create — New Application

$ARGUMENTS

---

## Task

Start a new application from a natural-language request.

### Steps

1. **Request analysis**
   - Understand what the user wants
   - If information is missing, invoke the `socratic-gate` skill to ask

2. **Project planning**
   - Invoke the `project-planner` agent for task breakdown
   - Determine tech stack (use `app-builder` skill's `tech-stack.md`)
   - Plan file structure
   - Write `docs/PLAN-{task-slug}.md`

3. **Application building (after approval)**
   - Orchestrate with the `app-builder` skill
   - Coordinate specialist agents:
     - `database-architect` → schema + migrations
     - `backend-specialist` → API + services
     - `frontend-specialist` → UI + pages

4. **Verification**
   - Run skill-local validation scripts co-located under `${CLAUDE_PLUGIN_ROOT}/skills/<skill>/scripts/`

---

## Usage

```
/create blog site
/create e-commerce app with product listing and cart
/create todo app
/create Instagram clone
/create CRM with customer management
/create FastAPI service that exposes Gemini with function calling
```

---

## Before Starting

If the request is unclear, ask:

- What type of application?
- What are the core features?
- Who will use it?

Use sensible defaults — add details later.
