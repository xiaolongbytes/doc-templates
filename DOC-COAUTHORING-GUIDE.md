# Tired of Writing Runbooks? Let AI Do the Scaffolding.

Create better documentation faster, with no hallucinations with `doc-coauthoring` skill.

- **You talk, AI writes** — Tell the skill what to document. It asks clarifying questions, then structures and polishes it into a real runbook.
- **It won't hallucinate** — The skill only uses details you provide. If it wants to infer something, it asks first. Everything unconfirmed gets marked `{{TODO}}` so you catch it before publishing.
- **Fresh reader catches blind spots** — Before you share, the skill tests the doc from a reader's perspective. "Wait, what's access control?" gets caught and fixed.

**Result:** Docs that are clear, complete, and actually yours (not AI guesses).

## 3-Minute Start

### 1. Jot down your rough notes
Notes on a napkin are fine. No need to organize:
- "Restart service prod03, takes 5-10 min, needs [these steps]"
- "Here's the context, here's what should happen"
- Bullet points, paragraphs, links — whatever

### 2. Open the skill and paste
- Type `/doc-coauthoring` in VS Code chat
- Paste or describe your notes

### 3. The skill does the rest
You answer a few quick questions. The skill asks:
- "Who's reading this?"
- "Anything you definitely want included?"
- Then it drafts, you refine, it tests.

Done.

## How It Prevents AI Hallucination

Worried the skill will invent details and you won't notice? Here's why it won't:

1. **It only uses your information.** The skill doesn't fill gaps with plausible-sounding AI guesses. If it doesn't know something, it asks you or marks it `{{TODO}}`.
2. **It asks before assuming.** See a detail in your notes that could mean multiple things? It asks, "Did you mean X or Y?" instead of picking.
3. **A fresh reader catches blind spots.** Before you share, the skill tests the doc with zero context. If there's a confusing part or missing step, the fresh perspective finds it.

This is the default behavior. It's slower than generic AI writing, but you get docs that are actually correct.

**Optional:** If you're experienced and in a hurry, you can switch to a faster mode that infers more. But for runbooks, compliance stuff, or anything your team owns, the default conservative approach is safer.

## Before & After

**Your rough note:**
```
Restart the service
1. SSH to prod03
2. Run systemctl restart myapp
3. Wait a bit
4. Check logs
5. Verify results
```

**What the skill produces:**
```
Prerequisites: SSH access to prod03 (confirm this is already provisioned?)

Steps:
1. SSH to prod03
2. Run: sudo systemctl restart myapp
3. Wait 2 minutes (service takes ~90s to restart, but wait for stability)
4. Check logs: tail -f /var/log/myapp.log
   - Look for: "Service initialized successfully"
   - If you see errors → {{TODO: What's the escalation path?}}
5. Verification:
   - Health check endpoint: curl prod03:8080/health
   - Expected: 200 OK response
   - If you don't see it in 30s, the restart failed → escalate to [team] 
```

Same information, but an on-call engineer actually knows what to do. No more "verify results"—it says what verification means.

## Common Questions

| I want to... | Just tell the skill... |
|-----------|-----------|
| **Skip a section** | "Skip this for now, not relevant" — marked as `{{TODO}}` to decide later |
| **Draft feels incomplete** | Totally fine. Leave `{{TODO}}` markers, assign to an expert later |
| **Catch inaccuracies** | Tell the skill right away: "That's wrong, mark it TODO instead of guessing" |

## One More Thing: The Reader Test

Before you publish, the skill automatically tests your doc with a fresh perspective using the `doc-reader-tester` skill. It reads it like a reviewer who has no context, no shared Teams conversations, nothing except the words on the page.

This catches:
- "Wait, what does 'verify' mean here?"
- "You didn't explain what that error means"
- "How do I know if this worked?"
- "Who is the IDOC team and how do I contact them?"

You get to see those questions and fix them before anyone else reads it. It's like having a reviewer built in, providing instant feedback. No more waiting.
