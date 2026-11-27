# Agent guidance

## Purpose

This repository is a reusable template for local CAD MCP workspaces. It owns
workspace/bootstrap behavior only.

It does not own:

- Forge Python API behavior: `brainboxemb/lib.py.solidcraft.forge`;
- Forge CAD MCP behavior: `brainboxemb/tool.cad.mcp.forge`;
- vcs2l or the shared developer-tools environment:
  `brainboxemb/tool.py-dev-env`;
- CAD model source: the concrete CAD-library repository configured by a
  workspace created from this template;
- SolidCraft engine behavior.

## Start work

Before changing the template:

1. read `docs/plan.md`;
2. read `docs/developer-manual.md`;
3. read `docs/manual.md` when changing user-visible workflow;
4. keep `docs/plan.md` and `docs/handoff.md` structurally aligned with
   `brainboxemb/meta.cad.solidcraft/templates/docs/`;
5. create a short-lived branch from `dev`;
6. keep one logical change per issue/branch/PR;
7. run the workspace unit tests.

## Branch model

Permanent branches are only `main` and `dev`.

- short-lived branch -> `dev`: squash;
- `dev` -> `main`: merge commit;
- `main` -> `dev`: merge commit only for stable-only content not already on
  dev.

Every integration uses `/merge` and the repository-local merge-policy
workflow. Never select merge/squash/rebase manually and never use direct GitHub
merge as fallback.

Merged temporary branches are cleanup candidates. `main` and `dev` never
are.

## Workspace safety

MCP and Forge are external source dependencies. Their Git materialization is
owned by vcs2l through committed `dependencies.repos`; do not reimplement
clone/fetch/checkout/update logic for them in workspace Python.

The CAD repository is different: it is the user's active worktree. Workspace
bootstrap may clone it when absent and verify its origin when present, but must
not reset, force-checkout, clean, pull or otherwise rewrite an existing CAD
checkout.

The exact target model owner is explicit local configuration. Never implement
fuzzy model discovery or automatic target switching.

Generated VS Code project context should expose the selected model owner as the
normal folder. Do not expose the complete external toolchain or CAD catalog as
ordinary assistant browse context.

vcs2l externals stay below ignored `ext/`. The active CAD worktree stays below
ignored `.repos/cad/`. Generated client files stay below ignored
`.generated/`.

## Bootstrap behavior

Bootstrap must use the shared vcs2l installation from the current shell or the
`brainboxemb-dev-tools` Conda environment owned by
`brainboxemb/tool.py-dev-env`. Project-specific Python dependencies belong in
this workspace's own Conda environment.

Do not expose Git clone/update behavior as an MCP model-work tool. Repository
management is an operator/workspace responsibility.
