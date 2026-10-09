# Production Quality Gate

Reusable workflow: `.github/workflows/prod-quality-gate.yaml`

## Purpose

Evaluates production smoke-test and deployed API-test statuses. The gate itself does not fail its job; it reports a decision so the caller can publish successful deployment metadata or start a rollback.

## Usage

```yaml
jobs:
  prod-quality-gate:
    needs: [smoke-test, api-test]
    if: always()
    uses: SparrowX-ECS/workflows-templates/.github/workflows/prod-quality-gate.yaml@<version>
    with:
      smoke-test-status: ${{ needs.smoke-test.outputs.status }}
      api-test-status: ${{ needs.api-test.outputs.status }}
```

## Inputs

| Input | Required | Expected values / description |
|---|---:|---|
| `smoke-test-status` | yes | `passed` or `failed` from `smoke-test.yaml` |
| `api-test-status` | yes | `passed` or `failed` from `api-test.yaml` |

## Outputs

| Output | Values | Description |
|---|---|---|
| `status` | `passed`, `failed` | `passed` only when both input statuses equal `passed` |

## Reports and summary

No JSON report is generated. The summary lists both input statuses and the resulting gate decision. The workflow intentionally exits successfully for either decision; downstream jobs must branch on `needs.prod-quality-gate.outputs.status`.
