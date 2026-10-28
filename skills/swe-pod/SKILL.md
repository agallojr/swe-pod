---
name: swe-pod
description: Read a SWE Pod Project Space's records to say where each change stands, regenerate STATUS.md, act on the human's go as the lead, or walk the traceability chain and report its breaks. Invoke in a Project Space when the human asks where things stand or for the status, says go or asks for the next thing to be done, asks for STATUS.md to be regenerated, or asks for a traceability check.
allowed-tools: Read, Grep, Glob, Bash, Write
---

# swe-pod: read a Project Space and act on it

A Project Space holds a project's specs, plans, ledgers, evidence and
project-level tests (`PROCESS.md §12`). This skill reads those records and
the code repos `pod.yaml` names, then does one of four things:

| The human | Section |
|---|---|
| asks "where are we?" or for the status | 2 Where things stand |
| asks for `STATUS.md` to be regenerated | 3 Regenerate STATUS.md |
| says "go", or asks for the next thing to be done | 4 Go |
| asks for a traceability check | 5 Trace |

Run it from the space checkout. Paths are relative to the space root unless
they say otherwise. The process is the space's `PROCESS.md`, at the version
`pod.yaml` pins; the record forms are swe-pod's `templates/records/`.

## 1. Read the records

### 1.1 Where and how

Read in this order: `AGENTS.md`, `STATUS.md`, `pod.yaml`, `space.yaml`, then
for each change its `spec.md`, `plan.md`, `ledger.md` and `evidence/`, then
`handoffs/` and `decisions/`.

- **As committed.** Read every record at the space checkout's `HEAD`, with
  `git -C <space> show HEAD:<path>` and
  `git -C <space> ls-tree -r --name-only HEAD`. A file that is not committed
  has not been filed, so it is not a record yet.
- **Open changes** are the folders `changes/<id>/` that hold a `spec.md`,
  `changes/archive/` excluded, whose state (1.2) is not integrated or
  abandoned.
- **Code repos, read-only.** For a branch head, run
  `git ls-remote <remote> refs/heads/change/<id>` with the remote from
  `pod.yaml`. For commits, fetch the branch into a local clone and read its
  log (section 5). Never commit, push, check out, rebase or create a branch
  in a code repo from this skill.

### 1.2 Change state

Read each change's state from its records, never from memory
(`PROCESS.md §2`). Take the first that holds:

| State | Read from |
|---|---|
| abandoned | `spec.md` has `abandoned:` with a reason |
| integrated | `spec.md` has `integrated:`, and the folder is under `changes/archive/` |
| proposed | `spec.md` has no `approved:` line |
| verified | the latest evidence (1.6) is green at the current commit set (1.5), or for a sharded verify the shards' latest records are green together (1.7); and the whole-change review is done with no open Critical or Important finding |
| building | the ledger has a dispatch line |
| planned | `plan.md` exists, and its `## Self-review` section is filled in: not empty, and no unchecked `- [ ]` item left |
| approved | `spec.md` has `approved:` |

The spec's header is its list of `- <key>: <value>` lines before its first
level-2 heading. Its `owner:`, `repos:`, `approved:`, `integrated:` and
`abandoned:` lines are read there only, never from an example further
down.

In a spec or plan, only `#` lines outside fenced code blocks are headings,
so an example spec inside a fence adds none. A section is named by its
level-2 heading, with any number before the name ignored, so
`## 12. Requirements` is the Requirements section. The spec's requirements
are its `### R<n>` headings and its scenarios its `#### R<n>.S<m>` headings
in that section. Its quality checks are the `### Q<n>` headings in its
Quality section; with no such section it declares none.

### 1.3 Plan tasks

The plan's tasks are its entries in sections 5 to 8:

- each heading `### T<n> <title>` that opens with one task id;
- each row of a task table whose first cell is a task id, as plan section 7
  lists its review tasks.

