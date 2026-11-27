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

Normal template work uses a short-lived branch from dev, then /merge to dev. Accepted dev checkpoints go to main through a dev-to-main PR and /merge.

Keep docs/plan.md as the source of truth for unfinished template work.
