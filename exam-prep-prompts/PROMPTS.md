# PROMPTS.md — Claude Code starter prompts (Stages 0–4)

How to use:
1. Create an empty folder, run `git init`, put `CLAUDE.md` (from this pack) in the root, open Claude Code there.
2. Paste ONE prompt at a time, in order. Wait until it finishes, run what it tells you, then `git commit`.
3. If the result is wrong, say "fix: <what you saw>" — do not paste the next stage yet.
4. Replace anything in [BRACKETS].

Stage map: 0 setup -> 1 site shell + blog + legal -> 2 database + question pipeline -> 3 practice mode with guest -> 4 accounts + adaptive practice + topic picker. (Mock test and payments come after Stage 4.)

---

## Stage 0 — Project setup
```
Read CLAUDE.md fully. Then set up the project:
1. Create a Next.js (App Router, TypeScript, Tailwind) app in this folder. Use pnpm.
2. Add ESLint, Prettier, Vitest. Add scripts: dev, build, lint, test.
3. Create folders: content/blog, content/taxonomy, scripts (for Python later), docs.
4. Create docs/DECISIONS.md and docs/OPEN_QUESTIONS.md (seed OPEN_QUESTIONS with section 13 of CLAUDE.md).
5. Create .env.example (no secrets) and make sure .env.local is gitignored.
6. Write a README.md with exact commands to run the project.
Constraints: follow CLAUDE.md section 2 (no "SAT" in package name, app title, or metadata; working name is [BRAND]).
Before coding, show me a 5-line plan. After finishing, run lint + build + test and show results.
```

## Stage 1 — Site shell, theme, blog, legal pages
```
Read CLAUDE.md. Build the public site shell. Thai default, English toggle.
1. Design tokens and theme: colorful, friendly, original (section 11). Thai font via next/font. Mobile-first. Light mode only for now.
2. Layout with header (Home, Learn, Practice, Blog, Pricing placeholder, Login placeholder) and footer.
3. FOOTER must include the bilingual trademark disclaimer from CLAUDE.md section 2.2, visible on every page.
4. Pages: Home (hero, what you get free vs premium, CTA "try without signing up"), Blog index + post page (MDX files in content/blog, with frontmatter title/date/tags), Pricing placeholder with a waitlist email form (store emails in a local JSON for now; no third-party), Privacy (PDPA placeholder), Terms placeholder, About.
5. Write 3 sample blog posts in Thai about study planning and how adaptive tests work in plain language. IMPORTANT: no copied exam content; do not use the word SAT in titles or URLs; if needed, mention it only in body as a truthful reference.
6. SEO basics: sitemap, robots, OpenGraph, per-page metadata (never "SAT" in metadata).
7. Add a unit test that fails the build if any page title/meta/domain config contains the string "SAT" (case-insensitive) — implement as a script scanning built metadata.
Show the plan first. Run lint, build, tests at the end.
```

## Stage 2 — Database and question pipeline (offline, no UI yet)
```
Read CLAUDE.md sections 5, 6, 10. Set up Supabase locally with the Supabase CLI (supabase init, local stack).
1. Write SQL migrations for the data model in section 10 with Row Level Security on every table. The client role must NOT be able to select questions.answer, explanation_th, distractor_notes_th. Create a view or RPC that returns only safe fields for published questions.
2. Create content/taxonomy/taxonomy.json with the 4 Math domains and 4 R&W domains and a first list of skills per domain (propose them; I will review). Add a seed script.
3. Create scripts/pipeline/ (Python 3.11):
   - generate.py: takes domain, skill, difficulty, count, and writes DRAFT questions as JSON using a prompt template I can edit in scripts/pipeline/prompts/. The prompt must describe the SKILL ONLY and must never include or mention any official question text.
   - verify_math.py: for math items, each draft includes a structured solution (python/sympy expression). Recompute the answer; reject mismatches; check distractors are not equal to the correct answer.
   - verify_english.py: calls a second, independent model to solve WITHOUT the key; reject disagreement; require a one-line reason why each wrong option is wrong.
   - dedupe.py: simple similarity check (embedding or n-gram) between new items and existing items; write similarity_score.
   - import.py: loads items into the DB with status flags (draft -> auto_verified -> ai_reviewed).
4. A tiny CLI review tool (scripts/review.py) listing items awaiting my approval, showing stem/choices/explanations, with keys: approve, reject, edit.
5. pytest tests for verify_math.py with 5 deliberately wrong items that must be rejected.
Rules: API keys only from env vars. Do not call any model until I confirm. First show me the plan and the generation prompt template text for approval.
```

