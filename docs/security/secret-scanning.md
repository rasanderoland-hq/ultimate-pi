# Secret scanning in this repo

Story: ROE-640 (leak fix 1, 2026-10-27). Canonical policy: rasande-knowledge
`docs/quality/secret-scanning-strategy.md`.

## Root cause

The house rule "never commit secrets" was prose only. The `pr-fast` gitleaks
step is opt-in (`vars.PR_FAST_GITLEAKS`) and was skipped here, the repo had no
`.gitleaks.toml` with the house token shapes (`vcp_`, `ctx7sk-`, `kupan_sk_`,
`GOCSPX-`), and no agent host ran a pre-commit hook. GitHub push protection is
not available for private repos on the free plan. Leaked values were found
only after the fact.

## Fix

- `.gitleaks.toml`: gitleaks default rules plus the house rules (shared copy
  from rasande-knowledge `templates/security/gitleaks.toml`; do not fork it).
- `.pre-commit-config.yaml`: gitleaks hook for the pre-commit framework.
- Agent hosts (box, 7420) run a global `core.hooksPath` pre-commit hook that
  runs `gitleaks git --staged --redact` (rasande-knowledge
  `scripts/install-global-git-hooks.sh`).
- `pr-fast` scans each PR's own commit range when the repo variable
  `PR_FAST_GITLEAKS` is `true`.

Agents never add an allowlist, `.gitleaksignore` entry or baseline. Existing
findings are listed (rule id and file only) for Gustaf to triage.

## Verify

```bash
gitleaks dir . --redact --no-banner            # working tree, uses .gitleaks.toml
gitleaks git --redact --no-banner --log-opts="origin/main..HEAD"   # your branch
gh variable list | grep PR_FAST_GITLEAKS       # expect true (not on Gentle Buffalo repos until Gustaf says yes)
```

In a PR, the `pr-fast` step "Secret scan (gitleaks, opt-in)" must show as run
(not skipped).
