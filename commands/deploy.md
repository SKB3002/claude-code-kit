---
description: Production deployment with pre-flight checks, deploy execution, and verification.
argument-hint: [check|preview|production|rollback]
---

# /deploy — Production Deployment

$ARGUMENTS

---

## Purpose

Run pre-flight checks, execute the deployment, and verify.

## Sub-commands

```
/deploy            - Interactive deployment wizard
/deploy check      - Pre-deployment checks only
/deploy preview    - Deploy to preview/staging
/deploy production - Deploy to production
/deploy rollback   - Rollback to previous version
```

---

## Pre-Deployment Checklist

```markdown
## 🚀 Pre-Deploy Checklist

### Code Quality
- [ ] Lint passes (`npm run lint` | `ruff check .`)
- [ ] Types pass (`npx tsc --noEmit` | `mypy .`)
- [ ] Tests passing (`npm test` | `pytest`)

### Security
- [ ] No hardcoded secrets
- [ ] Environment variables documented in `.env.example`
- [ ] Dependencies audited (`npm audit` | `pip-audit`)
- [ ] `${CLAUDE_PLUGIN_ROOT}/skills/vulnerability-scanner/scripts/security_scan.py .` clean

### Performance
- [ ] Bundle size acceptable (Node)
- [ ] No debug `console.log` / `print(...)` left
- [ ] Images optimized

### Documentation
- [ ] README updated
- [ ] CHANGELOG updated
- [ ] API docs current

### Ready to deploy? (y/n)
```

---

## Deployment Flow

```
/deploy → Pre-flight checks → Build → Deploy → Health check → ✅
           (fail → surface errors, offer rollback)
```

---

## Output Format

### Successful Deploy

```markdown
## 🚀 Deployment Complete
- Version: v1.2.3
- Environment: production
- Duration: 47s
- Platform: Vercel / Fly.io / Railway / K8s

### Health Check
✅ API responding
✅ DB connected
✅ Downstream services healthy
```

### Failed Deploy

```markdown
## ❌ Deployment Failed
### Error
Build failed at: <step>
### Resolution
1. Fix <file:line>
2. Run build locally
3. Re-try `/deploy`

Rollback available — run `/deploy rollback` if needed.
```

---

## Platform Support

| Platform | Command |
|----------|---------|
| Vercel | `vercel --prod` |
| Railway | `railway up` |
| Fly.io | `fly deploy` |
| Docker | `docker compose up -d` |
| Render | `render deploy` |
| Heroku | `git push heroku main` |
| Kubernetes | `kubectl apply -f k8s/` |
| FastAPI (gunicorn) | `gunicorn app.main:app -c gunicorn.conf.py` |

---

## Examples

```
/deploy
/deploy check
/deploy preview
/deploy production
/deploy rollback
```
