---
name: tech-plan
description: "Tech plan for one Jira ticket (tech-plans/<KEY>-tech-plan.md). Use when building a new plan, or editing an existing one: recording decisions, renames, open questions, subtasks."
argument-hint: <JIRA-KEY>
---

Ticket: $ARGUMENTS

`<KEY>` below is that ticket key in uppercase (`pes-1234` → `PES-1234`), everywhere it appears.

Produce a tech plan the user implements by following its numbered steps alone, top to bottom. The user reads the steps and nothing else, so every decision, blocker and warning a step depends on lives inside that step. Plan text in English; the ticket description stays in its original language; the Subtasks section is Hungarian.

The plan carries exactly the ticket's scope: every step traces to the ticket text, an acceptance criterion or an explicit user instruction. Anything else you notice is an **own finding**: raise it in chat, and it enters the plan only when the user says so, as an open question starting with "Own finding:". The ticket's out-of-scope list stays out of the plan entirely. Asked where a step comes from, answer with the ticket key and criterion, or "own finding", in the first sentence.

Updating an existing plan: apply "Maintaining the plan" directly; steps 1–6 only for a new plan or a start-over.

## 1. Load the ticket

- No ticket key in `$ARGUMENTS`: take it from the plan file being edited (`<KEY>-tech-plan.md`); neither present: ask for it and stop.
- Atlassian MCP: `getAccessibleAtlassianResources` once for the `cloudId`, then `getJiraIssue` with `view: full`. Fetch the parent, the Jira subtasks and the linked issues the same way; read linked Confluence pages with `getConfluenceContent`.
- Atlassian unreachable or unauthorized: tell the user and stop. A plan built on a guessed ticket is worthless.
- Look in `/Users/koosdaniel/Devel/Repos/Peak/tech-plans/` for `<KEY>-tech-plan.md`. If it exists and the request does not say whether to update it or start over, ask and wait. Read the parent's and sibling tickets' plans when present: they hold agreed decisions and scope splits.

Done when: ticket text, acceptance criteria, out-of-scope list, parent, subtasks, links and any prior plans are in context.

## 2. Map the current state

Find every place in the codebase the ticket touches. Hand broad sweeps to parallel `Explore` agents; read the decisive files yourself.

For each touchpoint, record from the live file:

- what the code does today, with `path:line` read in this step;
- the nearest existing **precedent** the new code copies (same-kind repository, client method, DTO, test file, migration). The plan's code follows that precedent, not generic framework knowledge;
- the real commands from `project.json` / `package.json` (nx targets for migrations, `nx test <project> --testFile=` for tests).

Data uncertainty (duplicates, legacy rows, values that differ per environment) is handled by the code or the migration itself: idempotent, collision-guarded updates that do nothing when nothing matches. The user cannot run queries on any environment, so every step stands without one.

Done when: every acceptance criterion maps to concrete files, and every proposed piece of code names its precedent or is flagged as having none.

## 3. Present options, then names

Show the implementation options in chat before asking anything: per decision axis (storage form, where logic lives, sync vs async, error behaviour), each option with pros, cons and its precedent, then a recommended combination. Stop and wait for the user's pick.

After the pick, list every genuinely new name the plan introduces (files, classes, methods, constants, columns, enum values, routes, migrations) with 1-2 proposals each, recommended first, and ask (`AskUserQuestion`, up to 4 questions per call). A name that mechanically follows from an agreed one (the file of a class, its test file, a singleton, an import path) follows the nearest sibling precedent without a question.

Done when: the user picked an option per axis, and every new name is agreed or marked `⚠️ name not agreed` in its step.

## 4. Split into steps

- Steps in implementation order, one project or layer each: `### Step N — <project>: <what>`.
- Substeps `N.1`, `N.2`, … are each one implementable change: the file by full path, the exact change, a code snippet for anything non-trivial (schema, signatures, queries, DTO fields), and the reason where it isn't obvious. Every created, changed and deleted file appears this way, so the plan alone rebuilds the change from a clean branch.
- Open points go inline in the step they affect, per the open-question format below. A point that blocks the step: `⚠️ BLOCKING PRECONDITION: <what> — <what to do until resolved>`.
- Ticket has Jira subtasks: tag each step with the subtask key covering it; a step no subtask covers gets `⚠️ no Jira subtask`.
- Every step that changes behaviour ends with its own test substep: file, cases as `should …` bullets, run command. A case that proves an acceptance criterion ends with `(AC "<criterion title>")`.
- Last step: verification — full test runs and lint on every touched project.

Done when: every acceptance criterion is proven by at least one test case, and each substep is implementable without reading anything outside its step.

## 5. Write the plan

Save to `/Users/koosdaniel/Devel/Repos/Peak/tech-plans/<KEY>-tech-plan.md` from the template below, then add a one-line pointer with that absolute path to the current worktree's `MEMORY.md`. Code references are markdown links relative to the plan file, through the worktree folder and with line ranges (`../<worktree>/<path>#L12-L30`), so they open from the shared folder.

