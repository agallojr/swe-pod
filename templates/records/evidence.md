---
# Record: changes/<id>/evidence/<YYYYMMDDTHHMMZ>.md, one file per verify run,
# named by its UTC start. Replace every <placeholder> and drop these comments.
id: ev-<YYYYMMDDTHHMMZ>
change: <id>
# every repo in pod.yaml; a repo the spec does not name, at its pin
commit_set: {space: <sha7>, <repo>: <sha7>, ...}
env: <environment class>
actor: <handle>/test
# every scenario of this change, then the regression suite's results under
# qualified ids: <change>/<Rn.Sm>
scenarios: {<Rn.Sm>: pass|fail|not-run, ...}
# each affected repo's unit suite; none-present when the repo has none
unit: {<repo>: pass|fail|none-present}
# a result for every check the spec declares; when the spec has no Quality
# section, write this line instead: quality: none-declared
quality: {<Qn>: pass|fail}
# optional: the plan's inspection checks and the builders' test suggestions;
# leave either key out when there are none
x-checks: {<In>: pass|fail}
x-suggestions: {T<n>.TS<k>: pass|fail}
# the whole run, in s
wall_time_s: <integer>
---
<!--
The body holds failure summaries only. Logs appear only as pointers, which
are space-relative paths or URLs, never a machine path; never paste a log.
A <result key> is any key above with a result: a scenario, a repo's unit
suite, a Qn, an In or a T<n>.TS<k>. A not-run (cap) result names its cap.
-->
## Failures
### <result key> fail
- summary: <what failed, against which expectation from the spec>
- log: <space-relative path or URL>

### <result key> not-run (cap)
- cap: <the cap that stopped it, with its value and unit>
- log: <space-relative path or URL>
