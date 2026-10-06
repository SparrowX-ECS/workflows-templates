# SparrowX reusable workflows

Reusable GitHub Actions workflows for the SparrowX sample services. SparrowX is fictional; these workflows demonstrate the platform contract a real small company could offer to application teams onboarding microservices to AWS ECS.

## Workflow catalog

| Workflow | Capability |
| --- | --- |
| `pre-build.yaml` | Detects whether application or deployment-relevant files changed and resolves image metadata |
| `test.yaml` | Runs Python or Node tests, optionally with PostgreSQL |
| `secret-scanning.yaml` | Scans the checked-out repository filesystem for secrets with Trivy |
| `sast.yaml` | Scans Python, JavaScript, and TypeScript source code with Semgrep |
| `build.yaml` | Builds once, tags with the Git SHA, and pushes to the environment-specific ECR namespace; skips an existing tag |
| `security-scan.yaml` | Scans the immutable image with Trivy |
| `deploy.yaml` | Resolves configuration and database outputs, deploys the shared CloudFormation service stack, waits for stability, and writes summaries/events |
| `smoke-test.yaml` | Calls the configured environment URL and smoke-test path |
| `publish-image-metadata.yaml` | Stores tag, digest, repository, and security metadata in SSM Parameter Store |
| `resolve-image.yaml` | Resolves a previously published tag and digest for deployment or promotion |
| `copy-image.yaml` | Copies the exact image by digest between environment ECR repositories and verifies the target digest |

## Delivery contract

Application repositories call a pinned template release, currently `v4.4.x`, and provide a small `ecs-parameters-dev.yaml` / `ecs-parameters-prod.yaml` pair. The normal flow is:

```text
pull request → test → build → Trivy scan → publish dev metadata
main         → resolve → deploy dev → smoke test → publish prod candidate
manual       → resolve candidate → copy by digest → deploy prod → smoke test
```

This implements **Build Once, Promote Many**. Production receives the same artifact built and tested for development; it is not rebuilt for promotion.

## Rollback support

The workflows support ECS circuit-breaker rollback during a failed rolling deployment, Git revert followed by the normal pipeline, and a manual selected-image rollback workflow in every service repository. The manual workflow accepts a known-good image tag, redeploys it through `deploy.yaml`, runs a smoke test, and republishes deployment metadata.

AWS access uses GitHub OIDC and environment-scoped variables/permissions. The templates are intentionally reusable across both Python APIs and the Node/Vite frontend.

## License

This is a proprietary portfolio project. It is publicly viewable but not open source. All rights are reserved. See [LICENSE.md](LICENSE.md).
