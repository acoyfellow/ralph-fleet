# Ralph Loop Implementation Status

## ✅ Completed

### 1. GitHub Actions Workflow
**File:** `.github/workflows/ralph-loop.yml`

**Capabilities:**
- ✅ Reads `scripts/ralph/prd.json` for pending stories
- ✅ Executes story verification commands
- ✅ Continues on error (retry logic)
- ✅ Heal step logs failure details (stub for LLM integration)
- ✅ Retries verification after heal attempt
- ✅ Updates PRD status to "completed" on success
- ✅ Appends to `scripts/ralph/progress.txt`
- ✅ Auto-commits changes as `ralph-loop[bot]`
- ✅ Uses `GHT` secret (GitHub token workaround)
- ✅ Validates YAML syntax

**Triggers:**
- Push to `main` or `copilot/**` branches
- Changes to `scripts/ralph/prd.json` or workflow file
- Manual workflow dispatch

### 2. Smoke Test Infrastructure
**Story:** `000 - Smoke test - Ralph Loop e2e`
- ✅ Added to PRD as first pending story
- ✅ Created `scripts/smoke-test.sh`
- ✅ Test designed to fail initially
- ✅ Clear fix instructions in error message
- ✅ Marker file excluded from git (in .gitignore)

### 3. Documentation
- ✅ `.github/workflows/README.md` - Workflow guide
- ✅ `E2E_SMOKE_TEST_SUMMARY.md` - Complete setup guide
- ✅ `IMPLEMENTATION_STATUS.md` - This file

## 🔄 Testing Required

### Prerequisites
1. **Configure GHT Secret:**
   - Go to: `Settings → Secrets → Actions`
   - Create secret: `GHT`
   - Value: Personal Access Token with `repo` + `workflow` scopes

### Test Flow
```bash
# 1. Push/merge this branch - workflow will run and FAIL (expected)
git push origin copilot/add-smoke-test-to-prd

# 2. Check GitHub Actions tab - should see:
#    - Workflow triggered
#    - Story 000 verification fails
#    - Heal step logs failure
#    - Overall workflow fails

# 3. Simulate the "heal" by fixing the issue:
touch scripts/.smoke-test-fixed
git add scripts/.smoke-test-fixed
git commit -m "fix: add smoke test marker (simulating heal)"
git push

# 4. Workflow runs again - should see:
#    - Story 000 verification passes
#    - PRD updated (status: "completed")
#    - progress.txt updated
#    - New commit by ralph-loop[bot]

# 5. Verify the loop:
cat scripts/ralph/prd.json | jq '.stories[0].status'
# Should output: "completed"
```

## 📋 What Happens E2E

1. **First Push** → Workflow triggers
2. **Check Stories** → Finds story 000 (pending)
3. **Run Verification** → `bash scripts/smoke-test.sh` → FAILS ❌
4. **Heal Attempt** → Logs details (LLM stub)
5. **Retry** → Still fails (no fix applied yet)
6. **Workflow Fails** → Exits with error

---

7. **Manual Fix** → `touch scripts/.smoke-test-fixed`
8. **Second Push** → Workflow triggers again
9. **Check Stories** → Still story 000 (pending)
10. **Run Verification** → `bash scripts/smoke-test.sh` → PASSES ✅
11. **Update PRD** → story 000 status → "completed"
12. **Commit Changes** → `ralph-loop[bot]` commits
13. **Workflow Succeeds** → All steps green

## 🚧 Known Limitations

### Heal Step is a Stub
Currently just logs failure details. To implement real LLM healing:

```yaml
- name: Attempt LLM heal on failure
  run: |
    # Call LLM API with context:
    # - Error logs from verification
    # - Story acceptance criteria
    # - Recent commits, file diffs
    # - PRD context
    
    # Parse LLM response
    # Apply suggested fix (create commit)
    # Push changes
```

### No Retry Limits
Workflow will retry once after heal. Production should:
- Limit heal attempts (e.g., max 3)
- Add exponential backoff
- Mark story as "blocked" if all retries fail

### No PR Integration
Heal step could:
- Create PR with suggested fix
- Comment on existing PR
- Tag relevant maintainers

## 🎯 Success Criteria

### Immediate (Manual Test)
- [ ] Workflow runs on push
- [ ] Story 000 fails verification initially
- [ ] Heal step logs failure
- [ ] After manual fix, verification passes
- [ ] PRD updated automatically
- [ ] progress.txt updated
- [ ] Commit pushed by ralph-loop[bot]

### Future (LLM Integration)
- [ ] LLM API receives failure context
- [ ] LLM suggests actionable fix
- [ ] Fix applied automatically
- [ ] Verification re-runs and passes
- [ ] No manual intervention needed

## 📁 Files Created/Modified

```
.github/
  workflows/
    ralph-loop.yml         # NEW - Main workflow
    README.md              # NEW - Workflow docs

scripts/
  ralph/
    prd.json               # MODIFIED - Added story 000
  smoke-test.sh            # NEW - Test script

.gitignore                 # MODIFIED - Added marker file

E2E_SMOKE_TEST_SUMMARY.md  # NEW - Setup guide
IMPLEMENTATION_STATUS.md   # NEW - This file
```

## 🔑 Key Design Decisions

1. **GHT vs GITHUB_TOKEN:** GitHub doesn't allow secrets prefixed with `GITHUB_`, so we use `GHT` instead.

2. **Smoke Test Design:** Intentionally fails to demonstrate the retry/heal loop. Real-world stories should pass if implemented correctly.

3. **Heal Stub:** Logs failure details but doesn't call LLM. This allows testing the workflow logic without external dependencies.

4. **Auto-commit:** On success, workflow commits as `ralph-loop[bot]` to avoid triggering itself recursively (workflow only triggers on PRD changes).

5. **Marker File Pattern:** Simple way to simulate a fix. Real healing would modify actual source files.

## 🚀 Ready to Test

All implementation complete. Needs:
1. GHT secret configured in repo
2. Push to trigger workflow
3. Manual intervention to simulate heal

Once tested, the workflow is ready for real LLM integration.
