# Agents: start here

This is the Project Space for `proj:<project>`. It holds the process, specs,
plans, records and project-level tests. Code and the other deliverables live in
the code repos `pod.yaml` names.

## Read order

1. `STATUS.md`: where each open change stands. The `swe-pod` skill generates
   it from the records; nobody edits it by hand. If it is missing, run the
   skill to make it.
2. `pod.yaml`: the code repos, their pins, their merge order and the gates.
3. `PROCESS.md`: the process, at the version `pod.yaml` pins.
4. For the change you work on: `changes/<id>/spec.md`, then `plan.md`, then
   `ledger.md`. Trust the ledger and the git logs over memory.
5. Your task's brief, if you have one: `changes/<id>/tasks/T<n>-brief.md`.

**Exception.** A test-writing agent reads only its brief and `spec.md`.
While it writes project-level tests it skips the read order above: no
`STATUS.md`, `plan.md` or `ledger.md`, no other brief, report or review, and
no code-repo diff or implementation body.

## Standing rules

- **Specs first.** Nothing is planned, built or tested for a change whose
  `spec.md` has no `approved:` line.
- **One branch.** This space has one branch, `main`, and never another. Work
  in flight is kept apart by folder, `changes/<id>/`. Code work happens only
  on `change/<id>`, the one branch per spec in each code repo the spec names.
- **Every write goes through the librarian.** Only the space's librarian
  commits here. A change's records reach it through that change's lead.
  `tests/`, `pod.yaml` and `changes/archive/` change only at integrate.
- **Records use the shareable profile.** Actors are human handles or
  `<handle>/<role>`, machines are environment classes such as `macos-laptop`,
  and paths are relative to this space or are URLs. No hostname, agent id,
  mesh task id, home-directory path or credential name.
- **New record ids are collision-free.** An id is the category, the UTC
  minute and four hex digits: `<category>-<YYYYMMDDTHHMMZ>-<4 hex>`.
  Evidence, decisions and handoffs are one file per event, named by UTC time.
- **Code repos hold only code, unit tests and shipped docs.** Specs, plans,
  ledgers, briefs, reports, reviews, evidence and project-level tests live in
  this space, never in a code repo.
