# SparrowX reusable workflows

Reusable GitHub Actions workflows for the SparrowX sample services. SparrowX is fictional; these workflows demonstrate the platform contract a real small company could offer to application teams onboarding microservices to AWS ECS.

## Workflow catalog

| Workflow | Capability |
| --- | --- |
| `pre-build.yaml` | Detects whether application or deployment-relevant files changed and resolves image metadata |
| `test.yaml` | Runs Python or Node tests, optionally with PostgreSQL |
| `source-security-scan.yaml` | Combines Trivy secret scanning and runtime-aware Semgrep SAST into one report artifact |
| `dast.yaml` | Runs runtime-specific OWASP ZAP API or web baseline scans against a deployed service |
| `build.yaml` | Builds with a Git SHA tag; non-authoritative builds stay local, while authoritative builds push to ECR and skip an existing tag |
| `image-security-scan.yaml` | Combines immutable-image vulnerability scanning and CycloneDX SBOM generation into one report artifact |
| `sign-image.yaml` | Signs an immutable ECR image with keyless Cosign/Sigstore signing |
| `verify-image-signature.yaml` | Verifies the image signature and trusted Sigstore certificate identity |
| `pr-quality-gate.yaml` | Applies PR security policy to source-security reports for secret scanning and SAST |
| `ci-quality-gate.yaml` | Applies CI security policy to image-security reports for vulnerabilities and SBOM |
| `dev-quality-gate.yaml` | Applies development DAST thresholds to the normalized ZAP report |
| `deploy.yaml` | Resolves configuration and database outputs, deploys the shared CloudFormation service stack, waits for stability, and writes summaries/events |
| `smoke-test.yaml` | Calls the configured environment URL and smoke-test path |
| `publish-image-metadata.yaml` | Stores tag, digest, repository, and security metadata in SSM Parameter Store |
| `resolve-image.yaml` | Resolves a previously published tag and digest for deployment or promotion |
| `copy-image.yaml` | Copies the exact image by digest between environment ECR repositories and verifies the target digest |

Workflow usage documentation is available in [`build.md`](build.md), [`source-security-scan.md`](source-security-scan.md), [`image-security-scan.md`](image-security-scan.md), [`dast.md`](dast.md), [`pr-quality-gate.md`](pr-quality-gate.md), [`ci-quality-gate.md`](ci-quality-gate.md), [`dev-quality-gate.md`](dev-quality-gate.md), [`sign-image.md`](sign-image.md), and [`verify-image-signature.md`](verify-image-signature.md). Signing resolves the supplied tag to a digest before signing; verification accepts only a full immutable digest URI.

## Delivery contract

Application repositories call a pinned template release, currently `v4.4.x`, and provide a small `ecs-parameters-dev.yaml` / `ecs-parameters-prod.yaml` pair. The normal flow is:

```text
pull request → source scan → PR quality gate → test → non-authoritative build
main         → authoritative build → image scan → CI quality gate → sign → publish metadata
development  → resolve → deploy → smoke test → DAST → DAST quality gate → publish candidate
manual       → resolve candidate → copy by digest → deploy prod → smoke test
```

This implements **Build Once, Promote Many**. Pull requests use a non-authoritative build for validation. After merge, the caller explicitly sets `authoritative-build: true` to publish the image to ECR. Production receives the same artifact built and tested for development; it is not rebuilt for promotion.

## Rollback support

The workflows support ECS circuit-breaker rollback during a failed rolling deployment, Git revert followed by the normal pipeline, and a manual selected-image rollback workflow in every service repository. The manual workflow accepts a known-good image tag, redeploys it through `deploy.yaml`, runs a smoke test, and republishes deployment metadata.

AWS access uses GitHub OIDC and environment-scoped variables/permissions. The templates are intentionally reusable across both Python APIs and the Node/Vite frontend.

## License

This is a proprietary portfolio project. It is publicly viewable but not open source. All rights are reserved. See [LICENSE.md](LICENSE.md).
