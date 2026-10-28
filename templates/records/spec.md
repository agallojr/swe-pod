# <id>: <one-sentence intent>

- size: <small|project|spike>
- owner: <handle>
- repos: [<repo>, ...]
- supersedes: []

<!--
Record: changes/<id>/spec.md. <id> is verb-led kebab-case. Replace every
<placeholder>, then delete these comments.

supersedes lists the qualified ids (<change>/<Rn>) of earlier requirements
this work changes; [] when none.

State lines. A new spec has none of them: the change is proposed until it has
approved:. Add each line to the header list above, exactly in this form, when
its event happens. <UTC> has the form YYYY-MM-DDTHH:MMZ.
- approved: <handle> <YYYY-MM-DDTHH:MMZ>
- integrated: <handle> <UTC>
- abandoned: <UTC> <reason>
An edit to Requirements after approval reopens approval for that edit only.
-->

## Why
<the problem this solves, in the human's words>

## Scope
In: <what this change covers>
Out: <what it leaves alone>

## Requirements
<!--
One behavior per requirement, with one SHALL or MUST, observable by someone
who has never seen the code; no class names, library choices or steps. At
least one scenario each, covering error and edge cases too. Every number
carries its unit and tolerance, and any convention it depends on: norm,
normalization, reference solution. Ids are local here (R1, R1.S1) and
qualified elsewhere (<id>/R1.S1).
-->
### R<n> <title>
<the system> SHALL <one observable behavior>.

#### R<n>.S<m> <title>
- GIVEN <the starting state>
- WHEN <the action>
- THEN <the observable result, with units and tolerances>

## Quality
<!--
One check per tool or review. Reuse a check an earlier spec declared by its
qualified id (<change>/Q<n>). Leave this section out when the change asks for
no quality check.
-->
### Q<n> <title>
The changed code SHALL <pass the check>.
- check: <what is run, on what>; passing is <what passing means>

## Approaches
<the two or three ways considered, what each trades off, the one picked and
why>

## Open calls
- <a default the author chose, which the human checks>
