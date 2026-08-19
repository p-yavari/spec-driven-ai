# Guidebook: Planning and Supervising AI-Built Software

Personal reference, built from real practice on the Euroslot Showroom project. Not tied to any one project or repo — read this before starting anything new, and before handing work to a coding agent.

---

## Part 1: Definitions

**Spec-Driven Development (SDD)** — the overall method: a written specification is the source of truth; code is a generated, verifiable artifact from it. Workflow: constitution → spec → plan → tasks → implement → review.

**Spec (`specs/spec.md`)** — *what* gets built. Purpose, audience, data model, functional and non-functional requirements, explicit out-of-scope items, open questions. A living document — updated whenever a gap is found, never silently worked around.

**Plan (`specs/plan.md`)** — *how* the spec gets built, technically. Concrete architecture: data model implementation, routing, tech stack choices with tradeoffs shown (not asserted), infrastructure. This is also where expensive-to-reverse technical decisions and their rationale get recorded — the plan doubles as your ADR (Architecture Decision Record) log.

**Tasks (`specs/tasks.md`)** — the plan broken into small, sequenced, individually reviewable implementation steps. One feature branch per task. Each task should be small enough to review its diff in full.

**Constitution (`AGENTS.md`)** — *how* agents are allowed to build anything, project-wide. Tech stack (locked), folder structure, decision-making process, shared sources of truth, Definition of Done checklist, git workflow, reliability/security requirements. Read by the coding agent at the start of every session. Versioned — every change bumps the version and is logged.

**Definition of Done (DoD)** — a standing checklist inside `AGENTS.md` that a feature must satisfy before being called complete: validation (client+server), real-time feedback, loading/empty/error states, accessibility, responsiveness, reused-not-duplicated utilities, tests run and shown, localization. Exists so quality doesn't depend on you personally knowing to ask for each item every time.

**Shared source of truth** — one canonical file/function for any cross-cutting rule (validators, form pattern, design tokens, access-control functions). The fix for "I have to repeat the same correction on every screen" and for the "fix a bug here, it reappears there" loop — both come from the same root cause: duplicated logic instead of one shared place.

**Right-sized / not over-built** — match the solution to actual scale and requirements, not to what sounds most impressive. No custom search infra, no microservices, no hand-rolled systems where a configured standard tool does the job. Applies to every technical decision, not just ones already made.

**ADR (Architecture Decision Record)** — the practice of writing down *why* an expensive-to-reverse decision was made, with the real alternatives that were considered. In this method, `plan.md` serves this role directly — no separate ADR file needed.

---

## Part 2: The Method, Step by Step

1. **Discovery.** Describe the idea, however vague. Get interviewed thoroughly — audience, data model, edge cases, non-functional requirements, legal/privacy — until nothing's left to ask, not until a fixed question count is hit.
2. **Spec.** Write it, then critique it yourself before accepting — look for missing non-functional requirements (SEO, accessibility, i18n, security) and vague statements that need to become concrete decisions.
3. **Plan.** Real technical alternatives with tradeoffs, decided by you — never a single option asserted as the obvious choice, especially for expensive-to-reverse decisions (framework, database, hosting, core data model).
4. **Tasks.** Broken small enough that each one's diff is genuinely reviewable in one sitting.
5. **Constitution (`AGENTS.md`).** Written before any code. Self-reviewed at least twice for gaps — real gaps kept surfacing on repeated passes in practice (environment/secrets handling, access control, reliability/ops, git workflow were all missed on the first pass).
6. **Implementation.** Handed to a coding agent with real repo access (e.g. Claude Code), one task at a time. Supervision habits below apply to every single task, not just the first few.

---

## Part 3: Starter Prompts

### New project, from scratch
```
I want to build [describe your idea, even if vague].

Interview me thoroughly before writing anything — don't stop after
a few questions. Keep asking follow-ups as gaps surface, the way a
careful product manager would: audience, data model, edge cases,
non-functional requirements (SEO, accessibility, i18n, performance,
security), legal/privacy, content/admin management. When my idea is
vague, don't fill gaps in yourself — ask me. Keep drilling until
there's genuinely nothing left to ask, not until you've hit some
fixed number of questions.

Then write specs/spec.md (purpose, audience, data model, functional
and non-functional requirements, out of scope, open questions).
Review your own draft critically and find its gaps before showing
it to me — don't rubber-stamp your first pass.

Then specs/plan.md — real technical tradeoffs shown for every
non-trivial decision (framework, database, hosting, etc.), not
choices asserted as obvious. Let me decide between options you
present.

Then specs/tasks.md — small, sequenced, individually reviewable
implementation steps.

Then AGENTS.md — a project constitution covering conventions,
shared sources of truth, a Definition of Done checklist, and git
workflow. Self-review it at least twice for gaps before we call it
done.

Take your time. I'd rather this take many turns than be shallow.
```

