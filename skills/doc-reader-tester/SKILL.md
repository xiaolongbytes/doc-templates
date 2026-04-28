---
name: doc-reader-tester
description: "Test a completed document from the perspective of a fresh reader with no prior context. Use when validating that a document is clear, unambiguous, and self-contained before sharing. Automatically invoked at Stage 3 of the doc-coauthoring workflow. Also invoke directly when a user asks to test, review, validate, or proofread an existing document for reader clarity."
inspired_by: https://github.com/anthropics/skills/blob/main/skills/doc-coauthoring/SKILL.md
---

# Doc Reader Tester

## When to Use

This skill is invoked in two scenarios:

1. **From `doc-coauthoring`** — automatically at Stage 3, after all sections are drafted and refined.
2. **Standalone** — when a user asks to test, validate, or proofread an existing document for reader clarity.

The goal is to simulate what a fresh reader experiences: no shared context, no assumed knowledge, only what is on the page.

---

## Input

Before starting, confirm what document to test. Accept any of the following:

- A file path or artifact link (read it with `read_file`)
- Pasted document content directly in the chat
- A shared document link (fetch with the appropriate integration if available)

---

## Operating Rules

1. State `Executing doc-reader-tester` before doing any other work.

## Procedure

### Step 1 — Predict Reader Questions

Announce: *"Predicting questions a reader would realistically ask about this document."*

Generate 5–10 questions that represent realistic reader discovery attempts. Think about:
- What someone would type into a search bar or ask an AI to find this doc
- What a first-time reader would ask after reading the first paragraph
- What a skeptical stakeholder would challenge

Number the questions clearly.

---

### Step 2 — Test with a Fresh Context

**If sub-agents are available (e.g., agent-mode or specialized sub-agents):**

Announce: *"Testing each question with a fresh assistant instance that has only the document — no context from this conversation."*

For each question, invoke a sub-agent with:
- The full document content only (no conversation history)
- The question

Collect the sub-agent's answer. Note where it:
- Answered correctly and confidently
- Hedged, guessed, or gave a partial answer
- Got it wrong or expressed confusion

Summarize results in a table:

| # | Question | Result | Issue |
|---|---|---|---|
| 1 | ... | ✅ Correct / ⚠️ Partial / ❌ Wrong | ... |

---

**If sub-agents are NOT available (e.g., no external assistant web interface):**

Announce: *"Sub-agents aren't available here, so the user will need to run the reader test manually. Here's how:"*

Provide this exact instruction to the user:

> 1. Open a **new** conversation in your assistant (for example, GitHub Copilot Chat) with no prior context from this chat
> 2. Paste the full document content, or share the link if connectors are enabled
> 3. Ask the assistant each of the following questions one at a time:

List the predicted questions.

> For each question, ask the assistant to also tell you:
> - Whether anything in the document was ambiguous or unclear
> - What background knowledge the document assumes
>
> Come back here and share what the assistant got wrong or struggled with.

Wait for the user to return with results before proceeding to Step 3.

---

### Step 3 — Run Blind Checks

**If sub-agents are available:**

Announce: *"Running additional blind checks on the document."*

Invoke a sub-agent with only the document content and these prompts (one at a time or combined):
- *"What in this document is ambiguous or unclear to a first-time reader?"*
- *"What background knowledge or context does this document assume the reader already has?"*
- *"Are there any internal contradictions, inconsistencies, or unsupported claims?"*

Summarize findings.

- **If sub-agents are NOT available:**

Ask the user to also ask the assistant:
- *"What in this doc might be ambiguous or unclear?"*
- *"What knowledge does this doc assume readers already have?"*
- *"Are there any internal contradictions or inconsistencies?"*

---

### Step 4 — Report and Fix

Compile all issues found across Steps 2 and 3. Group them by type:

- **Clarity gaps** — things a reader found confusing or ambiguous
- **Assumed knowledge** — context that isn't explained in the doc
- **Contradictions** — internal inconsistencies

If issues exist:

Report findings clearly, then announce intention to fix each gap. Work through the fixes section by section using `str_replace` (never reprint the whole document). After fixes, confirm which issues were addressed.

If no issues:

Announce: *"Reader testing passed. The document is clear and self-contained."*

---

### Exit Condition

- Testing is complete when:
- The assistant answers the predicted questions correctly and confidently, AND
- The blind checks surface no new ambiguities, assumed knowledge gaps, or contradictions

Once this bar is met, hand back to `doc-coauthoring` for Final Review (if invoked from there), or tell the user the document is ready to share (if invoked standalone).
