# T<n> report

- change: <id>
- task: T<n>
- actor: <handle>/<build|test>

<!--
Record: changes/<id>/tasks/T<n>-report.md. A test task's report goes in its
lane instead, as changes/<id>/evidence/<YYYYMMDDTHHMMZ>-T<n>-report.md. A
report is a claim; the commits and the test output are the evidence. Replace
every <placeholder>, then delete these comments.
-->

## Status
<one of: DONE, DONE_WITH_CONCERNS, BLOCKED, NEEDS_CONTEXT>

## What I did
<what was built, step by step, test first>

## Files
- <repo>/<path>: <created|changed>, <what it holds>

## Test output
<!--
For each check: the command, its red output before the change and its green
output after. Logs go in as pointers, never pasted whole.
-->
### <check or test name>
- command: <command>
- red: <output before the change>
- green: <output after the change>

## Concerns
- <a concern, or None>
<!--
A builder disputing a project-level test writes one line per test:
Disputed test: <test name>; spec clause: <clause>
-->

## Test suggestions
<!--
Optional. The for-the-tester part is in spec terms: the requirement or
scenario at risk, the input, and the failure expected; no code, no diff, and
no expected value the code computed. The for-the-reviewer part names the code
location.
-->
### T<n>.TS<k> <title>
- for the tester: <the requirement or scenario at risk, the input, and the
  failure expected>
- for the reviewer: <repo>/<path>:<line>[-<line>], <what is fragile there>
