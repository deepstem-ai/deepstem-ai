# Stage 3 — Guest practice mode (no login)

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
