# Sekrd GitHub Action

Run a [Sekrd](https://sekrd.com) deep security scan against your deployed site
from any GitHub workflow. Uploads findings as SARIF so they show up in the
**Security** tab. Posts a PR comment with score + critical/high findings.

Built for teams deploying AI-built / vibe-coded apps (Lovable, Bolt, v0,
Next.js on Vercel, Supabase, Firebase) that want a per-PR security signal
without setting up self-hosted DAST infrastructure.

## Quick start

```yaml
name: Security
on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read
  security-events: write  # required for SARIF upload
  pull-requests: write    # required for PR comments

jobs:
  sekrd:
    runs-on: ubuntu-latest
    steps:
      - uses: sekrdcom/sekrd-action@v1
        with:
          url: ${{ vars.PREVIEW_URL || 'https://staging.example.com' }}
          api-key: ${{ secrets.SEKRD_API_KEY }}
```

Get an API key: https://sekrd.com/dashboard/settings (Pro plan required for deep scans).

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `url` | ✓ | — | Public URL to scan (preview deploys work) |
| `api-key` | ✓ | — | Sekrd API key — store as secret |
| `scan-type` |   | `deep` | `deep` (15 providers, Pro) or `free` (3 providers) |
| `fail-on` |   | `critical` | Severity threshold: `critical`, `high`, `medium`, `low`, `never` |
| `upload-sarif` |   | `true` | Upload SARIF to GitHub Code Scanning |
| `comment-on-pr` |   | `true` | Post score + top findings as PR comment |

## Outputs

| Output | Description |
|---|---|
| `scan-id` | The scan UUID |
| `score` | Score 0-100 (higher = more secure) |
| `verdict` | `SHIP`, `BLOCK_GRACE`, or `BLOCK` |
| `critical-count` | Number of critical findings |
| `high-count` | Number of high findings |
| `report-url` | Public HTML report URL |

## What Sekrd scans

15 providers running in parallel per deep scan:

- **Secrets** — leaked Stripe / OpenAI / GitHub / AWS keys in client JS
- **Supabase** — RLS bypass detection (anon-key table reads, open policies)
- **Firebase** — Firestore / RTDB / Storage public-rule detection (multi-region)
- **Payments** — Stripe / Paddle / LemonSqueezy webhook signature verification
- **Auth** — security headers (CSP, HSTS, COOP, CORP, Permissions-Policy), cookie flags
- **DAST** — Nuclei-backed: CSRF, rate limiting, error-page leaks, open redirects, reflected XSS
- **Active DAST** — CORS origin reflection, IDOR sequential-ID probes
- **Compliance** — GDPR posture (privacy policy, cookie consent, data deletion, trackers)
- **JS Security** — dangerous sinks (`innerHTML=`, `eval`), internal-URL leaks, localStorage auth tokens
- **MCP** — Model Context Protocol server security (7 rules: tool-poisoning, rug-pull, command injection)
- **Cost Exposure** — public API keys (Google Maps, Mapbox) without domain restriction
- **Network** — edge port scanning + SSL/TLS posture
- **Infra** — DNS email security (SPF, DMARC)
- **Dependencies** — known-vulnerable package detection
- **IaC** — Terraform / CloudFormation posture (where accessible)

Every finding carries CWE, OWASP Top 10 2021, ASVS 4.0, and CVSS v3.1 — see the full
[CWE mapping](https://sekrd.com/docs/cwe-mapping).

## Example: fail PR only on critical

```yaml
- uses: sekrdcom/sekrd-action@v1
  with:
    url: https://deploy-preview-123--my-site.netlify.app
    api-key: ${{ secrets.SEKRD_API_KEY }}
    fail-on: critical       # default
```

## Example: block on high too

```yaml
- uses: sekrdcom/sekrd-action@v1
  with:
    url: https://staging.example.com
    api-key: ${{ secrets.SEKRD_API_KEY }}
    fail-on: high
```

## Example: report only, don't fail the build

```yaml
- uses: sekrdcom/sekrd-action@v1
  with:
    url: https://my-site.com
    api-key: ${{ secrets.SEKRD_API_KEY }}
    fail-on: never
    upload-sarif: true
    comment-on-pr: true
```

## Example: use the scan-id output

```yaml
- uses: sekrdcom/sekrd-action@v1
  id: sekrd
  with:
    url: https://my-site.com
    api-key: ${{ secrets.SEKRD_API_KEY }}

- name: Notify Slack on critical
  if: steps.sekrd.outputs.critical-count > 0
  run: |
    curl -X POST -H 'Content-Type: application/json' \
      -d "{\"text\":\"🚨 Sekrd found ${{ steps.sekrd.outputs.critical-count }} critical on ${{ steps.sekrd.outputs.report-url }}\"}" \
      $SLACK_WEBHOOK_URL
```

## Privacy

- Sekrd scans black-box: no source code uploaded. We fetch the public URL and analyze the response.
- API keys are stored encrypted at rest.
- Scan results are retained per your plan's retention policy (see [sekrd.com/pricing](https://sekrd.com/pricing)).
- Full privacy policy: https://sekrd.com/privacy
- Responsible-disclosure policy: https://sekrd.com/security-policy

## License

MIT. See `LICENSE`.

## Links

- Product: https://sekrd.com
- Pricing: https://sekrd.com/pricing
- Rule catalog: https://sekrd.com/rules
- CWE mapping: https://sekrd.com/docs/cwe-mapping
- Docs: https://sekrd.com/docs
- Trust center: https://sekrd.com/trust
