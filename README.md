# Ubuntu LTS Upgrade Script

A Bash script for fully automated, safe Ubuntu LTS release upgrades without manual interaction.

---

## Description

This script is aimed at administrators who want to standardize and simplify their upgrade process on Ubuntu servers. It relies on `do-release-upgrade`, automatically answers standard prompts, and deliberately aborts on known risk cases – no blind `yes | ...`.

### Key Features

- **Language selection:** English and German are supported; in non-interactive mode, German is selected automatically.
- **Preflight checks:** Verification of the operating system (Ubuntu LTS only), current version, available disk space (default: 6 GB on `/`, 200 MB on `/boot`), APT sources for third-party repositories, and known special cases (RabbitMQ, Dovecot, Postfix).
- **Comprehensive backup:** Saves `/etc`, APT sources, SSH configuration, system information, and DNS state before the upgrade.
- **Unattended mode:** Automatic configuration of `dpkg`, `needrestart`, `debconf`, and `apt` for fully non-interactive upgrades.
- **System updates:** Full package update (`update`, `upgrade`, `dist-upgrade`, `autoremove`) before the release upgrade.
- **Release verification:** Checks whether the target release is officially offered via `do-release-upgrade` – no blind upgrades to unreleased versions.
- **Release upgrade:** Runs `do-release-upgrade` with full session transcript logging.
- **Post-upgrade cleanup:** Removes package leftovers and captures system state after the upgrade.
- **Reboot control:** Configurable automatic or interactive reboot behavior.

### Optional: AdGuard Home / DNS

When `MANAGE_ADGUARD_DNS=yes` is set and a Docker container with the configured name exists, the script ensures that:

- `systemd-resolved` is disabled so AdGuard can take over port 53,
- `/etc/resolv.conf` is set statically to safe DNS servers,
- the AdGuard container is restarted after the upgrade.

Other Docker containers (e.g. Nextcloud, Vaultwarden, Portainer) are not modified.

### Optional: 2FA with Google Authenticator

If `libpam-google-authenticator` is installed, the script **non-destructively** checks whether the PAM entry for SSH is correctly present, and issues a warning if not.

---

## Requirements

- Ubuntu LTS server running **24.04 or later**, to be upgraded to the next LTS version (default: 26.04).
- `do-release-upgrade` must be available (package: `ubuntu-release-upgrader-core`); missing helper packages are installed automatically.
- The script must be run as **root**.

---

## Configuration

The script is fully controlled via environment variables – no changes to the script itself are necessary:

| Variable | Default | Description |
|---|---|---|
| `TARGET_LTS` | `26.04` | Target LTS version |
| `PROMPT_MODE` | `lts` | Release upgrade mode (`lts` or `normal`) |
| `ALLOW_DEVEL_UPGRADE` | `no` | Allow development release upgrade (`yes`/`no`) |
| `UNATTENDED_MODE` | `yes` | Fully automatic mode (`yes`/`no`) |
| `AUTO_CONFIRM` | `yes` | Automatically confirm prompts (`yes`/`no`) |
| `AUTO_REBOOT` | `no` | Automatic reboot after upgrade (`yes`/`no`) |
| `ASK_REBOOT` | `no` | Ask interactively whether to reboot (`yes`/`no`) |
| `ALLOW_THIRD_PARTY` | `no` | Allow third-party repositories (`yes`/`no`) |
| `ALLOW_SPECIAL_CASES` | `no` | Allow known special cases (RabbitMQ, Dovecot) (`yes`/`no`) |
| `MANAGE_ADGUARD_DNS` | `no` | Enable AdGuard/DNS management (`yes`/`no`) |
| `ADGUARD_CONTAINER_NAME` | `adguardhome` | Name of the AdGuard Docker container |
| `MIN_ROOT_FREE_GB` | `6` | Minimum free space on `/` in GB |
| `MIN_BOOT_FREE_MB` | `200` | Minimum free space on `/boot` in MB |
| `LOGFILE` | `/var/log/ubuntu-lts-upgrade.log` | Path to the log file |
| `BACKUP_DIR` | `/root/ubuntu_upgrade_backup_DATE` | Path to the backup directory |
| `LANGUAGE` | `de` | Default language (`de`/`en`) |
| `LANGUAGE_CHOICE` | *(empty)* | Override language without interactive prompt |

### Example with custom variables

```bash
TARGET_LTS=26.04 AUTO_REBOOT=yes MANAGE_ADGUARD_DNS=yes sudo ./upgrade-script.sh
```

---

## Usage

1. Download the script:
   ```bash
   wget https://raw.githubusercontent.com/your-username/your-repo/main/upgrade-script.sh
   chmod +x upgrade-script.sh
   ```

2. Run as root:
   ```bash
   sudo ./upgrade-script.sh
   ```

The script performs all steps automatically. With `UNATTENDED_MODE=yes` (default), no further interaction is required.

---

## Process Overview

1. Language selection (interactive or automatic)
2. Root check
3. Initialize logging
4. **Preflight checks:**
   - OS and LTS verification
   - Version range check (≥ 24.04, < target version)
   - Install missing helper packages
   - Create backup (`/etc`, APT, SSH, system state, DNS)
   - Configure unattended mode
   - Check available disk space
   - Set release prompt
   - Inventory third-party repositories and enforce policy
   - Check for known special cases
   - Non-destructively verify SSH and 2FA configuration
   - Print summary
5. Request confirmation (or proceed automatically)
6. Install all pending package updates
7. Verify release availability via `do-release-upgrade -c`
8. Run release upgrade (with session transcript)
9. AdGuard/DNS adjustment (optional)
10. Post-upgrade cleanup and state snapshot
11. Reboot (automatic, interactive, or manual)

---

## Logging & Troubleshooting

- All actions are logged to `/var/log/ubuntu-lts-upgrade.log`.
- The complete `do-release-upgrade` session transcript is saved as `do-release-upgrade.session.log` in the backup directory.
- Additional Ubuntu upgrade logs are typically found under `/var/log/dist-upgrade/`.
- The backup directory contains before/after snapshots of `dpkg -l`, `lsb_release`, and other system state information.

---

## Safety & Error Protection

- The script uses `set -Eeuo pipefail` and `umask 077`.
- Errors are logged with line number and command; temporary configuration files are automatically removed on exit.
- Third-party repositories cause an abort by default (`ALLOW_THIRD_PARTY=no`).
- Known special cases (RabbitMQ, Dovecot) cause an abort by default (`ALLOW_SPECIAL_CASES=no`).
- Reboot requirements after regular updates are detected and result in a controlled abort with instructions.
- The upgrade command itself is never called with `yes |`.

---

## Customization

The script is modular in design. Individual function blocks can be commented out or removed as needed:

- `manage_adguard_dns_after_upgrade` – AdGuard/DNS adjustment
- `verify_ssh_and_2fa_non_destructive` – SSH and 2FA verification
- `check_special_cases` – Detection of RabbitMQ, Dovecot, Postfix
- `enforce_repo_policy` – Third-party repository policy

---

## Disclaimer

Use this script at your own risk. It is strongly recommended to perform a full system backup before initiating the upgrade. The script creates a configuration backup on its own, but this does not replace a full system backup.

---

## License

This project is licensed under the **MIT License**. See the `LICENSE` file for details.

---

## Contact

For questions or suggestions, feel free to open a [GitHub Issue](../../issues) or send an email to [ptech09@schumacher.or.at](mailto:ptech09@schumacher.or.at).
