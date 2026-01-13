# E2E Smoke Test Setup - Summary

## What Was Built

### 1. Ralph Loop Workflow (`.github/workflows/ralph-loop.yml`)
Implements the complete Ralph Loop automation:

**Features:**
- ✅ Reads `scripts/ralph/prd.json` for pending stories
- ✅ Executes verification command from story
- ✅ Retries on failure with heal stub (logs for LLM assistance)
- ✅ Updates PRD + progress.txt on success
- ✅ Auto-commits changes as `ralph-loop[bot]`
- ✅ Uses `GHT` secret (GitHub doesn't allow `GITHUB_` prefix)

**Triggers:**
- Push to `main` or `copilot/**` when PRD or workflow changes
- Manual dispatch

### 2. Smoke Test Story (`scripts/ralph/prd.json` - Story 000)
First story in PRD designed to test the loop:

```json
{
  "id": "000",
  "title": "Smoke test - Ralph Loop e2e",
  "status": "pending",
  "verification": "bash scripts/smoke-test.sh"
}
```

### 3. Smoke Test Script (`scripts/smoke-test.sh`)
Intentionally fails until marker file exists:

```bash
# FAILS if missing: scripts/.smoke-test-fixed
# PASSES when file exists
```

## How It Works - E2E Flow

```
1. Push triggers workflow
   ↓
2. Workflow reads PRD → finds story 000 (pending)
   ↓
3. Runs: bash scripts/smoke-test.sh
   ↓
4. FAILS (missing marker file) ❌
   ↓
5. "Heal" step logs failure (stub - no actual LLM yet)
   ↓
6. Workflow exits with error
   ↓
7. [MANUAL] Simulate heal: touch scripts/.smoke-test-fixed
   ↓
8. Push marker file → triggers workflow again
   ↓
9. Runs verification again
   ↓
10. PASSES ✅
    ↓
11. Updates PRD: story 000 status → "completed"
    ↓
12. Commits + pushes changes automatically
```

## Setup Required

### In GitHub Repo Settings:
1. Go to: Settings → Secrets and variables → Actions
2. Create new secret:
   - Name: `GHT`
   - Value: Personal Access Token with scopes:
     - `repo` (full control)
     - `workflow` (update workflows)

### To Test:
```bash
# 1. Merge/push this PR - workflow WILL FAIL (expected!)
# Check Actions tab for the failure

# 2. Simulate the "heal" by adding the fix:
touch scripts/.smoke-test-fixed
git add scripts/.smoke-test-fixed
git commit -m "fix: add smoke test marker (simulating heal)"
git push

# 3. Workflow runs again - should PASS
# Check Actions tab - should see:
# - Story 000 verification passes
# - PRD updated (status: "completed")
# - New commit by ralph-loop[bot]
```

## What's Missing (Future Work)

The heal step is currently a **stub**. To complete the LLM integration:

1. **Capture failure context:**
   - Error logs from verification
   - Story details, acceptance criteria
   - Recent commits, file changes

2. **Call LLM API:**
   - Send context to LLM (OpenAI, Anthropic, etc.)
   - Request: "Analyze failure, suggest fix"
   - Parse response for actionable steps

3. **Apply fix:**
   - Create new commit with fix
   - Re-run verification
   - If pass → update PRD
   - If fail → escalate (create issue, notify)

4. **Add retry limits:**
   - Max 3 heal attempts
   - Exponential backoff between retries
   - Mark story as "blocked" if all retries fail

## Files Changed

- `.github/workflows/ralph-loop.yml` - Main workflow (NEW)
- `.github/workflows/README.md` - Workflow docs (NEW)
- `scripts/ralph/prd.json` - Added story 000 smoke test
- `scripts/smoke-test.sh` - Test script (NEW)
- `scripts/.smoke-test-fixed` - Will be created during test (NOT IN REPO)

## Success Criteria

✅ Workflow file exists and is valid YAML
✅ Smoke test script fails without marker file
✅ PRD story 000 is pending
✅ Workflow uses GHT secret (not GITHUB_TOKEN)
✅ Workflow has retry logic
✅ Workflow updates PRD on success
⏳ Workflow runs on push (needs GHT secret configured)
⏳ E2E test passes (manual intervention required)

## Next Steps

1. **Configure GHT secret** in repo settings
2. **Merge this PR** → workflow will run and fail (expected)
3. **Add marker file** → workflow re-runs and passes
4. **Verify** story 000 is marked "completed" in PRD
5. **(Future)** Replace heal stub with real LLM integration
