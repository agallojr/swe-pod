# STATUS.md format

`STATUS.md` sits at the root of a Project Space. The `swe-pod` skill builds
it from the records, as read in `SKILL.md` section 1, and nobody edits it by
hand (`PROCESS.md §15`). This file fixes its text, so two agents that build
it from the same records write the same bytes.

## Layout

The lines below are exact. Text in `<angle brackets>` is filled in by the
rules that follow; everything else is copied as written.

```text
# Status: <space id>

Generated from the records; never edited by hand. As of <UTC of the newest ledger line or record>.

- pins: <repo> <sha7>, ...
- open changes: <n> (<state>: <k>, ...)

## <id>   state: <state>   owner: <handle>

- gates: <gate> <handle> <UTC>, ...
- branches: change/<id>: <repo> <sha7>, ...
- tasks: waiting <k>, ready <k>, dispatched <k>, done <k>, blocked <k>

| task | role | runs | state | latest |
|---|---|---|---|---|
| T<n> | <role> | <local or mesh> | <task state> | <UTC> <event> |

- latest evidence: <ev id>, <env>, <green|not green>: <results>
- open findings: Critical <n>, Important <n>, Minor <n>
- next: <next>
```

One `## <id>` section follows the header for each open change, in byte
order of the change ids, each with its bullets and task table in the order
shown. For a sharded verify, the `latest evidence` bullet takes the sharded
form given under "Per open change", with one indented line per shard.
Blocks are separated by one empty line. No line has trailing spaces, and
the file ends with one newline.

## Header

- **Space id:** the `id:` value in `space.yaml`, such as `proj:demo`.
- **As of:** the newest UTC, as `YYYY-MM-DDTHH:MMZ`, among:
  - every ledger line under `changes/`, `changes/archive/` included;
  - the UTC that begins the name of each record under
    `changes/*/evidence/`, `handoffs/` and `decisions/`, written back with
    its dashes and colon (`20261102T1412Z` is `2026-11-02T14:12Z`);
  - the UTC on each `approved:`, `integrated:` or `abandoned:` line in a
    spec's header (`SKILL.md` section 1.2).

  With none of these yet, the line ends `As of no records yet.`
- **pins:** each repo under `repos:` in `pod.yaml`, in the order written,
  as `<repo> <sha7>`: its name and the first 7 hex digits of its `pin`,
  joined by `, `.
- **open changes:** `<n>` is the number of changes in state proposed,
  approved, planned, building or verified. The parenthesis lists each of
  those states that has at least one change, in that order, as
  `<state>: <k>`, joined by `, `. With no open change the line is
  `- open changes: 0`.

## Per open change

- **Heading:** the change id, its state (`SKILL.md` section 1.2) and the
  handle on the `owner:` line of the spec's header, separated as shown by
  three spaces.
- **gates:** the gates passed, in this order:
  - `spec <handle> <UTC>` for each `approved:` line in the spec's header,
    in the order written;
  - `go <handle> <UTC>` for the ledger's go line (`go by <handle>; ...`).
    A go line that names no handle gives `go <UTC>`.

  With none passed, the bullet is `- gates: none`.
- **branches:** before the go line, `- branches: none yet`. After it, each
  repo in the spec's `repos:` list, in that order, as `<repo> <sha7>`, the
  head of `change/<id>` by the Branch head rule (`SKILL.md` section 1.1):
  read from the remote when the remote can be read, and from the records
  when it cannot. A repo with no head by that rule is `<repo> missing`.
- **tasks:** the number of tasks in each task state, in the order shown,
  zeros included. Then an empty line and the task table.
