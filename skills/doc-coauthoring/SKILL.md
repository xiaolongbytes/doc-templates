---
name: doc-coauthoring
description: Guide users through a structured workflow for co-authoring documentation. Use when user wants to write documentation, proposals, technical specs, decision docs, or similar structured content. Default mode is conservative (only uses user-provided content; asks before inferring). Can opt into faster mode. This workflow helps users efficiently transfer context, refine content through iteration, and verify the doc works for readers. Trigger when user mentions writing docs, creating proposals, drafting specs, or similar documentation tasks.
inspired_by: https://github.com/anthropics/skills/blob/main/skills/doc-coauthoring/SKILL.md
---

# Doc Co-Authoring Workflow

This skill provides a structured workflow for guiding users through collaborative document creation. Act as an active guide, walking users through three stages: Context Gathering, Refinement & Structure, and Reader Testing.

## When to Offer This Workflow

**Trigger conditions:**
- User mentions writing documentation: "write a doc", "draft a proposal", "create a spec", "write up"
- User mentions specific doc types: "PRD", "design doc", "decision doc", "RFC"
- User seems to be starting a substantial writing task

## Modes

This skill supports two modes:

### Conservative (Default)
**Best for:** First-time writers, sensitive domains (runbooks, compliance docs), team learning
- Only uses information explicitly provided by the user
- Asks before adding any content not grounded in user notes
- Marks all gaps and inferred content as `{{TODO: [specific detail needed]}}`
- Automatically invokes `doc-reader-tester` for readability validation
- Slower but ensures accuracy and team ownership

### Standard (Opt-in)
**Best for:** Experienced writers, fast iteration, non-critical docs
- Infers context and adds helpful details from best practices
- Fills gaps proactively to speed up drafting
- Skips automatic readability validation
- Increased risk of inaccuracies, missed gaps, and less team/app specific guidance

**Initial offer:**
Offer the user a structured workflow for co-authoring the document in **conservative mode by default**. Explain the approach:

"I'll guide you through a conservative documentation workflow. I'll only use information you explicitly provide, ask before inferring anything, and mark all gaps as `{{TODO}}` items. This approach prioritizes accuracy and helps your team learn from the writing process. At the end, I'll automatically validate readability.

**Why conservative by default?** It's safer—you maintain control and ownership over what gets documented.

Would you prefer this conservative approach, or opt into a faster mode where I'm more generous with inferring details?"

Explain the three stages:

1. **Context Gathering**: User provides all relevant context while the assistant asks clarifying questions
2. **Refinement & Structure**: Iteratively build each section through brainstorming and editing (using only user-provided content in conservative mode)
3. **Reader Testing**: Test the doc with a fresh assistant instance (no context) to catch blind spots before others read it

If user declines conservative mode, ask which mode they prefer and note it for Stage 1 and 2 behavior.