A task id is `T` and a number, with an optional lowercase letter (`T19a`).
A heading opens with the ids before its title, joined by `, ` or `and`;
ids in the title do not count, so `### T21 Review of T1` is the task T21.
Some fields are stated once for several tasks: by a heading that opens
with several ids, such as `### T19a and T19b <title>`, which is not itself
a task, or by the sentence above a table. Each task takes those fields from
there. In an `after:` list, `T<a> to T<b>` means every plan task numbered
from a to b, letter suffixes included; text after the closing bracket is a
comment.

A **review task** is a task whose role is review. The whole-change review,
the one whose heading title is `Whole-change review`, reviews no single
task. Any other review task reviews the task given by the first of these
it has:

1. its Reviews cell, in a review table: the task id the cell begins with;
2. its heading, `### T<n> Review of T<m>`: T<m>;
3. its `after:` list, when that list names one task: that task.

With none of them, it reviews no single task.

### 1.4 Reading the ledger

The ledger is the fenced block in `changes/<id>/ledger.md`. An event is a
line that begins with a UTC, `YYYY-MM-DDTHH:MMZ`, plus the indented lines
that continue it. Events count in UTC order, and events with the same UTC
in the order written. The event forms are those in swe-pod's
`templates/records/ledger.md`. A ledger written before the forms existed
may record an event in words; such a line counts only where a rule below
says so, and otherwise only for the "As of" time.

- **Dispatch line:** an event that begins `dispatch`. It dispatches every
  task id written between `dispatch` and `->`, so one line may dispatch
  several tasks.
- **Status line:** an event that begins `T<n>` and then DONE,
  DONE_WITH_CONCERNS, BLOCKED or NEEDS_CONTEXT. An event that says
  `T<n> complete` in words is a DONE line for `T<n>`.
- **Review line:** an event that begins `T<n> review:` or
  `T<n> review r<k>:`. It is a review of `T<n>`, the task reviewed, and
  gives its findings as counts per severity after the `;`.
- **Fix-round line:** an event that begins `T<n> fix round`.
- **At-base and returned-to-test lines:** events that begin `T<n> at base:`
  and `T<n> returned to test:`.
- **Ruling on findings:** an event that begins
  `Ruling: T<n> open findings`. It closes the Critical and Important
  findings then open on `T<n>`, at any fix round, a whole-change review's
  included (`status-format.md`, "Open findings").
- **Go line:** an event that begins `go`.
- **Held:** an event `approval reopened for <Rn>; held T<a>[, T<b>]` holds
  those tasks until a later `spec approved by <handle>` event.

A task has **reported** when its newest status line is DONE or
DONE_WITH_CONCERNS and no dispatch line of it is newer. A task's review is
**closed** when the ledger has a review line for it and it has no open
Critical or Important finding (`status-format.md`, "Open findings").

Task states, findings and the latest result of each task come from the
ledger alone. The reports and reviews in `tasks/` are what its lines point
to.

### 1.5 Current commit set

- For each repo in the spec's `repos:` list: the head of `change/<id>` on
  its remote.
- For each other repo in `pod.yaml`: its pin.
- For the space: the content of `changes/<id>/tests/` and `tests/` at the
  space's `HEAD`.

An evidence record is **at the current commit set** when each repo's sha in
its `commit_set` matches the one above (the first 7 hex digits suffice),
and `git -C <space> diff --quiet <its space sha> HEAD -- changes/<id>/tests
tests` exits 0.

### 1.6 Latest evidence and verified

An **evidence record** is a file in `changes/<id>/evidence/` whose front
matter has an `id: ev-...` line. Nothing else there is one: not a test
task's report, `<YYYYMMDDTHHMMZ>-T<n>-report.md`, and not a log. A record
with a `shard:` key is a **shard record**; once a change has one, its
verify is sharded and 1.7 applies.

The **latest evidence** is the evidence record with the newest id,
`ev-<YYYYMMDDTHHMMZ>`. It is **green** when it holds a pass for every
result verify must give (`PROCESS.md §7`), and nothing in it is `fail` or
`not-run`:

