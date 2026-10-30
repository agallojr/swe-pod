# <id>: plan

- change: <id>; spec approved <YYYY-MM-DDTHH:MMZ>
- size: <small|project>
- planner: <handle>/plan
- base commit set: {space: <sha7>, <repo>: <sha7>, ...}
- code branches: change/<id> in <repo>[, <repo>]

<!--
Record: changes/<id>/plan.md. Write for an engineer who has never seen the
code or the spec; hold the decisions they cannot make alone. Sections 1 to 10
cover all downstream work as a dependency graph. The change is planned once
Self-review is filled in. Replace every <placeholder>, then delete these
comments.
-->

## 1. Goal and approach
<two or three sentences>

## 2. Interface contract
<the public surface the project-level tests drive: signatures with types, CLI
flags, file formats, output fields, exact names>

## 3. Global constraints
<copied verbatim from the spec: units, tolerances, version floors, naming
rules>

## 4. File map
| Repo | File | Create or change | One responsibility | Task |
|---|---|---|---|---|
| <repo> | <path> | <create or change> | <one responsibility> | T<n> |

## 5. Tasks
<!--
One entry per build task: the smallest unit with its own test cycle that a
reviewer could reject while approving its neighbor. Steps pin exact values and
assertions, and give a function body only for an algorithm the signature and
tests leave open. Each entry has a risk line. A builder's test is a unit test
or, in a repo with no unit suite, a check in the build task's done-when.
-->
### T<n> <title>
- role: build
- repo: <repo>
- after: [T<n>, ...]
- runs: <local|mesh>
- needs: <environment class|any>
- serves: [R<n>, ...]
- risk: <at risk|small>: <reason>
- touches: <path>, ...
- consumes: <interface> from T<n>
- produces: <interface>
- steps: <test first, watch it fail, minimal code, watch it pass, commit;
  with exact values and assertions>
- done when: <commands and their expected output>, and the review that
  covers it has no Critical or Important finding

## 6. Test tasks
<!--
Every test the plan lists names the risk it covers, and the plan lists no
test without one. First the risk table, one row per requirement; then the
expensive runs: every agent-class or live-class run the plan lists, with its
wall time in s, its spend in USD and its caps, and a Total row. A script-class
check gives its expected run time only, in the test map. An estimate is not a
quota; the ask point is twice it. A requirement at risk gets blind
project-level tests from the test-writing task; one not at risk gets only its
builder's tests. A plan that marks no requirement at risk has no test-writing
task: leave that entry out. A live or heavy test is listed only to reproduce
a reported defect and confirm its fix, one run at base and one at the branch
head, or with the reason no cheaper check can show the behavior.
-->
Risk:

| Requirement | Risk | Reason |
|---|---|---|
| R<n> | <at risk|small> | <the reason> |

Expensive runs:

| Run | Class | Task | Runs | Wall time (s) | Spend (USD) | Caps |
|---|---|---|---|---|---|---|
| <run> | <agent|live> | T<n> | <k> | <s> | <USD> | <caps> |
| Total | | | <k> | <s> | <USD> | |

Twice the estimate, the ask point: <s> s or <USD> USD.

### T<n> Project-level tests of <the requirements at risk>
- role: test
- after: []
- runs: <local|mesh>
- needs: <environment class|any>
- serves: [R<n>, ...]
- done when: one test per scenario of the requirements at risk in the test
  map, each with its cost class, expected_s, deliverables and the risk it
  covers, each run at the base commit set, with its result in the ledger

### T<n> Builder test suggestions
- role: test
- after: [<every build task>]
- runs: <local|mesh>
- needs: <environment class|any>
- done when: every suggestion the lead routed to the test agent, for a risk
  no planned test covers, is tested or declined with a reason; with none
  routed, the task does not run

