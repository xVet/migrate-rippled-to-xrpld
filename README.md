# migrate-rippled-to-xrpld

A bash script that migrates an XRPL core-server node from the old `rippled` package (3.1.3) to the renamed `xrpld` package (3.2.0), following the official XRPL migration guide.

> Reference: [Migrate from rippled to xrpld](https://xrpl.org/docs/infrastructure/installation/migrate-to-xrpld)

## What it does

Runs the migration end to end:

1. Backs up your config and identity (`rippled.cfg`, `validators.txt`, `wallet.db`, `validator-keys.json`) to `/root/rippled-backup`, with an optional full data snapshot.
2. Stops and removes the `rippled` package (`remove`, never `purge`).
3. Installs `xrpld` and stops the auto-started service.
4. Restores your config and validators into `/etc/xrpld`.
5. Migrates the data directories - keep your existing data in place, or re-sync from the network.
6. Restarts `xrpld`.
7. Verifies sync status with `server_info`.

## Before you run

- Update the package-signing GPG key first (steps 1-5 of the Ubuntu/Debian install guide). The script does not do this.
- Run as root (or with `sudo`).
- The backup is written to `/root/rippled-backup` on the same host. Copy it OFF the host yourself before the script removes `rippled` - a backup on the same disk does not protect you from disk failure.

## Usage

```sh
chmod +x migrate.sh
sudo ./migrate.sh --dry-run     # preview, changes nothing
sudo ./migrate.sh               # real run (prompts for mode)
```

## Options

| Flag | Meaning |
| --- | --- |
| `--mode keep\|resync` | Data strategy (default: prompt) |
| `--snapshot` | Take a full tar snapshot of the data dir during backup |
| `--backup-dir DIR` | Local backup dir (default: `/root/rippled-backup`) |
| `--yes` | Do not prompt; assume yes to confirmations |
| `--skip-verify` | Skip the final `server_info` check |
| `--dry-run` | Print what would run, change nothing |
| `-h`, `--help` | Show help |

## Modes

- **keep** - full-history and large-history nodes. Leaves your ledger data in place and hands ownership to the `xrpld` user. Nothing is moved or deleted.
- **resync** - validators and small-history nodes. Points `xrpld` at the new default paths and rebuilds the ledger from the network (minutes).

## Safety model

- Aborts on any error (`set -euo pipefail`) and will not proceed past backup if the backup step fails.
- Never runs `apt-get purge` and never deletes your original data.
- State-changing steps ask for confirmation unless `--yes` is given.
- Detects `apt` vs `yum`.

## What it does NOT do

- Update the package-signing key (do this first).
- Copy your backup off the host (do this yourself).

## Custom paths

If you customised `database_path`, `node_db`, or `debug_logfile`, override the defaults with environment variables (`OLD_DATA_DIR`, `OLD_LOG_DIR`, `BACKUP_DIR`, and so on) or review the data step. The script also reads your real paths from the config and checks ownership before starting.

## Compatibility

- Targets Debian/Ubuntu (`apt`) and RHEL (`yum`) hosts.
- Designed for migrating `rippled` 3.1.3 to `xrpld` 3.2.0, following the official guide.

## License

Released under the [MIT License](LICENSE).

## Disclaimer

Provided as-is, with no warranty. Always run `--dry-run` first, keep an off-host backup, and test on a non-critical node before running on production infrastructure.
