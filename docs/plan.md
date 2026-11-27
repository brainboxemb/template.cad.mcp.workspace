# template.cad.mcp.workspace plan

This file is the current source of truth for unfinished work in
`brainboxemb/template.cad.mcp.workspace`.

This repository provides the reusable operator-side workspace template for a
Forge CAD MCP assistant environment. It materializes and validates the local
toolchain, binds one exact CAD model owner and generates client configuration;
it does not own Forge, MCP, vcs2l, CAD model or SolidCraft product behavior.

## Current accepted baseline

- permanent source lanes are `main` and `dev`;
- normal source integration uses the repository-local `/merge` policy;
- MCP and Forge are exact immutable release tags in `dependencies.repos`,
  materialized with vcs2l below `ext/`;
- the current released compatibility baseline is Forge CAD MCP 0.0.2 with
  Forge/SCPF 0.25.0 and SolidCraft 0.49.0; the next MCP release will add live
  SolidCraft workbench control before this template updates its pin;
- the active target is selected locally and is either one exact
  `models/.../<owner>` catalog owner or one ignored standalone workbench;
- an existing CAD checkout is validated but never reset, cleaned, pulled or
  branch-switched by workspace maintenance;
- generated VS Code context exposes only the selected target owner as its normal
  project folder;
- workspace, merge-policy and branch-cleanup test groups pass independently on
  the accepted stable baseline.

## Lane rule

Normal work starts from `dev` on a short-lived branch.

- short-lived branch -> `dev`: squash through `/merge`;
- `dev` -> `main`: merge commit through `/merge`;
- `main` -> `dev`: merge commit only for stable-only content that is not
  already present on `dev`.

`main` and `dev` are permanent. Merged temporary branches are cleanup
candidates.

## Engineering constraints

The workspace exists to make the assistant loop predictable and cheap:

- external Git dependency materialization must reuse vcs2l rather than duplicate
  generic clone/fetch/checkout logic;
- the workspace uses one dedicated `cad-mcp-workspace` Conda environment for
  both vcs2l and the MCP/runtime dependencies;
- toolchain release tags are explicit rather than floating;
- model selection is explicit rather than discovered by the agent;
- the CAD worktree remains independent from external dependency state;
- the normal editor context should stay narrow enough that the assistant uses
  MCP boundaries rather than browsing the whole toolchain.

Shared cross-repository principles live in
`brainboxemb/meta.cad.solidcraft/docs/engineering-principles.md`.

## Active workstreams

### WS1 — first concrete workspace acceptance

State:

- owner/lane: template `dev` plus concrete
  `workspace.cad.mcp.models` consumer;
- current status: template bootstrap, CI/governance, vcs2l external dependency
  handling and the single `cad-mcp-workspace` Conda model are implemented and
  green on permanent `dev`;
- next concrete gate: create a local standalone workbench, run doctor and one
  real VS Code MCP environment/skill/authoring cycle before involving the CAD
  catalog;
- blocker/external work: the concrete Windows run has not yet been completed.

## Open issue assignment

| Issue | Lane/owner | Status | Next action |
| --- | --- | --- | --- |
| #50 | template `dev` | now | Add workbench project configuration, live SolidCraft demo procedure and adopt the next MCP release once published |
| #1 | concrete workspace acceptance | next | Run the 60 × 20 × 4 mm / Ø8 live workbench demo after the MCP/template update |

## Recently completed

- #47 added the lightweight standalone workbench target using MCP
  `--workspace`; it creates no catalog metadata, CI file or CAD branch.
- #41 fixed the Windows external-rematerialization hang by passing
  `dependencies.repos` to `vcs delete` with `--input` instead of through
  inherited stdin.
- #38 fixed the generated Python metadata false-positive found by the first
  real Windows external-reconciliation run; that retry then exposed #41,
  where `vcs delete` was still reading its manifest through inherited stdin.
- #35 added automatic vcs2l external reconciliation for bootstrap/update; the
  first real Windows run then exposed #38 because generated Python package
  metadata was being mistaken for user edits.
- #30 added `scripts/pr.ps1` for operator-side PR publication. It can push and
  open a PR, wait for PR checks, and request integration only by posting the
  existing `/merge` command; the repository merge policy remains authoritative.
- #28 restricted the first CAD checkout to the configured development branch
  with a blobless partial clone, avoiding unrelated generated/history blobs
  during initial workspace setup.
- #21 split the user workflow into basic installation, new-model,
  SolidCraft-to-Forge and existing-model procedures; added explicit owner
  creation, immutable native-reference import and target-optional installation.
- #17 rewrote `docs/developer-manual.md` and `docs/manual.md` as ordered,
  executable procedures. The Conda environment creation/update now appears at
  the bootstrap step where it actually happens, instead of as detached
  architecture/reference text.
- #13 simplified bootstrap/runtime to one dedicated `cad-mcp-workspace` Conda
  environment containing both vcs2l and the Forge MCP/runtime dependencies; no
  normal environment switching is required.
- #9 replaced duplicated MCP/Forge Git checkout logic with the existing vcs2l
  dependency engine: exact revisions now live only in `dependencies.repos`
  and materialize below `ext/`; the active CAD worktree remains separate.
- #5 aligned `docs/handoff.md` and `docs/plan.md` with the shared structures
  in `brainboxemb/meta.cad.solidcraft`; the handoff now contains only durable
  session-entry guidance and current state remains in this plan.
- Initial repository bootstrap established workspace configuration, Conda
  setup, target selection, generated client files and repository governance.
- #2 corrected CI test isolation; the repository-local `/merge` path was then
  exercised for both temporary -> `dev` and `dev` -> `main`.

## Next release/checkpoint

The next acceptance checkpoint is a real
`workspace.cad.mcp.models` consumer on Windows:

1. complete basic installation without selecting a target;
2. create a local standalone workbench with `project.yml` and prove the MCP
   authoring -> live SolidCraft apply/inspect/mutate/view/render cycle using the
   60 × 20 × 4 mm plate with one centered Ø8 mm hole;
3. create and verify one maintained Forge catalog model;
4. convert one independently authored SolidCraft .sc reference to Forge;
5. continue one existing Forge model from Git;
6. prove local/catalog build scratch stays out of maintained Git source.

Only after that succeeds should #1 close and the template be treated as proven
outside repository CI.
