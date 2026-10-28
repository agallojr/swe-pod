# swe-pod

The SWE Pod: a spec-first software engineering process run by a pod of
agents, local or remote, with or without an Agent Mesh.

- Design doc (base spec): `SPEC.md`
- Depends on: `agallojr/agent-mesh`, for its librarian
- Development: project `proj:swe-pod`, Project Space `agallojr/swe-pod-space`

Status: version 1 (`swe-pod-process@1`): the process, the six role
definitions, the Project Space and record templates, and the `swe-pod` skill.
First use is the pilot (`SPEC.md` §16).

To install it and start a Project Space, see `INSTALL.md`.

## How it works

`PROCESS.md` is the process, written as instructions the agents follow. It is
the authority; this is a summary. A change runs five stages:

1. **Spec.** The spec author draws out what you want, one question at a time,
   and writes the change's spec: requirements, each with at least one
   scenario. You approve it. Nothing is planned, built or tested before that.
2. **Plan.** A fresh planner maps every requirement to a build task and every
   scenario to a project-level test, and gives each task its dependencies.
   Tasks with no dependency between them run in parallel. At project size you
   skim the plan.
3. **Build.** At your go, the lead creates one branch, `change/<id>`, in each
   code repo the spec names. Builders commit code, unit tests and shipped docs
   to it, test first, and each build task is reviewed. In parallel, a test
   agent writes the project-level tests blind, from the spec alone.
4. **Verify.** A test agent runs the change's project-level tests, the
   space's regression suite, every unit suite and every declared quality check
   at one commit set, and writes the evidence. A fresh reviewer then reviews
   the whole change. Verify may be split into shards run by several test
   agents, local and remote together. A sharded verify counts only when the
   shards' evidence, all at the same commit set, together holds a pass for
   every scenario, unit suite and declared quality check.
5. **Integrate.** The lead files a final report that lists every ruling it
   made for you. On your word, the branches merge in dependency order, then
   the pins advance and the change's tests join the space's regression suite.

Records and project-level tests live in a Project Space: a git repo with one
branch, `main`, holding specs, plans, ledgers, briefs, reports, reviews,
evidence and decisions. Code repos hold only code, unit tests and the docs
that ship with the code. The roles are spec, plan, lead, build, test and
review, and each task runs as its own agent, started fresh from its brief
(`PROCESS.md` §10). There are no commands: you talk to the pod in plain words
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
  time. A check that records no expected run time, such as a unit suite or a
  declared quality check, takes it from its brief or from the record of its
  last run; with neither, the lead treats it as heavy and asks. With you
  present, the lead first gives you the command, the caps and the expected
  wall time in s of each heavy check, and of no other check.
