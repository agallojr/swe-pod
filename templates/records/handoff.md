# Handoff: <change id>, <YYYY-MM-DDTHH:MMZ>

- from: <handle>/<role>
- to: <handle>/<role>

<!--
Record: handoffs/<YYYYMMDDTHHMMZ>-<change id>.md, written at a stage
boundary. Mark done only what has evidence. The receiver re-runs the latest
verify before building on what it inherits. Replace every <placeholder>, then
delete these comments.
-->

## State
- change state: <proposed|approved|planned|building|verified|integrated|abandoned>
- done, with evidence: <task or result>: <evidence id, sha7 or space path>
- branches: <repo> change/<id> at <sha7>[, <repo> change/<id> at <sha7>]
- latest evidence: ev-<YYYYMMDDTHHMMZ>, <green|red>

## Half-done
- T<n>: <what exists, at which sha7, and what remains>

## Open questions
- <question>: <who can answer it>

## Next command
<the latest verify to re-run, then the one command or request that continues
the work>
