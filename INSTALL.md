# Installing the SWE Pod

The SWE Pod is Markdown: a process document, six role definitions, a skill and
templates. Nothing in it is compiled. Installing it makes the role definitions
and the skill visible to every Claude Code session you start, once per user.
Each project then gets its own Project Space, started from the template.

## What you need

- Claude Code and git.
- A clone of this repo. Below, `<swe-pod>` is the absolute path of that clone.
- The Agent Mesh product, for its `space-librarian` skill, which files records
  into a Project Space. The pod depends on it whenever no mesh is present; on
  a mesh, the space's librarian uses the same skill. A clone of the product is
  enough, with no bus. Below, `<agent-mesh>` is the absolute path of that
  clone.

## Install into your global Claude config

Link the six role definitions and the `swe-pod` skill into your global Claude
config, `~/.claude/` by default. Link rather than copy, so a pull of this repo
reaches every session.

```sh
mkdir -p ~/.claude/agents ~/.claude/skills
ln -sf <swe-pod>/agents/swe-*.md ~/.claude/agents/
ln -sfn <swe-pod>/skills/swe-pod ~/.claude/skills/swe-pod
```

Without a mesh, link agent-mesh's librarian skill the same way. On a mesh
node, the Agent Mesh installer links every product skill, this one included;
a node installed before the skill existed links it by hand, as here.

```sh
ln -sfn <agent-mesh>/skills/space-librarian ~/.claude/skills/space-librarian
```

Never install them into a project: not into `<project>/.claude/`, a code
repo or a Project Space. They are the product, installed once per user.
A project copy drifts from the product and rides along in that repo's
commits. A space pins the process version it follows in its
`<space>/pod.yaml` instead.

To check, list the links. A new session's `/agents` then lists the six
`swe-*` agents.

Every role runs on the most capable model available. The six role
definitions say `model: inherit`, so each pod agent runs on the model of the
session that starts it: start that session on the most capable model
available, and never pick a lesser one to save cost.

```sh
ls -l ~/.claude/agents/swe-*.md ~/.claude/skills/swe-pod/SKILL.md
```

### On a mesh

Mount this repo in the bus as a submodule at `<bus>/swe-pod/`, beside the
Agent Mesh product, so every node gets it with the bus. On each node, link
from that checkout: `<swe-pod>` is then `<bus>/swe-pod`. The mount path
matters, because a mesh task names its role definition relative to the bus
root, as `swe-pod/agents/swe-<role>.md`.

```sh
git -C <bus> submodule add -b main <swe-pod repo url> swe-pod
git -C <bus> config -f .gitmodules submodule.swe-pod.ignore all
```

Commit and push the mount under the bus's own rules. A node that takes a pod
role adds it, as `swe.<role>`, to `AGENT_ROLES` in its identity file. The
Agent Mesh operator manual covers roles and mounts.

## Start a Project Space

Start one Project Space per project, from `templates/space/`. Below, `<space>`
is the absolute path of its new checkout.

```sh
git init -b main <space>
cp -R <swe-pod>/templates/space/. <space>/
cp <swe-pod>/PROCESS.md <space>/
```

Then replace every `<placeholder>`:

- `<space>/space.yaml`: `id` as `proj:<name>`, the `title`, the `owners` by
  handle, and the `visibility`.
- `<space>/pod.yaml`: `project`, one line per code repo with its `remote` and
  its `pin` (the commit the project stands on), the merge order in `depends`,
  and the `gates`.
- `<space>/README.md` and `<space>/AGENTS.md`: the project's name, and one
  sentence on what it builds.

Leave `<space>/STATUS.md` out: the `swe-pod` skill generates it from the
records. Keep the record templates in the product; agents read them from
`templates/records/`.

Make the space's first commit yourself, and push it to the space's remote, a
new and empty git repo. From then on, every commit to the space goes through
its librarian.

```sh
git -C <space> add -A
git -C <space> commit -m "Start the Project Space"
git -C <space> remote add origin <space repo url>
git -C <space> push -u origin main
```

- **Single seat.** Open a Claude Code session in `<space>` and say what you
  want. The session is the spec author, planner, lead and the space's
  librarian (`README.md`, Use).
- **On a mesh.** Mount the space by hand under `<bus>/spaces/` and list it in
  `<bus>/spaces.yaml` with `access: rw` and its librarian's role, as the Agent
  Mesh operator manual's section on mounting a memory space says. A session
  then loads it with "load <project>".

## Rules for your guidance overlay

The pod's agents commit to code repos and to the Project Space, and they write
specs and tests as part of a change. If your deployment's base rules forbid
that without the human's say-so, add these three rules to your own guidance
overlay, the layer of deployment-specific rules on top of the base rules.

- **G1. Pod commit scope.** In a pod's code repos, the lead creates each
  `change/<id>` branch at the go, and build agents commit to it and push it.
  No other branch is created or pushed. The Project Space's librarian may
  commit to the space's `main` and push it. Never commit to or push `main` of
  a code repo, and never force-push. Merging, and deleting a merged branch,
  happen only on the human's word. Until this rule is in the overlay, the
  human makes these commits.
- **G2. A pod change counts as asking.** Where the base rules say "no
  separate spec files unless asked" or "write test modules only when asked",
  a pod change counts as asking.
- **G3. A listed test run after the go counts as operator-posted.** For
  the budget policy on long runs, such as a solver-run budget policy, a
  test or verify task the lead posts after the human's go counts as
  operator-posted for every run the plan lists, the runs a test-writing
  task makes at base included. The policy's caps, and the points where it
  stops to ask, still hold.

These rules loosen nothing else. The git gate, the solver-run fence and the
budget policy still bind every pod agent, and a check expected to take over
60 s runs under them like any solver run.