If user accepts (or if they're unsure), proceed to Stage 1 in conservative mode.

## Operating Rules

1. State `Executing doc-coauthoring (conservative mode)` or `Executing doc-coauthoring (standard mode)` before doing any other work, based on the mode selected.

## Stage 1: Context Gathering

**Goal:** Close the gap between what the user knows and what the assistant knows, enabling smart guidance later.

### Initial Questions

Start by asking the user for meta-context about the document:

1. What type of document is this? (e.g., technical spec, decision doc, proposal)
2. Who's the primary audience?
3. What's the desired impact when someone reads this?
4. Is there a template or specific format to follow?
5. Any other constraints or context to know?

Inform them they can answer in shorthand or dump information however works best for them.

**If user provides a template or mentions a doc type:**
- Ask if they have a template document to share
- If they provide a link to a shared document, use the appropriate integration to fetch it
- If they provide a file, read it

**If user mentions editing an existing shared document:**
- Use the appropriate integration to read the current state
  - Check for images without alt-text
  - If images exist without alt-text, explain that when others use a text-only assistant to understand the doc, the assistant won't be able to see them. Ask if they want alt-text generated. If so, request they paste each image into chat for descriptive alt-text generation.

### Info Dumping

Once initial questions are answered, encourage the user to dump all the context they have. Request information such as:
- Background on the project/problem
- Related team discussions or shared documents
- Why alternative solutions aren't being used
- Organizational context (team dynamics, past incidents, politics)
- Timeline pressures or constraints
- Technical architecture or dependencies
- Stakeholder concerns

Advise them not to worry about organizing it - just get it all out. Offer multiple ways to provide context:
- Info dump stream-of-consciousness
- Point to team channels or threads to read
- Link to shared documents

**If integrations are available** (e.g., Slack, Teams, Google Drive, SharePoint, or other MCP servers), mention that these can be used to pull in context directly.

**If no integrations are detected:** Suggest they can enable connectors in their assistant or chat app settings (for example, GitHub Copilot Chat or other connectors) to allow pulling context from messaging apps and document storage directly.

Inform them clarifying questions will be asked once they've done their initial dump.

**During context gathering:**

- If user mentions team channels or shared documents:
  - If integrations available: Inform them the content will be read now, then use the appropriate integration
  - If integrations not available: Explain lack of access. Suggest they enable connectors in their assistant/chat app settings (for example, GitHub Copilot connectors), or paste the relevant content directly.

- If user mentions entities/projects that are unknown:
  - Ask if connected tools should be searched to learn more
  - Wait for user confirmation before searching

- As user provides context, track what's being learned and what's still unclear

**Asking clarifying questions:**

When user signals they've done their initial dump (or after substantial context provided), ask clarifying questions to ensure understanding:

Generate 5-10 numbered questions based on gaps in the context.

Inform them they can use shorthand to answer (e.g., "1: yes, 2: see #channel, 3: no because backwards compat"), link to more docs, point to channels to read, or just keep info-dumping. Whatever's most efficient for them.

**Exit condition:**
Sufficient context has been gathered when questions show understanding - when edge cases and trade-offs can be asked about without needing basics explained.

**Transition:**
Ask if there's any more context they want to provide at this stage, or if it's time to move on to drafting the document.

If user wants to add more, let them. When ready, proceed to Stage 2.

## Stage 2: Refinement & Structure

**Goal:** Build the document section by section through brainstorming, curation, and iterative refinement.

**Instructions to user:**
Explain that the document will be built section by section. For each section:
1. Clarifying questions will be asked about what to include
2. 5-20 options will be brainstormed
3. User will indicate what to keep/remove/combine
4. The section will be drafted
5. It will be refined through surgical edits

Start with whichever section has the most unknowns (usually the core decision/proposal), then work through the rest.

**Section ordering:**

If the document structure is clear:
Ask which section they'd like to start with.

Suggest starting with whichever section has the most unknowns. For decision docs, that's usually the core proposal. For specs, it's typically the technical approach. Summary sections are best left for last.

If user doesn't know what sections they need:
Based on the type of document and template, suggest 3-5 sections appropriate for the doc type.

Ask if this structure works, or if they want to adjust it.

**Once structure is agreed:**

Create the initial document structure with placeholder text for all sections.

**If access to artifacts is available:**
Use `create_file` to create an artifact. This gives both the assistant and the user a scaffold to work from.

Inform them that the initial structure with placeholders for all sections will be created.

Create artifact with all section headers and brief placeholder text like "[To be written]" or "[Content here]".

Provide the scaffold link and indicate it's time to fill in each section.

**If no access to artifacts:**
Create a markdown file in the working directory. Name it appropriately (e.g., `decision-doc.md`, `technical-spec.md`).

Inform them that the initial structure with placeholders for all sections will be created.

Create file with all section headers and placeholder text.

Confirm the filename has been created and indicate it's time to fill in each section.

**For each section:**

### Step 1: Clarifying Questions

Announce work will begin on the [SECTION NAME] section. Ask 5-10 clarifying questions about what should be included:

Generate 5-10 specific questions based on context and section purpose.

Inform them they can answer in shorthand or just indicate what's important to cover.

### Step 2: Brainstorming

For the [SECTION NAME] section, brainstorm [5-20] things that might be included, depending on the section's complexity. Look for:
- Context shared that might have been forgotten
- Angles or considerations not yet mentioned

Generate 5-20 numbered options based on section complexity. At the end, offer to brainstorm more if they want additional options.

**In conservative mode:** Mark options that are NOT directly from the user's notes with a warning prefix like `[INFERRED]`. For example:
- `1. [FROM NOTES] Process for restarting JVM`
- `2. [INFERRED] Prerequisites (not in your notes — should I ask?)`

Ask the user: "For items marked [INFERRED], should I ask you for details, skip them, or add them as `{{TODO}}`?"

### Step 3: Curation

Ask which points should be kept, removed, or combined. Request brief justifications to help learn priorities for the next sections.

Provide examples:
- "Keep 1,4,7,9"
- "Remove 3 (duplicates 1)"
- "Remove 6 (audience already knows this)"
- "Combine 11 and 12"

**If user gives freeform feedback** (e.g., "looks good" or "I like most of it but...") instead of numbered selections, extract their preferences and proceed. Parse what they want kept/removed/changed and apply it.

### Step 4: Gap Check

Based on what they've selected, ask if there's anything important missing for the [SECTION NAME] section.

### Step 5: Drafting

Use `str_replace` to replace the placeholder text for this section with the actual drafted content.

Announce the [SECTION NAME] section will be drafted now based on what they've selected.

**In conservative mode:** Draft ONLY using content from the user's notes and selections. If the draft would benefit from details not in their notes, add a `{{TODO: [specific detail]}}` placeholder instead of inventing the content. For example:
- ✅ DO: "The restart process involves three steps: [from user notes] ... {{TODO: Add estimated time for JVM to restart}}"
- ❌ DON'T: Add estimated time yourself ("typically 5-10 minutes") if they didn't provide it

**If using artifacts:**
After drafting, provide a link to the artifact.

Ask them to read through it and indicate what to change. Note that being specific helps learning for the next sections.

**If using a file (no artifacts):**
After drafting, confirm completion.

Inform them the [SECTION NAME] section has been drafted in [filename]. Ask them to read through it and indicate what to change. Note that being specific helps learning for the next sections.

**Key instruction for user (include when drafting the first section):**
Provide a note: Instead of editing the doc directly, ask them to indicate what to change. This helps learning of their style for future sections. For example: "Remove the X bullet - already covered by Y" or "Make the third paragraph more concise".

### Step 6: Iterative Refinement

As user provides feedback:
- Use `str_replace` to make edits (never reprint the whole doc)
- **If using artifacts:** Provide link to artifact after each edit
- **If using files:** Just confirm edits are complete
- If user edits doc directly and asks to read it: mentally note the changes they made and keep them in mind for future sections (this shows their preferences)

**Continue iterating** until user is satisfied with the section.

### Quality Checking

After 3 consecutive iterations with no substantial changes, ask if anything can be removed without losing important information.

When section is done, confirm [SECTION NAME] is complete. Ask if ready to move to the next section.

**Repeat for all sections.**

### Near Completion

As approaching completion (80%+ of sections done), announce intention to re-read the entire document and check for:
- Flow and consistency across sections
- Redundancy or contradictions
- Anything that feels like "slop" or generic filler
- Whether every sentence carries weight

Read entire document and provide feedback.

**When all sections are drafted and refined:**
Announce all sections are drafted. Indicate intention to review the complete document one more time.

Review for overall coherence, flow, completeness.

Provide any final suggestions.

Ask if ready to move to Reader Testing, or if they want to refine anything else.

## Stage 3: Reader Testing

**Goal:** Verify the document works for a fresh reader with no shared context.

This stage is handled entirely by the **`doc-reader-tester`** skill.

**In conservative mode (default):** Reader testing is automatic after Stage 2 completion.

**In standard mode:** Reader testing is optional. Ask if the user wants readability validation before finishing.

Announce: *"All sections are complete. Now handing off to the doc-reader-tester skill to validate the document from a reader's perspective."*

Invoke the `doc-reader-tester` skill, passing:
- The completed document (file path, artifact link, or content)

The `doc-reader-tester` skill will:
1. Predict realistic reader questions
2. Test the document with a fresh context (sub-agent or manually guided)
3. Run blind checks for ambiguity, assumed knowledge, and contradictions
4. Report findings and fix any gaps
5. Confirm when the document passes

When `doc-reader-tester` returns control (document passes), proceed to **Final Review** below.

## Final Review

When Reader Testing passes:
Announce the doc has passed reader testing with a fresh assistant. Before completion:

1. Recommend they do a final read-through themselves - they own this document and are responsible for its quality
2. Suggest double-checking any facts, links, or technical details
3. Ask them to verify it achieves the impact they wanted

Ask if they want one more review, or if the work is done.

**If user wants final review, provide it. Otherwise:**
Announce document completion. Provide a few final tips:
- Consider linking this conversation in an appendix so readers can see how the doc was developed
- Use appendices to provide depth without bloating the main doc
- Update the doc as feedback is received from real readers

## Tips for Effective Guidance

**Tone:**
- Be direct and procedural
- Explain rationale briefly when it affects user behavior
- Don't try to "sell" the approach - just execute it

**Handling Deviations:**
- If user wants to skip a stage: Ask if they want to skip this and write freeform
- If user seems frustrated: Acknowledge this is taking longer than expected. Suggest ways to move faster
- Always give user agency to adjust the process

**Context Management:**
- Throughout, if context is missing on something mentioned, proactively ask
- Don't let gaps accumulate - address them as they come up

**Artifact Management:**
- Use `create_file` for drafting full sections
- Use `str_replace` for all edits
- Provide artifact link after every change
- Never use artifacts for brainstorming lists - that's just conversation

**Quality over Speed:**
- Don't rush through stages
- Each iteration should make meaningful improvements
- The goal is a document that actually works for readers