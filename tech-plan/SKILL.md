---
name: tech-plan
description: Build a technical implementation plan for one Jira ticket, split into steps and subtasks with implementation details.
disable-model-invocation: true
argument-hint: <JIRA-KEY>
---

Ticket: $ARGUMENTS

`<KEY>` below is that ticket key in uppercase (`pes-1234` → `PES-1234`), everywhere it appears.

Produce a tech plan the user implements by following its numbered steps alone, top to bottom. The user reads the steps and nothing else, so every decision, blocker and warning a step depends on lives inside that step. Plan text in English; the ticket description stays in its original language.

## 1. Load the ticket

- No ticket key in `$ARGUMENTS`: ask for it and stop.
- Atlassian MCP: `getAccessibleAtlassianResources` once for the `cloudId`, then `getJiraIssue` with `view: full`. Fetch the parent, the Jira subtasks and the linked issues the same way; read linked Confluence pages with `getConfluenceContent`.
- Atlassian unreachable or unauthorized: tell the user and stop. A plan built on a guessed ticket is worthless.
- Look in `/Users/koosdaniel/Devel/Repos/Peak/tech-plans/` for `<KEY>-tech-plan.md`. If it exists, ask whether to update it or start over, and wait. Read the parent's and sibling tickets' plans when present: they hold agreed decisions and scope splits.

Done when: ticket text, acceptance criteria, parent, subtasks, links and any prior plans are in context.

## 2. Map the current state

Find every place in the codebase the ticket touches. Hand broad sweeps to parallel `Explore` agents; read the decisive files yourself.

For each touchpoint, record from the live file:

- what the code does today, with `path:line` read in this step;
- the nearest existing pattern the new code copies (same-kind repository, client method, DTO, test file, migration). The plan's code follows that precedent, not generic framework knowledge;
- the real commands from `project.json` / `package.json` (nx targets for migrations, `nx test <project> --testFile=` for tests).

Done when: every acceptance criterion maps to concrete files, and every proposed piece of code names its precedent in the repo or is flagged as having none.

## 3. Settle forks and names

Collect in one list:

- design forks the ticket and the code leave open (storage form, where logic lives, sync vs async, error behavior);
- every new name the plan introduces: files, classes, methods, columns, enum values, migrations.

Ask the user in batches (`AskUserQuestion`, up to 4 questions per call), each with 1-2 proposed options, recommended one first. Record each answer as `Decided (<user>, <date>)` in the step it shapes.

Done when: every fork is decided or parked as ⚠️ in its step, and every new name is confirmed or marked `⚠️ name not agreed`.

## 4. Split into steps and subtasks

- Steps in implementation order, one project or layer each: `### Step N — <project>: <what>`.
- Subtasks `N.1`, `N.2`, … are each one implementable change: the file, the exact change, a code snippet for anything non-trivial (schema, signatures, queries, DTO fields), and the reason where it isn't obvious.
- Open points go inline in the step they affect: `⚠️ <caveat> — <what to do until resolved>`. A point that blocks the step: `⚠️ BLOCKING PRECONDITION: <what> — <what to do until resolved>`.
- Ticket has Jira subtasks: tag each step with the subtask key covering it; a step no subtask covers gets `⚠️ no Jira subtask`.
- Every step that changes behavior ends with its own test subtask: file, cases as `should …` bullets, run command.
- Last step: verification — full test runs and lint on every touched project.

Done when: every acceptance criterion is satisfied by at least one subtask, and each subtask is implementable without reading anything outside its step.

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

- ⚠️ <caveat> — <what to do until resolved>

1.2 Tests — `<test file>`:

- should …

Run: `NX_TUI=false npx nx test <project> --testFile=<path> --codeCoverage=false`

### Step N — Verification

N.1 …
````

## 6. Report

Reply with the plan file's link and at most 3 lines: step count, open ⚠️ count, blocking preconditions. The plan itself stays in the file.