### Continuing an existing project, fresh chat
```
This is a continuation of an existing project at [repo path].
Read AGENTS.md and specs/tasks.md in full, and check git log to
confirm what's already merged and which task is next.

From here on, apply rigorous supervision to anything a coding
agent reports:
- Demand actual proof — real diffs, real command output — never
  accept a summary like "it works" at face value.
- Any change to AGENTS.md requires an explicit diff shown and
  separate approval, every time, even for tooling-injected changes.
- Flag any file changed that's unrelated to the current task before
  approving.
- "Plan approved" is never "implementation approved" — always wait
  for explicit sign-off after seeing the diff, before commit/merge.
- If a claim turns out to be fabricated or wrong, re-examine any
  other unverified claim made in the same breath — don't just
  correct the one caught.
- Verify UI-facing work with a real browser tool, not just curl/API
  status checks.
- Catch small hygiene issues (debug files, dead dependencies,
  inconsistent naming) the first time they appear.

Whenever your review finds issues, end your response with a message
addressed to the coding agent wrapped in an actual markdown code
block (triple backticks) — not labeled or bolded text. Put your
reasoning and explanation to me outside the block; the block itself
should contain nothing but what I paste verbatim to the agent, ready
to copy with one click.

Confirm you've understood this, then tell me what the next task is
before doing anything.
```

---

## Part 4: Supervision Habits (apply to every task, not just the first few)

1. **Demand proof, not summaries.** "I ran X and it worked" is not evidence — ask for the literal `git status`/`git diff`/test output/command output every time.
2. **The constitution file is never modified silently.** Any change to `AGENTS.md` gets shown as an explicit diff before approval — every time, including tooling-injected changes and changes you yourself requested.
3. **Unrelated files in a diff are a stop sign.** If something changed that has nothing to do with the current task, ask what and why before approving.
4. **Approval sequence is non-negotiable.** Plan approved ≠ implementation approved. Show diff → explicit sign-off → commit → push/PR → explicit sign-off again → merge. "It was small" is never an exception.
5. **One caught fabrication raises suspicion on everything nearby.** If an agent invents a plausible-sounding detail once, re-examine any other unverified claim made in the same breath.
6. **Verify UI claims with a real browser, not just HTTP status codes.** A `curl` 200 proves the server responded, not that the page renders correctly or the console is clean.
7. **Test the real edge case, not an invented mock of it.** A test that mocks its own assumption about framework behavior proves internal consistency, not correctness — verify actual framework/library behavior against real docs or a real run.
8. **Contradicting explanations are a signal.** If two different explanations are given for the same event, stop and ask which is actually true.
9. **Catch small housekeeping issues immediately.** Debug artifacts, dead dependencies, inconsistent file casing, temp-scaffold leftovers — cheap to catch on first appearance, compound if not.
10. **Good behavior after a mistake earns trust — but doesn't suspend verification.** A direct, unhedged correction is the right response and worth recognizing as such; it doesn't mean the next claim gets a pass.
11. **Trace forward, not just inward.** Reviewing "is this task's own logic correct" is not the same as reviewing "what does this become once later tasks depend on it." Before approving a new entity, default record, or shared value, check the plan for how future tasks will reference it, and decide its lifecycle protections (deletable? renamable? guaranteed to exist?) now — not after real data depends on it. This was missed once in practice: a seeded default record had no protection against deletion, discovered only because a later task's relationship to it was checked by hand, not because any review step asked the forward-looking question by default.
12. **End every review that finds issues with the agent-facing message in an actual code block, not labeled text.** You are relaying between two separate agent conversations by hand. "Clearly marked" isn't precise enough — it produced a labeled paragraph, not something that copies cleanly. Require a fenced markdown code block containing nothing but what gets pasted verbatim; reasoning and explanation belong outside it, addressed to you.

---

## Part 5: Tooling Notes

- **GitHub PR workflow** (adopt once CI exists): feature branch → push → open PR → CI runs automatically → review diff on GitHub → explicit approval → merge. Stronger than local `git merge` since CI becomes a real automated gate, not a manual "I ran it locally" claim.
- **Automated first-pass review**: `anthropics/claude-code-action@v1` on GitHub Actions can review every PR diff automatically (bugs, security, performance) before it reaches you — configure its prompt to check against your `AGENTS.md` Definition of Done. Reduces manual copy-paste review loops; does not replace your final sign-off.
- **Browser verification tool** (e.g. Chrome DevTools MCP): required for any task with a real UI surface. `curl`/API checks are not sufficient substitutes.
- **GitHub Spec Kit**: adopt for the *next* new project, not mid-project on an existing one — migrating planning docs into its structure mid-build is real rework for no benefit. It automates spec/plan authoring and consistency-checking (`/analyze`); it does not replace the supervision habits in Part 4, which govern how you review what an agent claims it did, a different layer entirely.

---

## Part 6: Honest Limitations

A kickoff prompt sets posture, not performance. What actually made this project's process rigorous was **sustained pushback across many turns** — catching a missing UI element in a screenshot nobody asked about, refusing to accept "it works" without real output, noticing a second unverified claim right after the first was caught. That's not something a prompt replicates by itself. Expect quality to start close to this standard and drift toward shallower defaults over a long session unless the habits in Part 4 are actively kept up — the same rules needed re-stating here even after being written down once. Also confirm real verification tools (browser MCP, running infrastructure) are actually connected in any new environment — several of the real catches in this project only happened because tool output was demanded and available.
