# Stage 4 — Accounts, topic picker, adaptive practice, review

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
