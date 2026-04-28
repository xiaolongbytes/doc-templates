---
name: doc-navigator
description: "Search the repository's documentation and answer questions from it. Use whenever a user asks 'what is', 'how do I', 'where is', 'who is responsible for', or any question that may be answered by a README, wiki page, or docs folder in this repository. Triggers on: questions, how-to requests, concept lookups, process questions, onboarding queries, runbook searches, architecture questions, setup instructions, troubleshooting steps."
---

# Doc Navigator

## When to Use

Load this skill for any user question that may be answered by documentation in the current repository. The docs are the authoritative source — do not guess or rely on general knowledge when project-specific documentation may exist.

Typical triggers:
- "How do I…", "What is…", "Where is…", "Who owns…"
- Questions about setup, configuration, architecture, or processes
- Requests for runbook steps, onboarding instructions, or access procedures
- Troubleshooting questions that may have documented solutions
- Any factual question about the project, team, or codebase

---

## Documentation Discovery

Do **not** assume a fixed structure. Repositories organise documentation differently.

**At the start of every search, discover docs dynamically:**

1. Use `file_search` with the glob pattern `**/*.md` to list all Markdown files in the workspace.
2. Exclude files under `.github/` — those are agent config files, not documentation.
3. Prioritise files in common documentation locations in this order:
   - Repository root: `README.md`, `CONTRIBUTING.md`, `CHANGELOG.md`
   - `docs/**`
   - `wiki/**`
   - `doc/**`
   - Any remaining `.md` files at the repo root or subdirectories
4. Treat every discovered file as a searchable documentation page.

---

## Procedure

Follow these steps **in order** for every user question.

### Step 1 — Search the Docs

1. Use `semantic_search` with the user's full question as the query.
2. Follow up with `grep_search` using key terms extracted from the question to catch exact matches.
3. If results are weak or ambiguous, `read_file` on the most likely candidate file(s) to verify.

### Step 2 — Answer Found

If a matching answer is found:

- Summarize the answer clearly and concisely.
- Cite the exact source using this format:

  > **Source:** [FileName.md](FileName.md) › *Section Heading* › *Sub-heading (if applicable)*

- Quote the relevant passage if it adds precision.
- If multiple files contain relevant information, cite all of them.

### Step 3 — Answer Not Found

If no relevant answer exists after Steps 1–2:

1. **Log the unanswered question.** Append to the unanswered questions log for this repository (see Logging below).

2. **Respond to the user:**

   > I searched the repository documentation but could not find an answer to your question. I've logged it so the team can consider adding it to the docs.
   >
   > Would you like me to search the internet for an answer?

3. If the user says **yes**, use `fetch_webpage` or a web search. Clearly mark any externally sourced information as coming from outside the repository docs.

---

## Output Format

Always structure your answer as:

1. **Direct answer** — 1–3 sentences or a short list
2. **Details** — additional context from the docs if useful
3. **Source citation** — file + heading + sub-heading

If the answer spans multiple files, cite each one separately.

---

## Logging Unanswered Questions

Determine the log file path at runtime:

- **If `.github/skills/doc-navigator/` exists in the workspace:** log to `.github/skills/doc-navigator/unanswered-questions.md`
- **Otherwise:** log to `unanswered-questions.md` at the repository root

Append using this format:

```
- **Date:** YYYY-MM-DD | **Question:** <exact user question> | **Suggested Page:** <most appropriate existing or new doc page>
```

For **Suggested Page**: infer the best-fit existing page based on the question's topic. If no existing page fits, suggest a descriptive new page name (e.g. `docs/deployment-guide.md`).

**If the log file does not exist**, create it with this header first:

```markdown
# Unanswered Questions Log

Questions that could not be answered from repository documentation.
Use this log to identify gaps and improve the docs.

---
```
