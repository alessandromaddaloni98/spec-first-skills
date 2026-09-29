# spec-first-skills

Two [Claude Code](https://docs.claude.com/en/docs/claude-code) skills for a lightweight,
**spec-first** flavour of spec-driven development: design a change in one session,
save it as a spec, implement it in a fresh session with a clean context.

- **`spec-plan`** — at the end of a design conversation, writes the spec to `spec/`.
- **`spec-do`** — in a new session, implements the spec and closes it: verifies every
  step, fills in the Outcome, updates the Status.

No dependencies, no CLI: two `SKILL.md` files.

## Why

Designing a change with a coding agent takes a long conversation: exploring the code,
weighing options, discarding ideas. Implementing in that same session means carrying
all of it along — dead ends included — in the context window.

The fix is to split the work in two sessions and pass only the decisions across:

```
session 1: design ──► spec-plan ──► spec/<change>.md ──► session 2: spec-do ──► code + closed spec
```

Doing this by hand has two recurring costs, which these skills remove:

1. **Re-explaining the spec format every time.** `spec-plan` has a fixed template.
2. **Specs that are implemented but never updated.** `spec-do` makes closing the spec
   part of the work, not an afterthought.

## How it works

### `spec-plan`

1. Asks whether to create a new spec or update an existing one, then the file name.
2. Collects every ambiguous or undecided point from the conversation and asks them
   **in one message, before writing**. Discussed is not decided.
3. Checks that the files it cites exist.
4. Writes the spec, then does a **two-way final pass**:
   - *too much*: every line the implementation session wouldn't miss is removed;
   - *too little*: every step where the agent would have to decide something
     unwritten is a gap, and gets filled or asked.
5. Saves it as `ready` and tells you to start a fresh session.

Guiding principle:

> The spec must let the agent implement autonomously without choosing anything that
> isn't already decided, and without containing anything it doesn't need.

### `spec-do`

1. **Opening** — reads the spec, checks Status and Prerequisites, runs Step 0 (the
   premises the spec relies on), checks paths, asks all its questions in one message,
   and **waits for your go-ahead**.
2. **Implementation** — follows the steps in order, stops at declared gates, stops and
   asks on any decision that changes behaviour, logs minor decisions in the spec.
3. **Closing** (mandatory) — checks every step and acceptance criterion as
   done / not done / deviated, updates the docs marked "to update", writes the Outcome,
   sets the Status, reports, and **prints** the `git add` / `commit` / `push` commands
   for you to run. It never commits or pushes by itself.

## The spec

One Markdown file per change. Sections without real content are omitted.

| Section | Purpose |
|---|---|
| Header | Status (`draft` / `ready` / `in-progress` / `implemented`), Prerequisites, Related specs |
| 1. Context | Goal, size of the work, where the real difficulty is, out of scope |
| 2. File map | One line per file with its role in the change; docs to update |
| 3. Premises to verify (Step 0) | Commands to re-check the assumptions; stop if one fails |
| 4. Decisions | ID, date, decision, why; discarded alternatives |
| 5. Execution | Ordered steps with specific cautions |
| 6. Pitfalls | Wrong conclusions the agent may reach |
| 7. Acceptance criteria | Checklist |
| 8. Do NOT | Prohibitions specific to the change |
| 9. Open questions | What's undecided and what to do meanwhile |
| 10. Final report | What the agent must report at the end |
| 11. Outcome | Empty until implemented; filled by `spec-do` |
| Working rules | Fixed block: stop on ambiguity, no out-of-scope changes, no secrets |

See [`examples/spec/weather-client-retry.md`](examples/spec/weather-client-retry.md)
for a complete, implemented example.

## Install

Clone the repo and copy the two folders into your skills directory:

```bash
git clone https://github.com/alessandromaddaloni98/spec-first-skills.git
cd spec-first-skills

# for one project
mkdir -p your-project/.claude/skills
cp -r skills/spec-plan skills/spec-do your-project/.claude/skills/

# for all your projects
mkdir -p ~/.claude/skills
cp -r skills/spec-plan skills/spec-do ~/.claude/skills/
```

Start a new Claude Code session so the skills are loaded.

## Usage

At the end of a design conversation:

```
/spec-plan
```

Then, in a **new** session:

```
/spec-do spec/<change>.md
```

Both also trigger on natural phrases ("write the spec", "implement spec/X.md").

## Compared to OpenSpec and Spec Kit

[OpenSpec](https://github.com/Fission-AI/OpenSpec) and
[Spec Kit](https://github.com/github/spec-kit) are full frameworks. OpenSpec in
particular keeps **living specs** of the whole system and merges each change into
them on archive (closer to *spec-anchored* development).

These skills are deliberately smaller:

- **spec-first, per change** — a spec describes one change, not the system. The
  code stays the source of truth.
- **one file** instead of proposal / design / tasks / deltas.
- **execution guardrails** — Step 0 with a stop condition, Pitfalls, Do NOT, a
  mandatory Outcome — built from real mistakes coding agents make.
- **extracted, not generated** — `spec-plan` distills a design conversation that
  already happened, instead of producing a proposal from a one-line request.

Pick a framework if a team needs a maintained description of how the system behaves.
Pick these if you work alone or in a small team on vertical changes to an existing
codebase.

## When not to use it

For a fix of a few lines, where designing and implementing fit comfortably in one
session, a spec is pure overhead. Just do it.

## License

[MIT](LICENSE)
