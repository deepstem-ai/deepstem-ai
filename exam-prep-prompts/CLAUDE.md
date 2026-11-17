# CLAUDE.md — Test-prep platform (working name: BRAND)

Read this file fully before every task. It is the source of truth for scope, rules and architecture.
Owner is a solo non-engineer-first founder (teacher, knows basic dev). Explain decisions briefly, work in small steps, commit often.

## 1. What we are building
A Thai-first web platform where students learn and take practice tests for the digital SAT (Math + Reading & Writing).
- Free guest use without registering (limited), free account (more), paid premium (full).
- Adaptive practice and adaptive full mock tests.
- Learners can pick specific topics for learning and for tests.
- Thai explanations, colorful original design, original questions only.
- AI does content drafting, support and ops; humans verify correctness.

## 2. HARD LEGAL RULES (never break; ask the owner if unsure)
These come from College Board's published guidelines. The owner will have a lawyer review before launch.
1. Never put "SAT" in the domain, product name, company name, tagline, social page names, meta tags, or ads. The brand name must not contain it.
2. "SAT" may appear only as a truthful reference in body text, e.g. "practice for the SAT® exam", and every page that shows it must have a visible footer disclaimer: "SAT® is a registered trademark of College Board. This site is not affiliated with or endorsed by College Board." (Thai + English). A disclaimer only in Terms is NOT enough.
3. Never use the SAT or College Board logo, or imitate their branding. Our look is our own: colorful, friendly, original.
4. NEVER copy, paste, paraphrase-closely, or feed official questions (Bluebook, Khan Academy official SAT, College Board Question Bank, Official Study Guide) into any AI tool, prompt, file, or database. Never use them to train/evaluate/few-shot anything.
5. Questions are written from published SKILL DESCRIPTIONS only (domain + skill names, section 6), as brand-new problems with new numbers, contexts and passages. Reading passages are written fresh by AI; never copy from books, articles, tests or websites.
6. Mock test = copy the MECHANICS (structure, timing, module routing, tools like timer/flag), not the content or the exact look.
7. Scores shown to users are ESTIMATES: always label "ประมาณการจากแพลตฟอร์มนี้ ไม่ใช่คะแนนทางการ / Estimate only, not an official score". Show a range, never a single number. No score guarantees.
8. Do not build a "science score" feature (official details are unconfirmed).
9. Personal data: Thailand PDPA. Most users are under 20. Guests: no personal data stored beyond an anonymous session. Registered users under 20: collect minimum data; premium purchase requires a parent/guardian confirmation step. Never sell data. Keep a privacy policy page in Thai + English.

## 3. Test mechanics we mirror (config values, NOT hardcoded)
Store in a `test_config` table / config file so they can be changed without code edits. Verify against a real Bluebook practice test before launch.
- 2 sections: Reading & Writing, then Math. Each section = 2 modules. Module 2 is easier/harder based on module 1. Module 1 is the same mix for everyone. No going back to module 1.
- Per module: ~27 questions R&W (32 min), ~22 questions Math (35 min); 2 per module are unscored pretest items. 10-minute break between sections.
- Scores: section 200–800, total 400–1600 (estimates for us).
- Scoring in the real test uses Item Response Theory (which questions you got right matters, not just how many).
- Tools in test UI: countdown timer, question navigator, flag for review, answer eliminator, calculator (see section 9).

## 4. Tiers (hypothesis, tune by experiment — keep limits in config, not code)
| Feature | Guest | Free account | Premium |
|---|---|---|---|
| Blog + study tips | yes | yes | yes |
| Try questions | few per day, no saved history | larger free bank | full bank |
| Adaptive mini-test | 1 | limited, history saved | unlimited |
| Explanations (Thai) | short | partial | full for every question |
| Full mock test (2 sections) | no | 1 per month | many, timed |
| Topic practice | no | limited topics | all topics |
| Custom test builder (pick topics/count/time/difficulty) | no | no | yes |
| Weakness report + review plan | no | short summary | detailed |
| Mistake notebook / spaced review | no | limited | full |
Prices are NOT decided. Build a waitlist/price-test page first; payment later.

## 5. Topic taxonomy (from College Board's published content domains)
Topics are selectable for learning and testing. Two levels: domain -> skill. Every question must carry domain + skill + difficulty tag.
- Math: Algebra; Advanced Math; Problem-Solving and Data Analysis; Geometry and Trigonometry.
- Reading & Writing: Information and Ideas; Craft and Structure; Standard English Conventions; Expression of Ideas.
Skill lists live in `content/taxonomy.json` (we create it; owner reviews).

## 6. Content rules (question bank)
Every question record: id, subject (math|english), domain, skill, difficulty_est (1–4), stem, passage?, choices[4] or numeric answer, correct, explanation_th (why correct), distractor_notes_th (why each wrong option is wrong), source ('ai-original'), status, created_at, verified_by.
Status flow: draft -> auto_verified -> ai_reviewed -> human_approved -> published. Only `published` is ever shown to users.
Gates:
- Math: recompute the answer in code (sympy) from a structured solution; mismatch = reject. Distractors must be wrong by construction. Numeric/grid-in answers accept equivalent forms.
- English: a second independent model solves the item WITHOUT seeing the key. Disagreement = reject. Each wrong option must have a stated reason it is wrong. Reject items where two options are defensible.
- Human: owner spot-checks a fixed percentage of every batch; owner approves all English items at first.
- Users can report errors; reported items are auto-hidden after N reports and queued for the owner.
Originality check: reject items that look like known official questions; log a similarity score.

