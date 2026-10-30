# swe-pod

The SWE Pod: a spec-first software engineering process run by a pod of
agents, local or remote, with or without an Agent Mesh.

- Design doc (base spec): `SPEC.md`
- Depends on: `agallojr/agent-mesh`, for its librarian
- Development: project `proj:swe-pod`, Project Space `agallojr/swe-pod-space`

Status: version 2 (`swe-pod-process@2`): the process, the six role
definitions, the Project Space and record templates, and the `swe-pod` skill.
Version 2 sizes testing to risk (change `right-size-testing`). First use was
the pilot (`SPEC.md` §16).

To install it and start a Project Space, see `INSTALL.md`.

## How it works

`PROCESS.md` is the process, written as instructions the agents follow. It is
the authority; this is a summary. A change runs five stages:

1. **Spec.** The spec author draws out what you want, one question at a time,
   and writes the change's spec: requirements, each with at least one
   scenario. You approve it. Nothing is planned, built or tested before that.
2. **Plan.** A fresh planner maps every requirement to a build task and every
   scenario to a test, and gives each task its dependencies. Every test the
   plan lists names the risk it covers, and the plan lists no test without
   one. Risk, not size, decides independent tests: a requirement the plan
   marks at risk gets blind project-level tests, written by a test agent
   that never saw the build, at any size; a requirement not at risk gets
   none beyond its builder's tests, which the review checks against its
   scenarios. The plan estimates the wall time in s and spend in USD of its
   agent-class and live-class runs, and lists a live test only to reproduce
   and confirm a reported defect or where nothing cheaper can show the
   behavior. Tasks with no dependency between them run in parallel. At
   project size you skim the plan.
3. **Build.** At your go, the lead creates one branch, `change/<id>`, in each
   code repo the spec names. Builders commit code, unit tests and shipped docs
   to it, test first. There is one review per wave of build tasks, and a
   review of a build task of its own only when the plan marks it at risk; a
   change whose build tasks form one wave gets one review, the whole-change
   review. Minor findings never start a fix round. In parallel, a test agent
   writes the blind tests of the requirements at risk, from the spec alone.
4. **Verify.** A test agent runs the change's own tests, every unit suite,
   every declared quality check and the regression tests the change could
   break, at one commit set, and writes the evidence: a script-class
   regression test runs when it reads a file the change touches, and an
   agent-class or live-class one only when the plan names it for a risk. A
   full regression runs only on your word. A regression test verify rightly
   did not run, or one superseded by a passing test of the change, does not
   hold up green. The two results that can be no bar are regression-test
   results: on a scenario of the spec, a unit suite, a quality check or a
   test in `changes/<id>/tests/`, either one is a bar. Verify may be split
   into shards run by several test agents, local and remote together. A
   sharded verify counts only when the shards' evidence, all at the same
   commit set, together gives a pass for every scenario, unit suite,
   declared quality check and test of the change, and for every regression
   test a pass or a result that is no bar.
5. **Integrate.** The lead files a final report that lists every ruling it
   made for you and the test cost against the plan's estimate. On your word,
   the branches merge in dependency order, then the pins advance, and only
   the tests the plan marks to keep join the space's regression suite.

Records and project-level tests live in a Project Space: a git repo with one
branch, `main`, holding specs, plans, ledgers, briefs, reports, reviews,
evidence and decisions. Code repos hold only code, unit tests and the docs
that ship with the code. The roles are spec, plan, lead, build, test and
review, and each task runs as its own agent, started fresh from its brief
(`PROCESS.md` §10). Every role runs on the most capable model available. The
savings come from running fewer, better-targeted tests, never from weaker
agents. There are no commands: you talk to the pod in plain words
(`PROCESS.md` §14).

## Layout

```
SPEC.md                             the design doc (base spec), an input
spec-process.d2                     the process diagram, D2 source
spec-process.svg                    the process diagram, SVG
spec-process.png                    the process diagram, PNG
PROCESS.md                          the process, as agent instructions
agents/swe-spec.md                  role definition: spec author
agents/swe-plan.md                  role definition: planner
agents/swe-lead.md                  role definition: lead
agents/swe-build.md                 role definition: builder
agents/swe-test.md                  role definition: test agent
agents/swe-review.md                role definition: reviewer
templates/space/space.yaml          new space: memory-space manifest
templates/space/pod.yaml            new space: code repos, pins, merge order
templates/space/AGENTS.md           new space: read order for a cold arrival
templates/space/README.md           new space: its README
templates/space/changes/            new space: one folder per change
templates/space/tests/              new space: regression suite
templates/space/decisions/          new space: one file per decision
templates/space/handoffs/           new space: one file per handoff
templates/space/notes/              new space: notes
templates/space/lore/               new space: verified gotchas
templates/records/spec.md           record format: change spec
templates/records/plan.md           record format: plan
templates/records/brief.md          record format: task brief
templates/records/test-brief.md     record format: test-writing brief
templates/records/report.md         record format: task report
templates/records/review.md         record format: review
templates/records/evidence.md       record format: verify evidence
templates/records/ledger.md         record format: change ledger
templates/records/handoff.md        record format: handoff
templates/records/final-report.md   record format: final report
skills/swe-pod/SKILL.md             the swe-pod skill: status, go and trace
skills/swe-pod/status-format.md     the format of <space>/STATUS.md
README.md                           this file
INSTALL.md                          install, and rules for your overlay
```

