# template.cad.mcp.workspace

Reusable local workspace template for a Forge CAD assistant environment.

A repository created from this template does **not** own CAD models, Forge or
the MCP implementation. It coordinates the active CAD worktree, materializes
the pinned MCP/Forge external toolchain with vcs2l, creates the project Conda
environment and generates the VS Code/Copilot MCP configuration. The active
target can be either one exact CAD model owner or one ignored local standalone
workbench.

Typical concrete repository:

```text
template.cad.mcp.workspace
        ↓ use template
workspace.cad.mcp.models
```

Local generated state:

```text
workspace.cad.mcp.models/
├── dependencies.repos         # committed vcs2l pins
├── workspace.yml              # committed workspace/CAD config
├── workspace.local.yml        # local target, ignored
├── ext/                       # vcs2l externals, ignored
│   ├── mcp/
│   └── forge/
├── .repos/
│   └── cad/                   # active CAD worktree, ignored
├── .workbench/                # local standalone MCP experiments, ignored
└── .generated/
    ├── forge-cad.code-workspace
    ├── mcp-config.json
    └── state.json
```

`cad-mcp-workspace` is the single Conda environment for this workspace. It
contains vcs2l, the MCP runtime and the project-specific Python dependencies.
Bootstrap creates or updates it once and all workspace commands run through that
same environment.

Start with [docs/developer-manual.md](docs/developer-manual.md) when creating a
new concrete workspace. Daily operation is in [docs/manual.md](docs/manual.md).

The template uses `main` for stable content and `dev` for integration.
All normal source integrations use the repository-local `/merge` policy.
