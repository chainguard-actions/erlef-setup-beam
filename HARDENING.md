<!-- markdownlint-disable -->

# Hardening Report: erlef--setup-beam/v1.24.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **erlef--setup-beam/v1.24.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across every workflow file use mutable version tags instead of pinned 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved or compromised. Failing references include: `actions/checkout@v6.0.2`, `actions/setup-node@v6.3.0`, `raven-actions/actionlint@v2.1.2`.

Locations:

- `.github/workflows/action.yml:26`
- `.github/workflows/action.yml:37`
- `.github/workflows/action.yml:38`
- `.github/workflows/action.yml:46`
- `.github/workflows/action.yml:47`
- `.github/workflows/action.yml:57`
- `.github/workflows/action.yml:58`
- `.github/workflows/action.yml:74`
- `.github/workflows/action.yml:75`
- `.github/workflows/action.yml:91`
- `.github/workflows/action.yml:92`
- `.github/workflows/hexpm-mirrors.yml:22`
- `.github/workflows/macos.yml:71`
- `.github/workflows/macos.yml:101`
- `.github/workflows/ubuntu.yml:82`
- `.github/workflows/ubuntu.yml:143`
- `.github/workflows/update_3rd_party_licenses.yml:14`
- `.github/workflows/windows.yml:67`
- `.github/workflows/windows.yml:128`

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks directly interpolate GitHub Actions expressions (`${{ ... }}`) inside shell command strings. This allows expression values to be interpreted as shell code before the shell ever sees them. Affected patterns include: (1) `run: echo "Erlang/OTP ${{steps.setup-beam.outputs.otp-version}}"` — step output interpolated directly into shell; (2) `run: echo "Elixir ${{steps.setup-beam.outputs.elixir-version}}"` — same; (3) `run: echo "rebar3 ${{steps.setup-beam.outputs.rebar3-version}}"` — same; (4) `mix escript.install --force ${{matrix.combo.escript_packages}}` — matrix value interpolated as shell argument; (5) `${{matrix.combo.escript_script}}` — matrix value executed directly as a shell command, the most severe case. All these should use env: variables with double-quoted shell expansions instead.

Locations:

- `.github/workflows/macos.yml:81`
- `.github/workflows/macos.yml:84`
- `.github/workflows/macos.yml:87`
- `.github/workflows/macos.yml:101`
- `.github/workflows/macos.yml:102`
- `.github/workflows/ubuntu.yml:90`
- `.github/workflows/ubuntu.yml:92`
- `.github/workflows/ubuntu.yml:94`
- `.github/workflows/ubuntu.yml:96`
- `.github/workflows/ubuntu.yml:115`
- `.github/workflows/ubuntu.yml:116`
- `.github/workflows/windows.yml:73`
- `.github/workflows/windows.yml:75`
- `.github/workflows/windows.yml:77`
- `.github/workflows/windows.yml:79`
- `.github/workflows/windows.yml:103`
- `.github/workflows/windows.yml:104`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all unpinned `uses:` references by pinning to full 40-character SHA hashes: actions/checkout@v6.0.2→de0fac2e..., actions/setup-node@v6.3.0→53b83947..., raven-actions/actionlint@v2.1.2→205b530c... across action.yml, hexpm-mirrors.yml, macos.yml, ubuntu.yml, update_3rd_party_licenses.yml, and windows.yml. Fixed script injection by moving all ${{ steps.setup-beam.outputs.* }} expressions in run: blocks to env: blocks (OTP/Elixir/Gleam/rebar3 version echo steps in macos.yml, ubuntu.yml, windows.yml). For the escript_packages and escript_script matrix values (which are argument lists), used the xargs-based NUL-delimited tokenization pattern to safely split them into bash arrays, preventing both shell injection and argument-boundary issues.

