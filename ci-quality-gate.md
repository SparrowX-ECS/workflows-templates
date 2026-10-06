# CI Quality Gate

Reusable workflow: `.github/workflows/ci-quality-gate.yaml`

## Purpose

Downloads the source-security and image-security artifacts, validates their reports, reads the application security policy, and decides whether CI may continue.

It evaluates secret scanning, SAST, image vulnerabilities, and SBOM generation. It returns `failed` for policy violations, missing/invalid reports, scanner errors, or an invalid policy file.

## Usage

```yaml
jobs:
  ci-quality-gate:
    if: always()
    needs: [source-security-scan, image-security-scan]
    uses: SparrowX-ECS/workflow-templates/.github/workflows/ci-quality-gate.yaml@<version>
    with:
      source-security-artifact: ${{ needs.source-security-scan.outputs.artifact-name }}
      image-security-artifact: ${{ needs.image-security-scan.outputs.artifact-name }}
      security-policy-file: app-sec-policy.yaml
```

The policy file must exist in the application repository because this workflow checks out the repository before reading it.

## Inputs

| Input | Required | Description |
|---|---:|---|
| `source-security-artifact` | yes | Artifact name returned by `source-security-scan.yaml` |
| `image-security-artifact` | yes | Artifact name returned by `image-security-scan.yaml` |
| `security-policy-file` | yes | Repository path to the application security policy |

## Outputs

| Output | Values | Description |
|---|---|---|
| `status` | `passed`, `failed` | Overall CI security decision |

## Policy file format

```yaml
secret_scanning:
  block_on_findings: true
sast:
  max_findings: 0
image_scan:
  max_critical: 0
  max_high: 0
sbom:
  required: true
```

The quality gate loads and validates all five policy values before evaluating reports.

## Consumed JSON reports

The source artifact must contain:

```text
combined-source-security-scan-results.json
trivy-secret-results.json
semgrep-results.json
```

The image artifact must contain:

```text
combined-image-security-scan-results.json
trivy-results.json
sbom.cdx.json
```

The gate reads the combined reports for policy evaluation and recalculates image totals, CRITICAL findings, and HIGH findings from `trivy-results.json` to verify the combined report.

## Summary

The workflow summary is ordered as follows:

1. Overall CI security gate result.
2. Image security gate result.
3. Failure reasons, when applicable.
4. Secret scanning, SAST, image scan, and SBOM statuses and findings.
5. Links to the workflow artifact containing each JSON report.
