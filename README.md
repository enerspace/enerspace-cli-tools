# enerspace-cli-tools

Public distribution of enerSpace command-line tool binaries, consumed by each
tool's built-in auto-update.

## Layout

```
<tool>/<tool>-<os>-<arch>
```

e.g. `create-staging/create-staging-linux-amd64`.

## How updates work

Each tool contacts its license server on every run; the signed verify response
carries `latest_version` + `binary_sha256`. When a newer version is offered, the
tool downloads the matching binary from

```
https://raw.githubusercontent.com/enerspace/enerspace-cli-tools/main/<tool>/<tool>-<os>-<arch>
```

verifies the SHA-256 against the signed value, atomically replaces itself and
re-execs. Fail-open: no network / no newer version → it just keeps running.

## Releasing a new version

A release = commit the new binary here **and** update the license server's
`LICENSE_UPDATES` (version + sha256) to match — both from the **same** build
(garble output is not byte-reproducible).

The `create-staging` repo ships `release.sh`, which builds the hardened binary,
copies it into this repo and prints the exact `LICENSE_UPDATES` entry to set:

```
./release.sh /path/to/enerspace-cli-tools
```
