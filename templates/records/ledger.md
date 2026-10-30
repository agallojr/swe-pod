# Ledger: <id>

<!--
Record: changes/<id>/ledger.md, the change's log. It is one fenced block, one
event per line, as <YYYY-MM-DDTHH:MMZ> <event> in UTC, appended in time order
and never edited. Continuation lines are indented 18 spaces, the width of a
timestamp and its space. The block below lists every event form; a new
ledger's block starts empty. Machines appear only as environment classes. A
refused line never quotes the text that matched. Every Ruling: line reaches
the final report. The ask: line replaces the retired heavy: line: the lead
asks the operator only about an expensive run the plan does not list, or one
past twice the plan's estimate. Delete this comment in a real ledger.
-->

```
<YYYY-MM-DDTHH:MMZ> spec approved by <handle>
<YYYY-MM-DDTHH:MMZ> go by <handle>; branches change/<id>: <repo> <sha7>[, <repo> <sha7>]
<YYYY-MM-DDTHH:MMZ> dispatch T<n> -> <role> (<local|mesh>, <env class|any>)[, base <repo> <sha7>]
<YYYY-MM-DDTHH:MMZ> T<n> <DONE|DONE_WITH_CONCERNS|BLOCKED|NEEDS_CONTEXT>[, <repo> <sha7>..<sha7>][, unit <p>/<n> pass]
<YYYY-MM-DDTHH:MMZ> T<n> review: <spec ok|spec issues>; <k> <Severity> [<repo>/<path>:<line>]
<YYYY-MM-DDTHH:MMZ> T<n> fix round <r>/5: <a> addressed, <o> open, <sha7>..<sha7>
<YYYY-MM-DDTHH:MMZ> T<n> at base: <test name> <fail|pass> (<reason>)[, ...]
<YYYY-MM-DDTHH:MMZ> T<n> returned to test: <test name> passed at base
<YYYY-MM-DDTHH:MMZ> T<n> harness error: <test name>, attempt <k>/3, stopped at <where>
<YYYY-MM-DDTHH:MMZ> T<n> cost: actual <s> s, <USD|unknown> USD; estimate <s> s, <USD> USD
<YYYY-MM-DDTHH:MMZ> T<n>.TS<k> -> test: covers <risk>
<YYYY-MM-DDTHH:MMZ> T<n>.TS<k> -> <spec|builder>
<YYYY-MM-DDTHH:MMZ> T<n>.TS<k> returned to builder: carries a computed value
<YYYY-MM-DDTHH:MMZ> T<n>.TS<k> outcome: <tested <test name>|proposed as scenario|unit-tested|declined: <reason>|dropped>
<YYYY-MM-DDTHH:MMZ> T<n>.TS<k> outcome: declined: covered by <test name>
<YYYY-MM-DDTHH:MMZ> approval reopened for <Rn>[, <Rm>]; held T<a>[, T<b>]
<YYYY-MM-DDTHH:MMZ> ask: <handle> before T<n>: <run>; <unlisted|past twice the estimate>; spent <s> s, <USD> USD of <s> s, <USD> USD estimated
<YYYY-MM-DDTHH:MMZ> T<n> not dispatched: <run>; past twice the estimate, no operator present
<YYYY-MM-DDTHH:MMZ> Ruling: <what>; <why>; <cost if wrong>
                  <continuation of the line above, indented 18 spaces>
<YYYY-MM-DDTHH:MMZ> filed <space path>[, ...]
<YYYY-MM-DDTHH:MMZ> refused <space path>: <profile|reference|id|lane> line <n>
```