## 7. Adaptive design
- Test mode (mock): module 1 = fixed blueprint (mix of easy/medium/hard by domain percentages from config). After module 1, compute ability estimate on the server; route to easier or harder module 2. Never let the browser decide.
- Practice mode: pick next item by user's per-skill mastery (table `skill_mastery`), target ~70% expected success; harder after correct streaks, easier + hint/explanation after misses.
- Cold start (no response data yet): difficulty comes from AI estimate + owner tag; use simple rules (percent correct in module 1 vs threshold). When an item has enough responses, recalibrate (start with 1PL/Rasch; move to 2PL only with larger samples). Tools to evaluate: Python `catsim` (simulation), R `irtQ` (calibration) — use only in offline scripts.
- Put 2 unscored "pretest" items in every module from OUR new items to gather calibration data.
- Item exposure control: avoid showing the same item twice in a user's history unless in review mode.
- Show topic-only test results as "accuracy in topic", never as a SAT score.

## 8. Architecture
- Frontend: Next.js (App Router, TypeScript) + Tailwind. Mobile-first. Thai default language, English toggle. Blog in MDX files in repo.
- Backend/data: Supabase (Postgres, Auth, Row Level Security). Anonymous sign-in for guests; merge history into the account on register.
- Server decides: all answer checking and next-module routing run server-side (Next.js route handlers / server actions or Supabase edge functions). The browser must NEVER receive correct answers or explanations before the answer is submitted (explanations only after submit, and only for tiers allowed).
- Background jobs (Python, run locally or scheduled): difficulty recalibration, question generation + verification pipeline. Outputs written to DB with status flags.
- Payments (later phase): Stripe in Thailand: cards (can auto-renew) + PromptPay (one-time payments only; Stripe docs say PromptPay does not support automatic recurring). So sell fixed-term passes (1/3/6 months) via PromptPay plus optional card auto-renew. Webhooks verify and set `subscriptions`.
- Email: transactional only. LINE OA planned for notifications later.
- Analytics: privacy-friendly, no ad trackers, no selling data.

## 9. Calculator / graphing
The real test embeds Desmos. Desmos terms restrict non-commercial use; commercial embedding needs an API key/license. Until the owner gets a license: DO NOT embed Desmos. Build a simple in-house scientific calculator component behind an interface `CalculatorTool` so Desmos or our own graphing tool can be swapped in later.

## 10. Data model (starting point, Postgres)
profiles(id, display_name, birth_year, guardian_confirmed_at, tier, created_at)
questions(id, subject, domain, skill, difficulty_est, difficulty_irt, stem, passage, choices jsonb, answer jsonb, explanation_th, distractor_notes_th jsonb, status, similarity_score, source, created_at)
test_forms(id, kind[mock|practice|topic|custom], config jsonb)
attempts(id, user_id nullable, anon_session_id nullable, form_id, started_at, finished_at, estimate_low, estimate_high)
responses(id, attempt_id, question_id, module_no, answer jsonb, is_correct, time_ms, flagged)
skill_mastery(user_id, skill, mastery_p, n_attempts, updated_at)
subscriptions(id, user_id, plan, provider, provider_ref, status, current_period_end)
error_reports(id, question_id, user_id, note, created_at)
test_config(key, value jsonb)
Row Level Security ON for every table. Users can read only their own attempts/responses/mastery. `questions.answer` and `explanation_th` are never selectable by the client role; only server code reads them.

## 11. Design system
Colorful, friendly, original. Rounded cards, playful accent colors per subject (e.g. Math = one hue, English = another), clear typography with a Thai-capable font (e.g. Noto Sans Thai / Anuphan). High contrast, accessible (WCAG AA), large tap targets. Test screen: split pane (passage | question), top timer, bottom navigator. Do not imitate Bluebook visuals or College Board branding.

## 12. Working agreements for Claude Code
- Small steps. Before big changes: state the plan in 5 lines and wait for "go".
- Write tests for scoring/routing/verification code (these are the risky parts). Run them before saying done.
- Never commit secrets. Use `.env.local` and `.env.example`.
- After each task: update `docs/DECISIONS.md` (what/why), keep `README.md` runnable.
- If a task would touch a rule in section 2, STOP and ask the owner.
- Use git: one branch per stage, conventional commit messages.
- When unsure about an official fact (test format, scoring), do not invent: add a TODO in `docs/OPEN_QUESTIONS.md`.

## 13. Open questions (owner to resolve before public launch)
1. Lawyer review of brand name, footer disclaimer, terms, privacy policy (PDPA), refund policy.
2. Desmos license (or build own graphing tool).
3. Price test results (Thai market prices not yet known).
4. Verify test-format numbers against a real Bluebook practice test.
5. Science-score reporting from 2027: unconfirmed; check College Board before spring 2027.
