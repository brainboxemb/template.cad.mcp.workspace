# Developer manual

Use this document when creating or maintaining a concrete workspace repository from template.cad.mcp.workspace.

The end-user/model workflows are not duplicated here. After the concrete repository is configured, follow docs/manual/01-installation.md and then one of the model workflows under docs/manual/.

## Create a concrete workspace repository

1. Open brainboxemb/template.cad.mcp.workspace on GitHub.
2. Choose **Use this template**.
3. Create the concrete repository, for example brainboxemb/workspace.cad.mcp.models.
4. Clone that new repository.
5. Edit its committed workspace.yml so workspace.id and repositories.cad identify the concrete workspace/catalog.
6. Keep dependencies.repos aligned with the accepted MCP/Forge pair from the template.
7. Commit the concrete workspace identity.
8. Do not commit workspace.local.yml, ext/, .repos/ or .generated/.

For the models workspace the committed CAD identity is:

~~~yaml
workspace:
  id: models

repositories:
  cad:
    url: https://github.com/brainboxemb/2026-011-02.cad.solidcraft.git
    branch: dev
~~~

On first bootstrap the CAD repository is cloned as the configured development
branch only, with a blobless partial clone. This keeps generated or historical
large blobs from unrelated refs out of the initial workspace download. Once the
checkout exists, workspace maintenance only validates its origin and never
changes its branch or worktree state.

MCP and Forge are different: they are disposable vcs2l-managed externals below
`ext/`. Bootstrap and update validate them against `dependencies.repos`.
When a committed release tag changes, the workspace rematerializes the clean
external set through vcs2l. It must never require the operator to manually
delete `ext/`, and it must refuse to discard local external changes or an
unexpected repository origin.

After that, stop using this document for normal setup and follow:

1. docs/manual/01-installation.md
2. docs/manual/02-new-model.md
3. docs/manual/03-solidcraft-to-forge.md
4. docs/manual/04-existing-model.md

## Reusable behavior ownership

Reusable workspace behavior belongs in this template. Concrete workspace repositories own only their catalog identity, local acceptance state and any intentional consumer-specific differences.

Shared behavior includes:

- environment.yml;
- dependencies.repos layout;
- scripts/;
- tests/;
- workflow/governance files;
- docs/manual/ user procedures.

Do not silently advance MCP/Forge revisions while synchronizing unrelated changes.

## Source integration

Normal template work uses a short-lived branch from dev, then /merge to dev.
Accepted dev checkpoints go to main through a dev-to-main PR and /merge.

For local operator use, install GitHub CLI once and authenticate it:

~~~powershell
gh --version
gh auth status
~~~

If `gh` is missing, install GitHub CLI and run `gh auth login`.

The workspace helper can publish the current repository branch:

~~~powershell
.\scripts\pr.ps1 publish -Title "Describe the change"
~~~

For the active CAD worktree, point the same helper at that repository:

~~~powershell
.\scripts\pr.ps1 publish -Path .repos\cad -Title "Describe the model change"
~~~

`publish` pushes the branch, opens the PR, waits for PR checks and then posts
`/merge`. It never calls GitHub's merge endpoint directly. The repository
merge-policy workflow remains authoritative for the allowed branch integration
and merge method.

The actions can also be separated:

~~~powershell
.\scripts\pr.ps1 open -Title "Describe the change"
.\scripts\pr.ps1 merge
~~~

The normal base is inferred as short-lived branch -> `dev` and `dev` ->
`main`. Use `-Base` only for an intentional exceptional lane such as the
allowed stable `main` -> `dev` sync.

Keep docs/plan.md as the source of truth for unfinished template work.
