# Ralph Loop Workflow

## Overview

This workflow implements the core Ralph Loop automation:
1. Reads pending stories from `scripts/ralph/prd.json`
2. Executes verification for the first pending story
3. On failure: attempts LLM-assisted healing and retries
4. On success: updates PRD, marks story complete, commits changes

## Triggers

- Push to `main` or `copilot/**` branches when:
  - `scripts/ralph/prd.json` changes
  - `.github/workflows/ralph-loop.yml` changes
- Manual dispatch via Actions UI

## Secret Configuration

**Important**: This workflow uses `GHT` secret instead of `GITHUB_TOKEN` because GitHub Actions doesn't allow secrets prefixed with `GITHUB_`.

To configure:
1. Go to repo Settings → Secrets → Actions
2. Create secret named `GHT`
3. Value: Personal Access Token with `repo` and `workflow` scopes

## Workflow Steps

1. **Check for pending stories**: Query PRD for `status: "pending"` stories
2. **Execute verification**: Run verification command from story
3. **Heal on failure**: Log failure details for LLM assistance (stub for now)
4. **Retry**: Re-run verification after heal attempt
5. **Update PRD**: Mark story complete, update progress.txt
6. **Commit & push**: Auto-commit changes as `ralph-loop[bot]`

## Smoke Test

Story `000` is a smoke test designed to:
- **Fail initially**: Missing `scripts/.smoke-test-fixed` file
- **Heal**: Workflow should detect failure (heal stub logs it)
- **Manual fix**: Create the marker file to simulate heal
- **Succeed**: Re-run marks story complete

### Testing the Loop

```bash
# 1. Push will trigger workflow - it WILL FAIL (expected)
git push

# 2. Manually simulate the "heal" by fixing the issue
touch scripts/.smoke-test-fixed
git add scripts/.smoke-test-fixed
git commit -m "fix: add smoke test marker (simulating LLM heal)"
git push

# 3. Workflow runs again - should PASS and mark story 000 complete
```

## Expected Behavior

- ✅ Workflow detects pending story 000
- ❌ First run fails (missing marker file)
- 🔧 Heal step logs failure details
- 🔁 Manual intervention adds fix
- ✅ Second run succeeds
- 📝 PRD updated: story 000 status → "completed"
- 📖 progress.txt appended with completion note

## Future Enhancements

- Replace heal stub with actual LLM API call
- Auto-create PR with fix suggestions
- Parse error logs and provide context to LLM
- Implement retry limits and backoff

## Security Notes

**Command Execution**: The workflow uses `eval` to run verification commands from the PRD. This is intentional but poses security risks:
- ✅ Safe: PRD is repo-controlled (trusted source)
- ❌ Risk: Don't use with untrusted external input
- 🔒 Mitigation: Commands come only from committed PRD file

**Script Permissions**: Verification scripts (like `smoke-test.sh`) should have execute permissions set via `chmod +x`. The workflow explicitly calls them with `bash` to work even without +x.
