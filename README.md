# enerspace-cli-tools

Public distribution channel for enerSpace command-line tool binaries, consumed
by each tool's built-in auto-update.

## create-staging

`create-staging` builds a complete staging copy of a Shopware 6 shop. It ships
as a **single binary**: nothing to install, no setup, no dependencies of its
own.

> ⚠️ **Important:** `create-staging` runs **exclusively on enerSpace
> infrastructure**. Each run verifies the server against our license service;
> on servers outside enerSpace hosting the tool refuses to start.

### What it does

- ⚡ **A full copy in minutes:** files and database of the live shop, with media
  that takes up almost no extra disk space.
- 🔗 **Its own address:** the staging is served at
  `https://your-shop.com/staging-a3f19c2b7d`. The name is random by default, so
  the address cannot be guessed. Pick your own with `--staging-dir=NAME`.
- 🔒 **Password page** (`--protect`): an elegant animated login page guards the
  staging. Forgot the password? `--reset-password` issues a new one in seconds.
- 🕵️ **GDPR anonymization** (`--anonymize`): customer data is anonymized and
  generated documents such as invoices are left out, so external agencies can
  work on the copy safely.
- 🚧 **Never mistaken for the live shop:** mail delivery is off, a staging
  banner is shown, and search engines are kept out.
- 🚚 **Move and remove** (`--migrate`, `--remove`): bring an existing staging to
  the current address, or delete one including the contents of its database.
- 🔄 **Self-updating**, with a live progress display in the terminal and plain
  logs for cron.

### Options at a glance

| Option | Description |
|--------|-------------|
| `--protect` | Password-protect the staging |
| `--protect-allow=<path>` | Paths reachable without password (e.g. `/api`) |
| `--reset-password <SRC>` | New password for an existing staging, no rebuild |
| `--anonymize` | GDPR: anonymize customer data, drop generated documents |
| `--maintenance` | Put the staging into Shopware maintenance mode |
| `--staging-dir=NAME` | Name of the staging, served at `/staging-NAME` |
| `--new` | Create another staging instead of rebuilding an existing one |
| `--migrate <SRC>` | Move an existing staging to the current address |
| `--remove [SRC]` | Remove a staging, including the contents of its database |
| `--keep-db` | With `--remove`: leave the staging database untouched |
| `--media=link\|copy` | Share media with the live shop (default) or copy it |
| `--db-host` / `--db-port` | Staging database connection (default `127.0.0.1:3306`) |
| `--ui=auto\|rich\|plain` | Output style |

Run `./create-staging --help` for the full, always-current list.

### Requirements

- Linux server (x86_64) on enerSpace hosting, reachable via SSH
- Present on the server: `mariadb-dump`, `mysql`, `rsync`, `zstd`, `php`
  (checked on startup, with a clear message if anything is missing)

### Installation

Via SSH, in a directory of your choice:

```bash
curl -4 -L -o create-staging \
  "https://github.com/enerspace/enerspace-cli-tools/releases/latest/download/create-staging-linux-amd64"
chmod +x create-staging
./create-staging --version
```

The `-4` matters: the GitHub download is reachable over IPv4 only.

### Usage

```bash
./create-staging --help
./create-staging --protect httpdocs/ shop_staging staging_user 'DB_PASSWORD'
./create-staging --migrate httpdocs/
./create-staging --remove
```

Install once and you are done: on every run the tool checks for a newer version
and updates itself before doing any work. The
[release history](https://github.com/enerspace/enerspace-cli-tools/releases)
lists what changed in each version.

---

## About this repository

- **Binaries** ship as **GitHub Release assets**, so they stay out of the git
  history. Tag per release: `<tool>-v<version>`, asset `<tool>-<os>-<arch>`.
- **`<tool>/version.json`** holds the latest `{version, sha256}`. The license
  server reads it to know what to offer.
- On every run a tool asks its license server for the latest version, downloads
  the version-pinned asset if newer, verifies the SHA-256 against the signed
  value, replaces itself and re-execs. Fail-open.
- Releases are cut with `create-staging`'s `release.sh <path-to-this-repo>`,
  which builds the hardened binary, creates the release and updates
  `version.json`, all from the same build.
