---
# Record: changes/<id>/evidence/<YYYYMMDDTHHMMZ>.md, one file per verify run,
# named by its UTC start. A verify shard's record is
# changes/<id>/evidence/<YYYYMMDDTHHMMZ>-<shard id>.md, one file per shard run.
# Replace every <placeholder> and drop these comments.
# a shard's record: id: ev-<YYYYMMDDTHHMMZ>-<shard id>
id: ev-<YYYYMMDDTHHMMZ>
change: <id>
# a shard's record only: its shard id, and the tests the record ran, by test
# name; leave both keys out of an unsharded record
shard: <shard id>
tests: [<test name>, ...]
# every repo in pod.yaml; a repo the spec does not name, at its pin
commit_set: {space: <sha7>, <repo>: <sha7>, ...}
env: <environment class>
actor: <handle>/test
# the selection rule: targeted, the default, or full, only on the operator's
# word; a record with no selection: key is read as full
selection: targeted
# every file the change's diff from base to head changes, in each repo of the
# spec's repos:
touched: [<repo>:<path>, ...]
# only when a run used a fixed harness: the space commit holding it
harness: <space sha7>
# A result is one of pass, fail, not-run (cap <n> s), harness-error,
# not-selected or superseded by <key>, where <key> is a result key of this
# change in the same record (R1.S1, I1). A regression test the selection does
# not pick is not-selected; a picked test is run.
# every scenario of this change, then the regression suite's scenario tests
# under <change>/R<n>.S<m>; a shard gives only those it ran
scenarios: {<Rn.Sm>: <result>, ...}
# each affected repo's unit suite; none-present when the repo has none; a
# shard gives only those it ran
unit: {<repo>: pass|fail|none-present}
# a result for every check the spec declares, or for a shard every check it
# ran; when the spec has no Quality section, write this line instead:
# quality: none-declared
quality: {<Qn>: pass|fail}
# optional: the plan's inspection checks and the builders' test suggestions,
# then the regression suite's: <change>/I<n> under x-checks: and
# <change>/T<n>.TS<k> under x-suggestions:; leave either key out when there
# are none
x-checks: {<In>: <result>}
x-suggestions: {T<n>.TS<k>: <result>}
# the whole run, in s
wall_time_s: <integer>
---
<!--
The body holds failure summaries only. Logs appear only as pointers, which
are space-relative paths or URLs, never a machine path; never paste a log.
A <result key> is any key above with a result: a scenario, a repo's unit
suite, a Qn, an In or a T<n>.TS<k>. A not-run (cap <n> s) result names its
cap. A harness-error names where the session stopped; it is no pass or fail.
List under Undeclared cost class each test that declares no class: and no
mode:, which verify treats as live class; leave the section out when there is
none.
-->
## Failures
### <result key> fail
- summary: <what failed, against which expectation from the spec>
- log: <space-relative path or URL>

### <result key> not-run (cap <n> s)
- cap: <the cap that stopped it, with its value and unit>
- log: <space-relative path or URL>

### <result key> harness-error
- stopped at: <where the session stopped, before the test's first step>
- attempt: <k>/3
- log: <space-relative path or URL>

## Undeclared cost class
- <test name>: treated as live class
