---
name: spec-plan
description: Writes or updates a spec in spec/ from the current design conversation, for a single vertical change, so that a fresh session can implement it with a clean context. Use at the end of a design conversation, when the user says "write the spec", "save the spec", "turn this into a spec", "wrap up the design", or similar. Not for updating a spec at the end of an implementation session: that is spec-do.
---

# spec-plan

## Why this exists

The user works in two phases: one session to design a change (what to touch, how, why), then a fresh session that implements it with a clean context. This skill is the bridge: it turns the design session into a spec in `spec/`, which will be the only context of the implementation session.

A spec covers **one single vertical change** (one component). If the conversation contains more than one, ask the user which one goes into the spec.

## Guiding principle

> The spec must let the agent implement autonomously without choosing anything that isn't already decided, and without containing anything it doesn't need.

Every rule below follows from this. The goal is not brevity: a short spec with a gap is worse than a long complete one. But every extra line consumes the implementation session's context.

## Flow

1. **New or update?** Ask the user this first.
2. **New** → ask for the file name. If `spec/<name>.md` already exists, say so and ask for another name. Create `spec/` if it doesn't exist.
3. **Update** → list the specs in `spec/` with their Status and let the user choose. Read the chosen spec in full before changing it.
4. **Source**: the whole current conversation.
5. **Questions before writing.** List the ambiguous or undecided points that emerge from the conversation and ask them in a single message. Don't write until you have answers. Something discussed but not confirmed is not a decision: ask.
6. **Path check.** Verify that every file cited in the File map exists (existence only, not content). If a file doesn't exist, flag it: either the path is wrong, or the file must be created and the map must say so.
7. **Write the draft** following the template. In the File map, also list the docs (test guides, README, `docs/`) the change will make stale, with role "to update": `spec-do` updates exactly those at closing.
8. **Two-way final pass** (see below). Fix the draft; if gaps emerge that the conversation doesn't resolve, ask the user and integrate the answers.
9. **Save** with Status `ready` (or `draft` if an open question blocks the implementation) and reply with one line: the file path and an invitation to start a fresh session from the spec. No ready-made kickoff prompt, no summary of the content.

### Updating an existing spec

- Change only the sections touched by the conversation; don't rewrite the rest.
- New decisions are added with a progressive ID and a date.
- Superseded decisions are **not deleted**: mark them `superseded by Dx (date)`, and fix the text that depended on them.
- Update the Status.

## Final pass

Before saving, reread the draft with two checks.

**Too much.** For each line: *if I remove it, does the implementation session get something wrong or get stuck?* If not, remove it.

**Too little.** For each execution step: *does the agent have to decide something that isn't written here?* If yes, it's a gap:
- if the answer is in the conversation, add it to the spec;
- if it isn't, ask the user and write the answer into the spec;
- if the user wants to leave it open, it goes into Open questions, with what to do in the meantime.

A gap the agent fills on its own is the worst case: that's where the implementation session makes a decision that was ruled out during design.

## Anti-verbosity rules

- **Length proportional to the change.** A 20-line change across 3 files gets a one-page spec, not ten pages.
- **Empty sections are not written**, not even with "none" or "n/a".
- **Cite, don't duplicate.** If content already exists in another spec, `AGENTS.md`, a README or a docstring, reference it.
- **Nothing the agent can read by itself in the code.** No function descriptions, no file dumps. Snippets only when they must be copied verbatim (a prompt, a block to remove exactly, a command to run).
- **Decisions only, not history.** The conversation contains attempts, changes of mind, intermediate reasoning: only where things landed goes into the spec. Discarded alternatives get one line each, with the reason.

## Template

The spec is a **Markdown** file (`spec/<name>.md`) written in **English**, whatever language the conversation is in. File names, symbols and code stay as in the repo.

```markdown
# <Title: the change, in one line>

**Status**: draft | ready | in-progress | implemented (YYYY-MM-DD)
**Prerequisites**: <specs or conditions that must hold before starting>
**Related specs**: <upstream or downstream specs, with a word on the link>

> Implement this spec with the `spec-do` skill, which updates it at the end
> of the work (Outcome, Status, new entries in Decisions).

## 1. Context
<Goal: what changes and why, in a few lines.
Size of the work and where the real difficulty is (often not the code).
Current state of the component: only what's needed to understand the change.
Out of scope: what this change deliberately does not do.>

## 2. File map
| File | Role in this change |
|---|---|
<One row per file touched or read as reference. Only what the agent can't infer by opening the file.
Include the docs (guides, README, docs/) this change makes stale, with role "to update".>

## 3. Premises to verify (Step 0)
<The assumptions the spec relies on, each with the command to verify it.
If a premise no longer holds: stop and report, don't adapt.>

## 4. Decisions
| ID | Date | Decision | Why |
|---|---|---|---|
<Constraints to respect.
Discarded alternatives: one line each, with the reason.>

## 5. Execution
<Ordered steps. For each step: what to do, where, and the specific cautions.
Snippets only if they must be copied verbatim.>

## 6. Pitfalls
<Wrong conclusions the agent may reach (skewed comparisons, apparent asymmetries
to "fix", files that look like they need changes and don't).
Different from cautions: these are about what gets inferred, not how to edit.>

## 7. Acceptance criteria
- [ ] <checklist>

## 8. Do NOT
- <Prohibitions specific to this change, with the reason when not obvious.>

## 9. Open questions
<Points the user left open, with what to do in the meantime.>

## 10. Final report
<What to report at the end: outcome of each step, divergences with numbers
(not a generic "it works"), deviations from the spec and why.>

## 11. Outcome
<Empty. Filled in by whoever implements: what was done, deviations,
side effects, residual divergences.>

## Working rules
- If a choice is ambiguous and changes the system's behaviour: stop and ask.
- No improvements, simplifications or style fixes outside the scope.
- Don't delete data files without explicit confirmation.
- Don't read `.env` files, don't make calls with real credentials, never print secrets in console or logs.
- Distinguish new test failures from pre-existing ones; don't fix unrelated ones.
- Code, names, docstrings, comments and logs follow the repo's conventions.
```

Always include the "Working rules" block verbatim: it makes the rules reach the implementation session even when the spec is used without `spec-do`.
