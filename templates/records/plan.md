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
tests leave open.
-->
### T<n> <title>
- role: build
- repo: <repo>
- after: [T<n>, ...]
- runs: <local|mesh>
- needs: <environment class|any>
- serves: [R<n>, ...]
- touches: <path>, ...
- consumes: <interface> from T<n>
- produces: <interface>
- steps: <test first, watch it fail, minimal code, watch it pass, commit;
  with exact values and assertions>
- done when: <commands and their expected output>, and the task review has
  no Critical or Important finding

## 6. Test tasks
### T<n> Project-level tests <scope>
- role: test
- after: []
- runs: <local|mesh>
- needs: <environment class|any>
- serves: [R<n>, ...]
- done when: one test per scenario in the test map, each run at the base
  commit set, with its result in the ledger

### T<n> Builder test suggestions
- role: test
- after: [<every build task>]
- runs: <local|mesh>
- needs: <environment class|any>
- done when: every suggestion routed to the test agent is tested or declined
  with a reason

### T<n> Verify (<environment class>)
- role: test
- after: [<every build and test task>]
- runs: <local|mesh>
- needs: <the environment class the tests require>
- done when: an evidence record with a result for every scenario, the
  regression suite, each affected repo's unit suite and every declared quality
  check

## 7. Review tasks
### T<n> Review of T<m>
- role: review
- after: [T<m>]
- runs: <local|mesh>
- needs: any
- done when: a review record for T<m>'s diff from base to head

### T<n> Whole-change review
- role: review
- after: [<verify task>], at its first green run
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
- done when: merged in depends order on the human's word; tests promoted;
  pins advanced; final report filed

## 9. Test map
| Scenario | Test | Written by | Unit tests that also cover it |
|---|---|---|---|
| R<n>.S<m> | test_R<n>_S<m>_<slug> | T<n> | <repo>/<path> |

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