The role definitions are Claude Code agent definitions. The same files brief
a local subagent and a mesh executor. Each empty folder of the space template
holds a `.gitkeep`. Record templates stay here in the product, and agents
read them from it; a space copies only `templates/space/` and `PROCESS.md`.

## Use

The pod runs the same way with or without a mesh: the same briefs, the same
role definitions and the same records. A project can move between single
seat and mesh use at any stage boundary.

### Single seat

With no mesh present (no bus, core, lore or remote agent), one interactive
Claude Code session, opened in the Project Space's checkout, is the spec
author, planner, lead and the space's librarian. It starts a fresh local
subagent for each build, test and review task, from that task's brief. At
project size it plans through a fresh local subagent too, since spec and plan
must then be different agents.

When no mesh is present, the pod depends on agent-mesh's `space-librarian`
skill. The session files every record into the space through it, with the
same profile, reference and id checks a mesh librarian applies, and the skill
makes the commit. That needs the Agent Mesh product's code, installed as
`INSTALL.md` says, but no bus.

A session goes like this. The words are examples, not syntax.

1. Say what you want: "I want the nozzle solver to take a VQLS backend". The
   spec author plays it back, asks the one question that matters most, and
   drafts the spec.
2. Say "approved", then "go". The lead creates the branches and dispatches
   every task whose dependencies are done.
3. Ask "where are we?" at any time, and the `swe-pod` skill answers from the
   records. Ask it to regenerate the status view, `<space>/STATUS.md`, when
   you want that file brought up to date.
4. When verify is green, say "merge it". The lead integrates on your word.

### On a mesh

On an Agent Mesh, the pod's roles are the mesh roles `swe.spec`, `swe.plan`,
`swe.lead`, `swe.build`, `swe.test` and `swe.review`. A node takes one by
adding it to `AGENT_ROLES` in its identity file. The operator mounts the
Project Space in the bus by hand, under `<bus>/spaces/`, and lists it in
`<bus>/spaces.yaml`.

- **Dispatch.** For a task with `runs: mesh`, the lead posts a `task.request`
  to `role:swe.<role>`, with `project: proj:<name>` so the claiming node loads
  the space for its executor. `needs:` narrows it to nodes of one environment
  class, such as `linux-server`. The Context names the role definition
  relative to the bus root, as `swe-pod/agents/swe-<role>.md`, and the brief
  relative to the space. The executor reads the role definition first.
- **Results.** A mesh agent returns its files (report, tests, review,
  evidence) in its task result, under `files:`. The lead checks them and files
  them through the space's librarian, its only mesh writer, which uses the
  same `space-librarian` skill.
- **Build stays local for now.** A mesh executor does not touch git, so build
  tasks run local until a remote builder may commit to `change/<id>`.
- **Shareable records.** The ledger names the role, `local` or `mesh`, and the
  environment class, never a machine. Mesh task ids and agent ids stay in the
  bus, so the space can be handed to someone with no access to the mesh.
- **Fences.** Pod roles loosen no mesh fence. The git gate, the solver-run
  fence and the budget policy bind every pod agent. Heaviness is judged per
  check, from that check's own recorded expected run time alone: over 60 s is
  heavy, 60 s or less is not, whatever its cap, its label or the task's total
  time, and a heavy check runs under the fences like any solver run.
- **Asks about test runs.** The lead asks the operator about a test run only
  when it is an agent-class or live-class run the plan does not list, or
  when it would take the expensive runs past twice the plan's estimate in
  wall time or in spend, counting their actuals so far plus the estimate of
  the run to dispatch. Every other run the plan lists goes ahead with no
  ask. Your go on the plan covers it. With no operator present, a run past
  that point is not dispatched. The budget policy's own ask points still
  hold.
