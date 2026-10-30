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
  log (section 5). In sections 2, 3 and 5, never commit, push, check out,
  rebase or create a branch in a code repo. Section 4 acts as the lead, and
  there the lead's own definition says what it does in a code repo.
- **Branch head.** The head of `change/<id>` in a repo is given by the
  first of these that holds:
  1. the remote can be read (`git ls-remote` exits 0) and lists the
     branch: the sha it lists;
  2. the remote can be read and lists no such branch: none, written
     `<repo> missing`;
  3. the remote cannot be read (`git ls-remote` exits non-zero, as for a
     placeholder URL or a host that does not resolve): taken from the
     records. It is the end sha of the newest ledger range for that repo,
     in the event order of 1.4. A range for that repo is one written
     `<repo> <sha7>..<sha7>` on any task's line, or a fix-round line's
     bare `<sha7>..<sha7>` when the task's plan entry names that repo
     alone in `repo:`; a bare range of a task whose entry names several
     repos is a range for none of them. With no such range, it is that
     repo's sha on the go line. With neither, it is none, written
     `<repo> missing`.

  Rule 3 reads only committed records, so two agents that cannot read a
  remote give the same head. Every rule that needs a branch head (1.5,
  `status-format.md`) takes it from here.

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

A **review task** is a task whose role is review. A **build task** is one
whose role is build. A review task reviews the build tasks given by the
first of these it has:

1. its Reviews cell, in a review table: the task id the cell begins with;
2. its heading, `### T<n> Review of T<m>`: T<m>;
3. its `after:` list: every build task it names, one or more.

So a wave's review, `### T<n> Review of wave <w>`, reviews every build task
its `after:` list names, and so does the whole-change review, the one whose
heading title is `Whole-change review`, in a plan whose build tasks form one
wave: its `after:` list names them (`PROCESS.md §4`). A review task with
none of these, such as a whole-change review whose `after:` list names only
the verify task, reviews no build task.

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
  DONE_WITH_CONCERNS, BLOCKED or NEEDS_CONTEXT. An event that says in
  plain words that `T<n>` is finished, done or complete, such as
  `T0 complete` or `..., so T0 is finished`, is a DONE line for `T<n>`;
  one that says it is not, or not yet, is not.
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
- **Harness-error line:** an event that begins `T<n> harness error:`.
- **Not-dispatched line:** an event that begins `T<n> not dispatched:`. The
  task waits for the operator's word until a newer dispatch line names it.
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

- For each repo in the spec's `repos:` list: the head of `change/<id>`, by
  the Branch head rule (1.1). A repo with none matches no record.
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
`ev-<YYYYMMDDTHHMMZ>`. Verify must give a result for each of these
(`PROCESS.md §7`):

- every scenario of the spec, by its id;
- every test file `test_*.md` in `changes/<id>/tests/`, under the key
  that section 5, step 4, maps it to, such as `I<n>` under `x-checks:`;
- every regression test: each file `test_*.md` directly in a folder
  `tests/<change>/` of the space, where integrate puts a change's tests
  (`PROCESS.md §8`), and each one directly in `tests/` itself. A
  `<change>` folder counts only when `changes/archive/<yyyy-mm-dd>-<change>/`
  exists. Nothing deeper counts: fixture data such as
  `tests/<change>/fixtures/` holds no test of the suite. A test's name is
  its file name without `.md`, and its key is `<change>/<value>`:
  - `<change>`: for a test in `tests/<change>/`, that folder's name. For
    one directly in `tests/`, the archived change whose plan test map has
    a row with that test name, else the one whose final report's
    `## Scenarios` table has such a row; where several do, the one whose
    archive folder name sorts last. With neither, the change of a
    `scenario:` value that is one id written `<change>/<value>`.
  - `<value>`: the scenario id in that change's row, from its archived
    plan test map, else from its archived report's `## Scenarios` table.
    With no such row, the test's `scenario:` front-matter value, less any
    `<change>/` part, when that value is exactly one id. When it is not
    (it lists several ids, or the test has none), the key its file name
    maps to by section 5, step 4 (`test_I1_<slug>` gives `I1`). So each
    test has at most one key.

  A key `<change>/R<n>.S<m>` is looked for under `scenarios:`,
  `<change>/I<n>` under `x-checks:` and `<change>/T<n>.TS<k>` under
  `x-suggestions:`. A test with no key still counts, and its result is
  missing;
