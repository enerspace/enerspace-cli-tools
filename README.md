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

A release = commit the new binary here **and** register its version + sha256 in
the license server's database — both from the **same** build (garble output is
not byte-reproducible). The license server is **DB-driven**: it reads the latest
version + sha from its `tool_release` table and serves them, signed, in the
verify response. No `.env` edit, no restart.

The `create-staging` repo ships `release.sh`, which does it all:

```
./release.sh /path/to/enerspace-cli-tools [ssh-target] [remote-app-path]
```

It builds the hardened binary, copies it here (commit + push), and runs
`php bin/console app:release create-staging <version> <sha256>` on the license
server (directly over SSH if a target is given, otherwise it prints the command).

> The clients only auto-update from **1.5.0** on (older binaries have no update
> code); deploy the first 1.5.x manually.
