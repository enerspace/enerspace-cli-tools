# enerspace-cli-tools

Public distribution channel for enerSpace command-line tool binaries, consumed
by each tool's built-in auto-update.

## Installing create-staging (deployment)

`create-staging` builds a complete staging copy of a Shopware 6 shop. It ships
as a **single binary**: nothing to install, no setup, no dependencies of its
own.

> ⚠️ **Important:** `create-staging` runs **exclusively on enerSpace
> infrastructure**. Each run verifies the server against our license service;
> on servers outside enerSpace hosting the tool refuses to start.

### What it does

- ⚡ **Fast and storage-friendly:** a staging appears in minutes and its media
  takes up (almost) no extra disk space, because it is shared with the live
  shop.
- 🗃️ **Full database copy:** the live database is copied into a separate
  staging database, and the shop gets an identity of its own.
- 📁 **Project files stay private:** keys, logs and generated documents of a
  staging cannot be fetched over the internet.
- 🏷️ **As many environments as you need:** a new one gets an address nobody
  can guess, such as `https://your-shop.com/staging-a3f19c2b7d`, or pick the
  name yourself with `--staging-dir=NAME`. An existing environment keeps its
  address when rebuilt, and `--new` always adds another one.
- 🙈 **Invisible to search engines:** a staging is kept out of search results.
- ⏱️ **Fails fast:** wrong database credentials are reported within seconds,
  before anything is copied.
- 🚚 **Move an existing staging** (`--migrate`): an older staging moves to the
  current address in seconds, without a rebuild. Data, media and password
  protection stay as they are.
- 🗑️ **Remove a staging** (`--remove`): pick one from the list and it is gone,
  including the contents of its database.
- 🚧 **Safe by design:** mail delivery is disabled and a staging banner is
  shown, so the copy can never be mistaken for the live shop.
- 🔒 **Optional password protection** (`--protect`): an elegant animated
  login page guards the staging, images and files included, with path
  exemptions for APIs or health checks and a password that survives rebuilds.
  This is real protection, not a flimsy `.htaccess` prompt: passwords are
  stored only as salted hashes, and repeated wrong attempts lock the client
  out automatically. Forgot the password? `--reset-password` issues a new one
  in seconds.
- 🕵️ **GDPR anonymization** (`--anonymize`): customer data is anonymized and
  generated documents (invoices etc.) are removed from the copy. Perfect for
  third parties: you can safely let external agencies work on this staging.
- 🔄 **Self-updating:** checks for a newer version on every run and updates
  itself before starting.
- 🖥️ **Clean output:** a live progress UI on terminals, plain logs for
  cron/CI.

### Options at a glance

| Option | Description |
|--------|-------------|
| `--protect` | Password-protect the staging front end |
| `--protect-allow=<path>` | Paths reachable without password (e.g. `/api`), repeatable |
| `--reset-password <SRC>` | New password for an existing staging, no rebuild |
| `--anonymize` | GDPR: anonymize customer data, drop generated documents |
| `--maintenance` | Put the staging into Shopware maintenance mode |
| `--staging-dir=NAME` | Name of the staging, served at `/staging-NAME` (default: a random name) |
| `--new` | Create another staging with a random name instead of rebuilding an existing one |
| `--migrate <SRC>` | Move an existing staging to the current address, no rebuild |
| `--remove [SRC]` | Remove a staging: files and the contents of its database |
| `--keep-db` | With `--remove`: leave the staging database untouched |
| `--media=link\|copy` | Share media with the live shop (default) or copy it |
| `--db-host` / `--db-port` | Staging database connection (default `127.0.0.1:3306`) |
| `--ui=auto\|rich\|plain` | Output style (TUI vs. plain logs) |

Run `./create-staging --help` for the full, always-current list.

**Requirements**

- Linux server (x86_64) on enerSpace hosting, reachable via SSH
- Present on the server: `mariadb-dump`, `mysql`, `rsync`, `zstd`, `php`
  (the tool checks for these on startup and tells you if anything is missing)

**Installation** via SSH, in a directory of your choice (e.g. your home
directory):

```bash
curl -4 -L -o create-staging \
  "https://github.com/enerspace/enerspace-cli-tools/releases/latest/download/create-staging-linux-amd64"
chmod +x create-staging
./create-staging --version
```

The `-4` matters: the GitHub download is reachable over IPv4 only.
A specific version can be pinned via
`releases/download/create-staging-v<VERSION>/create-staging-linux-amd64`.

**Usage**

```bash
./create-staging --help
./create-staging --protect httpdocs/ shop_staging staging_user 'DB_PASSWORD'
./create-staging --protect --staging-dir=agentur httpdocs/ shop_stg2 stg2_user 'DB_PASSWORD'
./create-staging --migrate httpdocs/
./create-staging --remove
```

`--help` lists every option: password protection, GDPR anonymization,
maintenance mode, generating a fresh staging password with
`--reset-password`, and more.

**Updates**

Install once and you are done: on every real run the tool checks for a newer
version and updates itself before doing any work. See the
[release history](https://github.com/enerspace/enerspace-cli-tools/releases)
for what changed in each version.

---

## Layout

- **Binaries** ship as **GitHub Release assets** (so they are *not* in the git
  history and the repo stays small). Tag per release: `<tool>-v<version>`, asset
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
and updates `create-staging/version.json`, all from the same build. No license
server change, no restart.

> Clients only auto-update from **1.5.0** on (older binaries have no update
> code); deploy the first one manually.