- each repo in the spec's `repos:` under `unit:`, as `pass` or
  `none-present`;
- every quality check the spec declares, under `quality:`. A spec that
  declares none needs `quality: none-declared`.

**Results.** A result is one of `pass`, `fail`, `not-run (cap <n> s)`,
`harness-error`, `not-selected` or `superseded by <key>`, where `<key>` is a
result key of this change in the same record (`R1.S1`, `I1`): `R<n>.S<m>`
under `scenarios:`, `I<n>` under `x-checks:` and `T<n>.TS<k>` under
`x-suggestions:`.

**Selection** (`PROCESS.md §7`). The record's `selection:` is `targeted` or
`full`; a record with no `selection:` key is read as `full`. Under
`selection: targeted`, a regression test is picked when either holds:

- it is script class and one of its deliverable paths (less any
  ` as <name>`) is a `touched` path of the same repo, or a folder that holds
  one;
- it is agent or live class and the plan's test map has a row whose Test
  cell is its test name.

Under `selection: full`, every regression test is picked. Tests in another
project's space are never picked. To apply the rule:

- **Cost class.** A test's cost class is its front-matter `class:` value
  when it has one. Else it comes from `mode:`: `inspect` or `script` is
  script class; a mode that begins `agent` or `desk` is agent class; a mode
  that begins `live` is live class. A test with neither is live class, and
  the evidence names it under `## Undeclared cost class`.
- **Deliverable paths.** The `<repo>:<path>` entries of the test's
  front-matter `deliverables:` list, each less any ` as <name>`. A path is
  a folder that holds a touched path of the same repo when that touched
  path begins with it and a `/`, or with it alone when it ends in `/`. A
  path that is empty or `.` names the whole repo, which holds every
  touched path of that repo.
- **Touched paths.** For each repo in the spec's `repos:` whose remote can
  be read (1.1), the files its diff from the go line's base to the branch
  head (1.1) changes, read in a local clone:

  ```
  git -C <clone> fetch origin change/<id>
  git -C <clone> diff --name-only <base> <head>
  ```

  with `<base>` that repo's sha on the go line and `<head>` its branch
  head. A record whose `touched:` leaves out a file of that diff is not
  green. For a repo whose remote cannot be read (a fixture's placeholder
  remote), the record's `touched:` entries for that repo.
- **Named for a risk.** The plan's test map is its section 9 table; a test
  is named there when a row's Test cell, the second, is its test name.

**Green.** The latest evidence is **green** when every result verify must
give is a pass, with two results that are no bar:

- `not-selected`, when the record's `selection:` is `targeted` and the test
  is not picked;
- `superseded by <key>`, when `<key>` holds `pass` in the same record and
  the spec's `supersedes` lists the superseded test's requirement:
  `<change>/R<n>` for a test keyed `<change>/R<n>.S<m>`, and for a test
  keyed `<change>/I<n>` or `<change>/T<n>.TS<k>`, any requirement
  `<change>/R<n>`.

Every other result is a bar: `fail`, `not-run`, `harness-error`, a missing
result, a `not-selected` test that the selection picks, and a
`superseded by` result that lacks either condition. The two results that
can be no bar are regression-test results: on a scenario of the spec, a
unit suite, a quality check or a test in `changes/<id>/tests/`, either one
is a bar. A record is green only when no result in it is a bar.

A missing result is not a pass: a record with a result for Q1 only, for a
spec that declares Q1 and Q2, is not green, and the change is not verified
(R12.S1). Nor is a record with no `x-checks:` while `changes/<id>/tests/`
holds `test_I1_<slug>.md`, or one with a result for one regression test of
ten. Nor is a record that gives `not-selected` to a script-class regression
test reading a file the diff touches, or to an agent-class test the plan's
test map names, or that gives a picked test `not-run (cap 600 s)`.

### 1.7 Sharded verify

A sharded verify is read by the rule of `PROCESS.md §7`, the same rule the
lead applies (swe-pod `agents/swe-lead.md`, "Sharded verify").

- **Shards.** The plan tasks (1.3) whose id has a letter suffix and the
  number of a shard record's `shard:` value, such as `T19a` and `T19b` for
  `shard: T19a`, and each `shard:` value itself, in task order.
