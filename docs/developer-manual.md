# Developer manual

This document explains how to create and maintain a concrete CAD assistant
workspace from `template.cad.mcp.workspace`.

## Repository roles

```text
template.cad.mcp.workspace
        |
        | GitHub template
        v
workspace.cad.mcp.models
        |
        +-- dependencies.repos
        |      |
        |      +-- vcs2l -> ext/mcp
        |      +-- vcs2l -> ext/forge
        |
        +-- .repos/cad            -> active CAD worktree
        +-- workspace.local.yml   -> selected models/... owner
```

The concrete workspace repository owns only environment/configuration glue. CAD
source remains in the CAD repository. MCP and Forge source remain external
dependencies materialized by vcs2l.

## External dependency engine

The workspace uses vcs2l as its Git dependency engine. The template commits
`dependencies.repos` and pins `vcs2l==1.1.7` in the same
`cad-mcp-workspace` Conda environment that runs the MCP.

```text
cad-mcp-workspace
        ├── vcs2l
        └── Forge CAD MCP runtime

dependencies.repos
        -> vcs import --input dependencies.repos ext
```

Do not add a second implementation that parses those pins and performs Git
clone/fetch/checkout itself.

## Create workspace.cad.mcp.models

1. Choose **Use this template** and create:
   `brainboxemb/workspace.cad.mcp.models`.
2. Clone the new workspace repository.
3. Edit committed `workspace.yml`:

```yaml
workspace:
  id: models

repositories:
  cad:
    url: https://github.com/brainboxemb/2026-011-02.cad.solidcraft.git
    branch: dev
```

The MCP and Forge external pins are not in `workspace.yml`; they are owned by
`dependencies.repos`.

4. Create local configuration:

```powershell
Copy-Item workspace.local.example.yml workspace.local.yml
```

Set one exact target owner and the SolidCraft CLI:

```yaml
schema: 1

target:
  model: models/assemblies/simple_mft_assembly

solidcraft:
  cli: C:\\path\\to\\solidcraft-cli.exe
```

5. Run:

```powershell
.\scripts\bootstrap.ps1
.\scripts\doctor.ps1
```

6. Open `.generated/forge-cad.code-workspace`.
7. Configure the VS Code Copilot harness from
   `.generated/mcp-config.json`.
8. Perform an actual MCP `environment` call before model work.

## Checkout classes

The workspace intentionally has two different kinds of Git checkout.

### External toolchain: vcs2l-owned

```text
ext/mcp
ext/forge
```

These are exact external source dependencies declared in
`dependencies.repos`. Bootstrap/update materializes them with vcs2l. Changing
their revision is a committed dependency change.

### Active CAD worktree: workspace-owned location, user-owned Git state

```text
.repos/cad
```

The CAD repository is cloned on first bootstrap using the configured initial
branch. After that, workspace maintenance only verifies the origin. It never
switches/pulls/resets/cleans the worktree because normal CAD work requires local
branches and edits.

This distinction is why the CAD repository is not placed in
`dependencies.repos`.

## Why the target is local

`workspace.yml` says which CAD library belongs to this assistant environment.
`workspace.local.yml` says which exact model you are working on now.

That prevents a commit every time you move from one model to another and avoids
teaching the AI to search the catalog for a likely target.

## Why VS Code opens only the target owner

The checkouts necessarily exist on disk, but the normal assistant project
context should not be all of `ext/mcp`, `ext/forge` and `.repos/cad`.

That would let a client agent browse or edit outside the MCP boundary using
ordinary editor/file tools.

The generated `.code-workspace` therefore exposes the selected
`models/.../<owner>` directory as its normal folder. Forge/MCP and catalog
dependencies remain reachable through their intended MCP interfaces.

This is a workflow boundary, not a filesystem sandbox. A sufficiently
privileged editor agent can still access local files if the client grants it
such permission. The MCP itself nevertheless keeps one exact writable owner and
read-only exact dependency access.

## Bootstrap contract

`scripts/bootstrap.ps1`:

1. creates or updates the single `cad-mcp-workspace` Conda environment;
2. invokes `scripts/import-dependencies.ps1`;
3. that script runs vcs2l through `cad-mcp-workspace` and materializes
   `dependencies.repos` under `ext/`;
4. the Python workspace manager clones the CAD repository only if absent,
   installs MCP/SCPF in the same environment and generates client files.

No repository-management MCP tool is added.

## Update contract

`scripts/update.ps1`:

- reruns vcs2l against committed `dependencies.repos`;
- reinstalls packages from `ext/mcp` and `ext/forge`;
- regenerates client files;
- never updates/switches/resets CAD.

Changing a dependency pin is an explicit `dependencies.repos` source change;
`update.ps1` does not choose newer versions.

## Target selection contract

`scripts/select-model.ps1`:

- accepts a catalog-relative path only;
- rejects absolute paths and `..`;
- requires first path element `models`;
- requires an existing owner with `model.yml`;
- stores the value in ignored `workspace.local.yml`;
- regenerates client files.

It does not change Git branches.

## Template maintenance

Template source uses `main` and `dev`. Normal work:

```text
issue/* -> dev    squash through /merge
dev -> main       merge commit through /merge
main -> dev       merge commit only for stable-only content
```

Keep `docs/plan.md` as the source of truth for unfinished template work.
Do not put concrete model tasks in this repository's plan.

## Compatibility changes

When Forge CAD MCP adopts a different Forge/SolidCraft baseline:

1. update MCP/Forge revisions in `dependencies.repos` in one issue/PR;
2. update the SolidCraft compatibility value/documentation when required;
3. update tests and manuals in the same change;
4. prove vcs2l import + bootstrap + doctor against a concrete workspace;
5. only then update concrete workspace repositories.

Do not float the Forge checkout independently from MCP.
