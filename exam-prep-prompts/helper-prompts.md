# Helper prompts

## Review before merging

```
Review the last stage against CLAUDE.md section 2 (legal rules), section 8 (server decides answers), and section 10 (RLS). List violations or risks. Do not change code; give me a checklist.
```

## When stuck

```
Stop. Summarize in 5 lines what you changed, what is failing, and the 2 simplest options to fix. Do not edit anything until I choose.
```

## Weekly content batch (after Stage 2)

```
Generate [20] draft questions for [Math / Algebra / linear equations in one variable], difficulty [2]. Use only the skill description in content/taxonomy/taxonomy.json. Run verify_math.py and dedupe.py, then show me a table of accepted vs rejected with reasons. Do not publish anything.
```
