# Reusable Prompt — Subject-wise Interview Study Doc Generator

Copy the prompt below, replace `{{SUBJECT}}` (and the optional context line) with the
new subject, and run it. Everything else stays the same across subjects.

---

## PROMPT

```
You are helping me build a permanent interview-revision reference document for the subject: {{SUBJECT}}.

CONTEXT ABOUT ME
- I am a senior software engineer with 7 years of professional experience.
- This doc replaces me having to re-search blogs/YouTube/docs before interviews — it must
  be self-sufficient. Assume I won't open any other source while revising from it.
- Write answers at a senior/staff-interview depth: assume I already know basic programming,
  but explain {{SUBJECT}}-specific concepts, internals, and jargon from scratch — don't assume
  prior familiarity with THIS subject's terminology, only with general engineering.

TASK
Produce a single, well-structured Markdown document covering {{SUBJECT}} for interview prep.

STRUCTURE
1. Start with a short "How to use this doc" note and a Table of Contents linking to all 25 topics.
2. Identify the TOP 25 most frequently asked interview topics for {{SUBJECT}}, ordered
   roughly by how often they come up in real interviews (most common first). Pick topics
   the way a hiring panel at a product company would probe — fundamentals, internals,
   trade-offs, common pitfalls, and at least a few real-world/system-design-flavored topics
   relevant to {{SUBJECT}}.
3. For EACH of the 25 topics, include:
   - A 2-4 line "concept primer": what it is, why it exists/what problem it solves, and
     where it fits in the bigger picture. Define any jargon inline the first time it's used.
   - Exactly 10 interview questions on that topic, ordered from foundational to advanced/tricky.
   - For each question, give an "optimized answer" that is:
     - Structured as bullet points or short paragraphs (interview-spoken-style, not academic).
     - Technically precise, using correct terminology (bold key terms once, e.g. **idempotency**).
     - Backed by a short, concrete code snippet or real-world example wherever relevant —
       something I could literally say or draw on a whiteboard.
     - Includes common follow-up angles or "gotchas" interviewers probe next, as a short
       "Interviewer may follow up with:" line where applicable.
     - Long enough to be complete, short enough to revise in under a minute per question
       (roughly 80-150 words, more only if the concept genuinely needs it).
   - A short "Red flags to avoid saying" bullet list (2-3 items) of common wrong/incomplete
     answers candidates give for this topic.
4. End the document with:
   - A "Rapid-fire recap" section: one-line summaries of all 25 topics, for last-minute
     revision before walking into the interview.
   - A "Commonly confused pairs/terms" table specific to {{SUBJECT}} (e.g. concept A vs
     concept B), since these are frequent interview traps.

FORMATTING RULES
- Output valid, clean Markdown only. Use `##` for each of the 25 topics and `###` for
  "Q1, Q2 ... Q10" within each.
- Use fenced code blocks with language hints for all code/examples.
- Use tables where comparison is clearer than prose.
- No filler/marketing language, no repeated disclaimers, no "as an AI" text.
- Keep a consistent tone throughout — direct, technical, written for a peer engineer.

Do not stop early or truncate — generate all 25 topics with all 10 questions each in full
before ending your response. If the response must be split across multiple messages due to
length, stop at a clean topic boundary and continue seamlessly when I say "continue".
```

---

## How to use this
1. Replace `{{SUBJECT}}` with e.g. `Kubernetes`, `Kafka`, `System Design - Caching`,
   `SQL & Database Internals`, `Java Concurrency`, `GraphQL`, etc.
2. If the subject is broad (e.g. "System Design"), consider narrowing it per run
   (e.g. "System Design - Scalability & Caching", "System Design - Messaging Queues")
   so the 25 topics stay focused rather than shallow.
3. Save the output as `{{Subject}}_Interview_Guide.md` in this repo, matching the existing
   naming pattern (`NestJS_Senior_Interview_Questions.md`, `React-senior-interview-guide.md`, etc.).
