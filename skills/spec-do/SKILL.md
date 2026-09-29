---
name: spec-do
description: Implements a spec from spec/ written with spec-plan, in a fresh session. Opens with checks and questions, implements, then closes by verifying every step against the spec, filling the Outcome, updating Status and linked docs, and printing git commands. Use when the user says "read spec/X.md and proceed", "implement the spec", "go ahead with the spec", "ask me questions first, then implement", or similar. Also use during an implementation session when the user asks "update the spec", "did you finish everything in the spec?", or to close the spec.
---

# spec-do

## Why this exists

The user designs a change in one session and saves it as a spec with `spec-plan`. This skill runs the other half: a fresh session that reads the spec and implements it.

The spec gives the content (what to do, decisions, pitfalls, report format). This skill gives the procedure. Don't restate or reinterpret the spec: execute its sections.

The implementation is **not finished until the closing phase is done**: Outcome written, Status updated, linked docs checked. Without this step, specs end up implemented but never updated.

## Phase 1 — Opening

1. **Read the whole spec** before doing anything else. If the user didn't name one, list the specs in `spec/` with their Status and ask.
2. **Check the Status.**
   - `implemented` → say so and ask whether to proceed anyway.
   - `draft` → say the spec isn't marked ready and ask whether to proceed.
   - `in-progress` → read the Outcome to see what's already done, and resume from there.
3. **Check the Prerequisites.** Every spec cited there must be `implemented`. If not, stop and report.
4. **Run Step 0** (Premises to verify) exactly as written. If a premise no longer holds, stop and report it: don't adapt the spec on your own.
5. **Check the paths** in the File map: they must exist, unless the map says the file is to be created.
6. **Send one single message** with: the outcome of checks 2–5, and every question you have about the spec (ambiguities, gaps, conflicts with the code). If you have no questions, say so explicitly.
7. **Stop and wait** for the user's go-ahead. Always, even with no questions.

## Phase 2 — Implementation

1. On the go-ahead, set the Status to `in-progress (YYYY-MM-DD)`.
2. Follow the Execution steps in order.
3. **Respect the gates the spec declares** (e.g. "don't start the production phase before the user confirms"): stop there, report, and wait.
4. **New decisions during the work:**
   - if it changes the system's behaviour, or the spec is ambiguous or in conflict with the code → stop and ask;
   - if it's minor → decide, and add it to the Decisions table with a new ID and today's date.
   Never silently diverge from the spec.
5. Follow the spec's "Do NOT", "Pitfalls" and "Working rules" throughout.

## Phase 3 — Closing (mandatory)

Run it as soon as the implementation ends, without waiting to be asked.

1. **Point-by-point check.** Go through every Execution step and every Acceptance criterion. For each one: `done`, `not done` (with why), or `deviated` (with how and why). Run the checks that can be run; don't mark something `done` from memory.
2. **Linked docs.** For every doc listed in the File map as "to update", check it and update it. If another doc now contradicts the code, update it too and mention it.
3. **Write the Outcome** in the spec (section 11): date, what was done, deviations, side effects, residual divergences, what's left if anything. Short and factual, with numbers where the spec asks for them.
4. **Update the Status:**
   - everything done → `implemented (YYYY-MM-DD)`;
   - something left → stays `in-progress (YYYY-MM-DD)`, and the Outcome lists what's missing.
5. **Report to the user** in the format required by the spec's Final report (section 10). If the spec has none: outcome per step, test results (new failures separated from pre-existing ones), deviations.
6. **Git commands.** Never run `git add`, `git commit` or `git push` yourself. Print the commands for the user to run:
   - `git add` with the explicit list of files changed (code, spec, docs) — never `git add -A` or `git add .`;
   - `git commit -m "..."`: use the message the spec suggests if any, otherwise follow the style of `git log --oneline -10`;
   - `git push`.
   Never include `.env` or files with secrets in the list.

## Rules

- Never read `.env` files, never make calls with real credentials, never print secrets in console or logs.
- Don't delete data files without explicit confirmation.
- No improvements, refactors or style fixes outside the spec's scope.
- Distinguish new test failures from pre-existing ones; don't fix unrelated failures.
- Write spec updates (Outcome, Decisions) in English, like the rest of the spec. Code follows the repo's conventions. Talk to the user in their language.
