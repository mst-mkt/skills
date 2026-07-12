---
name: check-actions-workflow
description: Checks GitHub Actions workflows with pinact, actionlint, and zizmor. Use after creating or editing files under .github/workflows/, or when the user asks to validate, lint, or harden GitHub Actions workflows.
allowed-tools:
  - Bash(nix run nixpkgs#pinact -- run -u --min-age 3 .github/workflows/*.yaml)
  - Bash(nix run nixpkgs#actionlint -- .github/workflows/*.yaml)
  - Bash(nix run nixpkgs#zizmor -- --fix .github/workflows/*.yaml)
---

# Check Actions Workflow

Run against the workflow files you created or edited under `.github/workflows/`, passing them explicitly to each command.

## 1. Use the .yaml extension

Rename `.yml` workflow files to `.yaml`.

## 2. Pin and update actions (pinact)

```sh
nix run nixpkgs#pinact -- run -u --min-age 3 <workflow files>
```

Rewrites `uses:` to full commit SHAs with version comments and updates them to the latest release at least 3 days old (supply-chain cooldown; errors if no release qualifies).

## 3. Check action trustworthiness

Review every `uses:` reference at the version pinact just pinned. `actions/` and `github/` are the trust baseline. For anything else, check the source repository (author and code); a publisher the repository already depends on is not automatically safe, since new versions can be compromised. Warn the user and do not use an untrusted action without their approval.

## 4. Lint for correctness (actionlint)

```sh
nix run nixpkgs#actionlint -- <workflow files>
```

Checks schema, `${{ }}` expression types, action inputs, and `run:` scripts via shellcheck. Fix all errors.

## 5. Lint for security (zizmor)

```sh
nix run nixpkgs#zizmor -- --fix <workflow files>
```

Detects template injection, dangerous triggers, credential leaks, etc. `--fix` applies safe fixes; fix the rest manually. Exit codes: 0 = clean, 11-14 = findings (not a tool failure).

## 6. Converge

Re-run actionlint and zizmor after fixes until both pass. If a finding cannot or should not be fixed, report it to the user; never suppress findings (`# zizmor: ignore` comments, loosening flags) without their explicit approval.

## Optional: execute locally (wrkflw)

Only if the user wants to verify runtime behavior. Confirm with the user first: it can take a long time, and emulation mode executes steps directly on the host without isolation, so only run workflows that passed the trust check.

```sh
nix run nixpkgs#wrkflw -- run <workflow file> --runtime emulation
```