````markdown
---
name: <KEY>-tech-plan
description: "<KEY> — <one line: what changes, where>"
metadata:
  type: project
---

# <KEY> — <ticket title, verbatim>

Parent: <KEY> (<title>). Status at plan time: <status>, fetched <YYYY-MM-DD>. Assignee: <name>. Reporter: <name>.

## Ticket description (verbatim)

<full description and acceptance criteria, original language and formatting>

---

## Technical plan

Decisions: <agreed decisions, with who and when>

Scope: <what this ticket covers; what sibling tickets cover>

Current state (verified <YYYY-MM-DD>):

- <fact> — [file.ts:12-30](<path relative to this file>#L12-L30)

### Step 1 — <project>: <what>

1.1 <change> — [file.ts:40](<path>#L40)

```ts
<snippet>
```

- Names agreed (<YYYY-MM-DD>): `<name>`, `<name>`
- ⚠️ OQ-1: <question> — <what the plan does until decided>

  1.2 Tests — `<test file>`:

- should … (AC "<criterion title>")

Run: `NX_TUI=false npx nx test <project> --testFile=<path> --codeCoverage=false`

### Step N — Verification

N.1 …

## Subtasks

**<nx-project>: <rövid cím>**

- Tech plan: Step 1
- <rövid magyar pont>

## Open questions (all in one place; each is also ⚠️ inline in its step)

| #                                  | Question   | Until decided                  | Step |
| ---------------------------------- | ---------- | ------------------------------ | ---- |
| [OQ-1](<KEY>-tech-plan.md#L<line>) | <question> | <what the plan does meanwhile> | 1.1  |
````

## 6. Report

Reply with the plan file's link and at most 3 lines: step count, open ⚠️ count, blocking preconditions. The plan itself stays in the file.

## Decisions and open questions

- Decision: `- Decided (<who>, <YYYY-MM-DD>): <decision>` in the step it shapes.
- Agreed names: `- Names agreed (<YYYY-MM-DD>): …` in the step that introduces them. A later rename replaces the name everywhere and puts its own date next to it on that line.
- Open question: `- ⚠️ OQ-<n>: <question> — <what the plan does until decided>` in every step it affects, plus a row in the Open questions table linking to its main line. Numbers only grow; a closed number is never reused. The table is an overview; the inline line is what the user reads.
- Decided open question: the inline line becomes `- OQ-<n> decided (<who>, <YYYY-MM-DD>): <decision>` without ⚠️; steps, tests and subtasks change to match; the table row's question gets `— decided <YYYY-MM-DD>: <decision>` and "Until decided" becomes `Closed`. An option worth keeping for later stays on that line as "Possible solution if <condition>: …".

## Subtasks section

The Hungarian Jira subtask list, right before Open questions, in implementation order. Each subtask:

```markdown
**<nx-project>: <rövid cím>**

- Tech plan: <step or section refs>
- <rövid magyar pont>
```

- The title starts with the Nx project name from `project.json`, followed by a short Hungarian title; it carries no number and no section refs.
- The first bullet is `- Tech plan: <refs>`; the rest are short facts: what changes, which names, which codes.
- Bullets state the plan's chosen behaviour only. Warnings, caveats and open questions live in the steps.

## Code in the plan

- Write snippets straight from the precedent, comment-free; explanations go in the plan text around them.
- Names say what the value holds (`existingApostdRecordsByKey`, `apostdRecordsToUpdate`), never its role in the algorithm.
- Each branch states its own condition with explicit `if` / `else`, never relying on a callee that throws.
- A plain truthy check when the domain rules out meaningful falsy values (`if (count)` on a unique-key `updateMany`).
- `T[]` in type annotations; named constants for messages, keys and domain values.
- Logging through `logWithAuditType`, `errorWithAuditType`, `errorWithAlertType`; the alert type only on a path that rethrows.
- Migrations only through the project's nx targets (`migrate-dev --name=<name>`, `generate-types`); Prisma generates `migration.sql`.
- Tests follow the nearest existing test file of the same shape; case names start with `should`; runs go through `nx test`.
- Wording: ticket keys in full (`PES-1234`); "general ledger" written out; "read as", "parsed as" or "treated as" when describing how a value is understood; `…ByChunk` or `chunk…` for a value held once per chunk.

## Maintaining the plan

Every later edit, in the same go:

- The user approving a proposal, option, name or open question is the instruction to record it: write the decided line, update steps, tests, subtasks and the table, then report.
- A new preference or idea the user mentions while discussing stays in chat until they ask to record it. A question about the plan gets an answer in chat.
- A rename covers its derived names (file, test file, imports, references) in code snippets, steps, tests, subtasks and the names-agreed line.
- The Subtasks section changes with every edit that touches scope, steps, names, codes or decisions.
- Re-read a file right before citing `path:line`; numbers from an earlier read or a diff hunk drift.
- After any edit that adds or removes lines, remap every Open questions anchor to its line and check each target.
