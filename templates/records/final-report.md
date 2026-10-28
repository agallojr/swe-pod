# <id>: final report

- change: <id>
- spec: changes/<id>/spec.md
- latest evidence: ev-<YYYYMMDDTHHMMZ>

<!--
Record: changes/<id>/report.md, filed at integrate and also given in chat.
Replace every <placeholder>, then delete these comments.
-->

## Scenarios
| Scenario | Test | Result | Evidence id |
|---|---|---|---|
| R<n>.S<m> | test_R<n>_S<m>_<slug> | <pass, fail or not-run> | ev-<YYYYMMDDTHHMMZ> |

## Rulings
<!--
Every Ruling: line in the ledger, verbatim, each with its cost if wrong. The
list is exhaustive; None if the ledger has none.
-->
- <YYYY-MM-DDTHH:MMZ> Ruling: <what>; <why>; <cost if wrong>

## Deferred minors
- T<n> Minor <repo>/<path>:<line> <text>: <fixed, or deferred with the reason>

## Test suggestions
<!--
Every suggestion from every report, with its outcome from the ledger: tested
<test name>, proposed as scenario, unit-tested, declined: <reason>, or
dropped.
-->
- T<n>.TS<k> <title>: <outcome>

## Open questions
- <question>: <who can answer it>

## Branches
| Repo | Branch | Head |
|---|---|---|
| <repo> | change/<id> | <sha7> |
