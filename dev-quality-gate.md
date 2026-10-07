# Dev Quality Gate

Reusable workflow: `.github/workflows/dev-quality-gate.yaml`

## Purpose

Downloads the artifact from `dast.yaml`, validates the normalized and raw ZAP reports, and applies repository policy thresholds to HIGH and MEDIUM DAST findings.

## Usage

```yaml
jobs:
  dev-quality-gate:
    if: always()
    needs: dast
    uses: SparrowX-ECS/workflows-templates/.github/workflows/dev-quality-gate.yaml@<version>
    with:
      dast-artifact: ${{ needs.dast.outputs.artifact-name }}
      security-policy-file: app-sec-policy.yaml
```

## Inputs

| Input | Required | Description |
|---|---:|---|
| `dast-artifact` | yes | Artifact name returned by `dast.yaml` |
| `security-policy-file` | yes | Repository path to the DAST policy |

## Outputs

| Output | Values | Description |
|---|---|---|
| `status` | `passed`, `failed` | Overall DAST quality-gate decision |

## Policy format

```yaml
dast:
  max_high: 0
  max_medium: 0
```

Both values must be non-negative integers. Findings at or below the configured thresholds pass; findings above a threshold fail the gate. Missing or invalid policy values fail the gate.

## Consumed reports

The DAST artifact must contain:

```text
combined-dast-results.json
zap-results.json
zap-report.html
```

The gate validates the two JSON files and recalculates total, HIGH, and MEDIUM counts from `zap-results.json` to detect mismatches in the normalized report.

## Summary

The workflow writes `## Dev Quality Gate` to `$GITHUB_STEP_SUMMARY`, including the gate result, finding counts, failure reasons, and a link to the report artifact. It does not create a new report file.

API testing is intentionally not included yet because no API-testing workflow or report schema exists. It can be added later as a separate input and control with its own normalized report contract.
