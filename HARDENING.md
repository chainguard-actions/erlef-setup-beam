<!-- markdownlint-disable -->

# Hardening Report: erlef--setup-beam/v1.24.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **erlef--setup-beam/v1.24.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across every workflow file use mutable version tags instead of pinned 40-character SHA commit hashes. This exposes the workflows to supply-chain attacks if any referenced action is compromised or its tag is moved. Failing references include: `actions/checkout@v7.0.0`, `actions/setup-node@v6.3.0`, `raven-actions/actionlint@v2.1.2`.

Locations:

- `.github/workflows/action.yml:22`
- `.github/workflows/action.yml:38`
- `.github/workflows/action.yml:42`
- `.github/workflows/action.yml:54`
- `.github/workflows/action.yml:58`
- `.github/workflows/action.yml:70`
- `.github/workflows/action.yml:74`
- `.github/workflows/action.yml:87`
- `.github/workflows/action.yml:91`
- `.github/workflows/action.yml:104`
- `.github/workflows/action.yml:108`
- `.github/workflows/hexpm-mirrors.yml:22`
- `.github/workflows/macos.yml:74`
- `.github/workflows/macos.yml:113`
- `.github/workflows/ubuntu.yml:80`
- `.github/workflows/ubuntu.yml:143`
- `.github/workflows/update_3rd_party_licenses.yml:13`
- `.github/workflows/windows.yml:72`
- `.github/workflows/windows.yml:131`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a), bypassing shell quoting and enabling script injection. (1) In macos.yml, ubuntu.yml, and windows.yml, `steps.setup-beam.outputs.*` values are interpolated directly: e.g. `run: echo "Erlang/OTP ${{steps.setup-beam.outputs.otp-version}}"` — a compromised or malicious action output could inject shell metacharacters. (2) More critically, in macos.yml, ubuntu.yml, and windows.yml, matrix values are interpolated directly into shell commands: `mix escript.install --force ${{matrix.combo.escript_packages}}` and `${{matrix.combo.escript_script}}` — the latter executes the matrix value directly as a shell command, allowing arbitrary code execution if the matrix is attacker-influenced.

Locations:

- `.github/workflows/macos.yml:80`
- `.github/workflows/macos.yml:83`
- `.github/workflows/macos.yml:86`
- `.github/workflows/macos.yml:100`
- `.github/workflows/macos.yml:101`
- `.github/workflows/ubuntu.yml:87`
- `.github/workflows/ubuntu.yml:91`
- `.github/workflows/ubuntu.yml:95`
- `.github/workflows/ubuntu.yml:99`
- `.github/workflows/ubuntu.yml:120`
- `.github/workflows/ubuntu.yml:121`
- `.github/workflows/windows.yml:79`
- `.github/workflows/windows.yml:82`
- `.github/workflows/windows.yml:85`
- `.github/workflows/windows.yml:88`
- `.github/workflows/windows.yml:112`
- `.github/workflows/windows.yml:113`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all unpinned action references by resolving to full SHA hashes: actions/checkout@v7.0.0→9c091bb21b7c1c1d1991bb908d89e4e9dddfe3e0, actions/setup-node@v6.3.0→53b83947a5a98c8d113130e565377fae1a50d02f, raven-actions/actionlint@v2.1.2→205b530c5d9fa8f44ae9ed59f341a0db994aa6f8. Fixed script injection in macos.yml, ubuntu.yml, and windows.yml by moving all ${{ steps.setup-beam.outputs.* }} expressions into env: blocks (OTP_VERSION, ELIXIR_VERSION, GLEAM_VERSION, REBAR3_VERSION). Fixed the escript steps by moving escript_packages and escript_script matrix values into env: blocks and using xargs-based array tokenization to safely split and execute them without shell injection risk.

