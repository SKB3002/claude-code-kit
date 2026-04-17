---
description: Add or update features in an existing application. Iterative development mode.
argument-hint: <change to make>
---

# /enhance — Update Application

$ARGUMENTS

---

## Task

Add features or make updates to an existing application.

### Steps

1. **Understand current state**
   - Read the repo structure (use built-in `Explore` agent for quick mapping)
   - Identify existing tech stack and conventions
   - Read `CLAUDE.md` / `KIT_PROTOCOL.md` if present

2. **Plan changes**
   - Determine what will be added / changed / removed
   - Detect affected files and dependents
   - Flag risky changes (migrations, public APIs, auth)

3. **Present plan for major changes**
   ```
   To add the admin panel:
   - Create 15 files
   - Update 8 files
   - Needs 1 migration
   - ~10 min of work

   Proceed? (y/n)
   ```

4. **Apply**
   - Invoke relevant agents (parallel when independent)
   - Make changes
   - Run appropriate validation scripts

5. **Follow-up**
   - Hot reload / restart as needed
   - Commit each change cleanly

---

## Examples

```
/enhance add dark mode
/enhance build admin panel
/enhance integrate Stripe payments
/enhance add full-text search
/enhance edit profile page
/enhance make the dashboard responsive
/enhance add rate limiting to /api/v1/generate
/enhance switch from LangSmith to Langfuse
```

---

## Caution

- Get approval for major changes before editing files
- Warn on conflicting requests (e.g. "use Firebase" when project uses Postgres)
- Commit each logical change with a clear message
- If a change affects >10 files, propose a plan via `project-planner` first
