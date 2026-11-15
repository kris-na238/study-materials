# Study Materials — Interview Prep

A personal, self-contained knowledge base for senior software engineering interview prep. Each guide is written to stand on its own — the goal is to revise from these docs without needing to search blogs, YouTube, or official docs.

## Contents

| Folder | Topic | Guides |
|---|---|---|
| System Design | 20 core system design topics with 10+ interview questions each | [System Design Guide](System%20Design/System_Design_Interview_Guide.md) · [1M Requests/sec](System%20Design/Lets_handle_1_miilion_per_request.md) · [URL Shortener](System%20Design/Products/Url_Shortner.md) · [Links](System%20Design/links.md) |
| DSA | Complete DSA guide: 22 pattern chapters in 5 parts, 165 problems covering all of the Blind 75 (problem, approach, optimal JS solution, complexity, alternatives), plus a one-page revision sheet, pattern-recognition cheat sheets and a JS toolkit | [DSA Complete Guide](DSA/DSA_Complete_Guide.md) · [Links](DSA/links.md) |
| Backend | Node.js, NestJS, Python, and general programming language fundamentals | [Node.js](Backend/NodeJs/Nodejs-Complete-Guide.md) · [NestJS](Backend/NodeJs/NestJS_Senior_Interview_Questions.md) · [Python](Backend/Python/Python_Interview_Guide.md) · [Language Basics](Backend/Programming_language_Basics_.md) · [Links](Backend/links.md) |
| Frontend | React senior-level interview guide | [React Complete Guide](Frontend/ReactJS/React-Complete-Topic-Guide.md) · [Links](Frontend/links.md) |
| DB | DB fundamentals, SQL (MySQL/PostgreSQL), MongoDB, Redis, query practice, scenarios | [Cheatsheet](DB/00_Revision_Cheatsheet.md) · [Fundamentals](DB/01_DB_Fundamentals.md) · [SQL](DB/02_SQL_Databases.md) · [MongoDB](DB/03_NoSQL_MongoDB.md) · [Redis](DB/04_Redis.md) · [Query Practice](DB/05_Query_Practice.md) · [Scenarios](DB/06_Scenario_Questions.md) · [Links](DB/links.md) |
| AWS & DevOps | AWS, AI on AWS, and DevOps interview prep | [AWS & AI on AWS](AWS%20%26%20DevOps/AWS_and_AI_on_AWS_Interview_Guide.md) · [DevOps](AWS%20%26%20DevOps/DevOps_Interview_Guide.md) |
| AI | AI/LLM-related notes | [Links](AI/links.md) |
| Spritual | Personal notes, unrelated to interview prep | [Realization](Spritual/realization.md) · [Links](Spritual/links.md) |

## Other files

- **[Interview Requirement Gathering](Interview%20Requirement%20Gathering.md)**
- **[Operation_Architect.md](Utility/Operation_Architect.md)** — Interview syllabus/schedule and a two-level self-introduction script (experience summary + project deep-dives).
- **[Study_Doc_Prompt_Template.md](Utility/Study_Doc_Prompt_Template.md)** — Reusable prompt template for generating a new subject-wise study guide in the same format as the existing ones. Swap in `{{SUBJECT}}` to produce a new doc.
- **[InterviwerLLD.drawio.svg](Utility/InterviwerLLD.drawio.svg)** — Low-level design diagram (draw.io).
- **[Utility links](Utility/links.md)**

## How to use this repo

1. Pick a subject folder based on the interview round you're prepping for.
2. Read the guide top-to-bottom — each is structured with a table of contents and ordered roughly by how often topics come up in real interviews.
3. To add a new subject, copy the prompt in `Study_Doc_Prompt_Template.md`, replace `{{SUBJECT}}`, and run it to generate a matching guide.

