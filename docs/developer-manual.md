# Developer manual

Use this manual when you want to create a new concrete Forge CAD MCP workspace
repository from `template.cad.mcp.workspace`.

Example:

```text
template.cad.mcp.workspace
        ↓ Use this template
workspace.cad.mcp.models
```

At the end of this procedure you will have:

- one concrete workspace repository;
- one local Conda environment named `cad-mcp-workspace`;
- exact MCP and Forge checkouts under `ext/`;
- one active CAD repository under `.repos/cad/`;
- one selected CAD model owner;
- generated VS Code and MCP client configuration;
- a working `doctor` check.

## Before you start

The machine needs:

- Git;
- Conda available as `conda` in PowerShell;
- access to the GitHub repositories used by the workspace;
- a working SolidCraft CLI installation and its executable path.

You do **not** install vcs2l separately and you do **not** create a Conda
environment by hand. The workspace bootstrap does both.

## 1. Create the workspace repository

On GitHub, open `brainboxemb/template.cad.mcp.workspace` and choose
**Use this template**.

Create the new repository, for example:

```text
brainboxemb/workspace.cad.mcp.models
```

The new repository now contains the scripts and configuration needed for the
local workspace, but it does not yet contain the MCP, Forge or CAD checkouts.

## 2. Clone the new workspace repository

In PowerShell:

```powershell
git clone https://github.com/brainboxemb/workspace.cad.mcp.models.git
cd workspace.cad.mcp.models
```

Run all following commands from this repository root unless stated otherwise.

## 3. Configure the CAD repository

Edit committed `workspace.yml`.

For the models workspace:

```yaml
schema: 1

workspace:
  id: models

environment:
  conda_name: cad-mcp-workspace

repositories:
  cad:
    url: https://github.com/brainboxemb/2026-011-02.cad.solidcraft.git
    branch: dev

solidcraft:
  version: 0.49.0

paths:
  externals: ext
  cad: .repos/cad
  generated: .generated
```

This file identifies the concrete CAD repository for this workspace.

Do not put the MCP and Forge repository revisions in `workspace.yml`. Their
exact revisions are already committed in `dependencies.repos`.

Commit the concrete `workspace.yml` to the new workspace repository.

## 4. Configure your local target

Create the local configuration:

```powershell
Copy-Item workspace.local.example.yml workspace.local.yml
```

Edit `workspace.local.yml`:

```yaml
schema: 1

target:
  model: models/assemblies/simple_mft_assembly

solidcraft:
  cli: C:\path\to\solidcraft-cli.exe
```

`target.model` is the exact owner you want the MCP to edit.

The file is intentionally ignored by Git. Changing the active model is a local
workspace action, not a repository change.

It is fine that the CAD repository has not been cloned yet. Bootstrap will
clone it first and then validate this target path.

## 5. Run bootstrap

Run:

```powershell
.\scripts\bootstrap.ps1
```

You do not create or activate `cad-mcp-workspace` yourself.

On the first run, `bootstrap.ps1` does this in order:

1. checks whether the Conda environment `cad-mcp-workspace` exists;
2. if it does not exist, creates it from `environment.yml`;
3. if it already exists, updates that same environment from
   `environment.yml`;
4. that environment installs Python 3.12, vcs2l and the workspace Python
   dependencies;
5. runs vcs2l from that environment:

   ```text
   vcs import --input dependencies.repos ext
   ```

6. vcs2l materializes the exact MCP and Forge revisions as:

   ```text
   ext/mcp/
   ext/forge/
   ```

7. clones the configured CAD repository to `.repos/cad/` when it does not
   exist yet;
8. installs the MCP package from `ext/mcp` and SCPF from
   `ext/forge/cadlib` into the same `cad-mcp-workspace` environment;
9. validates the selected model owner from `workspace.local.yml`;
10. generates the VS Code workspace and MCP client configuration below
    `.generated/`.

There is one Conda environment for normal workspace use:

```text
cad-mcp-workspace
├── Python
├── vcs2l
├── Forge CAD MCP
├── SCPF
└── workspace Python dependencies
```

There is no normal step where you switch to another Conda environment.

After a successful bootstrap the relevant local tree is:

```text
workspace.cad.mcp.models/
├── ext/
│   ├── mcp/
│   └── forge/
├── .repos/
│   └── cad/
├── .generated/
│   ├── forge-cad.code-workspace
│   ├── mcp-config.json
│   └── state.json
└── workspace.local.yml
```

## 6. Run doctor

Run:

```powershell
.\scripts\doctor.ps1
```

Doctor runs through the same `cad-mcp-workspace` environment and validates the
configured toolchain, CAD repository, selected owner and SolidCraft CLI.

Do not continue to VS Code until doctor succeeds.

## 7. Open the generated VS Code workspace

Open:

```text
.generated/forge-cad.code-workspace
```

That workspace exposes the selected model owner as the normal project folder.
It does not expose the complete MCP, Forge and CAD checkout trees as ordinary
project folders.

The generated MCP stdio definition is:

```text
.generated/mcp-config.json
```

Use that definition for the VS Code Copilot MCP configuration. Do not copy the
absolute paths into a second manually maintained configuration.

## 8. Verify the MCP from the assistant

Start a new assistant session in the generated VS Code workspace.

First ask the assistant to call:

```text
environment
```

Verify that:

- `cad_ready` is true;
- the reported CAD library is the intended catalog;
- the reported target is the exact owner from `workspace.local.yml`.

Then ask it to call:

```text
skill_list
```

At this point the workspace is ready for CAD work.

## Creating another workspace

For another catalog or purpose, create another repository from the template and
repeat this procedure. Do not reuse one workspace by silently changing its
committed CAD repository identity.

## How the checkouts are divided

This distinction matters after setup, but you do not need it in order to run
bootstrap.

### MCP and Forge

`dependencies.repos` contains exact external revisions. vcs2l materializes
them under `ext/`.

Changing these revisions is a reviewed workspace/template dependency change.

### CAD repository

The CAD repository is the active worktree under `.repos/cad/`.

Bootstrap may clone it when absent. Once it exists, workspace maintenance does
not pull, reset, clean or switch it. Normal CAD branches and edits live there.

That is why the CAD repository is not an entry in `dependencies.repos`.

## Maintaining the template

Reusable bootstrap or workspace behavior belongs in
`brainboxemb/template.cad.mcp.workspace`.

Normal template development uses:

```text
issue/* -> dev    squash through /merge
dev -> main       merge commit through /merge
```

When a reusable change is accepted in the template, synchronize that change to
concrete workspace repositories without overwriting their consumer-specific
`workspace.yml`, plan, handoff or repository identity.
