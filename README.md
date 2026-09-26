# SparrowX Shared Workflows

Reusable GitHub Actions workflows used by the SparrowX ECS application repositories.

## Workflows

- `changes.yaml` detects application and deployment-relevant changes.
- `test.yaml` runs Python or Node.js tests and can start PostgreSQL for database-backed services.
- `build.yaml` builds Docker images and pushes immutable Git-SHA tags to ECR.
- `deploy.yaml` reads `ecs-parameters.yaml`, resolves shared CloudFormation outputs, and deploys the reusable ECS service template from `ecs-infrastructure`.

Application repositories call these workflows with `workflow_call`, for example:

```yaml
jobs:
  test:
    uses: SparrowX-ECS/workflows-templates/.github/workflows/test.yaml@v1.0.2
    with:
      runtime: python
      database: true
```

The templates centralize the platform’s CI/CD conventions while keeping service-specific configuration in each application repository’s `ecs-parameters.yaml`.