- **latest evidence:** `- latest evidence: none` until the change has an
  evidence record. Then the newest record (`SKILL.md` section 1.6) as
  `<ev id>, <env>, <green|not green>`, by the green rule of `SKILL.md`
  section 1.6, followed by `, stale` when it is not at the current commit
  set, then `: ` and its results, joined by `, `:
  - `scenarios <p>/<n> pass`, where `<n>` counts the spec's scenarios and
    `<p>` those the record passes;
  - `regression <p>/<n> pass[, <k> not-selected][, <j> superseded]` over
    the regression tests (`SKILL.md` section 1.6), only when the space has
    at least one. `<n>` counts the regression tests that need a pass,
    `<p>` their passes, and `<k>` and `<j>` the regression tests whose
    `not-selected` or `superseded` result is no bar; each bracketed part
    only when its count is not 0. A record that passes 6 picked tests and
    gives the other 84 `not-selected` shows
    `regression 6/6 pass, 84 not-selected`;
  - `unit <repo> <pass|fail|none-present|missing>` for each repo in the
    spec's `repos:` list, in that order;
  - `quality none-declared`, or `quality <p>/<n> pass` over the checks the
    spec declares;
  - `x-checks <p>/<n> pass` and `x-suggestions <p>/<n> pass`, where `<n>`
    counts the test files in `changes/<id>/tests/` that map to that key
    (`SKILL.md` section 5, step 4), each only when `<n>` is not 0.

  For a sharded verify (`SKILL.md` section 1.7), the bullet shows each
  shard's latest record, one indented line per shard, in task order:

  ```text
  - latest evidence: sharded, <green|not green>: <results>
    - <shard id>: <ev id>, <env>, <green|not green>[, stale]: <results>
    - <shard id>: none
  ```

  - The first line is `green` when the shards' latest records are green
    together (`SKILL.md` section 1.7). Its results are those above, over
    the latest records taken together: a result is a pass when one of them
    passes it and none gives it a bar (`SKILL.md` section 1.7); a
    regression test with no pass counts toward `<k>` or `<j>` when one of
    them gives it a `not-selected` or `superseded` result that is no bar
    and none gives it a bar. A repo's `unit` is `fail` when one of them
    gives `fail`, else the first of `pass` and `none-present` that one of
    them gives, else `missing`.
  - A shard's line is `<shard id>: none` until the shard has a record.
    Then it gives that record's id and `env`, `green` when no result in it
    is a bar (a missing result aside), and `, stale` when it is out of
    date. Its
    results are those above, but each `<n>` counts only the results of
    that kind the record gives, and a kind it gives none of is left out:
    `unit` for each repo the record lists, in the spec's `repos:` order,
    and `quality none-declared` when the record says so.
- **open findings:** the totals of section "Open findings", below.
- **next:** one of these, by the change's state:

  | State | `next:` |
  |---|---|
  | proposed | `next: approval of changes/<id>/spec.md, by the human` |
  | approved | `next: the plan, by the plan role` |
  | planned | `next: the go, by the human` |
  | building, verified | the first ready task, as `next: T<n> -> <role> (<local or mesh>, <env class or any>)`, with `runs:` and `needs:` from the plan |

  When a building or verified change has no ready task, the first of these
  that holds gives the bullet:
  - a verify shard is done and its latest record is out of date (`SKILL.md`
    section 1.7): `next: re-run T<a>[, T<b>] at the current commit set`,
    naming each such shard in task order;
  - every task is done: `next: none; every task is done`;
  - otherwise: `next: none ready; dispatched: T<a>, T<b>`, adding
    `; blocked: T<c>` when any task is blocked and leaving out a list that
    would be empty.

## The task table

One row per plan task (`SKILL.md` section 1.3), in task order: by number,
with a letter suffix after the bare number (`T19`, `T19a`, `T19b`, `T20`).

- **task:** the id, `T<n>`.
- **role:** the first word of the task's `role:` field.
- **runs:** `local` or `mesh`, from its `runs:` field.
- **state:** one of the five task states, below.
- **latest:** the task's newest event, as below, or `-` when it has none.

### Task states

Decide each task's state by the first of these that holds. The ledger terms
(reported, review closed, held) are defined in `SKILL.md` section 1.4.

1. **done.**
   - A build task: it has reported, and its review is closed.
   - A review task: the review of the task it reviews (`SKILL.md` section
     1.3) is closed. One that reviews no single task, such as the
     whole-change review, counts its own review lines.
   - A test or lead task: it has reported.
2. **blocked.** Its newest status line, later than its newest dispatch
   line, is BLOCKED or NEEDS_CONTEXT.
3. **dispatched.** A dispatch line names it. A review task too is
   dispatched only by its own dispatch line.
