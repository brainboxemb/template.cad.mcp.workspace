# Developer manual — workspace template and consumers

Use this document for local development of the template and for creating or updating a concrete consumer. VS Code, Git, Conda and GitHub CLI are sufficient; an IDE coding assistant is optional. Read AGENTS.md and the current main plan first.

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

A basic bootstrap or standalone workbench does not clone the CAD repository.
The configured CAD repository is materialized only when a catalog-model target
is used. That first catalog checkout clones only the configured development
branch with a blobless partial clone. Once the checkout exists, workspace
maintenance only validates its origin and never changes its branch or worktree
state.

MCP and Forge are different: they are disposable vcs2l-managed externals below
`ext/`. Bootstrap and update validate them against `dependencies.repos`.
When a committed release tag changes, the workspace rematerializes the clean
external set through vcs2l. It must never require the operator to manually
delete `ext/`, and it must refuse to discard local external changes or an
unexpected repository origin.

After that, stop using this document for normal setup and follow:

1. docs/manual/01-installation.md
2. docs/manual/02-local-workbench.md
3. docs/manual/02-new-model.md
4. docs/manual/03-solidcraft-to-forge.md
5. docs/manual/04-existing-model.md

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

## Develop the template in VS Code

Clone the template repository separately from the concrete workspace. Install
Git, VS Code (Python extension), Conda and GitHub CLI. From PowerShell:

~~~powershell
git clone https://github.com/brainboxemb/template.cad.mcp.workspace.git
cd template.cad.mcp.workspace
conda env create -f environment.yml
conda activate cad-mcp-workspace
code .
~~~

If the environment already exists, use `conda env update -n cad-mcp-workspace -f environment.yml` instead. Select its Python interpreter in VS Code.
For template unit tests there is no need to bootstrap a CAD target, clone
the catalog, install SolidCraft or materialize the externals. Use:

~~~powershell
.\scripts\test.ps1
~~~

This runs all three test groups from CI. In VS Code use pytest in tests/ for
targeted debugging; Python module source paths for policy tests are set by
the test script. Windows symlink permissions may affect platform-sensitive
tests; investigate any failure rather than skipping it silently.

For integration, create a separate disposable concrete consumer from the
template with an explicit catalog identity, then use its normal user manual
and bootstrap. Never test destructive changes against an active user CAD
checkout. Keep the normal user environment separate from experimental server
source installs. For AI-assisted development, open the template source
checkout, read AGENTS.md and give the IDE assistant one issue and its
acceptance criteria. Review and test code yourself.

## Source integration

Normal template work uses a short-lived branch from dev, then /merge to dev.
Accepted dev checkpoints go to main through a dev-to-main PR and /merge.
Plan/handoff-only changes use a main-based branch targeting main, per
AGENTS.md. Do not edit the canonical plan on dev.

After creating an issue, for example #123:

~~~powershell
.\scripts\branch.ps1 -Issue 123 -Name improve-workspace
# edit and review in VS Code
.\scripts\test.ps1
git diff
.\scripts\commit.ps1 -Message "Improve workspace (#123)" -Files @("scripts/workspace.py","tests/test_workspace.py")
.\scripts\pr.ps1 open -Title "Improve workspace (#123)" -Body "Refs #123. Test evidence: ..."
# review the PR, then request the governed merge
.\scripts\pr.ps1 merge
~~~

The branch helper fetches the selected base and refuses a dirty checkout.
The commit helper requires explicit files and an empty index. It does not
stage the whole repository. Inspect the result of /merge on GitHub before
continuing. For a stable checkpoint switch to dev, pull --ff-only and use
pr.ps1 open -Base main followed by pr.ps1 merge after review. The template
publishes stable source on main; it has no separate package release script.

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


## Adopt a template change in an existing consumer

GitHub's "Use this template" creates an initial copy, not an automatically
synchronized fork. Never merge unrelated template history into a consumer
or replace the consumer tree wholesale.

1. Complete and integrate the shared change in template dev, then main.
   Record the exact stable template commit and the previously adopted
   revision in a consumer issue. Review the template diff for that interval.
2. In the consumer, create an issue branch from its dev. Fetch the template
   under an explicit remote name (once per clone):

~~~powershell
git remote add workspace-template https://github.com/brainboxemb/template.cad.mcp.workspace.git
git fetch workspace-template main
git log -1 workspace-template/main
git diff <previous-template-sha> <new-template-sha> -- scripts tests docs environment.yml dependencies.repos
~~~

Replace the SHA placeholders with reviewed full commit IDs. If the previous
revision is unknown, compare the affected files individually and establish
the baseline; do not assume common ancestry. If the remote already exists,
check its URL rather than adding it again.

3. Transfer only the reviewed shared changes. For an unchanged shared file,
   use `git restore --source=<new-template-sha> -- <explicit-file>`; for a
   consumer-modified file, apply the relevant hunks manually in VS Code.
   Review deletions/renames explicitly. Preserve consumer workspace.yml
   identity/catalog, README, main-owned plan/handoff, intentional overrides,
   dependency pins unless adoption explicitly changes them, workspace.local.yml,
   ext/, .repos/cad/, .workbench/ and .generated/. Never use git reset --hard,
   clean, a blanket restore or a whole-template merge.
4. Review the consumer diff, run its tests (use scripts/test.ps1 once
   adopted), commit explicit files, open PR to dev and integrate via /merge.
   Integrate dev to main through its own reviewed PR. Record the adopted
   template SHA, files and any consumer differences in the issue.
5. On the operator's Windows machine, pull accepted consumer main,
   run scripts/update.ps1 when runtime/config/dependency changes require it,
   and run scripts/doctor.ps1 after selecting a target. Do not reset the
   active CAD checkout. Run appropriate real acceptance.

A released MCP/Forge dependency upgrade is a separate reviewed change in
dependencies.repos. Check the upstream compatibility pair and release tags;
never use dev/exp as a normal user dependency. Template adoption does not
automatically change those pins. A docs-only change needs no toolchain
reinstallation.

The consumer is not updated merely because the template PR merged.