### T<n> Verify (<environment class>)
- role: test
- after: [<every build and test task>]
- runs: <local|mesh>
- needs: <the environment class the tests require>
- done when: an evidence record with selection: targeted and touched:, with a
  result for every scenario (the done-when checks rerun), each affected
  repo's unit suite and every declared quality check, and for every
  regression test its result, not-selected when the selection does not pick
  it

## 7. Review tasks
<!--
The plan has one review per wave of build tasks, a review of one build task
only where its entry has `- risk: at risk`, and the whole-change review. A
wave is a group of build tasks the dependency summary runs together, and its
review covers those of them not reviewed alone. A plan whose build tasks form
one wave has one review, the whole-change review. Then leave out the other
two entries, and the whole-change review comes after the build tasks and
before verify.
-->
### T<n> Review of wave <w>
- role: review
- after: [<the build tasks of wave w not reviewed alone>]
- runs: <local|mesh>
- needs: any
- done when: a review record over those tasks' diffs from base to head

### T<n> Review of T<m>
- role: review
- after: [T<m>], a build task whose entry has risk: at risk
- runs: <local|mesh>
- needs: any
- done when: a review record for T<m>'s diff from base to head

### T<n> Whole-change review
- role: review
- after: [<verify task>], at its first green run; with one wave, [<every
  build task>], before verify
- runs: <local|mesh>
- needs: any
- done when: a review record over the spec, the plan, every repo's diff, the
  latest evidence, and the ledger's deferred minors and rulings

## 8. Integrate task
### T<n> Integrate
- role: lead
- after: [<verify task>, <whole-change review>]
- runs: local
- needs: any
- docs: <docs this change touches>
- open plans: <open plans that depend on this change>
- tests: <the kept tests to promote, and the superseded or stale tests to
  move out>
- done when: merged in depends order on the human's word; kept tests
  promoted, superseded and stale tests moved out; pins advanced; final
  report filed with its test cost

## 9. Test map
<!--
One row per scenario, and one per regression test the plan names for a risk,
keyed <change>/<value>. Class is script, agent or live. Keep is `keep` or
`not kept: <reason>`. A scenario checked in a build task's done-when has Test
T<n> done-when, Written by T<n>, and needs no test file.
-->
| Scenario | Test | Class | Risk it covers | Expected (s) | Keep | Written by | Unit tests that also cover it |
|---|---|---|---|---|---|---|---|
| R<n>.S<m> | test_R<n>_S<m>_<slug> | <script|agent|live> | <at risk: the part of the risk it covers> | <s> | keep | T<n> | <repo>/<path> |
| R<n>.S<m> | T<n> done-when | script | <small: the risk it covers> | <s> | not kept: a builder's check | T<n> | none |
| <change>/<value> | <test name> | <agent|live> | <the risk of this change it covers> | <s> | already in tests/ | <change> | none |

## 10. Dependency summary
- parallel: <the task groups that run at the same time>
- longest chain: T<a> -> T<b> -> ...
- shared files: <path>: T<a> before T<b>, never at the same time

## Self-review
- [ ] every requirement maps to a build task
- [ ] every scenario maps to a test
- [ ] no step says "TBD", "handle edge cases" or "add tests"
- [ ] names and types agree across tasks
- [ ] every task has role, after, runs and done-when
- [ ] every planned test names the risk it covers
- [ ] every agent- and live-class run has its wall time, spend and caps, with the totals
- [ ] a live or heavy test is listed only to reproduce a reported defect and confirm its fix, or with the reason no cheaper check can show the behavior
- [ ] the reviews follow section 7: one per wave, one per build task at risk, and the whole-change review

## Questions for the spec
<!--
Sort each question into one group. Group (c) goes to the human in one
message, most important first.
-->
### (a) Answered
1. <question>: <the answer, with the spec clause>

### (b) Minor, with the default planned
1. <question>: <the default planned, as listed under the spec's Open calls>

### (c) For the human
1. <question>: <why the answer changes what gets built>
