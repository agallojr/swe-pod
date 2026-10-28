# T<n> brief

- change: <id>
- task: T<n>
- role: <role>
- repo: <repo>

<!--
Record: changes/<id>/tasks/T<n>-brief.md, for a build or review task. A
test-writing task uses test-brief.md instead. The same brief serves local and
mesh dispatch. Replace every <placeholder>, then delete these comments.
-->

## Task
<the task's plan entry, verbatim>

## Interfaces
<the parts of the plan's interface contract this task builds to or consumes,
verbatim>

## Global constraints
<the plan's global constraints, verbatim>

## Rulings
<every ledger Ruling: line that bears on this task, verbatim; or None yet.>

## Start from
- <repo>: branch change/<id> at <sha7>
- space: main at <sha7>

## Result goes to
- code: commits on change/<id> in <repo>, trailers Change: <id> and
  Task: T<n>, pushed
- report: changes/<id>/tasks/T<n>-report.md
