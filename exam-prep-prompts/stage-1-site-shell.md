# Stage 1 — Site shell, theme, blog, legal pages

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
