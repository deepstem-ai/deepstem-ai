# Stage 2 — Database and question pipeline (offline, no UI yet)

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
