---
name: subagent-driven-plan-review
description: Use when reviewing a written implementation plan, plan-mode output, or task breakdown before execution, especially when stronger multi-agent scrutiny is needed than the default single plan reviewer.
---

# Subagent-Driven Plan Review

## Overview

Use a fixed tiered reviewer matrix to stress-test a plan before execution. The goal is to catch missing requirements, vague steps, bad sequencing, and hidden execution blockers before an implementer starts work.

This is a plan review workflow, not a code review workflow. Focus on whether the next agent can build the right thing without getting stuck or inventing missing details.

## Review Scope

Review a stable artifact:
- preferred: a saved plan file plus a spec or exact requirements file
- acceptable: exact plan text plus exact requirements text for a read-only review pass
- never: a moving target or your session history

If you need a multi-pass review loop with direct plan edits, use a saved plan artifact.

## Reviewer Matrices

Every reviewer checks the whole plan. Do not assign specialized reviewer roles. The only intended differences are model tier and reasoning effort.

### Codex

Dispatch exactly these four reviewers in parallel:

1. `gpt-5.3-codex` / `medium`
2. `gpt-5.2-codex` / `medium`
3. `gpt-5.4` / `medium`
4. `gpt-5.4-mini` / `xhigh`

### Claude

Dispatch exactly these three reviewers in parallel:

1. `Opus` / `medium`
2. `Sonnet` / `high`
3. `Haiku` / `xhigh`

Use the full reviewer matrix for the current platform. Do not reduce the reviewer count. Do not swap these models unless the runtime lacks one of them. Here, runtime family means the current platform matrix (`Codex` or `Claude`), not a model-name suffix. If a listed model is unavailable, keep the reviewer count the same, prefer the nearest stronger available model in the same platform matrix, and fall back to the nearest weaker available model only when no stronger substitute exists. Reuse of a substitute model is allowed only when needed to preserve the reviewer count. If the runtime still cannot satisfy the full matrix, stop and tell the user the matrix cannot run as designed in that environment. Say every substitution explicitly in the review result.

## Shared Prompt Contract

Give every reviewer:
- the exact review scope
- the plan artifact
- the spec or requirements artifact
- the same full-plan review rubric
- instructions to flag only issues that would cause wrong implementation, real rework, or execution blockage
- instructions to ignore style-only rewrites and alternate valid preferences
- instructions to cite exact plan locations for every blocking issue

Every reviewer should check the whole plan for:
- spec coverage
- ambiguity that could lead to divergent implementations
- execution blockers such as missing files, commands, prerequisites, or verification steps
- bad sequencing or decomposition that would create rework
- scope creep or unjustified extras
- missing or weak testing and verification coverage

Ask every reviewer to return this reviewer-level output:

````markdown
## Plan Review

**Status:** Approved | Issues Found
**Reviewer:** [model and reasoning effort]
**Summary:** [2-4 sentences]

**Issues:**
- [Severity: critical|important|minor] [Confidence: 0-100] [Location] [Problem] - [Why it matters] - [Concrete fix]

**Advisory Suggestions:**
- [optional, non-blocking suggestions]
````

If a reviewer has no blocking issues, require them to say `Issues: None`.

## Workflow

1. Freeze the review scope. Prefer a saved plan path and a saved spec path.
2. Detect the current runtime family and choose the matching reviewer matrix.
3. Dispatch the full reviewer matrix in parallel using fresh agents without inherited session context. If the plan and requirements cannot be passed directly, stop and tell the user the matrix cannot run on the current inputs without violating scope.
4. Aggregate candidate issues and deduplicate by location and failure mode.
   - Treat overlapping findings from different reviewers as corroboration, not separate issues.
   - Keep unique findings when they are concrete and supported by the plan or requirements artifact.
5. Keep only concrete, actionable findings.
   - Keep by default: confidence `80+`
   - Keep `60-79` only if another reviewer independently corroborates the same issue or you can verify it directly from the plan
   - Discard below `60` unless the user explicitly asked for speculative feedback
6. If the review scope is a saved plan artifact, the agent invoking this skill fixes kept issues in the plan.
7. If the review scope is text-only, return exact required edits instead of pretending the plan was updated.
8. Rerun the full reviewer matrix after material plan changes to a saved plan artifact.
9. Stop when the review is approved or after `3` full passes. If disagreement remains after that, surface the conflict to the user instead of forcing closure.

## What Counts As A Real Issue

- A required behavior from the spec is missing from the plan
- A step is ambiguous enough that two competent implementers could build different outcomes
- Task order would block execution or create avoidable rework
- File paths, commands, tests, prerequisites, or verification steps are missing where they are needed
- The plan adds scope that is not justified by the stated goal
- Steps are too large or vague to execute safely
- The plan asks the implementer to make unowned design decisions that should already be settled

## What Does Not Count

- Tone or wording preferences
- Alternate valid decompositions with no clear execution risk
- Requests for more polish when the plan is already buildable
- Personal taste about naming, section ordering, or formatting

## Required Final Output

After collecting reviewer outputs, the agent invoking this skill aggregates them and returns the completed review in these sections:

### review_scope

- what plan artifact was reviewed
- what requirements artifact was used

### reviewers_used

- the reviewer matrix used and any model substitutions

### kept_issues

- only concrete issues that survived deduplication and confidence filtering

### discarded_issues

- brief reason each discarded issue was dropped

### plan_updates_needed

- what must change before execution can start

### approval_status

- `approved` or `issues found`

If these sections are missing, the review is incomplete.

## Integration

- Use this as a standalone higher-scrutiny plan review when you want more than a single reviewer pass.
- Use this before implementation starts when you explicitly want the reviewer matrix in this skill.
- This skill requires subagent support. If the current runtime cannot dispatch subagents, stop and tell the user the matrix cannot run as designed in that environment.

## Red Flags

- "One plan reviewer is enough"
- "Different models should each get different review jobs"
- "The implementer can figure out the missing details"
- "This plan mostly matches the spec"
- "These steps are obvious, so I can leave them vague"
- "The plan is just a draft, so precise review is overkill"

All of these weaken the review gate. Stop and run the full reviewer-matrix pass.