4. **ready.** It is not held, and every task in its `after` list is done.
   A review task that reviews one task is ready instead once that task has
   reported and every other task in its `after` list is done, since the
   review is part of that task's done-when.
5. **waiting.** Anything else, held tasks included.

### Latest

A task's events are the dispatch, status, fix-round, at-base and
returned-to-test lines that name it. A review task's events are its own
dispatch lines and the review lines of the task it reviews (its own review
lines, when it reviews no single task). The newest is the one with the
latest UTC, and of those the one written last. The cell is its UTC, a
space, and the event in this form:

| Event | Cell text after the UTC |
|---|---|
| dispatch | `dispatch -> <role> (<local or mesh>, <env class or any>)` |
| status | the status word, then `, <repo> <sha7>..<sha7>` and `, unit <p>/<n> pass` when the line has them in that form; nothing else |
| in words, that the task is finished, done or complete (`SKILL.md` section 1.4) | `DONE` |
| review | `review: ` or `review r<k>: `, then `spec ok` or `spec issues`, then `; ` and its counts as `<k> <Severity>` in the order Critical, Important, Minor, joined by `, `, without file and line. Leave out each count of 0. When no count is left, or the line gives none, write `none`: `spec ok; 0 Important` gives `review: spec ok; none` |
| fix round | `fix round <r>/5: <a> addressed, <o> open` |
| at base | `at base: <f> fail, <p> pass`, counting the tests the line names |
| returned to test | `returned to test: <test name>` |

A `|` inside a cell is written `\|`.

## Open findings

The ledger's review and fix-round lines give the counts.

- **Critical and Important.** For each task that has a review line, take
  its newest review or fix-round line. A review line gives its Critical and
  Important counts. A fix-round line gives its `<o> open` findings, counted
  as Critical up to the number of Critical findings in the review before
  it, and the rest as Important. A ruling closes them at any round: when a
  `Ruling:` line newer than that line begins `Ruling: T<n> open findings`,
  as in `Ruling: T<n> open findings parked: <which>; <why>; <cost if wrong>`,
  the task counts 0. A `Ruling:` line in another form closes nothing. Add
  the counts over all tasks.
- **Minor.** The sum of the Minor counts of every review line of the
  change. Minor findings wait for the whole-change review and the final
  report's deferred minors, so none closes before integrate.

## Same records, same text

- Nothing comes from the wall clock, the session, the node or the space's
  own commit sha. Every UTC is copied from a record.
- Every list follows a fixed order given above: `pod.yaml` order for pins,
  the spec's order for repos and approvals, byte order for change ids, task
  order for tasks.
- Every empty case has its fixed text: `none`, `none yet`, `-`, `0`.
- The records are read as committed (`SKILL.md` section 1.1), and branch
  heads by its Branch head rule, from the remotes or, where a remote cannot
  be read, from the records, so the text changes only when a record, a pin
  or a branch head changes.

## Example

A space `proj:demo` with one change, `add-greeting`, in one repo, at a
fictional state:

```markdown
# Status: proj:demo

Generated from the records; never edited by hand. As of 2026-11-02T15:20Z.

- pins: greeter 77b01d4
- open changes: 1 (building: 1)

## add-greeting   state: building   owner: ana

- gates: spec ana 2026-11-01T15:20Z, go ana 2026-11-02T14:00Z
- branches: change/add-greeting: greeter 8e1d0c4
- tasks: waiting 3, ready 0, dispatched 2, done 1, blocked 0

| task | role | runs | state | latest |
|---|---|---|---|---|
| T1 | build | local | dispatched | 2026-11-02T14:41Z DONE, greeter 77b01d4..8e1d0c4, unit 14/14 pass |
| T2 | test | mesh | done | 2026-11-02T15:20Z at base: 2 fail, 0 pass |
| T3 | review | local | dispatched | 2026-11-02T14:50Z review: spec ok; 1 Important |
| T4 | test | mesh | waiting | - |
| T5 | review | local | waiting | - |
| T6 | lead | local | waiting | - |

- latest evidence: none
- open findings: Critical 0, Important 1, Minor 0
- next: none ready; dispatched: T1, T3
```
