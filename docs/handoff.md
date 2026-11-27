# template.cad.mcp.workspace handoff

Repository: `brainboxemb/template.cad.mcp.workspace`.

This document is the easy-to-find handoff entrypoint for a new work session.
It contains one copyable handoff message below.

The handoff message is deliberately an entrypoint, not a second repository
plan. Current work must always be derived from live GitHub state and
`docs/plan.md`.

Shared SolidCraft/CAD documentation conventions live in
`brainboxemb/meta.cad.solidcraft`. Use
`templates/docs/plan.md` and `templates/docs/handoff.md` there as structural
references.

## Handoff message

Copy the complete text block below into a new chat.

~~~text
You are continuing work on brainboxemb/template.cad.mcp.workspace.

Repository role:

- this repository is the reusable template for local Forge CAD MCP assistant
  workspaces and owns operator-side workspace/bootstrap behavior;
- it coordinates the project Conda environment, vcs2l-materialized MCP/Forge
  externals, the concrete CAD-library worktree and generated VS Code/Copilot
  client configuration;
- vcs2l/shared developer tooling remains owned by
  brainboxemb/tool.py-dev-env; Forge API behavior remains owned by
  brainboxemb/lib.py.solidcraft.forge; Forge MCP behavior by
  brainboxemb/tool.cad.mcp.forge; CAD model source by the configured CAD
  repository; SolidCraft engine behavior by SolidCraft.

Use this handoff only as the entrypoint. Do not treat copied handoff text as
fresher than GitHub or the repository plan.

Start by:

1. inspecting the current issues, pull requests and branches;
2. reading AGENTS.md;
3. reading docs/plan.md;
4. treating docs/plan.md as the source of truth for unfinished work;
5. following the current work order from the plan rather than from old handoff
   text;
6. creating an issue before implementation when the logical change has none;
7. keeping one logical change to one issue, branch and PR;
8. updating requirements/design/manual documentation when behavior or
   architecture changes;
9. updating the plan after merge or release/checkpoint.

Durable working rules:

- MCP and Forge external Git checkouts are declared in dependencies.repos and
  materialized with vcs2l; do not duplicate that dependency engine in workspace
  code;
- the exact target model owner is explicit local configuration; never add fuzzy
  model discovery or implicit target switching;
- after initial clone an existing CAD checkout is the user's real worktree and
  must not be reset, cleaned, pulled or branch-switched by workspace
  maintenance;
- generated VS Code project context exposes the selected target owner as the
  normal project folder rather than the complete external toolchain/CAD
  checkout set;
- external toolchain checkouts stay below ignored ext/, the active CAD worktree
  below ignored .repos/cad/, and generated client files below .generated/;
- permanent source branches are main and dev; all normal integrations use the
  repository-local /merge policy.

Shared document structures live in brainboxemb/meta.cad.solidcraft:
templates/docs/plan.md and templates/docs/handoff.md.

Do not copy exact current SHAs, runtime/release pins, active issue/PR status,
temporary bootstrap phases or a short-lived "next actions" list into this
handoff. Those facts belong in GitHub and docs/plan.md.
~~~

## Template guidance

The text block above is the actual handoff message.

Everything outside that block only explains how to use or maintain the
document. A consumer may translate the copyable message to the user's normal
chat language when that is intentional; repository-maintained explanatory text
outside the block remains English.

Keep repository-specific durable role and ownership guidance in the block when
it helps route work correctly. Do not turn the handoff into a status snapshot.