## Stage 3 — Guest practice mode (no login)
```
Read CLAUDE.md sections 4, 7, 8. Build the first learner feature: guest practice.
1. Anonymous session: use Supabase anonymous sign-in on first visit (no personal data). Show a small note "ไม่ต้องสมัคร ลองได้เลย".
2. Practice page: choose subject (Math / Reading & Writing), then domain (skill picker comes in Stage 4). Serve ONE question at a time from published questions.
3. Server-side answer checking: browser sends only the chosen option; the server returns correct/incorrect and a SHORT Thai explanation. The browser never receives the answer or explanation before submitting. Write a test that proves the client cannot read answers directly from the API/database.
4. Guest limits from test_config (e.g. questions per day) with a friendly "สมัครฟรีเพื่อบันทึกประวัติ" prompt when reached. Limits must be config, not code.
5. Simple in-house CalculatorTool component for math (interface so Desmos can replace it later).
6. Report-error button on each question writing to error_reports.
7. Question UI: split pane for passages, mobile-friendly, keyboard navigation, option eliminator.
Seed with 20 published sample questions (clearly flagged "demo") so I can try it. Show plan first; run all tests at the end.
```

## Stage 4 — Accounts, topic picker, adaptive practice, review
```
Read CLAUDE.md sections 4, 5, 7, 10. Add accounts and the adaptive/topic features.
1. Auth: email magic link (Supabase Auth). Registration asks only for email, display name, birth year. If birth year makes user under 20, show a guardian-consent notice (record guardian_confirmed_at only when a guardian email is confirmed) before any paid feature (payments come later). Merge the anonymous guest history into the new account.
2. Tiers: guest / free / premium read from profiles.tier; feature limits come from test_config. Premium can be set manually by me in the DB for now.
3. TOPIC PICKER: learners choose subject -> domain -> skill for learning and for practice. Each skill page shows a short Thai summary, then "start practice". Premium also gets a "custom test builder": choose topics, number of questions, timed/untimed, fixed or adaptive difficulty, only-wrong-before / only-new.
4. ADAPTIVE PRACTICE: implement next-question selection per skill using skill_mastery (target ~70% expected success). Put the selection logic in a pure, well-tested function (lib/adaptive/selectNext.ts) with unit tests and a simulation script that runs 1,000 fake students and shows that harder items are given as mastery rises. No external services.
5. Mastery update after every response (simple Elo/Rasch-style update in a pure function, with tests).
6. Dashboard: accuracy by domain and skill, 3 weakest skills with "practice this next", mistake notebook with "retry later" (simple spaced review: 1 day, 3 days, 7 days).
7. Topic-only results must be labelled "accuracy in this topic", never as a SAT score. No score estimate yet (mock test comes in the next stage).
Show the plan first. Then implement in this order and commit after each: auth -> tiers/limits -> topic picker -> adaptive selection + tests -> dashboard. Run lint, build, tests at the end.
```

---

## After Stage 4 (do NOT start yet)
- Stage 5: full mock test with 2-module routing, timer, break, and score RANGE estimate (cold-start rules first).
- Stage 6: waitlist price test -> Stripe (cards + PromptPay passes) -> parent confirmation for under-20 buyers.
- Stage 7: recalibration job (IRT), pretest items, error-report workflow, LINE OA.
- Before public launch: lawyer review, Desmos license or own graph tool, Bluebook comparison of test numbers.

## Helper prompts

Review before merging:
```
Review the last stage against CLAUDE.md section 2 (legal rules), section 8 (server decides answers), and section 10 (RLS). List violations or risks. Do not change code; give me a checklist.
```

When stuck:
```
Stop. Summarize in 5 lines what you changed, what is failing, and the 2 simplest options to fix. Do not edit anything until I choose.
```

Weekly content batch (after Stage 2):
```
Generate [20] draft questions for [Math / Algebra / linear equations in one variable], difficulty [2]. Use only the skill description in content/taxonomy/taxonomy.json. Run verify_math.py and dedupe.py, then show me a table of accepted vs rejected with reasons. Do not publish anything.
```
