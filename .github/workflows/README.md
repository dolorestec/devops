### Workflows

- [setup-environment](#setup-environment)
- [docker-publish](#docker-publish)
- [release-gitlow](#release-gitlow)

### setup-environment

- `setup-environment` is a workflow that runs on `workflow_call` event.
- It is used to setup the environment for the other workflows.

#### Usage

```yaml
jobs:  
  setup:
    name: Setup environment
    secrets: inherit
    uses: dolorestec/devops/.github/workflows/setup-environment.yml@develop
    with:
      github_token: ${{ secrets.GITHUB_TOKEN }}
      workflow: ${{ github.event.inputs.workflow }}
      workflow_ref: ${{ github.event.inputs.workflow_ref }}
```

### docker-publish

### release-gitlow
