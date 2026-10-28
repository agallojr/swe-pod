---
task: T<n>
diff: <repo> <sha7>..<sha7>
verdict: pass|fix
---
<!--
Record: changes/<id>/tasks/T<n>-review.md. The whole-change review uses this
format under its own task number. Write one diff: line per range, one for
each repo reviewed. verdict is fix while any Critical or Important finding is
open. Replace every <placeholder>, then delete these comments.
-->

## Spec compliance
- missing: <what the brief asks for that the diff lacks, or none>
- extra: <what the diff adds that the brief does not ask for, or none>
- misread: <where the diff reads the brief or spec differently, or none>

## Quality
- structure: <notes>
- error handling: <notes>
- tests: <whether the tests check real behavior>
- test suggestions: <for each T<n>.TS<k>, whether the unit tests cover its
  for-the-reviewer part>

## Findings
<!-- One line per finding, most severe first; or None. -->
- <Critical|Important|Minor> <repo>/<path>:<line> <text>
