# T<n> brief

- change: <id>
- task: T<n>
- role: test

<!--
Record: changes/<id>/tasks/T<n>-brief.md, for a test-writing task. It holds
the spec path, the interface contract, the global constraints and the test
map, and no implementation body, diff, builder report or build-task step.
Replace every <placeholder>, then delete these comments. The last line stays
exactly as written.
-->

## Task
<this test task's own plan entry, verbatim>

## Spec
changes/<id>/spec.md

## Interface contract
<the plan's interface contract, verbatim>

## Global constraints
<the plan's global constraints, verbatim, operator decisions included>

## Test map
<the plan's test map, verbatim>

## Start from
- base commit set: {space: <sha7>, <repo>: <sha7>, ...}
- commit set under test: {space: <sha7>, <repo>: <sha7>, ...}

## Suggestions
<!--
The suggestions task only; leave this section out of every other test-writing
brief. Each suggestion carries its for-the-tester part alone, in spec terms.
-->
### T<n>.TS<k> <title>
- for the tester: <the requirement or scenario at risk, the input, and the
  failure the builder expects>

## Result goes to
<!-- Keep the lines that apply to this task. -->
- tests: changes/<id>/tests/
- at base: each new test's result at the base commit set, with its reason,
  for the ledger
- evidence: changes/<id>/evidence/<YYYYMMDDTHHMMZ>.md (a verify task)

Do not read plan.md, ledger.md, any other brief, report or review, or any code-repo diff or implementation body.