- **A shard's latest record** is the shard record whose `shard:` is that
  shard and whose id, `ev-<YYYYMMDDTHHMMZ>-<shard id>`, has the newest UTC.
  Its `tests:` lists the tests it ran, by test name.
- **Current commit set.** Each repo as in 1.5. The space: the shards'
  space commit, the commit that filed the last of the latest shard briefs.
  The lead files the shards' dispatch lines before their briefs, so it is
  also the last of the shard briefs and dispatch lines, and the lead gives
  it to every shard at dispatch (`PROCESS.md §11`). It is the newest commit
  on the space's `HEAD` that changed a shard's brief:

  ```
  git -C <space> log -1 --format=%h HEAD -- changes/<id>/tasks/<shard id>-brief.md ...
  ```

  with every shard's brief path in place of the dots.
- A shard's latest record is **at the current commit set** when each repo's
  sha in its `commit_set` matches the one above (the first 7 hex digits
  suffice), its space sha is the shards' space commit, and, as in 1.5,
  `git -C <space> diff --quiet <that commit> HEAD -- changes/<id>/tests
  tests` exits 0. Otherwise it is **out of date**: a repo moved, it ran at
  another space commit, or `changes/<id>/tests/` or `tests/` changed after
  that commit.
- The shards' latest records are **green together** only when all three
  hold:
  - every shard has a latest record, and every one of them is at the
    current commit set;
  - none of them gives a result that is a bar by 1.6, a missing result
    aside, since each shard gives only what it ran. Each `not-selected` is
    judged by its own record's `selection:`, and each `superseded by <key>`
    result's `<key>` is looked for in all of them;
  - together they give a pass for every scenario, unit suite, declared
    quality check and test of the change, and for every regression test a
    pass or a result that is no bar. A result is given when at least one
    of the records gives it.
- An out-of-date shard keeps the change unverified until it runs again at
  the current commit set. The lead (swe-pod `agents/swe-lead.md`, "Sharded
  verify"):
  - if the commit set has moved since the shards' space commit, through a
    branch head or the tests, writes every shard's brief again, with the
    new commit set under test in `## Start from`, and dispatches every
    shard at the new shards' space commit;
  - if it has not moved, dispatches the shard again from its brief, at the
    same shards' space commit. That covers a shard whose record names
    another commit set, and a shard whose run failed for a flaky or
    environmental reason;
  - runs a shard that failed on the code or on a test again only once the
    fix lands, which moves the commit set, so every shard runs again.

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
`swe-lead` definition (swe-pod `agents/swe-lead.md`) for that change. Its Go
step applies as written: at the go it makes and pushes each `change/<id>`
itself when the pod commit scope allows it, and otherwise gives the human
the commands.

1. Read the spec's header. With no `approved:` line, dispatch nothing and
   say exactly, with the id filled in:

   ```
   Not dispatched: the spec at changes/<id>/spec.md awaits approval.
   ```

2. Read the change's state and its tasks (section 1). The ready tasks are
   those in state ready (`status-format.md`, "Task states"): not held,
   with no dispatch line yet, and whose `after` tasks are all done, or, for
   a review task that reviews build tasks (1.3), whose build tasks have all
   reported and every other task in its `after` list is done. The first of
   them, in task order, is the next task. With none ready, a verify shard
   that is done and whose latest record is out of date (1.7) is next: it
   runs again at the current commit set, as 1.7 says.
3. As the lead, write the brief for each ready task and dispatch it, the
   next task first, with its ledger line, as the lead's definition says.
   Its fences hold: an agent-class or live-class run the plan does not
   list, or one past twice the plan's estimate, waits for the operator's
   word (`PROCESS.md §11`).
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
   row in the plan's test map that gives a test name. When that name is
   `T<n> done-when`, the row needs no test file: the check is in that build
   task's done-when, and verify reruns it. Otherwise, once the task in the
   row's "Writes it" or "Written by" column is done, the test's file,
   `<test name>.md`, needs to be in `changes/<id>/tests/`; after integrate,
   in `tests/<id>/` when the row's Keep cell is `keep`, and in the archive
   folder's `tests/` when it is not.
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
through the space's librarian, and it writes nothing in a code repo. When
it acts as the lead (section 4), the `swe-lead` definition says what the
lead writes, each `change/<id>` it makes at the go included. It writes
nothing outside the shareable profile: no hostname, agent id, mesh task id,
home path or credential name.
