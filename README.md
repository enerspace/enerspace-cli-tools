# enerspace-cli-tools

Public distribution channel for enerSpace command-line tool binaries, consumed
by each tool's built-in auto-update.

## Layout

- **Binaries** ship as **GitHub Release assets** (so they are *not* in the git
  history — the repo stays small). Tag per release: `<tool>-v<version>`, asset
  `<tool>-<os>-<arch>`, e.g. release `create-staging-v1.5.2`, asset
  `create-staging-linux-amd64`.
- **`<tool>/version.json`** (committed, tiny) holds the latest `{version, sha256}`.
  The license server reads this (cached) to know what to offer.

## How updates work

1. A tool contacts its license server on every run. The signed verify response
   carries `latest_version` + `binary_sha256`, which the server reads from
   `…/main/<tool>/version.json`.
2. If newer, the tool downloads the **version-pinned Release asset**
   `https://github.com/enerspace/enerspace-cli-tools/releases/download/<tool>-v<version>/<tool>-<os>-<arch>`,
   verifies the SHA-256 against the signed value, atomically replaces itself and
   re-execs. Fail-open.

## Releasing

Run `create-staging`'s `release.sh` (needs the `gh` CLI):

```
./release.sh /path/to/enerspace-cli-tools
```

It builds the hardened binary, creates the GitHub Release with the binary asset,
and updates `create-staging/version.json` — all from the same build. No license
server change, no restart.

> Clients only auto-update from **1.5.0** on (older binaries have no update
> code); deploy the first one manually.
