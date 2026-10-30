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
<!--
Every Minor finding of every review. A review whose open findings are all
Minor starts no fix round: each is fixed in the whole-change fix wave or
parked by a ruling.
-->
- T<n> Minor <repo>/<path>:<line> <text>: <fixed in the whole-change fix wave, or parked by the ruling at <UTC>>

## Test suggestions
<!--
Every suggestion from every report, with its outcome from the ledger: tested
<test name>, proposed as scenario, unit-tested, declined: <reason> (declined:
covered by <test name> when a planned test covers its risk), or dropped. A
suggestion routed to the test agent names the risk it covers.
-->
- T<n>.TS<k> <title>: <outcome>

## Open questions
- <question>: <who can answer it>

## Branches
| Repo | Branch | Head |
|---|---|---|
| <repo> | change/<id> | <sha7> |

## Test cost
<!--
The plan's estimate beside the actual wall time in s and spend in USD of the
expensive runs, every agent-class or live-class run, with the ratio of actual
to estimate rounded to 0.1. When some expensive runs have no known spend, the
Actual spend cell gives the known sum and unknown for <k> of <n> runs; with
none known, unknown. The same holds for an Estimate cell no one estimated.
Never 0 USD for a run whose spend no one knows. The counts are of test agents
and review agents started, fix rounds run, and ask: lines in the ledger.
-->
| Expensive runs | Estimate | Actual | Actual / estimate |
|---|---|---|---|
| wall time (s) | <s> | <s> | <ratio, to 0.1> |
| spend (USD) | <USD> | <USD, or unknown> | <ratio, to 0.1, or unknown> |

- total test wall time: <s> s
- test agents: <n>
- review agents: <n>
- fix rounds: <n>
- asks to the operator about tests: <n>

## Tests kept
<!--
Each of the change's tests, kept (promoted to tests/<id>/) or not kept (left
in the archive folder), and each test moved out of tests/ as superseded or
stale, each with its reason.
-->
- kept: <test name>: <reason>
- not kept: <test name>: <reason>
- moved out: <test name>: <reason>

## Costly regression tests not run
<!--
Each agent-class or live-class regression test not run since its last pass,
so the operator can call a full regression; None if every one ran.
-->
- <test name>: last pass <ev id>
