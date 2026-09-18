# enerspace-cli-tools

Public distribution channel for enerSpace command-line tool binaries, consumed
by each tool's built-in auto-update.

## create-staging installieren (Deployment)

`create-staging` erstellt eine vollständige Staging-Kopie eines Shopware-6-Shops.
Es besteht aus **einer einzigen Binärdatei** – keine Abhängigkeits-Installation,
kein Setup. Es läuft ausschließlich auf Servern im enerSpace-Hosting.

**Voraussetzungen**

- Linux-Server (x86_64) im enerSpace-Hosting, Zugriff per SSH
- Auf dem Server vorhanden: `mariadb-dump`, `mysql`, `rsync`, `zstd`, `php`
  (das Tool prüft das selbst beim Start und meldet, was ggf. fehlt)

**Installation** – per SSH im gewünschten Verzeichnis (z. B. dem Home-Verzeichnis):

```bash
curl -4 -L -o create-staging \
  "https://github.com/enerspace/enerspace-cli-tools/releases/latest/download/create-staging-linux-amd64"
chmod +x create-staging
./create-staging --version
```

Das `-4` ist wichtig: der GitHub-Download ist nur über IPv4 erreichbar.
Eine bestimmte Version gibt es alternativ versionsgenau unter
`releases/download/create-staging-v<VERSION>/create-staging-linux-amd64`.

**Verwendung**

```bash
./create-staging --help
./create-staging --protect httpdocs/ shop_staging staging_user 'DB_PASSWORT'
```

Alle Optionen (Passwortschutz, DSGVO-Anonymisierung, Wartungsmodus, …) zeigt
`--help`.

**Updates**

Einmal installieren genügt: Das Tool prüft bei jedem echten Lauf auf eine
neuere Version und aktualisiert sich selbst, bevor es losläuft. Was sich
geändert hat, steht in der
[Versionsübersicht](https://github.com/enerspace/enerspace-cli-tools/releases).

---

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