- every scenario of the spec, by its id;
- every test file `test_*.md` in `changes/<id>/tests/`, under the key
  that section 5, step 4, maps it to, such as `I<n>` under `x-checks:`;
- every regression test: for each archived change,
  `changes/archive/<yyyy-mm-dd>-<change>/`, each row of its plan's test map
  whose test file, `<test name>.md`, is under the space's `tests/`, under
  `scenarios:` as `<change>/<Rn.Sm>`, with `<Rn.Sm>` from the row;
- each repo in the spec's `repos:` under `unit:`, as `pass` or
  `none-present`;
- every quality check the spec declares, under `quality:`. A spec that
  declares none needs `quality: none-declared`.

A missing result is not a pass: a record with a result for Q1 only, for a
spec that declares Q1 and Q2, is not green, and the change is not verified
(R12.S1). Nor is a record with no `x-checks:` while `changes/<id>/tests/`
holds `test_I1_<slug>.md`, or one with a result for one regression test of
ten.

### 1.7 Sharded verify

A sharded verify is read by the rule of `PROCESS.md §7`, the same rule the
lead applies (swe-pod `agents/swe-lead.md`, "Sharded verify").

- **Shards.** The plan tasks (1.3) whose id has a letter suffix and the
  number of a shard record's `shard:` value, such as `T19a` and `T19b` for
  `shard: T19a`, and each `shard:` value itself, in task order.
- **A shard's latest record** is the shard record whose `shard:` is that
  shard and whose id, `ev-<YYYYMMDDTHHMMZ>-<shard id>`, has the newest UTC.
  Its `tests:` lists the tests it ran, by test name.
- **Current commit set.** Each repo as in 1.5. The space: the commit that
  holds the latest shard briefs, the newest commit on the space's `HEAD`
  that changed a shard's brief:

  ```
  git -C <space> log -1 --format=%h HEAD -- changes/<id>/tasks/<shard id>-brief.md ...
  ```

  with every shard's brief path in place of the dots.
- A shard's latest record is **at the current commit set** when each repo's
  sha in its `commit_set` matches the one above (the first 7 hex digits
  suffice), its space sha is the commit that holds the latest shard
  briefs, and, as in 1.5, `git -C <space> diff --quiet <that commit> HEAD
  -- changes/<id>/tests tests` exits 0. Otherwise it is **out of date**.
- The shards' latest records are **green together** only when all three
  hold:
  - every shard has a latest record, and every one of them is at the
    current commit set;
  - nothing in any of them is `fail` or `not-run`;
  - together they hold a pass for every result 1.6 lists. A result is held
    when at least one of the records gives it.
- An out-of-date shard keeps the change unverified until it runs again at
  the current commit set. If no branch head has moved since the shard
  briefs were filed, the lead dispatches that shard again from its brief.
  If one has, the lead writes every shard's brief again, files them in one
  space commit, and dispatches every shard.

## 2. Where things stand

Answer from `AGENTS.md`, `STATUS.md` and the ledgers. Use `STATUS.md` as
committed when its "As of" UTC equals the newest one in the records
(`status-format.md`, "Header"). Otherwise work out the same content from the
records, as in section 3, without filing it, and say that `STATUS.md` is
behind.

For each open change, say in plain words its state and its next task, as
`next:` gives them, then anything blocked and any open Critical or
Important finding. For a sharded verify that is not green together (1.7),
name each shard whose latest record is out of date or missing, and say
that the change is not verified until that shard runs at the current
commit set. Give the full path to `STATUS.md` and to any record worth
opening.

## 3. Regenerate STATUS.md

1. Read the records (section 1).
2. Build the text to `status-format.md`, rule by rule.
3. Compare it with the committed `STATUS.md`. If they are the same, file
   nothing and say so.
4. Otherwise write it at the space root, and file it through the space's
   librarian: run agent-mesh's `space-librarian` skill on the space root
   and `STATUS.md` when this session holds the librarian, or else send it
   to the role that does. Report the `FILED` or `REFUSED` line.

