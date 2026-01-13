# AGENTS.md

Ralph Loop: **pick ONE story → implement → verify → commit → repeat**.

---

## STOP CONDITION

All stories in `scripts/ralph/prd.json` have `"status": "completed"`.

---

## ABSOLUTE RULES

1. **PRD only**: Work from `scripts/ralph/prd.json`
2. **One story per iteration**: Pick lowest priority pending story
3. **Verify before commit**: Run verification command, only commit green
4. **Memory is files**: git commits, prd.json, progress.txt

---

## LOCAL DEV

```bash
# Install dependencies
{{package_manager}} install

# Run dev server (if applicable)
{{package_manager}} run dev

# Run tests
{{package_manager}} test

# Type check
{{package_manager}} run tsc
```

---

## VERIFICATION CHECKLIST

Before committing:
- [ ] Tests pass
- [ ] Types pass
- [ ] Story acceptance criteria met
- [ ] prd.json status updated
- [ ] progress.txt updated with learnings

---

## ITERATION LOOP

1. Read `scripts/ralph/prd.json`
2. Pick highest priority `"status": "pending"` story
3. Implement ONLY that story
4. Run verification
5. Commit: `feat|fix|chore: [ID] - [Title]`
6. Set story `"status": "completed"`
7. Append learnings to `scripts/ralph/progress.txt`
8. Repeat
