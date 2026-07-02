# Centrally managed Trivy exemptions

The `scan` action (see `actions/scan`) always scans for `HIGH` and `CRITICAL`
findings and fails the build if any are present. Some consuming repositories
have a legitimate, risk-accepted reason to ignore specific `HIGH`/`CRITICAL`
findings (e.g. no upstream fix is available and compensating controls exist).

Rather than letting each repository self-manage its own `.trivyignore` file,
those exemptions are kept here, centrally, so every exemption goes through
review by `@UKHomeOffice/core-cloud-sauron` (enforced by `CODEOWNERS` on this
whole repository) and is visible in one place with a documented reason and
expiry date.

## How it works

When the `scan` action runs, it looks up a file at:

```
exemptions/<owner>/<repo>.trivyignore.yaml
```

using the calling repository's `owner/repo` (`github.repository`) by default.
If that file exists, it's passed to Trivy's `--ignorefile`; if not, the scan
runs with no exemptions, as before. Monorepos scanning more than one image
can override the lookup key per-scan via the `exemption_key` input.

## Requesting an exemption

1. Open a PR against this repository (not the consuming repo) adding or
   updating `exemptions/<owner>/<repo>.trivyignore.yaml`.
2. Every entry **must** include `id`, `statement` (why the risk is accepted)
   and `expired_at` (a review-by date — exemptions are not permanent). Trivy
   stops applying an entry once `expired_at` has passed, so the scan will
   start failing again and the exemption must be actively renewed.
3. Get it approved and merged like any other change in this repo.

## File format

```yaml
# exemptions/Home-Office-Digital/example-service.trivyignore.yaml
vulnerabilities:
  - id: CVE-2023-12345
    statement: "No upstream fix available; mitigated by network policy CCL-1234. Reviewed by #core-cloud-sauron."
    expired_at: "2026-10-01"

  - id: CVE-2023-67890
    # Optional: only exempt the finding for a specific file/package path
    # instead of the whole image.
    paths:
      - "usr/lib/python3.12/site-packages/some-package/METADATA"
    statement: "Vendored copy, unused code path. JIRA-4321."
    expired_at: "2026-09-15"
```

Only `vulnerabilities` entries are honoured by this feature — the scan
action does not run misconfiguration, secret, or license scanning.