Filing `STATUS.md` adds no ledger line. The records it is built from stay
as they were, so building it again gives the same text (R20.S1). When asked
for a dry run, or to show it without filing, stop after step 3 and print
it.

## 4. Go

"Go" and "do the next thing" mean acting as the change's lead: follow the
`swe-lead` definition (swe-pod `agents/swe-lead.md`) for that change.

1. Read the spec's header. With no `approved:` line, dispatch nothing and
   say exactly, with the id filled in:

   ```
   Not dispatched: the spec at changes/<id>/spec.md awaits approval.
   ```

2. Read the change's state and its tasks (section 1). The ready tasks are
   those whose `after` tasks are all done and which have no dispatch line
   yet (`status-format.md`, "Task states"). The first of them, in task
   order, is the next task. With none ready, a verify shard that is done
   and whose latest record is out of date (1.7) is next: it runs again at
   the current commit set, as 1.7 says.
3. As the lead, write the brief for each ready task and dispatch it, the
   next task first, with its ledger line, as the lead's definition says.
   Its fences hold: a heavy check waits for the human's word on its command,
   caps and expected wall time.
4. When no task is ready, say what the change waits on: the human's
   approval or go, the plan, or the tasks in flight.
5. File the lead's records through the librarian, then regenerate
   `STATUS.md` (section 3).

## 5. Trace

The trace walks each change's chain, requirement to task to commit to test
to evidence, and reports every link that is missing. It reads only; it
files nothing and prints its result.

1. **Requirement to task.** Once the change has a plan, each requirement,
   `### R<n>` under the spec's Requirements section, needs a build task
   whose `serves:` names it, alone or in a range `R<a> to R<b>`.
2. **Scenario to test.** Once the change has a plan, each scenario needs a
   row in the plan's test map that gives a test name. Once the task in the
   row's "Writes it" or "Written by" column is done, the test's file,
   `<test name>.md`, needs to be in `changes/<id>/tests/`, or in `tests/`
   after integrate.
3. **Task to commit.** Each build task that has reported needs at least one
   commit on its repo's `change/<id>` with both trailers, `Change: <id>` and
   `Task: T<n>`. Read them, read-only, in a local clone of the repo:

   ```
   git -C <clone> fetch origin change/<id>
   git -C <clone> log --format='%H %(trailers:key=Change,valueonly,separator=%x2C) %(trailers:key=Task,valueonly,separator=%x2C)' origin/change/<id>
   ```

4. **Test to evidence.** Once the change has evidence, each test file
   `test_*.md` in `changes/<id>/tests/` needs a result in the latest
   evidence: `test_R<n>_S<m>_<slug>` under `scenarios:` as `R<n>.S<m>`,
   `test_I<n>_<slug>` under `x-checks:` as `I<n>`, and
   `test_T<n>_TS<k>_<slug>` under `x-suggestions:` as `T<n>.TS<k>`. For a
   sharded verify, the result is needed in the latest record of a shard
   whose `tests:` lists the test (1.7).
5. **Evidence current.** The latest evidence needs to be at the current
   commit set (1.5). For a sharded verify, each shard's latest record
   needs to be at the current commit set (1.7), and each that is not is a
   break, by its id.

Print, for each change, this first line and then one line per break, in
the order of the steps above, by id within a step (`R2` before `R10`, test
names in byte order). These lines are exact:

```
TRACE <change id>: <n> breaks
BREAK requirement-without-task <Rn>
BREAK scenario-without-test <Rn.Sm>
BREAK task-without-commit T<n>
BREAK test-without-evidence <test name>
BREAK evidence-stale <ev id>
```

`<n>` is the number of BREAK lines that follow, `0` when there are none.
`<test name>` is the file name without `.md`.

## What it writes

In its own steps this skill writes one record, `STATUS.md`, and only
through the space's librarian. When it acts as the lead (section 4), the
`swe-lead` definition says what the lead writes. It writes nothing in a code
repo, and nothing outside the shareable profile: no hostname, agent id, mesh
task id, home path or credential name.
