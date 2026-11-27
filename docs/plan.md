# template.cad.mcp.workspace plan

This file is the current source of truth for unfinished work in
`brainboxemb/template.cad.mcp.workspace`.

This repository provides the reusable operator-side workspace template for a
Forge CAD MCP assistant environment. It materializes and validates the local
toolchain, binds one exact CAD model owner and generates client configuration;
it does not own Forge, MCP, CAD model or SolidCraft product behavior.

## Current accepted baseline

- permanent source lanes are `main` and `dev`;
- normal source integration uses the repository-local `/merge` policy;
- MCP and Forge are exact toolchain pins in committed `workspace.yml`;
- the current template baseline tracks Forge CAD MCP 0.0.1 with Forge/SCPF
  0.19.0 and SolidCraft 0.49.0;
- local child checkouts are materialized below ignored `.repos/`;
- local/generated client state is below ignored `.generated/` and
  `workspace.local.yml`;
- one exact `models/.../<owner>` target is selected locally;
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

- toolchain revisions are explicit rather than floating;
- model selection is explicit rather than discovered by the agent;
- bootstrap/update must be idempotent and fail closed on unexpected toolchain
  modifications;
- CAD source control remains independent from workspace source control;
- the normal editor context should stay narrow enough that the assistant uses
  MCP boundaries rather than browsing the whole toolchain.

Shared cross-repository principles live in
`brainboxemb/meta.cad.solidcraft/docs/engineering-principles.md`.

## Active workstreams

### WS1 — first concrete workspace acceptance

State:

- owner/lane: template `dev` plus a concrete
  `workspace.cad.mcp.models` consumer;
- current status: template bootstrap, CI and governance are implemented;
- next concrete gate: create the concrete workspace from this template,
  configure the real CAD catalog, then run Windows bootstrap + doctor and one
  real VS Code MCP environment/skill call;
- blocker/external work: the concrete workspace repository and local Windows
  acceptance have not yet been completed.

### WS2 — shared documentation convention alignment

State:

- owner/lane: template `dev`;
- current status: handoff/plan alignment is being corrected under issue #5;
- next concrete gate: merge the meta-aligned document structure and keep future
  status changes in this plan rather than the handoff;
- blocker/external work: none.

## Open issue assignment

| Issue | Lane/owner | Status | Next action |
| --- | --- | --- | --- |
| #1 | dev / concrete workspace acceptance | now | Create and exercise `workspace.cad.mcp.models` on Windows through bootstrap, doctor and a real MCP call |
| #5 | dev / documentation | now | Merge the meta-aligned handoff and plan structure |

## Recently completed

- Initial repository bootstrap established workspace configuration, Conda
  setup, toolchain/CAD checkout handling, target selection, generated client
  files and documentation.
- #2 corrected CI test isolation; the repository-local `/merge` path was then
  exercised for both temporary -> `dev` and `dev` -> `main`.

## Next release/checkpoint

The next acceptance checkpoint is a real
`workspace.cad.mcp.models` consumer on Windows:

1. create it from this template;
2. configure the active CAD catalog;
3. run `bootstrap.ps1` and `doctor.ps1`;
4. open the generated VS Code workspace;
5. complete an actual Forge MCP `environment` call and skill lookup.

Only after that succeeds should #1 close and the template be treated as proven
outside repository CI.
