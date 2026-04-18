---
description: Generate tests, run existing tests, or show coverage. Stack-aware (pytest, Jest, Vitest, etc.).
argument-hint: [target|coverage|watch]
tier: MEDIUM
tier-rationale: Single test-engineer agent; may write 1–5 test files or just run existing suite.
estimated-tokens: "15k–60k"
risk: Generating tests from scratch for a large module pulls many file reads and can drift toward HEAVY.
---

# /test — Test Generation & Execution

$ARGUMENTS

---

## Purpose

Generate tests, run existing tests, or report coverage. Adapts to the detected stack.

### Sub-commands

```
/test                — run all tests
/test <file/feature> — generate tests for target
/test coverage       — show coverage report
/test watch          — watch mode
```

---

## Stack Detection & Commands

| Stack | Run | Coverage | Watch |
|-------|-----|----------|-------|
| **Jest / Vitest** | `npm test` | `npm run coverage` or `vitest --coverage` | `vitest --watch` |
| **pytest** | `pytest` | `pytest --cov=app --cov-report=term-missing` | `pytest-watch` |
| **Playwright** | `npx playwright test` | `--reporter=html` | `npx playwright test --ui` |
| **go test** | `go test ./...` | `go test -cover ./...` | N/A |
| **cargo** | `cargo test` | `cargo tarpaulin` | `cargo watch -x test` |

---

## Generation Flow

1. **Analyze target**
   - Parse functions, methods, classes
   - Identify edge cases (empty input, None, large values, concurrent access)
   - Detect external dependencies to mock

2. **Generate test cases**
   - Happy path
   - Error cases
   - Edge cases
   - Integration (if target spans layers)

3. **Write tests**
   - Match project's existing test framework
   - Follow existing conventions (fixtures, naming, directory layout)
   - Mock only external boundaries; never the code under test

---

## Output Format

### For Generation

```markdown
## 🧪 Tests: [Target]

### Test Plan
| Test Case | Type | Coverage |
|-----------|------|----------|
| Should create user | Unit | Happy path |
| Should reject invalid email | Unit | Validation |
| Should handle DB error | Integration | Error case |

### Generated Tests
`tests/<file>.test.ts` — or — `tests/unit/test_<file>.py`

[code block]

Run with: `pytest tests/unit/test_<file>.py` or `npm test`
```

### For Execution

```
🧪 Running tests…

✅ tests/unit/test_auth.py (5 passed)
✅ tests/unit/test_user.py (8 passed)
❌ tests/integration/test_order.py (2 passed, 1 failed)

Failed:
  ✗ test_calculate_total_with_discount
    Expected: 90
    Received: 100

Total: 15 (14 passed, 1 failed)
```

---

## FastAPI Test Patterns

### Async Integration Test

```python
from httpx import AsyncClient

async def test_create_user(client: AsyncClient):
    r = await client.post("/api/v1/users", json={"email": "a@b.com"})
    assert r.status_code == 201
    assert r.json()["email"] == "a@b.com"
```

> Uses `asyncio_mode = "auto"` — no `@pytest.mark.asyncio` needed.

### Mocking LLM Calls

```python
from unittest.mock import AsyncMock

async def test_chat_uses_gemini(monkeypatch):
    mock = AsyncMock(return_value="hello")
    monkeypatch.setattr("app.services.llm.gemini_service.generate", mock)
    # ... call the endpoint, assert on behavior
```

---

## Examples

```
/test app/services/auth_service.py
/test user registration flow
/test coverage
/test --watch
/test integration/test_llm_retry.py
```

---

## Key Principles

- **Test behavior, not implementation**
- **One assertion per test** (when practical)
- **Descriptive names** (`test_rejects_invalid_email` beats `test_1`)
- **Arrange-Act-Assert pattern**
- **Mock external APIs, DBs in unit tests — real DB in integration tests**
